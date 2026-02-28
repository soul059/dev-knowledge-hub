# Chapter 3: File System Hierarchy

[← Previous: Linux Distributions](02-linux-distributions.md) | [Back to README](../README.md) | [Next: Boot Process →](04-boot-process.md)

---

## Overview

The Linux file system follows the **Filesystem Hierarchy Standard (FHS)**, which defines the directory structure and directory contents. Everything in Linux is a file — including devices, processes, and sockets. Understanding this hierarchy is crucial for DevOps engineers who manage servers, debug issues, and automate deployments.

---

## 3.1 The Root of Everything: `/`

```
/  (root)
├── bin/       ──→ Essential user binaries (ls, cp, cat)
├── boot/      ──→ Boot loader files (kernel, grub)
├── dev/       ──→ Device files (disks, terminals)
├── etc/       ──→ System configuration files
├── home/      ──→ User home directories
├── lib/       ──→ Essential shared libraries
├── lib64/     ──→ 64-bit libraries
├── media/     ──→ Mount points for removable media
├── mnt/       ──→ Temporary mount points
├── opt/       ──→ Optional/third-party software
├── proc/      ──→ Process & kernel info (virtual)
├── root/      ──→ Root user's home directory
├── run/       ──→ Runtime data (PIDs, sockets)
├── sbin/      ──→ System binaries (fdisk, iptables)
├── srv/       ──→ Service data (web, FTP)
├── sys/       ──→ Kernel & device info (virtual)
├── tmp/       ──→ Temporary files (cleared on reboot)
├── usr/       ──→ User programs & data
│   ├── bin/   ──→ User binaries
│   ├── lib/   ──→ Libraries
│   ├── local/ ──→ Locally installed software
│   ├── sbin/  ──→ Non-essential system binaries
│   └── share/ ──→ Architecture-independent data
└── var/       ──→ Variable data (logs, mail, spool)
    ├── log/   ──→ Log files
    ├── cache/ ──→ Application cache
    ├── lib/   ──→ Variable state information
    ├── run/   ──→ Runtime (symlink to /run)
    └── tmp/   ──→ Persistent temporary files
```

---

## 3.2 Detailed Directory Breakdown

### `/etc` — Configuration Central (Most Important for DevOps)

```
/etc/
├── hostname               ──→ System hostname
├── hosts                  ──→ Static hostname-to-IP mapping
├── resolv.conf            ──→ DNS resolver configuration
├── fstab                  ──→ Filesystem mount table
├── passwd                 ──→ User account info
├── shadow                 ──→ Encrypted passwords
├── group                  ──→ Group definitions
├── sudoers                ──→ sudo permissions
├── ssh/
│   ├── sshd_config        ──→ SSH server configuration
│   └── ssh_config         ──→ SSH client configuration
├── nginx/
│   └── nginx.conf         ──→ Nginx configuration
├── systemd/
│   └── system/            ──→ Systemd unit files
├── crontab                ──→ System cron jobs
├── sysctl.conf            ──→ Kernel parameters
├── security/
│   └── limits.conf        ──→ User resource limits
└── default/
    └── grub               ──→ GRUB boot loader config
```

```bash
# Common /etc operations for DevOps
cat /etc/hostname                     # Check hostname
cat /etc/hosts                        # View host mappings
cat /etc/resolv.conf                  # Check DNS settings
cat /etc/fstab                        # View mount table
grep -v '^#' /etc/ssh/sshd_config    # View active SSH config
```

### `/var` — Variable Data

```
/var/
├── log/                               
│   ├── syslog          ──→ System log (Ubuntu/Debian)
│   ├── messages        ──→ System log (RHEL/CentOS)
│   ├── auth.log        ──→ Authentication log
│   ├── secure          ──→ Auth log (RHEL)
│   ├── nginx/          ──→ Nginx logs
│   │   ├── access.log
│   │   └── error.log
│   └── journal/        ──→ systemd journal
├── cache/
│   └── apt/            ──→ APT package cache
├── lib/
│   ├── docker/         ──→ Docker data
│   └── mysql/          ──→ MySQL databases
└── spool/
    └── cron/           ──→ User crontabs
```

```bash
# Common /var operations
tail -f /var/log/syslog                # Live log monitoring
du -sh /var/log/*                      # Check log sizes
ls -la /var/lib/docker/                # Docker storage
journalctl -u nginx --since "1 hour ago"  # Systemd logs
```

### `/proc` — Virtual Filesystem (Process Info)

