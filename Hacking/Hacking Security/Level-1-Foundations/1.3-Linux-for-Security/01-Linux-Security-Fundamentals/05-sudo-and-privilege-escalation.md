# sudo and Privilege Escalation Prevention

## Unit 1 - Topic 5 | Linux Security Fundamentals

---

## Overview

**sudo** (superuser do) allows permitted users to execute commands as root or another user. It's the primary mechanism for **controlled privilege escalation** on Linux. However, misconfigured sudo is one of the most common **privilege escalation vectors** exploited by attackers and penetration testers.

---

## 1. How sudo Works

```
┌──────────────────────────────────────────────────────────────┐
│              SUDO WORKFLOW                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  User types: sudo apt update                                 │
│       │                                                      │
│       ▼                                                      │
│  1. Check /etc/sudoers (and /etc/sudoers.d/*)               │
│       │                                                      │
│       ▼                                                      │
│  2. Is user authorized for this command?                     │
│       │                                                      │
│     NO → "user is not in the sudoers file"                  │
│     YES ▼                                                    │
│  3. Prompt for USER's password (not root's)                  │
│       │                                                      │
│       ▼                                                      │
│  4. Execute command as root                                  │
│       │                                                      │
│       ▼                                                      │
│  5. Log action to /var/log/auth.log                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. sudoers File Syntax

```bash
# /etc/sudoers — ALWAYS edit with visudo (syntax checking)
# Format: WHO WHERE=(AS_WHOM) WHAT

# Full root access
alice   ALL=(ALL:ALL) ALL

# Explanation:
# alice     → the user
# ALL       → from any host
# (ALL:ALL) → as any user:group
# ALL       → any command

# COMMON CONFIGURATIONS:

# Allow without password
alice   ALL=(ALL) NOPASSWD: ALL          # ⚠️ Dangerous!

# Specific commands only
bob     ALL=(ALL) /usr/bin/systemctl restart apache2
bob     ALL=(ALL) /usr/bin/apt update, /usr/bin/apt upgrade

# Group-based (% prefix)
%sudo   ALL=(ALL:ALL) ALL                # sudo group gets full access
%devops ALL=(ALL) /usr/bin/docker, /usr/bin/kubectl

# Deny specific commands
alice   ALL=(ALL) ALL, !/bin/su, !/bin/bash, !/usr/bin/passwd root

# Command aliases
Cmnd_Alias NETWORKING = /sbin/iptables, /sbin/ifconfig, /bin/ping
alice   ALL=(ALL) NETWORKING
```

---

## 3. Dangerous sudo Misconfigurations

| Misconfiguration | Why Dangerous | Exploitation |
|-----------------|---------------|-------------|
| `NOPASSWD: ALL` | No password needed for root | Instant root access |
| `sudo vi` / `sudo vim` | Editor can spawn shell | `:!bash` in vi → root shell |
| `sudo find` | Find can execute commands | `sudo find / -exec /bin/bash \;` |
| `sudo python` | Python can spawn shell | `sudo python -c 'import os; os.system("/bin/bash")'` |
| `sudo less` / `sudo more` | Pagers can execute commands | `!bash` while in less → root shell |
| `sudo awk` | Awk can execute system commands | `sudo awk 'BEGIN {system("/bin/bash")}'` |
| `sudo nmap` | Interactive mode | `sudo nmap --interactive` then `!sh` |
| `sudo env` | Environment manipulation | `sudo env /bin/bash` |
| `sudo tar` | Archive extraction with commands | `sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash` |

```bash
# GTFOBins — curated list of sudo escape techniques
# https://gtfobins.github.io/

# CHECK what you can run with sudo:
sudo -l

# Example output showing exploitable config:
# (ALL) NOPASSWD: /usr/bin/vim
# EXPLOIT: sudo vim → :!bash → ROOT SHELL
```

---

## 4. Hardening sudo

```bash
# HARDENING CHECKLIST:

# 1. Use visudo (syntax validation)
visudo

