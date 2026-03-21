# Important Log Files

## Unit 3 - Topic 2 | Linux Logging and Auditing

---

## Overview

Linux stores logs in **/var/log/** as text files. Each log file serves a specific purpose — from authentication to kernel messages. For security professionals, knowing **which log to check** for specific events is critical for forensics, incident response, and compliance.

---

## 1. Essential Security Log Files

| Log File | Content | Security Use |
|----------|---------|-------------|
| `/var/log/auth.log` | Auth events (Debian/Ubuntu) | Login attempts, sudo, SSH |
| `/var/log/secure` | Auth events (RHEL/CentOS) | Same as auth.log |
| `/var/log/syslog` | General system messages | Service events, errors |
| `/var/log/messages` | General messages (RHEL) | System events |
| `/var/log/kern.log` | Kernel messages | Kernel exploits, hardware |
| `/var/log/faillog` | Failed login attempts | Brute force detection |
| `/var/log/lastlog` | Last login per user | Unusual login patterns |
| `/var/log/wtmp` | Login/logout history (binary) | `last` command |
| `/var/log/btmp` | Failed logins (binary) | `lastb` command |
| `/var/log/audit/audit.log` | Auditd events | Detailed system auditing |

---

## 2. Application-Specific Logs

| Log File | Application | Security Use |
|----------|-------------|-------------|
| `/var/log/apache2/access.log` | Apache web server | Web attack detection |
| `/var/log/apache2/error.log` | Apache errors | Application errors |
| `/var/log/nginx/access.log` | Nginx web server | Web traffic analysis |
| `/var/log/mysql/error.log` | MySQL/MariaDB | Database errors |
| `/var/log/mail.log` | Mail server | Email abuse, spam |
| `/var/log/cron.log` | Cron daemon | Scheduled task execution |
| `/var/log/dpkg.log` | Package manager | Software changes |
| `/var/log/apt/history.log` | APT operations | Package installations |

---

## 3. Reading Security Logs

```bash
# === AUTHENTICATION ANALYSIS ===

# Failed SSH logins
grep "Failed password" /var/log/auth.log | tail -20

# Successful logins
grep "Accepted" /var/log/auth.log | tail -20

# All sudo commands
grep "sudo:" /var/log/auth.log

# New user/group creation
grep -E "useradd|groupadd" /var/log/auth.log

# Account lockouts
grep "pam_faillock" /var/log/auth.log

# === LOGIN HISTORY ===
last                                # Login history (from wtmp)
last -f /var/log/wtmp               # Explicit file
lastb                               # Failed logins (from btmp)
lastlog                             # Last login per user

# === SYSTEM EVENTS ===
# Service start/stop
grep "systemd" /var/log/syslog | grep -E "Start|Stop"

# Kernel security events
grep -i "segfault\|oom-killer\|apparmor" /var/log/kern.log

# Package installations (potential backdoor)
grep " install " /var/log/dpkg.log | tail -20

# Cron job execution
grep CRON /var/log/syslog
```

---

## 4. Binary Log Files

| File | Tool to Read | Purpose |
|------|-------------|---------|
| `/var/log/wtmp` | `last` | Login/logout records |
| `/var/log/btmp` | `lastb` | Failed login records |
| `/var/log/lastlog` | `lastlog` | Most recent login per user |
| `/var/log/faillog` | `faillog -a` | Failed login summary |
| `/var/run/utmp` | `who` / `w` | Currently logged in |

```bash
# Currently logged in users
who
w

# Login history (last 20 entries)
last -20

# Failed login attempts
lastb -20

# Last login for all users
lastlog

# Users who never logged in
lastlog | grep "Never logged in"
```

---

## 5. Log Rotation

```bash
# logrotate manages log file rotation
# Config: /etc/logrotate.conf and /etc/logrotate.d/

# Example: /etc/logrotate.d/auth
/var/log/auth.log {
    weekly                  # Rotate weekly
    rotate 12               # Keep 12 weeks
    compress                # Compress old logs
    delaycompress           # Compress after 1 rotation
    missingok               # Don't error if missing
    notifempty              # Don't rotate if empty
    create 0640 root adm    # Permissions for new file
}

# Force rotation
logrotate -f /etc/logrotate.conf

# SECURITY NOTE:
# Attackers may try to delete or rotate logs to cover tracks
# Forward logs to remote SIEM to prevent tampering
```

---

## 6. Anti-Tampering

| Technique | Description |
|-----------|-------------|
| **Remote logging** | Forward to SIEM — attacker can't delete remote copies |
| **Append-only** | `chattr +a /var/log/auth.log` — only append, no modify |
| **Log integrity** | Hash log files periodically for verification |
| **Centralized logging** | All servers send to central log server |
| **File permissions** | Restrict write access to logging daemon only |

```bash
# Make log file append-only (requires root to undo)
chattr +a /var/log/auth.log

# Verify attribute
lsattr /var/log/auth.log
# -----a--------e--- /var/log/auth.log
```

---

## Summary Table

| Log | Key Content |
|-----|------------|
| **auth.log / secure** | Login, sudo, SSH, authentication |
| **syslog / messages** | General system events |
| **kern.log** | Kernel messages |
| **wtmp** | Login history (binary → `last`) |
| **btmp** | Failed logins (binary → `lastb`) |
| **audit.log** | Detailed auditd events |

---

## Quick Revision Questions

1. **Which log file shows failed SSH login attempts?**
2. **What command reads the binary /var/log/wtmp file?**
3. **How can you make a log file append-only?**
4. **Why should logs be forwarded to a remote server?**
5. **What log shows package installations?**

---

[← Previous: Syslog and journald](01-syslog-and-journald.md) | [Next: Auditd Configuration →](03-auditd-configuration.md)

---

*Unit 3 - Topic 2 of 6 | [Back to README](../README.md)*
