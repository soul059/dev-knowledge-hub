# Chapter 9.1: Delegation

[← Previous: Cloud Modules](../08-Ansible-Modules/05-cloud-modules.md) | [Next: Async and Polling →](02-async-and-polling.md)

---

## Overview

**Delegation** lets you execute a task on a different host than the one currently being processed. This is useful for load balancer management, database operations, monitoring API calls, or any scenario where an action for one host must be performed somewhere else.

---

## delegate_to

```yaml
- name: Configure web servers
  hosts: webservers
  tasks:
    - name: Remove from load balancer
      uri:
        url: "http://lb.example.com/api/remove"
        method: POST
        body: '{"host": "{{ inventory_hostname }}"}'
        body_format: json
      delegate_to: lb.example.com

    - name: Update application
      apt:
        name: myapp
        state: latest

    - name: Add back to load balancer
      uri:
        url: "http://lb.example.com/api/add"
        method: POST
        body: '{"host": "{{ inventory_hostname }}"}'
        body_format: json
      delegate_to: lb.example.com
```

```
  DELEGATION FLOW
  ══════════════════════════════════════════════════════════

  Processing: web01               Executed on: lb.example.com
  ┌──────────────┐                ┌──────────────────────┐
  │  web01       │──delegate_to──▶│  lb.example.com      │
  │  (context)   │                │  (execution)         │
  │              │                │                      │
  │ Variables:   │                │ Runs the task here,  │
  │ inventory_   │                │ but uses web01's     │
  │ hostname =   │                │ variables            │
  │ "web01"      │                │                      │
  └──────────────┘                └──────────────────────┘
```

> **Key concept**: The task runs on the delegated host, but all variables (facts, inventory_hostname, etc.) still refer to the **original** host being processed.

---

## delegate_to: localhost

Common pattern for API calls, local file operations, or cloud provisioning:

```yaml
- name: Configure servers
  hosts: webservers
  tasks:
    - name: Create DNS record
      community.general.cloudflare_dns:
        zone: example.com
        record: "{{ inventory_hostname }}"
        type: A
        value: "{{ ansible_default_ipv4.address }}"
      delegate_to: localhost

    - name: Send Slack notification
      uri:
        url: https://hooks.slack.com/services/xxx
        method: POST
        body: '{"text": "Deploying to {{ inventory_hostname }}"}'
        body_format: json
      delegate_to: localhost
```

---

## delegate_facts

By default, facts set during delegation are assigned to the **original** host. Use `delegate_facts: true` to assign them to the delegated host:

```yaml
- name: Gather LB facts
  setup:
  delegate_to: lb.example.com
  delegate_facts: true
  # Facts stored under lb.example.com, not current host

- name: Show LB memory
  debug:
    msg: "LB RAM: {{ hostvars['lb.example.com'].ansible_memtotal_mb }}"
```

---

## local_action (Shorthand)

`local_action` is a shorthand for `delegate_to: localhost`:

```yaml
# These are equivalent:
- name: API call (delegate_to)
  uri:
    url: http://api.example.com/deploy
    method: POST
  delegate_to: localhost

- name: API call (local_action)
  local_action:
    module: uri
    url: http://api.example.com/deploy
    method: POST
```

> **Recommendation**: Use `delegate_to: localhost` — it's more explicit and consistent.

---

## run_once with Delegation

Run a task once across the entire play, delegated to a specific host:

```yaml
- name: Configure all web servers
  hosts: webservers
  tasks:
    - name: Run database migration (once only)
      command: python manage.py migrate
      args:
        chdir: /opt/myapp
      delegate_to: db.example.com
      run_once: true

    - name: Clear CDN cache (once only)
      uri:
        url: https://cdn.example.com/api/purge
        method: POST
      delegate_to: localhost
      run_once: true
```

---

## Rolling Update with Delegation

