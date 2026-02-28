# Chapter 1: Partitioning & Disk Management

[← Previous: Network Troubleshooting](../05-Networking/07-network-troubleshooting.md) | [Back to README](../README.md) | [Next: File Systems →](02-file-systems.md)

---

## Overview

Disk management is essential for DevOps engineers setting up servers, expanding storage, and managing data volumes. This chapter covers disk identification, partitioning with `fdisk` and `parted`, and partition table formats (MBR vs GPT).

---

## 1.1 Disk Identification

```bash
# List all block devices
lsblk
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0   50G  0 disk
# ├─sda1   8:1    0    1G  0 part /boot
# ├─sda2   8:2    0   20G  0 part /
# └─sda3   8:3    0   29G  0 part /home
# sdb      8:16   0  100G  0 disk
# sr0     11:0    1 1024M  0 rom

# Detailed disk info
sudo fdisk -l
sudo fdisk -l /dev/sdb           # Specific disk

# Disk model and serial
lsblk -o NAME,SIZE,TYPE,MODEL,SERIAL

# By UUID
blkid
ls -l /dev/disk/by-uuid/
ls -l /dev/disk/by-id/
```

```
┌──────────────────────────────────────────────────────────────┐
│              LINUX DISK NAMING CONVENTION                     │
│                                                                │
│  /dev/sd*     → SCSI/SATA disks                              │
│  sda, sdb     → First disk, second disk                      │
│  sda1, sda2   → Partitions on sda                            │
│                                                                │
│  /dev/nvme*   → NVMe SSDs                                    │
│  nvme0n1      → First NVMe drive                             │
│  nvme0n1p1    → First partition on nvme0n1                    │
│                                                                │
│  /dev/vd*     → Virtio (KVM virtual disks)                   │
│  /dev/xvd*    → Xen (AWS EC2 older instances)                │
│                                                                │
│  /dev/dm-*    → Device mapper (LVM logical volumes)          │
│  /dev/md*     → Software RAID arrays                         │
│                                                                │
│  Storage Stack:                                               │
│  ┌──────────┐                                                 │
│  │   App    │                                                 │
│  ├──────────┤                                                 │
│  │   FS     │  ext4, xfs, btrfs                              │
│  ├──────────┤                                                 │
│  │   LVM    │  Logical volume (optional)                     │
│  ├──────────┤                                                 │
│  │ Partition│  sda1, sda2                                    │
│  ├──────────┤                                                 │
│  │   Disk   │  /dev/sda                                      │
│  └──────────┘                                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 MBR vs GPT

```
┌──────────────────────────────────────────────────────────────┐
│              MBR vs GPT PARTITION TABLES                      │
├──────────────────┬──────────────────┬────────────────────────┤
│ Feature          │ MBR              │ GPT                    │
├──────────────────┼──────────────────┼────────────────────────┤
│ Max disk size    │ 2 TB             │ 9.4 ZB (huge)          │
│ Max partitions   │ 4 primary        │ 128 partitions         │
│                  │ (or 3+1 extended)│                        │
│ Boot firmware    │ BIOS             │ UEFI (or BIOS+GPT)    │
│ Redundancy       │ Single copy      │ Backup header at end   │
│ CRC checking     │ No               │ Yes (data integrity)   │
│ Recommendation   │ Legacy systems   │ Modern systems (use!)  │
└──────────────────┴──────────────────┴────────────────────────┘
```

---

## 1.3 Partitioning with `fdisk` (MBR)

```bash
# Open fdisk for a disk
sudo fdisk /dev/sdb

# Interactive commands:
# m  → Help menu
# p  → Print partition table
# n  → New partition
# d  → Delete partition
# t  → Change partition type
# w  → Write changes and exit
# q  → Quit without saving

# Example: Create a new partition
sudo fdisk /dev/sdb
# Command: n
# Partition type: p (primary)
# Partition number: 1
# First sector: (Enter for default)
# Last sector: +20G
# Command: w

