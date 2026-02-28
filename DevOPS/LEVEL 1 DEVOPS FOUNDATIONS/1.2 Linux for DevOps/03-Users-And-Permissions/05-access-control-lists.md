# Chapter 5: Access Control Lists (ACLs)

[← Previous: Special Permissions](04-special-permissions.md) | [Back to README](../README.md) | [Next: Sudo and Sudoers →](06-sudo-and-sudoers.md)

---

## Overview

Standard Linux permissions (owner/group/others) are often too coarse for complex environments. ACLs provide fine-grained access control, letting you assign permissions to specific users or groups beyond the traditional model — without changing file ownership.

---

## 5.1 When to Use ACLs

```
┌──────────────────────────────────────────────────────────────┐
│  Standard Permissions: user1 owns file, group1 can read     │
│  But what if user2 (not in group1) also needs read access?  │
│                                                               │
│  WITHOUT ACL:                     WITH ACL:                  │
│  - Add user2 to group1?          - setfacl -m u:user2:r file│
│    (May grant too much access)     (Precise, targeted)       │
│  - Make world-readable?                                      │
│    (Too permissive!)                                         │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 ACL Commands

```bash
# Check if filesystem supports ACLs (usually default)
mount | grep acl
# Or check /etc/fstab for 'acl' option

# View ACLs
getfacl file.txt
# # file: file.txt
# # owner: devops
# # group: developers
# user::rw-
# group::r--
# other::r--

# Set ACL for a user
setfacl -m u:alice:rw file.txt       # alice gets rw
setfacl -m u:bob:r file.txt          # bob gets r only

# Set ACL for a group
setfacl -m g:qa:rx directory/        # qa group gets rx

# Remove specific ACL
setfacl -x u:alice file.txt          # Remove alice's ACL

# Remove all ACLs
setfacl -b file.txt                  # Remove all ACLs

# Recursive ACL
setfacl -R -m g:developers:rwx /opt/project/

# Default ACL (for new files created in directory)
setfacl -d -m g:developers:rwx /opt/project/

# Copy ACLs from one file to another
getfacl source.txt | setfacl --set-file=- target.txt
```

### The `+` Sign in `ls -l`

```bash
ls -la file.txt
# -rw-rw-r--+ 1 devops developers 4096 ...
#           ^
#           The '+' indicates ACLs are set on this file
```

---

## 5.3 ACL Mask

The mask defines the **maximum effective permissions** for named users and groups (not the owner).

```bash
# Set mask (limits effective permissions)
setfacl -m m::rx file.txt

# Even if alice has rw, effective is r (masked by rx → r)
getfacl file.txt
# user:alice:rw-    #effective:r--
# mask::r-x
```

---

## 5.4 Real-World Scenario

```bash
# Shared log directory: DevOps team reads all, developers read their app only
setfacl -R -m g:devops:rx /var/log/apps/
setfacl -m g:frontend:rx /var/log/apps/frontend/
setfacl -m g:backend:rx /var/log/apps/backend/

# Default ACL: new logs inherit permissions
setfacl -d -m g:devops:rx /var/log/apps/
```

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `getfacl file` | View ACLs |
| `setfacl -m u:user:perms file` | Set user ACL |
| `setfacl -m g:group:perms file` | Set group ACL |
| `setfacl -x u:user file` | Remove user ACL |
| `setfacl -b file` | Remove all ACLs |
| `setfacl -d -m ...` | Set default ACL (directories) |
| `setfacl -R -m ...` | Recursive ACL |

---

## Quick Revision Questions

1. **When would you use ACLs instead of standard permissions?** Give an example.
2. **What does the `+` sign in `ls -l` output mean?**
3. **What is a default ACL and how is it useful for shared directories?**
4. **What is the ACL mask and how does it affect effective permissions?**
5. **How do you remove all ACLs from a file?**
6. **How do you set an ACL so a specific group gets read-write access recursively?**

---

[← Previous: Special Permissions](04-special-permissions.md) | [Back to README](../README.md) | [Next: Sudo and Sudoers →](06-sudo-and-sudoers.md)
