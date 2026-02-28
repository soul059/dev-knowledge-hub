# Chapter 4.7: Blocks and Error Handling

[← Previous: Loops](06-loops.md) | [Next: Tags →](08-tags.md)

---

## Overview

**Blocks** group multiple tasks together so you can apply common directives (like `when`, `become`, `tags`) to all of them at once. Combined with `rescue` and `always`, blocks provide **try/catch/finally** style error handling in Ansible.

---

## Basic Blocks

```yaml
tasks:
  # Without block — repeating directives
  - name: Install nginx
    apt: name=nginx state=present
    become: yes
    when: ansible_os_family == "Debian"

  - name: Start nginx
    service: name=nginx state=started
    become: yes
    when: ansible_os_family == "Debian"

  # With block — DRY
  - block:
      - name: Install nginx
        apt:
          name: nginx
          state: present

      - name: Start nginx
        service:
          name: nginx
          state: started
    become: yes                              # Applies to all tasks
    when: ansible_os_family == "Debian"      # Applies to all tasks
```

---

## Error Handling: block / rescue / always

```
  BLOCK ERROR HANDLING (Try / Catch / Finally)
  ══════════════════════════════════════════════════════════

  ┌─────────────────────────────────────────────┐
  │  block:           ◄── TRY                   │
  │    - task A                                  │
  │    - task B  ← FAILS!                        │
  │    - task C  ← SKIPPED (block stops)         │
  └──────────────────┬──────────────────────────┘
                     │ failure
                     ▼
  ┌─────────────────────────────────────────────┐
  │  rescue:          ◄── CATCH                 │
  │    - recovery task 1                        │
  │    - recovery task 2                        │
  └──────────────────┬──────────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────────────┐
  │  always:          ◄── FINALLY               │
  │    - cleanup task (ALWAYS runs)             │
  └─────────────────────────────────────────────┘
```

### Example

```yaml
tasks:
  - name: Deploy application with error handling
    block:
      - name: Pull latest code
        git:
          repo: https://github.com/org/app.git
          dest: /opt/app
          version: "{{ app_version }}"

      - name: Run database migrations
        command: /opt/app/migrate.sh

      - name: Restart application
        service:
          name: myapp
          state: restarted

    rescue:
      - name: Rollback to previous version
        command: /opt/app/rollback.sh

      - name: Restart previous version
        service:
          name: myapp
          state: restarted

      - name: Send failure alert
        uri:
          url: "{{ slack_webhook }}"
          method: POST
          body_format: json
          body:
            text: "Deployment FAILED on {{ inventory_hostname }}. Rolled back."

    always:
      - name: Re-enable in load balancer
        uri:
          url: "http://{{ lb_host }}/api/enable/{{ inventory_hostname }}"
          method: POST
        delegate_to: localhost

      - name: Log deployment result
        lineinfile:
          path: /var/log/deployments.log
          line: "{{ ansible_date_time.iso8601 }} - {{ app_version }} - {{ 'FAILED' if ansible_failed_task is defined else 'SUCCESS' }}"
          create: yes
```

---

## Rescue Block Details

Important rescue behaviors:
- Rescue runs **only if** a task in `block` fails
- If rescue succeeds, the play **continues** (failure is recovered)
- If rescue also fails, the host is marked as failed
- `always` runs regardless of block or rescue outcome

### Special Variables in Rescue

```yaml
rescue:
  - name: Show what failed
    debug:
      msg: |
        Failed task: {{ ansible_failed_task.name }}
        Failed result: {{ ansible_failed_result }}
```

| Variable | Description |
|----------|-------------|
| `ansible_failed_task` | The task object that failed |
| `ansible_failed_task.name` | Name of the failed task |
| `ansible_failed_result` | The result from the failed task |

---

## Nested Blocks

```yaml
tasks:
  - block:
      - name: Main operation
        block:
          - name: Step 1
            command: /opt/step1.sh

          - name: Step 2 (might fail)
            command: /opt/step2.sh
        rescue:
          - name: Handle step failure
            debug:
              msg: "Inner block failed, handling..."

      - name: Continue after inner block
        debug:
          msg: "This runs even if inner block was rescued"

    rescue:
      - name: Handle outer failure
        debug:
          msg: "Something unexpected failed"

    always:
      - name: Final cleanup
        debug:
          msg: "Always runs"
```

---

## Error Handling Directives

### ignore_errors

```yaml
tasks:
  - name: This might fail
    command: /opt/risky-command.sh
    ignore_errors: yes                # Continue even if this fails

  - name: This still runs
    debug:
      msg: "Previous task failure was ignored"
```

### ignore_unreachable

```yaml
tasks:
  - name: Try to reach host
    ping:
    ignore_unreachable: yes          # Don't mark host as unreachable

  - name: Try alternative connection
    ping:
      data: "still here"
```

### any_errors_fatal

```yaml
- name: Critical deployment
  hosts: webservers
  any_errors_fatal: yes              # ANY host failure stops ENTIRE play
  tasks:
    - name: Health check
      uri:
        url: http://localhost/health
        status_code: 200
    # If ANY host fails, ALL hosts stop immediately
```

### max_fail_percentage

```yaml
- name: Rolling update
  hosts: webservers
  serial: 5
  max_fail_percentage: 30           # Fail play if >30% of batch fails
  tasks:
    - name: Deploy
      copy:
        src: app.tar.gz
        dest: /opt/app/
```

---

## fail Module

Explicitly fail a task with a message:

