# Chapter 4.8: Tags

[← Previous: Blocks and Error Handling](07-blocks-and-error-handling.md) | [Next: Variable Precedence →](../05-Variables-and-Facts/01-variable-precedence.md)

---

## Overview

**Tags** let you selectively run or skip specific tasks, roles, or plays in a playbook. Instead of running the entire playbook every time, you can target only the parts you need — useful for debugging, partial runs, and separating concerns like "install" vs "configure".

---

## Basic Tag Syntax

```yaml
tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present
    tags:
      - install
      - nginx

  - name: Configure nginx
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    tags:
      - configure
      - nginx

  - name: Start nginx
    service:
      name: nginx
      state: started
    tags:
      - service
      - nginx

  # Shorthand — single tag
  - name: Install curl
    apt:
      name: curl
      state: present
    tags: install

  # Shorthand — inline list
  - name: Deploy app
    copy:
      src: app/
      dest: /opt/app/
    tags: [deploy, app]
```

---

## Running with Tags

```bash
# Run ONLY tasks with specific tag
ansible-playbook site.yml --tags "configure"

# Run multiple tags
ansible-playbook site.yml --tags "install,configure"

# SKIP tasks with specific tag
ansible-playbook site.yml --skip-tags "install"

# Skip multiple tags
ansible-playbook site.yml --skip-tags "debug,test"

# Combine: run some, skip others
ansible-playbook site.yml --tags "nginx" --skip-tags "install"

# List all available tags
ansible-playbook site.yml --list-tags
```

```
  TAG FILTERING
  ══════════════════════════════════════════════════════════

  Playbook tasks:
  ┌─────────────────────────┬──────────────────────┐
  │ Task                    │ Tags                 │
  ├─────────────────────────┼──────────────────────┤
  │ Install nginx           │ install, nginx       │
  │ Configure nginx         │ configure, nginx     │
  │ Install PostgreSQL      │ install, database    │
  │ Configure PostgreSQL    │ configure, database  │
  │ Deploy application      │ deploy, app          │
  │ Run tests               │ test                 │
  └─────────────────────────┴──────────────────────┘

  --tags "nginx"
  → Install nginx ✅
  → Configure nginx ✅
  → (all others skipped)

  --tags "install"
  → Install nginx ✅
  → Install PostgreSQL ✅
  → (all others skipped)

  --skip-tags "test"
  → Everything EXCEPT "Run tests"

  --tags "nginx" --skip-tags "install"
  → Configure nginx ✅ only
```

---

## Tags on Plays and Roles

```yaml
---
# Tag on an entire play
- name: Configure web servers
  hosts: webservers
  tags: webserver               # ALL tasks in this play get this tag
  tasks:
    - name: Install nginx
      apt:
        name: nginx
    - name: Configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf

# Tags on roles
- name: Full stack setup
  hosts: all
  roles:
    - role: common
      tags: common

    - role: nginx
      tags: [nginx, webserver]

    - role: postgresql
      tags: [postgresql, database]

    - role: monitoring
      tags: monitoring
```

---

## Tags on Blocks

```yaml
tasks:
  - block:
      - name: Install PostgreSQL
        apt:
          name: postgresql
          state: present

      - name: Start PostgreSQL
        service:
          name: postgresql
          state: started

      - name: Create database
        postgresql_db:
          name: mydb
    tags: database                    # All tasks in block get this tag
    become: yes
```

---

## Tags on Includes and Imports

```yaml
tasks:
  # import_tasks — tags cascade to imported tasks
  - import_tasks: tasks/install.yml
    tags: install
    # Every task in install.yml gets the 'install' tag

  # include_tasks — tag applies to the include itself
  - include_tasks: tasks/configure.yml
    tags: configure
    # The tag applies to the include action;
    # Tasks inside ALSO need 'configure' tag to match,
    # OR the include only runs when 'configure' is specified

  # Practical difference:
  # import: --list-tasks shows imported tasks with cascaded tags
  # include: --list-tasks shows "include_tasks: ..." only
```

