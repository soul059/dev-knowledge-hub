# File Integrity Monitoring (FIM)

## Unit 4 - Topic 3 | Linux File System Security

---

## Overview

**File Integrity Monitoring (FIM)** detects unauthorized changes to critical system files. It works by creating a **baseline hash database** of files and periodically comparing current file states against the baseline. FIM is essential for detecting **rootkits**, **backdoors**, **configuration tampering**, and **compliance violations**.

---

## 1. How FIM Works

```
┌──────────────────────────────────────────────────────────────┐
│              FILE INTEGRITY MONITORING                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  STEP 1: INITIALIZE BASELINE                                 │
│  Scan critical files → calculate hashes → store database     │
│                                                              │
│  /etc/passwd  → sha256: a1b2c3d4e5f6...                     │
│  /usr/bin/ssh → sha256: 7a8b9c0d1e2f...                     │
│  /etc/shadow  → sha256: 3f4e5d6c7b8a...                     │
│                                                              │
│  STEP 2: PERIODIC CHECK                                      │
│  Re-scan files → compare hashes to baseline                  │
│                                                              │
│  /etc/passwd  → sha256: a1b2c3d4e5f6... ✅ No change        │
│  /usr/bin/ssh → sha256: DIFFERENT!      ❌ ALERT!            │
│  /etc/shadow  → sha256: 3f4e5d6c7b8a... ✅ No change        │
│                                                              │
│  STEP 3: ALERT                                               │
│  Modified /usr/bin/ssh → possible rootkit or backdoor!       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. What to Monitor

| Category | Files/Directories | Why |
|----------|------------------|-----|
| **System binaries** | /usr/bin/, /usr/sbin/, /bin/, /sbin/ | Detect trojanized commands |
| **Configuration** | /etc/passwd, /etc/shadow, /etc/sudoers | Detect unauthorized changes |
| **SSH** | /etc/ssh/, ~/.ssh/ | Detect key/config tampering |
| **Boot files** | /boot/, GRUB config | Detect bootkit installation |
| **Libraries** | /lib/, /usr/lib/ | Detect library injection |
| **Cron** | /etc/crontab, /var/spool/cron/ | Detect persistence |
| **Kernel modules** | /lib/modules/ | Detect rootkit modules |
| **Web files** | /var/www/ | Detect web shell/defacement |

---

## 3. Simple FIM with sha256sum

```bash
# Manual FIM (basic approach)

# Create baseline
find /etc /usr/bin /usr/sbin -type f -exec sha256sum {} \; > /root/baseline.sha256

# Check for changes
sha256sum -c /root/baseline.sha256 2>/dev/null | grep FAILED

# Output:
# /usr/bin/ssh: FAILED    ← This file was modified!

# Store baseline securely (read-only media or remote server)
```

---

## 4. FIM Detection Capabilities

| Change Type | Detection |
|-------------|----------|
| **Content modified** | Hash mismatch |
| **File added** | New file not in baseline |
| **File deleted** | File in baseline but missing |
| **Permissions changed** | Metadata comparison |
| **Ownership changed** | UID/GID comparison |
| **Timestamps changed** | mtime/ctime comparison |
| **ACL modified** | Extended attribute comparison |

---

## 5. FIM Solutions

| Tool | Type | Features |
|------|------|---------|
| **AIDE** | Open source | File integrity checker for Linux |
| **Tripwire** | Open/Commercial | Classic FIM tool |
| **OSSEC/Wazuh** | Open source | HIDS with built-in FIM |
| **Samhain** | Open source | Advanced FIM with stealth mode |
| **Auditd** | Built-in | File access monitoring (real-time) |
| **inotifywait** | Built-in | Real-time file change notification |

```bash
# Real-time monitoring with inotifywait
apt install inotify-tools
inotifywait -m -r -e modify,create,delete /etc/ /usr/bin/

# Output:
# /etc/ MODIFY passwd
# /usr/bin/ CREATE suspicious_binary
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **FIM** | Detects unauthorized file changes via hashing |
| **Baseline** | Initial known-good state of files |
| **What to monitor** | Binaries, configs, SSH, boot, cron |
| **Tools** | AIDE, Tripwire, Wazuh, Samhain |
| **Real-time** | inotifywait, auditd for instant alerts |
| **Storage** | Keep baseline on secure/remote media |

---

## Quick Revision Questions

1. **What is File Integrity Monitoring and why is it important?**
2. **How does FIM detect a trojanized binary?**
3. **What files should be included in a FIM baseline?**
4. **Name 3 FIM tools for Linux.**
5. **How can you do basic FIM with just sha256sum?**

---

[← Previous: Mount Options for Security](02-mount-options-for-security.md) | [Next: AIDE and Tripwire →](04-aide-and-tripwire.md)

---

*Unit 4 - Topic 3 of 6 | [Back to README](../README.md)*
