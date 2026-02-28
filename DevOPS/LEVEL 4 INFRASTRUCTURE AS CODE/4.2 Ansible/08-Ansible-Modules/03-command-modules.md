# Chapter 8.3: Command Modules

[← Previous: Service Modules](02-service-modules.md) | [Next: User and Group Modules →](04-user-and-group-modules.md)

---

## Overview

Ansible provides four modules for executing commands on remote hosts: **command**, **shell**, **raw**, and **script**. Each has different capabilities and trade-offs regarding safety, idempotency, and flexibility.

---

## Module Comparison

```
  COMMAND MODULES — WHEN TO USE WHICH
  ══════════════════════════════════════════════════════════

  command    Safest. No shell features. Use when possible.
  shell      Shell features (pipes, redirects). Less safe.
  raw        No Python needed. Bootstrap use only.
  script     Run local script on remote. No copy needed.
```

| Feature | `command` | `shell` | `raw` | `script` |
|---------|:---------:|:-------:|:-----:|:--------:|
| Shell features (pipes, redirects) | ✗ | ✓ | ✓ | — |
| Requires Python on remote | ✓ | ✓ | ✗ | ✗ |
| `creates`/`removes` support | ✓ | ✓ | ✗ | ✓ |
| `changed_when` recommended | ✓ | ✓ | ✓ | ✓ |
| Security risk | Low | Medium | High | Low |
| Idempotent by default | ✗ | ✗ | ✗ | ✗ |

---

## command Module

Executes a command **without** a shell. No pipes, redirects, or environment variable expansion.

```yaml
# Basic command
- name: Check disk space
  command: df -h
  register: disk_info
  changed_when: false

# With arguments
- name: Create database
  command: createdb myapp_production
  become_user: postgres

# Idempotent with creates
- name: Initialise database (only if not exists)
  command: /opt/myapp/bin/init-db.sh
  args:
    creates: /opt/myapp/data/db.sqlite    # Skip if file exists

# Idempotent with removes
- name: Remove temp files
  command: /opt/cleanup.sh
  args:
    removes: /tmp/myapp-build             # Skip if file doesn't exist

# Run from specific directory
- name: Run migrations
  command: python manage.py migrate
  args:
    chdir: /opt/myapp
```

### Key Parameters

| Parameter | Description |
|-----------|-------------|
| `cmd` / free-form | Command to run |
| `chdir` | Change to directory before running |
| `creates` | Skip if this file exists |
| `removes` | Skip if this file does NOT exist |
| `argv` | List of arguments (alternative to string) |

---

## shell Module

Executes through `/bin/sh`. Supports pipes, redirects, and environment variables.

```yaml
# Pipes
- name: Count running processes
  shell: ps aux | wc -l
  register: process_count
  changed_when: false

# Redirects
- name: Export database
  shell: pg_dump myapp > /backup/myapp-{{ ansible_date_time.date }}.sql
  become_user: postgres

# Environment variables
- name: Run with custom env
  shell: echo $APP_ENV
  environment:
    APP_ENV: production

# Multi-line command
- name: Build application
  shell: |
    cd /opt/myapp
    source venv/bin/activate
    pip install -r requirements.txt
    python manage.py collectstatic --noinput
  args:
    executable: /bin/bash

# Idempotent pattern
- name: Create SSL certificate (only if missing)
  shell: |
    openssl req -x509 -nodes -days 365 \
      -newkey rsa:2048 \
      -keyout /etc/ssl/private/server.key \
      -out /etc/ssl/certs/server.crt \
      -subj "/CN={{ server_name }}"
  args:
    creates: /etc/ssl/certs/server.crt
```

---

## raw Module

Executes directly via SSH — no Python required on the remote host.

```yaml
# Bootstrap Python (before Ansible modules can work)
- name: Install Python on bare host
  raw: apt-get install -y python3
  become: yes

# Check connectivity
- name: Test SSH connection
  raw: echo "Hello from $(hostname)"
  changed_when: false

# Bootstrap for new hosts
- name: Bootstrap Ansible prerequisites
  hosts: new_hosts
  gather_facts: false     # Can't gather facts without Python
  tasks:
    - name: Install Python
      raw: |
        test -e /usr/bin/python3 || \
        (apt-get update && apt-get install -y python3)
      become: yes
      changed_when: false

    - name: Now gather facts
      setup:
```

---

## script Module

Transfers a **local** script to the remote host and executes it.

```yaml
# Run a local script on remote hosts
- name: Run deployment script
  script: scripts/deploy.sh
  args:
    creates: /opt/myapp/.deployed

# With arguments
- name: Run setup script with args
  script: scripts/setup.sh --env production --port 8080

# With specific interpreter
- name: Run Python script
  script: scripts/health_check.py
  args:
    executable: /usr/bin/python3

# With chdir
- name: Run script in directory
  script: scripts/build.sh
  args:
    chdir: /opt/myapp
```

