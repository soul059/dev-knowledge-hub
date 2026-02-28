# Chapter 4: LVM (Logical Volume Manager)

[← Previous: Mounting & fstab](03-mounting-and-fstab.md) | [Back to README](../README.md) | [Next: RAID →](05-raid.md)

---

## Overview

LVM adds a flexible abstraction layer between physical disks and file systems. It allows you to resize volumes, add disks on the fly, take snapshots, and manage storage without downtime — a must-have skill for production server management.

---

## 4.1 LVM Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    LVM ARCHITECTURE                           │
│                                                                │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Logical Volumes (LV) — what you mount                 │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐            │  │
│  │  │ lv_root  │  │ lv_data  │  │ lv_logs  │            │  │
│  │  │  20 GB   │  │  50 GB   │  │  30 GB   │            │  │
│  │  │ (ext4)   │  │ (xfs)    │  │ (ext4)   │            │  │
│  │  └──────────┘  └──────────┘  └──────────┘            │  │
│  └────────────────────────┬───────────────────────────────┘  │
│                           │                                    │
│  ┌────────────────────────▼───────────────────────────────┐  │
│  │  Volume Group (VG) — storage pool                      │  │
│  │  ┌─────────────────────────────────────────────┐      │  │
│  │  │     vg_data (total: 200 GB)                  │      │  │
│  │  └─────────────────────────────────────────────┘      │  │
│  └────────────────────────┬───────────────────────────────┘  │
│                           │                                    │
│  ┌────────────────────────▼───────────────────────────────┐  │
│  │  Physical Volumes (PV) — actual disks/partitions       │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐            │  │
│  │  │ /dev/sdb │  │ /dev/sdc │  │ /dev/sdd │            │  │
│  │  │  50 GB   │  │  50 GB   │  │  100 GB  │            │  │
│  │  └──────────┘  └──────────┘  └──────────┘            │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                                │
│  PV → VG → LV → Filesystem → Mount Point                    │
│  (Physical) (Pool) (Virtual) (Format)   (/data)              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 Creating LVM Step by Step

```bash
# Step 1: Create Physical Volumes
sudo pvcreate /dev/sdb /dev/sdc
sudo pvs                            # List PVs
sudo pvdisplay                       # Detailed PV info

# Step 2: Create Volume Group
sudo vgcreate vg_data /dev/sdb /dev/sdc
sudo vgs                            # List VGs
sudo vgdisplay vg_data              # Detailed VG info

# Step 3: Create Logical Volumes
sudo lvcreate -n lv_app -L 30G vg_data       # Fixed size
sudo lvcreate -n lv_logs -L 20G vg_data      # Fixed size
sudo lvcreate -n lv_data -l 100%FREE vg_data # Use remaining space
sudo lvs                            # List LVs
sudo lvdisplay                       # Detailed LV info

# Step 4: Create File Systems
sudo mkfs.ext4 /dev/vg_data/lv_app
sudo mkfs.ext4 /dev/vg_data/lv_logs
sudo mkfs.xfs /dev/vg_data/lv_data

# Step 5: Mount
sudo mkdir -p /opt/app /var/log/app /data
sudo mount /dev/vg_data/lv_app /opt/app
sudo mount /dev/vg_data/lv_logs /var/log/app
sudo mount /dev/vg_data/lv_data /data

# Step 6: Add to fstab
echo '/dev/vg_data/lv_app  /opt/app      ext4 defaults,noatime 0 2' | sudo tee -a /etc/fstab
echo '/dev/vg_data/lv_logs /var/log/app  ext4 defaults,noatime 0 2' | sudo tee -a /etc/fstab
echo '/dev/vg_data/lv_data /data         xfs  defaults,noatime 0 2' | sudo tee -a /etc/fstab
```

---

## 4.3 Extending LVM (Online!)

```bash
# Extend a Logical Volume (NO downtime needed!)

# Option A: Add space from existing VG
sudo lvextend -L +10G /dev/vg_data/lv_app      # Add 10G
sudo lvextend -l +100%FREE /dev/vg_data/lv_app  # Use all free space

# Resize filesystem to match
sudo resize2fs /dev/vg_data/lv_app              # ext4 (online)
sudo xfs_growfs /data                            # XFS (online)

# Or extend + resize in one command
sudo lvextend -r -L +10G /dev/vg_data/lv_app    # -r = resize FS too

# Option B: Add a new disk to VG first
sudo pvcreate /dev/sdd
sudo vgextend vg_data /dev/sdd
sudo vgs                                         # More space!
sudo lvextend -r -L +50G /dev/vg_data/lv_app
```

```
┌──────────────────────────────────────────────────────────────┐
│         LVM EXTENSION WORKFLOW                                │
│                                                                │
│  Scenario: /data is full (30GB), need 50GB more              │
│                                                                │
│  1. Add new disk:                                             │
│     ┌──────────┐                                              │
│     │ /dev/sdd │  ← Attach new 100GB disk                    │
│     │  100 GB  │                                              │
│     └──────────┘                                              │
│                                                                │
│  2. pvcreate /dev/sdd                                         │
│  3. vgextend vg_data /dev/sdd                                 │
│  4. lvextend -r -L +50G /dev/vg_data/lv_data                 │
│                                                                │
│  Result: /data is now 80GB — NO downtime, NO data loss!      │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.4 Reducing LVM (Careful!)

```bash
# Shrinking (ext4 ONLY — XFS cannot shrink!)

# 1. Unmount first
sudo umount /opt/app

# 2. Check filesystem
sudo e2fsck -f /dev/vg_data/lv_app

