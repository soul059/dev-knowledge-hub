# Chapter 4.6: Loops

[← Previous: Conditionals (when)](05-conditionals-when.md) | [Next: Blocks and Error Handling →](07-blocks-and-error-handling.md)

---

## Overview

Loops let you repeat a task multiple times with different data. Instead of writing separate tasks for each user, package, or file, you use a single task with a loop. Ansible uses `loop` (recommended) and the older `with_*` syntax.

---

## Basic Loop

```yaml
tasks:
  # Modern syntax — use 'loop'
  - name: Install packages
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - nginx
      - curl
      - git
      - htop

  # Even simpler — pass list directly to module
  - name: Install packages (no loop needed)
    apt:
      name:
        - nginx
        - curl
        - git
        - htop
      state: present    # More efficient — single apt transaction
```

> **Tip:** Many modules like `apt`, `yum`, `pip` accept lists directly in `name:`. This is faster than looping because it runs a single transaction.

---

## Loop Execution Flow

```
  LOOP EXECUTION
  ══════════════════════════════════════════════════════════

  Task: "Create users"
  loop: [alice, bob, charlie]

  ┌───────────────────────────┐
  │ Iteration 1: item = alice │
  │  → user: name=alice       │──► ok / changed
  └───────────────────────────┘
  ┌───────────────────────────┐
  │ Iteration 2: item = bob   │
  │  → user: name=bob         │──► ok / changed
  └───────────────────────────┘
  ┌───────────────────────────┐
  │ Iteration 3: item = charlie│
  │  → user: name=charlie     │──► ok / changed
  └───────────────────────────┘

  Register result → results: [
    {item: alice, changed: true},
    {item: bob, changed: false},
    {item: charlie, changed: true}
  ]
```

---

## Loop with Dictionaries

```yaml
tasks:
  - name: Create users with details
    user:
      name: "{{ item.name }}"
      uid: "{{ item.uid }}"
      groups: "{{ item.groups }}"
      shell: "{{ item.shell | default('/bin/bash') }}"
    loop:
      - { name: alice, uid: 1001, groups: 'admin,web' }
      - { name: bob, uid: 1002, groups: 'web' }
      - { name: charlie, uid: 1003, groups: 'db,web' }

  - name: Copy multiple config files
    template:
      src: "{{ item.src }}"
      dest: "{{ item.dest }}"
      mode: "{{ item.mode | default('0644') }}"
    loop:
      - { src: 'nginx.conf.j2', dest: '/etc/nginx/nginx.conf' }
      - { src: 'site.conf.j2', dest: '/etc/nginx/sites-available/default' }
      - { src: 'ssl.conf.j2', dest: '/etc/nginx/conf.d/ssl.conf' }
    notify: Reload nginx
```

---

## Loop with Variables

```yaml
vars:
  packages:
    - nginx
    - postgresql
    - redis-server
  
  users:
    - name: deploy
      uid: 1100
      groups: web
    - name: monitor
      uid: 1101
      groups: monitoring

tasks:
  - name: Install packages from variable
    apt:
      name: "{{ item }}"
      state: present
    loop: "{{ packages }}"

  - name: Create users from variable
    user:
      name: "{{ item.name }}"
      uid: "{{ item.uid }}"
      groups: "{{ item.groups }}"
    loop: "{{ users }}"
```

---

## Loop Control

The `loop_control` directive customizes loop behavior:

```yaml
tasks:
  # Change the loop variable name
  - name: Create directories
    file:
      path: "/opt/{{ dir_item }}"
      state: directory
    loop:
      - app
      - logs
      - config
    loop_control:
      loop_var: dir_item       # Use 'dir_item' instead of 'item'

  # Add label to output (shorter log lines)
  - name: Configure services
    service:
      name: "{{ item.name }}"
      state: "{{ item.state }}"
    loop:
      - { name: nginx, state: started, port: 80, config: '/etc/nginx/nginx.conf' }
      - { name: redis, state: started, port: 6379, config: '/etc/redis/redis.conf' }
    loop_control:
      label: "{{ item.name }}"  # Only show name in output, not full dict

  # Add index
  - name: Create numbered files
    copy:
      content: "Server {{ ansible_loop.index }}"
      dest: "/tmp/server-{{ ansible_loop.index0 }}.txt"
    loop:
      - web01
      - web02
      - web03
    loop_control:
      extended: yes            # Enable ansible_loop variable

  # Add pause between iterations
  - name: Rolling restart
    service:
      name: myapp
      state: restarted
    loop: "{{ groups['appservers'] }}"
    loop_control:
      pause: 30                # Wait 30 seconds between each
```

### Extended Loop Variables

When `loop_control.extended: yes`:

