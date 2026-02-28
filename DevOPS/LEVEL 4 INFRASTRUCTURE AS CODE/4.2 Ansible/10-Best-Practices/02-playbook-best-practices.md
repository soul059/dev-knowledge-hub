# Chapter 10.2: Playbook Best Practices

[← Previous: Directory Layout](01-directory-layout.md) | [Next: Security Best Practices →](03-security-best-practices.md)

---

## Overview

Well-written playbooks are readable, maintainable, idempotent, and reusable. This chapter codifies the practices that separate production-quality Ansible code from quick scripts.

---

## Naming and Readability

### Always Name Tasks

```yaml
# BAD — no name, hard to debug
- apt:
    name: nginx
    state: present

# GOOD — descriptive name
- name: Install nginx web server
  apt:
    name: nginx
    state: present
```

### Use Consistent Naming Style

```
  NAMING CONVENTIONS
  ══════════════════════════════════════════════════════════

  Tasks:    "Verb + Object"       Install nginx
                                  Deploy app config
                                  Restart database service

  Vars:     snake_case            http_port
                                  max_connections
                                  ssl_cert_path

  Roles:    kebab-case or         nginx-server
            snake_case            database_primary

  Files:    kebab-case            deploy-app.yml
                                  web-servers.yml
```

---

## YAML Style

### Use Native YAML, Not Inline

```yaml
# BAD — inline style (hard to read)
- name: Install nginx
  apt: name=nginx state=present update_cache=yes

# GOOD — YAML dictionary style
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: true
```

### Use `true`/`false` Not `yes`/`no`

```yaml
# Recommended (explicit boolean)
become: true
gather_facts: false
update_cache: true

# Avoid (YAML 1.1 truthy strings)
become: yes
gather_facts: no
```

### Quote Strings That Look Like Other Types

```yaml
# GOOD — quoted to prevent misinterpretation
version: "1.0"              # Not a float!
port: "8080"                # When you want a string
enabled: "yes"              # When "yes" is a literal string, not boolean
```

---

## Idempotency

```
  IDEMPOTENT                     NOT IDEMPOTENT
  ═══════════════                ══════════════════

  apt:                           command: apt install nginx
    name: nginx                  shell: echo "line" >> file
    state: present               raw: useradd deploy

  ↓                              ↓
  Run 10 times = same result     Run 10 times = broken state
```

### Rules for Idempotency

```yaml
# 1. Use modules instead of shell/command
- name: Create user (GOOD)
  user:
    name: deploy
    state: present

# 2. If you must use shell, add creates/removes
- name: Compile app (conditionally)
  command: make install
  args:
    chdir: /opt/app
    creates: /opt/app/bin/myapp     # Skip if already exists

# 3. Use changed_when for scripts
- name: Run migration
  command: python manage.py migrate --check
  register: migration_check
  changed_when: "'No migrations' not in migration_check.stdout"
```

---

## Variable Best Practices

### Use Role Defaults for Overridable Values

```yaml
# roles/webserver/defaults/main.yml
---
webserver_port: 80              # Easy to override
webserver_max_connections: 256
webserver_worker_processes: auto
```

### Prefix Role Variables

```yaml
# BAD — generic name, easy collision
port: 80
max_conn: 256

# GOOD — role-prefixed
webserver_port: 80
webserver_max_connections: 256
database_port: 5432
database_max_connections: 100
```

### Don't Hardcode — Parameterise

```yaml
# BAD
- name: Install specific version
  apt:
    name: nginx=1.18.0-1
    state: present

# GOOD
- name: Install configured nginx version
  apt:
    name: "nginx={{ nginx_version }}"
    state: present
```

---

## Task Organisation

### Use Roles for Reusable Logic

```yaml
# BAD — everything inline
- hosts: webservers
  tasks:
    - name: Install nginx
      apt: ...
    - name: Configure nginx
      template: ...
    - name: Start nginx
      service: ...
    # 100 more tasks...

# GOOD — organised into roles
- hosts: webservers
  roles:
    - common
    - webserver
    - monitoring
```

### Use `block` for Logical Grouping

```yaml
- name: Database setup
  block:
    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present

    - name: Configure PostgreSQL
      template:
        src: postgresql.conf.j2
        dest: /etc/postgresql/14/main/postgresql.conf

    - name: Start PostgreSQL
      service:
        name: postgresql
        state: started
  rescue:
    - name: Log failure
      debug:
        msg: "Database setup failed on {{ inventory_hostname }}"
  when: "'dbservers' in group_names"
  become: true
```

### Use `import_tasks` for Static Inclusion

```yaml
# tasks/main.yml
---
- import_tasks: install.yml
- import_tasks: configure.yml
- import_tasks: service.yml
```

---

## Handler Best Practices

