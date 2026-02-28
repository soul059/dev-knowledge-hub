# Chapter 6.1: Role Structure and Concepts

[← Previous: Fact Caching](../05-Variables-and-Facts/07-fact-caching.md) | [Next: Creating Roles →](02-creating-roles.md)

---

## Overview

**Roles** are the primary mechanism for organising and reusing Ansible code. A role is a self-contained directory structure that bundles tasks, handlers, variables, templates, files, and metadata into a single distributable unit.

---

## Why Roles?

```
  WITHOUT ROLES                          WITH ROLES
  ═══════════════════                    ═══════════════════
  site.yml                               site.yml
    200-line playbook                      - role: webserver
    • tasks mixed together                 - role: database
    • variables scattered                  - role: monitoring
    • templates inline
    • hard to share or reuse            Each role: self-contained,
                                        testable, shareable
```

---

## Standard Role Directory Structure

```
  my_role/
  ├── defaults/
  │   └── main.yml          ◄── Default variables (lowest precedence)
  ├── vars/
  │   └── main.yml          ◄── Role variables (higher precedence)
  ├── tasks/
  │   └── main.yml          ◄── Task list (entry point)
  ├── handlers/
  │   └── main.yml          ◄── Handlers
  ├── templates/
  │   └── config.j2         ◄── Jinja2 templates
  ├── files/
  │   └── app.conf          ◄── Static files
  ├── meta/
  │   └── main.yml          ◄── Role metadata and dependencies
  ├── library/               ◄── Custom modules (optional)
  ├── module_utils/          ◄── Module utilities (optional)
  ├── lookup_plugins/        ◄── Custom lookup plugins (optional)
  ├── tests/                 ◄── Test playbooks (optional)
  │   ├── inventory
  │   └── test.yml
  └── README.md              ◄── Documentation
```

---

## Directory Purposes

| Directory | Purpose | Auto-loaded? |
|-----------|---------|:------------:|
| `defaults/` | Default variables — easily overridden | Yes |
| `vars/` | Role variables — harder to override | Yes |
| `tasks/` | Main task list executed when role runs | Yes |
| `handlers/` | Handlers triggered by `notify` | Yes |
| `templates/` | Jinja2 templates (used by `template` module) | Referenced |
| `files/` | Static files (used by `copy`, `script` modules) | Referenced |
| `meta/` | Role dependencies and metadata | Yes |
| `library/` | Custom modules available within the role | Yes |
| `tests/` | Test playbooks for the role | Manual |

---

## How Ansible Finds Roles

```
  ROLE SEARCH ORDER
  ══════════════════════════════════════════════════════════

  1. roles/ directory adjacent to playbook
  2. Paths in roles_path (ansible.cfg)
  3. ~/.ansible/roles
  4. /usr/share/ansible/roles
  5. /etc/ansible/roles
  6. Collections path (ansible_collections/)
```

```ini
# ansible.cfg
[defaults]
roles_path = ./roles:/opt/ansible/roles:~/.ansible/roles
```

---

## Using Roles in Playbooks

### Classic Syntax

```yaml
---
- name: Configure web servers
  hosts: webservers
  roles:
    - webserver            # Simple role name
    - database             # Another role
```

### With Parameters

```yaml
---
- name: Configure web servers
  hosts: webservers
  roles:
    - role: webserver
      vars:
        http_port: 8080
        server_name: example.com
    - role: database
      when: db_enabled | bool
      tags: database
```

### include_role / import_role (Task-Level)

```yaml
tasks:
  - name: Apply webserver role
    import_role:
      name: webserver      # Static — parsed at playbook load
    vars:
      http_port: 8080

  - name: Conditionally apply monitoring
    include_role:
      name: monitoring     # Dynamic — evaluated at runtime
    when: monitoring_enabled
```

---

## import_role vs include_role

```
  import_role (Static)                include_role (Dynamic)
  ═══════════════════════             ═══════════════════════
  • Parsed at playbook load           • Evaluated at runtime
  • Tags/when applied to              • Tags/when apply to
    every task inside role              the include itself
  • Cannot loop                       • Can use loops
  • Shows all tasks in                • Tasks shown when
    --list-tasks                        executed
  • Fails early on errors             • Fails when reached
```

