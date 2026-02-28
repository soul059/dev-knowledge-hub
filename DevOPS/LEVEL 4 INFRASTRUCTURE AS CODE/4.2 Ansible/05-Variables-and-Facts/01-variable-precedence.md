# Chapter 5.1: Variable Precedence

[← Previous: Tags](../04-Playbooks/08-tags.md) | [Next: Facts and setup Module →](02-facts-and-setup-module.md)

---

## Overview

Ansible variables can be defined in **over 20 different locations**. When the same variable is defined in multiple places, Ansible uses a strict **precedence order** to decide which value wins. Understanding this hierarchy is critical for predictable playbook behavior.

---

## The Complete Precedence Order

```
  VARIABLE PRECEDENCE (Lowest → Highest)
  ══════════════════════════════════════════════════════════

  1.  command line values (e.g., -u, not -e)
  2.  role defaults (roles/x/defaults/main.yml)
  3.  inventory file or script group vars
  4.  inventory group_vars/all
  5.  playbook group_vars/all
  6.  inventory group_vars/*
  7.  playbook group_vars/*
  8.  inventory file or script host vars
  9.  inventory host_vars/*
  10. playbook host_vars/*
  11. host facts / cached set_facts
  12. play vars
  13. play vars_prompt
  14. play vars_files
  15. role vars (roles/x/vars/main.yml)
  16. block vars (only for tasks in block)
  17. task vars (only for the task)
  18. include_vars
  19. set_facts / registered vars
  20. role (and include_role) params
  21. include params
  22. extra vars (-e)  ◄── ALWAYS WINS
```

---

## Simplified Mental Model

```
  SIMPLIFIED PRECEDENCE (Remember This!)
  ══════════════════════════════════════════════════════════

  LOWEST ───────────────────────────────────────── HIGHEST

  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │  Role    │  │Inventory │  │   Play   │  │  Extra   │
  │ Defaults │→ │  Vars    │→ │   Vars   │→ │   Vars   │
  │          │  │          │  │          │  │   (-e)   │
  └──────────┘  └──────────┘  └──────────┘  └──────────┘
      ▲              ▲             ▲             ▲
   Easiest to     Group &       vars:,        Cannot be
   override      host vars    set_fact,      overridden
                              include_vars
```

### Three Levels to Remember

| Level | Sources | Use Case |
|-------|---------|----------|
| **Defaults** (low) | `roles/x/defaults/`, `group_vars/all` | Sensible defaults users can override |
| **Standard** (mid) | `group_vars/*`, `host_vars/*`, play `vars:` | Normal configuration |
| **Override** (high) | `set_fact`, `include_vars`, `-e` | Force values, runtime overrides |

---

## Precedence Demonstrated

```yaml
# roles/webserver/defaults/main.yml
http_port: 80                      # Precedence: 2 (lowest)

# inventory/group_vars/webservers.yml
http_port: 8080                    # Precedence: 6

# inventory/host_vars/web01.yml
http_port: 8081                    # Precedence: 9

# playbook.yml
- hosts: webservers
  vars:
    http_port: 9090                # Precedence: 12
  tasks:
    - name: Show port
      debug:
        msg: "Port: {{ http_port }}"  # 9090 for all hosts

    - name: Override with set_fact
      set_fact:
        http_port: 3000            # Precedence: 19
    - debug:
        msg: "Port: {{ http_port }}"  # Now 3000
```

```bash
# Extra vars ALWAYS win
ansible-playbook site.yml -e "http_port=443"
# http_port = 443 everywhere, overrides everything
```

---

## Role defaults vs Role vars

```
  ROLE DEFAULTS vs ROLE VARS
  ══════════════════════════════════════════════════════════

  roles/nginx/
  ├── defaults/
  │   └── main.yml      ◄── Precedence 2 (VERY LOW)
  │       http_port: 80       "Here's a reasonable default"
  │                           "Feel free to override me"
  │
  └── vars/
      └── main.yml      ◄── Precedence 15 (HIGH)
          nginx_user: www-data  "This should NOT be changed"
                                "I know better than you"
```

