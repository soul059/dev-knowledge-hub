# Chapter 7.2: Template Module

[← Previous: Jinja2 Templates](01-jinja2-templates.md) | [Next: File Modules →](03-file-modules.md)

---

## Overview

The **template** module renders a Jinja2 template file from the control node and deploys the result to a managed host. It combines variable substitution, conditional logic, and loops to generate configuration files dynamically.

---

## Basic Usage

```yaml
- name: Deploy Nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
```

```
  TEMPLATE FLOW
  ══════════════════════════════════════════════════════════

  Control Node                      Managed Host
  ┌──────────────────┐              ┌──────────────────┐
  │ templates/       │              │                  │
  │  nginx.conf.j2   │──render──▶   │ /etc/nginx/      │
  │  (Jinja2 + vars) │              │  nginx.conf      │
  └──────────────────┘              │  (final output)  │
                                    └──────────────────┘
```

---

## Module Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `src` | Path to Jinja2 template (relative to `templates/` in role) | Required |
| `dest` | Absolute path on remote host | Required |
| `owner` | File owner | — |
| `group` | File group | — |
| `mode` | File permissions | — |
| `backup` | Create backup before overwrite | `no` |
| `validate` | Command to validate before placing (use `%s`) | — |
| `force` | Replace even if dest exists | `yes` |
| `newline_sequence` | Line-ending style (`\n`, `\r\n`, `\r`) | `\n` |
| `lstrip_blocks` | Strip leading whitespace from blocks | `no` |
| `trim_blocks` | Strip newlines after block tags | `yes` |

---

## Template Source Locations

```
  WHERE ANSIBLE LOOKS FOR TEMPLATES
  ══════════════════════════════════════════════════════════

  In a Role:
    roles/webserver/templates/nginx.conf.j2
    src: nginx.conf.j2           ◄── Relative to templates/

  In a Playbook:
    project/
    ├── playbook.yml
    └── templates/
        └── nginx.conf.j2
    src: templates/nginx.conf.j2  ◄── Relative to playbook
```

---

## Validation Before Deployment

```yaml
# Validate config before replacing
- name: Deploy Nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    validate: nginx -t -c %s       # %s = temp file path
  notify: restart nginx

# Apache validation
- name: Deploy Apache config
  template:
    src: httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf
    validate: apachectl configtest -f %s

# sudoers validation
- name: Deploy sudoers
  template:
    src: sudoers.j2
    dest: /etc/sudoers
    validate: visudo -cf %s
    mode: '0440'
```

If validation fails, the original file is **not** replaced.

---

## Backup Before Overwrite

```yaml
- name: Deploy config with backup
  template:
    src: app.conf.j2
    dest: /etc/myapp/app.conf
    backup: yes
  # Creates: /etc/myapp/app.conf.2024-01-15@10:30:45~
```

---

## ansible_managed Variable

```jinja2
# {{ ansible_managed }}
# This outputs a string like:
# Ansible managed: templates/nginx.conf.j2 modified on 2024-01-15 by user
```

```ini
# ansible.cfg — customise the string
[defaults]
ansible_managed = WARNING: This file is managed by Ansible. Do not edit manually.
```

---

## Whitespace Control in Templates

```yaml
- name: Deploy config with whitespace control
  template:
    src: config.j2
    dest: /etc/app/config.conf
    lstrip_blocks: yes      # Remove leading whitespace from block tags
    trim_blocks: yes        # Remove first newline after block tags
```

Without `lstrip_blocks`:

```
    server app1;
    server app2;
```

With `lstrip_blocks: yes`:

```
server app1;
server app2;
```

---

## Template with loop — /etc/hosts Example

```yaml
# vars
dns_entries:
  - { ip: "10.0.1.10", hostname: "web01", fqdn: "web01.example.com" }
  - { ip: "10.0.1.11", hostname: "web02", fqdn: "web02.example.com" }
  - { ip: "10.0.1.20", hostname: "db01",  fqdn: "db01.example.com" }
```

```jinja2
# templates/hosts.j2
# {{ ansible_managed }}
127.0.0.1   localhost
::1         localhost

# Application hosts
{% for entry in dns_entries %}
{{ entry.ip }}   {{ entry.fqdn }}   {{ entry.hostname }}
{% endfor %}
```