---

## Special Tags

### always

Tasks tagged `always` run **every time**, regardless of `--tags` filter:

```yaml
tasks:
  - name: Gather system info
    setup:
    tags: always                  # Runs even with --tags "deploy"

  - name: Install packages
    apt:
      name: nginx
    tags: install

  - name: Deploy code
    copy:
      src: app/
      dest: /opt/app/
    tags: deploy

  - name: Log execution
    lineinfile:
      path: /var/log/ansible.log
      line: "{{ ansible_date_time.iso8601 }} - playbook ran"
    tags: always                  # Always logs, no matter what tags
```

> **Note:** `--skip-tags always` is the only way to skip `always` tasks.

### never

Tasks tagged `never` are **skipped by default** unless explicitly included:

```yaml
tasks:
  - name: Normal task
    debug:
      msg: "This runs normally"

  - name: Debug info (hidden by default)
    debug:
      var: hostvars[inventory_hostname]
    tags: never                   # Skipped unless: --tags "never"

  - name: Dangerous cleanup
    file:
      path: /opt/app
      state: absent
    tags:
      - never                     # Skipped by default
      - cleanup                   # Run with: --tags "cleanup"
```

```bash
# This runs the 'cleanup' tagged task (overrides 'never')
ansible-playbook site.yml --tags "cleanup"
```

---

## Tag Strategies and Best Practices

### Consistent Naming Convention

```yaml
# Good: Verb-based tags
tags: [install, configure, deploy, verify, cleanup]

# Good: Component-based tags
tags: [nginx, postgresql, redis, app]

# Good: Combined
tags: [install-nginx, configure-nginx, deploy-app]
```

### Common Tag Patterns

```
  RECOMMENDED TAG TAXONOMY
  ══════════════════════════════════════════════════════════

  By Action:
  ├── install        — Package installation
  ├── configure      — Configuration file management
  ├── deploy         — Application deployment
  ├── service        — Service management (start/stop)
  ├── verify         — Health checks & verification
  ├── cleanup        — Remove temp files, old versions
  └── debug          — Debugging output (tag: never)

  By Component:
  ├── nginx
  ├── postgresql
  ├── redis
  ├── app
  └── monitoring

  Special:
  ├── always         — Always runs
  ├── never          — Skipped unless requested
  └── setup          — Initial setup only
```

### Full Example

```yaml
---
- name: Web server configuration
  hosts: webservers
  become: yes
  tasks:
    # Always run
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      tags: always

    # Installation
    - name: Install nginx
      apt:
        name: nginx
        state: present
      tags: [install, nginx]

    - name: Install certbot
      apt:
        name: certbot
        state: present
      tags: [install, ssl]

    # Configuration
    - name: Deploy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      tags: [configure, nginx]
      notify: Reload nginx

    - name: Deploy SSL certificate
      copy:
        src: "{{ ssl_cert }}"
        dest: /etc/ssl/certs/
      tags: [configure, ssl]
      notify: Reload nginx

    # Deployment
    - name: Deploy application
      synchronize:
        src: app/
        dest: /var/www/html/
      tags: [deploy, app]

    # Verification
    - name: Check nginx syntax
      command: nginx -t
      changed_when: false
      tags: [verify, nginx]

    - name: Check site responds
      uri:
        url: "http://localhost"
        status_code: 200
      tags: [verify]

    # Debug (hidden by default)
    - name: Show all variables
      debug:
        var: hostvars[inventory_hostname]
      tags: [never, debug]

  handlers:
    - name: Reload nginx
      service:
        name: nginx
        state: reloaded
```

```bash
# Fresh setup
ansible-playbook site.yml

# Just deploy new code
ansible-playbook site.yml --tags "deploy"

# Install and configure only
ansible-playbook site.yml --tags "install,configure"

# Verify everything
ansible-playbook site.yml --tags "verify"

# Debug with verbose output
ansible-playbook site.yml --tags "debug" -v

# Everything except SSL
ansible-playbook site.yml --skip-tags "ssl"
```

