# User and Group Management

## Unit 1 - Topic 4 | Linux Security Fundamentals

---

## Overview

Proper **user and group management** is fundamental to Linux security. Every process runs under a user context, and access to resources is controlled through user/group permissions. Mismanaged accounts are a primary vector for **unauthorized access**, **privilege escalation**, and **lateral movement**.

---

## 1. Key User Files

```
┌──────────────────────────────────────────────────────────────┐
│              USER MANAGEMENT FILES                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  /etc/passwd — User account information                      │
│  username:x:UID:GID:GECOS:home_dir:shell                    │
│  root:x:0:0:root:/root:/bin/bash                            │
│  alice:x:1001:1001:Alice:/home/alice:/bin/bash               │
│  www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin       │
│                                                              │
│  /etc/shadow — Encrypted passwords (root only)              │
│  username:$hash:lastchg:min:max:warn:inactive:expire        │
│  alice:$6$salt$hash:19234:0:99999:7:::                      │
│  │      │             │   │  │    │                         │
│  │      │             │   │  │    └─ Days before warning    │
│  │      │             │   │  └───── Max days between change │
│  │      │             │   └──────── Min days between change │
│  │      │             └──────────── Last change (epoch days)│
│  │      └────────────────────────── Password hash           │
│  └───────────────────────────────── Username                │
│                                                              │
│  /etc/group — Group information                              │
│  groupname:x:GID:member1,member2                            │
│  sudo:x:27:alice,bob                                        │
│                                                              │
│  /etc/gshadow — Group passwords (rarely used)               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. User Management Commands

```bash
# CREATE user
useradd -m -s /bin/bash -G sudo alice     # Create with home dir, bash, sudo group
useradd -r -s /usr/sbin/nologin svcacct   # Service account (no login)

# MODIFY user
usermod -aG docker alice                   # Add to docker group
usermod -s /usr/sbin/nologin bob           # Disable login shell
usermod -L bob                             # Lock account
usermod -U bob                             # Unlock account
usermod -e 2025-12-31 contractor           # Set account expiry

# DELETE user
userdel bob                                # Delete user (keep home)
userdel -r bob                             # Delete user + home dir

# PASSWORD management
passwd alice                               # Set/change password
passwd -l alice                            # Lock password
passwd -u alice                            # Unlock password
passwd -e alice                            # Force password change on next login
chage -l alice                             # View password aging info
chage -M 90 alice                          # Max 90 days between changes
chage -E 2025-12-31 alice                  # Account expires

# VIEW user info
id alice                                   # UID, GID, groups
whoami                                     # Current user
who                                        # Who is logged in
w                                          # Who is logged in + activity
last                                       # Login history
lastb                                      # Failed login attempts
```

---

## 3. Group Management

```bash
# Group commands
groupadd developers                        # Create group
groupmod -n devs developers                # Rename group
groupdel oldgroup                          # Delete group
gpasswd -a alice developers               # Add user to group
gpasswd -d alice developers               # Remove user from group

# View groups
groups alice                               # Show user's groups
getent group sudo                          # Show members of sudo group
cat /etc/group | grep alice                # All groups containing alice
```

---

## 4. Security Best Practices

| Practice | Implementation |
|----------|---------------|
| **Disable root login** | `PermitRootLogin no` in sshd_config |
| **No shared accounts** | Each person gets unique account |
| **Service accounts** | Use `/usr/sbin/nologin` shell |
| **Lock inactive accounts** | `usermod -L username` |
| **Set account expiry** | For contractors/temporary users |
| **Password policies** | Min length, complexity, aging via PAM |
| **Audit UID 0** | Only root should have UID 0 |
| **Remove unnecessary users** | Default accounts (games, ftp, etc.) |
| **Principle of least privilege** | Minimum group membership |

```bash
# SECURITY AUDITS

# Find all UID 0 accounts (should only be root)
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Find accounts with no password
awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow

# Find accounts with login shells
grep -v 'nologin\|false' /etc/passwd

# Find users in sudo/admin groups
getent group sudo wheel admin

# Check for accounts that never expire
awk -F: '$8 == "" {print $1}' /etc/shadow

# Find recently created users
find /home -maxdepth 1 -mtime -7 -type d
```

---

## 5. Password Hash Formats

| Prefix | Algorithm | Security |
|--------|-----------|:--------:|
| `$1$` | MD5 | ❌ Weak |
| `$5$` | SHA-256 | ✅ Acceptable |
| `$6$` | SHA-512 | ✅ Strong (default) |
| `$y$` | yescrypt | ✅ Modern (newest) |
| `$2b$` | bcrypt | ✅ Strong |

```bash
# Check which hash algorithm is used
head -1 /etc/shadow

# View password hashing configuration
cat /etc/login.defs | grep ENCRYPT_METHOD
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| `/etc/passwd` | User info — readable by all (no passwords) |
| `/etc/shadow` | Password hashes — root only |
| **Service accounts** | Use `nologin` shell, no password |
| **Account locking** | `usermod -L` or `passwd -l` |
| **Password aging** | `chage` to enforce rotation |
| **UID 0 audit** | Only root should have UID 0 |

---

## Quick Revision Questions

1. **What is the difference between `/etc/passwd` and `/etc/shadow`?**
2. **How do you create a service account with no login ability?**
3. **How would you find all users with UID 0?**
4. **What does `usermod -L` do?**
5. **What password hash prefix indicates SHA-512?**

---

[← Previous: File Permissions](03-file-permissions.md) | [Next: sudo and Privilege Escalation Prevention →](05-sudo-and-privilege-escalation.md)

---

*Unit 1 - Topic 4 of 5 | [Back to README](../README.md)*
