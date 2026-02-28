# Chapter 9.6: Performance Tuning

[← Previous: Ansible Tower / AWX](05-ansible-tower-awx.md) | [Next: Directory Layout →](../10-Best-Practices/01-directory-layout.md)

---

## Overview

As Ansible environments scale to hundreds or thousands of hosts, performance becomes critical. This chapter covers every major tuning lever — from SSH optimisation to pipelining, parallelism, fact caching, and strategy plugins.

---

## Performance Bottleneck Areas

```
  WHERE TIME IS SPENT
  ══════════════════════════════════════════════════════════

  ┌──────────────────────────────────────────────────────┐
  │                  Playbook Execution                  │
  │                                                      │
  │  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  │
  │  │ SSH      │  │ Fact     │  │ Module Transfer   │  │
  │  │ Connect  │  │ Gather   │  │ & Execution       │  │
  │  │          │  │          │  │                   │  │
  │  │ ~40%     │  │ ~25%     │  │ ~35%              │  │
  │  └──────────┘  └──────────┘  └───────────────────┘  │
  │                                                      │
  │  ◄── Optimise these three areas ──▶                  │
  └──────────────────────────────────────────────────────┘
```

---

## 1. SSH Optimisation

### ControlPersist (Persistent Connections)

```ini
# ansible.cfg
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o ControlPath=/tmp/ansible-%r@%h:%p
```

```
  WITHOUT ControlPersist          WITH ControlPersist
  ════════════════════            ═══════════════════
  Task 1: SSH connect (2s)       Task 1: SSH connect (2s)
  Task 2: SSH connect (2s)       Task 2: Reuse (0.01s)  ◄── Huge saving
  Task 3: SSH connect (2s)       Task 3: Reuse (0.01s)
  Task 4: SSH connect (2s)       Task 4: Reuse (0.01s)
  ────────────────────           ───────────────────
  Total SSH: 8s                  Total SSH: ~2s
```

### Pipelining

```ini
# ansible.cfg
[ssh_connection]
pipelining = True
```

```
  WITHOUT Pipelining              WITH Pipelining
  ════════════════════            ═══════════════════
  1. SSH connect                  1. SSH connect
  2. Create temp dir              2. Pipe module via stdin
  3. Copy module to temp dir      3. Execute module
  4. Execute module               4. Return JSON
  5. Clean temp dir               (No file transfer!)
  6. Return JSON

  Steps: 6                       Steps: 4
```

> **Note:** Pipelining requires `requiretty` to be disabled in `/etc/sudoers` on managed nodes: comment out `Defaults requiretty`.

---

## 2. Parallelism (Forks)

```ini
# ansible.cfg
[defaults]
forks = 20      # Default is 5
```

```
  forks = 5 (default)           forks = 20
  ════════════════════           ═══════════════════
  Batch 1: [h1-h5]              Batch 1: [h1-h20]
  Batch 2: [h6-h10]             Batch 2: [h21-h40]
  Batch 3: [h11-h15]            Batch 3: [h41-h50]
  Batch 4: [h16-h20]
  ...10 batches for 50 hosts    ...3 batches for 50 hosts
```

```bash
# Override at runtime
ansible-playbook site.yml -f 30
```

> **Rule of thumb:** Set forks to the number of CPU cores × 2-5, or the number of hosts — whichever is smaller.

---

## 3. Fact Gathering Optimisation

### Disable When Not Needed

```yaml
- name: Deploy static files
  hosts: webservers
  gather_facts: false       # Skip if facts not used
  tasks:
    - name: Copy files
      copy:
        src: files/
        dest: /var/www/
```

### Gather Only What You Need

```yaml
- name: Targeted fact gathering
  hosts: all
  gather_facts: true
  gather_subset:
    - '!all'
    - '!min'
    - network        # Only network facts
    - hardware       # Only hardware facts
```

| Subset | Content |
|--------|---------|
| `all` | Everything (default) |
| `min` | Minimal (hostname, IPs, OS) |
| `hardware` | CPU, memory, disk |
| `network` | Interfaces, IPs, routes |
| `virtual` | Virtualisation info |
| `!all` | Exclude all (use with specific subsets) |