Output:

```
# WARNING: This file is managed by Ansible.
127.0.0.1   localhost
::1         localhost

# Application hosts
10.0.1.10   web01.example.com   web01
10.0.1.11   web02.example.com   web02
10.0.1.20   db01.example.com    db01
```

---

## Template with Conditionals — HAProxy Example

```yaml
# vars
haproxy_backends:
  - name: app_backend
    balance: roundrobin
    servers:
      - { name: app1, ip: "10.0.1.10", port: 8080, weight: 3 }
      - { name: app2, ip: "10.0.1.11", port: 8080, weight: 2 }
    health_check: true
    check_interval: 5000
```

```jinja2
# templates/haproxy.cfg.j2
# {{ ansible_managed }}
global
    daemon
    maxconn 4096

defaults
    mode http
    timeout connect 5s
    timeout client  30s
    timeout server  30s

{% for backend in haproxy_backends %}
backend {{ backend.name }}
    balance {{ backend.balance | default('roundrobin') }}
{% if backend.health_check | default(false) %}
    option httpchk GET /health
{% endif %}
{% for server in backend.servers %}
    server {{ server.name }} {{ server.ip }}:{{ server.port }} weight={{ server.weight | default(1) }}{% if backend.health_check | default(false) %} check inter {{ backend.check_interval | default(5000) }}{% endif %}

{% endfor %}
{% endfor %}
```

---

## Real-World Scenario: Multi-Environment Config

```yaml
# group_vars/production.yml
env_name: production
debug_mode: false
log_level: warn
db_host: db.prod.internal
worker_count: 8

# group_vars/staging.yml
env_name: staging
debug_mode: true
log_level: debug
db_host: db.staging.internal
worker_count: 2
```

```jinja2
# templates/app.conf.j2
# {{ ansible_managed }}
# Environment: {{ env_name }}

[application]
debug = {{ debug_mode | lower }}
log_level = {{ log_level }}
workers = {{ worker_count }}

[database]
host = {{ db_host }}
port = {{ db_port | default(5432) }}
name = {{ db_name | default('myapp') }}

{% if env_name == 'production' %}
[ssl]
enabled = true
cert = /etc/ssl/certs/{{ server_name }}.crt
key = /etc/ssl/private/{{ server_name }}.key
{% endif %}
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `TemplateNotFound` | Check path; in roles use filename only, not `templates/` prefix |
| `UndefinedError` | Variable missing — add `| default()` or define in vars |
| Validation fails | Test validate command manually on the host |
| Wrong line endings | Set `newline_sequence: "\r\n"` for Windows targets |
| Extra blank lines | Use `trim_blocks: yes` and `lstrip_blocks: yes` |
| Permission denied on dest | Check `become: yes` and correct `owner`/`mode` |

---

## Summary Table

| Parameter | Purpose |
|-----------|---------|
| `src` | Template file (Jinja2) |
| `dest` | Destination path on remote host |
| `validate` | Validate before replacing (`%s` = temp file) |
| `backup` | Keep previous version |
| `mode`/`owner`/`group` | File permissions and ownership |
| `lstrip_blocks` | Clean indentation in output |
| `trim_blocks` | Remove newlines after block tags |
| `newline_sequence` | Control line endings |

---

## Quick Revision Questions

1. **Where does Ansible look for template files in a role?**
   > In the role's `templates/` directory. You reference just the filename (e.g., `src: nginx.conf.j2`).

2. **How does the `validate` parameter work?**
   > Ansible writes the rendered template to a temp file, runs the validate command with `%s` replaced by the temp path. If validation fails, the original file is not replaced.

3. **What does `backup: yes` do?**
   > Creates a timestamped copy of the existing file before overwriting it.

4. **How do you include the `ansible_managed` comment in a template?**
   > Add `# {{ ansible_managed }}` at the top of the template file.

5. **What do `trim_blocks` and `lstrip_blocks` control?**
   > `trim_blocks` removes the first newline after a block tag. `lstrip_blocks` removes leading whitespace from block tag lines.

---

[← Previous: Jinja2 Templates](01-jinja2-templates.md) | [Next: File Modules →](03-file-modules.md)
