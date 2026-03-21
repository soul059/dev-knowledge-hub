# Auditd Configuration

## Unit 3 - Topic 3 | Linux Logging and Auditing

---

## Overview

**auditd** is the Linux Audit Framework that provides detailed system call-level auditing. It goes far beyond syslog — tracking **file access**, **system calls**, **user commands**, **privilege escalation**, and **security-relevant events**. auditd is essential for **compliance** (PCI DSS, HIPAA), **forensics**, and **insider threat detection**.

---

## 1. Audit Framework Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              LINUX AUDIT FRAMEWORK                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────┐                                          │
│  │ Audit Rules    │ (/etc/audit/rules.d/*.rules)            │
│  └───────┬────────┘                                          │
│          │ loaded into                                       │
│  ┌───────▼────────┐                                          │
│  │ Kernel Audit   │ (intercepts system calls)               │
│  │ Subsystem      │                                          │
│  └───────┬────────┘                                          │
│          │ events                                            │
│  ┌───────▼────────┐                                          │
│  │   auditd       │ (user-space daemon)                     │
│  │   daemon       │                                          │
│  └───────┬────────┘                                          │
│          │ writes to                                         │
│  ┌───────▼────────┐                                          │
│  │ /var/log/audit/ │                                        │
│  │ audit.log       │                                        │
│  └─────────────────┘                                         │
│                                                              │
│  Tools: auditctl, ausearch, aureport, autrace               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. auditd Configuration

```bash
# Install auditd
apt install auditd                    # Debian/Ubuntu
yum install audit                     # RHEL/CentOS

# Main config: /etc/audit/auditd.conf
log_file = /var/log/audit/audit.log
log_format = ENRICHED
max_log_file = 50                     # 50 MB per log file
max_log_file_action = ROTATE
num_logs = 10                         # Keep 10 rotated files
space_left_action = SYSLOG
admin_space_left_action = HALT        # Stop if disk full

# Start and enable
systemctl enable --now auditd
```

---

## 3. Writing Audit Rules

```bash
# RULE TYPES:
# -w = file/directory watch
# -a = system call audit
# -k = key tag for searching

# === FILE WATCHES ===
# Monitor password file changes
auditctl -w /etc/passwd -p wa -k identity
auditctl -w /etc/shadow -p wa -k identity
auditctl -w /etc/group -p wa -k identity
auditctl -w /etc/sudoers -p wa -k sudoers_changes

# Permission flags: r=read, w=write, x=execute, a=attribute change

# Monitor SSH config
auditctl -w /etc/ssh/sshd_config -p wa -k ssh_config

# Monitor cron (persistence detection)
auditctl -w /etc/crontab -p wa -k cron_changes
auditctl -w /var/spool/cron/ -p wa -k cron_changes

# Monitor log tampering
auditctl -w /var/log/auth.log -p wa -k log_tampering

# === SYSTEM CALL AUDITING ===
# Monitor user/group creation
auditctl -a always,exit -F arch=b64 -S execve -F path=/usr/sbin/useradd -k user_creation
auditctl -a always,exit -F arch=b64 -S execve -F path=/usr/sbin/usermod -k user_modification

# Monitor privilege escalation
auditctl -a always,exit -F arch=b64 -S execve -F euid=0 -F auid>=1000 -k privilege_escalation

# Monitor network connections
auditctl -a always,exit -F arch=b64 -S connect -k network_connect

# Monitor module loading (rootkit detection)
auditctl -a always,exit -F arch=b64 -S init_module -S finit_module -k module_load

# === MAKE RULES PERSISTENT ===
# Save rules to /etc/audit/rules.d/security.rules
# Same syntax but without 'auditctl'
# -w /etc/passwd -p wa -k identity
# -w /etc/shadow -p wa -k identity
# Then load:
augenrules --load
```

---

## 4. Searching Audit Logs

```bash
# === ausearch — Search audit logs ===
# Search by key
ausearch -k identity                   # All identity-related events
ausearch -k ssh_config                 # SSH config changes

# Search by time
ausearch --start today
ausearch --start "03/14/2024" --end "03/15/2024"

# Search by user
ausearch -ua 1001                      # Events for UID 1001
ausearch -ua alice                     # Events for user alice

# Search by event type
ausearch -m USER_LOGIN                 # Login events
ausearch -m EXECVE                     # Command execution
ausearch -m USER_CMD                   # sudo commands

# Search for failed events
ausearch --success no

# === aureport — Generate reports ===
aureport --summary                     # Overall summary
aureport --login                       # Login report
aureport --auth                        # Authentication report
aureport --failed                      # All failed events
aureport --file                        # File access report
aureport --executable                  # Executed commands
aureport --key                         # Events by key
```

---

## 5. CIS Benchmark Audit Rules

```bash
# /etc/audit/rules.d/cis.rules
# Based on CIS Benchmark recommendations

# Date and time changes
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -k time-change

# User/group modification
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/gshadow -p wa -k identity

# Network configuration changes
-a always,exit -F arch=b64 -S sethostname -S setdomainname -k system-locale
-w /etc/hostname -p wa -k system-locale
-w /etc/hosts -p wa -k system-locale

# Login and logout
-w /var/log/lastlog -p wa -k logins
-w /var/run/faillock/ -p wa -k logins

# Session initiation
-w /var/run/utmp -p wa -k session
-w /var/log/wtmp -p wa -k logins
-w /var/log/btmp -p wa -k logins

# Sudo usage
-w /etc/sudoers -p wa -k scope
-w /etc/sudoers.d/ -p wa -k scope

# Make rules immutable (can't be changed without reboot)
-e 2
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **auditd** | System call-level auditing daemon |
| **File watches** | `-w /path -p wa -k tag` |
| **System calls** | `-a always,exit -S syscall` |
| **ausearch** | Search audit logs by key, time, user |
| **aureport** | Generate summary reports |
| **Immutable** | `-e 2` prevents rule changes without reboot |

---

## Quick Revision Questions

1. **What does auditd monitor that syslog does not?**
2. **Write an audit rule to monitor changes to /etc/passwd.**
3. **How do you search audit logs for SSH config changes?**
4. **What does `-e 2` do in audit rules?**
5. **What tool generates summary reports from audit logs?**

---

[← Previous: Important Log Files](02-important-log-files.md) | [Next: Log Analysis →](04-log-analysis.md)

---

*Unit 3 - Topic 3 of 6 | [Back to README](../README.md)*
