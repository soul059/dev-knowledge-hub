# Service Detection

## Unit 8 - Topic 2 | Active Reconnaissance

---

## Overview

**Service detection** (version scanning) identifies the specific software and version running on open ports. Knowing the exact service version enables targeted vulnerability research, exploit selection, and accurate risk assessment.

---

## 1. Service vs Port

```
PORT ALONE TELLS YOU VERY LITTLE:

Port 80   → HTTP?  (probably, but could be anything)
Port 443  → HTTPS? (probably, but could be SSH tunneled)
Port 8080 → HTTP proxy? Jenkins? Tomcat? API?
Port 22   → SSH?   (might be on a non-standard port)

SERVICE DETECTION TELLS YOU:
Port 80   → Apache httpd 2.4.49 (Ubuntu)
Port 443  → nginx 1.24.0 (OpenSSL 3.0.2)
Port 8080 → Apache Tomcat 9.0.73
Port 22   → OpenSSH 8.9p1 Ubuntu 3ubuntu0.4

VERSION TELLS YOU → EXACT VULNERABILITIES:
Apache 2.4.49  → CVE-2021-41773 (Path Traversal!) ★★★★★
OpenSSH 8.9p1  → No critical CVEs (safe)
Tomcat 9.0.73  → Check for known issues
```

---

## 2. Nmap Version Detection

```bash
# === BASIC VERSION DETECTION ===
nmap -sV target.com
# Probes open ports to determine service/version

# === VERSION DETECTION INTENSITY ===
nmap -sV --version-intensity 0 target.com  # Light (fast)
nmap -sV --version-intensity 5 target.com  # Default
nmap -sV --version-intensity 9 target.com  # All probes (slow)

# === VERSION DETECTION + OS DETECTION ===
nmap -sV -O target.com
# -sV: Service versions
# -O:  Operating system detection

# === NSE SCRIPTS FOR SERVICE DETECTION ===
nmap -sV -sC target.com
# -sC: Default scripts (banner grab, enum, etc.)

# Specific service scripts:
nmap -sV --script http-title target.com
nmap -sV --script ssl-cert target.com
nmap -sV --script smb-os-discovery target.com
nmap -sV --script mysql-info target.com

# === COMPREHENSIVE SERVICE SCAN ===
sudo nmap -sS -sV -sC -O -p- --open -T4 -oA services target.com
# SYN scan + versions + scripts + OS + all ports + open only
```

---

## 3. Banner Grabbing Techniques

```bash
# === MANUAL BANNER GRABBING ===

# Netcat:
nc -nv 203.0.113.10 22
# SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.4

nc -nv 203.0.113.10 80
HEAD / HTTP/1.0
Host: target.com

# HTTP headers reveal:
# Server: Apache/2.4.49 (Ubuntu)
# X-Powered-By: PHP/8.1.2

# === TELNET ===
telnet 203.0.113.10 25
# 220 mail.target.com ESMTP Postfix (Ubuntu)

# === CURL (HTTP Services) ===
curl -I https://target.com
# Server: nginx/1.24.0
# X-Powered-By: Express  (Node.js)

# === OPENSSL (HTTPS Services) ===
openssl s_client -connect target.com:443
# Shows: SSL/TLS version, certificate details

# === AUTOMATED BANNER GRAB ===
nmap -sV --script banner -p 22,80,443 target.com
```

---

## 4. Common Services and Ports

| Port | Service | Detection Clues |
|:----:|---------|----------------|
| 21 | FTP | Banner: "220 ProFTPD 1.3.5" |
| 22 | SSH | Banner: "SSH-2.0-OpenSSH_8.9" |
| 25 | SMTP | Banner: "220 ESMTP Postfix" |
| 53 | DNS | Response to DNS query |
| 80 | HTTP | Server header, page content |
| 110 | POP3 | Banner: "+OK Dovecot ready" |
| 143 | IMAP | Banner: "* OK [CAPABILITY IMAP4rev1]" |
| 443 | HTTPS | SSL cert, Server header |
| 445 | SMB | SMB negotiation response |
| 1433 | MSSQL | Banner: "Microsoft SQL Server" |
| 3306 | MySQL | Banner: "5.7.40-0ubuntu" |
| 3389 | RDP | RDP security negotiation |
| 5432 | PostgreSQL | SSL negotiation response |
| 8080 | HTTP Alt | Tomcat, Jenkins, proxy |
| 8443 | HTTPS Alt | Admin panels, API gateways |

---

## 5. Service → Vulnerability Research

```bash
# === SEARCHSPLOIT (Exploit-DB Offline) ===
searchsploit apache 2.4.49
# Apache HTTP Server 2.4.49 - Path Traversal & RCE

searchsploit openssh 8.9
# No results = likely no public exploits

searchsploit tomcat 9.0
# Lists known Tomcat vulnerabilities

# === CVE LOOKUP ===
# nvd.nist.gov — search by product and version
# cvedetails.com — comprehensive CVE database
# vulners.com — vulnerability search engine

# === NMAP VULN SCRIPTS ===
nmap --script vuln target.com
# Runs vulnerability detection scripts

nmap --script http-vuln* target.com
# HTTP-specific vulnerability checks

# === WORKFLOW ===
# 1. Port scan → find open ports
# 2. Service scan → identify software + version
# 3. Vulnerability research → find known CVEs
# 4. Exploit selection → match CVE to exploit
# 5. Exploitation → attempt authorized access
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **-sV** | Nmap version detection flag |
| **Banner grabbing** | Read service identification strings |
| **Version precision** | Exact version → exact CVE match |
| **searchsploit** | Offline Exploit-DB search |
| **NSE scripts** | Automated service enumeration |
| **Workflow** | Ports → Services → CVEs → Exploits |

---

## Quick Revision Questions

1. **Why is version detection more useful than just port scanning?**
2. **What Nmap flags perform version and OS detection?**
3. **How do you banner grab manually with netcat?**
4. **What is searchsploit and how do you use it?**
5. **What Nmap script category checks for vulnerabilities?**

---

[← Previous: Port Scanning](01-port-scanning.md) | [Next: OS Fingerprinting →](03-os-fingerprinting.md)

---

*Unit 8 - Topic 2 of 5 | [Back to README](../README.md)*
