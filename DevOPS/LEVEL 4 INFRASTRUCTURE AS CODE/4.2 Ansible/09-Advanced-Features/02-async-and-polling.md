# Chapter 9.2: Async and Polling

[← Previous: Delegation](01-delegation.md) | [Next: Callback Plugins →](03-callback-plugins.md)

---

## Overview

By default, Ansible waits for each task to complete before moving on. **Async** mode lets you start long-running tasks and either poll for completion or fire-and-forget. This prevents SSH timeouts and allows parallel task execution.

---

## How Async Works

```
  SYNCHRONOUS (default)
  ══════════════════════════════════════════════════════════

  Task starts ─────── waits ─────── waits ─────── completes
  [==================================================] 5 min
  SSH connection held open the entire time


  ASYNC WITH POLLING
  ══════════════════════════════════════════════════════════

  Task starts ── disconnect ── check ── check ── complete!
  [====]          [poll]       [poll]    [poll]
    └─ Returns job ID immediately, polls periodically


  ASYNC FIRE-AND-FORGET (poll: 0)
  ══════════════════════════════════════════════════════════

  Task starts ── disconnect ── moves to next task
  [====]
    └─ Returns job ID, never checks back (unless you do)
```

---

## Basic Async with Polling

```yaml
- name: Run long database migration
  command: /opt/scripts/migrate-database.sh
  async: 3600       # Maximum runtime: 1 hour
  poll: 30          # Check every 30 seconds

# Ansible will:
# 1. Start the command
# 2. Disconnect
# 3. Check every 30 seconds
# 4. Fail if not done in 3600 seconds
```

---

## Fire-and-Forget (poll: 0)

```yaml
- name: Start long backup
  command: /opt/scripts/full-backup.sh
  async: 7200        # Max 2 hours
  poll: 0            # Don't wait
  register: backup_job

# Continue with other tasks...
- name: Do other work
  apt:
    name: nginx
    state: present

# Check later
- name: Check if backup is done
  async_status:
    jid: "{{ backup_job.ansible_job_id }}"
  register: backup_result
  until: backup_result.finished
  retries: 60
  delay: 60          # Check every minute, up to 60 times
```

---

## async_status Module

```yaml
- name: Check job status
  async_status:
    jid: "{{ job.ansible_job_id }}"
  register: result

- debug:
    msg: |
      Finished: {{ result.finished }}       # 0 or 1
      Started: {{ result.started }}
      RC: {{ result.rc | default('n/a') }}
      Stdout: {{ result.stdout | default('') }}
```

---

## Parallel Async Tasks

Run multiple long tasks simultaneously, then wait for all:

```yaml
- name: Start backups on all databases
  command: "/opt/scripts/backup-db.sh {{ item }}"
  async: 3600
  poll: 0
  register: backup_jobs
  loop:
    - users_db
    - orders_db
    - analytics_db

- name: Wait for all backups to complete
  async_status:
    jid: "{{ item.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 120
  delay: 30
  loop: "{{ backup_jobs.results }}"
```

```
  PARALLEL ASYNC PATTERN
  ══════════════════════════════════════════════════════════

  Time ──────────────────────────────────────────────▶

  backup users_db:     [===============================]
  backup orders_db:    [====================]
  backup analytics_db: [=========================]
                                                     ▲
                                              All finish,
                                              playbook continues
```

---

## Avoiding SSH Timeouts

```yaml
# Problem: this task takes 20 minutes, SSH times out at 10
- name: Large data import (will timeout!)
  command: /opt/scripts/import-data.sh
  # SSH connection drops after 10 minutes → task fails

# Solution: async
- name: Large data import (async)
  command: /opt/scripts/import-data.sh
  async: 3600      # Allow up to 1 hour
  poll: 60         # Check every minute
```

---

## Async with Error Handling

```yaml
- name: Start risky operation
  command: /opt/scripts/risky-operation.sh
  async: 1800
  poll: 0
  register: risky_job

- name: Do intermediate work
  debug:
    msg: "Working on other things..."

- name: Check risky operation
  async_status:
    jid: "{{ risky_job.ansible_job_id }}"
  register: risky_result
  until: risky_result.finished
  retries: 30
  delay: 60
  failed_when: false     # Don't fail the play

- name: Handle failure
  debug:
    msg: "Risky operation failed: {{ risky_result.stderr }}"
  when: risky_result.finished and risky_result.rc != 0

- name: Handle success
  debug:
    msg: "Risky operation succeeded"
  when: risky_result.finished and risky_result.rc == 0
```

---

## Real-World Scenario: Parallel Package Updates

```yaml
---
- name: Update all servers in parallel
  hosts: all
  become: yes
  tasks:
    - name: Start system update
      apt:
        upgrade: dist
        update_cache: yes
      async: 3600
      poll: 0
      register: update_job

    - name: Wait for updates to complete
      async_status:
        jid: "{{ update_job.ansible_job_id }}"
      register: update_result
      until: update_result.finished
      retries: 60
      delay: 60

    - name: Check if reboot needed
      stat:
        path: /var/run/reboot-required
      register: reboot_required

    - name: Reboot if needed
      reboot:
        reboot_timeout: 300
      when: reboot_required.stat.exists
```

---

## Limitations

| Limitation | Description |
|-----------|-------------|
| Not all modules support async | Modules that require persistent connection context may fail |
| `async` ignored for some modules | `service`, `systemd` don't normally need it |
| Fire-and-forget may lose results | If the remote host reboots, job status is lost |
| Cannot use `become` password | Interactive password prompts don't work with async |

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Task times out | Increase `async` value |
| `async_status` shows `finished: 0` forever | Check if process is actually running on remote; may have crashed |
| Job ID not found | Job result files in `~/.ansible_async/` may have been cleaned |
| SSH timeout despite async | Ensure `async` is set on the correct task |
| Can't poll fire-and-forget tasks | Register the result and use `async_status` later |

---

## Summary Table

| Parameter | Description |
|-----------|-------------|
| `async: N` | Maximum runtime in seconds |
| `poll: N` | Check interval in seconds |
| `poll: 0` | Fire-and-forget (don't wait) |
| `async_status` | Check status of async job |
| `ansible_job_id` | Job identifier from registered result |
| `result.finished` | Whether job has completed (0/1) |

---

## Quick Revision Questions

1. **What do the `async` and `poll` parameters do?**
   > `async` sets the maximum allowed runtime in seconds. `poll` sets how often Ansible checks if the task is done.

2. **What does `poll: 0` mean?**
   > Fire-and-forget — Ansible starts the task and immediately moves on without waiting for it to complete.

3. **How do you check the status of an async task later?**
   > Use the `async_status` module with the `ansible_job_id` from the registered task result.

4. **Why would you use async instead of just increasing the SSH timeout?**
   > Async disconnects SSH between checks, avoiding connection drops. It also allows parallel execution of multiple long-running tasks.

5. **How do you wait for multiple fire-and-forget tasks to complete?**
   > Register all jobs, then loop over them with `async_status` using `until: result.finished` and `retries`/`delay`.

---

[← Previous: Delegation](01-delegation.md) | [Next: Callback Plugins →](03-callback-plugins.md)