# 3. Shrink filesystem first
sudo resize2fs /dev/vg_data/lv_app 15G

# 4. Then shrink LV
sudo lvreduce -L 15G /dev/vg_data/lv_app

# 5. Remount
sudo mount /dev/vg_data/lv_app /opt/app

# WARNING: Always shrink FS BEFORE LV, or you lose data!
# Safer: use lvreduce -r (auto-shrinks FS first)
sudo lvreduce -r -L 15G /dev/vg_data/lv_app
```

---

## 4.5 LVM Snapshots

```bash
# Create a snapshot (for backup or testing)
sudo lvcreate -s -n lv_app_snap -L 5G /dev/vg_data/lv_app
# -s = snapshot, -L 5G = space for changes (delta)

# Mount snapshot (read-only backup)
sudo mkdir /mnt/snapshot
sudo mount -o ro /dev/vg_data/lv_app_snap /mnt/snapshot

# Revert to snapshot (DESTRUCTIVE to current data!)
sudo umount /opt/app
sudo lvconvert --merge /dev/vg_data/lv_app_snap
sudo mount /opt/app

# Remove snapshot
sudo umount /mnt/snapshot
sudo lvremove /dev/vg_data/lv_app_snap
```

---

## 4.6 LVM Command Reference

```
┌──────────────────────────────────────────────────────────────┐
│              LVM COMMAND MATRIX                               │
├────────────┬──────────────┬──────────────┬───────────────────┤
│ Action     │ PV           │ VG           │ LV                │
├────────────┼──────────────┼──────────────┼───────────────────┤
│ Create     │ pvcreate     │ vgcreate     │ lvcreate          │
│ Display    │ pvdisplay    │ vgdisplay    │ lvdisplay         │
│ List       │ pvs          │ vgs          │ lvs               │
│ Extend     │              │ vgextend     │ lvextend          │
│ Reduce     │              │ vgreduce     │ lvreduce          │
│ Remove     │ pvremove     │ vgremove     │ lvremove          │
│ Scan       │ pvscan       │ vgscan       │ lvscan            │
│ Rename     │              │ vgrename     │ lvrename          │
│ Resize     │ pvresize     │              │ lvresize          │
└────────────┴──────────────┴──────────────┴───────────────────┘
```

---

## 4.7 Real-World Scenario: Cloud Server with LVM

```bash
#!/bin/bash
# cloud-lvm-setup.sh — Standard LVM setup for a cloud server

# /dev/sdb = 100GB EBS/Persistent Disk

echo "=== Setting up LVM on cloud server ==="

# Create PV
sudo pvcreate /dev/sdb

# Create VG
sudo vgcreate vg_app /dev/sdb

# Create LVs with smart allocation
sudo lvcreate -n lv_app    -L 40G vg_app    # Application
sudo lvcreate -n lv_data   -L 30G vg_app    # Data/uploads
sudo lvcreate -n lv_logs   -L 20G vg_app    # Logs
# Leave 10G free for snapshots and emergencies!

# Format
sudo mkfs.ext4 /dev/vg_app/lv_app
sudo mkfs.xfs /dev/vg_app/lv_data
sudo mkfs.ext4 /dev/vg_app/lv_logs

# Mount
sudo mkdir -p /opt/app /data /var/log/app
sudo mount /dev/vg_app/lv_app  /opt/app
sudo mount /dev/vg_app/lv_data /data
sudo mount /dev/vg_app/lv_logs /var/log/app

# Add to fstab
for lv in "lv_app /opt/app ext4" "lv_data /data xfs" "lv_logs /var/log/app ext4"; do
    set -- $lv
    echo "/dev/vg_app/$1  $2  $3  defaults,noatime,nofail 0 2" | sudo tee -a /etc/fstab
done

# Verify
sudo mount -a
df -hT | grep vg_app
sudo vgs
echo "=== LVM Setup Complete ==="
echo "Free space in VG for snapshots/growth:"
sudo vgs -o vg_free vg_app
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| No free space in VG | `vgs` shows 0 free | Add disk: `pvcreate` + `vgextend` |
| Can't shrink XFS | XFS doesn't support shrink | Backup, recreate smaller LV, restore |
| Snapshot full (100%) | Too many changes | Extend snapshot or remove it |
| LV path not found | DM not active | `sudo vgchange -ay vg_name` |
| Can't remove VG | LVs still exist | Remove all LVs first |
| `lvextend` but FS same size | Forgot to resize FS | `resize2fs` or `xfs_growfs` |

---

## Summary Table

| Task | Commands |
|------|----------|
| Full setup | `pvcreate` → `vgcreate` → `lvcreate` → `mkfs` → `mount` |
| Extend LV | `lvextend -r -L +10G /dev/vg/lv` |
| Add disk | `pvcreate /dev/sdX` → `vgextend vg /dev/sdX` |
| Snapshot | `lvcreate -s -n snap -L 5G /dev/vg/lv` |
| View all | `pvs`, `vgs`, `lvs` |

---

## Quick Revision Questions

1. **What are the three layers of LVM? Explain each.**
2. **How do you extend a logical volume online without downtime?**
3. **What is the difference between `lvextend` and `resize2fs`? Why do you need both?**
4. **Can you shrink an XFS file system? Why or why not?**
5. **What is an LVM snapshot and when would you use one?**
6. **Why should you leave some free space in a Volume Group?**

---

[← Previous: Mounting & fstab](03-mounting-and-fstab.md) | [Back to README](../README.md) | [Next: RAID →](05-raid.md)
