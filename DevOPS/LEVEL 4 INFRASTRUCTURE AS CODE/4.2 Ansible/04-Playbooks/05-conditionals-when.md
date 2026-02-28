# Chapter 4.5: Conditionals (when)

[← Previous: Variables in Playbooks](04-variables-in-playbooks.md) | [Next: Loops →](06-loops.md)

---

## Overview

The `when` keyword lets you conditionally run tasks based on variables, facts, registered results, or any expression that evaluates to `true` or `false`. Conditions use **raw Jinja2 expressions** — no `{{ }}` curly braces needed.

---

## Basic when Syntax

```yaml
tasks:
  # Simple boolean
  - name: Install nginx
    apt:
      name: nginx
      state: present
    when: install_nginx          # Variable is truthy

  # Comparison
  - name: Only on Debian
    apt:
      name: nginx
    when: ansible_os_family == "Debian"

  # Negation
  - name: Skip on CentOS
    debug:
      msg: "Not CentOS"
    when: ansible_distribution != "CentOS"

  # Numeric comparison
  - name: Warn if low memory
    debug:
      msg: "Low memory!"
    when: ansible_memtotal_mb < 2048
```

> **Important:** Do NOT use `{{ }}` in `when:` — it's already a Jinja2 context.

```yaml
# ✅ Correct
when: ansible_os_family == "Debian"

# ❌ Wrong (works but gives a deprecation warning)
when: "{{ ansible_os_family }} == 'Debian'"
```

---

## Common Condition Patterns

```
  CONDITION PATTERNS
  ══════════════════════════════════════════════════════════

  ┌──────────────────────────────────────────────────────┐
  │ Pattern               │ Example                      │
  ├───────────────────────┼──────────────────────────────┤
  │ Equality              │ var == "value"               │
  │ Inequality            │ var != "value"               │
  │ Greater/Less          │ var > 10 / var < 5           │
  │ Boolean true          │ when: my_bool                │
  │ Boolean false         │ when: not my_bool            │
  │ Defined               │ when: var is defined         │
  │ Undefined             │ when: var is undefined       │
  │ In list               │ when: var in ['a','b']       │
  │ Not in list           │ when: var not in ['a','b']   │
  │ String contains       │ when: "'text' in var"        │
  │ String match          │ when: var is match("pat.*")  │
  │ String search         │ when: var is search("text")  │
  │ Result success        │ when: result is success      │
  │ Result failed         │ when: result is failed       │
  │ Result changed        │ when: result is changed      │
  │ Result skipped        │ when: result is skipped      │
  │ File exists           │ when: stat_result.stat.exists│
  └───────────────────────┴──────────────────────────────┘
```

---

## Multiple Conditions

### AND (all must be true)

```yaml
# List format — implicit AND
- name: Install on Debian with enough memory
  apt:
    name: elasticsearch
  when:
    - ansible_os_family == "Debian"
    - ansible_memtotal_mb >= 4096
    - install_elasticsearch | default(true)

# Inline AND
- name: Same as above
  apt:
    name: elasticsearch
  when: ansible_os_family == "Debian" and ansible_memtotal_mb >= 4096
```

### OR (any must be true)

```yaml
- name: Install on Debian or Ubuntu
  apt:
    name: nginx
  when: ansible_distribution == "Debian" or ansible_distribution == "Ubuntu"

# Cleaner with 'in'
- name: Same but cleaner
  apt:
    name: nginx
  when: ansible_distribution in ["Debian", "Ubuntu"]
```

### Complex Logic

```yaml
- name: Complex condition
  debug:
    msg: "Meets all criteria"
  when: >
    (ansible_os_family == "Debian" or ansible_os_family == "RedHat")
    and
    ansible_memtotal_mb >= 2048
    and
    (env == "production" or force_install | default(false))
```

---

## Conditionals with Facts

```yaml
---
- name: OS-specific configuration
  hosts: all
  become: yes
  tasks:
    # Package manager based on OS
    - name: Install nginx (Debian/Ubuntu)
      apt:
        name: nginx
        state: present
      when: ansible_os_family == "Debian"

    - name: Install nginx (RedHat/CentOS)
      yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"

    # Version-specific tasks
    - name: Configure for Ubuntu 22.04+
      template:
        src: modern.conf.j2
        dest: /etc/app/config.conf
      when:
        - ansible_distribution == "Ubuntu"
        - ansible_distribution_major_version | int >= 22

    # Architecture-specific
    - name: Install 64-bit binary
      copy:
        src: app-amd64
        dest: /usr/local/bin/app
      when: ansible_architecture == "x86_64"
```

---

## Conditionals with Registered Variables

```yaml
tasks:
  - name: Check if app is installed
    command: which myapp
    register: app_check
    ignore_errors: yes            # Don't fail if not found
    changed_when: false

  - name: Install app if not found
    package:
      name: myapp
      state: present
    when: app_check.rc != 0       # Return code non-zero = not found

  - name: Check service status
    command: systemctl is-active nginx
    register: nginx_status
    changed_when: false
    failed_when: false            # Don't fail on inactive

  - name: Start nginx if not running
    service:
      name: nginx
      state: started
    when: nginx_status.stdout != "active"

  - name: Check config file
    stat:
      path: /etc/myapp/config.yml
    register: config_stat

  - name: Create default config
    template:
      src: default-config.yml.j2
      dest: /etc/myapp/config.yml
    when: not config_stat.stat.exists
```

---

## Conditionals with Loops

When using `when` with `loop`, the condition is evaluated **per item**:

