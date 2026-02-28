# Chapter 1: Linux Architecture

[← Back to README](../README.md) | [Next: Linux Distributions →](02-linux-distributions.md)

---

## Overview

Understanding Linux architecture is the foundation of everything you'll do as a DevOps engineer. Linux is a modular, multi-user, multitasking operating system built on a layered architecture that separates hardware interaction from user applications.

---

## 1.1 What is Linux?

Linux is an **open-source, Unix-like operating system kernel** first released by Linus Torvalds in 1991. What people commonly call "Linux" is actually **GNU/Linux** — the Linux kernel combined with GNU utilities, libraries, and other software to form a complete operating system.

### Key Characteristics

- **Open Source:** Source code freely available under GPLv2
- **Multi-user:** Multiple users can access the system simultaneously
- **Multitasking:** Multiple processes run concurrently
- **Portable:** Runs on diverse hardware (servers, desktops, embedded, mobile)
- **Modular:** Components can be loaded/unloaded as needed
- **Secure:** Built-in permission model, SELinux, namespaces

---

## 1.2 Linux Architecture Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                      USER SPACE                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    Applications                            │  │
│  │   (Firefox, Docker, Nginx, kubectl, Ansible, Terraform)   │  │
│  └───────────────────────┬───────────────────────────────────┘  │
│                          │                                       │
│  ┌───────────────────────▼───────────────────────────────────┐  │
│  │                  System Libraries                          │  │
│  │          (glibc, libpthread, libssl, libcurl)             │  │
│  └───────────────────────┬───────────────────────────────────┘  │
│                          │                                       │
│  ┌───────────────────────▼───────────────────────────────────┐  │
│  │                    System Calls                            │  │
│  │    (open, read, write, fork, exec, socket, ioctl)         │  │
│  └───────────────────────┬───────────────────────────────────┘  │
├──────────────────────────┼──────────────────────────────────────┤
│                      KERNEL SPACE                                │
│  ┌───────────────────────▼───────────────────────────────────┐  │
│  │                   Linux Kernel                             │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │  │
│  │  │ Process  │ │  Memory  │ │   File   │ │ Network  │    │  │
│  │  │  Mgmt    │ │   Mgmt   │ │  System  │ │  Stack   │    │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │  │
│  │  ┌──────────┐ ┌──────────┐ ┌───────────────────────┐    │  │
│  │  │  Device  │ │ Security │ │   I/O Scheduler       │    │  │
│  │  │ Drivers  │ │  Module  │ │                       │    │  │
│  │  └──────────┘ └──────────┘ └───────────────────────┘    │  │
│  └───────────────────────┬───────────────────────────────────┘  │
├──────────────────────────┼──────────────────────────────────────┤
│                          ▼                                       │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                     HARDWARE                               │  │
│  │    CPU  │  RAM  │  Disk  │  NIC  │  GPU  │  USB          │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Layer Details

#### 1. Hardware Layer
The physical components: CPU, RAM, hard drives, network interfaces, etc. The kernel abstracts these so applications don't need to know hardware specifics.

#### 2. Kernel Layer
The core of the OS. It manages:

| Subsystem | Responsibility |
|-----------|---------------|
| **Process Management** | Creating, scheduling, terminating processes |
| **Memory Management** | Virtual memory, paging, caching, swap |
| **File System** | VFS layer, ext4, xfs, btrfs, NFS support |
| **Network Stack** | TCP/IP, routing, packet filtering, sockets |
| **Device Drivers** | Interface between hardware and kernel |
| **Security Module** | SELinux, AppArmor, capabilities, namespaces |

#### 3. System Call Interface
The **gateway between user space and kernel space**. When an application needs to access hardware or kernel services, it makes a system call.

```
  Application: "I want to read a file"
       │
       ▼
  read() ──→ System Call Interface ──→ Kernel ──→ Disk Driver ──→ Disk
       │                                                           │
       ◄───────────────────── Data flows back ◄────────────────────┘
```

Common system calls:
```bash
# View available system calls
man syscalls

# Trace system calls of a command
strace ls -la /tmp
strace -c ls /tmp    # Summary view

# Example output:
# % time     calls  syscall
# 26.32       12    openat
# 21.05        8    read
# 15.79        6    write
# 10.53        4    close
```

#### 4. System Libraries
Wrapper functions around system calls. **glibc** (GNU C Library) is the most important — nearly every program links against it.

#### 5. User Applications
Everything you interact with: shells (bash, zsh), editors (vim, nano), servers (nginx, apache), DevOps tools (docker, kubernetes, terraform).

---

## 1.3 Kernel Types

```
┌──────────────────────────────────────────────────────────────┐
│                    Kernel Types Comparison                     │
├──────────────┬─────────────────┬─────────────────────────────┤
│  Monolithic  │   Microkernel   │      Hybrid                 │
│              │                 │                              │
│ ┌──────────┐ │ ┌──────────┐   │ ┌──────────┐                │
│ │Everything│ │ │ Minimal  │   │ │  Core +  │                │
│ │ in one   │ │ │  core    │   │ │  Some    │                │
│ │ block    │ │ │          │   │ │ modules  │                │
│ └──────────┘ │ └──┬───┬───┘   │ └──────────┘                │
│              │ ┌──┘   └──┐    │                              │
│              │ │FS│ │Net│   │  Linux = Monolithic            │
│              │ └──┘ └───┘    │  (with loadable modules)      │
├──────────────┼─────────────────┼─────────────────────────────┤
│ Linux, Unix  │ Minix, QNX     │ Windows NT, macOS            │
└──────────────┴─────────────────┴─────────────────────────────┘
```

