# Chapter 4.1: Playbook Structure

[← Previous: Inventory Plugins](../03-Inventory-Management/06-inventory-plugins.md) | [Next: Plays and Tasks →](02-plays-and-tasks.md)

---

## Overview

A playbook is an **ordered list of plays** written in YAML. Each play maps a group of hosts to a set of tasks. Playbooks are the foundation of Ansible automation — they define **what** to configure and **how**.

---

## Anatomy of a Playbook

```
  PLAYBOOK STRUCTURE
  ══════════════════════════════════════════════════════════

  playbook.yml
  ┌─────────────────────────────────────────────────────┐
  │  ---                                                │
  │  # Play 1                                           │
  │  - name: Configure web servers                      │
  │    hosts: webservers                                │
  │    become: yes                                      │
  │    vars:                                            │
  │      http_port: 80                                  │
  │    tasks:                                           │
  │      - name: Install nginx     ◄── Task 1          │
  │        apt:                                         │
  │          name: nginx                                │
  │          state: present                             │
  │      - name: Start nginx       ◄── Task 2          │
  │        service:                                     │
  │          name: nginx                                │
  │          state: started                             │
  │    handlers:                                        │
  │      - name: Restart nginx     ◄── Handler         │
  │        service:                                     │
  │          name: nginx                                │
  │          state: restarted                           │
  │                                                     │
  │  # Play 2                                           │
  │  - name: Configure database servers                 │
  │    hosts: dbservers                                 │
  │    become: yes                                      │
  │    tasks:                                           │
  │      - name: Install PostgreSQL                     │
  │        apt:                                         │
  │          name: postgresql                           │
  │          state: present                             │
  └─────────────────────────────────────────────────────┘
```

---

## Minimal Playbook

```yaml
---
- name: My first playbook
  hosts: all
  tasks:
    - name: Ping all hosts
      ping:
```

## Complete Playbook

```yaml
---
# site.yml - Main playbook
- name: Configure web servers
  hosts: webservers
  become: yes
  gather_facts: yes
  vars:
    http_port: 80
    document_root: /var/www/html
  vars_files:
    - vars/common.yml
    - vars/secrets.yml
  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
  roles:
    - common
    - nginx
  tasks:
    - name: Deploy application
      copy:
        src: files/index.html
        dest: "{{ document_root }}/index.html"
      notify: Restart nginx
  post_tasks:
    - name: Verify site is up
      uri:
        url: "http://localhost:{{ http_port }}"
        status_code: 200
  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

---

## Play-level Keywords

```
  PLAY KEYWORDS
  ══════════════════════════════════════════════════════════

  ┌──────────────────┬──────────────────────────────────────┐
  │ Keyword          │ Description                          │
  ├──────────────────┼──────────────────────────────────────┤
  │ name             │ Description of the play              │
  │ hosts            │ Target host pattern                  │
  │ become           │ Enable privilege escalation          │
  │ become_user      │ User to escalate to (default: root)  │
  │ gather_facts     │ Collect system facts (default: yes)  │
  │ vars             │ Define play-level variables          │
  │ vars_files       │ Load variables from files            │
  │ vars_prompt      │ Prompt user for variables            │
  │ pre_tasks        │ Tasks to run before roles            │
  │ roles            │ Roles to apply                       │
  │ tasks            │ Main task list                       │
  │ post_tasks       │ Tasks to run after roles/tasks       │
  │ handlers         │ Tasks triggered by notify            │
  │ environment      │ Set environment variables            │
  │ serial           │ Rolling update batch size            │
  │ max_fail_percentage │ Acceptable failure threshold     │
  │ any_errors_fatal │ Stop on any host failure             │
  │ strategy         │ Execution strategy (linear/free)     │
  │ tags             │ Tags for selective execution         │
  │ connection       │ Connection type override             │
  │ collections      │ Collections to search                │
  └──────────────────┴──────────────────────────────────────┘
```

---

## Execution Order

```
  PLAY EXECUTION ORDER
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────┐
  │  1. Variable Loading            │
  │     (vars, vars_files,          │
  │      group_vars, host_vars)     │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │  2. Fact Gathering              │
  │     (gather_facts: yes)         │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │  3. pre_tasks                   │
  │     + handlers if notified      │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │  4. roles                       │
  │     + handlers if notified      │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │  5. tasks                       │
  │     + handlers if notified      │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │  6. post_tasks                  │
  │     + handlers if notified      │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │  7. Remaining handlers          │
  └─────────────────────────────────┘
