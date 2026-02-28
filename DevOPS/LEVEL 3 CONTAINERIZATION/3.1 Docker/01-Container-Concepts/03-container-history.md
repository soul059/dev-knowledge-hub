# Chapter 3: Container History (chroot, LXC)

> **Unit 1: Container Concepts** | [Course Home](../README.md)

---

## Overview

Containers didn't appear overnight. They evolved over **decades** of innovation in process isolation on Unix and Linux systems. Understanding this history helps you appreciate why Docker became so successful and how container technology actually works under the hood.

---

## 3.1 Timeline of Container Evolution

```
1979        2000        2004-2008    2008      2013       2014-Now
  |           |            |           |         |           |
  v           v            v           v         v           v
chroot    FreeBSD      Solaris     LXC       Docker      Kubernetes
          Jails        Zones      (Linux)    launches    OCI, CRI
                       + cgroups

+--------+  +--------+  +---------+  +------+  +--------+  +---------+
| chroot |->| Jails  |->| Zones/  |->| LXC  |->| Docker |->| K8s/OCI |
| (1979) |  | (2000) |  | cgroups |  |(2008)|  | (2013) |  | (2014+) |
+--------+  +--------+  +---------+  +------+  +--------+  +---------+
  Basic       Full       Resource    Linux      User         Industry
  FS          process    control +   native     friendly     standard
  isolation   isolation  isolation  containers  containers   orchestration
```

---

## 3.2 chroot (1979) — The Beginning

**chroot** (change root) was introduced in **Version 7 Unix** in 1979 and added to BSD in 1982. It changes the apparent root directory for a process and its children.

### How chroot Works

```
  Normal process:                   chroot'ed process:
  
  /                                 / (appears as root)
  ├── bin/                          ├── bin/
  ├── etc/                          ├── etc/
  ├── home/                         ├── lib/
  │   └── user/                     └── app/
  ├── var/                              └── myapp
  └── ...
                                    Actually located at:
  Process sees FULL                 /var/chroot/myapp/
  filesystem                       
                                    Process CANNOT access
                                    anything above this!
```

### chroot Example

```bash
# Create a chroot environment
mkdir -p /myroot/{bin,lib,lib64,etc}

# Copy essential binaries
cp /bin/bash /myroot/bin/
cp /bin/ls /myroot/bin/

# Copy required libraries (use ldd to find them)
ldd /bin/bash  # shows shared library dependencies
cp /lib/x86_64-linux-gnu/libtinfo.so.6 /myroot/lib/
cp /lib/x86_64-linux-gnu/libc.so.6 /myroot/lib/
cp /lib64/ld-linux-x86-64.so.2 /myroot/lib64/

# Enter the chroot
sudo chroot /myroot /bin/bash

# Inside chroot:
# ls /        -> shows only bin, lib, lib64, etc
# cd /../../  -> still stuck in /myroot
```

### Limitations of chroot

| Limitation | Description |
|-----------|-------------|
| **Only filesystem isolation** | Processes, network, and IPC are NOT isolated |
| **Easy to escape** | Root user can break out of chroot |
| **No resource limits** | Can consume all host CPU/memory |
| **No network isolation** | Shares host network stack |
| **Manual setup** | Must manually copy all dependencies |

---

## 3.3 FreeBSD Jails (2000)

FreeBSD Jails extended chroot with **proper process and network isolation** — a major leap forward.

### What Jails Added

```
+----------------------------------------------------+
|  FreeBSD Host                                       |
|                                                     |
|  +-------------------+  +-------------------+       |
|  |     Jail 1        |  |     Jail 2        |       |
|  |  +-----------+    |  |  +-----------+    |       |
|  |  | Own PID 1 |    |  |  | Own PID 1 |    |       |
|  |  +-----------+    |  |  +-----------+    |       |
|  |  | Own IP:        |  |  | Own IP:        |       |
|  |  | 10.0.0.1       |  |  | 10.0.0.2       |       |
|  |  | Own hostname   |  |  | Own hostname   |       |
|  |  | Own users      |  |  | Own users      |       |
|  |  | Own filesystem |  |  | Own filesystem |       |
|  |  +-----------+    |  |  +-----------+    |       |
|  +-------------------+  +-------------------+       |
|                                                     |
|  Shared FreeBSD Kernel                              |
+----------------------------------------------------+
```

### Key Improvements Over chroot

