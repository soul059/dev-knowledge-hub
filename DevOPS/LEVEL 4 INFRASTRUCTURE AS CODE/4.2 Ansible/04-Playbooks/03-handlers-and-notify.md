# Chapter 4.3: Handlers and Notify

[← Previous: Plays and Tasks](02-plays-and-tasks.md) | [Next: Variables in Playbooks →](04-variables-in-playbooks.md)

---

## Overview

**Handlers** are special tasks that only run when **notified** by other tasks. They are used to perform actions that should only happen when a change occurs — most commonly restarting or reloading services after configuration changes.

---

## How Handlers Work

```
  HANDLER NOTIFICATION FLOW
  ══════════════════════════════════════════════════════════

  Task: "Update nginx config"
  ┌──────────────────────────┐
  │  copy:                   │
  │    src: nginx.conf       │     ┌─── changed ──────┐
  │    dest: /etc/nginx/     │─────┤                   │
  │  notify: Restart nginx   │     │                   ▼
  └──────────────────────────┘     │    ┌──────────────────────┐
                                   │    │ HANDLER QUEUE        │
  Task: "Update site config"       │    │                      │
  ┌──────────────────────────┐     │    │ • Restart nginx ✓    │
  │  template:               │     │    │                      │
  │    src: site.conf.j2     │─────┘    └──────────┬───────────┘
  │    dest: /etc/nginx/...  │  changed            │
  │  notify: Restart nginx   │                     │
  └──────────────────────────┘                     │
                                                   │
  ═══════════ END OF ALL TASKS ═════════════       │
                                                   ▼
                                    ┌──────────────────────┐
                                    │ EXECUTE HANDLERS     │
                                    │                      │
                                    │ Restart nginx        │
                                    │ (runs ONCE even if   │
                                    │  notified twice)     │
                                    └──────────────────────┘
```

### Key Rules

1. Handlers run **at the end** of all tasks in a play (by default)
2. Handlers run **only once** even if notified multiple times
3. Handlers run **only if** at least one notifying task reports `changed`
4. Handlers run in the **order they are defined**, not the order they are notified
5. If no task is `changed`, the handler **never runs**

---

## Basic Handler Example

```yaml
---
- name: Configure web server
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Copy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx           # Notify handler

    - name: Copy site config
      template:
        src: site.conf.j2
        dest: /etc/nginx/sites-available/default
      notify:
        - Reload nginx                # Can notify multiple handlers
        - Clear cache

  handlers:
    - name: Restart nginx             # Handler name MUST match
      service:
        name: nginx
        state: restarted

    - name: Reload nginx
      service:
        name: nginx
        state: reloaded

    - name: Clear cache
      file:
        path: /var/cache/nginx
        state: absent
```

---

## Handler Execution Timing

```
  DEFAULT HANDLER TIMING
  ══════════════════════════════════════════════════════════

  ┌──────────────────────────────────────────────────┐
  │ PLAY                                             │
  │                                                  │
  │  pre_tasks:                                      │
  │    ├── task 1                                    │
  │    └── task 2 → notify: Handler A                │
  │  ►►►  Flush handlers (Handler A runs here) ◄◄◄  │
  │                                                  │
  │  roles:                                          │
  │    └── role tasks → notify: Handler B            │
  │  ►►►  Flush handlers (Handler B runs here) ◄◄◄  │
  │                                                  │
  │  tasks:                                          │
  │    ├── task 3 → notify: Handler C                │
  │    └── task 4 → notify: Handler D                │
  │  ►►►  Flush handlers (C and D run here)  ◄◄◄    │
  │                                                  │
  │  post_tasks:                                     │
  │    └── task 5                                    │
  │  ►►►  Flush handlers                     ◄◄◄    │
  └──────────────────────────────────────────────────┘
```

Handlers flush automatically after each section (`pre_tasks`, `roles`, `tasks`, `post_tasks`).

---

## Forcing Handlers Mid-Play

Use `meta: flush_handlers` to trigger handlers immediately:

```yaml
---
- name: Configure and verify
  hosts: webservers
  become: yes
  tasks:
    - name: Update nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx

    # Force handler to run NOW
    - meta: flush_handlers

    - name: Verify nginx is running
      uri:
        url: http://localhost
        status_code: 200
      retries: 5
      delay: 2

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

---

## Listen Directive

Multiple handlers can **listen** to the same notification topic:

```yaml
---
- name: Configure application
  hosts: appservers
  become: yes
  tasks:
    - name: Deploy new application code
      copy:
        src: app/
        dest: /opt/myapp/
      notify: "deploy complete"      # Notify a TOPIC, not a handler name

  handlers:
    - name: Restart application
      service:
        name: myapp
        state: restarted
      listen: "deploy complete"       # Listens to the topic

    - name: Clear application cache
      file:
        path: /tmp/myapp-cache
        state: absent
      listen: "deploy complete"       # Also listens to same topic

    - name: Send deployment notification
      uri:
        url: "{{ slack_webhook }}"
        method: POST
        body_format: json
        body:
          text: "App redeployed on {{ inventory_hostname }}"
      listen: "deploy complete"       # All three run when notified
```

```
  LISTEN vs NOTIFY
  ══════════════════════════════════════════════════════════

  Task ──notify──► "deploy complete"
                         │
                         ├──► Handler 1 (listen: "deploy complete")
                         ├──► Handler 2 (listen: "deploy complete")
                         └──► Handler 3 (listen: "deploy complete")

  Without listen:
  Task ──notify──► "Restart app"  ──► Only ONE handler