Linux uses a **monolithic kernel with loadable kernel modules (LKMs)**. This gives the performance benefits of a monolithic kernel while allowing dynamic module loading.

```bash
# List loaded kernel modules
lsmod

# Get info about a module
modinfo ext4

# Load a module
sudo modprobe vfat

# Remove a module
sudo modprobe -r vfat

# List module dependencies
modprobe --show-depends ext4
```

---

## 1.4 User Space vs Kernel Space

```
┌─────────────────────────────────────────────┐
│              VIRTUAL MEMORY MAP             │
│                                             │
│  0xFFFFFFFF ┌──────────────────────┐        │
│             │                      │        │
│             │    KERNEL SPACE      │        │
│             │    (Ring 0)          │        │
│             │    - Full hardware   │        │
│             │      access          │        │
│             │    - Protected       │        │
│             │                      │        │
│  0xC0000000 ├──────────────────────┤        │
│             │                      │        │
│             │    USER SPACE        │        │
│             │    (Ring 3)          │        │
│             │    - Restricted      │        │
│             │      access          │        │
│             │    - Process         │        │
│             │      isolation       │        │
│             │                      │        │
│  0x00000000 └──────────────────────┘        │
│             (32-bit example)                │
└─────────────────────────────────────────────┘
```

| Feature | User Space | Kernel Space |
|---------|-----------|--------------|
| **Privilege** | Restricted (Ring 3) | Full access (Ring 0) |
| **Memory** | Virtual, isolated per process | Direct hardware access |
| **Crash Impact** | Only the process dies | Entire system crashes (kernel panic) |
| **Access** | Through system calls only | Direct hardware manipulation |
| **Examples** | bash, nginx, docker | Device drivers, scheduler, filesystem |

---

## 1.5 Key Kernel Commands for DevOps

```bash
# Check kernel version
uname -r
# Output: 5.15.0-76-generic

# Full system info
uname -a
# Output: Linux hostname 5.15.0-76-generic #83-Ubuntu SMP x86_64 GNU/Linux

# Detailed kernel info
cat /proc/version

# Kernel parameters at boot
cat /proc/cmdline

# View/modify kernel parameters at runtime
sysctl -a                          # List all parameters
sysctl vm.swappiness               # View specific parameter
sudo sysctl vm.swappiness=10       # Set parameter temporarily
# Persist: add to /etc/sysctl.conf or /etc/sysctl.d/99-custom.conf

# Kernel ring buffer (boot/hardware messages)
dmesg | tail -20
dmesg | grep -i error
```

---

## 1.6 Real-World DevOps Scenario

### Scenario: Tuning a Production Server

You're deploying a high-traffic web application. The default kernel parameters aren't optimized for your workload.

```bash
# Problem: Too many TIME_WAIT connections
# Check current value
sysctl net.ipv4.tcp_tw_reuse
# net.ipv4.tcp_tw_reuse = 0

# Solution: Enable TCP connection reuse
sudo sysctl -w net.ipv4.tcp_tw_reuse=1

# Problem: Running out of file descriptors
# Check current limits
ulimit -n          # Per-process limit
cat /proc/sys/fs/file-max   # System-wide limit

# Solution: Increase limits
sudo sysctl -w fs.file-max=2097152

# Make persistent
cat <<EOF | sudo tee /etc/sysctl.d/99-devops-tuning.conf
net.ipv4.tcp_tw_reuse = 1
fs.file-max = 2097152
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
vm.swappiness = 10
EOF

sudo sysctl --system   # Reload all sysctl configs
```

---

## Troubleshooting Tips

| Symptom | Diagnostic Command | Likely Cause |
|---------|-------------------|--------------|
| System unresponsive | `dmesg \| tail` | Kernel panic, OOM killer |
| Module not loading | `modprobe -v module_name` | Missing dependency |
| High kernel CPU usage | `top` (check `%sy`) | IRQ storms, driver issues |
| "Operation not permitted" | `id`, `whoami` | Need root/kernel privileges |
| Kernel version mismatch | `uname -r` vs package | Reboot needed after update |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Linux** | Open-source Unix-like kernel, GPLv2 |
| **Architecture** | Hardware → Kernel → Syscalls → Libraries → Applications |
| **Kernel Type** | Monolithic with loadable modules |
| **User Space** | Ring 3, restricted, process-isolated |
| **Kernel Space** | Ring 0, full hardware access |
| **System Call** | Interface between user and kernel space |
| **Kernel Modules** | Dynamically loadable kernel extensions (`.ko` files) |
| **sysctl** | View/modify kernel parameters at runtime |
| **dmesg** | Kernel ring buffer messages |
| **uname** | Display kernel/system information |

---

## Quick Revision Questions

1. **What are the main layers of Linux architecture?** Describe each layer's role.
2. **What is the difference between user space and kernel space?** Why is this separation important?
3. **How does an application access hardware resources?** Explain the role of system calls.
4. **What is the difference between a monolithic and microkernel?** Where does Linux fit?
5. **How would you check the kernel version on a running system?** Name at least two methods.
6. **What is `sysctl` used for?** Give an example of a kernel parameter you might tune for a web server.

---

[← Back to README](../README.md) | [Next: Linux Distributions →](02-linux-distributions.md)