### Fact Caching

```ini
# ansible.cfg
[defaults]
gathering = smart                    # Only gather if cache expired
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts_cache
fact_caching_timeout = 3600          # 1 hour

# Or use Redis for multi-user environments
# fact_caching = redis
# fact_caching_connection = localhost:6379:0
```

```
  FACT CACHING FLOW
  ══════════════════════════════════════════════════════════

  First Run:                    Subsequent Runs:
  gather_facts ──▶ SSH to host  gather_facts ──▶ Read cache
                   collect               (if not expired)
                   cache locally         0.001s vs 2-5s
```

---

## 4. Strategy Plugins

```ini
# ansible.cfg
[defaults]
strategy = free
```

### linear (Default)

```
  linear: All hosts complete task before moving on
  ══════════════════════════════════════════════════════════
  Task 1: ──h1──  ──h2──  ──h3──  │ wait │
  Task 2:                          ──h1── ──h2── ──h3──│
  Task 3:                                              ──...
```

### free

```
  free: Hosts proceed independently
  ══════════════════════════════════════════════════════════
  Host 1: ──T1── ──T2── ──T3── ──T4── done
  Host 2: ──T1──── ──T2── ──T3────── ──T4── done
  Host 3: ──T1── ──T2──── ──T3── ──T4────── done

  No waiting — each host moves to next task immediately
```

```yaml
# Per-play strategy
- name: Fast independent deployment
  hosts: webservers
  strategy: free
  tasks:
    - name: Update package
      apt:
        name: nginx
        state: latest
```

### host_pinned

Like `free` but keeps a host's tasks on the same worker — reduces context switching.

---

## 5. Mitogen Strategy Plugin

**Mitogen** is a third-party plugin that dramatically speeds up Ansible by replacing SSH-based module transfer with a persistent Python interpreter tunnel.

```bash
pip install mitogen
```

```ini
# ansible.cfg
[defaults]
strategy_plugins = /path/to/mitogen/ansible_mitogen/plugins/strategy
strategy = mitogen_linear
```

```
  STANDARD ANSIBLE               MITOGEN
  ════════════════════           ═══════════════════
  Per task:                      One-time setup:
  1. SSH connect                 1. SSH connect
  2. Create temp dir             2. Bootstrap Python
  3. sftp module.py              
  4. Execute module              Per task:
  5. Read result                 1. Send bytecode
  6. Delete temp                 2. Execute in-process
                                 3. Return result
  
  ~150ms overhead/task           ~15ms overhead/task
```

> Mitogen can provide **1.5-7x speedup** depending on workload.

---

## 6. Async Tasks for Parallelism

```yaml
- name: Long-running tasks in parallel
  hosts: all
  tasks:
    - name: Run dist-upgrade
      apt:
        upgrade: dist
      async: 3600
      poll: 0
      register: upgrade_job

    - name: Do other work while upgrade runs
      debug:
        msg: "Upgrade running in background"

    - name: Wait for upgrade
      async_status:
        jid: "{{ upgrade_job.ansible_job_id }}"
      register: result
      until: result.finished
      retries: 60
      delay: 30
```

---

## 7. Additional Tuning Settings

```ini
# ansible.cfg — all performance-related settings
[defaults]
forks = 20
strategy = free
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/facts_cache
fact_caching_timeout = 3600
internal_poll_interval = 0.001      # Reduce polling interval
stdout_callback = yaml

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
control_path_dir = /tmp/ansible-cp
retries = 3

# Transfer method (default: smart)
transfer_method = piped             # Avoid sftp overhead

[inventory]
enable_plugins = host_list, ini, yaml
# Disable plugins you don't use
```

### Callback Plugins for Profiling

```ini
[defaults]
callbacks_enabled = timer, profile_tasks, profile_roles
```