# Verify
lsblk
sudo fdisk -l /dev/sdb
```

---

## 1.4 Partitioning with `parted` (GPT)

```bash
# Open parted
sudo parted /dev/sdb

# View partition table
(parted) print

# Create GPT label
(parted) mklabel gpt

# Create partition
(parted) mkpart primary ext4 0% 50%
(parted) mkpart primary xfs 50% 100%

# View result
(parted) print
(parted) quit

# Non-interactive one-liners
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 0% 100%
```

---

## 1.5 Inform Kernel of Changes

```bash
# After partitioning, tell the kernel
sudo partprobe /dev/sdb

# Or
sudo partx -a /dev/sdb

# Verify
lsblk /dev/sdb
cat /proc/partitions
```

---

## 1.6 Real-World Scenario: Adding a Data Disk to a Server

```bash
#!/bin/bash
# add-data-disk.sh — Partition, format, and mount a new disk

DISK="/dev/sdb"
MOUNT="/data"

echo "=== Adding data disk: $DISK ==="

# 1. Check disk exists
if [ ! -b "$DISK" ]; then
    echo "ERROR: $DISK not found"
    exit 1
fi

# 2. Create GPT partition table and single partition
sudo parted -s "$DISK" mklabel gpt
sudo parted -s "$DISK" mkpart primary ext4 0% 100%
sudo partprobe "$DISK"
sleep 2

# 3. Format with ext4
sudo mkfs.ext4 -L data "${DISK}1"

# 4. Create mount point and mount
sudo mkdir -p "$MOUNT"
sudo mount "${DISK}1" "$MOUNT"

# 5. Add to fstab for persistence
UUID=$(blkid -s UUID -o value "${DISK}1")
echo "UUID=$UUID  $MOUNT  ext4  defaults,noatime  0 2" | sudo tee -a /etc/fstab

# 6. Verify
df -h "$MOUNT"
echo "=== Disk ready at $MOUNT ==="
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Disk not showing up | `lsblk` or `fdisk -l` | Check hardware, rescan: `echo "- - -" > /sys/class/scsi_host/host0/scan` |
| Partition changes not visible | Kernel using old table | Run `sudo partprobe /dev/sdX` |
| "Disk is too large for MBR" | Disk > 2TB with MBR | Switch to GPT: `parted mklabel gpt` |
| Can't create 5th partition (MBR) | MBR limit: 4 primary | Use extended partition or switch to GPT |
| "Device is busy" | Partition is mounted | Unmount first: `sudo umount /dev/sdX1` |
| Wrong partition created | Incorrect size/type | Delete with `fdisk d` and recreate |

---

## Summary Table

| Command | Purpose | Example |
|---------|---------|---------|
| `lsblk` | List block devices | `lsblk -f` |
| `blkid` | Show UUIDs and FS types | `blkid /dev/sda1` |
| `fdisk` | MBR partitioning | `sudo fdisk /dev/sdb` |
| `parted` | GPT partitioning | `sudo parted /dev/sdb` |
| `partprobe` | Refresh kernel partition table | `sudo partprobe /dev/sdb` |
| `fdisk -l` | List all partitions | `sudo fdisk -l` |

---

## Quick Revision Questions

1. **What is the difference between MBR and GPT? When would you use each?**
2. **What is the maximum disk size MBR supports? How many primary partitions?**
3. **What is the naming convention for NVMe drives vs SATA drives in Linux?**
4. **After creating a partition with `fdisk`, what must you do before the kernel sees the changes?**
5. **How do you find the UUID of a partition? Why use UUID instead of device name in `/etc/fstab`?**
6. **Write the steps to add a new 100GB disk as a single ext4 partition mounted at `/data`.**

---

[← Previous: Network Troubleshooting](../05-Networking/07-network-troubleshooting.md) | [Back to README](../README.md) | [Next: File Systems →](02-file-systems.md)
