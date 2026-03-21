# SELinux Basics

## Unit 5 - Topic 3 | Linux Access Control

---

## Overview

**SELinux (Security-Enhanced Linux)** is a MAC implementation developed by the NSA. It assigns **security labels (contexts)** to every process, file, port, and user, then uses a **policy** to decide what interactions are allowed. SELinux is the default MAC system on Red Hat, CentOS, Fedora, and Android.

---

## 1. SELinux Context Format

```
SECURITY CONTEXT FORMAT:
user:role:type:level

Example — File:
system_u:object_r:httpd_sys_content_t:s0
│        │        │                    │
│        │        │                    └── MLS level (Multi-Level Security)
│        │        └───────────────────── TYPE (most important for policy)
│        └────────────────────────────── ROLE (determines allowed types)
└─────────────────────────────────────── USER (SELinux user, not Linux user)

Example — Process:
system_u:system_r:httpd_t:s0

TYPE ENFORCEMENT (TE) is the primary mechanism:
"Process of type httpd_t can access files of type httpd_sys_content_t"
```

---

## 2. Essential SELinux Commands

```bash
# === STATUS ===
getenforce                            # Enforcing/Permissive/Disabled
sestatus                              # Detailed status

# === MODE SWITCHING ===
setenforce 0                          # Switch to Permissive (temporary)
setenforce 1                          # Switch to Enforcing (temporary)
# Permanent: edit /etc/selinux/config
# SELINUX=enforcing

# === VIEW CONTEXTS ===
ls -Z /var/www/html/                  # File contexts
ps -eZ | grep httpd                   # Process contexts
id -Z                                 # Current user context

# === CHANGE FILE CONTEXT ===
# Set context manually
chcon -t httpd_sys_content_t /var/www/html/index.html

# Restore default context (from policy)
restorecon -Rv /var/www/html/

# === MANAGE BOOLEANS ===
# Booleans are on/off switches for policy features
getsebool -a                          # List all booleans
getsebool httpd_can_network_connect   # Check specific boolean
setsebool -P httpd_can_network_connect on  # Enable permanently (-P)

# === PORT LABELS ===
semanage port -l | grep http          # View HTTP port labels
semanage port -a -t http_port_t -p tcp 8080  # Allow Apache on 8080

# === FILE CONTEXT RULES ===
semanage fcontext -l | grep httpd     # View file context rules
semanage fcontext -a -t httpd_sys_content_t "/custom/web(/.*)?"
restorecon -Rv /custom/web/           # Apply the rule
```

---

## 3. Troubleshooting SELinux

```bash
# === VIEW DENIALS ===
# AVC (Access Vector Cache) denials in audit log
grep "AVC" /var/log/audit/audit.log
ausearch -m AVC -ts recent

# Example denial:
# type=AVC msg=audit(1647123456.789:123): avc:  denied  { read }
# for pid=1234 comm="httpd" name="config.php"
# scontext=system_u:system_r:httpd_t:s0
# tcontext=unconfined_u:object_r:user_home_t:s0

# Translation: httpd (httpd_t) tried to read a file labeled user_home_t
# Policy doesn't allow httpd_t to read user_home_t → DENIED

# === AUTOMATED TROUBLESHOOTING ===
apt install setroubleshoot-server     # or yum install
sealert -a /var/log/audit/audit.log   # Analyze denials
# Provides human-readable explanations and suggested fixes

# === GENERATE POLICY MODULE ===
# If a denial is legitimate, create a custom policy:
ausearch -m AVC -ts recent | audit2allow -M mypolicy
semodule -i mypolicy.pp               # Install custom module
```

---

## 4. Common SELinux Booleans

| Boolean | Default | Purpose |
|---------|:-------:|---------|
| `httpd_can_network_connect` | off | Allow Apache to make network connections |
| `httpd_can_sendmail` | off | Allow Apache to send email |
| `httpd_enable_cgi` | on | Allow CGI scripts |
| `allow_ftpd_full_access` | off | Full FTP access to filesystem |
| `samba_enable_home_dirs` | off | Samba access to home dirs |
| `ssh_sysadm_login` | off | SSH login as sysadm_r role |

---

## 5. SELinux Policy Types

| Policy | Description | Use |
|--------|-------------|-----|
| **Targeted** | Only targeted services confined (default) | Most Linux servers |
| **MLS** | Multi-Level Security (military grade) | Classified environments |
| **Minimum** | Only selected processes confined | Lightweight |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SELinux** | Label-based MAC from NSA |
| **Context** | user:role:type:level |
| **Type Enforcement** | Primary access control mechanism |
| **Booleans** | On/off switches for policy features |
| **restorecon** | Fix file contexts |
| **audit2allow** | Generate custom policy from denials |

---

## Quick Revision Questions

1. **What is the format of an SELinux security context?**
2. **How do you check if SELinux is enforcing?**
3. **What is a SELinux boolean and how do you change one?**
4. **How do you troubleshoot an SELinux denial?**
5. **What does `restorecon` do?**

---

[← Previous: Mandatory Access Control](02-mandatory-access-control.md) | [Next: AppArmor Basics →](04-apparmor-basics.md)

---

*Unit 5 - Topic 3 of 6 | [Back to README](../README.md)*
