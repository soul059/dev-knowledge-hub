# Chapter 5: RAID

[← Previous: LVM (Logical Volume Manager)](04-lvm.md) | [Back to README](../README.md) | [Next: Disk Monitoring & Troubleshooting →](06-disk-monitoring.md)

---

## Overview

RAID (Redundant Array of Independent Disks) combines multiple disks for redundancy, performance, or both. While cloud environments abstract hardware RAID, understanding RAID levels and Linux software RAID (`mdadm`) is important for on-premises servers and interviews.

---

## 5.1 RAID Levels

```
┌──────────────────────────────────────────────────────────────┐
│                    RAID LEVEL COMPARISON                       │
│                                                                │
│  RAID 0 (Striping) — Performance, NO redundancy              │
│  ┌──────┐ ┌──────┐                                           │
│  │Disk 1│ │Disk 2│   Data split across disks                 │
│  │ A1   │ │ A2   │   Speed: 2x read/write                   │
│  │ A3   │ │ A4   │   Risk: 1 disk fails = ALL data lost     │
│  └──────┘ └──────┘   Min disks: 2  Capacity: N × disk       │
│                                                                │
│  RAID 1 (Mirroring) — Redundancy, moderate performance       │
│  ┌──────┐ ┌──────┐                                           │
│  │Disk 1│ │Disk 2│   Data duplicated on both disks           │
│  │  A1  │ │  A1  │   Read: 2x  Write: 1x                    │
│  │  A2  │ │  A2  │   1 disk can fail safely                  │
│  └──────┘ └──────┘   Min disks: 2  Capacity: 1 × disk       │
│                                                                │
│  RAID 5 (Striping + Distributed Parity)                      │
│  ┌──────┐ ┌──────┐ ┌──────┐                                 │
│  │Disk 1│ │Disk 2│ │Disk 3│   Parity distributed            │
│  │  A1  │ │  A2  │ │  Ap  │   1 disk can fail safely        │
│  │  B1  │ │  Bp  │ │  B2  │   Good balance of space/safety  │
│  │  Cp  │ │  C1  │ │  C2  │                                 │
│  └──────┘ └──────┘ └──────┘   Min disks: 3                  │
│                       Capacity: (N-1) × disk                  │
│                                                                │
│  RAID 6 (Double Parity)                                       │
│  Like RAID 5 but 2 parity blocks                              │
│  2 disks can fail safely                                      │
│  Min disks: 4   Capacity: (N-2) × disk                       │
│                                                                │
│  RAID 10 (1+0, Mirrored Stripes)                             │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐                        │
│  │  D1  │ │  D2  │ │  D3  │ │  D4  │                        │
│  │  A1  │ │  A1  │ │  A2  │ │  A2  │                        │
│  │  A3  │ │  A3  │ │  A4  │ │  A4  │                        │
│  └──────┘ └──────┘ └──────┘ └──────┘                         │
│  Mirror pair 1      Mirror pair 2                             │
│  Best performance + redundancy                                │
│  Min disks: 4   Capacity: N/2 × disk                         │
└──────────────────────────────────────────────────────────────┘
```

```
┌──────────────────────────────────────────────────────────────┐
│             RAID SELECTION GUIDE                              │
├──────────┬───────┬───────┬───────────┬───────────────────────┤
│ Level    │ Disks │ Fault │ Capacity  │ Best For              │
├──────────┼───────┼───────┼───────────┼───────────────────────┤
│ RAID 0   │ 2+    │ None  │ N × disk  │ Temp/cache, speed     │
│ RAID 1   │ 2     │ 1     │ 1 × disk  │ OS drives, boot       │
│ RAID 5   │ 3+    │ 1     │ (N-1)×disk│ General storage       │
│ RAID 6   │ 4+    │ 2     │ (N-2)×disk│ Large arrays          │
│ RAID 10  │ 4+    │ 1/pair│ N/2 ×disk │ Databases, high I/O   │
└──────────┴───────┴───────┴───────────┴───────────────────────┘
```

