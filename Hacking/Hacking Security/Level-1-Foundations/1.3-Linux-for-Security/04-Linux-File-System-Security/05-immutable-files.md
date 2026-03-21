# Immutable Files

## Unit 4 - Topic 5 | Linux File System Security

---

## Overview

Linux **file attributes** provide an extra layer of protection beyond standard permissions. The most security-relevant attribute is the **immutable flag** (`+i`) — which prevents a file from being modified, deleted, or renamed, even by root. This is valuable for protecting critical configuration files from tampering.

---

## 1. File Attributes with chattr/lsattr

```bash
# Set immutable attribute (only root can set/remove)
chattr +i /etc/passwd
chattr +i /etc/shadow
chattr +i /etc/sudoers

# Remove immutable attribute
chattr -i /etc/passwd

# View attributes
lsattr /etc/passwd
# ----i---------e--- /etc/passwd
# The 'i' flag means immutable

# Set append-only (can only add data, not modify/delete)
chattr +a /var/log/auth.log

# Remove append-only
chattr -a /var/log/auth.log

# View all attributes on directory
lsattr /etc/
```

---

## 2. Attribute Reference

| Attribute | Flag | Description | Security Use |
|-----------|:----:|-------------|-------------|
| **Immutable** | `i` | Cannot be modified, deleted, renamed, or linked | Protect configs |
| **Append-only** | `a` | Can only be appended to (not modified/deleted) | Protect logs |
| **No dump** | `d` | Excluded from dump backups | N/A |
| **Secure deletion** | `s` | Securely zero blocks on deletion | Data destruction |
| **No atime update** | `A` | Don't update access time | Performance |
| **Synchronous** | `S` | Write changes immediately to disk | Data integrity |

---

## 3. Immutable File Behavior

```bash
# With immutable flag set on /etc/passwd:

echo "hacker:x:0:0::/root:/bin/bash" >> /etc/passwd
# Operation not permitted

rm /etc/passwd
# Operation not permitted

mv /etc/passwd /etc/passwd.bak
# Operation not permitted

chmod 777 /etc/passwd
# Operation not permitted

chown nobody /etc/passwd
# Operation not permitted

# Even root cannot modify until flag is removed!
# Must: chattr -i /etc/passwd first
```

---

## 4. Security Use Cases

| Use Case | Command | Purpose |
|----------|---------|---------|
| Protect passwd/shadow | `chattr +i /etc/passwd /etc/shadow` | Prevent unauthorized user creation |
| Protect sudoers | `chattr +i /etc/sudoers` | Prevent privilege escalation |
| Protect SSH config | `chattr +i /etc/ssh/sshd_config` | Prevent SSH reconfiguration |
| Protect logs | `chattr +a /var/log/auth.log` | Prevent log deletion |
| Protect resolv.conf | `chattr +i /etc/resolv.conf` | Prevent DNS hijacking |
| Protect hosts file | `chattr +i /etc/hosts` | Prevent local DNS poisoning |

---

## 5. Limitations

| Limitation | Detail |
|-----------|--------|
| **Root can remove** | Root can `chattr -i` and then modify |
| **Requires ext filesystem** | Only works on ext2/ext3/ext4, XFS (partial) |
| **Breaks updates** | Must remove `+i` before system updates |
| **Not MAC** | This is DAC-level protection — rootkits with root bypass it |
| **Kernel bypass** | Kernel-level rootkits can bypass file attributes |

```bash
# Automated: Remove immutable before updates, restore after
#!/bin/bash
# pre-update.sh
chattr -i /etc/passwd /etc/shadow /etc/sudoers
apt update && apt upgrade -y
chattr +i /etc/passwd /etc/shadow /etc/sudoers
```

---

## 6. Audit Immutable Files

```bash
# Find all immutable files
lsattr -R / 2>/dev/null | grep "\-i\-"

# Find all append-only files
lsattr -R / 2>/dev/null | grep "\-a\-"

# Monitor attribute changes with auditd
auditctl -w /etc/passwd -p a -k passwd_attr
# Alerts when attributes are changed
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Immutable (`+i`)** | Prevents ALL changes, even by root |
| **Append-only (`+a`)** | Only allows appending data |
| **chattr** | Set file attributes |
| **lsattr** | View file attributes |
| **Use cases** | Protect /etc/passwd, /etc/shadow, logs |
| **Limitation** | Root can remove flag; breaks updates |

---

## Quick Revision Questions

1. **What does the immutable flag (`+i`) prevent?**
2. **How do you set and remove the immutable attribute?**
3. **What is the difference between immutable and append-only?**
4. **Why would you make /etc/resolv.conf immutable?**
5. **What is the main limitation of file attributes?**

---

[← Previous: AIDE and Tripwire](04-aide-and-tripwire.md) | [Next: Disk Encryption (LUKS) →](06-disk-encryption-luks.md)

---

*Unit 4 - Topic 5 of 6 | [Back to README](../README.md)*