```yaml
# Use descriptive handler names
handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted

  # Use listen for multiple triggers
  - name: Restart web stack
    service:
      name: nginx
      state: restarted
    listen: "web config changed"

  - name: Clear cache
    command: /usr/local/bin/clear-cache
    listen: "web config changed"
```

```yaml
# Notify with listen keyword
tasks:
  - name: Update nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: "web config changed"    # Triggers both handlers
```

---

## Conditional and Loop Patterns

```yaml
# Prefer 'when' over 'shell if'
- name: Install EPEL on RedHat
  yum:
    name: epel-release
    state: present
  when: ansible_os_family == "RedHat"

# Use lists for multiple packages
- name: Install required packages
  apt:
    name:
      - nginx
      - python3
      - git
    state: present
  # Don't loop — apt handles lists natively

# Use loop only when needed
- name: Create application users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: app1, groups: www-data }
    - { name: app2, groups: www-data }
```

---

## Check Mode and Diff Mode

```yaml
# Always support check mode in custom tasks
- name: Deploy application
  command: /opt/deploy.sh
  when: not ansible_check_mode    # Skip in check mode

# Use --check and --diff regularly
# ansible-playbook site.yml --check --diff
```

```
  CHECK + DIFF OUTPUT
  ══════════════════════════════════════════════════════════

  TASK [Deploy nginx config] ********************************
  --- before: /etc/nginx/nginx.conf
  +++ after: /tmp/nginx.conf.j2
  @@ -1,3 +1,3 @@
  -worker_processes 2;
  +worker_processes 4;

  changed: [web01] (check mode, no changes made)
```

---

## Tags Strategy

```yaml
# Use tags consistently
- name: Install packages
  apt:
    name: nginx
    state: present
  tags:
    - install
    - nginx
    - packages

- name: Configure nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  tags:
    - configure
    - nginx

# Standard tag categories:
# install, configure, deploy, service, security, monitoring
```

```bash
# Run only tagged tasks
ansible-playbook site.yml --tags "configure"
ansible-playbook site.yml --skip-tags "install"
```

---

## Real-World Scenario: Playbook Review Checklist

```
  CODE REVIEW CHECKLIST FOR ANSIBLE
  ══════════════════════════════════════════════════════════

  □ Every task has a descriptive name
  □ Native YAML syntax (not inline key=value)
  □ Booleans use true/false (not yes/no)
  □ Variables are snake_case and role-prefixed
  □ No hardcoded values — use variables
  □ Modules used instead of shell/command where possible
  □ shell/command tasks use creates/removes or changed_when
  □ Secrets use Vault, not plain text
  □ Handlers used for service restarts
  □ Tags applied consistently
  □ Works in check mode (--check)
  □ Tested with ansible-lint
  □ Plays are idempotent (safe to re-run)
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Playbook not idempotent | Replace shell/command with proper modules; add `creates`/`changed_when` |
| Variable name conflicts | Prefix all role variables with the role name |
| Too many tasks in one file | Split into roles or use `import_tasks` |
| Handler not triggered | Check notify string matches handler name exactly |
| Lint warnings | Run `ansible-lint` and fix all warnings |
| `yes`/`no` interpreted wrong | Use `true`/`false` for booleans, quote strings |

---

## Summary Table

| Practice | Recommendation |
|----------|---------------|
| Task names | Always use descriptive "Verb + Object" names |
| YAML style | Native YAML dict, not inline `key=value` |
| Booleans | Use `true`/`false` |
| Variables | snake_case, role-prefixed, in defaults/ |
| Idempotency | Use modules; add creates/changed_when for commands |
| Organisation | Roles > long task lists |
| Handlers | Use `listen` for multi-action triggers |
| Packages | Pass list to module, don't loop |
| Tags | Consistent categories (install, configure, deploy) |
| Review | ansible-lint + check mode + diff |

---

## Quick Revision Questions

1. **Why should every task have a `name`?**
   > Names make playbook output readable, simplify debugging, and serve as documentation for what each task does.

2. **What makes a playbook idempotent?**
   > Using modules with declarative state (e.g., `state: present`) and guarding commands with `creates`, `removes`, or `changed_when` so re-runs produce no changes.

3. **Why should role variables be prefixed?**
   > To prevent naming collisions when multiple roles are used together (e.g., `webserver_port` vs `database_port` instead of both using `port`).

4. **When should you use `shell`/`command` vs a module?**
   > Prefer modules for any supported operation. Use shell/command only when no suitable module exists, and always add idempotency guards.

5. **What is the benefit of `listen` in handlers?**
   > It lets multiple handlers respond to a single event name, so one `notify` can trigger several related actions (restart + cache clear).

---

[← Previous: Directory Layout](01-directory-layout.md) | [Next: Security Best Practices →](03-security-best-practices.md)