```
  ┌────────────────────────────┬────────────────────────────┐
  │ Variable                   │ Description                │
  ├────────────────────────────┼────────────────────────────┤
  │ ansible_loop.index         │ Current iteration (1-based)│
  │ ansible_loop.index0        │ Current iteration (0-based)│
  │ ansible_loop.first         │ True on first iteration    │
  │ ansible_loop.last          │ True on last iteration     │
  │ ansible_loop.length        │ Total number of items      │
  │ ansible_loop.revindex      │ Reverse (1-based)          │
  │ ansible_loop.revindex0     │ Reverse (0-based)          │
  │ ansible_loop.previtem      │ Previous item              │
  │ ansible_loop.nextitem      │ Next item                  │
  └────────────────────────────┴────────────────────────────┘
```

---

## Loops with Filters

```yaml
tasks:
  # Flatten nested list
  - name: Install all packages
    apt:
      name: "{{ item }}"
    loop: "{{ [web_packages, db_packages, common_packages] | flatten }}"

  # Unique items only
  - name: Create unique directories
    file:
      path: "{{ item }}"
      state: directory
    loop: "{{ all_paths | unique }}"

  # Product (cross join)
  - name: Grant permissions
    mysql_user:
      name: "{{ item.0 }}"
      priv: "{{ item.1 }}.*:ALL"
    loop: "{{ users | product(databases) | list }}"
    # users: [alice, bob]
    # databases: [app_db, log_db]
    # Result: (alice,app_db), (alice,log_db), (bob,app_db), (bob,log_db)
```

---

## Loop with dict2items

```yaml
vars:
  user_shells:
    alice: /bin/bash
    bob: /bin/zsh
    charlie: /bin/fish

tasks:
  - name: Set user shells
    user:
      name: "{{ item.key }}"
      shell: "{{ item.value }}"
    loop: "{{ user_shells | dict2items }}"
    # Each item has .key and .value
```

---

## Loop with Conditionals

```yaml
tasks:
  - name: Install only required packages
    apt:
      name: "{{ item.name }}"
      state: present
    loop:
      - { name: nginx, required: true }
      - { name: apache2, required: false }
      - { name: curl, required: true }
      - { name: wget, required: false }
    when: item.required            # Per-item condition

  - name: Configure running services only
    template:
      src: "{{ item }}.conf.j2"
      dest: "/etc/{{ item }}/{{ item }}.conf"
    loop: "{{ services }}"
    when: item in running_services
```

---

## Registering Loop Results

```yaml
tasks:
  - name: Check service status
    command: "systemctl is-active {{ item }}"
    loop:
      - nginx
      - postgresql
      - redis
    register: service_status
    changed_when: false
    failed_when: false

  - name: Show stopped services
    debug:
      msg: "{{ item.item }} is {{ item.stdout }}"
    loop: "{{ service_status.results }}"
    when: item.stdout != "active"
```

```
  REGISTERED LOOP RESULT STRUCTURE
  ══════════════════════════════════════════════════════════

  service_status:
  ├── changed: false
  ├── msg: "All items completed"
  └── results:                    ◄── List of per-item results
      ├── [0]:
      │   ├── item: nginx         ◄── Original loop item
      │   ├── stdout: "active"
      │   ├── rc: 0
      │   └── changed: false
      ├── [1]:
      │   ├── item: postgresql
      │   ├── stdout: "inactive"
      │   ├── rc: 3
      │   └── changed: false
      └── [2]:
          ├── item: redis
          ├── stdout: "active"
          ├── rc: 0
          └── changed: false
```

---

## Legacy with_* Syntax (Still Valid)

```yaml
tasks:
  # with_items → loop
  - name: Install packages (legacy)
    apt:
      name: "{{ item }}"
    with_items:
      - nginx
      - curl

  # with_dict → loop + dict2items
  - name: Set sysctls (legacy)
    sysctl:
      name: "{{ item.key }}"
      value: "{{ item.value }}"
    with_dict:
      net.ipv4.ip_forward: 1
      net.core.somaxconn: 65535

  # with_fileglob → loop + fileglob lookup
  - name: Copy all configs (legacy)
    copy:
      src: "{{ item }}"
      dest: /etc/myapp/conf.d/
    with_fileglob:
      - "files/conf.d/*.conf"

  # with_sequence → loop + range
  - name: Create numbered dirs (legacy)
    file:
      path: "/tmp/dir-{{ item }}"
      state: directory
    with_sequence: start=1 end=5

  # Modern equivalent
  - name: Create numbered dirs (modern)
    file:
      path: "/tmp/dir-{{ item }}"
      state: directory
    loop: "{{ range(1, 6) | list }}"
```

### Migration Reference

```
  ┌─────────────────────┬──────────────────────────────────┐
  │ Legacy              │ Modern Equivalent                │
  ├─────────────────────┼──────────────────────────────────┤
  │ with_items          │ loop                             │
  │ with_list           │ loop                             │
  │ with_dict           │ loop + dict2items filter         │
  │ with_fileglob       │ loop + fileglob lookup           │
  │ with_sequence       │ loop + range filter              │
  │ with_subelements    │ loop + subelements filter        │
  │ with_nested         │ loop + product filter            │
  │ with_random_choice  │ random filter                    │
  │ with_first_found   │ lookup('first_found', list)      │
  │ with_together       │ loop + zip filter                │
  │ with_indexed_items  │ loop + flatten + loop_control    │
  └─────────────────────┴──────────────────────────────────┘
```

