# Chapter 9.3: Callback Plugins

[← Previous: Async and Polling](02-async-and-polling.md) | [Next: Custom Modules →](04-custom-modules.md)

---

## Overview

**Callback plugins** control how Ansible displays output and responds to events during playbook execution. They let you customise terminal output, send notifications, log to files, integrate with monitoring systems, and generate reports.

---

## How Callback Plugins Work

```
  ANSIBLE EXECUTION ENGINE
  ══════════════════════════════════════════════════════════

  ┌─────────────┐     Events     ┌──────────────────────┐
  │  Playbook   │───────────────▶│  Callback Manager    │
  │  Runner     │                │                      │
  │             │  task_start    │  ┌────────────────┐   │
  │             │  task_ok       │  │  stdout plugin │   │  ◄── Only ONE
  │             │  task_failed   │  └────────────────┘   │
  │             │  play_start    │  ┌────────────────┐   │
  │             │  stats         │  │  notify plugin │   │  ◄── Multiple
  │             │  ...           │  │  log plugin    │   │      allowed
  └─────────────┘                │  │  timer plugin  │   │
                                 │  └────────────────┘   │
                                 └──────────────────────┘
```

---

## Two Types of Callbacks

| Type | Description | Active Count |
|------|-------------|:------------:|
| **stdout** | Controls terminal output format | Exactly **one** |
| **notification** | Additional actions on events | **Multiple** |

---

## Built-in stdout Plugins

```ini
# ansible.cfg
[defaults]
stdout_callback = yaml       # Change output format
```

| Plugin | Output Style |
|--------|-------------|
| `default` | Standard Ansible output |
| `yaml` | YAML-formatted output (readable) |
| `json` | JSON-formatted output |
| `debug` | Verbose with full stdout/stderr |
| `dense` | Minimal, one-line-per-task |
| `minimal` | Bare minimum output |
| `unixy` | Unix-style condensed output |
| `oneline` | Everything on one line per task |

### Examples

```
  default:
  TASK [Install nginx] *************************************
  changed: [web01]

  yaml:
  TASK [Install nginx] *************************************
  changed: [web01]
    apt:
      name: nginx
      state: present

  dense:
  web01 | TASK: Install nginx | changed

  minimal:
  web01 | CHANGED

  json:
  {"task": "Install nginx", "host": "web01", "status": "changed"}
```

---

## Notification Callback Plugins

```ini
# ansible.cfg
[defaults]
callback_whitelist = timer, profile_tasks, mail

# Or in newer versions:
callbacks_enabled = timer, profile_tasks, mail
```

### timer — Show Total Execution Time

```ini
[defaults]
callbacks_enabled = timer
```

```
  Playbook run took 0 days, 0 hours, 2 minutes, 34 seconds
```

### profile_tasks — Task Timing

```ini
[defaults]
callbacks_enabled = profile_tasks
```

```
  Thursday 15 January 2024  10:30:45 +0000 (0:00:12.345)
  ============================================================
  Install nginx ---------------------------------------- 12.35s
  Deploy config ----------------------------------------  3.21s
  Restart nginx ----------------------------------------  1.05s
  Gather facts -----------------------------------------  2.50s
  ============================================================
```

### profile_roles — Role Timing

```ini
[defaults]
callbacks_enabled = profile_roles
```

```
  Role                          Time
  ============================================================
  webserver                     25.34s
  common                        12.50s
  security                       8.21s
```

### log_plays — Log to File

```ini
[defaults]
callbacks_enabled = log_plays
log_path = /var/log/ansible/playbook.log
```

---

## Configuring Callback Plugins

### ansible.cfg

```ini
[defaults]
# Stdout plugin (only one)
stdout_callback = yaml

# Notification plugins (multiple)
callbacks_enabled = timer, profile_tasks, profile_roles

# Some plugins have their own sections
[callback_mail]
to = ops@example.com
sender = ansible@example.com
smtphost = smtp.example.com
```

### Environment Variables

```bash
# Override stdout callback
export ANSIBLE_STDOUT_CALLBACK=json

# Enable additional callbacks
export ANSIBLE_CALLBACKS_ENABLED=timer,profile_tasks
```

---

## Custom Callback Plugin