```
/proc/
├── 1/                    ──→ PID 1 (init/systemd) info
│   ├── cmdline           ──→ Command that started process
│   ├── status            ──→ Process status
│   ├── fd/               ──→ Open file descriptors
│   └── environ           ──→ Environment variables
├── cpuinfo               ──→ CPU information
├── meminfo               ──→ Memory information
├── loadavg               ──→ System load averages
├── uptime                ──→ System uptime
├── filesystems           ──→ Supported filesystems
├── mounts                ──→ Currently mounted filesystems
├── net/
│   ├── tcp               ──→ Active TCP connections
│   └── dev               ──→ Network interface stats
└── sys/
    ├── fs/               ──→ Filesystem parameters
    ├── net/              ──→ Network parameters
    └── vm/               ──→ Virtual memory parameters
```

```bash
# Useful /proc reads for DevOps
cat /proc/cpuinfo | grep "model name"   # CPU model
cat /proc/meminfo | head -5             # Memory stats
cat /proc/loadavg                        # Load averages
cat /proc/uptime                         # Uptime in seconds
cat /proc/sys/fs/file-max               # Max open files
cat /proc/sys/net/ipv4/ip_forward       # IP forwarding status
ls /proc/$(pgrep nginx | head -1)/fd    # Nginx open files
```

### `/dev` — Device Files

```
/dev/
├── sda                   ──→ First SATA/SCSI disk
│   ├── sda1              ──→ First partition
│   └── sda2              ──→ Second partition
├── nvme0n1               ──→ NVMe disk
│   └── nvme0n1p1         ──→ First NVMe partition
├── xvda                  ──→ Xen virtual disk (AWS)
├── null                  ──→ Discard output (/dev/null)
├── zero                  ──→ Source of zero bytes
├── random                ──→ Random number generator
├── urandom               ──→ Non-blocking random
├── tty                   ──→ Current terminal
├── stdin   → fd/0        ──→ Standard input
├── stdout  → fd/1        ──→ Standard output
└── stderr  → fd/2        ──→ Standard error
```

```bash
# Device operations
lsblk                          # List block devices
ls -la /dev/sd*                # List SATA devices
ls -la /dev/nvme*              # List NVMe devices

# Redirect output to /dev/null (discard)
command > /dev/null 2>&1

# Generate random data
head -c 32 /dev/urandom | base64   # Generate random string
```

---

## 3.3 File Types in Linux

```
┌──────────────────────────────────────────────────────────┐
│                   LINUX FILE TYPES                        │
├──────────┬───────────────────┬───────────────────────────┤
│ Symbol   │ Type              │ Example                   │
├──────────┼───────────────────┼───────────────────────────┤
│    -     │ Regular file      │ /etc/passwd               │
│    d     │ Directory         │ /home/user                │
│    l     │ Symbolic link     │ /usr/bin/python → python3 │
│    c     │ Character device  │ /dev/tty                  │
│    b     │ Block device      │ /dev/sda                  │
│    s     │ Socket            │ /var/run/docker.sock      │
│    p     │ Named pipe (FIFO) │ Created with mkfifo       │
└──────────┴───────────────────┴───────────────────────────┘
```

```bash
# Identify file types
file /etc/passwd              # Regular text file
file /dev/sda                 # Block device
file /usr/bin/ls              # ELF executable
ls -la /dev/tty               # crw--- (character device)
ls -la /var/run/docker.sock   # srw--- (socket)

# Find all symlinks in a directory
find /usr/bin -type l | head -10

# Find all block devices
find /dev -type b 2>/dev/null
```

---

## 3.4 Important Paths for DevOps Engineers

```
┌─────────────────────────────────────────────────────────────┐
│              DEVOPS-CRITICAL PATHS                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CONFIGURATION:                                              │
│  /etc/ssh/sshd_config          SSH server settings          │
│  /etc/nginx/nginx.conf         Nginx configuration          │
│  /etc/systemd/system/          Custom service units         │
│  /etc/crontab                  System cron jobs             │
│  /etc/fstab                    Filesystem mounts            │
│  /etc/hosts                    Static DNS entries           │
│  /etc/resolv.conf              DNS resolvers                │
│  /etc/sysctl.conf              Kernel parameters            │
│                                                              │
│  LOGS:                                                       │
│  /var/log/syslog               System log (Debian)          │
│  /var/log/messages             System log (RHEL)            │
│  /var/log/auth.log             Auth attempts (Debian)       │
│  /var/log/secure               Auth attempts (RHEL)         │
│                                                              │
│  APPLICATION DATA:                                           │
│  /var/lib/docker/              Docker images & containers   │
│  /opt/                         Third-party applications     │
│  /srv/                         Service data (web roots)     │
│                                                              │
│  RUNTIME:                                                    │
│  /run/                         PID files, sockets           │
│  /tmp/                         Temporary files              │
│  /proc/, /sys/                 Kernel & hardware info       │
└─────────────────────────────────────────────────────────────┘
```

