# Mount Options for Security

## Unit 4 - Topic 2 | Linux File System Security

---

## Overview

**Mount options** control how filesystems behave when accessed. Security-relevant options like **noexec**, **nosuid**, and **nodev** prevent execution of binaries, SUID exploitation, and device file abuse on specific partitions. Proper mount options are a low-effort, high-impact hardening measure.

---

## 1. Security-Relevant Mount Options

| Option | Description | Security Benefit |
|--------|-------------|-----------------|
| **noexec** | Prevent executing binaries | Blocks malware execution |
| **nosuid** | Ignore SUID/SGID bits | Prevents privilege escalation |
| **nodev** | Ignore device files | Prevents device file exploits |
| **ro** | Read-only mount | Prevents modification |
| **noatime** | Don't update access times | Minor performance + less disk writes |
| **hidepid=2** | Hide other users' processes | Privacy in /proc |

---

## 2. Recommended Mount Options

```bash
# /etc/fstab — Persistent mount configuration

# /tmp — World-writable temp directory
tmpfs  /tmp  tmpfs  defaults,noexec,nosuid,nodev,size=2G  0 0

# /var/tmp — Persistent temp directory
/dev/sda5  /var/tmp  ext4  defaults,noexec,nosuid,nodev  0 2

# /home — User home directories
/dev/sda3  /home  ext4  defaults,nosuid,nodev  0 2

# /dev/shm — Shared memory
tmpfs  /dev/shm  tmpfs  defaults,noexec,nosuid,nodev  0 0

# /var — Variable data
/dev/sda4  /var  ext4  defaults,nosuid  0 2

# /var/log — Log files
/dev/sda6  /var/log  ext4  defaults,noexec,nosuid,nodev  0 2

# /boot — Bootloader (read-only when not updating)
/dev/sda1  /boot  ext4  defaults,nosuid,nodev,noexec  0 2

# /proc — Hide process info from other users
proc  /proc  proc  defaults,hidepid=2  0 0
```

---

## 3. Applying Mount Options

```bash
# Apply changes without reboot
mount -o remount,noexec,nosuid,nodev /tmp
mount -o remount,noexec,nosuid,nodev /dev/shm

# Verify mount options
mount | grep /tmp
# Expected: tmpfs on /tmp type tmpfs (rw,nosuid,nodev,noexec)

findmnt /tmp
# Shows mount options in detail

# If /tmp is not a separate partition, use bind mount:
# 1. Create a file for the mount
dd if=/dev/zero of=/tmpfs bs=1M count=2048
mkfs.ext4 /tmpfs
# 2. Add to fstab
# /tmpfs  /tmp  ext4  loop,noexec,nosuid,nodev  0 0
mount -a
```

---

## 4. What Each Option Prevents

```
noexec:
  Attacker uploads malware.elf to /tmp
  chmod +x /tmp/malware.elf
  /tmp/malware.elf               ← BLOCKED! "Permission denied"

  BYPASS: bash /tmp/script.sh    ← Still works! (bash interprets)
  BYPASS: python3 /tmp/exploit.py ← Still works! (python interprets)
  Note: noexec blocks direct execution, not interpreted scripts

nosuid:
  Attacker places SUID binary in /tmp
  -rwsr-xr-x 1 root root /tmp/privesc
  /tmp/privesc                   ← SUID bit IGNORED! Runs as user

nodev:
  Attacker creates device file in /tmp
  mknod /tmp/evil b 8 0          ← Device file blocked!
```

---

## 5. CIS Benchmark Recommendations

| Partition | Required Options |
|-----------|-----------------|
| `/tmp` | noexec, nosuid, nodev |
| `/var/tmp` | noexec, nosuid, nodev |
| `/dev/shm` | noexec, nosuid, nodev |
| `/home` | nosuid, nodev |
| `/var/log` | noexec, nosuid, nodev |
| `/var/log/audit` | noexec, nosuid, nodev |
| `/boot` | nosuid, nodev, noexec |

```bash
# Audit current mount options
for dir in /tmp /var/tmp /dev/shm /home /var/log /boot; do
    echo "=== $dir ==="
    findmnt -n -o OPTIONS "$dir" 2>/dev/null || echo "Not a separate mount"
done
```

---

## Summary Table

| Option | Purpose | Apply To |
|--------|---------|----------|
| **noexec** | Block binary execution | /tmp, /var/tmp, /dev/shm |
| **nosuid** | Ignore SUID bits | /tmp, /home, /var |
| **nodev** | Block device files | /tmp, /home, /dev/shm |
| **ro** | Read-only | /boot (when not updating) |
| **hidepid=2** | Hide processes | /proc |

---

## Quick Revision Questions

1. **What does the `noexec` mount option prevent?**
2. **Can `noexec` stop interpreted scripts from running?**
3. **Why should `/dev/shm` have `noexec,nosuid,nodev`?**
4. **How do you apply mount options without rebooting?**
5. **What is `hidepid=2` and where is it applied?**

---

[← Previous: File System Hierarchy](01-file-system-hierarchy.md) | [Next: File Integrity Monitoring →](03-file-integrity-monitoring.md)

---

*Unit 4 - Topic 2 of 6 | [Back to README](../README.md)*