```python
# callback_plugins/custom_notify.py
from ansible.plugins.callback import CallbackBase
import json
import requests

class CallbackModule(CallbackBase):
    CALLBACK_VERSION = 2.0
    CALLBACK_TYPE = 'notification'
    CALLBACK_NAME = 'custom_notify'
    CALLBACK_NEEDS_WHITELIST = True

    def __init__(self):
        super().__init__()
        self.webhook_url = "https://hooks.example.com/ansible"

    def v2_playbook_on_stats(self, stats):
        """Called at the end of a playbook run."""
        hosts = sorted(stats.processed.keys())
        summary = {}
        for h in hosts:
            s = stats.summarize(h)
            summary[h] = s

        payload = {
            "playbook_summary": summary,
            "total_hosts": len(hosts),
        }

        try:
            requests.post(self.webhook_url, json=payload)
        except Exception as e:
            self._display.warning(f"Notification failed: {e}")

    def v2_runner_on_failed(self, result, ignore_errors=False):
        """Called when a task fails."""
        if not ignore_errors:
            payload = {
                "event": "task_failed",
                "host": result._host.get_name(),
                "task": result._task.get_name(),
                "message": result._result.get('msg', ''),
            }
            try:
                requests.post(self.webhook_url, json=payload)
            except Exception:
                pass
```

```
  CALLBACK PLUGIN LOCATION
  ══════════════════════════════════════════════════════════

  project/
  ├── ansible.cfg
  ├── callback_plugins/          ◄── Auto-discovered
  │   └── custom_notify.py
  ├── playbooks/
  └── roles/

  Or global: ~/.ansible/plugins/callback/
```

---

## Useful Callback Events

| Event Method | When Triggered |
|-------------|----------------|
| `v2_playbook_on_start` | Playbook begins |
| `v2_playbook_on_play_start` | Each play begins |
| `v2_playbook_on_task_start` | Each task begins |
| `v2_runner_on_ok` | Task completes successfully |
| `v2_runner_on_failed` | Task fails |
| `v2_runner_on_skipped` | Task skipped |
| `v2_runner_on_unreachable` | Host unreachable |
| `v2_playbook_on_stats` | Playbook ends (final stats) |

---

## Real-World Scenario: CI/CD Output and Notifications

```ini
# ansible.cfg for CI/CD
[defaults]
stdout_callback = yaml
callbacks_enabled = timer, profile_tasks, junit

[callback_junit]
output_dir = ./test-results/
fail_on_change = true
```

```
  CI/CD PIPELINE
  ══════════════════════════════════════════════════════════

  ansible-playbook site.yml
       │
       ├── YAML stdout  ──▶  Console (human-readable)
       ├── timer         ──▶  Total execution time
       ├── profile_tasks ──▶  Per-task timing
       └── junit         ──▶  ./test-results/  (JUnit XML)
                                   │
                              CI tool reads ──▶ Test reports
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Callback not loading | Add to `callbacks_enabled` in ansible.cfg |
| Only one stdout plugin works | That's by design — only one stdout callback at a time |
| Custom plugin not found | Place in `callback_plugins/` dir or configure `callback_plugins` path |
| `CALLBACK_NEEDS_WHITELIST` | Must be listed in `callbacks_enabled` |
| Environment variable overrides config | Check `ANSIBLE_STDOUT_CALLBACK` env var |

---

## Summary Table

| Plugin | Type | Purpose |
|--------|------|---------|
| `default` | stdout | Standard output |
| `yaml` | stdout | YAML-formatted output |
| `json` | stdout | JSON-formatted output |
| `debug` | stdout | Verbose with full output |
| `timer` | notification | Total playbook runtime |
| `profile_tasks` | notification | Per-task timing |
| `profile_roles` | notification | Per-role timing |
| `log_plays` | notification | Log to file |
| `junit` | notification | JUnit XML reports |
| `mail` | notification | Email on failure |

---

## Quick Revision Questions

1. **What is the difference between stdout and notification callback plugins?**
   > Only one stdout plugin can be active (controls output format). Multiple notification plugins can run simultaneously (handle events).

2. **How do you enable notification callback plugins?**
   > Add them to `callbacks_enabled` (or `callback_whitelist`) in `ansible.cfg`.

3. **What does the `profile_tasks` callback do?**
   > It shows execution time for each task, sorted by duration, helping identify bottlenecks.

4. **Where do you place custom callback plugins?**
   > In a `callback_plugins/` directory adjacent to your playbook, or in the path configured by `callback_plugins` in ansible.cfg.

5. **How would you generate JUnit XML reports for a CI pipeline?**
   > Enable the `junit` callback plugin and configure `output_dir` in `[callback_junit]` section.

---

[← Previous: Async and Polling](02-async-and-polling.md) | [Next: Custom Modules →](04-custom-modules.md)
