# Chapter 1: User Management

[← Previous: Finding Files](../02-Command-Line-Essentials/05-finding-files.md) | [Back to README](../README.md) | [Next: Group Management →](02-group-management.md)

---

## Overview

User management is a foundational system administration task. Every process runs under a user account, and every file has an owner. As a DevOps engineer, you'll create service accounts for applications, manage developer access, and enforce security through proper user configuration.

---

## 1.1 User Architecture in Linux

```
┌──────────────────────────────────────────────────────────────┐
│                    USER MANAGEMENT FILES                      │
│                                                               │
│  ┌──────────────────┐  ┌──────────────────┐                  │
│  │   /etc/passwd    │  │   /etc/shadow    │                  │
│  │  (User accounts) │  │   (Passwords)    │                  │
│  │                  │  │                  │                  │
│  │  username:x:     │  │  username:$6$... │                  │
│  │  UID:GID:        │  │  :last:min:max:  │                  │
│  │  comment:home:   │  │  warn:inactive:  │                  │
│  │  shell           │  │  expire          │                  │
│  └──────────────────┘  └──────────────────┘                  │
│                                                               │
│  ┌──────────────────┐  ┌──────────────────┐                  │
│  │   /etc/group     │  │  /etc/gshadow    │                  │
│  │  (Groups)        │  │  (Group pass)    │                  │
│  └──────────────────┘  └──────────────────┘                  │
│                                                               │
│  ┌──────────────────┐                                        │
│  │   /etc/skel/     │  ← Template for new home directories  │
│  │  .bashrc         │                                        │
│  │  .profile        │                                        │
│  │  .bash_logout    │                                        │
│  └──────────────────┘                                        │
└──────────────────────────────────────────────────────────────┘
```

### Understanding `/etc/passwd`

```
root:x:0:0:root:/root:/bin/bash
│    │ │ │ │     │     │
│    │ │ │ │     │     └── Login shell
│    │ │ │ │     └── Home directory
│    │ │ │ └── GECOS (comment/full name)
│    │ │ └── Primary GID
│    │ └── UID
│    └── Password placeholder (actual in /etc/shadow)
└── Username
```

### Understanding `/etc/shadow`

```
devops:$6$rounds=5000$salt$hash:19400:0:99999:7:::
│      │                        │     │ │     │
│      │                        │     │ │     └── Warning days before expiry
│      │                        │     │ └── Maximum password age
│      │                        │     └── Minimum password age
│      │                        └── Last password change (days since epoch)
│      └── Encrypted password ($6$ = SHA-512)
└── Username
```

### UID Ranges

```
┌──────────────────────────────────────────────────┐
│              UID ALLOCATION                       │
├──────────────┬───────────────────────────────────┤
│ UID 0        │ root (superuser)                  │
│ UID 1-999    │ System/service accounts            │
│ UID 1000+    │ Regular users                      │
│ UID 65534    │ nobody (unprivileged)              │
└──────────────┴───────────────────────────────────┘
```

---

## 1.2 Creating Users — `useradd`

```bash
# Basic user creation
sudo useradd devops

# Create with home directory (recommended)
sudo useradd -m devops

# Create with all common options
sudo useradd -m -d /home/devops -s /bin/bash -c "DevOps Engineer" -G docker,sudo devops

# Flags explained:
# -m          Create home directory
# -d PATH     Set home directory path
# -s SHELL    Set login shell
# -c COMMENT  Set description/full name
# -G GROUPS   Add to supplementary groups
# -g GROUP    Set primary group
# -u UID      Set specific UID
# -e DATE     Account expiration date (YYYY-MM-DD)
# -r          Create system account (UID < 1000, no home)

# Set password
sudo passwd devops
# Enter new UNIX password: ********

# Create system account (for services)
sudo useradd -r -s /usr/sbin/nologin -d /nonexistent myservice

# Non-interactive password setting (for scripts)
echo "devops:SecureP@ss123" | sudo chpasswd

# Create user with encrypted password
sudo useradd -m -p $(openssl passwd -6 "password") devops
```

### Default User Settings

```bash
# View default settings
useradd -D
# GROUP=100
# HOME=/home
# INACTIVE=-1
# EXPIRE=
# SHELL=/bin/sh
# SKEL=/etc/skel
# CREATE_MAIL_SPOOL=yes

# Change defaults
sudo useradd -D -s /bin/bash     # Default shell to bash

# View /etc/login.defs (system-wide defaults)
grep -v '^#' /etc/login.defs | grep -v '^$'
# PASS_MAX_DAYS   99999
# PASS_MIN_DAYS   0
# PASS_WARN_AGE   7
# UID_MIN         1000
# UID_MAX         60000
```

---

## 1.3 Modifying Users — `usermod`

```bash
# Change username
sudo usermod -l newname oldname

# Change home directory (and move contents)
sudo usermod -d /home/newhome -m username

# Change shell
sudo usermod -s /bin/zsh username

# Change comment (full name)
sudo usermod -c "Senior DevOps Engineer" devops

# Add to supplementary group (APPEND!)
sudo usermod -aG docker devops       # ← -a is crucial!
sudo usermod -aG sudo devops

# DANGER: Without -a, replaces all supplementary groups!
# sudo usermod -G docker devops      # ← WRONG: removes all other groups

# Lock account (disable login)
sudo usermod -L username
# Adds ! before password hash in /etc/shadow

# Unlock account
sudo usermod -U username

# Set account expiration
sudo usermod -e 2026-12-31 contractor

# Change UID
sudo usermod -u 2000 username
```