```

---

## Multi-Play Playbooks

```yaml
---
# site.yml - Orchestrating multiple plays
- name: Common configuration for all servers
  hosts: all
  become: yes
  roles:
    - common
    - security

- name: Configure load balancers
  hosts: loadbalancers
  become: yes
  roles:
    - haproxy

- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - nginx
    - app-deploy

- name: Configure database servers
  hosts: dbservers
  become: yes
  roles:
    - postgresql

- name: Verify deployment
  hosts: loadbalancers
  tasks:
    - name: Check health endpoint
      uri:
        url: http://localhost/health
        status_code: 200
```

---

## Running Playbooks

```bash
# Basic execution
ansible-playbook site.yml

# With specific inventory
ansible-playbook -i inventory/production site.yml

# Dry run (check mode)
ansible-playbook site.yml --check

# Dry run with diff
ansible-playbook site.yml --check --diff

# Syntax check only
ansible-playbook site.yml --syntax-check

# Limit to specific hosts
ansible-playbook site.yml --limit webservers

# Verbose output
ansible-playbook site.yml -v    # or -vv, -vvv, -vvvv

# Step through tasks interactively
ansible-playbook site.yml --step

# Start at a specific task
ansible-playbook site.yml --start-at-task "Deploy application"

# List all tasks without running
ansible-playbook site.yml --list-tasks

# List all hosts that would be affected
ansible-playbook site.yml --list-hosts
```

---

## Importing and Including Playbooks

```yaml
# site.yml - Main entry point
---
# Import (static, resolved at parse time)
- import_playbook: webservers.yml
- import_playbook: dbservers.yml
- import_playbook: monitoring.yml

# Can also use conditional imports
- import_playbook: disaster_recovery.yml
  when: dr_mode | default(false)
```

---

## Real-World Scenario: Full Stack Deployment

```yaml
# deploy.yml
---
- name: Pre-deployment checks
  hosts: loadbalancers
  tasks:
    - name: Disable server in LB
      haproxy:
        state: disabled
        backend: app_backend
        host: "{{ item }}"
      loop: "{{ groups['webservers'] }}"

- name: Deploy application
  hosts: webservers
  serial: 1                    # One server at a time
  become: yes
  tasks:
    - name: Pull latest code
      git:
        repo: https://github.com/org/app.git
        dest: /opt/app
        version: "{{ app_version }}"
    - name: Restart application
      service:
        name: myapp
        state: restarted

- name: Re-enable in load balancer
  hosts: loadbalancers
  tasks:
    - name: Enable server in LB
      haproxy:
        state: enabled
        backend: app_backend
        host: "{{ item }}"
      loop: "{{ groups['webservers'] }}"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| YAML syntax error | Run `ansible-playbook --syntax-check` or use a YAML linter |
| Tasks not running in order | Verify indentation; tasks are under correct play |
| Handlers not firing | Ensure task has `notify:` and the handler name matches exactly |
| "No hosts matched" | Check `hosts:` pattern and inventory |
| Play skipping all tasks | Check `when:` conditionals and `gather_facts` if facts are needed |

---

## Summary Table

| Element | Description |
|---------|-------------|
| Playbook | YAML file containing one or more plays |
| Play | Maps hosts to tasks/roles |
| Task | Single action using a module |
| Handler | Task that runs only when notified |
| `pre_tasks` | Run before roles |
| `post_tasks` | Run after tasks |
| `serial` | Control rolling update batch size |
| `become` | Enable sudo/root execution |
| `gather_facts` | Collect system info before tasks |
| `import_playbook` | Include another playbook statically |

---

## Quick Revision Questions

1. **What is the execution order within a play?**
   > vars loading → fact gathering → pre_tasks → roles → tasks → post_tasks → handlers.

2. **What is the difference between `import_playbook` and `include_playbook`?**
   > `import_playbook` is static (parsed at load time). There's no `include_playbook` — use `import_playbook` for playbook-level inclusion.

3. **What does `serial: 1` do in a play?**
   > It runs the play on one host at a time (rolling update) instead of all hosts simultaneously.

4. **How do you run a playbook without making changes?**
   > Use `--check` flag for dry run, optionally with `--diff` to see what would change.

5. **What is the purpose of `pre_tasks` and `post_tasks`?**
   > `pre_tasks` run before roles (e.g., update cache). `post_tasks` run after everything (e.g., verify deployment).

6. **How do you start a playbook from a specific task?**
   > Use `--start-at-task "Task Name"` to skip all tasks before the named one.

---

[← Previous: Inventory Plugins](../03-Inventory-Management/06-inventory-plugins.md) | [Next: Plays and Tasks →](02-plays-and-tasks.md)
