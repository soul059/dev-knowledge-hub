# Chapter 6: Sudo and Sudoers

[← Previous: Access Control Lists](05-access-control-lists.md) | [Back to README](../README.md) | [Next: Process Lifecycle →](../04-Process-Management/01-process-lifecycle.md)

---

## Overview

`sudo` ("superuser do") allows authorized users to run commands as root or another user, providing auditable, granular privilege escalation. The `/etc/sudoers` file controls who can do what — making it one of the most security-critical files on any Linux system.

---

## 6.1 How Sudo Works

```
┌───────────────────────────────────────────────────────────┐
│                    SUDO WORKFLOW                           │
│                                                            │
│  User types: sudo apt update                              │
│       │                                                    │
│       ▼                                                    │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │ Authenticate│───→│ Check        │───→│ Execute as   │  │
│  │ (password)  │    │ /etc/sudoers │    │ root (or     │  │
│  │             │    │              │    │ target user) │  │
│  └─────────────┘    └──────────────┘    └──────────────┘  │
│                            │                               │
│                     ┌──────▼──────┐                        │
│                     │ Log action  │                        │
│                     │ /var/log/   │                        │
│                     │ auth.log    │                        │
│                     └─────────────┘                        │
└───────────────────────────────────────────────────────────┘
```

---

## 6.2 Basic Sudo Commands

```bash
# Run command as root
sudo apt update

# Run command as specific user
sudo -u postgres psql

# Open root shell
sudo -i                    # Login shell (root's environment)
sudo -s                    # Non-login shell (keeps your env)
sudo su -                  # Switch to root

# Edit a file with root privileges
sudo -e /etc/hosts         # Uses $EDITOR (safer than sudo vim)
sudoedit /etc/hosts        # Same thing

# Check sudo privileges for current user
sudo -l

# Run last command with sudo
sudo !!

# Reset sudo timestamp (require password again)
sudo -k

# Run without password (if configured)
sudo -n command            # Fails if password needed

# Preserve environment (use carefully)
sudo -E command
```

---

## 6.3 The Sudoers File

```bash
# NEVER edit /etc/sudoers directly! Use visudo:
sudo visudo
# visudo checks syntax before saving — prevents lockout!

# Add drop-in files (recommended):
sudo visudo -f /etc/sudoers.d/devops
```

### Sudoers Syntax

```
┌───────────────────────────────────────────────────────────────┐
│  SUDOERS ENTRY FORMAT:                                        │
│                                                                │
│  user  host=(runas)  commands                                 │
│                                                                │
│  Examples:                                                     │
│  devops  ALL=(ALL)  ALL                                       │
│  │       │    │     │                                          │
│  │       │    │     └── Can run any command                    │
│  │       │    └── Can run as any user                          │
│  │       └── On any host                                       │
│  └── Username                                                  │
│                                                                │
│  %developers  ALL=(ALL)  ALL                                  │
│  ^                                                             │
│  └── Group (prefix with %)                                    │
│                                                                │
│  devops  ALL=(ALL)  NOPASSWD: ALL                             │
│                     ^^^^^^^^^                                  │
│                     └── No password required                   │
└───────────────────────────────────────────────────────────────┘
```

### Common Sudoers Configurations

```bash
# Full admin access
devops ALL=(ALL:ALL) ALL

# Admin without password (automation/CI)
deploy ALL=(ALL) NOPASSWD: ALL

# Limited access — specific commands only
developer ALL=(ALL) /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx

# Group-based access
%devops ALL=(ALL) NOPASSWD: ALL
%developers ALL=(ALL) /usr/bin/docker, /usr/bin/docker-compose

# Command aliases for cleaner configs
Cmnd_Alias SERVICES = /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
Cmnd_Alias NETWORKING = /usr/sbin/iptables, /usr/bin/ss, /usr/sbin/ip
Cmnd_Alias MONITORING = /usr/bin/top, /usr/bin/htop, /usr/bin/journalctl

%devops ALL=(ALL) NOPASSWD: SERVICES, NETWORKING, MONITORING

# Deny specific commands
devops ALL=(ALL) ALL, !/usr/bin/su, !/bin/bash
# Can do everything EXCEPT su and bash as root

# Require password for some, not others
devops ALL=(ALL) NOPASSWD: /usr/bin/systemctl status *, PASSWD: /usr/bin/systemctl restart *
```

### Best Practices for Sudoers

```
┌──────────────────────────────────────────────────────────────┐
│  SUDOERS BEST PRACTICES                                      │
│                                                               │
│  1. Always use visudo to edit                                │
│  2. Use /etc/sudoers.d/ drop-in files                       │
│  3. Grant least privilege — specific commands only           │
│  4. Use groups (%groupname) instead of individual users      │
│  5. Avoid NOPASSWD in production (except automation)         │
│  6. Log and audit sudo usage                                 │
│  7. Never allow 'sudo bash' or 'sudo su' without controls   │
│  8. Test with sudo -l before relying on configs              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.4 Real-World Scenario: Role-Based Access

```bash
# /etc/sudoers.d/web-team
# Web developers can manage nginx and view logs
Cmnd_Alias WEB_CMDS = /usr/bin/systemctl * nginx, /usr/bin/journalctl -u nginx, /usr/bin/tail /var/log/nginx/*
%web-team ALL=(ALL) NOPASSWD: WEB_CMDS

# /etc/sudoers.d/db-team
# Database admins can manage PostgreSQL
Cmnd_Alias DB_CMDS = /usr/bin/systemctl * postgresql, /usr/bin/pg_dump, /usr/bin/psql
%db-team ALL=(ALL) NOPASSWD: DB_CMDS

# /etc/sudoers.d/deploy
# Deploy user for CI/CD (fully automated)
deploy ALL=(ALL) NOPASSWD: ALL
```

---

## 6.5 Sudo vs Su

| Feature | `sudo` | `su` |
|---------|--------|------|
| **Authentication** | User's own password | Target user's password |
| **Granularity** | Per-command control | Full shell access |
| **Auditing** | Logged per command | Harder to trace |
| **Configuration** | `/etc/sudoers` | `/etc/passwd`, `/etc/shadow` |
| **Best Practice** | Preferred for DevOps | Avoid in production |

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `sudo command` | Run as root |
| `sudo -u user cmd` | Run as specific user |
| `sudo -i` | Root login shell |
| `sudo -l` | List allowed commands |
| `sudo -k` | Reset auth timestamp |
| `visudo` | Safely edit sudoers |
| `visudo -f /etc/sudoers.d/file` | Edit drop-in file |

---

## Quick Revision Questions

1. **What is the difference between `sudo -i` and `sudo -s`?**
2. **Why must you always use `visudo` to edit the sudoers file?**
3. **How do you grant a user sudo access for specific commands only?** Write the sudoers entry.
4. **What does `NOPASSWD:` do and when is it appropriate?**
5. **How would you set up role-based sudo access for a web team?** 
6. **What is the difference between `sudo` and `su`?** Which is preferred and why?

---

[← Previous: Access Control Lists](05-access-control-lists.md) | [Back to README](../README.md) | [Next: Process Lifecycle →](../04-Process-Management/01-process-lifecycle.md)
