# Mandatory Access Control (MAC)

## Unit 5 - Topic 2 | Linux Access Control

---

## Overview

**Mandatory Access Control (MAC)** enforces security policies defined by the **system administrator**, not the resource owner. Unlike DAC, MAC policies **cannot be overridden by users or even root** (in most configurations). Linux implements MAC through the **Linux Security Modules (LSM)** framework, with **SELinux** and **AppArmor** being the most common implementations.

---

## 1. MAC vs DAC

```
┌──────────────────────────────────────────────────────────────┐
│              DAC vs MAC                                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  DAC (Standard Linux Permissions):                           │
│  ┌──────────┐                                                │
│  │ Process  │──── User's permissions ──── Read/Write file   │
│  │ (as bob) │     (bob's rwx)             ✅ Allowed         │
│  └──────────┘                                                │
│  Root can access EVERYTHING                                  │
│                                                              │
│  MAC (SELinux/AppArmor):                                     │
│  ┌──────────┐                                                │
│  │ Process  │──── DAC check ──── MAC check ──── File        │
│  │ (as bob) │     ✅ Pass       ❌ Denied!                   │
│  └──────────┘                                                │
│  Even root is restricted by MAC policy                       │
│  Process must have DAC AND MAC permission                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. How MAC Works

| Concept | Description |
|---------|-------------|
| **Labels/Contexts** | Every process and file has a security label |
| **Policy** | Rules define which labels can access which labels |
| **Enforcement** | Kernel enforces policy — cannot be bypassed |
| **No discretion** | Users cannot change policy or override |
| **Root restricted** | Root must follow MAC policy too |

```
EXAMPLE:
File label:     system_u:object_r:httpd_sys_content_t:s0
Process label:  system_u:system_r:httpd_t:s0

Policy rule: httpd_t can read httpd_sys_content_t ✅
             httpd_t cannot read shadow_t ❌

Even if Apache runs as root, SELinux prevents it from
reading /etc/shadow because the labels don't match!
```

---

## 3. LSM (Linux Security Modules)

```
┌──────────────────────────────────────────────────────────────┐
│              LSM FRAMEWORK                                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  System Call → DAC Check → LSM Hook → MAC Check → Execute   │
│                                                              │
│  LSM Implementations:                                        │
│  ├── SELinux (Red Hat, Fedora, CentOS, Android)             │
│  ├── AppArmor (Ubuntu, SUSE, Debian)                        │
│  ├── Smack (Embedded, IoT)                                  │
│  ├── TOMOYO (Learning-based)                                │
│  └── Yama (ptrace restrictions)                              │
│                                                              │
│  Note: Only ONE major LSM can be active at a time           │
│  (SELinux OR AppArmor, not both)                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. MAC Modes

| Mode | SELinux | AppArmor | Description |
|------|---------|----------|-------------|
| **Disabled** | `disabled` | N/A | MAC completely off |
| **Permissive** | `permissive` | `complain` | Logs violations but doesn't block |
| **Enforcing** | `enforcing` | `enforce` | Blocks violations and logs |

```bash
# Check current mode
# SELinux:
getenforce                           # Returns: Enforcing/Permissive/Disabled
sestatus                             # Detailed status

# AppArmor:
aa-status                            # AppArmor status
cat /sys/module/apparmor/parameters/mode
```

---

## 5. Real-World MAC Benefits

| Scenario | Without MAC | With MAC |
|----------|------------|---------|
| **Web server compromise** | Attacker reads /etc/shadow | SELinux blocks — httpd_t can't access shadow_t |
| **Privilege escalation** | Root shell → full access | MAC still restricts by label |
| **Container escape** | Access host filesystem | MAC confines container processes |
| **Zero-day exploit** | Full system compromise | Damage limited to process's MAC context |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **MAC** | System-enforced access control (admin-defined) |
| **DAC + MAC** | Both must allow access for it to succeed |
| **LSM** | Kernel framework for MAC implementations |
| **SELinux** | Label-based MAC (Red Hat family) |
| **AppArmor** | Path-based MAC (Ubuntu family) |
| **Key benefit** | Limits damage even when root is compromised |

---

## Quick Revision Questions

1. **What is the key difference between DAC and MAC?**
2. **Can root bypass MAC policies?**
3. **What is the LSM framework?**
4. **What are the three MAC modes?**
5. **How does MAC limit the impact of a web server compromise?**

---

[← Previous: Discretionary Access Control](01-discretionary-access-control.md) | [Next: SELinux Basics →](03-selinux-basics.md)

---

*Unit 5 - Topic 2 of 6 | [Back to README](../README.md)*
