# Chapter 5.3: Magic Variables

[← Previous: Facts and setup Module](02-facts-and-setup-module.md) | [Next: Vault (Encrypting Secrets) →](04-vault.md)

---

## Overview

**Magic variables** (also called special variables) are automatically defined by Ansible. They provide information about the current play, host, groups, inventory, and execution context. Unlike facts, they are not gathered from managed hosts — they come from Ansible's internal state.

---

## Essential Magic Variables

```
  MAGIC VARIABLES CATEGORIES
  ══════════════════════════════════════════════════════════

  ┌──────────────────────────────────────┐
  │           HOST IDENTITY              │
  │  inventory_hostname                  │
  │  inventory_hostname_short            │
  │  ansible_host                        │
  │  ansible_connection                  │
  └──────────────────────────────────────┘
  ┌──────────────────────────────────────┐
  │           GROUP INFO                 │
  │  group_names                         │
  │  groups                              │
  │  groups['all']                       │
  └──────────────────────────────────────┘
  ┌──────────────────────────────────────┐
  │         HOST VARIABLES               │
  │  hostvars                            │
  │  hostvars['web01']                   │
  └──────────────────────────────────────┘
  ┌──────────────────────────────────────┐
  │          PLAY CONTEXT                │
  │  ansible_play_hosts                  │
  │  ansible_play_batch                  │
  │  play_hosts (deprecated)             │
  │  ansible_play_name                   │
  └──────────────────────────────────────┘
  ┌──────────────────────────────────────┐
  │          PATHS & META                │
  │  playbook_dir                        │
  │  role_path                           │
  │  ansible_check_mode                  │
  │  ansible_version                     │
  └──────────────────────────────────────┘
```

---

## Host Identity Variables

```yaml
tasks:
  - name: Show host identity
    debug:
      msg: |
        inventory_hostname: {{ inventory_hostname }}
        inventory_hostname_short: {{ inventory_hostname_short }}
        ansible_host: {{ ansible_host | default('not set') }}
```

| Variable | Description | Example |
|----------|-------------|---------|
| `inventory_hostname` | Name as defined in inventory | `web01.example.com` |
| `inventory_hostname_short` | First part of hostname | `web01` |
| `ansible_host` | Actual address to connect to | `10.0.1.10` |

```ini
# In inventory
web01.example.com ansible_host=10.0.1.10

# inventory_hostname = "web01.example.com"
# inventory_hostname_short = "web01"
# ansible_host = "10.0.1.10"
```

---

## Group Variables

### group_names

List of groups the **current host** belongs to (excludes `all`):

```yaml
- debug:
    msg: "This host is in groups: {{ group_names }}"
    # e.g., ["webservers", "production"]

- name: Apply web config
  template:
    src: web.conf.j2
    dest: /etc/app/web.conf
  when: "'webservers' in group_names"
```

### groups

Dictionary of **all groups** and their member hostnames:

```yaml
- debug:
    msg: |
      All hosts: {{ groups['all'] }}
      Web servers: {{ groups['webservers'] }}
      DB servers: {{ groups.get('dbservers', []) }}

# Useful patterns
- name: Configure app with all DB hosts
  template:
    src: db-config.j2
    dest: /etc/app/databases.yml
  vars:
    db_hosts: "{{ groups['dbservers'] }}"

- name: Get first web server
  debug:
    msg: "Primary web: {{ groups['webservers'][0] }}"
```

---

## hostvars

Access **any host's** variables and facts:

```yaml
tasks:
  # Access another host's fact
  - name: Get DB server IP
    debug:
      msg: "DB IP: {{ hostvars['db01']['ansible_default_ipv4']['address'] }}"

  # Iterate over group and access each host's vars
  - name: Build server list
    debug:
      msg: "{{ item }} → {{ hostvars[item]['ansible_default_ipv4']['address'] }}"
    loop: "{{ groups['webservers'] }}"

  # Access another host's custom variable
  - name: Get DB port from host vars
    debug:
      msg: "DB port: {{ hostvars['db01']['db_port'] | default(5432) }}"
```

```
  HOSTVARS STRUCTURE
  ══════════════════════════════════════════════════════════

  hostvars
  ├── web01
  │   ├── ansible_hostname: "web01"
  │   ├── ansible_default_ipv4:
  │   │   └── address: "10.0.1.10"
  │   ├── ansible_os_family: "Debian"
  │   └── http_port: 80                 ◄── Custom variable
  ├── web02
  │   ├── ansible_hostname: "web02"
  │   └── ...
  └── db01
      ├── ansible_hostname: "db01"
      ├── ansible_default_ipv4:
      │   └── address: "10.0.2.10"
      └── db_port: 5432
```

> **Note:** A host's facts are only available in `hostvars` if that host has had facts gathered (was part of a play with `gather_facts: yes`).

---

## Play Context Variables

```yaml
tasks:
  - name: Show play context
    debug:
      msg: |
        Play name: {{ ansible_play_name }}
        All play hosts: {{ ansible_play_hosts }}
        Current batch: {{ ansible_play_batch }}
        Remaining hosts: {{ ansible_play_hosts_all }}
```

| Variable | Description |
|----------|-------------|
| `ansible_play_hosts` | Active hosts in current play (not failed) |
| `ansible_play_hosts_all` | All hosts targeted (including failed) |
| `ansible_play_batch` | Current batch when using `serial` |
| `ansible_play_name` | Name of the current play |