```yaml
tasks:
  - name: Install only missing packages
    apt:
      name: "{{ item.name }}"
      state: present
    loop:
      - { name: 'nginx', install: true }
      - { name: 'apache2', install: false }
      - { name: 'curl', install: true }
    when: item.install             # Evaluated per item

  - name: Create users if not system users
    user:
      name: "{{ item.name }}"
      shell: "{{ item.shell }}"
    loop:
      - { name: 'alice', shell: '/bin/bash', uid: 1001 }
      - { name: 'bob', shell: '/bin/zsh', uid: 1002 }
      - { name: 'nginx', shell: '/sbin/nologin', uid: 101 }
    when: item.uid >= 1000         # Skip system UIDs
```

---

## Conditional Imports and Includes

```yaml
tasks:
  # Dynamic include based on OS
  - name: Include OS-specific tasks
    include_tasks: "{{ ansible_os_family | lower }}.yml"

  # Conditional include
  - name: Include SSL tasks if enabled
    include_tasks: ssl.yml
    when: enable_ssl | default(false)

  # Conditional role
- name: Configure monitoring
  hosts: all
  roles:
    - role: prometheus-exporter
      when: monitoring_enabled | default(true)
    - role: custom-checks
      when: custom_monitoring | default(false)
```

---

## Conditional with Block

Apply a condition to multiple tasks at once:

```yaml
tasks:
  - block:
      - name: Install PostgreSQL
        apt:
          name: postgresql
          state: present

      - name: Start PostgreSQL
        service:
          name: postgresql
          state: started
          enabled: yes

      - name: Create database
        postgresql_db:
          name: "{{ db_name }}"
    when: database_type == "postgresql"     # Applies to ALL tasks in block

  - block:
      - name: Install MySQL
        apt:
          name: mysql-server
          state: present

      - name: Start MySQL
        service:
          name: mysql
          state: started
    when: database_type == "mysql"
```

---

## Assertion (assert module)

For strict validation where you want to **fail** if conditions aren't met:

```yaml
tasks:
  - name: Validate deployment prerequisites
    assert:
      that:
        - ansible_memtotal_mb >= 2048
        - ansible_processor_vcpus >= 2
        - disk_space_gb | int >= 20
      fail_msg: "Host does not meet minimum requirements"
      success_msg: "All prerequisites met"

  - name: Validate variables are set
    assert:
      that:
        - app_version is defined
        - app_version | length > 0
        - env in ['staging', 'production']
      fail_msg: "Required variables missing or invalid"
```

---

## Real-World Scenario: Conditional Deployment

```yaml
---
- name: Smart deployment
  hosts: appservers
  become: yes
  vars:
    min_disk_gb: 5
  tasks:
    # Pre-flight checks
    - name: Get disk space
      shell: df -BG / | tail -1 | awk '{print $4}' | tr -d 'G'
      register: free_disk
      changed_when: false

    - name: Check available disk
      assert:
        that: free_disk.stdout | int >= min_disk_gb
        fail_msg: "Only {{ free_disk.stdout }}GB free (need {{ min_disk_gb }}GB)"

    - name: Check current version
      command: cat /opt/app/VERSION
      register: current_version
      changed_when: false
      failed_when: false

    # Skip if already at target version
    - name: Deploy new version
      block:
        - name: Stop application
          service:
            name: myapp
            state: stopped

        - name: Deploy code
          unarchive:
            src: "releases/app-{{ target_version }}.tar.gz"
            dest: /opt/app/

        - name: Run database migration
          command: /opt/app/migrate.sh
          when: run_migrations | default(true)

        - name: Start application
          service:
            name: myapp
            state: started
      when: current_version.stdout | default('0') != target_version

    - name: Already at target version
      debug:
        msg: "Already running {{ target_version }}, skipping deployment"
      when: current_version.stdout | default('0') == target_version
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| "conditional check failed" | Ensure variable is defined; use `default()` filter |
| Using `{{ }}` in when clause | Remove curlies; `when:` is already Jinja2 context |
| String comparison fails | Check for trailing whitespace; use `trim` filter |
| Boolean comparison unexpected | Use `| bool` filter to ensure proper boolean conversion |
| Condition skips unexpectedly | Add `debug` task before to print the variable value |
| AND vs OR confusion | List items = AND; use `or` keyword for OR logic |

---

## Summary Table

| Feature | Example |
|---------|---------|
| Basic condition | `when: var == "value"` |
| AND (list) | `when:\n  - cond1\n  - cond2` |
| OR | `when: cond1 or cond2` |
| Negation | `when: not my_bool` |
| Defined check | `when: var is defined` |
| In list | `when: var in ['a','b']` |
| Register check | `when: result.rc == 0` |
| Result status | `when: result is success` |
| With loops | Condition evaluated per item |
| Block condition | Single `when` for multiple tasks |
| Assert | Fail if conditions not met |

---

## Quick Revision Questions

1. **Do you use `{{ }}` in a `when` clause?**
   > No. `when:` already evaluates raw Jinja2 expressions. Using `{{ }}` gives a deprecation warning.

2. **How do you write an AND condition with multiple criteria?**
   > Use a YAML list — each item is implicitly AND-ed: `when:\n  - condition1\n  - condition2`

3. **How do you check if a variable is defined before using it?**
   > Use `when: my_var is defined` or provide a default: `my_var | default('fallback')`.

4. **How does `when` interact with loops?**
   > The condition is evaluated for each item in the loop independently. Items that don't match are skipped.

5. **What is the difference between `when` and `assert`?**
   > `when` skips a task; `assert` fails the play entirely if conditions aren't met. Use `assert` for mandatory prerequisites.

6. **How do you apply a condition to multiple tasks at once?**
   > Wrap the tasks in a `block:` and put the `when:` on the block.

---

[← Previous: Variables in Playbooks](04-variables-in-playbooks.md) | [Next: Loops →](06-loops.md)