```yaml
# roles/nginx/defaults/main.yml  — OVERRIDABLE
# Use for: values users commonly change
http_port: 80
https_port: 443
worker_processes: auto
worker_connections: 1024

# roles/nginx/vars/main.yml — HARD TO OVERRIDE
# Use for: internal role constants
nginx_conf_path: /etc/nginx/nginx.conf
nginx_pid_file: /run/nginx.pid
nginx_user: www-data
```

---

## group_vars and host_vars Precedence

```
  GROUP VARS ORDERING
  ══════════════════════════════════════════════════════════

  Host: web01
  Groups: [all, webservers, production]

  Variable: log_level

  group_vars/all.yml:          log_level: warn    (Precedence 4-5)
  group_vars/webservers.yml:   log_level: info    (Precedence 6-7)
  group_vars/production.yml:   log_level: error   (Precedence 6-7)
  host_vars/web01.yml:         log_level: debug   (Precedence 9-10)

  Result: log_level = debug  (host_vars wins)
```

### When Multiple Groups Have Same Precedence

Groups at the same level are sorted **alphabetically**, and the **last one wins**:

```ini
# inventory
[webservers]
web01

[production]
web01

[databases]
web01
```

```yaml
# group_vars/databases.yml
app_env: db_server

# group_vars/production.yml
app_env: prod

# group_vars/webservers.yml
app_env: web_server

# For web01: app_env = "web_server"
# (alphabetical: databases < production < webservers)
```

> **Best Practice:** Avoid relying on alphabetical ordering. Use `ansible_group_priority` or explicit host_vars.

```yaml
# Explicit group priority in inventory
[production:vars]
ansible_group_priority=10    # Higher number = wins over other groups
```

---

## include_vars Precedence

`include_vars` has **very high precedence** (18) — use with care:

```yaml
tasks:
  - name: Load environment config
    include_vars:
      file: "vars/{{ env }}.yml"
    # These vars override play vars, group_vars, host_vars

  - name: Load OS-specific vars
    include_vars:
      file: "vars/{{ ansible_os_family }}.yml"
    # Also high precedence
```

---

## Practical Precedence Examples

### Example 1: Environment-specific Configuration

```
  project/
  ├── group_vars/
  │   ├── all.yml               ◄── Base defaults
  │   ├── webservers.yml        ◄── Web-specific
  │   ├── staging.yml           ◄── Staging overrides
  │   └── production.yml        ◄── Production overrides
  ├── host_vars/
  │   └── web01.yml             ◄── Host-specific
  └── site.yml
```

```yaml
# group_vars/all.yml
log_level: info
debug_mode: false
max_connections: 100

# group_vars/webservers.yml
max_connections: 500
worker_processes: 4

# group_vars/staging.yml
debug_mode: true
log_level: debug

# group_vars/production.yml
debug_mode: false
log_level: warn
max_connections: 1000

# host_vars/web01.yml
worker_processes: 8    # This host has more CPUs
```

### Example 2: Debugging Precedence Issues

```yaml
tasks:
  - name: Show where http_port comes from
    debug:
      msg: |
        http_port = {{ http_port }}
        Defined in play vars: {{ vars.get('http_port', 'NOT SET') }}
        Group vars: check group_vars/ files
        Host vars: check host_vars/{{ inventory_hostname }}.yml

  # Nuclear option: see ALL variables
  - name: Dump all variables
    debug:
      var: hostvars[inventory_hostname]
    tags: [never, debug]
```

```bash
# Check variable value and source
ansible -i inventory webservers -m debug -a "var=http_port"

# Check all variables for a host
ansible -i inventory web01 -m debug -a "var=hostvars[inventory_hostname]"
```

---

## Real-World Scenario: Multi-Environment Deployment

