# Chapter 2.4: Modules Overview

[← Previous: Ad-hoc Commands](03-ad-hoc-commands.md) | [Next: ansible.cfg Configuration →](05-ansible-cfg-configuration.md)

---

## Overview

Modules are the **building blocks** of Ansible automation. Each module is a self-contained unit of code that performs a specific task — installing packages, managing files, controlling services, and more.

---

## How Modules Work

```
  MODULE EXECUTION LIFECYCLE
  ══════════════════════════════════════════════════════════

  Control Node                              Managed Node
  ┌──────────────┐                         ┌──────────────┐
  │ 1. Load      │       SSH/WinRM         │              │
  │    module    │ ───────────────────────► │ 2. Create    │
  │    code      │    Copy module +         │    temp dir  │
  │              │    arguments             │    /tmp/.ansi│
  │              │                         │              │
  │              │                         │ 3. Execute   │
  │              │                         │    module    │
  │              │                         │    with args │
  │              │                         │              │
  │ 5. Parse     │ ◄─────────────────────── │ 4. Return   │
  │    JSON      │    JSON result           │    JSON     │
  │    result    │                         │    result   │
  │              │                         │              │
  │              │ ───────────────────────► │ 6. Cleanup  │
  │              │    Remove temp files     │    temp dir │
  └──────────────┘                         └──────────────┘
```

---

## Module Categories

```
  ┌─────────────────────────────────────────────────────────────┐
  │                    MODULE CATEGORIES                         │
  ├──────────────┬──────────────────────────────────────────────┤
  │  System      │  user, group, service, systemd, cron,       │
  │              │  hostname, timezone, sysctl, mount           │
  ├──────────────┼──────────────────────────────────────────────┤
  │  Files       │  copy, file, template, fetch, assemble,     │
  │              │  lineinfile, blockinfile, archive, unarchive │
  ├──────────────┼──────────────────────────────────────────────┤
  │  Packages    │  apt, yum, dnf, pip, npm, gem, package      │
  ├──────────────┼──────────────────────────────────────────────┤
  │  Networking  │  uri, get_url, firewalld, ufw, nmcli        │
  ├──────────────┼──────────────────────────────────────────────┤
  │  Cloud       │  ec2_instance, azure_rm_*, gcp_compute_*    │
  ├──────────────┼──────────────────────────────────────────────┤
  │  Database    │  mysql_db, mysql_user, postgresql_db         │
  ├──────────────┼──────────────────────────────────────────────┤
  │  Commands    │  command, shell, raw, script, expect        │
  ├──────────────┼──────────────────────────────────────────────┤
  │  Source Ctrl │  git, subversion                            │
  ├──────────────┼──────────────────────────────────────────────┤
  │  Containers  │  docker_container, docker_image, k8s        │
  ├──────────────┼──────────────────────────────────────────────┤
  │  Crypto      │  openssl_certificate, openssl_privatekey    │
  └──────────────┴──────────────────────────────────────────────┘
```

---

## Most Used Modules with Examples

### `apt` / `yum` / `package` — Package Management

```yaml
# Install specific package (Debian/Ubuntu)
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

# Install multiple packages
- name: Install web stack
  apt:
    name:
      - nginx
      - php-fpm
      - mysql-client
    state: present

# Remove a package
- name: Remove apache
  apt:
    name: apache2
    state: absent
    purge: yes

# Cross-platform (auto-detects OS)
- name: Install package (any OS)
  package:
    name: git
    state: present
```

### `service` / `systemd` — Service Management

```yaml
# Start and enable a service
- name: Start nginx
  service:
    name: nginx
    state: started
    enabled: yes

# Restart a service
- name: Restart nginx
  service:
    name: nginx
    state: restarted

# Using systemd module (more options)
- name: Reload systemd and restart
  systemd:
    name: myapp
    state: restarted
    daemon_reload: yes
```

### `copy` — Copy Files

```yaml
# Copy a file from control node to managed node
- name: Copy config
  copy:
    src: files/nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
    backup: yes

# Copy inline content
- name: Create MOTD
  copy:
    content: "Welcome to {{ inventory_hostname }}\n"
    dest: /etc/motd
```

### `file` — File/Directory Management

