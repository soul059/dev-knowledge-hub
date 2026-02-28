# Chapter 8.2: Service Modules

[← Previous: Package Modules](01-package-modules.md) | [Next: Command Modules →](03-command-modules.md)

---

## Overview

Service modules manage system services (daemons) — starting, stopping, restarting, enabling on boot, and checking status. Ansible provides `service` (generic), `systemd` (systemd-specific), and `sysvinit` modules.

---

## service Module (Generic)

```yaml
# Start a service
- name: Start Nginx
  service:
    name: nginx
    state: started

# Stop a service
- name: Stop Nginx
  service:
    name: nginx
    state: stopped

# Restart a service
- name: Restart Nginx
  service:
    name: nginx
    state: restarted

# Reload a service
- name: Reload Nginx config
  service:
    name: nginx
    state: reloaded

# Enable on boot
- name: Enable Nginx on boot
  service:
    name: nginx
    enabled: yes

# Start and enable
- name: Start and enable Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

---

## Service States

```
  SERVICE STATE TRANSITIONS
  ══════════════════════════════════════════════════════════

  ┌─────────┐   start    ┌─────────┐
  │ Stopped │───────────▶│ Running │
  └─────────┘            └─────────┘
       ▲                      │
       │    stop              │
       └──────────────────────┘
                              │
                    restart ───┤──▶ Stop → Start
                    reload  ───┘──▶ Re-read config (no downtime)
```

| State | Behaviour |
|-------|-----------|
| `started` | Start if stopped; no-op if running |
| `stopped` | Stop if running; no-op if stopped |
| `restarted` | Stop then start (always runs) |
| `reloaded` | Reload config without full restart |

---

## systemd Module

Provides extra systemd-specific features.

```yaml
# Start a systemd service
- name: Start Docker
  systemd:
    name: docker
    state: started
    enabled: yes

# Daemon reload (after unit file changes)
- name: Reload systemd daemon
  systemd:
    daemon_reload: yes

# Reload then restart
- name: Deploy and restart
  systemd:
    name: myapp
    state: restarted
    daemon_reload: yes

# Mask a service (prevent starting)
- name: Mask unwanted service
  systemd:
    name: cups
    masked: yes

# Unmask a service
- name: Unmask service
  systemd:
    name: cups
    masked: no

# Check service scope
- name: Manage user service
  systemd:
    name: myapp
    scope: user
    state: started
```

### systemd vs service

| Feature | `service` | `systemd` |
|---------|:---------:|:---------:|
| Generic (works on any init) | ✓ | — |
| `daemon_reload` | — | ✓ |
| `masked` | — | ✓ |
| `scope` (system/user/global) | — | ✓ |
| Recommended for systemd hosts | — | ✓ |

---

## Creating Custom systemd Unit Files

```yaml
- name: Deploy systemd unit file
  template:
    src: myapp.service.j2
    dest: /etc/systemd/system/myapp.service
    owner: root
    mode: '0644'
  notify:
    - reload systemd
    - restart myapp
```

```jinja2
# templates/myapp.service.j2
# {{ ansible_managed }}
[Unit]
Description={{ app_description | default('My Application') }}
After=network.target
{% if app_requires_db | default(false) %}
After=postgresql.service
Requires=postgresql.service
{% endif %}

[Service]
Type=simple
User={{ app_user }}
Group={{ app_group }}
WorkingDirectory={{ app_dir }}
ExecStart={{ app_dir }}/venv/bin/gunicorn \
    --workers {{ app_workers | default(4) }} \
    --bind {{ app_bind | default('0.0.0.0:8000') }} \
    wsgi:app
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5
Environment=APP_ENV={{ app_env | default('production') }}

[Install]
WantedBy=multi-user.target
```

```yaml
handlers:
  - name: reload systemd
    systemd:
      daemon_reload: yes

  - name: restart myapp
    systemd:
      name: myapp
      state: restarted
```

---

## Checking Service Status

```yaml
- name: Get service facts
  service_facts:

