# Chapter 3: Mounting & fstab

[← Previous: File Systems](02-file-systems.md) | [Back to README](../README.md) | [Next: LVM (Logical Volume Manager) →](04-lvm.md)

---

## Overview

Mounting attaches a file system to a directory in the Linux directory tree. The `/etc/fstab` file makes mounts persistent across reboots. Proper fstab configuration is critical — a wrong entry can prevent a server from booting.

---

## 3.1 Mounting Basics

```
┌──────────────────────────────────────────────────────────────┐
│                MOUNT CONCEPT                                  │
│                                                                │
│  Before mount:          After mount:                          │
│  /                      /                                     │
│  ├── home/              ├── home/                             │
│  ├── data/  (empty)     ├── data/  ← /dev/sdb1 attached     │
│  └── var/               │   ├── app/                         │
│                         │   ├── logs/                         │
│  /dev/sdb1              │   └── config/                      │
│  (unattached)           └── var/                              │
│                                                                │
│  mount /dev/sdb1 /data                                        │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Mount a partition
sudo mount /dev/sdb1 /data
sudo mount /dev/sdb1 /data -t ext4         # Specify FS type

# Mount with options
sudo mount -o rw,noatime /dev/sdb1 /data
sudo mount -o ro /dev/sdb1 /mnt            # Read-only
sudo mount -o remount,rw /data             # Remount with new options

# Unmount
sudo umount /data
sudo umount /dev/sdb1                       # By device
sudo umount -l /data                        # Lazy unmount (busy FS)
sudo umount -f /data                        # Force unmount (NFS)

# View current mounts
mount | column -t
findmnt                                     # Tree view
findmnt -t ext4                             # Filter by type
cat /proc/mounts                            # Kernel's view
df -hT                                      # With usage
```

---

## 3.2 Common Mount Options

```
┌──────────────────────────────────────────────────────────────┐
│              MOUNT OPTIONS REFERENCE                          │
├──────────────┬───────────────────────────────────────────────┤
│ Option       │ Description                                   │
├──────────────┼───────────────────────────────────────────────┤
│ defaults     │ rw, suid, dev, exec, auto, nouser, async      │
│ rw           │ Read-write                                     │
│ ro           │ Read-only                                      │
│ noatime      │ Don't update access time (performance boost)  │
│ nodiratime   │ Don't update directory access time             │
│ noexec       │ Cannot execute binaries (security)             │
│ nosuid       │ Ignore SUID/SGID bits (security)              │
│ nodev        │ Ignore device files (security)                 │
│ auto         │ Mount automatically with `mount -a`            │
│ noauto       │ Don't auto-mount                              │
│ user         │ Allow non-root users to mount                  │
│ nofail       │ Don't fail boot if device missing             │
│ _netdev      │ Wait for network before mounting (NFS, iSCSI)│
│ discard      │ Enable TRIM for SSDs                          │
│ x-systemd.   │ Systemd mount options                         │
│ automount    │ Mount on first access                         │
└──────────────┴───────────────────────────────────────────────┘
```

---

## 3.3 `/etc/fstab` — Persistent Mounts

```bash
# /etc/fstab format:
# <device>       <mount>  <type>  <options>         <dump>  <fsck>

# System partitions
UUID=abc123...   /        ext4    defaults           0       1
UUID=def456...   /boot    ext4    defaults           0       2
UUID=ghi789...   swap     swap    sw                 0       0

# Data partitions
UUID=xyz000...   /data    ext4    defaults,noatime   0       2
UUID=aaa111...   /logs    xfs     defaults,noatime   0       2

# NFS mount
server:/share    /nfs     nfs     defaults,_netdev   0       0

# tmpfs (RAM disk)
tmpfs            /tmp     tmpfs   defaults,size=2G   0       0

# CD-ROM
/dev/sr0         /cdrom   iso9660 noauto,user        0       0
```

```
┌──────────────────────────────────────────────────────────────┐
│              FSTAB FIELDS EXPLAINED                           │
│                                                                │
│  Field 1: Device                                              │
│  UUID=xyz...    ← Preferred (survives disk reordering)       │
│  /dev/sdb1      ← Can change if disks are added/moved       │
│  LABEL=data     ← By label                                   │
│                                                                │
│  Field 2: Mount Point                                        │
│  /data, /mnt, /home  ← Must exist as a directory             │
│                                                                │
│  Field 3: File System Type                                    │
│  ext4, xfs, nfs, swap, tmpfs, auto                           │
│                                                                │
│  Field 4: Options                                             │
│  defaults, noatime, nofail, ro, etc.                          │
│                                                                │
│  Field 5: Dump (backup)                                       │
│  0 = don't backup     1 = backup with dump                   │
│                                                                │
│  Field 6: fsck Order                                         │
│  0 = don't check      1 = root (first)     2 = other         │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.4 Working with fstab Safely

```bash
# ALWAYS test before rebooting!