# 2. Restrict commands — NEVER give ALL unless necessary
bob ALL=(ALL) /usr/bin/systemctl restart nginx

# 3. Use FULL PATHS in sudoers
# BAD:  bob ALL=(ALL) systemctl
# GOOD: bob ALL=(ALL) /usr/bin/systemctl restart nginx

# 4. Block shell escapes — deny known dangerous binaries
alice ALL=(ALL) ALL, !/usr/bin/vim, !/usr/bin/vi, !/usr/bin/python3, \
    !/usr/bin/less, !/usr/bin/more, !/bin/bash, !/bin/sh, !/usr/bin/find

# 5. Require password (avoid NOPASSWD)
bob ALL=(ALL) /usr/bin/systemctl restart nginx  # requires password

# 6. Set timestamp timeout (default 15 min)
Defaults timestamp_timeout=5     # Re-prompt after 5 minutes
Defaults timestamp_timeout=0     # Always prompt

# 7. Log all sudo commands
Defaults logfile="/var/log/sudo.log"
Defaults log_input, log_output
Defaults iolog_dir="/var/log/sudo-io/%{user}"

# 8. Use sudoers.d directory for modular config
# /etc/sudoers.d/alice
alice ALL=(ALL) /usr/bin/systemctl restart nginx

# 9. Restrict sudo to specific groups
# Only users in 'sudo' group can use sudo
```

---

## 5. Privilege Escalation Prevention

```
COMMON LINUX PRIVESC VECTORS:
┌──────────────────────────────────────────────────────────────┐
│ 1. SUID/SGID binaries           → find / -perm -4000       │
│ 2. Misconfigured sudo           → sudo -l                   │
│ 3. Writable /etc/passwd         → ls -la /etc/passwd        │
│ 4. Cron jobs (writable scripts) → cat /etc/crontab          │
│ 5. Kernel exploits              → uname -r (check version)  │
│ 6. Writable PATH directories    → echo $PATH                │
│ 7. Docker group membership      → id (groups)               │
│ 8. Capabilities                 → getcap -r / 2>/dev/null   │
│ 9. NFS no_root_squash           → cat /etc/exports          │
│ 10. Weak file permissions       → find / -writable          │
└──────────────────────────────────────────────────────────────┘

PREVENTION:
• Audit SUID binaries regularly
• Use principle of least privilege for sudo
• Set proper file permissions
• Keep kernel updated
• Monitor cron for unauthorized changes
• Never add users to docker group unnecessarily
• Use SELinux/AppArmor for mandatory access control
```

---

## 6. Enumeration Tools (for auditing)

| Tool | Purpose |
|------|---------|
| **LinPEAS** | Automated Linux privilege escalation scanner |
| **LinEnum** | Linux enumeration script |
| **linux-exploit-suggester** | Suggests kernel exploits based on version |
| **pspy** | Monitors processes without root (finds cron jobs) |
| **GTFOBins** | Database of sudo/SUID escape techniques |

```bash
# Run LinPEAS for security audit
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Quick manual enumeration
sudo -l                            # Check sudo privileges
find / -perm -4000 2>/dev/null     # SUID binaries
cat /etc/crontab                   # Cron jobs
getcap -r / 2>/dev/null            # Linux capabilities
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **sudo** | Controlled privilege escalation |
| **visudo** | Always use for editing sudoers |
| **NOPASSWD** | Avoid — removes authentication barrier |
| **Shell escape** | Many commands (vi, find, python) can spawn shells |
| **GTFOBins** | Reference for sudo/SUID exploitation |
| **Prevention** | Least privilege, full paths, audit regularly |

---

## Quick Revision Questions

1. **How do you safely edit the sudoers file?**
2. **What does `sudo -l` show and why is it important for attackers?**
3. **How can `sudo vim` be exploited for privilege escalation?**
4. **Name 5 common Linux privilege escalation vectors.**
5. **What is GTFOBins?**

---

[← Previous: User and Group Management](04-user-and-group-management.md)

---

*Unit 1 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Linux Hardening →](../02-Linux-Hardening/01-minimal-installation.md)*
