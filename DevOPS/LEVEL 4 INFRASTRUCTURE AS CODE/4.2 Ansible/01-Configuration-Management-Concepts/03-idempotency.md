# Chapter 1.3: Idempotency

[← Previous: Push vs Pull Model](02-push-vs-pull-model.md) | [Next: CM Tools Comparison →](04-cm-tools-comparison.md)

---

## Overview

**Idempotency** is one of the most important concepts in configuration management. An operation is idempotent if **running it once or multiple times produces the exact same result** without unintended side effects.

---

## Understanding Idempotency

```
  IDEMPOTENT OPERATION
  ══════════════════════════════════════════════════

  Run 1:  State A ──── Apply Config ────► State B  (changed)
  Run 2:  State B ──── Apply Config ────► State B  (no change)
  Run 3:  State B ──── Apply Config ────► State B  (no change)
  ...
  Run N:  State B ──── Apply Config ────► State B  (no change)

  Result: ALWAYS State B, regardless of how many times run
```

```
  NON-IDEMPOTENT OPERATION
  ══════════════════════════════════════════════════

  Run 1:  State A ──── Append line ────► State B  (1 line added)
  Run 2:  State B ──── Append line ────► State C  (2 lines now!)
  Run 3:  State C ──── Append line ────► State D  (3 lines now!)
  ...
  Run N:  State ? ──── Append line ────► State ?? (N lines!)

  Result: DIFFERENT each time → NOT idempotent ❌
```

---

## Why Idempotency Matters

```
  ┌─────────────────────────────────────────────────────────┐
  │           WHY IDEMPOTENCY IS CRITICAL                    │
  ├─────────────────────────────────────────────────────────┤
  │                                                         │
  │  1. SAFETY                                              │
  │     Running a playbook twice won't break anything       │
  │                                                         │
  │  2. DRIFT CORRECTION                                    │
  │     Re-running brings systems back to desired state     │
  │                                                         │
  │  3. PREDICTABILITY                                      │
  │     You always know what the end state will be          │
  │                                                         │
  │  4. CONFIDENCE                                          │
  │     "It's safe to run again" - no fear of side effects  │
  │                                                         │
  │  5. CONTINUOUS ENFORCEMENT                              │
  │     Can run on a schedule without accumulating changes  │
  │                                                         │
  └─────────────────────────────────────────────────────────┘
```

---

## Idempotent vs Non-Idempotent Examples

### Example 1: Package Installation

```yaml
# ✅ IDEMPOTENT - Ansible apt module
- name: Install nginx
  apt:
    name: nginx
    state: present    # "ensure it's there"
  # Run 1: Not installed → installs nginx (changed)
  # Run 2: Already installed → does nothing (ok)
  # Run 3: Already installed → does nothing (ok)

# ❌ NON-IDEMPOTENT - Shell command
- name: Install nginx (bad way)
  shell: apt-get install -y nginx
  # Run 1: Installs nginx
  # Run 2: Runs again unnecessarily (always "changed")
  # Run 3: Runs again unnecessarily (always "changed")
```

### Example 2: File Content

```yaml
# ✅ IDEMPOTENT - copy module
- name: Set MOTD
  copy:
    content: "Welcome to Production Server\n"
    dest: /etc/motd
  # Always results in the same file content

# ❌ NON-IDEMPOTENT - shell append
- name: Set MOTD (bad way)
  shell: echo "Welcome to Production Server" >> /etc/motd
  # Run 1: 1 line
  # Run 2: 2 lines (duplicated!)
  # Run 3: 3 lines (tripled!)
```

### Example 3: User Management

```yaml
# ✅ IDEMPOTENT - user module
- name: Create deploy user
  user:
    name: deploy
    uid: 1500
    groups: sudo
    state: present
  # Run 1: Creates user (changed)
  # Run 2: User exists with correct settings (ok)

# ❌ NON-IDEMPOTENT - shell command
- name: Create deploy user (bad way)
  shell: useradd -u 1500 -G sudo deploy
  # Run 1: Creates user
  # Run 2: ERROR! User already exists
```

---

## How Ansible Achieves Idempotency

```
  ANSIBLE MODULE EXECUTION FLOW
  ══════════════════════════════════════════════════

  ┌────────────────┐
  │  Task Called    │
  │  apt:          │
  │    name: nginx │
  │    state:      │
  │    present     │
  └───────┬────────┘
          │
          ▼
  ┌────────────────┐     ┌─────────────┐
  │  Check Current │────►│ Is nginx    │
  │  State         │     │ installed?  │
  └────────────────┘     └──────┬──────┘
                                │
                    ┌───────────┼───────────┐
                    │ YES                   │ NO
                    ▼                       ▼
           ┌──────────────┐       ┌──────────────┐
           │  Do Nothing  │       │  Install     │
           │  Report: ok  │       │  nginx       │
           │  changed: no │       │  Report: ok  │
           └──────────────┘       │  changed: yes│
                                  └──────────────┘
```

### The Check → Compare → Act Pattern

Most Ansible modules follow this pattern:

```
  1. GATHER   →  Check current state on the target
  2. COMPARE  →  Compare current state with desired state
  3. ACT      →  Only make changes if there's a difference
  4. REPORT   →  Return what happened (changed/ok/failed)
```

---

## Making Non-Idempotent Tasks Idempotent

### Strategy 1: Use `creates` or `removes` Parameters

