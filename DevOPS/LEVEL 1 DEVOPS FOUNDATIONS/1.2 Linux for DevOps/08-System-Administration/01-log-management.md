# Chapter 1: Log Management

[← Previous: Shell Scripting Best Practices](../07-Shell-Scripting/09-best-practices.md) | [Back to README](../README.md) | [Next: System Monitoring →](02-system-monitoring.md)

---

## Overview

Log management is the backbone of observability in Linux servers. Knowing where logs live, how to read them, how to rotate them, and how to centralize them is critical for debugging incidents, auditing security, and maintaining compliance in DevOps environments.

---

## 1.1 Linux Log Architecture

```
┌──────────────────────────────────────────────────────────────┐
│               LINUX LOGGING ARCHITECTURE                      │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Applications                    Kernel                       │
│  ┌─────────┐ ┌─────────┐       ┌─────────┐                  │
│  │ Nginx   │ │ Docker  │       │ dmesg   │                  │
│  │ MySQL   │ │ Custom  │       │ kmsg    │                  │
│  └────┬────┘ └────┬────┘       └────┬────┘                  │
│       │           │                  │                        │
│       ▼           ▼                  ▼                        │
│  ┌─────────────────────────────────────────┐                 │
│  │         systemd-journald                │                 │
│  │     (binary structured logs)            │                 │
│  └────────────────┬────────────────────────┘                 │
│                   │                                           │
│          ┌────────┴────────┐                                 │
│          ▼                 ▼                                  │
│   ┌────────────┐   ┌────────────┐                            │
│   │  rsyslog   │   │ journalctl │                            │
│   │ (text logs)│   │ (query)    │                            │
│   └─────┬──────┘   └────────────┘                            │
│         ▼                                                     │
│   /var/log/                                                   │
│   ├── syslog / messages                                       │
│   ├── auth.log / secure                                       │
│   ├── kern.log                                                │
│   ├── dmesg                                                   │
│   └── nginx/ mysql/ ...                                       │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 Important Log Files

```
┌────────────────────────┬──────────────────────────────────────┐
│ Log File               │ Contents                             │
├────────────────────────┼──────────────────────────────────────┤
│ /var/log/syslog        │ System messages (Ubuntu/Debian)      │
│ /var/log/messages      │ System messages (RHEL/CentOS)        │
│ /var/log/auth.log      │ Auth events (Ubuntu)                 │
│ /var/log/secure        │ Auth events (RHEL)                   │
│ /var/log/kern.log      │ Kernel messages                      │
│ /var/log/dmesg         │ Boot-time kernel messages            │
│ /var/log/boot.log      │ Boot process log                     │
│ /var/log/cron          │ Cron job execution log               │
│ /var/log/faillog       │ Failed login attempts                │
│ /var/log/lastlog       │ Last login for each user             │
│ /var/log/wtmp          │ Login/logout history (binary)        │
│ /var/log/btmp          │ Bad login attempts (binary)          │
│ /var/log/apt/          │ APT package manager logs             │
│ /var/log/dnf.log       │ DNF package manager log              │
│ /var/log/nginx/        │ Nginx access & error logs            │
│ /var/log/mysql/        │ MySQL server logs                    │
│ /var/log/containers/   │ Docker/Podman container logs         │
└────────────────────────┴──────────────────────────────────────┘
```

---

## 1.3 Viewing Logs

```bash
# Traditional log files
cat /var/log/syslog                    # View entire log
tail -20 /var/log/syslog               # Last 20 lines
tail -f /var/log/syslog                # Follow in real-time
tail -f /var/log/syslog | grep "error" # Follow + filter

less /var/log/syslog                   # Paginate (search with /)
head -50 /var/log/auth.log             # First 50 lines

# View binary logs
last                                   # Login history (/var/log/wtmp)
lastb                                  # Bad logins (/var/log/btmp)
lastlog                                # Last login per user
faillog -a                             # Failed login counts

# Kernel messages
dmesg                                  # Kernel ring buffer
dmesg -T                               # Human-readable timestamps
dmesg --level=err,warn                 # Only errors and warnings
dmesg -w                               # Follow (watch) mode
```

---

## 1.4 journalctl — Systemd Journal

```bash
# View all logs
journalctl                             # All logs (paged)
journalctl --no-pager                  # Without pager

# Follow (like tail -f)
journalctl -f
journalctl -f -u nginx                # Follow specific unit

