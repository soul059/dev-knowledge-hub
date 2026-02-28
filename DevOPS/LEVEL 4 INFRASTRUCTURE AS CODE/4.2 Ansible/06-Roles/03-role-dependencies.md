# Chapter 6.3: Role Dependencies

[← Previous: Creating Roles](02-creating-roles.md) | [Next: Ansible Galaxy →](04-ansible-galaxy.md)

---

## Overview

**Role dependencies** let you declare that a role requires other roles to run first. When Ansible encounters a role with dependencies, it automatically applies the dependent roles before the main role, keeping your roles modular and avoiding duplication.

---

## Defining Dependencies

Dependencies are declared in `meta/main.yml`:

```yaml
# roles/webserver/meta/main.yml
---
dependencies:
  - common
  - role: firewall
    vars:
      firewall_allowed_ports:
        - 80
        - 443
  - role: ssl_certificates
    when: enable_ssl | bool
```

---

## Dependency Resolution Flow

```
  PLAYBOOK SPECIFIES: webserver
  ══════════════════════════════════════════════════════════

  Ansible reads webserver/meta/main.yml
    └── dependencies:
         ├── common
         │   └── (common has no deps — runs first)
         ├── firewall
         │   └── (firewall has no deps — runs second)
         └── ssl_certificates
             └── (runs third, if condition met)

  Execution order:
  ┌──────────────────┐
  │ 1. common        │
  │ 2. firewall      │
  │ 3. ssl_certs     │  ◄── Only if enable_ssl
  │ 4. webserver     │  ◄── Main role runs last
  └──────────────────┘
```

---

## Nested Dependencies

```yaml
# roles/webserver/meta/main.yml
dependencies:
  - common
  - firewall

# roles/firewall/meta/main.yml
dependencies:
  - common          # common is a dependency of both

# roles/database/meta/main.yml
dependencies:
  - common
  - firewall
```

```
  DEDUPLICATION (default behavior)
  ══════════════════════════════════════════════════════════

  Playbook:
    roles:
      - webserver       (deps: common → firewall)
      - database        (deps: common → firewall)

  Actual execution:
    1. common           ◄── Runs once
    2. firewall         ◄── Runs once
    3. webserver
    4. database         ◄── common & firewall NOT re-run
```

---

## allow_duplicates

Override deduplication when a dependency must run multiple times with different parameters:

```yaml
# roles/firewall/meta/main.yml
---
allow_duplicates: true
```

```
  WITH allow_duplicates: true on firewall
  ══════════════════════════════════════════════════════════

  Playbook:
    roles:
      - role: webserver    (deps: firewall port=80)
      - role: database     (deps: firewall port=5432)

  Actual execution:
    1. firewall (port=80)     ◄── Runs for webserver
    2. webserver
    3. firewall (port=5432)   ◄── Runs again for database
    4. database
```

---

## Dependency Syntax Variations

```yaml
dependencies:
  # Simple name
  - common

  # With variables
  - role: firewall
    vars:
      port: 80

  # With conditions
  - role: ssl_certificates
    when: enable_ssl | bool

  # With tags
  - role: monitoring
    tags: monitoring

  # From a collection
  - role: geerlingguy.docker

  # From Galaxy with version
  - src: geerlingguy.nginx
    version: "3.1.0"
    name: nginx
```

---

## Conditional Dependencies

```yaml
# roles/app/meta/main.yml
---
dependencies:
  - role: database
    when: app_needs_database | default(true)
  - role: cache
    when: app_needs_cache | default(false)
  - role: queue
    when: app_needs_queue | default(false)
```

```yaml
# Using the role with conditions
- hosts: app_servers
  roles:
    - role: app
      vars:
        app_needs_database: true
        app_needs_cache: true
        app_needs_queue: false
```

---

## Dependency vs import_role/include_role

| Approach | When to Use |
|----------|-------------|
| `dependencies` in `meta/main.yml` | Role always needs another role first |
| `import_role` in tasks | Conditional or task-level role inclusion |
| `include_role` in tasks | Dynamic, runtime-determined role inclusion |

```yaml
# meta/main.yml approach — always runs
dependencies:
  - common

# tasks/main.yml approach — more control
tasks:
  - name: Include monitoring if needed
    include_role:
      name: monitoring
    when: install_monitoring | bool
```

---

## Real-World Scenario: Application Stack Dependencies

```yaml
# roles/django_app/meta/main.yml
---
dependencies:
  - role: common
  - role: python
    vars:
      python_version: "3.11"
  - role: nginx
    vars:
      nginx_proxy_pass: "http://127.0.0.1:8000"
  - role: postgresql_client
  - role: redis_client
    when: use_redis_cache | default(false)
```

```
  DEPENDENCY TREE
  ══════════════════════════════════════════════════════════

                   django_app
                  /    |     \      \
              common  python  nginx  postgresql_client
                        |       |
                     (v3.11)  (proxy)

  Execution: common → python → nginx → pg_client → django_app
```

---

## Circular Dependency Protection

Ansible **does not** allow circular dependencies:

```yaml
# roles/a/meta/main.yml
dependencies:
  - b

# roles/b/meta/main.yml
dependencies:
  - a     # ERROR: circular dependency!
```

```
  ERROR! A recursive loop was detected with role 'a'
```

**Solution**: Extract shared logic into a third role:

```yaml
# roles/shared/meta/main.yml
dependencies: []

# roles/a/meta/main.yml
dependencies:
  - shared

# roles/b/meta/main.yml
dependencies:
  - shared
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Circular dependency error | Extract common tasks into a shared role |
| Dependency runs but vars not applied | Use `vars:` under the role entry in dependencies |
| Dependency skipped unexpectedly | Check `when` conditions; missing vars default to undefined |
| Dependency runs multiple times | That's default dedup working — use `allow_duplicates: true` if needed |
| Galaxy dependency not found | Run `ansible-galaxy install -r requirements.yml` first |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| `dependencies:` | List in `meta/main.yml` |
| Execution order | Dependencies run before the main role |
| Deduplication | Same role runs only once by default |
| `allow_duplicates` | Override to run role multiple times |
| Conditional deps | Use `when:` on dependency entries |
| Circular deps | Not allowed — causes immediate error |
| Nested deps | Resolved recursively, depth-first |

---

## Quick Revision Questions

1. **Where are role dependencies declared?**
   > In the role's `meta/main.yml` file under the `dependencies:` key.

2. **In what order do dependencies execute relative to the main role?**
   > Dependencies execute **before** the main role's tasks.

3. **What happens if two roles both depend on the same role?**
   > By default, the shared dependency runs only once (deduplication). Use `allow_duplicates: true` to override.

4. **Can you add conditions to role dependencies?**
   > Yes, add `when:` to the dependency entry: `- role: name` / `when: condition`.

5. **What error occurs with circular role dependencies?**
   > Ansible raises `ERROR! A recursive loop was detected` and stops execution.

---

[← Previous: Creating Roles](02-creating-roles.md) | [Next: Ansible Galaxy →](04-ansible-galaxy.md)