---

## 3.5 Everything is a File

One of Linux's core design principles is **"everything is a file."** This means:

```
┌───────────────────────────────────────────────────────────┐
│  Linux treats EVERYTHING as a file:                       │
│                                                           │
│  Regular files    →  /etc/passwd                          │
│  Directories      →  /home/user/                          │
│  Devices          →  /dev/sda (disk)                      │
│  Processes        →  /proc/1234/status                    │
│  Network sockets  →  /var/run/docker.sock                 │
│  Kernel params    →  /proc/sys/net/ipv4/ip_forward        │
│  System info      →  /proc/cpuinfo                        │
│  Pipes            →  mkfifo /tmp/mypipe                   │
│                                                           │
│  This means you can read/write them using standard        │
│  file I/O: cat, echo, read(), write()                     │
└───────────────────────────────────────────────────────────┘
```

```bash
# Read CPU info like a file
cat /proc/cpuinfo

# Enable IP forwarding by writing to a file
echo 1 > /proc/sys/net/ipv4/ip_forward

# Check process memory by reading a file
cat /proc/$(pgrep nginx | head -1)/status | grep VmRSS
```

---

## 3.6 Real-World DevOps Scenario

### Scenario: Debugging a Full Disk on a Production Server

```bash
# 1. Check disk usage at the top level
df -h
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        50G   48G   2G   96% /

# 2. Find what's consuming space
du -sh /* 2>/dev/null | sort -rh | head -10
# 35G    /var
# 8G     /usr
# 3G     /home

# 3. Drill into /var
du -sh /var/* 2>/dev/null | sort -rh | head -5
# 30G    /var/log
# 3G     /var/lib
# 1G     /var/cache

# 4. Find the culprit log files
du -sh /var/log/* | sort -rh | head -5
# 25G    /var/log/nginx/access.log
# 3G     /var/log/syslog
# 1G     /var/log/kern.log

# 5. Solution: Rotate and truncate
sudo truncate -s 0 /var/log/nginx/access.log    # Clear log
sudo logrotate -f /etc/logrotate.d/nginx         # Force rotation

# 6. Find and delete old large files
find /var/log -name "*.gz" -mtime +30 -delete    # Delete rotated logs > 30 days

# 7. Check for deleted files still held open
lsof +L1 | grep deleted
# Restart the service holding the deleted file to reclaim space
```

---

## Troubleshooting Tips

| Issue | Diagnostic | Solution |
|-------|-----------|---------|
| Disk full (`No space left on device`) | `df -h`, `du -sh /*` | Clean logs, rotate, expand disk |
| Can't find config file | `find / -name "filename" 2>/dev/null` | Check standard paths |
| Deleted file still using space | `lsof +L1 \| grep deleted` | Restart the process |
| `/tmp` filling up | `du -sh /tmp/*`, `systemd-tmpfiles` | Clear temp files, check tmpfiles.d |
| Permission denied on `/dev/*` | `ls -la /dev/device` | Check user groups, use `udev` rules |

---

## Summary Table

| Directory | Purpose | DevOps Relevance |
|-----------|---------|-----------------|
| `/etc` | System configuration | SSH, services, cron, network config |
| `/var/log` | System and app logs | Monitoring, debugging, compliance |
| `/var/lib` | Persistent app data | Docker data, databases |
| `/proc` | Virtual process filesystem | System metrics, debugging |
| `/dev` | Device files | Disk management, I/O redirection |
| `/tmp` | Temporary files | Build artifacts, short-lived data |
| `/opt` | Third-party software | Custom app installations |
| `/usr/local` | Locally installed software | Manual installations |
| `/home` | User directories | Developer environments |
| `/boot` | Boot files | Kernel updates, GRUB |

---

## Quick Revision Questions

1. **What is the FHS?** Name the top-level directories and their purposes.
2. **Where would you look for Nginx logs?** What about SSH configuration?
3. **What is `/proc` and how is it different from a regular directory?** Give 3 useful files in `/proc`.
4. **Explain the "everything is a file" philosophy.** Give an example with a device and a process.
5. **A server is reporting "No space left on device." Walk through your troubleshooting steps.**
6. **What is the difference between `/tmp` and `/var/tmp`?** When would you use each?

---

[← Previous: Linux Distributions](02-linux-distributions.md) | [Back to README](../README.md) | [Next: Boot Process →](04-boot-process.md)
