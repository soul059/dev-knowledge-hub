# Syslog and journald

## Unit 3 - Topic 1 | Linux Logging and Auditing

---

## Overview

Linux uses two primary logging systems: **rsyslog** (traditional syslog daemon) and **journald** (systemd's journal). Both capture system events, authentication attempts, service status, and kernel messages. Understanding these systems is critical for **forensics**, **incident response**, and **security monitoring**.

---

## 1. Logging Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              LINUX LOGGING ARCHITECTURE                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Kernel ──────────┐                                          │
│  Services ────────┤                                          │
│  Applications ────┤                                          │
│  Auth events ─────┤                                          │
│                   ▼                                          │
│  ┌──────────────────────────┐                               │
│  │    systemd-journald      │ (binary, structured)          │
│  │    /run/log/journal/     │ (volatile by default)         │
│  └──────────┬───────────────┘                               │
│             │ forwards to                                    │
│  ┌──────────▼───────────────┐                               │
│  │    rsyslog               │ (text files)                  │
│  │    /var/log/*            │ (persistent)                  │
│  └──────────────────────────┘                               │
│                                                              │
│  journald: Structured, binary, queryable                     │
│  rsyslog:  Text files, traditional, widely supported         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. rsyslog Configuration

```bash
# Main config: /etc/rsyslog.conf
# Additional: /etc/rsyslog.d/*.conf

# SYSLOG FACILITY.SEVERITY FORMAT:
# facility.severity    action

# Facility: auth, kern, mail, daemon, user, local0-7
# Severity: emerg, alert, crit, err, warning, notice, info, debug

# Default rules:
auth,authpriv.*          /var/log/auth.log      # Authentication
*.*;auth,authpriv.none   /var/log/syslog        # General (except auth)
kern.*                   /var/log/kern.log      # Kernel
mail.*                   /var/log/mail.log      # Mail
daemon.*                 /var/log/daemon.log    # Daemon messages

# Custom rules:
# Log all critical messages to separate file
*.crit                   /var/log/critical.log

# Send auth logs to remote server
auth.*                   @@10.0.0.100:514       # TCP (@@)
auth.*                   @10.0.0.100:514        # UDP (@)

# Restart rsyslog after changes
systemctl restart rsyslog
```

---

## 3. journald (systemd Journal)

```bash
# === VIEWING LOGS ===
journalctl                            # All logs
journalctl -f                         # Follow (like tail -f)
journalctl -b                         # Since last boot
journalctl -b -1                      # Previous boot
journalctl --since "2024-01-01"       # Since date
journalctl --since "1 hour ago"       # Relative time
journalctl -p err                     # Only errors and above
journalctl -p crit                    # Only critical and above

# === FILTER BY UNIT ===
journalctl -u sshd                    # SSH daemon logs
journalctl -u nginx                   # Nginx logs
journalctl -u sshd --since "today"    # SSH logs today

# === FILTER BY FIELDS ===
journalctl _UID=1001                  # Specific user
journalctl _PID=1234                  # Specific process
journalctl _COMM=sudo                 # sudo commands

# === OUTPUT FORMATS ===
journalctl -o json                    # JSON output
journalctl -o json-pretty             # Pretty JSON
journalctl -o verbose                 # All fields
journalctl -o short-iso               # ISO timestamps

# === DISK USAGE ===
journalctl --disk-usage               # Check journal size
journalctl --vacuum-size=500M         # Limit to 500MB
journalctl --vacuum-time=30d          # Keep only 30 days
```

---

## 4. Making journald Persistent

```bash
# By default, journald stores logs in /run/log/journal/ (volatile — lost on reboot!)

# Make persistent:
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal

# Or edit /etc/systemd/journald.conf:
[Journal]
Storage=persistent              # auto, persistent, volatile, none
Compress=yes
SystemMaxUse=1G                 # Max disk usage
SystemMaxFileSize=100M          # Max per-file size
MaxRetentionSec=90day           # Keep logs for 90 days

# Restart
systemctl restart systemd-journald
```

---

## 5. Security-Focused Log Queries

```bash
# Failed SSH logins
journalctl -u sshd | grep "Failed password"

# Successful logins
journalctl -u sshd | grep "Accepted"

# All sudo commands
journalctl _COMM=sudo

# Root logins
journalctl _UID=0 -u sshd

# Service failures
journalctl -p err -b                  # All errors since boot

# Kernel security messages
journalctl -k | grep -i "segfault\|apparmor\|selinux\|audit"
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **rsyslog** | Traditional text-based logging to /var/log/ |
| **journald** | systemd binary structured journal |
| **Persistence** | journald volatile by default — make persistent |
| **journalctl** | Query tool for systemd journal |
| **Forwarding** | rsyslog can forward to remote SIEM |
| **Security** | Monitor auth.log for failed/successful logins |

---

## Quick Revision Questions

1. **What is the difference between rsyslog and journald?**
2. **How do you make journald logs persistent?**
3. **What command shows only SSH-related logs?**
4. **How do you forward logs to a remote server with rsyslog?**
5. **What does `journalctl -p err` show?**

---

[Next: Important Log Files →](02-important-log-files.md)

---

*Unit 3 - Topic 1 of 6 | [Back to README](../README.md)*
