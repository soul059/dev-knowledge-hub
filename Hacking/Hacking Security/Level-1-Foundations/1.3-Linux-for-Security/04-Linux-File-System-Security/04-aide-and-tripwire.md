# AIDE and Tripwire

## Unit 4 - Topic 4 | Linux File System Security

---

## Overview

**AIDE (Advanced Intrusion Detection Environment)** and **Tripwire** are dedicated File Integrity Monitoring (FIM) tools that create cryptographic baselines and detect unauthorized changes. Both are widely used in enterprise environments for **compliance**, **rootkit detection**, and **change management**.

---

## 1. AIDE — Advanced Intrusion Detection Environment

```bash
# INSTALL
apt install aide                      # Debian/Ubuntu
yum install aide                      # RHEL/CentOS

# CONFIGURATION: /etc/aide/aide.conf (Debian) or /etc/aide.conf

# Define what to check
# Attributes: p=permissions, i=inode, n=links, u=user, g=group,
#             s=size, b=blocks, m=mtime, c=ctime, S=check for size,
#             md5=MD5, sha256=SHA-256, sha512=SHA-512

# Example rules:
NORMAL = p+i+n+u+g+s+m+c+sha256
DIR = p+i+n+u+g
LOG = p+i+n+u+g+S

# Directories to monitor
/etc     NORMAL
/usr/bin NORMAL
/usr/sbin NORMAL
/boot    NORMAL
/lib     NORMAL

# Directories to exclude
!/var/log
!/tmp
!/var/tmp
!/proc
!/sys
!/run

# INITIALIZE DATABASE (create baseline)
aide --init
# or
aideinit                              # Debian wrapper

# Move new database to become the baseline
cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# CHECK FOR CHANGES
aide --check

# OUTPUT EXAMPLE:
# Changed entries:
# ---------------------------------------------------
# f ... .C... : /etc/passwd
# f >ps....C. : /usr/bin/ssh
# 
# f = file changed
# p = permissions, s = size, C = checksum
```

---

## 2. AIDE Automation

```bash
# CRON JOB — Daily integrity check
# /etc/cron.daily/aide-check
#!/bin/bash
LOGFILE="/var/log/aide/aide-$(date +%Y%m%d).log"
aide --check > "$LOGFILE" 2>&1
if [ $? -ne 0 ]; then
    mail -s "AIDE Alert: File Changes Detected on $(hostname)" admin@example.com < "$LOGFILE"
fi

chmod +x /etc/cron.daily/aide-check

# UPDATE DATABASE after legitimate changes
aide --update
cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

---

## 3. Tripwire

```bash
# INSTALL
apt install tripwire                  # Interactive setup

# During installation:
# 1. Set site key passphrase (protects policy)
# 2. Set local key passphrase (protects database)

# CONFIGURATION
# Policy file: /etc/tripwire/twpol.txt
# Config file: /etc/tripwire/twcfg.txt

# Policy example:
(
  rulename = "Critical System Files",
  severity = 100
)
{
  /etc/passwd              -> $(SEC_CRIT);
  /etc/shadow              -> $(SEC_CRIT);
  /etc/sudoers             -> $(SEC_CRIT);
  /usr/bin                 -> $(SEC_BIN);
  /usr/sbin                -> $(SEC_BIN);
}

# INITIALIZE
tripwire --init

# CHECK
tripwire --check

# UPDATE after legitimate changes
tripwire --update --twrfile /var/lib/tripwire/report/latest.twr

# VIEW REPORT
twprint --print-report --twrfile /var/lib/tripwire/report/latest.twr
```

---

## 4. AIDE vs Tripwire Comparison

| Feature | AIDE | Tripwire |
|---------|:----:|:--------:|
| **License** | GPL (free) | Open Source + Commercial |
| **Configuration** | Simple text file | Policy language |
| **Database format** | Plain file | Signed/encrypted |
| **Signed database** | ❌ | ✅ (cryptographic signing) |
| **Ease of use** | ★★★★★ | ★★★ |
| **Enterprise features** | Basic | Advanced (commercial) |
| **Speed** | Fast | Moderate |

---

## 5. Best Practices

| Practice | Description |
|----------|-------------|
| **Secure the database** | Store baseline on read-only media or remote server |
| **Automate checks** | Run via cron daily, email alerts |
| **Update after changes** | Update baseline after legitimate patches/changes |
| **Monitor the monitor** | Alert if AIDE/Tripwire is stopped or removed |
| **Exclude dynamic files** | Don't monitor /var/log, /tmp, /proc |
| **Use strong hashes** | SHA-256 or SHA-512 (not MD5) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **AIDE** | Free, simple FIM — widely used |
| **Tripwire** | Classic FIM with signed databases |
| **Baseline** | `aide --init` / `tripwire --init` |
| **Check** | `aide --check` / `tripwire --check` |
| **Automate** | Daily cron job with email alerts |
| **Database security** | Store off-system to prevent tampering |

---

## Quick Revision Questions

1. **How do you initialize an AIDE database?**
2. **What is the difference between AIDE and Tripwire?**
3. **Why should the FIM database be stored off-system?**
4. **When should you update the baseline database?**
5. **What files should be excluded from FIM monitoring?**

---

[← Previous: File Integrity Monitoring](03-file-integrity-monitoring.md) | [Next: Immutable Files →](05-immutable-files.md)

---

*Unit 4 - Topic 4 of 6 | [Back to README](../README.md)*