- **Process isolation** — each jail has its own PID space
- **Network isolation** — each jail gets its own IP address
- **User isolation** — root in a jail ≠ root on the host
- **Restricted system calls** — jails cannot modify host kernel

---

## 3.4 Solaris Zones (2004)

Sun Microsystems introduced **Solaris Zones** (also called Solaris Containers), bringing resource management and isolation together.

```
+-------------------------------------------------------+
|  Solaris Host (Global Zone)                            |
|                                                        |
|  +-------------------+    +-------------------+        |
|  | Non-Global Zone 1 |    | Non-Global Zone 2 |        |
|  |                    |    |                    |        |
|  | Own processes      |    | Own processes      |        |
|  | Own filesystem    |    | Own filesystem    |        |
|  | Own network       |    | Own network       |        |
|  | Resource limits:  |    | Resource limits:  |        |
|  |   CPU: 2 cores    |    |   CPU: 4 cores    |        |
|  |   RAM: 4 GB       |    |   RAM: 8 GB       |        |
|  +-------------------+    +-------------------+        |
|                                                        |
|  Solaris Kernel                                        |
+-------------------------------------------------------+
```

### What Solaris Zones Added

- **Resource controls** — CPU, memory, and I/O limits per zone
- **Near-native performance** — no hypervisor overhead
- **Live migration** — move zones between physical servers
- **Branded zones** — run Linux binaries on Solaris

---

## 3.5 Linux cgroups (2006-2008)

**cgroups** (control groups) were developed by Google engineers Paul Menage and Rohit Seth, merged into the Linux kernel in **2.6.24 (2008)**.

### What cgroups Control

```
+--------------------------------------------------+
|         Linux Kernel cgroups Hierarchy            |
|                                                   |
|  /sys/fs/cgroup/                                  |
|  ├── cpu/                                         |
|  │   ├── group_A/   (50% CPU)                    |
|  │   └── group_B/   (30% CPU)                    |
|  ├── memory/                                      |
|  │   ├── group_A/   (512 MB limit)               |
|  │   └── group_B/   (1 GB limit)                 |
|  ├── blkio/                                       |
|  │   ├── group_A/   (100 MB/s disk)              |
|  │   └── group_B/   (200 MB/s disk)              |
|  └── pids/                                        |
|      ├── group_A/   (max 100 processes)           |
|      └── group_B/   (max 200 processes)           |
+--------------------------------------------------+
```

### cgroups v1 vs v2

| Feature | cgroups v1 | cgroups v2 |
|---------|-----------|-----------|
| Hierarchy | Multiple trees | Single unified tree |
| Introduced | 2008 | 2016 |
| Delegation | Complex | Simplified |
| Pressure info | Not available | PSI (Pressure Stall Information) |
| Default in Docker | Legacy support | Modern default |

---

## 3.6 Linux Namespaces (2002-2016)

Namespaces were gradually added to the Linux kernel over many years:

```
  2002         2006         2006         2006         2013         2016
    |            |            |            |            |            |
    v            v            v            v            v            v
  Mount        UTS          IPC          PID         User         cgroup
  namespace    namespace    namespace    namespace   namespace    namespace
  
  +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
  | Own FS |  | Own    |  | Own    |  | Own    |  | Own    |  | Own    |
  | mount  |  | host-  |  | shared |  | PID    |  | UID/   |  | cgroup |
  | points |  | name   |  | memory |  | space  |  | GID    |  | view   |
  +--------+  +--------+  +--------+  +--------+  +--------+  +--------+
```

---

## 3.7 LXC — Linux Containers (2008)

**LXC** (Linux Containers) was the first complete implementation of Linux container management, combining namespaces and cgroups into a usable tool.

### LXC Architecture

```
+-------------------------------------------------------+
|  Linux Host                                            |
|                                                        |
|  +---------------------+  +---------------------+      |
|  |    LXC Container 1  |  |    LXC Container 2  |      |
|  |  +---------+        |  |  +---------+        |      |
|  |  | init    |        |  |  | init    |        |      |
|  |  | systemd |        |  |  | systemd |        |      |
|  |  | sshd    |        |  |  | sshd    |        |      |
|  |  | nginx   |        |  |  | postgres|        |      |
|  |  +---------+        |  |  +---------+        |      |
|  |  Full OS userspace  |  |  Full OS userspace  |      |
|  +---------------------+  +---------------------+      |
|                                                        |
|  liblxc (userspace tools)                              |
|  +----------------------------------------------------+|
|  |  Linux Kernel (namespaces + cgroups)               ||
|  +----------------------------------------------------+|
+-------------------------------------------------------+
```

