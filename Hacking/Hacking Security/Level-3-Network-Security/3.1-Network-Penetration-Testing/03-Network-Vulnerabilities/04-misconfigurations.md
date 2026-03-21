# Misconfigurations

## Unit 3 - Topic 4 | Network Vulnerabilities

---

## Overview

**Misconfigurations** are security weaknesses caused by incorrect or insecure settings rather than software bugs. They're the most common vulnerability type and often provide easier exploitation paths than software vulnerabilities.

---

## 1. Common Misconfiguration Categories

```
NETWORK MISCONFIGURATIONS:

FIREWALL:
├── Overly permissive rules (allow all outbound)
├── Missing egress filtering
├── Default allow rules
├── Unused open ports
├── No logging enabled
└── Management interface exposed to internet

SERVICES:
├── Unnecessary services running
├── Services bound to 0.0.0.0 (all interfaces)
├── Anonymous access enabled (FTP, SMB)
├── Debug mode enabled in production
├── Directory listing enabled (HTTP)
└── Error messages revealing internals

ENCRYPTION:
├── HTTP instead of HTTPS
├── Weak TLS versions (TLS 1.0/1.1)
├── Weak cipher suites
├── Self-signed certificates
├── Expired certificates
└── Missing HSTS headers

ACCESS CONTROL:
├── Flat network (no segmentation)
├── Excessive user permissions
├── Shared admin accounts
├── No network access control (NAC)
└── Open wireless network
```

---

## 2. Detecting Misconfigurations

```bash
# === ANONYMOUS ACCESS ===
# FTP anonymous login:
nmap -p 21 --script ftp-anon target.com
ftp target.com   # Login as: anonymous

# SMB null session:
smbclient -L //target.com -N
enum4linux -a target.com

# NFS open shares:
showmount -e target.com
# /home  *(everyone can mount!)

# === WEB SERVER MISCONFIGS ===
# Directory listing:
curl https://target.com/uploads/
# If you see file listing → misconfigured!

# .htaccess bypass:
curl https://target.com/.env
curl https://target.com/.git/config
curl https://target.com/wp-config.php.bak

# HTTP methods:
nmap -p 80 --script http-methods target.com
# PUT and DELETE enabled = dangerous!

# === SSL/TLS CONFIGURATION ===
nmap -p 443 --script ssl-enum-ciphers target.com
# Shows: supported TLS versions, cipher suites
# BAD: TLS 1.0, RC4, DES, NULL ciphers

testssl.sh target.com
# Comprehensive SSL/TLS testing

# === SNMP CONFIGURATION ===
nmap -sU -p 161 --script snmp-info target.com
snmpwalk -v2c -c public target.com
# "public" community string = misconfigured!
```

---

## 3. Critical Misconfigurations

| Misconfiguration | Risk Level | Exploitation |
|-----------------|:----------:|-------------|
| **Anonymous FTP with write** | ★★★★★ | Upload web shell |
| **SMB null session** | ★★★★ | Enumerate users, shares |
| **NFS everyone access** | ★★★★★ | Mount and read/write files |
| **MongoDB no auth** | ★★★★★ | Full database access |
| **Redis no auth** | ★★★★★ | RCE via module load |
| **SNMP public community** | ★★★★ | Read system information |
| **HTTP PUT enabled** | ★★★★★ | Upload web shell |
| **Debug mode** | ★★★★ | Detailed error → exploit |
| **Default SSH keys** | ★★★★★ | Immediate access |

---

## 4. Cloud Misconfigurations

```
CLOUD-SPECIFIC MISCONFIGURATIONS:

AWS:
├── Public S3 buckets
├── Overly permissive IAM policies
├── Security groups allowing 0.0.0.0/0
├── Unencrypted EBS volumes
├── Public RDS instances
└── Missing CloudTrail logging

AZURE:
├── Public blob storage
├── Network Security Group too permissive
├── Storage account public access
├── Missing activity logs
└── Key Vault access policies too broad

COMMON ACROSS ALL CLOUDS:
├── API keys in public repositories
├── Exposed management consoles
├── No MFA on admin accounts
├── Over-privileged service accounts
├── Missing encryption at rest
└── Logging and monitoring gaps
```

---

## 5. Defensive Hardening Checklist

```
HARDENING CHECKLIST:

NETWORK:
□ Close unnecessary ports
□ Implement network segmentation
□ Enable firewall egress filtering
□ Disable LLMNR and NBT-NS
□ Enable logging on all devices

SERVICES:
□ Disable anonymous access
□ Remove default accounts
□ Bind services to specific interfaces
□ Disable directory listing
□ Remove debug/development endpoints

ENCRYPTION:
□ Enforce TLS 1.2+ only
□ Use strong cipher suites
□ Enable HSTS
□ Use valid certificates
□ Encrypt data at rest

ACCESS:
□ Implement least privilege
□ Enable MFA for admin accounts
□ Use individual accounts (no shared)
□ Review permissions regularly
□ Implement NAC
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Misconfigurations** | Incorrect settings, not software bugs |
| **Anonymous access** | FTP, SMB, NFS with no authentication |
| **Web misconfigs** | Directory listing, debug mode, PUT method |
| **SSL/TLS** | Weak versions and cipher suites |
| **Cloud** | Public buckets, permissive security groups |
| **Defense** | Hardening checklists + regular audits |

---

## Quick Revision Questions

1. **Why are misconfigurations more common than software vulnerabilities?**
2. **How do you test for anonymous FTP access?**
3. **What makes an NFS export dangerous?**
4. **Name 3 critical cloud misconfigurations.**
5. **What is the HTTP PUT method misconfiguration?**

---

[← Previous: Unpatched Services](03-unpatched-services.md) | [Next: Legacy Protocols →](05-legacy-protocols.md)

---

*Unit 3 - Topic 4 of 6 | [Back to README](../README.md)*
