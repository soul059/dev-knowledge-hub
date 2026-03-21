# AD Enumeration Techniques

## Unit 2 - Topic 1 | AD Enumeration

---

## Overview

**AD enumeration** is the process of querying Active Directory to discover users, groups, computers, trusts, policies, and attack paths. It's typically the first action after gaining initial domain access — transforming a single foothold into a complete map of the environment.

---

## 1. Enumeration Strategy

```
AD ENUMERATION WORKFLOW:

1. IDENTIFY DOMAIN INFO
   ├── Domain name and SID
   ├── Domain Controllers
   ├── Domain functional level
   └── Forest structure

2. ENUMERATE USERS
   ├── All domain users
   ├── Admin accounts (adminCount=1)
   ├── Service accounts (SPNs)
   ├── Disabled/inactive accounts
   └── Descriptions (may contain passwords!)

3. ENUMERATE GROUPS
   ├── Domain Admins members
   ├── Enterprise Admins members
   ├── High-privilege groups
   ├── Nested group memberships
   └── Custom admin groups

4. ENUMERATE COMPUTERS
   ├── Domain Controllers
   ├── Servers (OS, version)
   ├── Workstations
   ├── Unconstrained delegation
   └── LAPS configured

5. ENUMERATE POLICIES
   ├── Password policy
   ├── Lockout policy
   ├── GPOs and permissions
   └── Fine-grained password policies

6. ENUMERATE TRUSTS
   ├── Domain trusts
   ├── Forest trusts
   ├── Trust direction and type
   └── SID filtering status

7. MAP ATTACK PATHS
   └── BloodHound for visual analysis
```

---

## 2. Built-in Windows Commands

```powershell
# === DOMAIN INFORMATION ===
systeminfo | findstr /B /C:"Domain"
echo %USERDNSDOMAIN%
echo %logonserver%
nltest /dclist:corp.local
nltest /domain_trusts

# === USER ENUMERATION ===
net user /domain
net user administrator /domain
net user jsmith /domain

# === GROUP ENUMERATION ===
net group /domain
net group "Domain Admins" /domain
net group "Enterprise Admins" /domain
net group "Schema Admins" /domain

# === COMPUTER ENUMERATION ===
net group "Domain Computers" /domain
net group "Domain Controllers" /domain

# === POLICY ===
net accounts /domain
# Shows: password policy, lockout settings

# === SHARES ===
net view \\dc01
net share
```

---

## 3. Key Enumeration Tools

| Tool | Type | Best For |
|------|------|---------|
| **PowerView** | PowerShell | Comprehensive AD enum |
| **BloodHound** | Graph-based | Attack path visualization |
| **ldapsearch** | Linux CLI | LDAP queries from Linux |
| **CrackMapExec** | Python | Network-wide enum |
| **ADFind** | Windows CLI | Fast LDAP queries |
| **enum4linux-ng** | Linux | SMB/RPC enumeration |
| **Impacket** | Python | Multiple AD tools |
| **SharpView** | C# | .NET version of PowerView |

---

## 4. Quick Enumeration Cheat Sheet

```bash
# === FROM LINUX (Impacket + CME) ===
# Domain info:
crackmapexec smb dc01 -u user -p pass

# Users:
crackmapexec smb dc01 -u user -p pass --users
GetADUsers.py corp.local/user:pass -dc-ip dc01 -all

# Groups:
crackmapexec smb dc01 -u user -p pass --groups

# Shares:
crackmapexec smb 10.0.0.0/24 -u user -p pass --shares

# Password policy:
crackmapexec smb dc01 -u user -p pass --pass-pol

# Computers:
GetADComputers.py corp.local/user:pass -dc-ip dc01

# === FROM WINDOWS (PowerView) ===
Import-Module PowerView.ps1

Get-Domain                      # Domain info
Get-DomainController            # DCs
Get-DomainUser                  # All users
Get-DomainGroup                 # All groups
Get-DomainComputer              # All computers
Get-DomainGPO                   # All GPOs
Get-DomainTrust                 # Trusts
Get-DomainPolicy                # Policies
```

---

## 5. What to Look For

| Finding | Why Important |
|---------|--------------|
| **Passwords in descriptions** | Instant credential access |
| **Kerberoastable accounts** | Offline hash cracking |
| **ASREProastable accounts** | No-password hash capture |
| **Unconstrained delegation** | TGT theft |
| **Weak password policy** | Enables spraying/brute force |
| **No LAPS** | Same local admin everywhere |
| **Old OS** | Unpatched vulnerabilities |
| **Nested admin groups** | Hidden privilege escalation |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Enumeration** | First step after initial AD access |
| **PowerView** | PowerShell-based AD enum framework |
| **CrackMapExec** | Linux-based network AD enum |
| **BloodHound** | Visual attack path analysis |
| **Key targets** | Admins, SPNs, delegation, descriptions |
| **Workflow** | Domain → Users → Groups → Computers → Trusts |

---

## Quick Revision Questions

1. **What is the first thing to enumerate after gaining domain access?**
2. **What built-in Windows commands enumerate AD?**
3. **Why should you check user description fields?**
4. **What tool provides visual attack path analysis?**
5. **How do you enumerate the domain password policy?**

---

[Next: LDAP Queries →](02-ldap-queries.md)

---

*Unit 2 - Topic 1 of 6 | [Back to README](../README.md)*