```yaml
tasks:
  - name: Check prerequisites
    command: which docker
    register: docker_check
    changed_when: false
    failed_when: false

  - name: Fail if Docker not installed
    fail:
      msg: "Docker is required but not installed on {{ inventory_hostname }}"
    when: docker_check.rc != 0

  - name: Validate configuration
    fail:
      msg: |
        Invalid configuration detected:
        - env must be 'staging' or 'production'
        - Got: {{ env | default('undefined') }}
    when: env is not defined or env not in ['staging', 'production']
```

---

## Combining Blocks with Other Directives

```yaml
tasks:
  # Block with loop (loop on individual tasks inside, not on block)
  - block:
      - name: Install packages
        apt:
          name: "{{ item }}"
          state: present
        loop:
          - nginx
          - php-fpm

      - name: Configure services
        template:
          src: "{{ item }}.conf.j2"
          dest: "/etc/{{ item }}/{{ item }}.conf"
        loop:
          - nginx
          - php-fpm
    become: yes
    when: ansible_os_family == "Debian"
    tags: [webserver]

  # Block with delegation
  - block:
      - name: Remove from LB
        uri:
          url: "http://lb/disable/{{ inventory_hostname }}"
          method: POST

      - name: Deploy
        copy:
          src: app.tar.gz
          dest: /opt/

    rescue:
      - name: Alert
        debug:
          msg: "Deploy failed for {{ inventory_hostname }}"

    always:
      - name: Re-add to LB
        uri:
          url: "http://lb/enable/{{ inventory_hostname }}"
          method: POST
```

---

## Real-World Scenario: Database Migration with Rollback

```yaml
---
- name: Database migration with safety
  hosts: dbservers
  become: yes
  become_user: postgres
  serial: 1
  vars:
    backup_dir: /var/backups/postgresql
    db_name: production_db

  tasks:
    - name: Database migration
      block:
        # Step 1: Create backup
        - name: Create database backup
          command: >
            pg_dump {{ db_name }}
            -F c
            -f {{ backup_dir }}/{{ db_name }}_{{ ansible_date_time.epoch }}.dump
          register: backup_result

        - name: Store backup filename
          set_fact:
            backup_file: "{{ backup_dir }}/{{ db_name }}_{{ ansible_date_time.epoch }}.dump"

        # Step 2: Run migrations
        - name: Run schema migrations
          command: /opt/app/bin/migrate --env production
          register: migration_result

        # Step 3: Verify
        - name: Verify migration
          command: /opt/app/bin/verify-schema
          register: verify_result
          failed_when: "'PASS' not in verify_result.stdout"

      rescue:
        # Rollback on any failure
        - name: Log migration failure
          debug:
            msg: "Migration failed: {{ ansible_failed_task.name }}"

        - name: Restore from backup
          command: >
            pg_restore
            --dbname={{ db_name }}
            --clean
            {{ backup_file }}
          when: backup_file is defined

        - name: Send failure notification
          uri:
            url: "{{ pagerduty_webhook }}"
            method: POST
            body_format: json
            body:
              event: "DB migration failed on {{ inventory_hostname }}"
              severity: critical
          delegate_to: localhost

        - name: Fail the play after rollback
          fail:
            msg: "Migration failed and was rolled back. Manual review required."

      always:
        - name: Clean old backups (keep 5)
          shell: >
            ls -t {{ backup_dir }}/*.dump | tail -n +6 | xargs rm -f
          changed_when: false

        - name: Log migration attempt
          lineinfile:
            path: /var/log/db-migrations.log
            line: >
              {{ ansible_date_time.iso8601 }}
              | {{ db_name }}
              | {{ 'SUCCESS' if migration_result is defined and migration_result.rc == 0 else 'FAILED' }}
            create: yes
          become_user: root
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Rescue not triggering | Ensure the failing task is inside `block:`, not outside it |
| Rescue hides the real error | Use `ansible_failed_task` and `ansible_failed_result` in rescue |
| `always` not running | `always` should always run; check for syntax errors in the block structure |
| Want to fail after rescue | Add `fail:` task at end of rescue to propagate the failure |
| `ignore_errors` hides important failures | Use `block/rescue` instead for controlled error handling |
| `any_errors_fatal` too aggressive | Use `max_fail_percentage` for rolling deployments |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| `block:` | Group tasks; apply common directives |
| `rescue:` | Tasks to run if block fails (catch) |
| `always:` | Tasks that always run (finally) |
| `ignore_errors` | Continue on task failure |
| `ignore_unreachable` | Continue if host unreachable |
| `any_errors_fatal` | Stop play on any host failure |
| `max_fail_percentage` | Acceptable failure rate |
| `fail` module | Explicitly fail with message |
| `ansible_failed_task` | Task that caused rescue |
| `ansible_failed_result` | Result of the failed task |

---

## Quick Revision Questions

1. **What is the Ansible equivalent of try/catch/finally?**
   > `block` (try), `rescue` (catch), `always` (finally). Rescue runs only on block failure; always runs regardless.

2. **What happens if a `rescue` block also fails?**
   > The host is marked as failed, and the play stops for that host (unless `ignore_errors` is used elsewhere).

3. **Does the play continue after a successful `rescue`?**
   > Yes. If rescue succeeds, the failure is considered recovered and subsequent tasks continue normally.

4. **What is the difference between `ignore_errors` and `block/rescue`?**
   > `ignore_errors` silently ignores failures. `block/rescue` lets you handle failures with recovery logic.

5. **How do you inspect what failed inside a `rescue` block?**
   > Use `ansible_failed_task.name` for the task name and `ansible_failed_result` for the full failure details.

6. **What does `any_errors_fatal: yes` do?**
   > If any host fails a task, the entire play stops for all hosts immediately — not just the failing host.

---

[← Previous: Loops](06-loops.md) | [Next: Tags →](08-tags.md)
