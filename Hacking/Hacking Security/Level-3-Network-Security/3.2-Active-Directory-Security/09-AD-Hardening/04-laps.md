# LAPS (Local Administrator Password Solution)

## Unit 9 - Topic 4 | AD Hardening

---

## Overview

**LAPS** (Local Administrator Password Solution) automatically manages and rotates local administrator passwords on domain-joined machines, storing them securely in Active Directory. It prevents lateral movement via shared local admin passwords — one of the most common attack paths in AD environments.

---

## 1. The Problem LAPS Solves

```
WITHOUT LAPS:
┌──────────┐  ┌──────────┐  ┌──────────┐
│   WS01   │  │   WS02   │  │   WS03   │
│          │  │          │  │          │
│ Local    │  │ Local    │  │ Local    │
│ Admin:   │  │ Admin:   │  │ Admin:   │
│ P@ssw0rd │  │ P@ssw0rd │  │ P@ssw0rd │
└──────────┘  └──────────┘  └──────────┘
     ↑ Same password everywhere!
     
Attacker dumps hash on WS01
→ PtH to WS02, WS03... and EVERY machine!
→ Massive lateral movement!

WITH LAPS:
┌──────────┐  ┌──────────┐  ┌──────────┐
│   WS01   │  │   WS02   │  │   WS03   │
│          │  │          │  │          │
│ Local    │  │ Local    │  │ Local    │
│ Admin:   │  │ Admin:   │  │ Admin:   │
│ x7Kp!9mQ │  │ Rt5#wLzN │  │ Yq2@vBnF │
└──────────┘  └──────────┘  └──────────┘
     ↑ UNIQUE password per machine!
     ↑ Auto-rotated every 30 days!
     
Attacker dumps hash on WS01
→ Only works on WS01!
→ Lateral movement BLOCKED!
```

---

## 2. LAPS Architecture

```
LAPS COMPONENTS:

CLIENT-SIDE:
├── LAPS CSE (Client-Side Extension)
│   ├── Group Policy extension on each machine
│   ├── Generates random password
│   ├── Changes local admin password
│   └── Writes new password to AD
│
└── Runs during GPO processing
    (every 90 min or at boot/logon)

SERVER-SIDE:
├── AD Schema Extension
│   ├── ms-Mcs-AdmPwd (password attribute)
│   ├── ms-Mcs-AdmPwdExpirationTime (expiry)
│   └── Stored on computer objects in AD
│
├── ACL Controls
│   ├── Who can READ the password?
│   ├── Who can RESET/FORCE expiration?
│   └── Default: Domain Admins only
│
└── GPO Settings
    ├── Password length and complexity
    ├── Password age
    ├── Target admin account name
    └── Enable/disable

FLOW:
┌────────┐   1. GPO triggers    ┌────────────┐
│Machine │   password change    │ LAPS CSE   │
│        │ ─────────────────→  │ generates  │
│        │                      │ random pwd │
│        │   2. Changes local   │            │
│        │   admin password     │            │
│        │ ←─────────────────── │            │
│        │                      │            │
│        │   3. Writes pwd      │            │
│        │   to AD attribute    │            │
│        │ ─────────────────→  │ → AD (ms-  │
│        │                      │ Mcs-AdmPwd)│
└────────┘                      └────────────┘
```

---

## 3. Deploying LAPS