# Filter by unit (service)
journalctl -u nginx.service
journalctl -u docker.service --since "1 hour ago"
journalctl -u sshd -u nginx           # Multiple units

# Filter by time
journalctl --since "2024-01-15 10:00:00"
journalctl --since "1 hour ago"
journalctl --since yesterday --until today
journalctl --since "2024-01-15" --until "2024-01-16"

# Filter by priority (severity)
journalctl -p err                      # Errors and above
journalctl -p warning                  # Warnings and above
# Levels: emerg(0), alert(1), crit(2), err(3), warning(4),
#         notice(5), info(6), debug(7)

# Filter by boot
journalctl -b                          # Current boot
journalctl -b -1                       # Previous boot
journalctl --list-boots                # List all boots

# Output formats
journalctl -o json                     # JSON format
journalctl -o json-pretty              # Pretty JSON
journalctl -o short-iso                # ISO timestamps
journalctl -o cat                      # Message only (no metadata)

# Disk usage
journalctl --disk-usage
# Clean old entries
journalctl --vacuum-time=7d            # Keep last 7 days
journalctl --vacuum-size=500M          # Keep last 500MB
```

```
┌──────────────────────────────────────────────────────────────┐
│           SYSLOG SEVERITY LEVELS                              │
├──────────┬──────────┬────────────────────────────────────────┤
│ Level    │ Keyword  │ Description                            │
├──────────┼──────────┼────────────────────────────────────────┤
│ 0        │ emerg    │ System unusable                        │
│ 1        │ alert    │ Action must be taken immediately       │
│ 2        │ crit     │ Critical conditions                    │
│ 3        │ err      │ Error conditions                       │
│ 4        │ warning  │ Warning conditions                     │
│ 5        │ notice   │ Normal but significant                 │
│ 6        │ info     │ Informational messages                 │
│ 7        │ debug    │ Debug-level messages                   │
└──────────┴──────────┴────────────────────────────────────────┘
```

---

## 1.5 rsyslog Configuration

```bash
# Config file: /etc/rsyslog.conf

# Facility.Priority    Action
# Example rules:
auth,authpriv.*        /var/log/auth.log
*.*;auth,authpriv.none /var/log/syslog
kern.*                 /var/log/kern.log
mail.*                 /var/log/mail.log
*.emerg                :omusrmsg:*

# Custom application logging
# /etc/rsyslog.d/myapp.conf
if $programname == 'myapp' then /var/log/myapp.log
& stop

# Remote logging (send to central server)
*.* @logserver.example.com:514     # UDP
*.* @@logserver.example.com:514    # TCP

# Restart after config changes
sudo systemctl restart rsyslog
```

---

## 1.6 Log Rotation

```bash
# logrotate manages log file rotation
# Config: /etc/logrotate.conf
# App configs: /etc/logrotate.d/

# Example: /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    daily              # Rotate daily
    missingok          # Don't error if log missing
    rotate 14          # Keep 14 rotated files
    compress           # gzip old logs
    delaycompress      # Don't compress most recent
    notifempty         # Don't rotate empty logs
    create 0640 www-data adm   # Permissions for new file
    sharedscripts      # Run postrotate once for all logs
    postrotate
        [ -f /var/run/nginx.pid ] && kill -USR1 $(cat /var/run/nginx.pid)
    endscript
}

# Test rotation (dry run)
sudo logrotate -d /etc/logrotate.d/nginx

# Force rotation
sudo logrotate -f /etc/logrotate.d/nginx

# Check logrotate status
cat /var/lib/logrotate/status
```

```
┌──────────────────────────────────────────────────────────────┐
│            LOG ROTATION LIFECYCLE                             │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Day 1:  app.log (current)                                    │
│  Day 2:  app.log (new) ← app.log.1                           │
│  Day 3:  app.log (new) ← app.log.1 ← app.log.2.gz           │
│  Day 4:  app.log (new) ← app.log.1 ← app.log.2.gz           │
│                                         ← app.log.3.gz       │
│   ...                                                         │
│  Day 15: app.log.14.gz is deleted (rotate 14)                 │
│                                                               │
│  Rotation triggers:                                           │
│  · Time-based: daily, weekly, monthly                         │
│  · Size-based: size 100M                                      │
│  · Both: maxsize 100M (rotate if size OR time)                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.7 Custom Application Logging

