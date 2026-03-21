# Banner Grabbing

## Unit 2 - Topic 1 | Service Enumeration

---

## Overview

**Banner grabbing** captures the identification strings (banners) that services send when a connection is established. These banners reveal software names, versions, operating systems, and configuration details — essential intelligence for vulnerability research.

---

## 1. What Banners Reveal

```
SERVICE BANNER EXAMPLES:

SSH:
SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.4
│      │          │         │
│      │          │         └── Distribution & patch level
│      │          └── Version 8.9p1
│      └── Product: OpenSSH
└── Protocol version 2.0

HTTP:
Server: Apache/2.4.49 (Ubuntu)
X-Powered-By: PHP/8.1.2
│                    │
│                    └── Backend language & version
└── Web server & version

SMTP:
220 mail.target.com ESMTP Postfix (Ubuntu)
│                          │         │
│                          │         └── OS
│                          └── Mail server
└── SMTP ready code

FTP:
220 (vsFTPd 3.0.3)
         │
         └── FTP server & version

MySQL:
5.7.40-0ubuntu0.18.04.1
│       │
│       └── OS distribution
└── MySQL version
```

---

## 2. Banner Grabbing Techniques

```bash
# === NETCAT (Most Basic) ===
nc -nv 203.0.113.10 22
# SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.4

nc -nv 203.0.113.10 80
HEAD / HTTP/1.0
Host: target.com
# (press Enter twice)

nc -nv 203.0.113.10 25
# 220 mail.target.com ESMTP Postfix

# === TELNET ===
telnet 203.0.113.10 21
# 220 (vsFTPd 3.0.3)

telnet 203.0.113.10 3306
# Shows MySQL version in banner

# === CURL (HTTP-specific) ===
curl -I https://target.com
# Server: nginx/1.24.0
# X-Powered-By: PHP/8.1

curl -sI https://target.com | grep -i "server\|x-powered\|x-aspnet"

# === OPENSSL (HTTPS/TLS services) ===
openssl s_client -connect target.com:443
# Shows: SSL version, certificate details, cipher
# Then type: HEAD / HTTP/1.0 (to get HTTP headers)

# === NMAP BANNER SCRIPT ===
nmap --script banner -p 21,22,25,80,110,143 target.com
# Grabs banners from all specified ports

nmap -sV --script banner target.com
# Version detection + explicit banner grab
```

---

## 3. Service-Specific Banner Grabs

```bash
# === SSH ===
nc -nv target.com 22
# → SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.4

# === SMTP ===
nc -nv target.com 25
EHLO test
# → Lists supported SMTP extensions and capabilities

# === FTP ===
nc -nv target.com 21
# → 220 (vsFTPd 3.0.3)

# === SMB (Using Nmap) ===
nmap -p 445 --script smb-os-discovery target.com
# → OS: Windows Server 2019, Domain: CORP

# === SNMP ===
snmpwalk -v2c -c public target.com system
# → System description, OS, uptime

# === RDP (Nmap Script) ===
nmap -p 3389 --script rdp-ntlm-info target.com
# → OS version, domain name, DNS name

# === MySQL ===
nmap -p 3306 --script mysql-info target.com
# → Version, capabilities, salt
```

---

## 4. Banner Analysis

| Banner Element | Intelligence Value |
|---------------|-------------------|
| **Product name** | Identify software for CVE lookup |
| **Version number** | Exact CVE matching |
| **OS information** | Attack vector selection |
| **Patch level** | Determine if patched |
| **Configuration** | Identify misconfigurations |
| **Custom banners** | Organization identification |

---

## 5. Defensive Countermeasures

```bash
# === HIDE/MODIFY BANNERS ===

# Apache — suppress version:
# ServerTokens Prod
# ServerSignature Off

# nginx — hide version:
# server_tokens off;

# OpenSSH — can't easily hide but keep updated

# Postfix — custom banner:
# smtpd_banner = $myhostname ESMTP

# === IDS DETECTION ===
# Alert on banner grab patterns:
# Multiple connections to different ports
# HEAD/OPTIONS requests
# Connections that disconnect after banner
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Banner** | Service identification string on connect |
| **Netcat** | `nc -nv target port` for basic grab |
| **Nmap** | `--script banner` for automated grabbing |
| **curl -I** | HTTP header banner information |
| **Intelligence** | Product + Version = CVE lookup |
| **Defense** | Suppress version info in configs |

---

## Quick Revision Questions

1. **What information do service banners typically reveal?**
2. **How do you grab a banner with Netcat?**
3. **What Nmap script performs banner grabbing?**
4. **How can defenders hide banner information?**
5. **Why is SSH banner grabbing valuable?**

---

[Next: Version Detection →](02-version-detection.md)

---

*Unit 2 - Topic 1 of 6 | [Back to README](../README.md)*