---

## Role Execution Order

```
  PLAYBOOK EXECUTION WITH ROLES
  ══════════════════════════════════════════════════════════

  1. pre_tasks           ◄── Before everything
  2. handlers (from pre_tasks)
  3. roles               ◄── All roles execute here
  4. tasks               ◄── Regular tasks section
  5. handlers (from roles + tasks)
  6. post_tasks           ◄── After everything
  7. handlers (from post_tasks)
```

```yaml
---
- name: Full execution order
  hosts: webservers
  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

  roles:
    - common
    - webserver

  tasks:
    - name: Verify service
      uri:
        url: "http://localhost"
        status_code: 200

  post_tasks:
    - name: Send notification
      slack:
        msg: "Deployment complete"
```

---

## Role Deduplication

By default, Ansible runs a role **only once** per play, even if listed multiple times:

```yaml
roles:
  - role: firewall
    vars:
      port: 80
  - role: firewall       # SKIPPED — already applied
    vars:
      port: 443
```

**Override** with `allow_duplicates: true` in `meta/main.yml`:

```yaml
# roles/firewall/meta/main.yml
---
allow_duplicates: true
```

Now both invocations run with their respective variables.

---

## Real-World Scenario: Project Using Multiple Roles

```
  project/
  ├── ansible.cfg
  ├── inventory/
  │   ├── production
  │   └── staging
  ├── group_vars/
  │   ├── all.yml
  │   ├── webservers.yml
  │   └── dbservers.yml
  ├── site.yml
  ├── webservers.yml
  ├── dbservers.yml
  └── roles/
      ├── common/            ◄── Base config for all hosts
      ├── webserver/         ◄── Nginx + app setup
      ├── database/          ◄── PostgreSQL setup
      ├── monitoring/        ◄── Prometheus + node_exporter
      └── security/          ◄── Firewall + hardening
```

```yaml
# site.yml
---
- import_playbook: webservers.yml
- import_playbook: dbservers.yml

# webservers.yml
---
- name: Configure web servers
  hosts: webservers
  roles:
    - common
    - security
    - webserver
    - monitoring

# dbservers.yml
---
- name: Configure database servers
  hosts: dbservers
  roles:
    - common
    - security
    - database
    - monitoring
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `ERROR! the role 'x' was not found` | Check `roles_path` in ansible.cfg, verify directory name matches |
| Role runs but tasks skipped | Ensure `tasks/main.yml` exists and isn't empty |
| Role runs only once with different vars | Set `allow_duplicates: true` in `meta/main.yml` |
| Variables not picked up | Check correct directory: `defaults/` vs `vars/` |
| Templates not found | Place them inside the role's `templates/` directory |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| `defaults/main.yml` | Low-precedence default variables |
| `vars/main.yml` | High-precedence role variables |
| `tasks/main.yml` | Entry point for role tasks |
| `meta/main.yml` | Dependencies and metadata |
| `import_role` | Static inclusion at parse time |
| `include_role` | Dynamic inclusion at runtime |
| `allow_duplicates` | Let role run multiple times |
| Role search path | Adjacent `roles/`, then `roles_path`, then defaults |

---

## Quick Revision Questions

1. **What is the entry-point file a role executes first?**
   > `tasks/main.yml` — Ansible looks for this file when a role is applied.

2. **What is the difference between `defaults/` and `vars/` in a role?**
   > `defaults/` has the lowest variable precedence and is intended to be overridden. `vars/` has higher precedence and is harder to override.

3. **In what order do `pre_tasks`, `roles`, `tasks`, and `post_tasks` execute?**
   > `pre_tasks` → handlers → `roles` → `tasks` → handlers → `post_tasks` → handlers.

4. **How do you allow a role to run multiple times in the same play?**
   > Set `allow_duplicates: true` in the role's `meta/main.yml`.

5. **What is the difference between `import_role` and `include_role`?**
   > `import_role` is static (parsed at load time), `include_role` is dynamic (evaluated at runtime) and supports loops.

---

[← Previous: Fact Caching](../05-Variables-and-Facts/07-fact-caching.md) | [Next: Creating Roles →](02-creating-roles.md)