### LXC vs Docker (Key Differences)

| Feature | LXC | Docker |
|---------|-----|--------|
| **Focus** | System containers (full OS) | Application containers (single app) |
| **Init system** | Runs init/systemd inside | Runs single process (PID 1) |
| **Image format** | Tarballs, templates | Layered images (Dockerfile) |
| **Ecosystem** | Limited | Docker Hub, Compose, Swarm |
| **Portability** | Manual configuration | Build once, run anywhere |
| **Learning curve** | Steep | Gentle |

### Why Docker Won Over LXC

```
  LXC (2008):                         Docker (2013):
  
  - Write shell scripts               - Write a Dockerfile
  - Configure namespaces manually      - docker build . 
  - Set up cgroups by hand             - docker run nginx
  - Share templates via tarballs       - docker push/pull
  - No ecosystem                       - Docker Hub ecosystem
  
  LXC = Powerful but complex          Docker = Simple and productive
```

**Docker initially used LXC** as its execution driver but later replaced it with **libcontainer** (now **runc**) for better control and portability.

---

## 3.8 Docker (2013) — The Revolution

Docker was created by **Solomon Hykes** at **dotCloud** (a PaaS company) and open-sourced at **PyCon 2013**.

### What Docker Added

1. **Dockerfile** — declarative, repeatable image builds
2. **Docker Hub** — centralized image registry
3. **Layered images** — efficient storage and distribution
4. **Simple CLI** — `docker run` instead of complex scripts
5. **Ecosystem** — Compose, Swarm, Machine
6. **Developer experience** — made containers accessible to everyone

---

## 3.9 Evolution Summary

```
1979: chroot ............... Filesystem isolation only
2000: FreeBSD Jails ........ + Process & network isolation
2004: Solaris Zones ........ + Resource controls
2006: cgroups (Linux) ...... + Resource limiting in Linux
2008: LXC .................. Combined namespaces + cgroups
2013: Docker ............... Made containers user-friendly
2014: Kubernetes ........... Container orchestration at scale
2015: OCI .................. Industry standards for containers
2017: containerd ........... Industry-standard container runtime
```

---

## Summary Table

| Technology | Year | Key Contribution | Limitation |
|-----------|------|-------------------|------------|
| chroot | 1979 | Filesystem isolation | No process/network isolation |
| FreeBSD Jails | 2000 | Full process/network isolation | FreeBSD only |
| Solaris Zones | 2004 | Resource controls + isolation | Solaris only |
| cgroups | 2006-08 | Resource limiting on Linux | Just a kernel feature, not a tool |
| Namespaces | 2002-16 | Process/network/FS isolation | Just a kernel feature, not a tool |
| LXC | 2008 | First complete Linux containers | Complex, system-container focused |
| Docker | 2013 | User-friendly app containers | Initially Linux only |
| OCI/containerd | 2015-17 | Industry standards | — |

---

## Quick Revision Questions

1. **What does chroot do, and what is its primary limitation?**
   - chroot changes the apparent root directory for a process. Its limitation is that it only provides filesystem isolation — not process, network, or resource isolation

2. **How did FreeBSD Jails improve upon chroot?**
   - Jails added process isolation (own PID space), network isolation (own IP), user isolation (root in jail ≠ host root), and restricted system calls

3. **What are cgroups and who developed them?**
   - cgroups (control groups) are a Linux kernel feature developed by Google engineers, providing resource limitation (CPU, memory, I/O) for groups of processes

4. **Why did Docker succeed where LXC did not gain mass adoption?**
   - Docker provided a simple CLI, declarative Dockerfiles, layered images, Docker Hub registry, and a rich ecosystem — making containers accessible to average developers

5. **What is the difference between LXC containers and Docker containers?**
   - LXC focuses on system containers (full OS with init system), while Docker focuses on application containers (single process per container with layered images)

6. **What was Docker's original relationship with LXC?**
   - Docker initially used LXC as its execution backend, but later replaced it with its own library (libcontainer, now runc) for better portability and control

---

[← Previous: Containers vs Virtual Machines](02-containers-vs-vms.md) | [Next: OCI Standards →](04-oci-standards.md)