```
  PROFILE OUTPUT
  ══════════════════════════════════════════════════════════

  Thursday 15 Jan 2024  14:30:00 +0000 (0:00:02.123)
  ============================================================
  apt : Install packages ------------------------------ 45.32s
  template : Deploy nginx config ----------------------  3.12s
  service : Restart nginx -----------------------------  1.05s
  setup ------------------------------------------------ 2.50s
  ============================================================
  Total: 52.00s

  ◄── Identify the 45s apt task as the bottleneck
```

---

## Performance Tuning Checklist

```
  ┌─────────────────────────────────────────────────────┐
  │  PERFORMANCE TUNING CHECKLIST                       │
  │                                                     │
  │  SSH Layer:                                         │
  │  [✓] ControlPersist enabled                         │
  │  [✓] Pipelining enabled                             │
  │  [✓] requiretty disabled on managed nodes           │
  │                                                     │
  │  Parallelism:                                       │
  │  [✓] Forks increased from default 5                 │
  │  [✓] Strategy = free (if task order doesn't matter) │
  │                                                     │
  │  Facts:                                             │
  │  [✓] gather_facts: false where not needed           │
  │  [✓] gather_subset limits applied                   │
  │  [✓] Fact caching enabled (jsonfile or redis)       │
  │                                                     │
  │  Profiling:                                         │
  │  [✓] profile_tasks enabled to find bottlenecks      │
  │  [✓] timer callback for total runtime               │
  │                                                     │
  │  Advanced:                                          │
  │  [ ] Mitogen (if compatible with Ansible version)   │
  │  [ ] Async tasks for long-running operations        │
  │  [ ] serial batching for rolling updates            │
  └─────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| High SSH overhead | Enable ControlPersist and pipelining |
| Slow fact gathering | Use `gather_facts: false` or `gather_subset` |
| Memory issues with high forks | Reduce forks; each fork uses ~100MB |
| Pipelining breaks sudo | Disable `requiretty` in `/etc/sudoers` |
| Mitogen compatibility issues | Check version compatibility matrix |
| Tasks slower than expected | Enable `profile_tasks` to identify bottlenecks |
| Control path too long | Shorten `control_path_dir` path |

---

## Summary Table

| Technique | Setting | Impact |
|-----------|---------|--------|
| ControlPersist | `ssh_args = -o ControlPersist=60s` | Reuse SSH connections |
| Pipelining | `pipelining = True` | Skip module file transfer |
| Forks | `forks = 20` | Parallel host execution |
| Free strategy | `strategy = free` | No inter-host waiting |
| Fact caching | `fact_caching = jsonfile` | Skip repeated fact gathering |
| Gather subset | `gather_subset: [network]` | Collect fewer facts |
| Disable facts | `gather_facts: false` | Skip entirely |
| Mitogen | `strategy = mitogen_linear` | 1.5-7x speedup |
| Async | `async: 3600, poll: 0` | Background long tasks |
| Profiling | `callbacks_enabled = profile_tasks` | Find bottlenecks |

---

## Quick Revision Questions

1. **What does pipelining do and what is its prerequisite?**
   > Pipelining sends the module code via stdin instead of copying files over SFTP, reducing overhead. It requires `requiretty` to be disabled in `/etc/sudoers` on managed nodes.

2. **What is the default value of forks and how would you increase it?**
   > Default is 5. Increase it in `ansible.cfg` with `forks = 20` or at runtime with `-f 20`.

3. **How does the `free` strategy differ from `linear`?**
   > `linear` waits for all hosts to complete a task before proceeding. `free` lets each host move to its next task independently, eliminating wait time.

4. **What are three ways to optimise fact gathering?**
   > Disable with `gather_facts: false`, limit with `gather_subset`, or cache with `fact_caching` (jsonfile/redis).

5. **How does Mitogen improve Ansible performance?**
   > Mitogen replaces SSH-based file transfer with a persistent Python interpreter tunnel, reducing per-task overhead from ~150ms to ~15ms (1.5-7x speedup).

6. **How would you profile which tasks are slowest?**
   > Enable the `profile_tasks` callback plugin in `callbacks_enabled` to see execution time per task, sorted by duration.

---

[← Previous: Ansible Tower / AWX](05-ansible-tower-awx.md) | [Next: Directory Layout →](../10-Best-Practices/01-directory-layout.md)
