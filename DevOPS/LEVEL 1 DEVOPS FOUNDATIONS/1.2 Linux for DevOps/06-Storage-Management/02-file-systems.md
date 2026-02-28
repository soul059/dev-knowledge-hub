# Chapter 2: File Systems

[← Previous: Partitioning & Disk Management](01-partitioning-disk-management.md) | [Back to README](../README.md) | [Next: Mounting & fstab →](03-mounting-and-fstab.md)

---

## Overview

A file system organizes how data is stored and retrieved on a partition. Choosing the right file system and understanding formatting, tuning, and repair is crucial for server reliability and performance in production environments.

---

## 2.1 Linux File System Types

```
┌──────────────────────────────────────────────────────────────┐
│              LINUX FILE SYSTEM COMPARISON                     │
├──────────┬────────────┬────────────┬────────────┬────────────┤
│ Feature  │ ext4       │ XFS        │ Btrfs      │ tmpfs      │
├──────────┼────────────┼────────────┼────────────┼────────────┤
│ Default  │ Ubuntu,    │ RHEL 8+,   │ SUSE,      │ /tmp,      │
│ distro   │ Debian     │ CentOS 8+  │ Fedora opt │ /dev/shm   │
├──────────┼────────────┼────────────┼────────────┼────────────┤
│ Max file │ 16 TB      │ 8 EB       │ 16 EB      │ RAM-based  │
│ Max FS   │ 1 EB       │ 8 EB       │ 16 EB      │            │
├──────────┼────────────┼────────────┼────────────┼────────────┤
│ Journal  │ Yes        │ Yes        │ CoW        │ N/A        │
├──────────┼────────────┼────────────┼────────────┼────────────┤
│ Shrink   │ Yes        │ No         │ Yes        │ N/A        │
│ Grow     │ Yes        │ Yes        │ Yes        │ N/A        │
├──────────┼────────────┼────────────┼────────────┼────────────┤
│ Snapshots│ No         │ No         │ Yes        │ N/A        │
├──────────┼────────────┼────────────┼────────────┼────────────┤
│ Best for │ General,   │ Large files│ Snapshots, │ Temp data, │
│          │ databases  │ parallel IO│ subvolumes │ caches     │
├──────────┼────────────┼────────────┼────────────┼────────────┤
│ DevOps   │ Most       │ Performance│ Advanced   │ Build      │
│ use      │ workloads  │ critical   │ storage    │ artifacts  │
└──────────┴────────────┴────────────┴────────────┴────────────┘
```

---

## 2.2 Creating File Systems

```bash
# Format with ext4
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 -L "app-data" /dev/sdb1          # With label

# Format with XFS
sudo mkfs.xfs /dev/sdb1
sudo mkfs.xfs -L "logs" -f /dev/sdb1            # Force + label

# Format with Btrfs
sudo mkfs.btrfs -L "backups" /dev/sdb1

# Check file system type
lsblk -f
blkid /dev/sdb1
file -sL /dev/sdb1
```

```
┌──────────────────────────────────────────────────────────────┐
│          FILE SYSTEM STRUCTURE (ext4)                         │
│                                                                │
│  ┌───────────────────────────────────────────────────────┐   │
│  │ Superblock                                            │   │
│  │ (FS metadata: size, block count, mount count...)      │   │
│  ├───────────────────────────────────────────────────────┤   │
│  │ Block Group 0                                         │   │
│  │ ┌─────────────┬──────────┬────────┬─────────────┐    │   │
│  │ │ Group Desc  │ Bitmaps  │ Inodes │ Data Blocks │    │   │
│  │ └─────────────┴──────────┴────────┴─────────────┘    │   │
│  ├───────────────────────────────────────────────────────┤   │
│  │ Block Group 1                                         │   │
│  │ ┌─────────────┬──────────┬────────┬─────────────┐    │   │
│  │ │ Group Desc  │ Bitmaps  │ Inodes │ Data Blocks │    │   │
│  │ └─────────────┴──────────┴────────┴─────────────┘    │   │
│  ├───────────────────────────────────────────────────────┤   │
│  │ ... more block groups ...                             │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                                │
│  Inode = metadata (permissions, size, timestamps, pointers)  │
│  Data Blocks = actual file content                            │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.3 File System Tuning

```bash
# ext4 tuning
sudo tune2fs -l /dev/sda1                   # View FS parameters
sudo tune2fs -c 30 /dev/sda1               # Check every 30 mounts
sudo tune2fs -i 30d /dev/sda1              # Check every 30 days
sudo tune2fs -L "new-label" /dev/sda1      # Change label
sudo tune2fs -m 1 /dev/sda1               # Reserved blocks: 1% (default 5%)
# Reducing to 1% on data disks saves space

