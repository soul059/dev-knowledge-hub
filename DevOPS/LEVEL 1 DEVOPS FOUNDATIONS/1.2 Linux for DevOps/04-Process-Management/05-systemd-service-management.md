# Chapter 5: Systemd Service Management

[← Previous: Background & Foreground Jobs](04-background-foreground-jobs.md) | [Back to README](../README.md) | [Next: Cron Jobs & Scheduling →](06-cron-jobs-scheduling.md)

---

## Overview

`systemd` is the init system and service manager used by most modern Linux distributions. For DevOps engineers, mastering `systemctl` and writing unit files is critical for managing services like Nginx, Docker, databases, and custom applications.

---

## 5.1 Systemd Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     SYSTEMD ARCHITECTURE                     │
│                                                               │
│  ┌───────────────────────────────────┐                       │
│  │         systemctl (CLI)            │                       │
│  └──────────────┬────────────────────┘                       │
│                  │                                             │
│  ┌──────────────▼────────────────────┐                       │
│  │         systemd (PID 1)           │                       │
│  │     Init System + Service Mgr     │                       │
│  └──┬──────┬──────┬───────┬─────────┘                       │
│     │      │      │       │                                   │
│  ┌──▼──┐┌──▼──┐┌──▼───┐┌─▼──────┐                          │
│  │Units ││Timer││Socket││Target  │                          │
│  │.serv.││.timr││.sock ││.target │                          │
│  └──────┘└─────┘└──────┘└────────┘                          │
│                                                               │
│  Unit Files Location:                                        │
│  /usr/lib/systemd/system/    ← Package defaults              │
│  /etc/systemd/system/        ← Admin overrides (priority)    │
│  /run/systemd/system/        ← Runtime (transient)           │
└─────────────────────────────────────────────────────────────┘
```

---

## 5.2 `systemctl` — Managing Services

```bash
# Start / Stop / Restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx         # Stop + Start
sudo systemctl reload nginx          # Reload config (no downtime)
sudo systemctl reload-or-restart nginx

# Enable / Disable (auto-start at boot)
sudo systemctl enable nginx          # Enable at boot
sudo systemctl enable --now nginx    # Enable + Start immediately
sudo systemctl disable nginx         # Disable at boot
sudo systemctl disable --now nginx   # Disable + Stop immediately

# Status
systemctl status nginx               # Detailed status
systemctl is-active nginx            # active / inactive
systemctl is-enabled nginx           # enabled / disabled
systemctl is-failed nginx            # failed / active

# List services
systemctl list-units --type=service                 # Running services
systemctl list-units --type=service --state=failed  # Failed services
systemctl list-unit-files --type=service            # All service files

# Mask / Unmask (completely prevent starting)
sudo systemctl mask nginx            # Cannot be started at all
sudo systemctl unmask nginx          # Remove mask

# Reload systemd daemon (after editing unit files)
sudo systemctl daemon-reload
```

---

## 5.3 Service Status Output Explained

```
┌──────────────────────────────────────────────────────────────┐
│  $ systemctl status nginx                                     │
│                                                                │
│  ● nginx.service - A high performance web server              │
│     Loaded: loaded (/lib/systemd/system/nginx.service;        │
│             enabled; vendor preset: enabled)                   │
│     Active: active (running) since Wed 2024-01-15 10:30:00    │
│     Docs: man:nginx(8)                                        │
│    Process: 1234 ExecStartPre=/usr/sbin/nginx -t (code=exited)│
│   Main PID: 1240 (nginx)                                      │
│      Tasks: 5 (limit: 4915)                                   │
│     Memory: 12.5M                                             │
│        CPU: 250ms                                              │
│     CGroup: /system.slice/nginx.service                       │
│             ├─1240 nginx: master process /usr/sbin/nginx      │
│             ├─1241 nginx: worker process                      │
│             └─1242 nginx: worker process                      │
│                                                                │
│  Jan 15 10:30:00 server systemd[1]: Starting A high perf...   │
│  Jan 15 10:30:00 server systemd[1]: Started A high perf...    │
│                                                                │
│  KEY FIELDS:                                                   │
│  ● (green dot) = running     ○ (empty) = inactive             │
│  ● (red dot)   = failed                                        │
│  Loaded: Where the unit file is + enabled/disabled             │
│  Active: Current state + since when                            │
│  Main PID: Process ID of the main daemon                      │
│  CGroup: All processes in the service's control group          │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.4 Writing Custom Unit Files

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My DevOps Application
Documentation=https://docs.myapp.io
After=network.target postgresql.service
Wants=postgresql.service
Requires=network-online.target

[Service]
Type=simple
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp
EnvironmentFile=/etc/myapp/env
ExecStartPre=/opt/myapp/pre-start.sh
ExecStart=/opt/myapp/bin/myapp --config /etc/myapp/config.yml
ExecStartPost=/opt/myapp/post-start.sh
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -TERM $MAINPID
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

# Security hardening
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/myapp /var/log/myapp

[Install]
WantedBy=multi-user.target
```

### Unit File Sections Explained

```
┌─────────────────────────────────────────────────────────────┐
│                UNIT FILE STRUCTURE                            │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  [Unit]       ← Metadata and dependencies                    │
│  Description  ← Human-readable name                          │
│  After        ← Start AFTER these units                      │
│  Requires     ← Hard dependency (fails if dep fails)         │
│  Wants        ← Soft dependency (continues if dep fails)     │
│                                                               │
│  [Service]    ← Service behavior                             │
│  Type         ← simple | forking | oneshot | notify          │
│  ExecStart    ← Command to start the service                 │
│  ExecReload   ← Command to reload config                     │
│  Restart      ← When to restart: always | on-failure | no    │
│  User/Group   ← Run as specific user                         │
│                                                               │
│  [Install]    ← How the service is enabled                   │
│  WantedBy     ← Target to attach to (multi-user.target)     │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Service Types

