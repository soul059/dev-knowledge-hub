# Discretionary Access Control (DAC)

## Unit 5 - Topic 1 | Linux Access Control

---

## Overview

**Discretionary Access Control (DAC)** is the default access control model in Linux. The **owner** of a resource decides who can access it using standard file permissions (rwx) and Access Control Lists (ACLs). DAC is "discretionary" because resource owners have the discretion to set permissions as they choose.

---

## 1. DAC Model

```
┌──────────────────────────────────────────────────────────────┐
│              DAC — OWNER CONTROLS ACCESS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  File: /home/alice/report.pdf                                │
│  Owner: alice                                                │
│                                                              │
│  Alice (owner) decides:                                      │
│  ├── Owner (alice): rwx (read, write, execute)              │
│  ├── Group (devs):  r-x (read, execute)                     │
│  └── Others:        --- (no access)                          │
│                                                              │
│  Result: chmod 750 report.pdf                                │
│                                                              │
│  KEY PRINCIPLE: Owner has full control over their files      │
│  WEAKNESS: Root bypasses ALL DAC controls                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Standard DAC Components

| Component | Description | Commands |
|-----------|-------------|----------|
| **Permissions** | rwx for owner, group, others | `chmod` |
| **Ownership** | User and group assignment | `chown`, `chgrp` |
| **umask** | Default permission mask | `umask 027` |
| **SUID/SGID** | Special execution permissions | `chmod u+s`, `chmod g+s` |
| **Sticky bit** | Only owner can delete files | `chmod +t` |
| **ACLs** | Fine-grained per-user permissions | `setfacl`, `getfacl` |

---

## 3. Access Control Lists (ACLs)

```bash
# ACLs extend basic permissions — grant access to specific users/groups
# beyond owner/group/other model

# View ACLs
getfacl /home/alice/project/

# Grant read access to specific user
setfacl -m u:bob:rx /home/alice/project/
# bob gets read+execute, regardless of "others" permission

# Grant access to specific group
setfacl -m g:auditors:r /home/alice/project/report.pdf

# Set default ACL (applied to new files in directory)
setfacl -d -m u:bob:rx /home/alice/project/

# Remove specific ACL
setfacl -x u:bob /home/alice/project/

# Remove all ACLs
setfacl -b /home/alice/project/

# ACL indicator in ls output
ls -la /home/alice/
# drwxr-x---+ 2 alice devs 4096 ...
#           ^ Plus sign indicates ACLs present
```

---

## 4. DAC Weaknesses

| Weakness | Description |
|----------|-------------|
| **Root bypass** | Root can access ANY file regardless of permissions |
| **Owner discretion** | Users may set weak permissions (777) |
| **No process isolation** | A compromised process inherits user's permissions |
| **No mandatory policy** | No system-wide enforcement beyond permissions |
| **SUID risk** | SUID binaries run with file owner's privileges |

```
ATTACK SCENARIO:
1. User alice sets chmod 777 on sensitive data directory
2. Attacker compromises any user account
3. Attacker reads alice's sensitive data
4. DAC cannot prevent this — alice authorized it

SOLUTION: Mandatory Access Control (MAC) — SELinux/AppArmor
→ System-enforced policies OVERRIDE user decisions
```

---

## 5. DAC vs MAC Comparison

| Feature | DAC | MAC |
|---------|:---:|:---:|
| **Who sets policy?** | File owner | System administrator |
| **Root bypass?** | Yes | No (policy-enforced) |
| **User can override?** | Yes | No |
| **Default in Linux?** | Yes | Optional (SELinux/AppArmor) |
| **Flexibility** | High | Lower |
| **Security** | Moderate | Strong |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DAC** | Owner controls access to resources |
| **Permissions** | rwx for owner/group/others |
| **ACLs** | Fine-grained per-user permissions beyond rwx |
| **Weakness** | Root bypasses all, users may set weak perms |
| **setfacl/getfacl** | Manage extended ACLs |
| **MAC needed** | DAC alone insufficient for high-security |

---

## Quick Revision Questions

1. **What makes DAC "discretionary"?**
2. **How do ACLs extend standard Linux permissions?**
3. **What is the main weakness of DAC?**
4. **How do you grant read access to a specific user using ACLs?**
5. **How does MAC differ from DAC?**

---

[Next: Mandatory Access Control (MAC) →](02-mandatory-access-control.md)

---

*Unit 5 - Topic 1 of 6 | [Back to README](../README.md)*