---

## Making Commands Idempotent

### creates / removes

```yaml
- name: Extract archive (idempotent)
  command: tar xzf /tmp/app.tar.gz -C /opt/
  args:
    creates: /opt/app/bin/app     # Don't run if already extracted

- name: Run cleanup (only if needed)
  command: /opt/cleanup.sh
  args:
    removes: /tmp/build-artifacts  # Only run if dir exists
```

### changed_when / failed_when

```yaml
# Read-only commands — never "changed"
- name: Check version
  command: nginx -v
  register: nginx_version
  changed_when: false

# Changed only when output indicates action
- name: Apply database migration
  command: python manage.py migrate
  args:
    chdir: /opt/myapp
  register: migration_result
  changed_when: "'Applying' in migration_result.stdout"

# Custom failure condition
- name: Check application health
  shell: curl -s http://localhost:8000/health
  register: health
  failed_when: "'healthy' not in health.stdout"
  changed_when: false
```

---

## Return Values

```yaml
- name: Run a command
  command: whoami
  register: result

- name: Show return values
  debug:
    msg: |
      stdout: {{ result.stdout }}
      stderr: {{ result.stderr }}
      rc: {{ result.rc }}
      changed: {{ result.changed }}
      stdout_lines: {{ result.stdout_lines }}
```

| Return Value | Description |
|-------------|-------------|
| `result.stdout` | Standard output as string |
| `result.stderr` | Standard error as string |
| `result.rc` | Return code (0 = success) |
| `result.changed` | Whether task changed anything |
| `result.stdout_lines` | stdout as list of lines |
| `result.start` | Command start time |
| `result.end` | Command end time |
| `result.delta` | Command duration |

---

## Real-World Scenario: Application Build and Deploy

```yaml
---
- name: Build and deploy application
  hosts: build_server
  become: yes
  vars:
    app_dir: /opt/myapp
    app_version: "2.1.0"

  tasks:
    - name: Check if already deployed
      stat:
        path: "{{ app_dir }}/releases/{{ app_version }}"
      register: release_dir

    - name: Clone repository
      command: >
        git clone --branch v{{ app_version }}
        https://github.com/org/myapp.git
        {{ app_dir }}/releases/{{ app_version }}
      when: not release_dir.stat.exists

    - name: Install dependencies
      shell: |
        cd {{ app_dir }}/releases/{{ app_version }}
        python3 -m venv venv
        source venv/bin/activate
        pip install -r requirements.txt
      args:
        creates: "{{ app_dir }}/releases/{{ app_version }}/venv"
        executable: /bin/bash

    - name: Run database migrations
      command: >
        {{ app_dir }}/releases/{{ app_version }}/venv/bin/python
        manage.py migrate --noinput
      args:
        chdir: "{{ app_dir }}/releases/{{ app_version }}"
      register: migrate_result
      changed_when: "'Applying' in migrate_result.stdout"

    - name: Run tests
      command: >
        {{ app_dir }}/releases/{{ app_version }}/venv/bin/python
        -m pytest tests/
      args:
        chdir: "{{ app_dir }}/releases/{{ app_version }}"
      changed_when: false
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `command` fails with pipes | Use `shell` module instead |
| Shell injection risk | Prefer `command` over `shell`; avoid user input in commands |
| Always shows "changed" | Add `changed_when: false` for read-only commands |
| Command not found | Provide full path or set `environment` with correct PATH |
| Script fails on remote | Check shebang line and file permissions |
| `raw` fails | Ensure SSH access works without Python/Ansible |

---

## Summary Table

| Module | Use Case | Shell Features | Python Required |
|--------|----------|:--------------:|:---------------:|
| `command` | Simple commands, safest option | ✗ | ✓ |
| `shell` | Pipes, redirects, env vars needed | ✓ | ✓ |
| `raw` | Bootstrap, no Python on target | ✓ | ✗ |
| `script` | Run local script on remote | — | ✗ |

---

## Quick Revision Questions

1. **Why is `command` preferred over `shell`?**
   > `command` doesn't use a shell, so it avoids shell injection risks and unexpected interpretation of special characters.

2. **When should you use `raw` instead of `command`?**
   > When the remote host doesn't have Python installed (e.g., bootstrapping a new host).

3. **How do you make a `command` task idempotent?**
   > Use `creates:` (skip if file exists) or `removes:` (skip if file missing), or add `changed_when` logic based on output.

4. **What does `changed_when: false` do?**
   > It tells Ansible the task never makes changes, so it always reports "ok" instead of "changed".

5. **How does the `script` module differ from `command`?**
   > `script` transfers a local script to the remote host and runs it. `command` runs a command already available on the remote host.

---

[← Previous: Service Modules](02-service-modules.md) | [Next: User and Group Modules →](04-user-and-group-modules.md)
