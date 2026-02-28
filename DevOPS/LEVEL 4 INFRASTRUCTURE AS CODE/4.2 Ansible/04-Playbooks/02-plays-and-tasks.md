# Chapter 4.2: Plays and Tasks

[← Previous: Playbook Structure](01-playbook-structure.md) | [Next: Handlers and Notify →](03-handlers-and-notify.md)

---

## Overview

A **play** targets a set of hosts and defines what to do on them. A **task** is a single unit of work within a play — one call to an Ansible module. Understanding the relationship between plays and tasks is key to writing effective playbooks.

---

## Plays Explained

```
  PLAY STRUCTURE
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────────────┐
  │             PLAY                                    │
  │  ┌────────────────────────────────────────┐         │
  │  │  hosts: webservers                     │ ◄─ WHO  │
  │  │  become: yes                           │ ◄─ HOW  │
  │  │  vars:                                 │ ◄─ WITH │
  │  │    key: value                          │         │
  │  └────────────────────────────────────────┘         │
  │                                                     │
  │  ┌────────────────────────────────────────┐         │
  │  │  TASKS                          WHAT   │         │
  │  │  ┌──────────────────────────────────┐  │         │
  │  │  │ Task 1: Install package          │  │         │
  │  │  └──────────────────────────────────┘  │         │
  │  │  ┌──────────────────────────────────┐  │         │
  │  │  │ Task 2: Configure service        │  │         │
  │  │  └──────────────────────────────────┘  │         │
  │  │  ┌──────────────────────────────────┐  │         │
  │  │  │ Task 3: Start service            │  │         │
  │  │  └──────────────────────────────────┘  │         │
  │  └────────────────────────────────────────┘         │
  └─────────────────────────────────────────────────────┘
```

A play answers three questions:
1. **WHO** — Which hosts to target (`hosts:`)
2. **HOW** — How to connect and escalate (`become:`, `connection:`)
3. **WHAT** — What tasks, roles, and handlers to run

```yaml
---
- name: Configure web tier        # Play description
  hosts: webservers               # WHO
  become: yes                     # HOW (privilege escalation)
  gather_facts: yes               # Collect system info
  tasks:                          # WHAT
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

---

## Task Anatomy

```
  TASK STRUCTURE
  ══════════════════════════════════════════════════════════

  - name: Install nginx           ◄── Human-readable label
    apt:                           ◄── Module name
      name: nginx                  ◄── Module parameter
      state: present               ◄── Module parameter
    become: yes                    ◄── Task-level directive
    when: ansible_os_family == "Debian"  ◄── Conditional
    register: install_result       ◄── Capture output
    notify: Restart nginx          ◄── Trigger handler
    tags:                          ◄── Tags for filtering
      - packages
      - nginx
```

### Task Keywords

```yaml
- name: Full task example
  ansible.builtin.apt:               # Module (FQCN recommended)
    name: nginx
    state: present
  become: yes                        # Privilege escalation
  become_user: root                  # Target user
  when: ansible_os_family == "Debian" # Conditional
  register: result                   # Save return value
  notify: Restart nginx              # Trigger handler
  changed_when: false                # Override changed status
  failed_when: result.rc != 0       # Custom failure condition
  ignore_errors: yes                 # Don't stop on failure
  no_log: true                       # Hide sensitive output
  tags: [install, nginx]             # Tags
  retries: 3                         # Retry count
  delay: 5                           # Seconds between retries
  until: result is success           # Retry condition
  timeout: 300                       # Max seconds for task
  environment:                       # Environment variables
    http_proxy: http://proxy:8080
  delegate_to: localhost             # Run on different host
  run_once: true                     # Run on first host only
```

---

## Task Execution Flow

```
  TASK EXECUTION ON EACH HOST
  ══════════════════════════════════════════════════════════

  ┌─────────────────────┐
  │ Evaluate 'when'     │──── False ───► SKIP
  └──────────┬──────────┘
             │ True
             ▼
  ┌─────────────────────┐
  │ Generate module args │
  │ (template variables) │
  └──────────┬──────────┘
             ▼
  ┌─────────────────────┐
  │ Copy module to host  │
  │ (via SSH/WinRM)      │
  └──────────┬──────────┘
             ▼
  ┌─────────────────────┐
  │ Execute module       │
  │ on remote host       │
  └──────────┬──────────┘
             ▼
  ┌─────────────────────┐
  │ Return JSON result   │
  └──────────┬──────────┘
             ▼
  ┌──────────┴──────────┐
  │ Evaluate result:     │
  │  ok / changed /      │
  │  failed / skipped    │
  └──────────┬──────────┘
             ▼
  ┌─────────────────────┐
  │ Register variable    │
  │ (if register: used)  │
  └──────────┬──────────┘
             ▼
  ┌─────────────────────┐
  │ Notify handler       │
  │ (if changed + notify)│
  └─────────────────────┘
