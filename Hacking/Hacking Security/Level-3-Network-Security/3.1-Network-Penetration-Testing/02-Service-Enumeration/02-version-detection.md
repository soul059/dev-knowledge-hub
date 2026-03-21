# Version Detection

## Unit 2 - Topic 2 | Service Enumeration

---

## Overview

**Version detection** goes beyond banner grabbing to actively probe services and identify exact software versions. While banners can be modified or suppressed, version detection uses protocol-specific probes that are much harder to fake.

---

## 1. Nmap Version Detection

```bash
# === BASIC VERSION DETECTION ===
nmap -sV target.com
# Probes open ports with service-specific packets
# Returns: service name, product, version, extra info

# === VERSION INTENSITY ===
nmap -sV --version-intensity 0 target.com  # Light probes (fast)
nmap -sV --version-intensity 5 target.com  # Default
nmap -sV --version-intensity 9 target.com  # All probes (thorough)

# === VERSION + DEFAULT SCRIPTS ===
nmap -sV -sC target.com
# Combines version detection with default NSE scripts
# Scripts provide additional version-specific information

# === EXAMPLE OUTPUT ===
# PORT     STATE SERVICE  VERSION
# 22/tcp   open  ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
# 80/tcp   open  http     Apache httpd 2.4.49 ((Ubuntu))
# 443/tcp  open  ssl/http nginx 1.24.0
# 3306/tcp open  mysql    MySQL 5.7.40-0ubuntu0.18.04.1
# 8080/tcp open  http     Apache Tomcat 9.0.73
```

---

## 2. How Version Detection Works

```
NMAP VERSION DETECTION PROCESS:

1. PORT FOUND OPEN (from port scan)
   └── Port 80 is open

2. NULL PROBE (connect and wait)
   └── Some services send banner immediately
   └── Matches against nmap-service-probes database

3. SERVICE-SPECIFIC PROBES
   ├── Send HTTP request → analyze response
   ├── Send SSL ClientHello → analyze certificate
   ├── Send SSH version string → get server version
   └── Each probe has regex patterns for matching

4. MATCH AGAINST DATABASE
   └── nmap-service-probes contains 11,000+ signatures
   └── Returns: product, version, OS, devicetype

5. CONFIDENCE REPORTING
   └── Reports match confidence level
   └── Aggressive guessing with --version-all

PROBE DATABASE: /usr/share/nmap/nmap-service-probes
```

---

## 3. Service-Specific Version Detection

```bash
# === WEB SERVERS ===
# HTTP headers:
curl -sI target.com | grep -i server
# Server: Apache/2.4.49

# whatweb for detailed detection:
whatweb target.com
# Apache[2.4.49], PHP[8.1], jQuery[3.6.0], WordPress[6.3]

# === SSL/TLS VERSION ===
nmap -p 443 --script ssl-enum-ciphers target.com
# Shows: TLS versions, cipher suites, certificate

openssl s_client -connect target.com:443
# SSL version, cipher, certificate chain

# === DATABASE SERVERS ===
nmap -p 3306 --script mysql-info target.com
nmap -p 1433 --script ms-sql-info target.com
nmap -p 5432 --script pgsql-info target.com

# === MAIL SERVERS ===
nmap -p 25 --script smtp-commands target.com
# Lists SMTP capabilities → identifies software

# === SMB/WINDOWS ===
nmap -p 445 --script smb-os-discovery target.com
# Returns: OS, domain, computer name, NetBIOS name

# === SNMP ===
snmpwalk -v2c -c public target.com 1.3.6.1.2.1.1.1.0
# System description with OS and version
```

---

## 4. Version to Vulnerability Mapping

```bash
# === WORKFLOW ===
# Step 1: Version Detection
nmap -sV target.com -oX scan.xml

# Step 2: Automatic Exploit Search
searchsploit --nmap scan.xml
# Searches Exploit-DB for each detected version

# Step 3: Manual CVE Research
# Apache 2.4.49 → CVE-2021-41773 (Critical!)
# MySQL 5.7.40  → Check MySQL advisories
# OpenSSH 8.9   → Generally safe

# === EXAMPLE SEARCHSPLOIT OUTPUT ===
# [*] Nmap file successfully parsed
# ────────────────────────────────────────
# Apache httpd 2.4.49
# │ Apache HTTP Server 2.4.49 - Path Traversal & RCE
# │ Path: exploits/multiple/webapps/50383.sh
# ────────────────────────────────────────
# MySQL 5.7.40
# │ No matching results
# ────────────────────────────────────────
```

---

## 5. Comparison of Detection Methods

| Method | Accuracy | Speed | Stealth |
|--------|:--------:|:-----:|:-------:|
| **Banner grab (nc)** | ★★★ | ★★★★★ | ★★★★ |
| **Nmap -sV** | ★★★★★ | ★★★ | ★★★ |
| **Nmap -sV -sC** | ★★★★★ | ★★ | ★★ |
| **WhatWeb** | ★★★★ | ★★★★ | ★★★ |
| **Manual probing** | ★★★★ | ★ | ★★★★ |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **-sV** | Nmap service version detection |
| **Intensity** | 0 (light) to 9 (all probes) |
| **Probe database** | 11,000+ signatures in nmap-service-probes |
| **searchsploit** | `--nmap scan.xml` auto CVE lookup |
| **WhatWeb** | Web-specific technology fingerprinting |
| **Accuracy** | Active probing > passive banner grab |

---

## Quick Revision Questions

1. **How does Nmap version detection differ from banner grabbing?**
2. **What does `--version-intensity 9` do?**
3. **How do you automatically search for exploits from Nmap results?**
4. **What Nmap script detects Windows OS version via SMB?**
5. **Why is active version detection harder to fake than banners?**

---

[← Previous: Banner Grabbing](01-banner-grabbing.md) | [Next: Script Scanning (NSE) →](03-script-scanning-nse.md)

---

*Unit 2 - Topic 2 of 6 | [Back to README](../README.md)*
