# Forensic Tools

## Unit 7 - Topic 5 | Linux Security Tools

---

## Overview

**Digital forensics** on Linux involves collecting, preserving, and analyzing evidence from systems, files, and network traffic. Linux provides both **native tools** and **specialized forensic suites** for incident response, malware analysis, and legal investigations.

---

## 1. Forensic Principles

```
┌──────────────────────────────────────────────────────────────┐
│              FORENSIC INVESTIGATION PHASES                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. IDENTIFICATION                                           │
│     Determine what happened and what evidence exists         │
│                                                              │
│  2. PRESERVATION                                             │
│     Create forensic images (don't modify original!)          │
│     Document chain of custody                                │
│                                                              │
│  3. COLLECTION                                               │
│     Gather volatile data (RAM, connections, processes)       │
│     Gather non-volatile data (disk images, logs)             │
│                                                              │
│  4. ANALYSIS                                                 │
│     Examine evidence using forensic tools                    │
│     Timeline analysis, artifact recovery                     │
│                                                              │
│  5. REPORTING                                                │
│     Document findings, methodology, conclusions              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Disk Forensics

```bash
# === DISK IMAGING ===
# dd — raw disk copy
sudo dd if=/dev/sda of=/mnt/evidence/disk.img bs=4M status=progress

# dc3dd — forensic dd with hashing
sudo dc3dd if=/dev/sda of=/mnt/evidence/disk.img hash=sha256 log=imaging.log

# Verify image integrity
sha256sum /dev/sda
sha256sum /mnt/evidence/disk.img     # Must match!

# === MOUNT FORENSIC IMAGE (READ-ONLY) ===
sudo mount -o ro,loop,noexec disk.img /mnt/evidence

# === FILE RECOVERY ===
# Foremost — file carving
foremost -i disk.img -o recovered/

# Scalpel — faster file carving
scalpel disk.img -o recovered/

# PhotoRec — file recovery
photorec disk.img

# TestDisk — partition recovery
testdisk disk.img
```

---

## 3. Memory Forensics

```bash
# === MEMORY ACQUISITION ===
# LiME (Linux Memory Extractor)
sudo insmod lime.ko "path=/mnt/evidence/memory.lime format=lime"

# fmem
sudo modprobe fmem
sudo dd if=/dev/fmem of=/mnt/evidence/memory.raw bs=1M

# /proc/kcore (live system)
sudo cp /proc/kcore /mnt/evidence/kcore

# === MEMORY ANALYSIS (Volatility) ===
# Volatility 3
vol -f memory.lime linux.pslist          # Process list
vol -f memory.lime linux.pstree          # Process tree
vol -f memory.lime linux.netstat         # Network connections
vol -f memory.lime linux.bash            # Bash history from memory
vol -f memory.lime linux.lsmod           # Loaded kernel modules
vol -f memory.lime linux.check_syscall   # Detect rootkits (hooked syscalls)

# What to look for in memory:
# • Running processes (malware)
# • Network connections (C2 callbacks)
# • Loaded modules (rootkits)
# • Credentials in memory
# • Command history
```

---

## 4. File System Analysis

```bash
# === TIMELINE ANALYSIS ===
# Find recently modified files
find / -mtime -1 -type f 2>/dev/null    # Modified in last 24 hours
find / -ctime -1 -type f 2>/dev/null    # Changed in last 24 hours

# === FILE METADATA ===
stat suspicious_file                      # Full file metadata
file suspicious_file                      # File type identification
strings suspicious_file | head -50        # Readable strings
xxd suspicious_file | head -20           # Hex dump

# === HASH FILES ===
md5sum suspicious_file                    # MD5 hash
sha256sum suspicious_file                 # SHA-256 hash
# Compare against VirusTotal, malware databases

# === DELETED FILE RECOVERY ===
# ext4 recovery with extundelete
extundelete /dev/sda1 --restore-all

# === SLEUTH KIT (TSK) ===
fls -r disk.img                          # List files (including deleted)
icat disk.img 12345                      # Extract file by inode
mactime -b timeline.body > timeline.csv  # Create timeline
ils disk.img                             # List inodes
blkcat disk.img 1000                     # View block content
```

---

## 5. Log Forensics

```bash
# === IMPORTANT LOG FILES ===
/var/log/auth.log          # Authentication events
/var/log/syslog            # System messages
/var/log/kern.log          # Kernel messages
/var/log/secure            # Security events (RHEL)
/var/log/lastlog           # Last login for each user
/var/log/wtmp              # Login/logout history (binary)
/var/log/btmp              # Failed login attempts (binary)

# Read binary logs
last                       # Parse /var/log/wtmp
lastb                      # Parse /var/log/btmp (failed logins)
lastlog                    # Last login times

# Search for suspicious activity
grep "Failed password" /var/log/auth.log | tail -20
grep "Accepted password" /var/log/auth.log | tail -20
grep "sudo" /var/log/auth.log | tail -20
grep "useradd\|userdel\|usermod" /var/log/auth.log
```

---

## Summary Table

| Tool | Purpose |
|------|---------|
| **dd / dc3dd** | Disk imaging |
| **foremost / scalpel** | File carving / recovery |
| **Volatility** | Memory forensics |
| **LiME** | Linux memory acquisition |
| **Sleuth Kit** | File system analysis |
| **strings / file** | Basic file analysis |

---

## Quick Revision Questions

1. **Why must forensic images be mounted read-only?**
2. **What is file carving and which tools perform it?**
3. **How do you acquire memory from a running Linux system?**
4. **What does Volatility's `linux.pslist` plugin show?**
5. **How do you view failed login attempts?**

---

[← Previous: Traffic Analysis](04-traffic-analysis.md) | [Next: Security Distributions →](06-security-distributions.md)

---

*Unit 7 - Topic 5 of 6 | [Back to README](../README.md)*
