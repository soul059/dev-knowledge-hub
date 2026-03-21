# Group Policy Basics

## Unit 1 - Topic 4 | Active Directory Fundamentals

---

## Overview

**Group Policy Objects (GPOs)** centrally manage settings and configurations for users and computers in Active Directory. GPOs control security settings, software deployment, scripts, and more — making them both a powerful management tool and an attractive attack target.

---

## 1. How Group Policy Works

```
GROUP POLICY ARCHITECTURE:

┌─────────────┐    SYSVOL Share    ┌─────────────┐
│   Domain     │ ←────────────────│  Group Policy│
│  Controller  │                   │   Objects    │
│              │    LDAP (GPO     │   (GPOs)     │
│  SYSVOL      │    link info)    │              │
│  \\DC\SYSVOL │                   └─────────────┘
└──────┬──────┘
       │ SMB (download GPO files)
       ↓
┌─────────────┐
│  Client PC   │  Applies GPO settings:
│              │  ├── Security policies
│              │  ├── Software deployment
│              │  ├── Startup scripts
│              │  ├── Registry settings
│              │  └── Login scripts
└─────────────┘

GPO PROCESSING ORDER (LSDOU):
1. Local Policy          ← Applied first (lowest priority)
2. Site Policy           ← Geographic location
3. Domain Policy         ← Domain-wide settings
4. OU Policy             ← Most specific (highest priority)
   └── Nested OU Policy  ← Most specific wins!

REFRESH INTERVAL:
├── Computer: Every 90 minutes (± 30 min random)
├── Domain Controller: Every 5 minutes
├── Force refresh: gpupdate /force
└── Check applied: gpresult /r
```

---

## 2. GPO Security Settings

```
SECURITY-RELEVANT GPO SETTINGS:

ACCOUNT POLICIES:
├── Password Policy
│   ├── Minimum length
│   ├── Complexity requirements
│   ├── Maximum age
│   └── History (prevent reuse)
├── Account Lockout
│   ├── Threshold (attempts before lockout)
│   ├── Duration (how long locked)
│   └── Observation window
└── Kerberos Policy
    ├── Ticket lifetime
    └── Renewal settings

LOCAL POLICIES:
├── Audit Policy (what to log)
├── User Rights Assignment
│   ├── SeDebugPrivilege (attach debugger)
│   ├── SeImpersonatePrivilege (token impersonation)
│   ├── SeBackupPrivilege (read any file)
│   └── SeRemoteShutdownPrivilege
└── Security Options
    ├── SMB signing requirements
    ├── NTLM authentication settings
    └── Admin approval mode

RESTRICTED GROUPS:
├── Control group memberships
├── Add/remove from local Administrators
└── Enforced on every refresh
```

---

## 3. GPO Enumeration

```bash
# === POWERVIEW ===
Get-DomainGPO | select displayname, gpcfilesyspath
Get-DomainGPO -ComputerIdentity ws-001
Get-DomainOU | Get-DomainGPO

# === GROUPER (GPO Auditing) ===
# Finds interesting GPO settings:
# Passwords in GPP, startup scripts, etc.

# === NET COMMANDS ===
gpresult /r                    # Applied GPOs
gpresult /r /scope:computer    # Computer GPOs only

# === SYSVOL ENUMERATION ===
# GPO files stored in SYSVOL:
dir \\dc01\SYSVOL\corp.local\Policies\

# Each GPO folder:
# {GUID}\Machine\      ← Computer settings
# {GUID}\User\         ← User settings
# {GUID}\Machine\Preferences\  ← Group Policy Preferences

# === FIND PASSWORDS IN GPP ===
# Group Policy Preferences can contain encrypted passwords!
# AES key was published by Microsoft → can be decrypted!

# Search SYSVOL for cpassword:
findstr /S /I cpassword \\dc01\SYSVOL\corp.local\Policies\*.xml

# Decrypt with gpp-decrypt:
gpp-decrypt "edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ"
```

---

## 4. GPO Attack Vectors

```
GPO ATTACK SCENARIOS:

1. GPP PASSWORD DISCLOSURE (MS14-025):
   ├── Group Policy Preferences stored encrypted passwords
   ├── Microsoft published the AES key (!)
   ├── Any domain user can read SYSVOL
   ├── Decrypt with gpp-decrypt or Metasploit
   └── Often finds local admin passwords

2. GPO ABUSE (If you have write access):
   ├── Modify GPO → add scheduled task
   ├── Add startup script → runs as SYSTEM
   ├── Create local admin account
   ├── Disable security settings
   └── Deploy malicious software

3. GPO LINK MANIPULATION:
   ├── Link GPO to sensitive OUs
   ├── Affect all computers/users in that OU
   └── Requires specific AD permissions

4. STARTUP SCRIPT ABUSE:
   ├── Scripts in SYSVOL run at boot/logon
   ├── If writable → replace with malicious script
   ├── Runs as SYSTEM (machine) or user
   └── Affects all computers linked to GPO
```

---

## 5. GPO Security Recommendations

| Defense | Implementation |
|---------|---------------|
| **Remove GPP passwords** | Delete all `cpassword` in SYSVOL |
| **Audit GPO permissions** | Review who can modify GPOs |
| **Monitor GPO changes** | Alert on GPO modifications |
| **Restrict SYSVOL write** | Only admins can modify SYSVOL |
| **Use LAPS** | Instead of GPP for local admin passwords |
| **Block inheritance** | Where appropriate |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **GPO** | Centralized configuration management |
| **SYSVOL** | Network share containing GPO files |
| **LSDOU** | Processing order: Local→Site→Domain→OU |
| **GPP passwords** | Encrypted but key is public (MS14-025) |
| **GPO abuse** | Write access = deploy malware via GPO |
| **Defense** | Audit permissions, monitor changes |

---

## Quick Revision Questions

1. **What is the GPO processing order?**
2. **Where are GPO files physically stored?**
3. **Why is the GPP password vulnerability significant?**
4. **How can GPO write access lead to domain compromise?**
5. **How often do clients refresh Group Policy?**

---

[← Previous: Objects](03-objects-users-groups-computers.md) | [Next: Authentication Protocols →](05-authentication-protocols.md)

---

*Unit 1 - Topic 4 of 5 | [Back to README](../README.md)*
