# Common Network Vulnerabilities

## Unit 3 - Topic 1 | Network Vulnerabilities

---

## Overview

**Common network vulnerabilities** are weaknesses in network infrastructure, services, and configurations that attackers exploit to gain unauthorized access, intercept data, or disrupt services. Understanding these vulnerabilities is essential for both exploitation and remediation.

---

## 1. Vulnerability Categories

```
NETWORK VULNERABILITY TAXONOMY:

SOFTWARE VULNERABILITIES:
├── Unpatched services (known CVEs)
├── Zero-day exploits (unknown to vendor)
├── Buffer overflows
├── Remote code execution (RCE)
└── Authentication bypass

CONFIGURATION VULNERABILITIES:
├── Default credentials
├── Open ports/services (unnecessary)
├── Weak encryption
├── Missing access controls
└── Excessive permissions

PROTOCOL VULNERABILITIES:
├── ARP (no authentication → spoofing)
├── DNS (cache poisoning)
├── DHCP (rogue servers)
├── SNMP (cleartext community strings)
├── Telnet (cleartext transmission)
└── FTP (cleartext credentials)

ARCHITECTURE VULNERABILITIES:
├── Flat network (no segmentation)
├── Single point of failure
├── Missing monitoring/logging
├── Inadequate backup
└── No redundancy
```

---

## 2. Critical Network CVEs (Historical)

| CVE | Name | Target | CVSS | Impact |
|-----|------|--------|:----:|--------|
| CVE-2017-0144 | **EternalBlue** | SMBv1 (Windows) | 9.8 | RCE → WannaCry |
| CVE-2014-0160 | **Heartbleed** | OpenSSL | 7.5 | Memory disclosure |
| CVE-2014-6271 | **Shellshock** | Bash (CGI) | 9.8 | RCE |
| CVE-2021-44228 | **Log4Shell** | Log4j | 10.0 | RCE |
| CVE-2021-34527 | **PrintNightmare** | Windows Print | 8.8 | RCE + LPE |
| CVE-2020-1472 | **Zerologon** | Netlogon | 10.0 | DC takeover |
| CVE-2021-26855 | **ProxyLogon** | Exchange | 9.8 | RCE |
| CVE-2019-0708 | **BlueKeep** | RDP | 9.8 | RCE |

---

## 3. Vulnerability Discovery Process

```bash
# === STEP 1: SCAN ===
sudo nmap -sS -sV -sC -O -p- -T4 -oA network_scan target_range

# === STEP 2: IDENTIFY VULNERABLE VERSIONS ===
searchsploit --nmap network_scan.xml

# === STEP 3: VULNERABILITY SCANNER ===
# Nessus / OpenVAS for comprehensive scanning
# Checks: missing patches, misconfigs, default creds

# === STEP 4: MANUAL VERIFICATION ===
# Verify scanner findings aren't false positives
# Test specific CVEs with Metasploit or manual exploits

# === STEP 5: DOCUMENT AND PRIORITIZE ===
# Create vulnerability matrix
# Prioritize by CVSS × Exploitability × Exposure
```

---

## 4. Common Attack Vectors

```
NETWORK ATTACK VECTORS:

EXTERNAL (Internet-facing):
├── Web application vulnerabilities
├── VPN gateway exploits
├── Email server attacks (ProxyLogon)
├── RDP brute force (BlueKeep)
├── DNS attacks
└── Exposed management interfaces

INTERNAL (After initial access):
├── LLMNR/NBT-NS poisoning → credential capture
├── SMB relay attacks
├── Kerberoasting
├── ARP spoofing → MitM
├── Lateral movement via Pass-the-Hash
├── Privilege escalation exploits
└── Domain controller attacks (Zerologon)

WIRELESS:
├── WPA2 handshake capture + crack
├── Evil twin attacks
├── Deauthentication attacks
├── Rogue access points
└── Bluetooth exploitation
```

---

## 5. Vulnerability Impact Levels

| Level | Example | Business Impact |
|:-----:|---------|----------------|
| **Critical** | RCE on internet-facing server | Full system compromise |
| **High** | Authentication bypass | Unauthorized data access |
| **Medium** | Information disclosure | Intelligence for further attacks |
| **Low** | Server header disclosure | Minor information leak |
| **Informational** | Open but unused port | Minimal direct impact |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Software vulns** | Known CVEs in unpatched services |
| **Config vulns** | Default creds, weak encryption |
| **Protocol vulns** | Inherent protocol weaknesses |
| **EternalBlue** | SMBv1 RCE (most impactful network vuln) |
| **Discovery** | Scan → identify → verify → prioritize |
| **Prioritization** | CVSS + exploit availability + exposure |

---

## Quick Revision Questions

1. **What are the four categories of network vulnerabilities?**
2. **Name 3 critical network CVEs and their impact.**
3. **What is the vulnerability discovery process?**
4. **Why are protocol vulnerabilities particularly dangerous?**
5. **How do you prioritize discovered vulnerabilities?**

---

[Next: Default Credentials →](02-default-credentials.md)

---

*Unit 3 - Topic 1 of 6 | [Back to README](../README.md)*