```yaml
# Useful for run_once patterns
- name: Send notification with all hosts
  uri:
    url: "{{ webhook }}"
    body:
      hosts: "{{ ansible_play_hosts | join(', ') }}"
  delegate_to: localhost
  run_once: true
```

---

## Path Variables

```yaml
tasks:
  - debug:
      msg: |
        Playbook directory: {{ playbook_dir }}
        Role path: {{ role_path | default('N/A') }}
        Inventory directory: {{ inventory_dir }}
        Inventory file: {{ inventory_file }}
```

| Variable | Description | Example |
|----------|-------------|---------|
| `playbook_dir` | Playbook file directory | `/opt/ansible/playbooks` |
| `role_path` | Current role's directory | `/opt/ansible/roles/nginx` |
| `inventory_dir` | Inventory file directory | `/opt/ansible/inventory` |
| `inventory_file` | Inventory filename | `/opt/ansible/inventory/hosts` |

---

## Execution State Variables

```yaml
tasks:
  - name: Show execution context
    debug:
      msg: |
        Check mode: {{ ansible_check_mode }}
        Diff mode: {{ ansible_diff_mode }}
        Verbosity: {{ ansible_verbosity }}
        Ansible version: {{ ansible_version.full }}
        Python: {{ ansible_playbook_python }}

  # Conditionally handle check mode
  - name: Do something only in real run
    command: /opt/deploy.sh
    when: not ansible_check_mode

  # Different behavior in check mode
  - name: Simulate deployment
    debug:
      msg: "Would deploy version {{ version }}"
    when: ansible_check_mode
```

| Variable | Description |
|----------|-------------|
| `ansible_check_mode` | `true` if running with `--check` |
| `ansible_diff_mode` | `true` if running with `--diff` |
| `ansible_verbosity` | Verbosity level (0-4) |
| `ansible_version.full` | Ansible version string |
| `ansible_playbook_python` | Python interpreter on control node |

---

## Task and Role Variables

```yaml
# Available inside roles
- debug:
    msg: |
      Role name: {{ role_name }}
      Role path: {{ role_path }}

# Available in tasks
- debug:
    msg: |
      Task name: {{ ansible_task_name | default('unknown') }}

# In rescue blocks
rescue:
  - debug:
      msg: |
        Failed task: {{ ansible_failed_task.name }}
        Failure result: {{ ansible_failed_result }}
```

---

## Real-World Scenario: Dynamic Configuration with Magic Variables

```yaml
---
- name: Generate HAProxy configuration
  hosts: loadbalancers
  become: yes
  tasks:
    - name: Deploy HAProxy config
      template:
        src: haproxy.cfg.j2
        dest: /etc/haproxy/haproxy.cfg
      notify: Reload HAProxy

  handlers:
    - name: Reload HAProxy
      service:
        name: haproxy
        state: reloaded
```

```jinja2
{# haproxy.cfg.j2 — Uses magic variables #}
# Managed by Ansible from {{ playbook_dir }}
# Generated on {{ ansible_date_time.iso8601 }}

global
    log stdout format raw local0

defaults
    mode http
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    option httpchk GET /health
{% for host in groups['webservers'] %}
    server {{ hostvars[host]['inventory_hostname_short'] }} {{ hostvars[host]['ansible_default_ipv4']['address'] }}:{{ hostvars[host].get('http_port', 8080) }} check
{% endfor %}

# Monitoring
listen stats
    bind *:9000
    stats enable
    stats uri /stats
```

This template dynamically builds the HAProxy config using:
- `groups['webservers']` — to iterate over all web servers
- `hostvars[host]` — to get each server's IP and port
- `inventory_hostname_short` — for the server label

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `hostvars['host']` has no facts | Ensure the host is included in a play with `gather_facts: yes` |
| `inventory_hostname` vs `ansible_hostname` | `inventory_hostname` is from inventory; `ansible_hostname` is the actual hostname from the host |
| `groups` is empty | Check inventory is loaded correctly; verify with `--list-hosts` |
| Magic variable undefined | Some variables are only available in specific contexts (e.g., `role_path` only in roles) |
| Facts from previous play unavailable | Facts persist; check the host was in both plays |

---

## Summary Table

| Variable | Description |
|----------|-------------|
| `inventory_hostname` | Host name from inventory |
| `group_names` | Groups current host belongs to |
| `groups` | All groups with member lists |
| `hostvars` | All hosts' variables and facts |
| `ansible_play_hosts` | Active hosts in play |
| `ansible_play_batch` | Current batch (with serial) |
| `playbook_dir` | Directory containing playbook |
| `role_path` | Current role directory |
| `ansible_check_mode` | True if `--check` |
| `ansible_version` | Ansible version info |

---

## Quick Revision Questions

1. **What is the difference between `inventory_hostname` and `ansible_hostname`?**
   > `inventory_hostname` is the name in the inventory file. `ansible_hostname` is the actual hostname reported by the remote host's OS.

2. **How do you access another host's variables?**
   > Use `hostvars['hostname']['variable_name']`. The host must have been part of a play with fact gathering.

3. **What does `group_names` contain?**
   > A list of all groups the current host belongs to, excluding the implicit `all` group.

4. **How can you check if a playbook is running in check mode?**
   > Test `ansible_check_mode` — it's `true` when running with `--check`.

5. **How do you dynamically build a config referencing all hosts in a group?**
   > Use `groups['groupname']` to get the list, then `hostvars[host]` in a Jinja2 loop to access each host's facts.

---

[← Previous: Facts and setup Module](02-facts-and-setup-module.md) | [Next: Vault (Encrypting Secrets) →](04-vault.md)
