# Chapter 4: Linux Boot Process

[← Previous: File System Hierarchy](03-file-system-hierarchy.md) | [Back to README](../README.md) | [Next: Package Managers →](05-package-managers.md)

---

## Overview

Understanding the Linux boot process is essential for DevOps engineers who need to troubleshoot boot failures, configure services at startup, and optimize boot times. The boot process follows a well-defined sequence from hardware power-on to a fully running system.

---

## 4.1 Boot Process Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                    LINUX BOOT PROCESS                            │
│                                                                   │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│  │  BIOS/   │───→│  Boot    │───→│  Kernel  │───→│  init/   │   │
│  │  UEFI    │    │  Loader  │    │  Loading │    │  systemd │   │
│  │          │    │  (GRUB2) │    │          │    │  (PID 1) │   │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘   │
│       │               │               │               │         │
│       ▼               ▼               ▼               ▼         │
│  Power-On         Load kernel     Decompress      Start         │
│  Self Test        from disk       & init          services      │
│  (POST)           (/boot)         hardware        (targets)     │
│                                                                   │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│  │ Hardware │───→│  MBR/    │───→│ initramfs│───→│  Login   │   │
│  │  Init    │    │  GPT     │    │ (initrd) │    │  Prompt  │   │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4.2 Stage-by-Stage Breakdown

### Stage 1: BIOS/UEFI

```
┌───────────────────────────────────────────────┐
│              BIOS vs UEFI                      │
├───────────────┬───────────────────────────────┤
│    BIOS       │        UEFI                   │
│  (Legacy)     │    (Modern)                   │
├───────────────┼───────────────────────────────┤
│ 16-bit mode   │ 32/64-bit mode               │
│ MBR partition │ GPT partition table           │
│ 2TB disk max  │ 9.4 ZB disk max              │
│ 4 primary     │ 128 partitions               │
│ partitions    │                               │
│ No secure boot│ Secure Boot support           │
│ ROM-based     │ Stored in flash memory        │
│ Slow POST     │ Faster boot                   │
└───────────────┴───────────────────────────────┘
```

**What happens:**
1. Hardware powered on → firmware (BIOS/UEFI) executes
2. POST (Power-On Self Test) — checks CPU, RAM, peripherals
3. Firmware locates and loads the boot loader from:
   - **BIOS:** First 512 bytes of boot disk (MBR)
   - **UEFI:** EFI System Partition (`/boot/efi`)

```bash
# Check if system uses BIOS or UEFI
[ -d /sys/firmware/efi ] && echo "UEFI" || echo "BIOS"

# View UEFI boot entries
efibootmgr -v

# Check partition table type
sudo fdisk -l /dev/sda | grep "Disklabel type"
```

### Stage 2: Boot Loader (GRUB2)

```
┌─────────────────────────────────────────────────────────┐
│                    GRUB2 BOOT LOADER                     │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │     GRUB Menu (if multiple kernels/OS)           │   │
│  │  ┌────────────────────────────────────────────┐  │   │
│  │  │ * Ubuntu, with Linux 5.15.0-76-generic     │  │   │
│  │  │   Ubuntu, with Linux 5.15.0-72-generic     │  │   │
│  │  │   Advanced options for Ubuntu               │  │   │
│  │  │   Memory test (memtest86+)                  │  │   │
│  │  └────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  Config: /etc/default/grub                               │
│  Generated: /boot/grub/grub.cfg                          │
│  Update: sudo update-grub  (or grub2-mkconfig)          │
└─────────────────────────────────────────────────────────┘
```

**What happens:**
1. GRUB2 loads from MBR/EFI partition
2. Displays boot menu (if configured)
3. Loads the selected kernel image and initramfs into RAM
4. Passes control to the kernel with boot parameters

```bash
# GRUB configuration
cat /etc/default/grub

# Key GRUB parameters:
# GRUB_TIMEOUT=5              # Menu timeout in seconds
# GRUB_CMDLINE_LINUX=""       # Kernel boot parameters
# GRUB_DEFAULT=0              # Default menu entry

# Update GRUB after changes
# Ubuntu/Debian:
sudo update-grub

# RHEL/CentOS:
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# View installed kernels
ls /boot/vmlinuz-*

# Check current boot parameters
cat /proc/cmdline
```