- name: Show Nginx status
  debug:
    msg: "Nginx is {{ ansible_facts.services['nginx.service'].state }}"

- name: Fail if database not running
  assert:
    that:
      - ansible_facts.services['postgresql.service'].state == 'running'
    fail_msg: "PostgreSQL is not running!"

# Check with command
- name: Check if service is active
  command: systemctl is-active nginx
  register: nginx_status
  changed_when: false
  failed_when: false

- name: Report status
  debug:
    msg: "Nginx is {{ nginx_status.stdout }}"
```

---

## wait_for — Verify Service is Ready

```yaml
# Wait for a port
- name: Start Nginx
  service:
    name: nginx
    state: started

- name: Wait for Nginx to be ready
  wait_for:
    port: 80
    delay: 2
    timeout: 30

# Wait for a socket file
- name: Wait for PHP-FPM socket
  wait_for:
    path: /run/php/php-fpm.sock
    state: present
    timeout: 30

# Wait for a log line
- name: Wait for app to be ready
  wait_for:
    path: /var/log/myapp/startup.log
    search_regex: "Application started"
    timeout: 60
```

---

## Real-World Scenario: Full Service Lifecycle

```yaml
---
- name: Deploy and manage application
  hosts: app_servers
  become: yes
  vars:
    app_name: myapp
    app_user: appuser
    app_dir: /opt/myapp

  tasks:
    - name: Deploy unit file
      template:
        src: myapp.service.j2
        dest: /etc/systemd/system/{{ app_name }}.service
        mode: '0644'
      notify: reload systemd

    - name: Deploy application code
      synchronize:
        src: app/
        dest: "{{ app_dir }}/"
      notify: restart myapp

    - name: Ensure service is started and enabled
      systemd:
        name: "{{ app_name }}"
        state: started
        enabled: yes

    - name: Wait for application to be ready
      wait_for:
        port: 8000
        delay: 3
        timeout: 30

    - name: Verify health check
      uri:
        url: "http://localhost:8000/health"
        status_code: 200
      retries: 5
      delay: 5

  handlers:
    - name: reload systemd
      systemd:
        daemon_reload: yes

    - name: restart myapp
      systemd:
        name: "{{ app_name }}"
        state: restarted
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| `Could not find the requested service` | Check service name with `systemctl list-units` |
| Service starts but crashes | Check `journalctl -u service_name` for errors |
| `daemon_reload` needed error | Add `daemon_reload: yes` to `systemd` module |
| `reloaded` not supported | Not all services support reload — use `restarted` |
| Service disabled after reboot | Set `enabled: yes` |
| Port not open after start | Use `wait_for` with a delay to check readiness |

---

## Summary Table

| Module | Purpose |
|--------|---------|
| `service` | Generic service management (any init system) |
| `systemd` | systemd-specific (daemon_reload, mask, scope) |
| `service_facts` | Gather status of all services |
| `wait_for` | Wait for port/file/pattern readiness |
| `uri` | HTTP health check after start |

| State | Effect |
|-------|--------|
| `started` | Start if not running |
| `stopped` | Stop if running |
| `restarted` | Always stop + start |
| `reloaded` | Reload config (no restart) |

---

## Quick Revision Questions

1. **What is the difference between `state: started` and `state: restarted`?**
   > `started` only starts the service if it's not running (idempotent). `restarted` always stops and starts the service.

2. **When do you need `daemon_reload: yes`?**
   > After creating or modifying a systemd unit file (`/etc/systemd/system/*.service`), so systemd picks up the changes.

3. **How do you prevent a service from ever being started?**
   > Use the `systemd` module with `masked: yes` — this makes the service impossible to start.

4. **How do you verify a service is actually ready to accept connections?**
   > Use the `wait_for` module to wait for a port, socket file, or log pattern after starting the service.

5. **What module gives you the status of all services?**
   > The `service_facts` module populates `ansible_facts.services` with the state of every known service.

---

[← Previous: Package Modules](01-package-modules.md) | [Next: Command Modules →](03-command-modules.md)
