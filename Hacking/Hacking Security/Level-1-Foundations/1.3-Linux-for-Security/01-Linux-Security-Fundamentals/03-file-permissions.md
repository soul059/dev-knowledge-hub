# File Permissions (Standard and Special)

## Unit 1 - Topic 3 | Linux Security Fundamentals

---

## Overview

Linux file permissions are the **first line of defense** in access control. They determine who can read, write, and execute files and directories. Misconfigurations are one of the most common causes of **privilege escalation** and **data breaches** on Linux systems.

---

## 1. Standard Permissions

```
┌──────────────────────────────────────────────────────────────┐
│              FILE PERMISSION STRUCTURE                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  -rwxr-xr-- 1 alice devs 4096 Mar 14 10:00 script.sh       │
│  │││││││││││                                                 │
│  ││├─┤├─┤├─┤                                                 │
│  ││ │  │  │                                                  │
│  ││ │  │  └── Others (r--)  = 4 (read only)                │
│  ││ │  └───── Group  (r-x)  = 5 (read + execute)           │
│  ││ └──────── Owner  (rwx)  = 7 (read + write + execute)   │
│  │└────────── File type (- = file, d = dir, l = link)       │
│  └─────────── Total: 754                                    │
│                                                              │
│  PERMISSION VALUES:                                          │
│  r = 4 (read)                                               │
│  w = 2 (write)                                              │
│  x = 1 (execute)                                            │
│  - = 0 (none)                                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

| Octal | Binary | Permission | Meaning |
|:-----:|:------:|:----------:|---------|
| 0 | 000 | `---` | No access |
| 1 | 001 | `--x` | Execute only |
| 2 | 010 | `-w-` | Write only |
| 3 | 011 | `-wx` | Write + execute |
| 4 | 100 | `r--` | Read only |
| 5 | 101 | `r-x` | Read + execute |
| 6 | 110 | `rw-` | Read + write |
| 7 | 111 | `rwx` | Full access |

---

## 2. Common Permission Commands

```bash
# View permissions
ls -la /etc/shadow
ls -la /home/user/

# Change permissions (numeric)
chmod 755 script.sh                 # rwxr-xr-x
chmod 644 config.txt                # rw-r--r--
chmod 600 secret.key                # rw------- (owner only)
chmod 700 ~/private/                # rwx------ (owner only dir)

# Change permissions (symbolic)
chmod u+x script.sh                 # Add execute for owner
chmod g-w file.txt                  # Remove write for group
chmod o-rwx sensitive.conf          # Remove all for others
chmod a+r public.txt                # Add read for all

# Change ownership
chown root:root /etc/shadow
chown alice:devs project/
chown -R alice:devs project/        # Recursive

# Default permissions (umask)
umask                               # View current umask
umask 027                           # New files: 640, dirs: 750
umask 077                           # New files: 600, dirs: 700
```

---

## 3. Special Permissions — CRITICAL for Security

### SUID (Set User ID) — `4xxx`

```
When SUID is set on an executable, it runs with the
FILE OWNER'S privileges, not the user who runs it.

Example: /usr/bin/passwd has SUID set
-rwsr-xr-x 1 root root 68208 /usr/bin/passwd
    ^
    s = SUID bit

Normal user runs passwd → it runs as ROOT
(needed to modify /etc/shadow)

⚠️ SECURITY RISK:
If an attacker finds a SUID binary with a vulnerability,
they can escalate to the binary's owner (usually root)!

# Find all SUID files (CRITICAL security check)
find / -type f -perm -4000 -ls 2>/dev/null

# Common legitimate SUID binaries:
/usr/bin/passwd       # Password change
/usr/bin/sudo         # Privilege escalation
/usr/bin/su           # Switch user
/usr/bin/ping         # ICMP (needs raw socket)
/usr/bin/mount        # Mount filesystems
```

### SGID (Set Group ID) — `2xxx`

```
On files:  Runs with file's GROUP privileges
On dirs:   New files inherit directory's group

-rwxr-sr-x 1 root shadow 40000 /usr/bin/expiry
         ^
         s = SGID bit

# Find all SGID files
find / -type f -perm -2000 -ls 2>/dev/null

# SGID on directory (common for shared folders)
chmod 2770 /shared/project/
# New files in /shared/project/ inherit group ownership
```

### Sticky Bit — `1xxx`

```
On directories: Only the file OWNER can delete their files
(even if others have write permission to the directory)

drwxrwxrwt 10 root root 4096 /tmp
          ^
          t = sticky bit

# Set sticky bit
chmod 1777 /tmp
chmod +t /shared/

# Without sticky bit on /tmp:
# Any user could delete any other user's files!
```

---

## 4. Permission Security Checklist

| Check | Command | Expected |
|-------|---------|----------|
| `/etc/shadow` permissions | `ls -la /etc/shadow` | `-rw-r----- root shadow` |
| `/etc/passwd` permissions | `ls -la /etc/passwd` | `-rw-r--r-- root root` |
| `/etc/sudoers` permissions | `ls -la /etc/sudoers` | `-r--r----- root root` |
| World-writable files | `find / -perm -o+w -type f` | Minimal results |
| No-owner files | `find / -nouser -o -nogroup` | Should be empty |
| SUID files | `find / -perm -4000` | Only expected binaries |
| Home directory perms | `ls -ld /home/*` | `drwx------` (700) |

---

## 5. Privilege Escalation via Permissions

```
COMMON MISCONFIGURATIONS:
1. SUID on custom scripts/binaries
   chmod u+s /opt/backup.sh        # Anyone runs as root!

2. World-writable config files
   chmod 777 /etc/cron.d/backup    # Anyone can add cron jobs!

3. Group-writable sensitive files
   chmod 660 /etc/shadow           # Group members can read hashes!

4. Writable /etc/passwd
   If writable, attacker adds a root-level user:
   echo 'hacker:$(openssl passwd -1 pass):0:0::/root:/bin/bash' >> /etc/passwd

5. SUID binary with command injection
   # If /opt/backup (SUID root) calls "tar" without full path:
   export PATH=/tmp:$PATH
   echo '/bin/bash' > /tmp/tar && chmod +x /tmp/tar
   /opt/backup                      # Gets root shell!
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Standard perms** | rwx for owner, group, others (e.g., 755) |
| **SUID** | Runs as file owner — exploit for privilege escalation |
| **SGID** | Runs as file group / inherits group in directories |
| **Sticky bit** | Only owner can delete files in directory |
| **umask** | Default permission mask for new files |
| **Key check** | `find / -perm -4000` — audit all SUID binaries |

---

## Quick Revision Questions

1. **What does permission `755` mean?**
2. **Why is SUID dangerous on custom scripts?**
3. **What is the sticky bit and why is it used on `/tmp`?**
4. **How would you find all SUID binaries on a system?**
5. **What permissions should `/etc/shadow` have?**

---

[← Previous: Security-Relevant Directories](02-security-relevant-directories.md) | [Next: User and Group Management →](04-user-and-group-management.md)

---

*Unit 1 - Topic 3 of 5 | [Back to README](../README.md)*
