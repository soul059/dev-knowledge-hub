# Namespaces and cgroups

## Unit 5 - Topic 6 | Linux Access Control

---

## Overview

**Namespaces** and **cgroups** are Linux kernel features that provide **process isolation** and **resource control**. They are the foundational technologies behind **containers** (Docker, Kubernetes). Understanding them is essential for container security, process sandboxing, and understanding how isolation works (and fails) at the kernel level.

---

## 1. Linux Namespaces

Namespaces isolate what a process can **see**. Each namespace type isolates a different system resource.

| Namespace | Flag | What It Isolates |
|-----------|------|-----------------|
| **PID** | `CLONE_NEWPID` | Process IDs — process sees different PIDs |
| **Network** | `CLONE_NEWNET` | Network stack — separate interfaces, IPs, routes |
| **Mount** | `CLONE_NEWNS` | Filesystem mounts — different mount points |
| **UTS** | `CLONE_NEWUTS` | Hostname — different hostname per namespace |
| **IPC** | `CLONE_NEWIPC` | Inter-process communication — separate IPC |
| **User** | `CLONE_NEWUSER` | User/group IDs — root in namespace ≠ root on host |
| **Cgroup** | `CLONE_NEWCGROUP` | Cgroup visibility |
| **Time** | `CLONE_NEWTIME` | System clock (Linux 5.6+) |

```
┌──────────────────────────────────────────────────────────────┐
│              NAMESPACE ISOLATION                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  HOST SYSTEM                                                 │
│  ┌────────────────────────────────────────┐                  │
│  │ PID 1 (systemd)                       │                  │
│  │ PID 2345 (sshd)                       │                  │
│  │ PID 3456 (nginx)                      │                  │
│  │                                        │                  │
│  │  CONTAINER A (PID Namespace)           │                  │
│  │  ┌────────────────────────┐            │                  │
│  │  │ PID 1 (app)  ← Sees   │ On host:   │                  │
│  │  │ PID 2 (worker) itself  │ PID 5678   │                  │
│  │  │               as PID 1 │ PID 5679   │                  │
│  │  └────────────────────────┘            │                  │
│  │                                        │                  │
│  │  CONTAINER B (PID Namespace)           │                  │
│  │  ┌────────────────────────┐            │                  │
│  │  │ PID 1 (db)             │ On host:   │                  │
│  │  │ PID 2 (backup)         │ PID 7890   │                  │
│  │  └────────────────────────┘            │                  │
│  └────────────────────────────────────────┘                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Working with Namespaces

```bash
# View namespaces of a process
ls -la /proc/<PID>/ns/
lsns                                  # List all namespaces

# Create new namespace (unshare)
unshare --pid --mount --fork bash      # New PID + mount namespace
unshare --net bash                     # New network namespace
unshare --user bash                    # New user namespace

# Enter existing namespace (nsenter)
nsenter -t <PID> -n                    # Enter network namespace of PID
nsenter -t <PID> -p -m                 # Enter PID + mount namespace

# Network namespace example
ip netns add testns                    # Create named network namespace
ip netns exec testns ip addr           # Run command in namespace
ip netns exec testns bash              # Shell in namespace
ip netns delete testns                 # Delete namespace
```

---

## 3. cgroups (Control Groups)

cgroups control what resources a process can **use**.

| Resource | Control |
|----------|---------|
| **CPU** | Limit CPU time/cores |
| **Memory** | Limit RAM usage |
| **I/O** | Limit disk read/write bandwidth |
| **Network** | Bandwidth throttling |
| **PIDs** | Limit number of processes |

```bash
# cgroups v2 (modern) — /sys/fs/cgroup/

# View cgroup of a process
cat /proc/<PID>/cgroup

# Create a cgroup
mkdir /sys/fs/cgroup/mygroup

# Set memory limit (256MB)
echo 268435456 > /sys/fs/cgroup/mygroup/memory.max

# Set CPU limit (50%)
echo "50000 100000" > /sys/fs/cgroup/mygroup/cpu.max

# Limit PIDs (prevent fork bomb)
echo 100 > /sys/fs/cgroup/mygroup/pids.max

# Assign process to cgroup
echo <PID> > /sys/fs/cgroup/mygroup/cgroup.procs

# Using systemd (preferred method)
systemctl set-property nginx.service MemoryMax=512M
systemctl set-property nginx.service CPUQuota=50%
```

---

## 4. Security Implications

| Feature | Security Benefit | Attack Surface |
|---------|-----------------|---------------|
| **PID namespace** | Can't see/kill host processes | Container escape via /proc |
| **Network namespace** | Isolated network stack | Namespace escape, shared kernel |
| **User namespace** | Root in container ≠ root on host | Privilege escalation vulnerabilities |
| **cgroup CPU limit** | Prevent CPU DoS | Misconfigured limits |
| **cgroup memory limit** | Prevent memory exhaustion | OOM kills |
| **cgroup PID limit** | Prevent fork bombs | Process starvation |

---

## 5. Container Security Connection

```
CONTAINERS = Namespaces + cgroups + Union FS

Docker container isolation:
├── PID Namespace      → Isolated process tree
├── Network Namespace  → Isolated network
├── Mount Namespace    → Isolated filesystem
├── UTS Namespace      → Isolated hostname
├── IPC Namespace      → Isolated IPC
├── User Namespace     → (optional) UID mapping
├── cgroups            → Resource limits
├── seccomp            → System call filtering
└── AppArmor/SELinux   → MAC policy

⚠️ All containers share the SAME KERNEL
→ Kernel exploit = container escape = host compromise
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Namespaces** | Isolate what a process can SEE |
| **cgroups** | Control what resources a process can USE |
| **PID namespace** | Isolated process tree |
| **User namespace** | Root mapping (container root ≠ host root) |
| **Containers** | Built on namespaces + cgroups |
| **Shared kernel** | All containers share host kernel — key risk |

---

## Quick Revision Questions

1. **What is the difference between namespaces and cgroups?**
2. **Name 5 types of Linux namespaces.**
3. **How do cgroups prevent fork bombs?**
4. **Why is the user namespace important for container security?**
5. **What is the key security risk of containers sharing a kernel?**

---

[← Previous: Linux Capabilities](05-linux-capabilities.md)

---

*Unit 5 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Linux Network Security →](../06-Linux-Network-Security/01-network-configuration-files.md)*