```yaml
# ❌ Non-idempotent
- name: Download application
  shell: wget https://example.com/app.tar.gz -O /opt/app.tar.gz

# ✅ Made idempotent with 'creates'
- name: Download application
  shell: wget https://example.com/app.tar.gz -O /opt/app.tar.gz
  args:
    creates: /opt/app.tar.gz    # Skip if file already exists
```

### Strategy 2: Use `changed_when` and `failed_when`

```yaml
# Check first, then act
- name: Check if database exists
  shell: mysql -e "SHOW DATABASES LIKE 'myapp'"
  register: db_check
  changed_when: false    # This check never "changes" anything

- name: Create database if not exists
  shell: mysql -e "CREATE DATABASE myapp"
  when: "'myapp' not in db_check.stdout"
```

### Strategy 3: Use Proper Modules Instead of Shell

```yaml
# ❌ Non-idempotent shell
- name: Add config line
  shell: echo "max_connections = 100" >> /etc/postgresql/postgresql.conf

# ✅ Idempotent lineinfile module
- name: Set max connections
  lineinfile:
    path: /etc/postgresql/postgresql.conf
    regexp: '^max_connections'
    line: 'max_connections = 100'
```

### Strategy 4: Use `stat` Module for Conditional Execution

```yaml
- name: Check if app is already installed
  stat:
    path: /opt/myapp/bin/myapp
  register: app_binary

- name: Install application
  shell: /tmp/install.sh
  when: not app_binary.stat.exists
```

---

## Ansible Output and Idempotency

```
  PLAY RECAP
  ═══════════════════════════════════════════════════

  First Run (changes needed):
  ──────────────────────────────────────────────────
  webserver1  : ok=5  changed=4  unreachable=0  failed=0
  webserver2  : ok=5  changed=4  unreachable=0  failed=0

  Second Run (already in desired state):
  ──────────────────────────────────────────────────
  webserver1  : ok=5  changed=0  unreachable=0  failed=0
  webserver2  : ok=5  changed=0  unreachable=0  failed=0
                      ─────────
                      ▲
                      │
              This should be 0 on subsequent runs!
              If not, your playbook has idempotency issues.
```

---

## Common Idempotency Pitfalls

```
  ┌──────────────────────────────────────────────────────────┐
  │              IDEMPOTENCY VIOLATIONS                       │
  ├──────────────────────────────────────────────────────────┤
  │                                                          │
  │  1. Using shell/command without guards                   │
  │     shell: echo "data" >> /etc/config                    │
  │                                                          │
  │  2. Restarting services unconditionally                  │
  │     service: name=nginx state=restarted  ← always runs! │
  │     USE: handlers instead                                │
  │                                                          │
  │  3. Using get_url without checksum                       │
  │     get_url: url=... dest=...  ← re-downloads each time │
  │                                                          │
  │  4. Running scripts without creates/removes              │
  │     script: setup.sh  ← runs every time                 │
  │                                                          │
  │  5. Using shell for things modules can do                │
  │     shell: useradd bob  ← use 'user' module instead     │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Safe Re-runs After Failure
A playbook fails halfway through at task 6 of 10. You fix the issue and re-run:
- **Idempotent**: Tasks 1-5 report "ok" (no changes), task 6+ applies remaining changes
- **Non-idempotent**: Tasks 1-5 duplicate actions, potentially breaking things

### Scenario 2: Scheduled Compliance Runs
You schedule a playbook to run every 4 hours for compliance:
- **Idempotent**: Only makes changes when drift is detected
- **Non-idempotent**: Keeps modifying files, restarting services every 4 hours

### Scenario 3: Multiple Team Members
Three engineers run the same playbook against the same servers:
- **Idempotent**: First run makes changes, subsequent runs are no-ops
- **Non-idempotent**: Each run adds duplicate configurations

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `changed` count not zero on re-run | Check for shell/command tasks without guards |
| Services restarting every run | Use handlers instead of `state: restarted` |
| Files growing with duplicate lines | Use `lineinfile` or `blockinfile` instead of shell append |
| Scripts running every time | Add `creates:` parameter or use `stat` + `when` |
| `--check` mode shows false changes | Some modules don't support check mode; test manually |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Idempotency | Same result regardless of number of executions |
| changed=0 | Indicator that playbook is idempotent on re-run |
| Check-Compare-Act | Pattern used by modules to ensure idempotency |
| `creates` | Shell/command parameter to skip if file exists |
| `changed_when` | Override when Ansible considers a task "changed" |
| `when` | Conditional to control task execution |
| Handlers | Run actions only when notified (e.g., restart service) |
| lineinfile | Idempotent module for managing lines in files |

---

## Quick Revision Questions

1. **Define idempotency in the context of configuration management.**
   > An operation is idempotent if running it once or multiple times produces the exact same end state without side effects.

2. **How can you verify that a playbook is idempotent?**
   > Run it twice and check that the second run shows `changed=0` for all hosts.

3. **Why is `shell: echo "line" >> file` not idempotent?**
   > Because each run appends another copy of the line, resulting in duplicates.

4. **How does the `creates` parameter help with idempotency?**
   > It tells shell/command to skip execution if the specified file already exists.

5. **What Ansible module should you use instead of `shell: useradd`?**
   > The `user` module, which checks if the user exists before creating.

6. **Why should you use handlers instead of `service: state=restarted`?**
   > Handlers only run when notified by a changed task, avoiding unnecessary service restarts on every playbook run.

---

[← Previous: Push vs Pull Model](02-push-vs-pull-model.md) | [Next: CM Tools Comparison →](04-cm-tools-comparison.md)
