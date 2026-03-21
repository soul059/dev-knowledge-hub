# Linux Architecture Overview

## Unit 1 - Topic 1 | Linux Security Fundamentals

---

## Overview

Understanding **Linux architecture** is essential for security professionals because every attack and defense involves interacting with one or more architectural components. From kernel exploits to user-space vulnerabilities, knowing how Linux is structured helps you understand **where attacks occur** and **where defenses should be placed**.

---

## 1. Linux Architecture Layers

```
┌──────────────────────────────────────────────────────────────┐
│              LINUX ARCHITECTURE                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────┐                │
│  │         USER APPLICATIONS                │  Ring 3        │
│  │    (Firefox, Apache, Python, bash)       │  (User Space)  │
│  └──────────────────┬───────────────────────┘                │
│                     │                                        │
│  ┌──────────────────┴───────────────────────┐                │
│  │         SYSTEM LIBRARIES                 │                │
│  │    (glibc, libssl, libpam, libcap)       │                │
│  └──────────────────┬───────────────────────┘                │
│                     │ System Calls                           │
│  ════════════════════╪══════════════════════════              │
│                     │ (Kernel boundary)                      │
│  ┌──────────────────┴───────────────────────┐                │
│  │         KERNEL                           │  Ring 0        │
│  │  ┌────────────┐ ┌─────────────────┐      │  (Kernel Space)│
│  │  │ Process    │ │ Memory          │      │                │
│  │  │ Management │ │ Management      │      │                │
│  │  ├────────────┤ ├─────────────────┤      │                │
│  │  │ File System│ │ Network Stack   │      │                │
│  │  ├────────────┤ ├─────────────────┤      │                │
│  │  │ Device     │ │ Security        │      │                │
│  │  │ Drivers    │ │ (LSM, SELinux)  │      │                │
│  │  └────────────┘ └─────────────────┘      │                │
│  └──────────────────────────────────────────┘                │
│                     │                                        │
│  ┌──────────────────┴───────────────────────┐                │
│  │         HARDWARE                         │                │
│  │    (CPU, RAM, Disk, Network, USB)        │                │
│  └──────────────────────────────────────────┘                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Key Architectural Components

| Component | Description | Security Relevance |
|-----------|-------------|-------------------|
| **Kernel** | Core OS — manages hardware, memory, processes | Kernel exploits = full system compromise |
| **System Calls** | Interface between user space and kernel | Syscall filtering (seccomp) restricts capabilities |
| **Shell** | Command interpreter (bash, zsh, sh) | Shell injection, command injection attacks |
| **File System** | Organizes data on disk | Permissions, SUID, file integrity |
| **Process Management** | Creates, schedules, terminates processes | Process injection, privilege escalation |
| **Network Stack** | TCP/IP implementation in kernel | Packet filtering (netfilter/iptables) |
| **LSM (Linux Security Modules)** | Framework for security extensions | SELinux, AppArmor enforcement |

---

## 3. Boot Process (Security Perspective)

```
BOOT SEQUENCE:
1. BIOS/UEFI ─────── Firmware executes
                      ⚠️ Secure Boot verifies bootloader
2. GRUB2 ─────────── Bootloader loads kernel
                      ⚠️ GRUB password prevents boot tampering
3. Kernel ────────── Loads into memory, initializes hardware
                      ⚠️ Kernel parameters can disable security
4. initramfs ─────── Temporary root filesystem
                      ⚠️ Can contain backdoors if tampered
5. systemd (PID 1) ─ First user-space process
                      ⚠️ Controls all service startup
6. Services ──────── Network, SSH, logging, etc.
                      ⚠️ Attack surface — minimize services
7. Login ─────────── getty/display manager
                      ⚠️ PAM enforces authentication
```

---

## 4. User Space vs Kernel Space

| Feature | User Space | Kernel Space |
|---------|-----------|-------------|
| **Privilege** | Restricted (Ring 3) | Full access (Ring 0) |
| **Memory access** | Own process memory only | All system memory |
| **Hardware access** | Via system calls only | Direct hardware access |
| **Crash impact** | Process dies | System crash (kernel panic) |
| **Exploit impact** | Limited to user privileges | Full system compromise |

```bash
# View running kernel version
uname -r
uname -a

# View kernel modules (potential attack surface)
lsmod

# View system calls made by a process
strace -p <PID>
strace ls /tmp

# View kernel parameters
sysctl -a
```

---

## 5. Important Processes

| Process | PID | Role | Security Note |
|---------|:---:|------|--------------|
| **systemd** | 1 | Init system, manages services | Controls all service startup |
| **kthreadd** | 2 | Kernel thread daemon | Kernel-space threads |
| **sshd** | varies | SSH server | Remote access — high-value target |
| **rsyslogd** | varies | Logging daemon | Attacker may try to stop/modify |
| **cron** | varies | Scheduled tasks | Persistence mechanism for attackers |

```bash
# View process tree
ps auxf
pstree -p

# View all listening services
ss -tlnp
netstat -tlnp

# View open files by process
lsof -p <PID>
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Architecture** | Hardware → Kernel → Libraries → Applications |
| **Kernel** | Ring 0 — full system access, exploit = game over |
| **System Calls** | Bridge between user space and kernel |
| **Boot Security** | Secure Boot, GRUB password, minimal services |
| **LSM** | SELinux/AppArmor enforce mandatory access control |
| **Key Commands** | `uname`, `lsmod`, `strace`, `ps`, `ss` |

---

## Quick Revision Questions

1. **What is the difference between user space and kernel space?**
2. **Why are kernel vulnerabilities more dangerous than user-space bugs?**
3. **What role does systemd play in the boot process?**
4. **How can Secure Boot protect against boot-level attacks?**
5. **What command shows system calls made by a process?**

---

[Next: Security-Relevant Directories →](02-security-relevant-directories.md)

---

*Unit 1 - Topic 1 of 5 | [Back to README](../README.md)*