### Stage 3: Kernel Initialization

```
┌─────────────────────────────────────────────────────────┐
│                KERNEL INITIALIZATION                     │
│                                                          │
│  1. Decompress kernel image (vmlinuz → vmlinux)         │
│           │                                              │
│  2. Initialize hardware (CPU, memory, interrupts)       │
│           │                                              │
│  3. Mount initramfs as temporary root filesystem        │
│           │                                              │
│  4. Load essential drivers from initramfs               │
│     (disk controllers, filesystem drivers)              │
│           │                                              │
│  5. Mount real root filesystem (from /etc/fstab)        │
│           │                                              │
│  6. Execute /sbin/init (or systemd) as PID 1            │
│           │                                              │
│  7. initramfs is unmounted and freed                    │
└─────────────────────────────────────────────────────────┘
```

```bash
# View kernel boot messages
dmesg | head -50
dmesg | grep -i "command line"
dmesg | grep -i "mount"

# View initramfs contents
lsinitramfs /boot/initrd.img-$(uname -r)          # Ubuntu
lsinitrd /boot/initramfs-$(uname -r).img           # RHEL

# Rebuild initramfs
# Ubuntu:
sudo update-initramfs -u

# RHEL/CentOS:
sudo dracut --force
```

### Stage 4: systemd (PID 1)

```
┌────────────────────────────────────────────────────────────────┐
│                    SYSTEMD TARGETS                              │
│                                                                 │
│  poweroff.target ←── 0  (Halt)                                 │
│                                                                 │
│  rescue.target   ←── 1  (Single user / Recovery)               │
│       │                                                         │
│  multi-user.target ←── 3  (Full multi-user, no GUI)            │
│       │                  ★ Most common for servers              │
│       │                                                         │
│  graphical.target  ←── 5  (Multi-user + GUI)                   │
│                         Desktop systems                         │
│                                                                 │
│  reboot.target   ←── 6  (Reboot)                               │
│                                                                 │
│  emergency.target ──→ Minimal boot for troubleshooting         │
└────────────────────────────────────────────────────────────────┘
```

**What happens:**
1. systemd starts as PID 1
2. Reads default target (e.g., `multi-user.target`)
3. Resolves dependency tree
4. Starts all services in parallel where possible
5. Reaches the target state → system is ready

```bash
# Check default target (runlevel)
systemctl get-default

# Set default target
sudo systemctl set-default multi-user.target   # Server (no GUI)
sudo systemctl set-default graphical.target    # Desktop (with GUI)

# List target dependencies
systemctl list-dependencies multi-user.target

# Check boot time
systemd-analyze
# Startup finished in 2.5s (kernel) + 5.3s (userspace) = 7.8s

# Blame: which service took the longest
systemd-analyze blame | head -10

# Boot timeline as SVG
systemd-analyze plot > boot-timeline.svg
```

---

## 4.3 Complete Boot Sequence Diagram

```
Power On
   │
   ▼
┌──────────────┐
│ BIOS / UEFI  │  POST, hardware init
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   MBR / GPT  │  Find boot partition
└──────┬───────┘
       │
       ▼
┌──────────────┐
│    GRUB2     │  Show menu, load kernel + initramfs
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Kernel     │  Hardware init, mount initramfs
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  initramfs   │  Load disk/FS drivers, find root FS
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Mount root   │  Switch from initramfs to real /
│ filesystem   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   systemd    │  PID 1, start services
│  (init)      │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  default     │  Start all service units for
│  target      │  the configured target
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Login Prompt │  getty / sshd / display manager
└──────────────┘
```

---

## 4.4 GRUB2 Recovery — Real-World Scenario

### Scenario: System Won't Boot After Kernel Update