# 1. Get the UUID
sudo blkid /dev/sdb1
# /dev/sdb1: UUID="a1b2c3d4-..." TYPE="ext4"

# 2. Create mount point
sudo mkdir -p /data

# 3. Add entry to fstab
echo 'UUID=a1b2c3d4-... /data ext4 defaults,noatime,nofail 0 2' | sudo tee -a /etc/fstab

# 4. Test the fstab entry (WITHOUT rebooting)
sudo mount -a                    # Mount all fstab entries
echo $?                          # 0 = success

# 5. Verify
df -h /data
findmnt /data

# CRITICAL: Use "nofail" for non-essential mounts!
# Without nofail: missing disk = system won't boot!
# With nofail: missing disk = warning but system boots
```

---

## 3.5 Swap Space

```bash
# Check current swap
swapon --show
free -h

# Create swap file (modern approach)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Add to fstab
echo '/swapfile swap swap sw 0 0' | sudo tee -a /etc/fstab

# Create swap partition
sudo mkswap /dev/sdb2
sudo swapon /dev/sdb2

# Remove swap
sudo swapoff /swapfile
sudo rm /swapfile
# Remove fstab entry

# Adjust swappiness (how aggressively to use swap)
cat /proc/sys/vm/swappiness              # Default: 60
sudo sysctl vm.swappiness=10             # Temporary
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf   # Persistent
# Servers: 10-20 (prefer RAM)
# Desktops: 60 (default)
```

---

## 3.6 Real-World Scenario: Multi-Disk Server Setup

```bash
#!/bin/bash
# server-storage-setup.sh — Standard server disk configuration

# Layout:
# /dev/sda → OS (already partitioned)
# /dev/sdb → Application data
# /dev/sdc → Logs

echo "=== Server Storage Setup ==="

# Partition sdb for app data
sudo parted -s /dev/sdb mklabel gpt
sudo parted -s /dev/sdb mkpart primary ext4 0% 100%
sudo mkfs.ext4 -L app-data /dev/sdb1
sudo mkdir -p /opt/app
UUID_APP=$(blkid -s UUID -o value /dev/sdb1)
echo "UUID=$UUID_APP /opt/app ext4 defaults,noatime,nofail 0 2" | sudo tee -a /etc/fstab

# Partition sdc for logs
sudo parted -s /dev/sdc mklabel gpt
sudo parted -s /dev/sdc mkpart primary xfs 0% 100%
sudo mkfs.xfs -L logs /dev/sdc1
sudo mkdir -p /var/log/app
UUID_LOG=$(blkid -s UUID -o value /dev/sdc1)
echo "UUID=$UUID_LOG /var/log/app xfs defaults,noatime,nofail 0 2" | sudo tee -a /etc/fstab

# Create swap file
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
echo '/swapfile swap swap sw 0 0' | sudo tee -a /etc/fstab

# Mount everything
sudo mount -a
sudo swapon -a

# Verify
echo -e "\n--- Storage Summary ---"
df -hT | grep -E "Filesystem|sdb|sdc|app"
swapon --show
echo "=== Setup Complete ==="
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Server won't boot (fstab error) | Bad fstab entry | Boot rescue mode; fix `/etc/fstab` |
| "device is busy" on umount | Files open on mount | `lsof +D /mount` or `fuser -mv /mount` |
| Mount lost after reboot | Not in fstab | Add UUID-based entry to `/etc/fstab` |
| Wrong UUID in fstab | Disk replaced/reformatted | Update UUID: `blkid` → edit fstab |
| NFS mount hangs on boot | Network not ready | Add `_netdev,nofail` options |
| Swap not active after reboot | Missing fstab entry | Add swap entry to fstab |

---

## Summary Table

| Command | Purpose | Example |
|---------|---------|---------|
| `mount` | Attach filesystem | `mount /dev/sdb1 /data` |
| `umount` | Detach filesystem | `umount /data` |
| `mount -a` | Mount all fstab entries | `mount -a` |
| `findmnt` | Show mount tree | `findmnt -t ext4` |
| `blkid` | Get UUID for fstab | `blkid /dev/sdb1` |
| `swapon` | Enable swap | `swapon /swapfile` |
| `swapoff` | Disable swap | `swapoff /swapfile` |

---

## Quick Revision Questions

1. **What are the 6 fields in `/etc/fstab`? Explain each.**
2. **Why should you use UUID instead of `/dev/sdX` in fstab?**
3. **What does the `nofail` mount option do and why is it important?**
4. **How do you test fstab changes without rebooting?**
5. **How do you create a 4GB swap file and make it persistent?**
6. **What does `noatime` do and why is it commonly used on servers?**

---

[← Previous: File Systems](02-file-systems.md) | [Back to README](../README.md) | [Next: LVM (Logical Volume Manager) →](04-lvm.md)