```yaml
---
- name: Rolling deployment
  hosts: webservers
  serial: 1               # One host at a time
  tasks:
    - name: Disable in load balancer
      haproxy:
        state: disabled
        host: "{{ inventory_hostname }}"
        backend: app_servers
        socket: /var/run/haproxy.sock
      delegate_to: "{{ item }}"
      loop: "{{ groups['loadbalancers'] }}"

    - name: Deploy application
      unarchive:
        src: "releases/app-{{ version }}.tar.gz"
        dest: /opt/app/
      notify: restart app

    - name: Wait for app to be ready
      wait_for:
        port: 8080
        delay: 5
        timeout: 30

    - name: Enable in load balancer
      haproxy:
        state: enabled
        host: "{{ inventory_hostname }}"
        backend: app_servers
        socket: /var/run/haproxy.sock
      delegate_to: "{{ item }}"
      loop: "{{ groups['loadbalancers'] }}"

  handlers:
    - name: restart app
      service:
        name: myapp
        state: restarted
```

---

## Real-World Scenario: Multi-Tier Deployment

```yaml
---
- name: Deploy with cross-host coordination
  hosts: app_servers
  serial: 2
  tasks:
    # 1. Notify monitoring
    - name: Set maintenance mode
      uri:
        url: "https://monitoring.example.com/api/downtime"
        method: POST
        body:
          host: "{{ inventory_hostname }}"
          duration: 600
        body_format: json
      delegate_to: localhost
      run_once: false

    # 2. Remove from LB
    - name: Remove from load balancer
      command: >
        /usr/local/bin/lb-ctl remove
        --backend app
        --server {{ inventory_hostname }}
      delegate_to: lb01.example.com

    # 3. Deploy on actual host (no delegation)
    - name: Deploy application
      apt:
        name: myapp={{ app_version }}
        state: present
      notify: restart myapp

    - name: Flush handlers
      meta: flush_handlers

    # 4. Health check
    - name: Verify health
      uri:
        url: "http://{{ inventory_hostname }}:8080/health"
        status_code: 200
      retries: 10
      delay: 3
      delegate_to: localhost

    # 5. Re-add to LB
    - name: Add back to load balancer
      command: >
        /usr/local/bin/lb-ctl add
        --backend app
        --server {{ inventory_hostname }}
      delegate_to: lb01.example.com

  handlers:
    - name: restart myapp
      service:
        name: myapp
        state: restarted
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Variables resolve to wrong host | Remember: variables come from the original host, not the delegate |
| `delegate_facts` not working | Must set `delegate_facts: true` explicitly |
| SSH connection error to delegate | Ensure delegate host is in inventory or accessible |
| Task runs multiple times | Use `run_once: true` for one-time delegated tasks |
| `become` applies wrong user | `become` context is on the delegated host |

---

## Summary Table

| Feature | Description |
|---------|-------------|
| `delegate_to` | Execute task on a different host |
| `delegate_to: localhost` | Run task on control node |
| `local_action` | Shorthand for localhost delegation |
| `delegate_facts: true` | Store facts on the delegated host |
| `run_once: true` | Execute only once across all hosts |
| Variable context | Always from the original host |

---

## Quick Revision Questions

1. **What does `delegate_to` do?**
   > It runs the task on the specified host instead of the current host being processed. Variables still reference the original host.

2. **Which host's variables are used during delegation?**
   > The original host's variables — the host being iterated over in the play, not the delegation target.

3. **How do you store gathered facts on the delegated host?**
   > Add `delegate_facts: true` to the task alongside `delegate_to`.

4. **What is the difference between `delegate_to: localhost` and `local_action`?**
   > They're equivalent, but `delegate_to: localhost` is preferred for consistency and clarity.

5. **When would you combine `delegate_to` with `run_once`?**
   > When you need a task to happen exactly once (e.g., database migration) on a specific host, rather than once per host in the play.

---

[← Previous: Cloud Modules](../08-Ansible-Modules/05-cloud-modules.md) | [Next: Async and Polling →](02-async-and-polling.md)