---

## Real-World Scenario: CI/CD Pipeline Integration

```yaml
# pipeline.yml — Different CI stages use different tags
---
- name: Application lifecycle
  hosts: appservers
  become: yes
  tasks:
    - name: Run unit tests
      command: pytest /opt/app/tests/
      tags: [test, ci]
      changed_when: false

    - name: Build application
      command: /opt/app/build.sh
      tags: [build, ci]

    - name: Deploy to staging
      copy:
        src: build/
        dest: /opt/app/
      tags: [deploy, staging]

    - name: Run integration tests
      command: pytest /opt/app/integration/
      tags: [test, integration, staging]
      changed_when: false

    - name: Deploy to production
      copy:
        src: build/
        dest: /opt/app/
      tags: [deploy, production]

    - name: Run smoke tests
      uri:
        url: "http://localhost:{{ app_port }}/health"
        status_code: 200
      tags: [test, smoke, production]

    - name: Cleanup build artifacts
      file:
        path: /tmp/build-cache
        state: absent
      tags: [never, cleanup]
```

```bash
# CI pipeline stages
# Stage 1: Build & Test
ansible-playbook pipeline.yml --tags "build,test,ci"

# Stage 2: Deploy to staging
ansible-playbook pipeline.yml --tags "deploy,staging"

# Stage 3: Integration tests
ansible-playbook pipeline.yml --tags "test,integration"

# Stage 4: Deploy to production
ansible-playbook pipeline.yml --tags "deploy,production"

# Stage 5: Smoke tests
ansible-playbook pipeline.yml --tags "test,smoke"

# Manual: Cleanup
ansible-playbook pipeline.yml --tags "cleanup"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Tasks skipped unexpectedly | Check that `--tags` matches the task's tags exactly |
| `always` task is skipping | Only `--skip-tags always` can skip it; check playbook syntax |
| `never` task runs when unwanted | `--tags "never"` or any other tag on the task enables it |
| Tags don't cascade into includes | Use `import_tasks` instead; `include_tasks` tags behave differently |
| `--list-tags` missing tags | Dynamic includes aren't resolved at parse time; use imports |
| Handlers not running with tags | Handlers run if notified, regardless of tags (unless handler has tag) |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| `tags:` | Label tasks for selective execution |
| `--tags` | Run only matching tasks |
| `--skip-tags` | Skip matching tasks |
| `--list-tags` | Show all tags in playbook |
| `always` tag | Task always runs (unless `--skip-tags always`) |
| `never` tag | Task skipped unless explicitly included |
| Tag on block | Applies to all tasks in the block |
| Tag on role | Applies to all tasks in the role |
| Tag on import | Cascades to imported tasks |
| Tag on include | Applies to the include action only |

---

## Quick Revision Questions

1. **How do you run only tasks tagged "deploy"?**
   > `ansible-playbook site.yml --tags "deploy"`

2. **What does the `always` special tag do?**
   > Tasks tagged `always` run every time the playbook is executed, regardless of which `--tags` filter is used.

3. **What does the `never` special tag do?**
   > Tasks tagged `never` are skipped by default. They only run if explicitly included with `--tags "never"` or another tag on the same task.

4. **How do tags behave differently between `import_tasks` and `include_tasks`?**
   > Tags on `import_tasks` cascade down to all imported tasks. Tags on `include_tasks` apply to the include action itself, not to individual tasks inside.

5. **How do you see all available tags in a playbook?**
   > Run `ansible-playbook site.yml --list-tags`.

6. **Do handlers need tags to run?**
   > No. Handlers run when notified regardless of tags, unless the handler itself has a tag and `--tags` is specified.

---

[← Previous: Blocks and Error Handling](07-blocks-and-error-handling.md) | [Next: Variable Precedence →](../05-Variables-and-Facts/01-variable-precedence.md)
