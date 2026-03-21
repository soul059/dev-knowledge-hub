# User and Group Enumeration

## Unit 2 - Topic 5 | AD Enumeration

---

## Overview

**User and group enumeration** identifies all accounts, their memberships, privileges, and security attributes. Finding high-privilege accounts, service accounts with SPNs, and nested group memberships reveals the attack surface for credential attacks and privilege escalation.

---

## 1. User Enumeration Techniques

```bash
# === HIGH-VALUE USER DISCOVERY ===

# FROM LINUX:
# All users:
crackmapexec smb dc01 -u user -p pass --users

# Impacket:
GetADUsers.py corp.local/user:pass -dc-ip dc01 -all

# === FROM WINDOWS (PowerView) ===
# All users with key properties:
Get-DomainUser | select samaccountname, description, \
  memberof, pwdlastset, lastlogon, admincount

# Privileged users (adminCount=1):
Get-DomainUser -AdminCount | select samaccountname, memberof

# Service accounts (have SPN):
Get-DomainUser -SPN | select samaccountname, serviceprincipalname

# ASREProastable:
Get-DomainUser -PreAuthNotRequired | select samaccountname

# Passwords in descriptions:
Get-DomainUser -Properties name,description | \
  Where-Object {$_.description -match "pass|pwd|cred"}

# Accounts with password never expires:
Get-DomainUser -Properties name,useraccountcontrol | \
  Where-Object {$_.useraccountcontrol -band 65536}

# Recently created accounts (potential backdoor):
$date = (Get-Date).AddDays(-30)
Get-ADUser -Filter {WhenCreated -gt $date} -Properties WhenCreated
```

---

## 2. Group Enumeration

```bash
# === HIGH-VALUE GROUPS ===
# PowerView:
Get-DomainGroupMember "Domain Admins" -Recurse
Get-DomainGroupMember "Enterprise Admins" -Recurse
Get-DomainGroupMember "Schema Admins"
Get-DomainGroupMember "Administrators"
Get-DomainGroupMember "Account Operators"
Get-DomainGroupMember "Server Operators"
Get-DomainGroupMember "Backup Operators"
Get-DomainGroupMember "DNS Admins"

# All groups:
Get-DomainGroup | select name, groupscope, description

# Custom admin groups:
Get-DomainGroup -Properties name | \
  Where-Object {$_.name -match "admin|operator|manager"}

# === NESTED GROUP MEMBERSHIPS ===
# User → Group A → Group B → Domain Admins
# The user IS effectively a Domain Admin!

# Find nested membership:
Get-DomainGroupMember "Domain Admins" -Recurse
# -Recurse: Follows nested groups

# FROM LINUX:
crackmapexec smb dc01 -u user -p pass --groups

# Impacket:
GetADUsers.py corp.local/user:pass -dc-ip dc01 -all
```

---

## 3. Critical Account Types

```
ACCOUNTS TO FIND AND TARGET:

DOMAIN ADMINS:
├── Full control over the entire domain
├── Target: Steal credentials, compromise sessions
└── Query: Get-DomainGroupMember "Domain Admins"

SERVICE ACCOUNTS:
├── Often have SPNs → Kerberoastable
├── Often have weak passwords (set once)
├── Often have excessive privileges
├── Query: Get-DomainUser -SPN
└── Example: svc_sql, svc_backup, svc_web

ASREP-ROASTABLE:
├── Don't require Kerberos pre-authentication
├── Can get hash without knowing password
├── Query: Get-DomainUser -PreAuthNotRequired
└── Crack with: hashcat -m 18200

COMPUTER ACCOUNTS:
├── Machine accounts (end with $)
├── Can be used for lateral movement
├── Some have unconstrained delegation
└── Query: Get-DomainComputer -Unconstrained

KRBTGT:
├── Kerberos service account
├── Hash used to encrypt all TGTs
├── If compromised → Golden Ticket!
├── Should NEVER be directly accessible
└── Password should be changed regularly
```

---

## 4. ACL-Based Privilege Paths

```powershell
# === DANGEROUS ACL PERMISSIONS ===
# Find objects where users have dangerous rights:

# GenericAll on a group (can add yourself as member):
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs | \
  Where-Object {$_.ActiveDirectoryRights -match "GenericAll"} | \
  select SecurityIdentifier, ActiveDirectoryRights

# ForceChangePassword (can reset user's password):
Get-DomainObjectAcl -ResolveGUIDs | \
  Where-Object {$_.ObjectAceType -match "User-Force-Change-Password"} | \
  select ObjectDN, SecurityIdentifier

# WriteDacl (can modify permissions on object):
Get-DomainObjectAcl -ResolveGUIDs | \
  Where-Object {$_.ActiveDirectoryRights -match "WriteDacl"} | \
  select ObjectDN, SecurityIdentifier

# === ACL ABUSE SCENARIO ===
# User "jsmith" has GenericAll on "Domain Admins" group
# → jsmith can add himself to Domain Admins!
Add-DomainGroupMember -Identity "Domain Admins" -Members "jsmith"
# Instant Domain Admin!
```

---

## 5. Enumeration Automation

```bash
# === AUTOMATED ENUMERATION SCRIPTS ===

# ADRecon (PowerShell):
# Generates comprehensive AD report:
.\ADRecon.ps1
# Output: CSV, Excel, HTML reports

# PingCastle:
# AD security assessment:
PingCastle.exe --healthcheck --server dc01.corp.local
# Generates risk score and recommendations

# BloodHound + SharpHound:
.\SharpHound.exe -c all
# Import to BloodHound for visual analysis

# Plumhound (BloodHound report generator):
python3 PlumHound.py -x tasks/default.tasks \
  -s "bolt://localhost:7687" -u neo4j -p password
# Generates actionable reports from BloodHound data
```

---

## Summary Table

| Target | Query | Priority |
|--------|-------|:--------:|
| **Domain Admins** | `Get-DomainGroupMember "DA"` | 🔴 Critical |
| **Kerberoastable** | `Get-DomainUser -SPN` | 🔴 Critical |
| **ASREProastable** | `Get-DomainUser -PreAuthNotRequired` | 🔴 High |
| **Descriptions** | Filter for "pass" keyword | 🔴 High |
| **Nested groups** | `-Recurse` flag | 🟡 Medium |
| **ACL abuse** | `Find-InterestingDomainAcl` | 🟡 Medium |

---

## Quick Revision Questions

1. **Why are service accounts high-priority targets?**
2. **How do nested group memberships create hidden admin paths?**
3. **What is GenericAll and why is it dangerous?**
4. **How do you find passwords stored in user descriptions?**
5. **What automated tools generate AD security reports?**

---

[← Previous: BloodHound Basics](04-bloodhound-basics.md) | [Next: Trust Enumeration →](06-trust-enumeration.md)

---

*Unit 2 - Topic 5 of 6 | [Back to README](../README.md)*
