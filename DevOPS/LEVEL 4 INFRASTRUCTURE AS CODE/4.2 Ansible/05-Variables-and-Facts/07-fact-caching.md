# Chapter 5.7: Fact Caching

[← Previous: Filters](06-filters.md) | [Next: Role Structure and Concepts →](../06-Roles/01-role-structure.md)

---

## Overview

**Fact caching** stores gathered facts so they persist between playbook runs. Without caching, Ansible gathers all facts from scratch every run. Caching speeds up execution, reduces network overhead, and lets you access facts from hosts not in the current play.

---

## Why Cache Facts?

```
  WITHOUT CACHING
  ══════════════════════════════════════════════════════════

  Run 1: Gather facts (30 hosts × 3 seconds) = 90 seconds
  Run 2: Gather facts (30 hosts × 3 seconds) = 90 seconds
  Run 3: Gather facts (30 hosts × 3 seconds) = 90 seconds


  WITH CACHING
  ══════════════════════════════════════════════════════════

  Run 1: Gather + cache (30 hosts × 3 sec) = 90 seconds
  Run 2: Read from cache                     = ~1 second
  Run 3: Read from cache                     = ~1 second
  (Cache expires after configured timeout)
```

---

## Caching Backends

```
  ┌─────────────────┬──────────────────────────────────────┐
  │ Backend          │ Description                          │
  ├─────────────────┼──────────────────────────────────────┤
  │ memory           │ Default; lives only for current run  │
  │ jsonfile         │ JSON files on disk                   │
  │ yaml             │ YAML files on disk                   │
  │ redis            │ Redis key-value store                │
  │ memcached        │ Memcached                            │
  │ mongodb          │ MongoDB database                     │
  │ pickle           │ Python pickle files                  │
  └─────────────────┴──────────────────────────────────────┘
```

---

## Configuring Fact Caching

### JSON File Backend (Simple)

```ini
# ansible.cfg
[defaults]
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts_cache
fact_caching_timeout = 86400    # 24 hours in seconds
```

```
  JSON CACHE STRUCTURE
  ══════════════════════════════════════════════════════════

  /tmp/ansible_facts_cache/
  ├── web01                  ◄── One file per host
  ├── web02                      (hostname = filename)
  ├── db01
  └── db02

  Each file contains the full JSON facts for that host.
```

### YAML Backend

```ini
[defaults]
gathering = smart
fact_caching = yaml
fact_caching_connection = /tmp/ansible_facts_cache
fact_caching_timeout = 86400
```

### Redis Backend

```bash
# Install redis plugin
pip install redis
```

```ini
[defaults]
gathering = smart
fact_caching = redis
fact_caching_connection = localhost:6379:0    # host:port:db_number
fact_caching_timeout = 86400
# fact_caching_prefix = ansible_facts_       # Optional key prefix
```

---

## Gathering Strategies

```ini
# ansible.cfg
[defaults]
gathering = smart    # or: implicit, explicit
```

| Strategy | Behavior |
|----------|----------|
| `implicit` | Always gather facts (default without caching) |
| `explicit` | Only gather when `gather_facts: yes` in play |
| `smart` | Gather only if not already cached; use cache if available |

```yaml
# With gathering = smart
- name: First play gathers and caches
  hosts: webservers
  gather_facts: yes       # Gathers and caches
  tasks:
    - debug:
        msg: "{{ ansible_hostname }}"

- name: Second play uses cache
  hosts: webservers
  gather_facts: yes       # Uses cached facts (smart mode)
  tasks:
    - debug:
        msg: "{{ ansible_hostname }}"   # From cache, no SSH
```

---

## set_fact with cacheable

```yaml
tasks:
  - name: Set a fact that persists to cache
    set_fact:
      application_version: "2.1.0"
      cacheable: yes                     # Stored in fact cache

  # In a later playbook run:
  - name: Use cached custom fact
    debug:
      msg: "Version: {{ application_version }}"   # Available from cache
```

---

## Clearing the Cache

```bash
# Delete JSON cache files
rm -rf /tmp/ansible_facts_cache/*

# Clear Redis cache
redis-cli FLUSHDB

# Ansible ignores cache (re-gather)
ansible-playbook site.yml --flush-cache
```

---

## Real-World Scenario: Large Infrastructure Fact Caching

```ini
# ansible.cfg for large environments
[defaults]
gathering = smart
fact_caching = redis
fact_caching_connection = redis-cache.internal:6379:0
fact_caching_timeout = 43200    # 12 hours
gather_subset = !ohai,!facter   # Skip slow third-party facts

[ssh_connection]
pipelining = True               # Faster SSH
```

```yaml
---
# refresh-facts.yml — Run periodically (cron every 12h)
- name: Refresh fact cache
  hosts: all
  gather_facts: yes
  gather_subset: all
  tasks:
    - name: Store deployment info
      set_fact:
        last_fact_refresh: "{{ ansible_date_time.iso8601 }}"
        cacheable: yes

# site.yml — Uses cached facts (fast)
- name: Configure servers
  hosts: webservers
  gather_facts: yes    # Uses cache (smart mode)
  tasks:
    - name: Show cached facts
      debug:
        msg: |
          Host: {{ ansible_hostname }}
          OS: {{ ansible_distribution }}
          Last refresh: {{ last_fact_refresh | default('unknown') }}
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Stale facts being used | Reduce `fact_caching_timeout` or use `--flush-cache` |
| Cache directory not created | Create it manually: `mkdir -p /tmp/ansible_facts_cache` |
| Redis connection refused | Verify Redis is running and accessible from control node |
| `smart` still gathering every time | Ensure `fact_caching` is configured with a persistent backend |
| Custom `set_fact` not cached | Add `cacheable: yes` to the `set_fact` task |
| Permission error on cache files | Check directory permissions; Ansible user needs read/write |

---

## Summary Table

| Setting | Description |
|---------|-------------|
| `gathering = smart` | Use cache if available |
| `fact_caching = jsonfile` | Store facts as JSON files |
| `fact_caching = redis` | Store facts in Redis |
| `fact_caching_connection` | Cache backend path/connection |
| `fact_caching_timeout` | Cache TTL in seconds |
| `cacheable: yes` | Persist `set_fact` to cache |
| `--flush-cache` | Ignore and rebuild cache |
| `gather_subset` | Limit what facts are collected |

---

## Quick Revision Questions

1. **What is the default fact caching backend?**
   > `memory` — facts exist only for the duration of the current playbook run.

2. **What does `gathering = smart` do?**
   > It only gathers facts if they're not already cached, using the configured caching backend.

3. **How do you make a `set_fact` variable persist across runs?**
   > Add `cacheable: yes` to the `set_fact` task and configure a persistent caching backend.

4. **How do you force Ansible to re-gather facts ignoring the cache?**
   > Run with `--flush-cache` flag.

5. **What are the two simplest caching backends for getting started?**
   > `jsonfile` and `yaml` — both store facts as files on disk, requiring no external services.

---

[← Previous: Filters](06-filters.md) | [Next: Role Structure and Concepts →](../06-Roles/01-role-structure.md)