# XFS tuning
xfs_info /dev/sda1                          # View XFS info
sudo xfs_admin -L "new-label" /dev/sda1    # Change label

# View inode usage
df -i                                       # Inode usage per mount
```

---

## 2.4 File System Check & Repair

```bash
# ext4 check (partition must be UNMOUNTED)
sudo umount /dev/sdb1
sudo fsck /dev/sdb1                         # Check
sudo fsck -y /dev/sdb1                      # Auto-fix
sudo e2fsck -f /dev/sdb1                    # Force check

# XFS check
sudo xfs_repair /dev/sdb1                   # Repair
sudo xfs_repair -n /dev/sdb1               # Dry run (check only)

# Check for bad blocks
sudo badblocks -sv /dev/sdb

# NEVER run fsck on mounted file systems!
```

---

## 2.5 Disk Usage Commands

```bash
# Disk free space
df -h                          # Human-readable
df -hT                         # Include FS type
df -h /data                    # Specific mount

# Directory size
du -sh /var/log                # Summary of directory
du -sh /var/log/*              # Each item in directory
du -sh /var/log/* | sort -rh   # Sorted largest first
du -h --max-depth=1 /          # Top-level directory sizes

# Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null
find /var -type f -size +50M | head -20

# ncdu — interactive disk usage (install: apt install ncdu)
ncdu /var
```

---

## 2.6 Real-World Scenario: Handling "No Space Left on Device"

```bash
#!/bin/bash
# disk-cleanup.sh — Emergency disk space recovery

echo "=== Disk Space Emergency Cleanup ==="

# 1. Check current usage
echo -e "\n--- Current Usage ---"
df -hT | head -5

# 2. Find largest directories
echo -e "\n--- Top 10 Largest Directories ---"
du -h --max-depth=2 / 2>/dev/null | sort -rh | head -10

# 3. Find large files
echo -e "\n--- Files > 100MB ---"
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh | head -10

# 4. Common cleanup targets
echo -e "\n--- Safe Cleanup Actions ---"

# Old log files
echo "Log files:"
du -sh /var/log/
# sudo journalctl --vacuum-size=200M

# Package cache
echo "APT cache:"
du -sh /var/cache/apt/
# sudo apt clean

# Old kernels (Ubuntu)
# dpkg -l 'linux-image-*' | grep '^ii'
# sudo apt autoremove

# Docker cleanup
if command -v docker &>/dev/null; then
    echo "Docker usage:"
    docker system df
    # docker system prune -af
fi

# Temp files
echo "Temp files:"
du -sh /tmp/
# find /tmp -type f -atime +7 -delete

# Check for deleted but open files (still consuming space)
echo -e "\n--- Deleted Files Still Open ---"
sudo lsof +D /var/log 2>/dev/null | grep "(deleted)" | head -5
# Restart the service holding the file to free space

echo -e "\n=== Cleanup Complete ==="
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| "No space left on device" | `df -h` shows 100% | Clean logs, temp files, Docker images |
| "No space" but `df` shows space | Inodes exhausted | Check `df -i`; delete many small files |
| Deleted file still using space | Open file descriptor | `lsof +D /path \| grep deleted`; restart process |
| Slow file system | Fragmentation or bad disk | Check with `hdparm -t`; defrag or replace |
| Can't format disk | Disk busy | Unmount all partitions first |
| Wrong FS type for workload | Performance issues | XFS for large files; ext4 for general use |

---

## Summary Table

| Command | Purpose | Example |
|---------|---------|---------|
| `mkfs.ext4` | Create ext4 FS | `mkfs.ext4 -L data /dev/sdb1` |
| `mkfs.xfs` | Create XFS FS | `mkfs.xfs /dev/sdb1` |
| `fsck` | Check/repair ext FS | `fsck -y /dev/sdb1` |
| `xfs_repair` | Check/repair XFS | `xfs_repair /dev/sdb1` |
| `tune2fs` | Tune ext4 parameters | `tune2fs -m 1 /dev/sdb1` |
| `df` | Disk free space | `df -hT` |
| `du` | Directory usage | `du -sh /var/log/*` |
| `blkid` | Show FS type & UUID | `blkid` |

---

## Quick Revision Questions

1. **What are the key differences between ext4 and XFS?**
2. **What does `df -i` show and when is it useful?**
3. **Why should you never run `fsck` on a mounted file system?**
4. **A disk shows "no space" but only 50% used according to `df -h`. What could be the cause?**
5. **What does `tune2fs -m 1 /dev/sdb1` do and why would you use it?**
6. **How do you recover space from a deleted but still-open log file?**

---

[← Previous: Partitioning & Disk Management](01-partitioning-disk-management.md) | [Back to README](../README.md) | [Next: Mounting & fstab →](03-mounting-and-fstab.md)