```
  project/
  ├── inventories/
  │   ├── staging/
  │   │   ├── hosts.yml
  │   │   ├── group_vars/
  │   │   │   ├── all.yml        ◄── staging defaults
  │   │   │   └── webservers.yml
  │   │   └── host_vars/
  │   │       └── staging-web01.yml
  │   └── production/
  │       ├── hosts.yml
  │       ├── group_vars/
  │       │   ├── all.yml        ◄── production defaults
  │       │   └── webservers.yml
  │       └── host_vars/
  │           ├── prod-web01.yml
  │           └── prod-web02.yml
  ├── roles/
  │   └── webapp/
  │       ├── defaults/main.yml  ◄── Lowest: App defaults
  │       └── vars/main.yml     ◄── High: Internal constants
  └── deploy.yml
```

```yaml
# roles/webapp/defaults/main.yml
app_port: 8080
app_workers: 2
app_log_level: info
app_debug: false

# inventories/staging/group_vars/all.yml
env: staging
app_debug: true
app_log_level: debug

# inventories/production/group_vars/all.yml
env: production
app_debug: false
app_log_level: warn
app_workers: 8

# inventories/production/host_vars/prod-web01.yml
app_workers: 16    # Larger instance
```

```bash
# Deploy to staging (inherits staging vars)
ansible-playbook -i inventories/staging deploy.yml

# Deploy to production (inherits production vars)
ansible-playbook -i inventories/production deploy.yml

# Emergency override
ansible-playbook -i inventories/production deploy.yml -e "app_debug=true"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Variable has unexpected value | Use `debug` module to print it; check all group_vars/host_vars files |
| `-e` not overriding | Ensure syntax is correct: `-e "var=value"` or `-e @file.yml` |
| Role defaults not being overridden | Check that your override is at higher precedence (group_vars or play vars) |
| Group vars conflict between groups | Set `ansible_group_priority` or use host_vars |
| `include_vars` overriding play vars | Expected; `include_vars` has precedence 18 (very high) |
| Can't figure out where variable is set | Run with `-v` or dump `hostvars[inventory_hostname]` |

---

## Summary Table

| Precedence Level | Source | Typical Use |
|------------------|--------|-------------|
| 2 (Lowest) | Role defaults | Overridable sane defaults |
| 4-5 | `group_vars/all` | Global base configuration |
| 6-7 | `group_vars/*` | Group-specific config |
| 9-10 | `host_vars/*` | Host-specific overrides |
| 12 | Play `vars:` | Playbook-level values |
| 15 | Role vars | Internal role constants |
| 18 | `include_vars` | Dynamic runtime loading |
| 19 | `set_fact` / `register` | Runtime computed values |
| 22 (Highest) | Extra vars (`-e`) | CLI overrides; always wins |

---

## Quick Revision Questions

1. **Which variable source has the highest precedence?**
   > Extra vars (`-e` on CLI) — precedence 22. They always win and cannot be overridden.

2. **What is the difference between role defaults and role vars?**
   > Role defaults (precedence 2) are meant to be easily overridden. Role vars (precedence 15) are internal constants that are hard to override.

3. **If the same variable is in `group_vars/webservers.yml` and `host_vars/web01.yml`, which wins?**
   > `host_vars` wins — it has higher precedence (9-10) than group_vars (6-7).

4. **When two groups define the same variable at the same level, which wins?**
   > The group that comes last alphabetically wins. Use `ansible_group_priority` to control this explicitly.

5. **Where should you define values you want users/teams to easily override?**
   > In `roles/*/defaults/main.yml` — it has the lowest precedence, making it easy to override from any other level.

6. **How can you debug which value a variable is getting?**
   > Use `debug: var=my_variable` or dump all vars with `debug: var=hostvars[inventory_hostname]`.

---

[← Previous: Tags](../04-Playbooks/08-tags.md) | [Next: Facts and setup Module →](02-facts-and-setup-module.md)