```

---

## Task Return Values

Every task returns a JSON object:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
  register: install_result

- name: Show result
  debug:
    var: install_result
```

Output:
```json
{
    "changed": true,
    "msg": "1 package(s) installed",
    "rc": 0,
    "stdout": "...",
    "stderr": ""
}
```

### Common Return Keys

| Key | Type | Description |
|-----|------|-------------|
| `changed` | bool | Whether the task made changes |
| `failed` | bool | Whether the task failed |
| `msg` | str | Human-readable message |
| `rc` | int | Return code (command modules) |
| `stdout` | str | Standard output |
| `stderr` | str | Standard error |
| `stdout_lines` | list | stdout split by newlines |
| `results` | list | Per-item results (with loops) |
| `skipped` | bool | Whether the task was skipped |

---

## Controlling Task Status

### changed_when

```yaml
# Command always reports "changed"; override it
- name: Check if service exists
  command: systemctl is-active nginx
  register: nginx_status
  changed_when: false     # Never mark as changed

# Mark changed based on output
- name: Run database migration
  command: python manage.py migrate
  register: migration
  changed_when: "'No migrations to apply' not in migration.stdout"
```

### failed_when

```yaml
# Custom failure condition
- name: Check disk space
  command: df -h /
  register: disk
  failed_when: "'100%' in disk.stdout"

# Multiple failure conditions
- name: Run health check
  command: /opt/app/healthcheck.sh
  register: health
  failed_when:
    - health.rc != 0
    - "'degraded' not in health.stdout"
```

---

## Parallel Execution (Strategy)

```
  LINEAR STRATEGY (Default)
  ══════════════════════════════════════════════════════════

  Task 1 ──► Host A ──► Host B ──► Host C ──► DONE
  Task 2 ──► Host A ──► Host B ──► Host C ──► DONE
  Task 3 ──► Host A ──► Host B ──► Host C ──► DONE


  FREE STRATEGY
  ══════════════════════════════════════════════════════════

  Host A: Task 1 ──► Task 2 ──► Task 3 ──► DONE
  Host B:    Task 1 ──► Task 2 ──► Task 3 ──► DONE
  Host C: Task 1 ──► Task 2 ──────► Task 3 ──► DONE

  (Each host progresses independently)
```

```yaml
# Use free strategy
- name: Independent configuration
  hosts: all
  strategy: free
  tasks:
    - name: Update packages
      apt:
        upgrade: yes

# Control parallelism with forks
# In ansible.cfg:
# [defaults]
# forks = 20
```

---

## Serial Execution (Rolling Updates)

```yaml
# Update 2 servers at a time
- name: Rolling deployment
  hosts: webservers
  serial: 2
  tasks:
    - name: Deploy application
      copy:
        src: app.tar.gz
        dest: /opt/app/

# Percentage-based
- name: Canary deployment
  hosts: webservers
  serial: "25%"
  tasks:
    - name: Deploy new version
      copy:
        src: app-v2.tar.gz
        dest: /opt/app/

# Increasing batch size (canary pattern)
- name: Progressive deployment
  hosts: webservers
  serial:
    - 1         # First: test on 1 host
    - 5         # Then: 5 hosts
    - "100%"    # Finally: all remaining
  max_fail_percentage: 10
  tasks:
    - name: Deploy
      copy:
        src: app-v2.tar.gz
        dest: /opt/app/
```

---

## Delegation and Local Tasks

```yaml
# Run task on a different host
- name: Add server to load balancer
  command: /usr/bin/add-to-lb {{ inventory_hostname }}
  delegate_to: lb.example.com

# Run task on control node
- name: Save output locally
  copy:
    content: "{{ report }}"
    dest: /tmp/report.txt
  delegate_to: localhost

# Shortcut for delegate_to: localhost
- name: Get API token
  uri:
    url: https://api.example.com/token
    method: POST
  connection: local

# Run once across all hosts
- name: Send deployment notification
  uri:
    url: https://slack.com/api/chat.postMessage
    method: POST
    body_format: json
    body:
      channel: "#deployments"
      text: "Deployment started"
  delegate_to: localhost
  run_once: true
```

---

## Task Includes and Imports

```yaml
# Static import (processed at parse time)
- name: Configure web server
  hosts: webservers
  tasks:
    - import_tasks: tasks/install_nginx.yml
    - import_tasks: tasks/configure_nginx.yml
      vars:
        server_name: example.com

# Dynamic include (processed at runtime)
- name: OS-specific configuration
  hosts: all
  tasks:
    - include_tasks: "tasks/{{ ansible_os_family }}.yml"
```