---

## 1.4 Deleting Users — `userdel`

```bash
# Delete user (keep home directory)
sudo userdel username

# Delete user AND home directory
sudo userdel -r username

# Force delete (even if user is logged in)
sudo userdel -rf username

# Before deleting, check for user's files
find / -user username 2>/dev/null

# Before deleting, check running processes
ps -u username
```

---

## 1.5 User Information Commands

```bash
# Current user info
whoami                          # Current username
id                              # UID, GID, groups
id devops                       # Info for specific user
id -u devops                    # UID only
id -g devops                    # Primary GID only
id -G devops                    # All group IDs
id -Gn devops                   # All group names

# Who is logged in
who                             # Current logins
w                               # Logins with activity
last                            # Login history
last -n 10                      # Last 10 logins
lastlog                         # Last login for all users
lastb                           # Failed login attempts

# List all users
cat /etc/passwd | cut -d: -f1
getent passwd                   # Works with LDAP/NIS too
getent passwd devops            # Specific user

# Check password status
sudo passwd -S devops
# devops P 02/20/2026 0 99999 7 -1
# P = Password set, L = Locked, NP = No Password

# Check account aging
sudo chage -l devops
# Last password change                 : Feb 20, 2026
# Password expires                     : never
# Password inactive                    : never
# Account expires                      : never
# Minimum number of days between pw change : 0
# Maximum number of days between pw change : 99999
# Number of days of warning before pw expires : 7
```

---

## 1.6 Password Policies

```bash
# Set password aging with chage
sudo chage -M 90 devops          # Max 90 days between changes
sudo chage -m 7 devops           # Min 7 days between changes
sudo chage -W 14 devops          # Warn 14 days before expiry
sudo chage -I 30 devops          # Inactive (lock) 30 days after expiry
sudo chage -E 2026-12-31 devops  # Account expires on date

# Force password change at next login
sudo chage -d 0 devops

# PAM password quality (Ubuntu)
sudo apt install libpam-pwquality
# Edit: /etc/security/pwquality.conf
# minlen = 12
# dcredit = -1    (require digit)
# ucredit = -1    (require uppercase)
# lcredit = -1    (require lowercase)
# ocredit = -1    (require special char)
```

---

## 1.7 Real-World DevOps Scenario

### Scenario: Onboarding a New Developer

```bash
#!/bin/bash
# onboard-user.sh — Automated user onboarding

set -euo pipefail

USERNAME=$1
FULLNAME=$2
SSH_KEY=$3

echo "Creating user: $USERNAME ($FULLNAME)"

# Create user with home directory and bash shell
sudo useradd -m -s /bin/bash -c "$FULLNAME" "$USERNAME"

# Add to necessary groups
sudo usermod -aG docker,developers "$USERNAME"

# Set temporary password and force change at first login
TEMP_PASS=$(openssl rand -base64 12)
echo "$USERNAME:$TEMP_PASS" | sudo chpasswd
sudo chage -d 0 "$USERNAME"

# Set up SSH key
sudo mkdir -p /home/$USERNAME/.ssh
echo "$SSH_KEY" | sudo tee /home/$USERNAME/.ssh/authorized_keys
sudo chmod 700 /home/$USERNAME/.ssh
sudo chmod 600 /home/$USERNAME/.ssh/authorized_keys
sudo chown -R $USERNAME:$USERNAME /home/$USERNAME/.ssh

# Set password policy
sudo chage -M 90 -W 14 -m 7 "$USERNAME"

echo "User $USERNAME created successfully"
echo "Temporary password: $TEMP_PASS"
echo "User must change password on first login"

# Usage: ./onboard-user.sh jdoe "John Doe" "ssh-rsa AAAA...key..."
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|---------|
| `useradd: user already exists` | Duplicate username | Check `id username`, use different name |
| `usermod: group doesn't exist` | Group not created | Create group first: `groupadd` |
| User can't login | Account locked or no shell | Check `passwd -S`, `usermod -U` |
| `su` fails for user | No password set | `sudo passwd username` |
| Groups not applied after `usermod` | Session not refreshed | User must log out and back in |
| Files orphaned after `userdel` | UID mismatched | `find / -nouser`, reassign with `chown` |

---

## Summary Table

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `useradd` | Create user | `-m`, `-s`, `-G`, `-c`, `-r` |
| `usermod` | Modify user | `-aG`, `-L`, `-U`, `-s`, `-d` |
| `userdel` | Delete user | `-r` (remove home) |
| `passwd` | Set/change password | `-S` (status), `-l` (lock) |
| `chage` | Password aging policies | `-M`, `-m`, `-W`, `-d 0` |
| `id` | Show user/group info | `-u`, `-g`, `-Gn` |
| `who`/`w` | Show logged-in users | — |
| `last` | Login history | `-n` (limit) |
| `chpasswd` | Batch password setting | Reads from stdin |

---

## Quick Revision Questions

1. **What files are involved in user management?** Explain the purpose of each.
2. **What is the difference between `useradd` and `adduser`?** (Hint: distribution-specific)
3. **Why is the `-a` flag critical when using `usermod -G`?** What happens without it?
4. **How do you create a service account that cannot log in interactively?** Write the command.
5. **How would you force a user to change their password at next login?**
6. **Explain the UID ranges in Linux.** What UIDs do regular users get?

---

[← Previous: Finding Files](../02-Command-Line-Essentials/05-finding-files.md) | [Back to README](../README.md) | [Next: Group Management →](02-group-management.md)
