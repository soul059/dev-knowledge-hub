# File System Hierarchy

## Unit 4 - Topic 1 | Linux File System Security

---

## Overview

The **Filesystem Hierarchy Standard (FHS)** defines the directory structure of Linux. From a security perspective, understanding the hierarchy is essential for knowing **where sensitive data resides**, **what needs protection**, and **where attackers hide malware or persistence mechanisms**.

---

## 1. FHS Directory Map

```
/                           Root of the filesystem
├── bin/   → /usr/bin/      Essential user binaries
├── sbin/  → /usr/sbin/     Essential system binaries
├── boot/                   Bootloader, kernel, initramfs
├── dev/                    Device files (/dev/null, /dev/sda)
├── etc/                    System configuration files
├── home/                   User home directories
├── lib/   → /usr/lib/      Shared libraries
├── media/                  Removable media mount points
├── mnt/                    Temporary mount points
├── opt/                    Third-party/optional software
├── proc/                   Virtual FS — process/kernel info
├── root/                   Root user's home directory
├── run/                    Runtime data (PIDs, sockets)
├── srv/                    Service data (web, FTP)
├── sys/                    Virtual FS — hardware/kernel
├── tmp/                    Temporary files (world-writable!)
├── usr/                    User programs, libraries, docs
│   ├── bin/                User binaries
│   ├── sbin/               System admin binaries
│   ├── lib/                Libraries
│   ├── local/              Locally compiled software
│   └── share/              Shared data
└── var/                    Variable data
    ├── log/                Log files
    ├── spool/              Mail, cron queues
    ├── tmp/                Persistent temp files
    ├── lib/                State data (databases)
    └── www/                Web server files
```

---

## 2. Security Classification

| Directory | Security Level | Threat |
|-----------|:---:|--------|
| `/etc/shadow` | 🔴 Critical | Password hashes |
| `/etc/sudoers` | 🔴 Critical | Privilege escalation config |
| `/boot/` | 🔴 Critical | Kernel/bootloader tampering |
| `/root/` | 🔴 Critical | Root user's files, SSH keys |
| `/var/log/` | 🟡 High | Forensic evidence |
| `/home/` | 🟡 High | User data, SSH keys |
| `/etc/ssh/` | 🟡 High | SSH configuration, host keys |
| `/tmp/` | 🟡 High | Malware staging area |
| `/dev/shm/` | 🟡 High | In-memory file system abuse |
| `/usr/local/` | 🟢 Medium | Custom software |
| `/opt/` | 🟢 Medium | Third-party applications |

---

## 3. Separate Partitions for Security

```
RECOMMENDED PARTITION SCHEME:
┌───────────┬──────────┬──────────────────────────────────┐
│ Partition │ Size     │ Security Benefit                 │
├───────────┼──────────┼──────────────────────────────────┤
│ /         │ 20-30 GB │ Root filesystem                  │
│ /boot     │ 512 MB   │ Protect bootloader integrity     │
│ /home     │ Varies   │ nosuid,nodev — prevent SUID      │
│ /tmp      │ 2-5 GB   │ noexec,nosuid,nodev              │
│ /var      │ 10-20 GB │ Prevent log filling root FS      │
│ /var/log  │ 5-10 GB  │ Isolate logs from other data     │
│ /var/tmp  │ 2-5 GB   │ noexec,nosuid,nodev              │
│ swap      │ 2x RAM   │ Encrypt swap for data protection │
└───────────┴──────────┴──────────────────────────────────┘

WHY SEPARATE PARTITIONS?
• /tmp full won't crash the system (separate from /)
• /var/log full won't prevent logins
• Mount options (noexec, nosuid) per partition
• Disk quotas per partition
• Different encryption per sensitivity level
```

---

## 4. File System Types

| Filesystem | Features | Security Notes |
|-----------|---------|---------------|
| **ext4** | Default Linux FS, journaling | Supports ACLs, attrs |
| **XFS** | High performance, scalable | Used by RHEL default |
| **Btrfs** | Snapshots, checksums | Integrity verification |
| **tmpfs** | RAM-based (volatile) | /tmp, /dev/shm — fast but lost on reboot |
| **ZFS** | Checksums, snapshots, encryption | Most features, needs extra setup |
| **OverlayFS** | Layered filesystem | Used by Docker containers |

---

## 5. Security Commands

```bash
# View filesystem info
df -h                                 # Disk usage
lsblk                                # Block devices
mount                                 # Currently mounted filesystems
cat /etc/fstab                        # Persistent mount config
findmnt                               # Tree view of mounts

# Find files by type or attribute
find / -type f -perm -4000            # SUID files
find / -type f -perm -2000            # SGID files
find / -type f -perm -o+w            # World-writable files
find / -type d -perm -o+w            # World-writable directories
find / -nouser -o -nogroup            # Files with no owner
find / -name "*.php" -o -name "*.py"  # Script files
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **FHS** | Standard Linux directory structure |
| **/etc/** | Configuration — most security-critical |
| **/tmp/** | World-writable — mount with noexec |
| **Separate partitions** | Isolate /tmp, /var/log, /home |
| **Mount options** | noexec, nosuid, nodev for security |
| **find** | Audit permissions, SUID, world-writable files |

---

## Quick Revision Questions

1. **Why should /tmp be on a separate partition?**
2. **What mount options harden /tmp?**
3. **Which directory contains password hashes?**
4. **Why separate /var/log onto its own partition?**
5. **What command shows all currently mounted filesystems?**

---

[Next: Mount Options for Security →](02-mount-options-for-security.md)

---

*Unit 4 - Topic 1 of 6 | [Back to README](../README.md)*
