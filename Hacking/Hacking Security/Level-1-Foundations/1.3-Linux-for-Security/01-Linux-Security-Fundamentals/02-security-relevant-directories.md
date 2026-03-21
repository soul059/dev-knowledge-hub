# Security-Relevant Directories

## Unit 1 - Topic 2 | Linux Security Fundamentals

---

## Overview

The Linux file system follows the **Filesystem Hierarchy Standard (FHS)**. Certain directories are critical for security — they contain **configuration files**, **credentials**, **logs**, **executables**, and **sensitive data**. Knowing these directories is essential for hardening, forensics, and penetration testing.

---

## 1. Critical Security Directories

```
/
├── etc/          ← System configuration (CRITICAL for security)
├── var/          ← Variable data: logs, mail, spool
├── tmp/          ← Temporary files (world-writable!)
├── home/         ← User home directories
├── root/         ← Root user's home
├── bin/          ← Essential binaries
├── sbin/         ← System binaries (admin commands)
├── usr/          ← User programs and libraries
├── proc/         ← Virtual FS — process and kernel info
├── sys/          ← Virtual FS — hardware/kernel info
├── dev/          ← Device files
├── boot/         ← Bootloader and kernel
├── opt/          ← Optional/third-party software
├── srv/          ← Service data (web, FTP)
└── mnt/ media/   ← Mount points
```

---

## 2. /etc — Configuration Files

| File/Directory | Purpose | Security Relevance |
|---------------|---------|-------------------|
| `/etc/passwd` | User account info | Lists all users (readable by all) |
| `/etc/shadow` | Hashed passwords | Only readable by root — target for attackers |
| `/etc/group` | Group membership | Group privilege assignments |
| `/etc/sudoers` | sudo permissions | Misconfiguration = privilege escalation |
| `/etc/ssh/sshd_config` | SSH server config | Hardening target |
| `/etc/pam.d/` | PAM modules | Authentication configuration |
| `/etc/crontab` | System cron jobs | Persistence mechanism |
| `/etc/hosts` | Static hostname mapping | DNS poisoning locally |
| `/etc/resolv.conf` | DNS resolver config | DNS configuration |
| `/etc/fstab` | Filesystem mount table | Mount option security |
| `/etc/login.defs` | Login defaults | Password policies |
| `/etc/securetty` | Root login terminals | Restrict root login locations |
| `/etc/sysctl.conf` | Kernel parameters | Kernel hardening |

```bash
# Check password file for anomalies
cat /etc/passwd | grep ':0:'        # Find UID 0 accounts (should only be root)
cat /etc/passwd | grep '/bin/bash'  # Users with login shells

# Check shadow permissions
ls -la /etc/shadow                   # Should be -rw-r----- root:shadow

# Check sudoers
cat /etc/sudoers                     # Or visudo for safe editing
sudo -l                              # List current user's sudo privileges
```

---

## 3. /var — Logs and Variable Data

| Path | Content | Security Use |
|------|---------|-------------|
| `/var/log/` | System and application logs | Forensics, audit trail |
| `/var/log/auth.log` | Authentication events | Login attempts, sudo usage |
| `/var/log/syslog` | General system log | System events |
| `/var/log/kern.log` | Kernel messages | Kernel exploits, hardware issues |
| `/var/log/apache2/` | Web server logs | Web attack detection |
| `/var/log/secure` | Auth log (RHEL/CentOS) | Same as auth.log |
| `/var/spool/cron/` | User crontabs | Scheduled persistence |
| `/var/mail/` | User mail | May contain sensitive info |
| `/var/tmp/` | Persistent temp files | Survives reboot (unlike /tmp) |

```bash
# Check recent failed logins
grep "Failed password" /var/log/auth.log | tail -20

# Check for unauthorized cron jobs
ls -la /var/spool/cron/crontabs/

# Monitor logs in real-time
tail -f /var/log/auth.log
```

---

## 4. /tmp and /dev/shm — World-Writable Directories

```
⚠️ SECURITY CONCERN: World-writable directories

/tmp          — Temporary files (cleared on reboot)
/var/tmp      — Temporary files (persistent across reboot)
/dev/shm      — Shared memory (tmpfs — in RAM)

RISKS:
• Attackers store malware, exploits, and tools here
• Symlink attacks (race conditions)
• Privilege escalation via SUID binaries in /tmp

HARDENING:
• Mount /tmp with noexec,nosuid,nodev
• Mount /dev/shm with noexec,nosuid,nodev
• Use tmpfs for /tmp
• Enable sticky bit (default on most systems)
```

```bash
# Check /tmp permissions
ls -ld /tmp                          # Should show drwxrwxrwt (sticky bit)

# Check mount options
mount | grep tmp

# Find suspicious files in /tmp
find /tmp -type f -perm -4000        # SUID files in /tmp (suspicious!)
find /tmp -type f -name "*.sh"       # Scripts in /tmp
find /tmp -newer /etc/passwd         # Recently modified files
```

---

## 5. /proc and /sys — Virtual Filesystems

| Path | Content | Security Use |
|------|---------|-------------|
| `/proc/[PID]/` | Per-process info | Process forensics |
| `/proc/[PID]/cmdline` | Command line arguments | See what a process is doing |
| `/proc/[PID]/environ` | Environment variables | May contain secrets! |
| `/proc/[PID]/fd/` | Open file descriptors | See open files |
| `/proc/[PID]/maps` | Memory mappings | Memory forensics |
| `/proc/net/tcp` | Active TCP connections | Network analysis |
| `/proc/sys/` | Kernel tunable parameters | Kernel hardening |

```bash
# View process info
cat /proc/1/cmdline | tr '\0' ' '    # PID 1 command line
cat /proc/self/environ | tr '\0' '\n' # Current process env vars
ls -la /proc/self/fd/                 # Open file descriptors

# View network connections via /proc
cat /proc/net/tcp                     # Active TCP connections
```

---

## Summary Table

| Directory | Security Relevance |
|-----------|-------------------|
| `/etc/shadow` | Password hashes — protect at all costs |
| `/etc/sudoers` | Privilege escalation vector |
| `/var/log/` | Forensic evidence — attackers try to delete |
| `/tmp` | World-writable — mount with noexec |
| `/proc/` | Process and kernel information disclosure |
| `/etc/crontab` | Persistence mechanism for attackers |

---

## Quick Revision Questions

1. **Why is `/etc/shadow` more security-sensitive than `/etc/passwd`?**
2. **What security risk does `/tmp` pose and how is it mitigated?**
3. **What information can be found in `/proc/[PID]/environ`?**
4. **Which log file shows authentication events on Debian/Ubuntu?**
5. **How would you find all UID 0 accounts on a system?**

---

[← Previous: Linux Architecture Overview](01-linux-architecture-overview.md) | [Next: File Permissions →](03-file-permissions.md)

---

*Unit 1 - Topic 2 of 5 | [Back to README](../README.md)*