```
┌────────────────────────────────────────────────────────┐
│  GRUB RESCUE STEPS:                                    │
│                                                        │
│  1. At GRUB menu, press 'e' to edit boot entry        │
│                                                        │
│  2. Find the line starting with 'linux'               │
│     linux /boot/vmlinuz-5.15.0-76-generic root=...    │
│                                                        │
│  3. Add 'single' or 'init=/bin/bash' at end of line   │
│     linux /boot/vmlinuz-... root=... single           │
│                                                        │
│  4. Press Ctrl-X or F10 to boot                       │
│                                                        │
│  5. Once in single user mode:                         │
│     mount -o remount,rw /                             │
│     # Fix the issue                                   │
│     # Roll back kernel, fix fstab, etc.               │
│     reboot                                            │
└────────────────────────────────────────────────────────┘
```

```bash
# Boot to previous kernel from GRUB menu
# Select "Advanced options" → choose older kernel

# After booting, remove problematic kernel
# Ubuntu:
sudo apt remove linux-image-5.15.0-76-generic
sudo update-grub

# RHEL/CentOS:
sudo rpm -e kernel-5.14.0-284.el9
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## 4.5 Boot Performance Analysis

```bash
# Total boot time
systemd-analyze
# Startup finished in 1.8s (firmware) + 2.5s (loader) + 
#                    3.2s (kernel) + 8.1s (userspace) = 15.6s

# Top 10 slowest services
systemd-analyze blame | head -10
# 5.2s    NetworkManager-wait-online.service
# 2.1s    docker.service
# 1.8s    snapd.service
# 1.2s    apt-daily.service

# Critical chain (bottleneck path)
systemd-analyze critical-chain
# graphical.target @8.1s
# └─multi-user.target @8.1s
#   └─docker.service @3.2s +2.1s
#     └─network-online.target @3.1s

# Disable unnecessary services to speed up boot
sudo systemctl disable snapd.service
sudo systemctl mask NetworkManager-wait-online.service
```

---

## Troubleshooting Tips

| Symptom | Stage | Solution |
|---------|-------|---------|
| No display after power on | BIOS/UEFI | Check hardware, BIOS settings, cable |
| "GRUB rescue>" prompt | GRUB | `set root=(hd0,1)`, reconfigure GRUB |
| Kernel panic | Kernel | Boot old kernel, check initramfs |
| Stuck at "Starting services" | systemd | Boot to `rescue.target`, check logs |
| Very slow boot | systemd | `systemd-analyze blame`, disable slow services |
| "Emergency mode" | fstab | Fix `/etc/fstab` entries, run `fsck` |
| Can't find root filesystem | initramfs | Rebuild initramfs, check UUID in fstab |

---

## Summary Table

| Boot Stage | Component | Location | Config File |
|-----------|-----------|----------|-------------|
| 1. Firmware | BIOS/UEFI | ROM/Flash | BIOS Setup (F2/Del) |
| 2. Boot Loader | GRUB2 | MBR or `/boot/efi` | `/etc/default/grub` |
| 3. Kernel | vmlinuz | `/boot/vmlinuz-*` | `/proc/cmdline` |
| 4. Initial RAM disk | initramfs | `/boot/initrd*` | `update-initramfs` / `dracut` |
| 5. Init System | systemd | `/lib/systemd/systemd` | `/etc/systemd/system/` |
| 6. Target | multi-user | systemd units | `systemctl get-default` |

---

## Quick Revision Questions

1. **List the 6 stages of the Linux boot process in order.** What happens at each stage?
2. **What is the difference between BIOS and UEFI?** Which partition table does each use?
3. **What is initramfs and why is it needed?** When would you rebuild it?
4. **How would you boot into single-user mode for recovery?** Describe the GRUB steps.
5. **What command shows which services are slowing down boot?** How can you fix slow boot times?
6. **What is PID 1 in a modern Linux system?** What role does it play?

---

[← Previous: File System Hierarchy](03-file-system-hierarchy.md) | [Back to README](../README.md) | [Next: Package Managers →](05-package-managers.md)