```bash
# Using logger command (writes to syslog)
logger "Application deployed successfully"
logger -p local0.info "Deploy version 2.1.0"
logger -t myapp "Starting health check"
logger -p user.err "Database connection failed"

# In scripts
log() {
    local level="${1:-info}"
    shift
    logger -t "$(basename "$0")" -p "user.${level}" "$*"
    echo "$(date '+%F %T') [${level^^}] $*"
}

log info "Deployment started"
log error "Failed to connect to database"
```

---

## 1.8 Real-World Scenario: Log Analysis & Alerting Script

```bash
#!/usr/bin/env bash
# log-alert.sh — Monitor logs and alert on patterns
# Run via cron: */5 * * * * /opt/scripts/log-alert.sh

set -euo pipefail

ALERT_EMAIL="ops@company.com"
STATE_FILE="/var/run/log-alert.state"
LOG_FILE="/var/log/syslog"

# Get last checked position
if [ -f "$STATE_FILE" ]; then
    LAST_LINE=$(cat "$STATE_FILE")
else
    LAST_LINE=0
fi

CURRENT_LINES=$(wc -l < "$LOG_FILE")

# If log was rotated (current < last), reset
if (( CURRENT_LINES < LAST_LINE )); then
    LAST_LINE=0
fi

# Check new lines only
NEW_LINES=$((CURRENT_LINES - LAST_LINE))
if (( NEW_LINES <= 0 )); then
    exit 0
fi

# Extract new content
ALERTS=$(tail -n "$NEW_LINES" "$LOG_FILE" | grep -iE "error|critical|failed|panic|oom" || true)

if [ -n "$ALERTS" ]; then
    ALERT_COUNT=$(echo "$ALERTS" | wc -l)
    
    {
        echo "Host: $(hostname)"
        echo "Time: $(date)"
        echo "New alerts: $ALERT_COUNT"
        echo ""
        echo "--- Alert Details ---"
        echo "$ALERTS" | head -50   # Limit to 50 lines
    } | mail -s "Log Alert: $(hostname) ($ALERT_COUNT issues)" "$ALERT_EMAIL" 2>/dev/null
    
    echo "$(date '+%F %T') Sent alert ($ALERT_COUNT issues)" >> /var/log/log-alert.log
fi

# Save position
echo "$CURRENT_LINES" > "$STATE_FILE"
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| `/var/log` disk full | Large unrotated logs | Set up logrotate, clean old logs: `find /var/log -name '*.gz' -mtime +30 -delete` |
| journalctl shows no old logs | Volatile storage | Edit `/etc/systemd/journald.conf`: set `Storage=persistent` |
| Log timestamps wrong | Time zone or NTP issue | Fix with `timedatectl set-timezone` and enable NTP |
| App logs not appearing | Wrong syslog facility | Check rsyslog config and app logging config |
| `logrotate` not working | Config syntax error | Test with `logrotate -d /etc/logrotate.d/app` |
| Docker logs filling disk | No log rotation | Set `--log-opt max-size=10m --log-opt max-file=3` |

---

## Summary Table

| Tool | Purpose | Key Command |
|------|---------|-------------|
| `journalctl` | Query systemd journal | `journalctl -u nginx -f` |
| `tail -f` | Follow log file | `tail -f /var/log/syslog` |
| `rsyslog` | Traditional syslog daemon | Config: `/etc/rsyslog.conf` |
| `logrotate` | Rotate & compress logs | Config: `/etc/logrotate.d/` |
| `logger` | Write to syslog from CLI | `logger -t myapp "message"` |
| `dmesg` | Kernel messages | `dmesg -T --level=err` |
| `last` | Login history | `last -10` |
| `lastb` | Failed logins | `sudo lastb` |

---

## Quick Revision Questions

1. **What is the difference between `/var/log/syslog` and `journalctl`?**
2. **How do you view only error-level messages from the last hour using `journalctl`?**
3. **Write a logrotate config that rotates daily, keeps 7 days, compresses, and reloads Nginx.**
4. **How do you send logs to a remote syslog server?**
5. **What is the `logger` command used for?**
6. **How do you limit Docker container log size to prevent disk exhaustion?**

---

[← Previous: Shell Scripting Best Practices](../07-Shell-Scripting/09-best-practices.md) | [Back to README](../README.md) | [Next: System Monitoring →](02-system-monitoring.md)