```
  IMPORT vs INCLUDE
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────┬───────────────────────────┐
  │     import_tasks        │    include_tasks          │
  ├─────────────────────────┼───────────────────────────┤
  │  Static                 │  Dynamic                  │
  │  Parsed at load time    │  Processed at runtime     │
  │  Can't use loops        │  Can use loops            │
  │  Tags cascade to tasks  │  Tags apply to include    │
  │  Errors caught early    │  Errors at runtime        │
  │  Better for fixed tasks │  Better for conditional   │
  │  Shows in --list-tasks  │  Not in --list-tasks      │
  └─────────────────────────┴───────────────────────────┘
```

---

## Real-World Scenario: Multi-tier Application Deployment

```yaml
---
# deploy-app.yml
- name: Pre-flight checks
  hosts: all
  gather_facts: yes
  tasks:
    - name: Verify minimum disk space
      assert:
        that: ansible_mounts | selectattr('mount','eq','/') | map(attribute='size_available') | first > 1073741824
        msg: "Less than 1GB free on /"

- name: Take web servers out of rotation
  hosts: webservers
  serial: 1
  tasks:
    - name: Disable in LB
      uri:
        url: "http://{{ lb_host }}/api/disable/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost

    - name: Stop application
      service:
        name: myapp
        state: stopped

    - name: Deploy new code
      unarchive:
        src: "releases/myapp-{{ version }}.tar.gz"
        dest: /opt/myapp/

    - name: Start application
      service:
        name: myapp
        state: started

    - name: Wait for app to be ready
      uri:
        url: "http://localhost:8080/health"
        status_code: 200
      retries: 30
      delay: 2
      until: health_check is success
      register: health_check

    - name: Re-enable in LB
      uri:
        url: "http://{{ lb_host }}/api/enable/{{ inventory_hostname }}"
        method: POST
      delegate_to: localhost

- name: Post-deployment verification
  hosts: localhost
  tasks:
    - name: Run smoke tests
      command: pytest tests/smoke/ -v
      register: smoke
      changed_when: false

    - name: Notify team
      uri:
        url: "{{ slack_webhook }}"
        method: POST
        body_format: json
        body:
          text: "v{{ version }} deployed successfully"
      when: smoke is success
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Task runs on wrong host | Check `hosts:` in the play and `delegate_to:` on the task |
| Task always shows "changed" | Use `changed_when: false` if task is read-only |
| Task fails but shouldn't stop play | Use `ignore_errors: yes` or `failed_when:` |
| Tasks run too slowly | Increase `forks`, use `strategy: free`, or enable pipelining |
| `include_tasks` not visible in `--list-tasks` | Includes are dynamic; use `import_tasks` for visibility |
| Variable not available in imported tasks | Use `vars:` on the import or define in play scope |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Play | Container mapping hosts to tasks/roles |
| Task | Single module call; unit of work |
| `register` | Save task output for later use |
| `changed_when` | Override when task reports "changed" |
| `failed_when` | Override when task reports "failed" |
| `delegate_to` | Run task on a different host |
| `run_once` | Execute task on first host only |
| `serial` | Rolling update; limit concurrent hosts |
| `strategy: free` | Hosts progress independently |
| `import_tasks` | Static inclusion at parse time |
| `include_tasks` | Dynamic inclusion at runtime |

---

## Quick Revision Questions

1. **What is the difference between a play and a task?**
   > A play maps a group of hosts to roles/tasks. A task is a single action (module call) within a play.

2. **How do you prevent a task from showing "changed" every run?**
   > Use `changed_when: false` or `changed_when:` with a condition that evaluates the command output.

3. **What is the difference between `import_tasks` and `include_tasks`?**
   > `import_tasks` is static (parsed at playbook load). `include_tasks` is dynamic (processed at runtime), supports loops and variables in filenames.

4. **How does `serial` enable rolling deployments?**
   > `serial` limits how many hosts run the play concurrently (e.g., `serial: 2` runs on 2 hosts at a time, completing fully before the next batch).

5. **What does `delegate_to: localhost` do?**
   > It runs the task on the Ansible control node instead of the target host, while still having access to the target host's variables.

6. **What is the `free` strategy?**
   > Unlike the default `linear` strategy where all hosts complete each task before moving on, `free` lets each host progress through tasks independently at its own pace.

---

[← Previous: Playbook Structure](01-playbook-structure.md) | [Next: Handlers and Notify →](03-handlers-and-notify.md)