```
┌────────────┬──────────────────────────────────────────────┐
│ Type       │ Description                                  │
├────────────┼──────────────────────────────────────────────┤
│ simple     │ ExecStart is the main process (default)      │
│ forking    │ Process forks; parent exits (traditional)    │
│ oneshot    │ Runs once and exits (startup scripts)        │
│ notify     │ Sends sd_notify when ready                   │
│ idle       │ Runs after all jobs dispatched               │
└────────────┴──────────────────────────────────────────────┘
```

---

## 5.5 Systemd Targets (Runlevels)

```
┌──────────────────────────────────────────────────────────────┐
│              SYSTEMD TARGETS vs SYSVINIT RUNLEVELS            │
├──────────────────┬──────────┬────────────────────────────────┤
│ Target           │ Runlevel │ Purpose                        │
├──────────────────┼──────────┼────────────────────────────────┤
│ poweroff.target  │    0     │ System halt                    │
│ rescue.target    │    1     │ Single-user / rescue           │
│ multi-user.target│    3     │ Multi-user, no GUI (SERVERS)   │
│ graphical.target │    5     │ Multi-user with GUI            │
│ reboot.target    │    6     │ Reboot                         │
│ emergency.target │   N/A    │ Emergency shell                │
└──────────────────┴──────────┴────────────────────────────────┘
```

```bash
# Check current target
systemctl get-default

# Set default target
sudo systemctl set-default multi-user.target     # Servers
sudo systemctl set-default graphical.target       # Desktop

# Switch target now
sudo systemctl isolate multi-user.target

# List all targets
systemctl list-units --type=target
```

---

## 5.6 Viewing Logs with `journalctl`

```bash
# All logs
journalctl

# Specific service
journalctl -u nginx
journalctl -u nginx --since "1 hour ago"
journalctl -u nginx --since "2024-01-15" --until "2024-01-16"

# Follow logs (like tail -f)
journalctl -u nginx -f

# Recent logs
journalctl -u nginx -n 50       # Last 50 lines

# Boot logs
journalctl -b                    # Current boot
journalctl -b -1                 # Previous boot
journalctl --list-boots           # List all boots

# Priority filter
journalctl -p err                # Errors and above
journalctl -p warning -u nginx   # Warnings for nginx

# Output formats
journalctl -u nginx -o json-pretty    # JSON format
journalctl -u nginx -o short-iso      # ISO timestamps

# Disk usage
journalctl --disk-usage
sudo journalctl --vacuum-size=500M    # Trim to 500MB
sudo journalctl --vacuum-time=7d      # Keep only 7 days
```

---

## 5.7 Real-World Scenario: Deploying a Node.js App as a Service

```ini
# /etc/systemd/system/nodeapp.service
[Unit]
Description=Node.js Production API
After=network.target

[Service]
Type=simple
User=nodeapp
Group=nodeapp
WorkingDirectory=/opt/nodeapp
EnvironmentFile=/etc/nodeapp/.env
ExecStart=/usr/bin/node /opt/nodeapp/server.js
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=nodeapp

# Limits
LimitNOFILE=65535
TimeoutStartSec=30
TimeoutStopSec=30

# Security
NoNewPrivileges=true
ProtectSystem=full

[Install]
WantedBy=multi-user.target
```

```bash
# Deploy steps
sudo useradd -r -s /sbin/nologin nodeapp
sudo mkdir -p /opt/nodeapp
sudo cp -r ./dist/* /opt/nodeapp/
sudo chown -R nodeapp:nodeapp /opt/nodeapp

sudo cp nodeapp.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now nodeapp

# Verify
systemctl status nodeapp
journalctl -u nodeapp -f
curl http://localhost:3000/health
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Service fails to start | `systemctl status svc` + `journalctl -u svc` | Check ExecStart path, permissions, config |
| "Main process exited, code=exited, status=217" | User doesn't exist | Create user with `useradd -r` |
| Changes to unit file not applied | Daemon not reloaded | Run `systemctl daemon-reload` |
| Service keeps restarting | Crash loop | Check `RestartSec`, fix root cause in logs |
| "Unit is masked" | Service was masked | `systemctl unmask service` |
| Can't enable at boot | Missing `[Install]` section | Add `WantedBy=multi-user.target` |

---

## Summary Table

| Command | Purpose | Example |
|---------|---------|---------|
| `systemctl start` | Start service | `systemctl start nginx` |
| `systemctl stop` | Stop service | `systemctl stop nginx` |
| `systemctl restart` | Restart service | `systemctl restart nginx` |
| `systemctl reload` | Reload config | `systemctl reload nginx` |
| `systemctl enable` | Auto-start at boot | `systemctl enable --now nginx` |
| `systemctl status` | Check status | `systemctl status nginx` |
| `systemctl mask` | Prevent starting | `systemctl mask service` |
| `daemon-reload` | Reload unit files | `systemctl daemon-reload` |
| `journalctl -u` | View service logs | `journalctl -u nginx -f` |

---

## Quick Revision Questions

1. **What is the difference between `systemctl enable` and `systemctl start`?**
2. **After editing a unit file, what must you run before restarting the service?**
3. **What is the difference between `Requires` and `Wants` in a unit file?**
4. **How do you view the last 100 lines of logs for a service and follow new entries?**
5. **What is the difference between `systemctl mask` and `systemctl disable`?**
6. **Write the key parts of a unit file for a Python Flask app running on port 5000.**

---

[← Previous: Background & Foreground Jobs](04-background-foreground-jobs.md) | [Back to README](../README.md) | [Next: Cron Jobs & Scheduling →](06-cron-jobs-scheduling.md)