```yaml
# Create a directory
- name: Create app directory
  file:
    path: /opt/myapp
    state: directory
    owner: deploy
    group: deploy
    mode: '0755'

# Create a symlink
- name: Link current release
  file:
    src: /opt/myapp/releases/v2.1
    dest: /opt/myapp/current
    state: link

# Delete a file
- name: Remove old config
  file:
    path: /etc/old.conf
    state: absent
```

### `template` — Jinja2 Templates

```yaml
# Render a template
- name: Deploy nginx config
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    mode: '0644'
  notify: Restart nginx
```

### `command` vs `shell` vs `raw`

```yaml
# command - Simple commands (no shell features)
- name: Get hostname
  command: hostname -f
  register: result

# shell - Supports pipes, redirects, env vars
- name: Count processes
  shell: ps aux | wc -l
  register: proc_count

# raw - No Python needed (bootstrapping)
- name: Install Python on bare server
  raw: apt-get install -y python3
```

### `git` — Source Control

```yaml
# Clone a repository
- name: Clone application
  git:
    repo: https://github.com/myorg/myapp.git
    dest: /opt/myapp
    version: main
    force: yes
```

### `user` / `group` — User Management

```yaml
# Create a user
- name: Create deploy user
  user:
    name: deploy
    shell: /bin/bash
    groups: sudo,docker
    append: yes
    create_home: yes
    generate_ssh_key: yes
```

---

## Module Return Values

```
  EVERY MODULE RETURNS JSON
  ══════════════════════════════════════════════════════════

  {
    "changed": true,          ◄── Was something modified?
    "msg": "...",             ◄── Human-readable message
    "rc": 0,                  ◄── Return code (for commands)
    "stdout": "...",          ◄── Standard output
    "stderr": "...",          ◄── Standard error
    "results": [...],         ◄── For loops, list of results
    "ansible_facts": {...}    ◄── New facts to add
  }

  Use 'register' to capture return values:
  ──────────────────────────────────────────
  - name: Check disk space
    command: df -h /
    register: disk_result

  - name: Show disk usage
    debug:
      msg: "{{ disk_result.stdout }}"
```

---

## Module Documentation

```bash
# List all available modules
ansible-doc --list

# Get documentation for a specific module
ansible-doc apt

# Show just the examples
ansible-doc apt -s    # snippet mode

# Search for modules
ansible-doc --list | grep docker
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Module not found | Check Ansible version, install required collection |
| "No such module" error | Verify module name spelling, check `ansible-doc --list` |
| Module returns "changed" every time | Module may not support idempotency (use guards) |
| command/shell always shows changed | Use `changed_when: false` or `creates:` parameter |
| Module argument errors | Check docs with `ansible-doc <module>` |

---

## Summary Table

| Module | Purpose | Key Arguments |
|--------|---------|---------------|
| `apt`/`yum` | Package management | `name`, `state` (present/absent/latest) |
| `service` | Manage services | `name`, `state`, `enabled` |
| `copy` | Copy files | `src`, `dest`, `content`, `mode` |
| `file` | Manage files/dirs | `path`, `state`, `owner`, `mode` |
| `template` | Render Jinja2 templates | `src`, `dest`, `mode` |
| `command` | Run commands (no shell) | Free-form command |
| `shell` | Run commands (with shell) | Free-form command |
| `git` | Git operations | `repo`, `dest`, `version` |
| `user` | Manage users | `name`, `state`, `groups`, `shell` |
| `debug` | Print messages/variables | `msg`, `var` |

---

## Quick Revision Questions

1. **What is the difference between `command` and `shell` modules?**
   > `command` runs without a shell (no pipes/redirects). `shell` runs through `/bin/sh` with full shell features.

2. **How do you view documentation for a module from the CLI?**
   > Use `ansible-doc <module_name>`, e.g., `ansible-doc apt`.

3. **What does the `state: present` parameter mean for the `apt` module?**
   > It ensures the package is installed. If already installed, it does nothing (idempotent).

4. **How do you capture the output of a module for later use?**
   > Use the `register` keyword to save the module's return value into a variable.

5. **When would you use the `raw` module instead of `command`?**
   > When the target node doesn't have Python installed (e.g., bootstrapping a new server).

6. **What is the `package` module and when is it useful?**
   > It's a generic module that auto-detects the OS package manager (apt, yum, etc.), useful for cross-platform playbooks.

---

[← Previous: Ad-hoc Commands](03-ad-hoc-commands.md) | [Next: ansible.cfg Configuration →](05-ansible-cfg-configuration.md)
