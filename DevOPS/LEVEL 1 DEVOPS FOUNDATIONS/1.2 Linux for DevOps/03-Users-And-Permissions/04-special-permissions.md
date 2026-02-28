# Chapter 4: Special Permissions (SUID, SGID, Sticky Bit)

[← Previous: File Permissions](03-file-permissions.md) | [Back to README](../README.md) | [Next: Access Control Lists →](05-access-control-lists.md)

---

## Overview

Beyond standard `rwx` permissions, Linux has three special permission bits: **SUID**, **SGID**, and the **Sticky Bit**. These control elevated execution privileges and shared directory behavior — critical for understanding system security and managing shared environments.

---

## 4.1 Special Permissions Overview

```
┌──────────────────────────────────────────────────────────────────┐
│              SPECIAL PERMISSION BITS                              │
│                                                                   │
│  ┌─────────┬───────┬───────────────────────────────────────────┐ │
│  │  Bit    │ Octal │ Effect                                    │ │
│  ├─────────┼───────┼───────────────────────────────────────────┤ │
│  │  SUID   │ 4000  │ File executes as file OWNER               │ │
│  │         │       │ (shown as 's' in owner's x position)     │ │
│  ├─────────┼───────┼───────────────────────────────────────────┤ │
│  │  SGID   │ 2000  │ File: executes as file GROUP              │ │
│  │         │       │ Dir: new files inherit directory's group  │ │
│  │         │       │ (shown as 's' in group's x position)     │ │
│  ├─────────┼───────┼───────────────────────────────────────────┤ │
│  │ Sticky  │ 1000  │ Dir: only file owner can delete their     │ │
│  │  Bit    │       │ own files (shown as 't' in others' x)    │ │
│  └─────────┴───────┴───────────────────────────────────────────┘ │
│                                                                   │
│  Position in ls -l output:                                       │
│  -rwsr-xr-x    ← SUID (s in owner execute)                     │
│  -rwxr-sr-x    ← SGID (s in group execute)                     │
│  drwxrwxrwt    ← Sticky (t in others execute)                   │
│                                                                   │
│  Capital S or T = special bit set but NO execute permission      │
│  -rwSr--r--    ← SUID set, but owner has no execute             │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4.2 SUID (Set User ID)

When a file with SUID is executed, it runs with the **file owner's privileges**, not the executing user's.

```bash
# Classic example: passwd command
ls -la /usr/bin/passwd
# -rwsr-xr-x 1 root root 68208 ... /usr/bin/passwd
#    ^
#    SUID bit — runs as root even when invoked by regular user
# This allows users to change their own password
# (which modifies /etc/shadow, owned by root)

# Set SUID
sudo chmod u+s /path/to/binary
sudo chmod 4755 /path/to/binary

# Remove SUID
sudo chmod u-s /path/to/binary

# Find all SUID files (security audit)
find / -type f -perm -4000 2>/dev/null
# /usr/bin/passwd
# /usr/bin/sudo
# /usr/bin/su
# /usr/bin/mount
```

> **Security Warning:** SUID on scripts is ignored by most modern kernels. SUID should only be set on compiled binaries that are specifically designed for it. Unnecessary SUID binaries are a security risk.

---

## 4.3 SGID (Set Group ID)

**On files:** Executes with the file's group privileges.  
**On directories:** New files created inside inherit the directory's group (most useful for DevOps).

```bash
# SGID on a directory (most common DevOps use)
sudo mkdir /opt/shared-project
sudo chgrp developers /opt/shared-project
sudo chmod 2775 /opt/shared-project
#         ^
#         SGID bit

# Now any file created inside inherits "developers" group
touch /opt/shared-project/newfile.txt
ls -la /opt/shared-project/newfile.txt
# -rw-rw-r-- 1 devops developers ... newfile.txt
#                      ^^^^^^^^^^
#                      Inherited from directory!

# Without SGID, files would get the user's primary group instead

# Set SGID
sudo chmod g+s directory/
sudo chmod 2775 directory/

# Remove SGID
sudo chmod g-s directory/

# Find SGID files/dirs
find / -perm -2000 2>/dev/null
```

---

## 4.4 Sticky Bit

The Sticky Bit on a directory means only the **file owner, directory owner, or root** can delete or rename files — even if others have write access. Essential for shared directories like `/tmp`.

```bash
# Classic example: /tmp
ls -ld /tmp
# drwxrwxrwt 15 root root 4096 ... /tmp
#          ^
#          Sticky bit — users can only delete their own files

# Set sticky bit
sudo chmod +t /shared/directory
sudo chmod 1777 /shared/directory

# Remove sticky bit
sudo chmod -t /shared/directory

# Verify
ls -ld /shared/directory
# drwxrwxrwt ... ← 't' in others' execute
```

---

## 4.5 Complete Octal Notation

```
┌───────────────────────────────────────────────────────────┐
│  FULL OCTAL: [Special][Owner][Group][Others]              │
│                                                            │
│  chmod 4755 file    → SUID + rwxr-xr-x  (-rwsr-xr-x)   │
│  chmod 2775 dir     → SGID + rwxrwxr-x  (drwxrwsr-x)   │
│  chmod 1777 dir     → Sticky + rwxrwxrwx (drwxrwxrwt)   │
│  chmod 6755 file    → SUID+SGID + rwxr-xr-x             │
│  chmod 0644 file    → No special + rw-r--r--              │
│                                                            │
│  Special digit:                                            │
│  4 = SUID                                                  │
│  2 = SGID                                                  │
│  1 = Sticky Bit                                            │
│  Add them: 4+2 = 6 (SUID + SGID)                         │
└───────────────────────────────────────────────────────────┘
```

---

## 4.6 Real-World Scenario: Shared Team Directory

```bash
#!/bin/bash
# setup-shared-workspace.sh

PROJECT="/opt/team-project"
GROUP="developers"

# Create group if not exists
sudo groupadd -f $GROUP

# Create directory structure
sudo mkdir -p $PROJECT/{code,docs,builds,logs}

# Set group ownership
sudo chgrp -R $GROUP $PROJECT

# Set SGID on all directories (new files inherit group)
find $PROJECT -type d -exec chmod 2775 {} +

# Set file permissions
find $PROJECT -type f -exec chmod 664 {} +

# Add sticky bit to builds (only owner can delete their builds)
sudo chmod 1775 $PROJECT/builds

# Verify
ls -la $PROJECT/
# drwxrwsr-x ... code/      ← SGID (s in group)
# drwxrwsr-x ... docs/      ← SGID
# drwxrwsr-t ... builds/    ← SGID + Sticky (s + t)
# drwxrwsr-x ... logs/      ← SGID
```

---

## Summary Table

| Bit | Octal | On Files | On Directories | Symbol |
|-----|-------|----------|----------------|--------|
| **SUID** | 4000 | Runs as file owner | No effect | `s` in owner `x` |
| **SGID** | 2000 | Runs as file group | New files inherit group | `s` in group `x` |
| **Sticky** | 1000 | No effect | Only owner can delete | `t` in others `x` |

---

## Quick Revision Questions

1. **What is SUID and why does `/usr/bin/passwd` need it?**
2. **How does SGID on a directory help DevOps teams?** Give a practical example.
3. **What is the Sticky Bit and why is it set on `/tmp`?**
4. **What does `chmod 2775` mean?** Break it down completely.
5. **How would you find all SUID files on a system?** Why is this important for security?
6. **What does a capital `S` or `T` mean in `ls -l` output?**

---

[← Previous: File Permissions](03-file-permissions.md) | [Back to README](../README.md) | [Next: Access Control Lists →](05-access-control-lists.md)
