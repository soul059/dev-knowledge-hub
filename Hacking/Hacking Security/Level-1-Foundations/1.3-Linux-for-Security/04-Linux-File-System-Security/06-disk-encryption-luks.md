# Disk Encryption (LUKS)

## Unit 4 - Topic 6 | Linux File System Security

---

## Overview

**LUKS (Linux Unified Key Setup)** is the standard for Linux disk encryption. It provides **full disk encryption (FDE)** or **partition-level encryption**, protecting data at rest from unauthorized access. If a server or laptop is stolen, LUKS ensures the data remains encrypted and inaccessible without the correct passphrase or key.

---

## 1. Why Encrypt Disks?

| Threat | Without Encryption | With Encryption |
|--------|:-:|:-:|
| **Stolen laptop** | All data exposed | Data encrypted, inaccessible |
| **Decommissioned drive** | Data recoverable | Data encrypted |
| **Physical access** | Boot from USB, read disk | Passphrase required |
| **Forensic recovery** | Deleted files recoverable | Encrypted blocks unreadable |
| **Compliance** | PCI DSS / HIPAA violation | ✅ Compliant |

---

## 2. LUKS Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              LUKS ENCRYPTION LAYERS                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────┐                      │
│  │ Application Data                  │                      │
│  ├────────────────────────────────────┤                      │
│  │ File System (ext4, XFS)           │                      │
│  ├────────────────────────────────────┤                      │
│  │ Device Mapper (dm-crypt)          │ ← Encryption layer   │
│  ├────────────────────────────────────┤                      │
│  │ LUKS Header + Encrypted Partition │                      │
│  ├────────────────────────────────────┤                      │
│  │ Physical Disk                     │                      │
│  └────────────────────────────────────┘                      │
│                                                              │
│  LUKS Header contains:                                       │
│  • Cipher info (aes-xts-plain64)                            │
│  • Key slots (up to 8 different passphrases)                │
│  • Master key (encrypted with passphrase)                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Creating Encrypted Partitions

```bash
# INSTALL cryptsetup
apt install cryptsetup

# STEP 1: Create LUKS partition
cryptsetup luksFormat /dev/sdb1
# Prompts for passphrase — THIS IS YOUR ENCRYPTION KEY

# STEP 2: Open (unlock) the encrypted partition
cryptsetup luksOpen /dev/sdb1 secure_data
# Creates /dev/mapper/secure_data

# STEP 3: Create filesystem
mkfs.ext4 /dev/mapper/secure_data

# STEP 4: Mount
mkdir /mnt/secure
mount /dev/mapper/secure_data /mnt/secure

# STEP 5: Use it
cp sensitive_files /mnt/secure/

# STEP 6: Unmount and close
umount /mnt/secure
cryptsetup luksClose secure_data
# Data is now inaccessible without passphrase
```

---

## 4. Key Management

```bash
# View LUKS header info
cryptsetup luksDump /dev/sdb1

# Add additional passphrase (key slot)
cryptsetup luksAddKey /dev/sdb1
# Now two passphrases can unlock the volume

# Remove a key slot
cryptsetup luksRemoveKey /dev/sdb1

# Use a key file instead of passphrase
dd if=/dev/urandom of=/root/luks-key bs=4096 count=1
chmod 600 /root/luks-key
cryptsetup luksAddKey /dev/sdb1 /root/luks-key

# Open with key file
cryptsetup luksOpen /dev/sdb1 secure_data --key-file /root/luks-key

# Auto-mount at boot with key file
# /etc/crypttab:
# secure_data  /dev/sdb1  /root/luks-key  luks
# /etc/fstab:
# /dev/mapper/secure_data  /mnt/secure  ext4  defaults  0 2
```

---

## 5. Full Disk Encryption

```bash
# During OS installation:
# Most installers offer "Encrypt the entire disk" option
# Uses LUKS + LVM for flexibility

# Typical FDE layout:
# /dev/sda1 — /boot (unencrypted — needed to boot)
# /dev/sda2 — LUKS encrypted container
#   └── LVM inside LUKS
#       ├── /dev/vg0/root  → /
#       ├── /dev/vg0/swap  → swap (encrypted!)
#       └── /dev/vg0/home  → /home

# Boot process with FDE:
# 1. GRUB loads from /boot (unencrypted)
# 2. Kernel loads and prompts for LUKS passphrase
# 3. LUKS decrypts, LVM activated
# 4. Root filesystem mounted, boot continues
```

---

## 6. Swap Encryption

```bash
# Swap may contain sensitive data from RAM!
# Encrypt swap to prevent data leakage

# Random key (different each boot — no suspend/hibernate)
# /etc/crypttab:
# cryptswap  /dev/sda3  /dev/urandom  swap,cipher=aes-xts-plain64,size=256

# Then in /etc/fstab:
# /dev/mapper/cryptswap  none  swap  sw  0  0

# Apply
cryptdisks_start cryptswap
swapon -a
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **LUKS** | Linux standard disk encryption |
| **dm-crypt** | Kernel-level encryption module |
| **cryptsetup** | User-space tool for LUKS management |
| **Key slots** | Up to 8 passphrases per volume |
| **FDE** | Full disk encryption during installation |
| **Swap** | Must encrypt swap to protect RAM data |

---

## Quick Revision Questions

1. **What does LUKS stand for and what does it do?**
2. **How do you create and mount an encrypted partition?**
3. **Why should swap be encrypted?**
4. **How many key slots does LUKS support?**
5. **Why is /boot typically left unencrypted with FDE?**

---

[← Previous: Immutable Files](05-immutable-files.md)

---

*Unit 4 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Linux Access Control →](../05-Linux-Access-Control/01-discretionary-access-control.md)*