---

## 5.2 Software RAID with `mdadm`

```bash
# Install mdadm
sudo apt install mdadm       # Ubuntu
sudo dnf install mdadm       # RHEL

# Create RAID 1 (mirror)
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Create RAID 5
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# Check status
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# Save configuration
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u     # Ubuntu

# Use the array
sudo mkfs.ext4 /dev/md0
sudo mkdir /data
sudo mount /dev/md0 /data
# Add to /etc/fstab with UUID
```

---

## 5.3 Managing RAID Arrays

```bash
# Check array health
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# Replace a failed disk
# 1. Mark as failed (or it fails automatically)
sudo mdadm /dev/md0 --fail /dev/sdc

# 2. Remove failed disk
sudo mdadm /dev/md0 --remove /dev/sdc

# 3. Add replacement
sudo mdadm /dev/md0 --add /dev/sde

# 4. Watch rebuild
watch cat /proc/mdstat

# Add a spare disk
sudo mdadm /dev/md0 --add /dev/sdf    # Becomes hot spare

# Stop array
sudo mdadm --stop /dev/md0

# Assemble existing array
sudo mdadm --assemble /dev/md0 /dev/sdb /dev/sdc
```

---

## 5.4 Real-World Scenario: RAID 1 for OS Redundancy

```bash
#!/bin/bash
# setup-raid1.sh — Mirror two disks for OS/boot redundancy

echo "=== Creating RAID 1 Mirror ==="

# Create RAID 1 from sdb and sdc
sudo mdadm --create /dev/md0 \
    --level=1 \
    --raid-devices=2 \
    /dev/sdb /dev/sdc

# Wait for initial sync
echo "Waiting for sync..."
while grep -q "resync" /proc/mdstat; do
    cat /proc/mdstat | grep "resync"
    sleep 10
done

# Format and mount
sudo mkfs.ext4 -L "raid-data" /dev/md0
sudo mkdir -p /data
sudo mount /dev/md0 /data

# Save config
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u

# Add to fstab
UUID=$(blkid -s UUID -o value /dev/md0)
echo "UUID=$UUID /data ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab

echo "=== RAID 1 Ready ==="
sudo mdadm --detail /dev/md0
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| RAID degraded | `cat /proc/mdstat` shows [U_] | Replace failed disk, add to array |
| Rebuild slow | Heavy I/O during rebuild | `echo 50000 > /proc/sys/dev/raid/speed_limit_max` |
| Array won't assemble | Config missing | `mdadm --assemble --scan` or specify disks |
| RAID 5 with 2 failed disks | Data lost | RAID 5 only tolerates 1 failure; use RAID 6 |
| Wrong RAID level chosen | Performance/redundancy mismatch | Plan capacity and fault tolerance before setup |

---

## Summary Table

| Command | Purpose | Example |
|---------|---------|---------|
| `mdadm --create` | Create array | `mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sd{b,c}` |
| `mdadm --detail` | Array details | `mdadm --detail /dev/md0` |
| `cat /proc/mdstat` | Quick status | Shows sync progress |
| `mdadm --fail` | Mark disk failed | `mdadm /dev/md0 --fail /dev/sdc` |
| `mdadm --remove` | Remove disk | `mdadm /dev/md0 --remove /dev/sdc` |
| `mdadm --add` | Add disk | `mdadm /dev/md0 --add /dev/sde` |

---

## Quick Revision Questions

1. **What is the difference between RAID 0, RAID 1, and RAID 5?**
2. **How many disks can fail in RAID 5? In RAID 6? In RAID 10?**
3. **Which RAID level gives the best performance for databases?**
4. **How do you replace a failed disk in a Linux software RAID array?**
5. **What is a hot spare in RAID?**
6. **Calculate the usable capacity of a RAID 5 array with 4 × 1TB disks.**

---

[← Previous: LVM (Logical Volume Manager)](04-lvm.md) | [Back to README](../README.md) | [Next: Disk Monitoring & Troubleshooting →](06-disk-monitoring.md)