```

---

## Handler Chaining

Handlers can notify other handlers:

```yaml
handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
    notify: Verify nginx        # Chain to another handler

  - name: Verify nginx
    uri:
      url: http://localhost
      status_code: 200
    register: verify_result
    retries: 3
    delay: 2
```

---

## Handlers in Roles

```
  HANDLER LOCATION IN ROLES
  ══════════════════════════════════════════════════════════

  roles/nginx/
  ├── tasks/
  │   └── main.yml         ◄── notify: Restart nginx
  ├── handlers/
  │   └── main.yml         ◄── name: Restart nginx
  ├── templates/
  │   └── nginx.conf.j2
  └── defaults/
      └── main.yml
```

```yaml
# roles/nginx/tasks/main.yml
- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx

# roles/nginx/handlers/main.yml
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

---

## Handlers with Conditions and Variables

```yaml
handlers:
  # Handler with a condition
  - name: Restart apache
    service:
      name: "{{ apache_service }}"
      state: restarted
    when: ansible_os_family == "Debian"

  # Handler using a variable for service name
  - name: Restart webserver
    service:
      name: "{{ web_service_name }}"
      state: restarted
```

---

## Force Handlers on Failure

By default, if a task fails, pending handlers **do not run**. Override with:

```yaml
# Command line
ansible-playbook site.yml --force-handlers

# Or in the play
- name: Configure servers
  hosts: all
  force_handlers: yes     # Handlers run even if tasks fail
  tasks:
    - name: Update config
      template:
        src: app.conf.j2
        dest: /etc/app/app.conf
      notify: Restart app

    - name: This might fail
      command: /opt/app/migrate.sh

  handlers:
    - name: Restart app
      service:
        name: myapp
        state: restarted
      # This handler WILL run even if migrate.sh fails
```

---

## Real-World Scenario: SSL Certificate Renewal

```yaml
---
- name: Renew SSL certificates
  hosts: webservers
  become: yes
  force_handlers: yes
  tasks:
    - name: Copy new SSL certificate
      copy:
        src: "certs/{{ domain }}.crt"
        dest: /etc/ssl/certs/{{ domain }}.crt
        mode: '0644'
      notify: Reload web services

    - name: Copy new SSL key
      copy:
        src: "certs/{{ domain }}.key"
        dest: /etc/ssl/private/{{ domain }}.key
        mode: '0600'
      notify: Reload web services

    - name: Update certificate bundle
      copy:
        src: "certs/ca-bundle.crt"
        dest: /etc/ssl/certs/ca-bundle.crt
      notify: Reload web services

    - meta: flush_handlers   # Reload NOW before testing

    - name: Verify SSL certificate
      uri:
        url: "https://{{ domain }}"
        validate_certs: yes
        status_code: 200
      delegate_to: localhost

  handlers:
    - name: Reload web services
      service:
        name: nginx
        state: reloaded

    - name: Notify ops team
      uri:
        url: "{{ slack_webhook }}"
        method: POST
        body_format: json
        body:
          text: "SSL renewed on {{ inventory_hostname }}"
      listen: "Reload web services"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Handler never runs | Verify the notifying task actually reports `changed` (not `ok`) |
| "Handler not found" error | Ensure `notify:` string exactly matches handler `name:` (case-sensitive) |
| Handler runs too late | Use `meta: flush_handlers` to trigger handlers immediately |
| Handler skipped after task failure | Use `force_handlers: yes` in the play or `--force-handlers` on CLI |
| Handler runs every time | Ensure the notifying task is truly idempotent (uses `state: present`, `template:`, etc.) |
| Multiple services need restart | Use `listen:` directive or notify a list of handlers |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| Handler | A task that runs only when notified |
| `notify:` | Declare handler(s) to trigger on change |
| `listen:` | Subscribe handler to a topic name |
| `meta: flush_handlers` | Run pending handlers immediately |
| `force_handlers: yes` | Run handlers even if play fails |
| Handler ordering | Handlers run in definition order |
| Handler dedup | Handlers run once even if notified multiple times |
| Handler chaining | Handlers can notify other handlers |
| `--force-handlers` | CLI flag to always run notified handlers |

---

## Quick Revision Questions

1. **When do handlers execute by default?**
   > Handlers execute at the end of each section (pre_tasks, roles, tasks, post_tasks) within a play — not after each individual task.

2. **What happens if a handler is notified multiple times?**
   > It runs only once, after all notifying tasks complete. Duplicate notifications are de-duplicated.

3. **How do you trigger handlers immediately instead of at the end?**
   > Use `meta: flush_handlers` as a task where you want handlers to execute.

4. **What is the `listen` directive used for?**
   > It allows multiple handlers to subscribe to the same notification topic, so one `notify` can trigger several handlers.

5. **What happens to handlers if a task fails?**
   > By default, pending handlers are skipped. Use `force_handlers: yes` or `--force-handlers` to override this.

6. **In what order do handlers run?**
   > Handlers run in the order they are defined in the `handlers:` section, regardless of the order they were notified.

---

[← Previous: Plays and Tasks](02-plays-and-tasks.md) | [Next: Variables in Playbooks →](04-variables-in-playbooks.md)