```powershell
# === LEGACY LAPS (Microsoft LAPS) ===

# Step 1: Extend AD Schema (run on DC):
Import-Module AdmPwd.PS
Update-AdmPwdADSchema
# Adds ms-Mcs-AdmPwd and ms-Mcs-AdmPwdExpirationTime

# Step 2: Set permissions (who can READ passwords):
Set-AdmPwdReadPasswordPermission -OrgUnit "OU=Workstations,DC=corp,DC=local" \
  -AllowedPrincipals "CORP\Tier2_Admins"
# Only Tier2_Admins can read workstation passwords

Set-AdmPwdReadPasswordPermission -OrgUnit "OU=Servers,DC=corp,DC=local" \
  -AllowedPrincipals "CORP\Tier1_Admins"

# Step 3: Grant machines permission to update their own password:
Set-AdmPwdComputerSelfPermission -OrgUnit "OU=Workstations,DC=corp,DC=local"

# Step 4: Configure GPO:
# Computer Config → Admin Templates → LAPS
# "Enable local admin password management" = Enabled
# "Password Settings":
#   Complexity: Large+small+numbers+special
#   Length: 20 characters
#   Age: 30 days

# Step 5: Install LAPS CSE on machines:
msiexec /i LAPS.x64.msi /quiet

# === WINDOWS LAPS (Built-in, Windows 11/Server 2022+) ===
# No separate install needed on newer systems!
# Additional features:
# - Password encryption in AD
# - Password history
# - DSRM password management
# - Azure AD support
```

---

## 4. LAPS from Attacker's Perspective

```bash
# === IF YOU CAN READ LAPS PASSWORDS ===
# You need read access to ms-Mcs-AdmPwd attribute

# PowerView:
Get-DomainComputer -Properties ms-Mcs-AdmPwd, ms-Mcs-AdmPwdExpirationTime
# Shows: password and expiry date

# AD Module:
Get-ADComputer -Filter * -Properties ms-Mcs-AdmPwd | \
  Where-Object {$_.ms-Mcs-AdmPwd -ne $null} | \
  Select-Object Name, ms-Mcs-AdmPwd

# CrackMapExec:
crackmapexec ldap dc01 -u user -p pass -M laps

# From Linux:
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(objectClass=computer)" \
  dNSHostName ms-Mcs-AdmPwd

# === WHO CAN READ LAPS? ===
# Check permissions:
Find-AdmPwdExtendedRights -OrgUnit "OU=Workstations,DC=corp,DC=local"
# Shows which groups/users can read passwords

# BloodHound:
# Shows "ReadLAPSPassword" edges
# Find paths from owned users to LAPS readable computers

# === USE LAPS PASSWORD ===
# Once you have the password:
crackmapexec smb target -u Administrator -p 'LAPS_PASSWORD' \
  --local-auth
# --local-auth: Use local account, not domain

evil-winrm -i target -u Administrator -p 'LAPS_PASSWORD'
```

---

## 5. LAPS Defense Best Practices

| Practice | Description |
|----------|-------------|
| **Principle of least privilege** | Only authorized groups can read passwords |
| **Tier-based access** | Tier 2 admins read workstation LAPS, Tier 1 reads servers |
| **Audit LAPS reads** | Monitor who reads LAPS passwords (Event 4662) |
| **Use Windows LAPS** | Encrypted passwords, history, Azure support |
| **Short rotation** | 14-30 day password rotation |
| **Strong passwords** | 20+ characters, full complexity |
| **Monitor permissions** | Regular audit of who has LAPS read rights |
| **Deploy everywhere** | ALL domain-joined machines should have LAPS |

```powershell
# Audit LAPS password reads:
# Enable advanced auditing on computer objects
# Event ID 4662: Operation on AD object
# Filter for: ms-Mcs-AdmPwd attribute access
# Alert on unexpected users reading LAPS passwords
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **LAPS** | Unique, auto-rotating local admin passwords |
| **Attribute** | ms-Mcs-AdmPwd on computer objects |
| **Prevents** | Lateral movement via shared local admin |
| **Access control** | ACL-based: who can read passwords |
| **Rotation** | Configurable (default 30 days) |
| **Windows LAPS** | Built-in, encrypted, with history |

---

## Quick Revision Questions

1. **What problem does LAPS solve in AD environments?**
2. **Where are LAPS passwords stored?**
3. **How does an attacker check if they can read LAPS passwords?**
4. **What is the difference between legacy LAPS and Windows LAPS?**
5. **How should LAPS read permissions be structured with tiering?**

---

[← Previous: Credential Guard](03-credential-guard.md) | [Next: Privileged Access Workstations →](05-privileged-access-workstations.md)

---

*Unit 9 - Topic 4 of 6 | [Back to README](../README.md)*