---

## Until (Retry) Loops

Retry a task until a condition is met:

```yaml
tasks:
  - name: Wait for application to start
    uri:
      url: http://localhost:8080/health
      status_code: 200
    register: health_check
    until: health_check.status == 200
    retries: 30           # Number of attempts
    delay: 10             # Seconds between retries
    # Total max wait: 30 * 10 = 300 seconds

  - name: Wait for file to appear
    stat:
      path: /tmp/ready.flag
    register: flag
    until: flag.stat.exists
    retries: 60
    delay: 5
```

---

## Real-World Scenario: Multi-App Server Setup

```yaml
---
- name: Configure application servers
  hosts: appservers
  become: yes
  vars:
    applications:
      - name: frontend
        port: 3000
        repo: https://github.com/org/frontend.git
        version: v2.1.0
        env: production
      - name: api
        port: 8080
        repo: https://github.com/org/api.git
        version: v1.5.3
        env: production
      - name: worker
        port: 9090
        repo: https://github.com/org/worker.git
        version: v1.2.0
        env: production

  tasks:
    - name: Create app directories
      file:
        path: "/opt/{{ item.name }}"
        state: directory
        owner: appuser
        group: appgroup
      loop: "{{ applications }}"
      loop_control:
        label: "{{ item.name }}"

    - name: Clone/update application repos
      git:
        repo: "{{ item.repo }}"
        dest: "/opt/{{ item.name }}"
        version: "{{ item.version }}"
      loop: "{{ applications }}"
      loop_control:
        label: "{{ item.name }} ({{ item.version }})"
      notify: Restart apps

    - name: Deploy systemd service files
      template:
        src: app-service.j2
        dest: "/etc/systemd/system/{{ item.name }}.service"
      loop: "{{ applications }}"
      loop_control:
        label: "{{ item.name }}"
      notify:
        - Reload systemd
        - Restart apps

    - name: Ensure all apps are running
      service:
        name: "{{ item.name }}"
        state: started
        enabled: yes
      loop: "{{ applications }}"
      loop_control:
        label: "{{ item.name }}"

    - name: Verify all apps respond
      uri:
        url: "http://localhost:{{ item.port }}/health"
        status_code: 200
      loop: "{{ applications }}"
      loop_control:
        label: "{{ item.name }}:{{ item.port }}"
      retries: 5
      delay: 3
      register: health_checks

  handlers:
    - name: Reload systemd
      command: systemctl daemon-reload

    - name: Restart apps
      service:
        name: "{{ item.name }}"
        state: restarted
      loop: "{{ applications }}"
      loop_control:
        label: "{{ item.name }}"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `item` is undefined | Ensure `loop:` is at task level, not inside module args |
| Nested loop conflict | Use `loop_control.loop_var` to rename `item` in inner or outer loop |
| Too much output with dicts | Use `loop_control.label` to show only key fields |
| Loop over dict keys fails | Use `dict2items` filter: `loop: "{{ mydict \| dict2items }}"` |
| Need index in loop | Use `loop_control.extended: yes` and `ansible_loop.index` |
| `with_items` flattens unexpectedly | Switch to `loop:` which does NOT auto-flatten |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| `loop:` | Modern loop syntax (recommended) |
| `item` | Default loop variable |
| `loop_control.loop_var` | Rename loop variable |
| `loop_control.label` | Customize output label |
| `loop_control.extended` | Enable `ansible_loop` metadata |
| `loop_control.pause` | Pause between iterations |
| `dict2items` | Convert dict to list of key/value |
| `flatten` | Flatten nested lists |
| `product` | Cross-product of lists |
| `until` / `retries` / `delay` | Retry loop |
| `register` with loops | Results stored in `.results` list |

---

## Quick Revision Questions

1. **What keyword is recommended for loops in modern Ansible?**
   > `loop:` — it replaces the deprecated `with_*` syntax with clearer semantics.

2. **How do you access each element in a loop?**
   > Using the `item` variable (or a custom name set via `loop_control.loop_var`).

3. **How do you loop over a dictionary?**
   > Convert it first: `loop: "{{ my_dict | dict2items }}"`, then access `item.key` and `item.value`.

4. **What does `loop_control.label` do?**
   > It controls what's shown in the task output per iteration (e.g., show just the name instead of the entire dictionary).

5. **How do you access loop results from a `register` with a loop?**
   > The registered variable has a `.results` list; each element has `.item` (original loop item) plus the task return values.

6. **How does `until` differ from `loop`?**
   > `until` retries the same task with the same parameters until a condition is true. `loop` iterates over a list of items with different data each time.

---

[← Previous: Conditionals (when)](05-conditionals-when.md) | [Next: Blocks and Error Handling →](07-blocks-and-error-handling.md)
