# Linux Capabilities

## Unit 5 - Topic 5 | Linux Access Control

---

## Overview

**Linux capabilities** break up the monolithic root privilege into distinct units. Instead of giving a process full root access (UID 0), you grant only the **specific capabilities** it needs. This follows the **principle of least privilege** — a web server only needs `CAP_NET_BIND_SERVICE` to bind to port 80, not full root.

---

## 1. The Problem with Root

```
┌──────────────────────────────────────────────────────────────┐
│              WHY CAPABILITIES EXIST                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TRADITIONAL MODEL:                                          │
│  ┌─────────┐                                                 │
│  │  root   │ ──► ALL privileges (dangerous!)                │
│  │ (UID 0) │     • Bind ports < 1024                        │
│  │         │     • Load kernel modules                      │
│  │         │     • Override file permissions                 │
│  │         │     • Change system clock                       │
│  │         │     • Kill any process                          │
│  │         │     • Reboot system                             │
│  └─────────┘     • ... everything                           │
│                                                              │
│  CAPABILITIES MODEL:                                         │
│  ┌───────────────┐                                           │
│  │ nginx process │ ──► Only CAP_NET_BIND_SERVICE            │
│  │ (non-root)    │     Can bind port 80/443                 │
│  │               │     Cannot load modules ❌               │
│  │               │     Cannot read /etc/shadow ❌            │
│  └───────────────┘     Cannot reboot ❌                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Key Capabilities

| Capability | Permission | Security Concern |
|-----------|-----------|-----------------|
| `CAP_NET_BIND_SERVICE` | Bind to ports < 1024 | Safe to grant |
| `CAP_NET_RAW` | Use raw sockets (ping, packet crafting) | Network attacks |
| `CAP_SYS_ADMIN` | Broad admin operations (mount, namespace) | Almost as dangerous as root |
| `CAP_SYS_PTRACE` | Debug/trace other processes | Process injection |
| `CAP_DAC_OVERRIDE` | Bypass file read/write permissions | Full file access |
| `CAP_DAC_READ_SEARCH` | Bypass file read permissions | Read any file |
| `CAP_NET_ADMIN` | Network configuration | Interface/firewall manipulation |
| `CAP_SYS_MODULE` | Load/unload kernel modules | Rootkit loading |
| `CAP_CHOWN` | Change file ownership | Ownership takeover |
| `CAP_SETUID` | Change UID | Become root |

---

## 3. Managing Capabilities

```bash
# === VIEW CAPABILITIES ===
# View capabilities of a file
getcap /usr/bin/ping
# /usr/bin/ping cap_net_raw=ep

# Find all files with capabilities
getcap -r / 2>/dev/null

# View capabilities of running process
cat /proc/<PID>/status | grep Cap
# CapPrm: 0000003fffffffff  ← Permitted
# CapEff: 0000003fffffffff  ← Effective
# CapBnd: 0000003fffffffff  ← Bounding

# Decode capability hex values
capsh --decode=0000003fffffffff

# === SET CAPABILITIES ===
# Grant capability to binary
setcap cap_net_bind_service=+ep /usr/local/bin/mywebserver
# Now mywebserver can bind to port 80 without running as root!

# Remove capabilities
setcap -r /usr/local/bin/mywebserver

# === CAPABILITY SETS ===
# ep = effective + permitted
# eip = effective + inheritable + permitted
```

---

## 4. Capability Sets Explained

| Set | Description |
|-----|-------------|
| **Effective** | Currently active capabilities |
| **Permitted** | Capabilities the process CAN activate |
| **Inheritable** | Capabilities passed to child processes |
| **Bounding** | Upper limit — cannot exceed this |
| **Ambient** | Preserved across execve (without SUID) |

---

## 5. Security Auditing Capabilities

```bash
# FIND ALL BINARIES WITH CAPABILITIES (security audit)
getcap -r / 2>/dev/null

# DANGEROUS capabilities to watch for:
# cap_sys_admin — nearly equivalent to root
# cap_sys_ptrace — can debug and inject into processes
# cap_dac_override — bypass all file permissions
# cap_sys_module — load kernel modules (rootkits!)
# cap_setuid — change UID (become root)

# Drop capabilities in a process (defense)
# Using capsh to run with reduced capabilities
capsh --drop=cap_sys_admin -- -c '/usr/local/bin/myapp'
```

---

## 6. Capabilities vs SUID

| Feature | SUID | Capabilities |
|---------|:----:|:---:|
| **Granularity** | All-or-nothing (full root) | Specific permissions only |
| **Risk** | High — full root on exploit | Lower — limited permissions |
| **Best practice** | Avoid when possible | Preferred alternative |
| **Example** | `chmod u+s /usr/bin/ping` | `setcap cap_net_raw+ep /usr/bin/ping` |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Capabilities** | Fine-grained root privilege decomposition |
| **CAP_NET_BIND_SERVICE** | Bind ports < 1024 without root |
| **CAP_SYS_ADMIN** | Dangerous — almost full root |
| **getcap** | View file capabilities |
| **setcap** | Set file capabilities |
| **Audit** | `getcap -r / 2>/dev/null` — find all capability binaries |

---

## Quick Revision Questions

1. **What problem do Linux capabilities solve?**
2. **What capability allows binding to ports below 1024?**
3. **Why is CAP_SYS_ADMIN considered almost as dangerous as root?**
4. **How do capabilities compare to SUID?**
5. **How do you audit all files with capabilities on a system?**

---

[← Previous: AppArmor Basics](04-apparmor-basics.md) | [Next: Namespaces and cgroups →](06-namespaces-and-cgroups.md)

---

*Unit 5 - Topic 5 of 6 | [Back to README](../README.md)*
