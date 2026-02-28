# Chapter 3.3: Inventory Variables

[← Previous: Dynamic Inventory](02-dynamic-inventory.md) | [Next: Groups and Group Variables →](04-groups-and-group-variables.md)

---

## Overview

Inventory variables let you assign **per-host or per-group settings** that customize how Ansible connects to and configures managed nodes. Variables can be defined inline, in separate files, or inherited through group hierarchies.

---

## Where to Define Variables

```
  INVENTORY VARIABLE LOCATIONS
  ══════════════════════════════════════════════════════════

  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │  1. INLINE (in inventory file)                       │
  │     web1 ansible_host=10.0.1.1 http_port=80         │
  │                                                      │
  │  2. GROUP VARS SECTION                               │
  │     [webservers:vars]                                │
  │     http_port=80                                     │
  │                                                      │
  │  3. group_vars/ DIRECTORY                            │
  │     group_vars/webservers.yml                        │
  │     group_vars/all.yml                               │
  │                                                      │
  │  4. host_vars/ DIRECTORY                             │
  │     host_vars/web1.example.com.yml                   │
  │                                                      │
  └──────────────────────────────────────────────────────┘

  Precedence (low → high):
  all group → parent group → child group → host
```

---

## Inline Variables

```ini
# Inventory with inline variables
[webservers]
web1 ansible_host=192.168.1.10 http_port=80 max_clients=200
web2 ansible_host=192.168.1.11 http_port=8080 max_clients=100

[dbservers]
db1 ansible_host=192.168.2.10 db_port=5432 db_name=production
db2 ansible_host=192.168.2.11 db_port=5432 db_name=replica
```

> **Note**: Inline variables are hard to maintain. Use `host_vars/` files for anything beyond connection parameters.

---

## group_vars and host_vars Files

```
  RECOMMENDED DIRECTORY LAYOUT
  ══════════════════════════════════════════════════════════

  project/
  ├── ansible.cfg
  ├── inventory/
  │   └── hosts.ini
  ├── group_vars/
  │   ├── all.yml              ← Applies to ALL hosts
  │   ├── all/                 ← Or use a directory
  │   │   ├── vars.yml
  │   │   └── vault.yml        ← Encrypted secrets
  │   ├── webservers.yml       ← Webserver group vars
  │   ├── dbservers.yml        ← Database group vars
  │   └── production.yml       ← Production meta-group vars
  ├── host_vars/
  │   ├── web1.example.com.yml ← Host-specific vars
  │   └── db1.example.com.yml
  └── playbooks/
```

### group_vars/all.yml
```yaml
# group_vars/all.yml - Variables for every host
---
ansible_python_interpreter: /usr/bin/python3
ntp_server: time.example.com
dns_servers:
  - 8.8.8.8
  - 8.8.4.4
timezone: UTC
admin_email: ops@example.com
```

### group_vars/webservers.yml
```yaml
# group_vars/webservers.yml
---
ansible_user: deploy
http_port: 80
https_port: 443
max_connections: 1000
document_root: /var/www/html
ssl_enabled: true
nginx_worker_processes: auto
```

### host_vars/web1.example.com.yml
```yaml
# host_vars/web1.example.com.yml
---
ansible_host: 192.168.1.10
server_role: primary
max_connections: 2000    # Override group default
ssl_certificate: /etc/ssl/certs/web1.pem
custom_vhosts:
  - domain: app.example.com
    root: /var/www/app
  - domain: api.example.com
    root: /var/www/api
```

---

## Variable Types

```yaml
# String
hostname: "web-server-01"

# Number
http_port: 80
max_retries: 3

# Boolean
ssl_enabled: true
debug_mode: false

# List / Array
packages:
  - nginx
  - php-fpm
  - certbot

dns_servers:
  - 8.8.8.8
  - 8.8.4.4

# Dictionary / Map
database:
  host: db1.example.com
  port: 5432
  name: myapp
  user: appuser

# Complex nested structure
app_config:
  name: myapp
  version: "2.1.0"
  settings:
    debug: false
    log_level: info
  servers:
    - host: app1
      port: 8080
    - host: app2
      port: 8081
```

---

## Built-in Connection Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `ansible_host` | Host to connect to | inventory hostname |
| `ansible_port` | SSH port | 22 |
| `ansible_user` | SSH username | current user |
| `ansible_password` | SSH password | (use keys) |
| `ansible_ssh_private_key_file` | SSH private key path | ~/.ssh/id_rsa |
| `ansible_connection` | Connection type | ssh |
| `ansible_become` | Enable privilege escalation | false |
| `ansible_become_user` | User to escalate to | root |
| `ansible_become_method` | Escalation method | sudo |
| `ansible_python_interpreter` | Python path on target | auto |
| `ansible_shell_type` | Shell type | sh |

---

## Using Variables in Playbooks

```yaml
# site.yml
---
- name: Configure web servers
  hosts: webservers
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    # Use inventory variable
    - name: Configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      vars:
        port: "{{ http_port }}"       # From group_vars/webservers.yml
        workers: "{{ nginx_worker_processes }}"

    - name: Set timezone
      timezone:
        name: "{{ timezone }}"        # From group_vars/all.yml

    - name: Show host info
      debug:
        msg: "{{ inventory_hostname }} → {{ ansible_host }} ({{ server_role | default('worker') }})"
```

---

## Real-World Scenarios

### Scenario: Multi-tier Application
```yaml
# group_vars/all.yml
app_name: myapp
app_version: "3.2.1"

# group_vars/webservers.yml
nginx_upstream_port: 8080

# group_vars/dbservers.yml
postgresql_version: 15
postgresql_max_connections: 200

# host_vars/db-primary.example.com.yml
postgresql_role: primary
postgresql_max_connections: 500   # Override for primary

# host_vars/db-replica.example.com.yml
postgresql_role: replica
postgresql_primary_host: db-primary.example.com
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Variable not found | Check file naming matches group/host name exactly |
| Wrong value used | Check variable precedence (host > group > all) |
| YAML parse error | Validate YAML syntax, watch for indentation |
| Secrets in plain text | Use `ansible-vault` for sensitive variables |
| Variable from wrong group | Use `ansible-inventory --host <name>` to debug |

---

## Summary Table

| Location | Scope | Precedence |
|----------|-------|------------|
| `group_vars/all.yml` | All hosts | Lowest |
| `group_vars/<group>.yml` | Group members | Medium |
| `host_vars/<host>.yml` | Single host | Highest |
| Inline in inventory | Single host | Highest |
| `[group:vars]` section | Group members | Medium |

---

## Quick Revision Questions

1. **What is the precedence order for inventory variables?**
   > all group vars → parent group vars → child group vars → host vars (host wins).

2. **Where should sensitive variables be stored?**
   > In vault-encrypted files, typically in `group_vars/all/vault.yml` or similar.

3. **What is `group_vars/all.yml` used for?**
   > Variables that apply to every host across all groups (e.g., NTP server, DNS, timezone).

4. **How do you override a group variable for a specific host?**
   > Define the same variable in `host_vars/<hostname>.yml` — host vars take precedence over group vars.

5. **What command shows all variables assigned to a specific host?**
   > `ansible-inventory --host <hostname>` shows merged variables for that host.

6. **Can you use a directory instead of a file for group_vars?**
   > Yes. `group_vars/webservers/` directory with multiple YAML files is equivalent to `group_vars/webservers.yml`.

---

[← Previous: Dynamic Inventory](02-dynamic-inventory.md) | [Next: Groups and Group Variables →](04-groups-and-group-variables.md)
