# Chapter 3.4: Groups and Group Variables

[← Previous: Inventory Variables](03-inventory-variables.md) | [Next: Host Patterns →](05-host-patterns.md)

---

## Overview

Groups organize hosts into logical categories, enabling you to target specific sets of machines and apply shared variables. Groups can be nested to create hierarchies matching your infrastructure topology.

---

## Group Hierarchy

```
  GROUP HIERARCHY EXAMPLE
  ══════════════════════════════════════════════════════════

                        all
                         │
            ┌────────────┼────────────┐
            │            │            │
       production     staging      development
            │            │            │
     ┌──────┼──────┐    ...         ...
     │      │      │
  webservers│  dbservers
     │      │      │
  ┌──┤   ┌──┤   ┌──┤
  web1│   app1│  db1│
  web2│   app2│  db2│
  web3│      │     │
      │      │     │
  appservers─┘     │
                   │
  Each host inherits variables from ALL groups it belongs to
```

---

## Defining Groups

### INI Format
```ini
# === Direct Groups ===
[webservers]
web1.example.com
web2.example.com

[appservers]
app1.example.com
app2.example.com

[dbservers]
db1.example.com
db2.example.com

[loadbalancers]
lb1.example.com

# === Meta Groups (Children) ===
[frontend:children]
webservers
loadbalancers

[backend:children]
appservers
dbservers

[production:children]
frontend
backend

# === Group Variables ===
[webservers:vars]
http_port=80
ansible_user=webadmin

[dbservers:vars]
db_port=5432
ansible_user=dbadmin

[production:vars]
env=production
monitoring=true
```

### YAML Format
```yaml
all:
  children:
    production:
      vars:
        env: production
        monitoring: true
      children:
        frontend:
          children:
            webservers:
              vars:
                http_port: 80
              hosts:
                web1.example.com:
                web2.example.com:
            loadbalancers:
              hosts:
                lb1.example.com:
        backend:
          children:
            appservers:
              hosts:
                app1.example.com:
                app2.example.com:
            dbservers:
              vars:
                db_port: 5432
              hosts:
                db1.example.com:
                db2.example.com:
```

---

## Variable Inheritance

```
  VARIABLE INHERITANCE (Bottom-Up)
  ══════════════════════════════════════════════════════════

  web1.example.com receives variables from:

  ┌─────────────────────────────────────────┐
  │  all:vars           (lowest priority)   │
  │    ntp_server: time.example.com         │
  │                                         │
  │  production:vars                        │
  │    env: production                      │
  │    monitoring: true                     │
  │                                         │
  │  frontend:vars                          │
  │    proxy_protocol: true                 │
  │                                         │
  │  webservers:vars                        │
  │    http_port: 80                        │
  │    ansible_user: webadmin               │
  │                                         │
  │  host_vars/web1.example.com.yml         │
  │    ansible_host: 192.168.1.10           │
  │    max_connections: 2000   (highest)    │
  └─────────────────────────────────────────┘

  Final variable set for web1:
  {
    ntp_server: time.example.com,  ← from all
    env: production,               ← from production
    monitoring: true,              ← from production
    proxy_protocol: true,          ← from frontend
    http_port: 80,                 ← from webservers
    ansible_user: webadmin,        ← from webservers
    ansible_host: 192.168.1.10,    ← from host_vars
    max_connections: 2000          ← from host_vars
  }
```

---

## A Host in Multiple Groups

A single host can belong to many groups simultaneously:

```ini
[webservers]
server1.example.com

[production]
server1.example.com

[us_east]
server1.example.com

[monitored]
server1.example.com
```

```
  server1.example.com group memberships:
  ┌──────────┐ ┌──────────┐ ┌────────┐ ┌──────────┐ ┌───┐
  │webservers│ │production│ │us_east │ │monitored │ │all│
  └──────────┘ └──────────┘ └────────┘ └──────────┘ └───┘
       │            │           │           │          │
       └────────────┴───────────┴───────────┴──────────┘
                              │
                     Variables merged from
                     all groups (precedence
                     based on alphabetical
                     order for same-level)
```

---

## group_vars Directory Structure

```yaml
# group_vars/all.yml - Global defaults
---
ntp_server: time.example.com
dns_servers: [8.8.8.8, 8.8.4.4]
admin_email: ops@example.com

# group_vars/webservers.yml
---
http_port: 80
nginx_worker_processes: auto
ssl_enabled: true

# group_vars/dbservers.yml
---
postgresql_version: 15
backup_enabled: true
backup_schedule: "0 2 * * *"

# group_vars/production.yml
---
env: production
log_level: warn
monitoring_enabled: true

# group_vars/staging.yml
---
env: staging
log_level: debug
monitoring_enabled: false
```

---

## Real-World Scenarios

### Scenario: E-Commerce Platform

```ini
# Functional groups
[web_frontends]
fe[01:04].shop.com

[api_servers]
api[01:06].shop.com

[search_servers]
search[01:03].shop.com

[cache_servers]
redis[01:02].shop.com

[db_primary]
mysql-primary.shop.com

[db_replicas]
mysql-replica[01:03].shop.com

# Location groups
[datacenter_east:children]
dc_east_web
dc_east_db

[datacenter_west:children]
dc_west_web
dc_west_db

# Environment groups
[production:children]
web_frontends
api_servers
search_servers
cache_servers
db_primary
db_replicas

# Feature groups
[needs_ssl:children]
web_frontends
api_servers

[needs_backup:children]
db_primary
db_replicas
search_servers
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Variable from wrong group applied | Check group membership with `ansible-inventory --graph` |
| Children syntax error | In INI: `[parent:children]`, in YAML: `children:` key |
| Conflicting group variables | Higher specificity wins; same level = alphabetical |
| Host not in expected group | Verify spelling and inventory file loaded |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Group | Collection of hosts targeted together |
| Children groups | `[parent:children]` - groups of groups |
| `all` | Implicit group containing every host |
| `ungrouped` | Hosts not in any named group |
| Group vars | Variables shared by all group members |
| Variable inheritance | Variables flow from parent to child groups |
| Multiple membership | A host can belong to many groups |
| group_vars/ | Directory for group variable YAML files |

---

## Quick Revision Questions

1. **How do you create a group of groups in INI format?**
   > Use `[parent_group:children]` section listing child group names.

2. **If a host belongs to multiple groups with the same variable, which value wins?**
   > The more specific (child) group wins. For groups at the same level, it's processed alphabetically (last wins).

3. **Can a host belong to multiple groups?**
   > Yes. A host can be in any number of groups and inherits variables from all of them.

4. **What are the two implicit groups in every inventory?**
   > `all` (every host) and `ungrouped` (hosts not in any named group).

5. **How do group_vars work with meta groups?**
   > Meta group variables are inherited by all hosts in child groups but at lower precedence than child group variables.

6. **What command shows which groups a host belongs to?**
   > `ansible-inventory --graph` shows the full hierarchy, or `ansible-inventory --host <name>` shows merged variables.

---

[← Previous: Inventory Variables](03-inventory-variables.md) | [Next: Host Patterns →](05-host-patterns.md)
