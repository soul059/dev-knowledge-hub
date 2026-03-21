# Objects: Users, Groups, and Computers

## Unit 1 - Topic 3 | Active Directory Fundamentals

---

## Overview

**AD objects** represent network resources — users, groups, computers, and more. Each object has attributes that define its properties and security settings. Understanding object types, attributes, and relationships is essential for AD enumeration and exploitation.

---

## 1. User Objects

```
USER OBJECT ATTRIBUTES:

IDENTITY:
├── sAMAccountName    → jsmith (login name)
├── userPrincipalName → jsmith@corp.local
├── distinguishedName → CN=John Smith,OU=Users,DC=corp,DC=local
├── objectSID         → S-1-5-21-xxxx-xxxx-xxxx-1104
└── objectGUID        → Unique identifier (never changes)

SECURITY:
├── userAccountControl → Account flags (enabled, locked, etc.)
├── pwdLastSet         → When password was last changed
├── lastLogon          → Last interactive logon
├── logonCount         → Number of logons
├── badPwdCount        → Failed password attempts
├── adminCount         → 1 if privileged account
├── memberOf           → Group memberships
└── servicePrincipalName → SPN (Kerberoastable if set!)

INTERESTING FLAGS (userAccountControl):
├── 0x0002 — ACCOUNTDISABLE
├── 0x0010 — LOCKOUT
├── 0x0020 — PASSWD_NOTREQD (password not required!)
├── 0x0080 — ENCRYPTED_TEXT_PWD_ALLOWED
├── 0x10000 — DONT_EXPIRE_PASSWORD
├── 0x400000 — DONT_REQ_PREAUTH (ASREProastable!)
└── 0x1000000 — TRUSTED_TO_AUTH_FOR_DELEGATION
```

---

## 2. Group Objects

```
GROUP TYPES:

SECURITY GROUPS (Used for access control):
├── Domain Admins        — Full control of domain
├── Enterprise Admins    — Full control of forest
├── Schema Admins        — Can modify AD schema
├── Administrators       — Built-in admin group
├── Server Operators     — Manage domain servers
├── Account Operators    — Manage user accounts
├── Backup Operators     — Can backup/restore DC
├── Print Operators      — Manage printers
├── DNS Admins           — Manage DNS (can get DA!)
└── Custom groups        — Organization-specific

GROUP SCOPE:
├── Domain Local  — Access within single domain
├── Global        — Members from one domain, access anywhere
└── Universal     — Members from any domain, access anywhere

HIGH-VALUE GROUPS (Targets for escalation):
├── Domain Admins             — S-1-5-21-domain-512
├── Enterprise Admins         — S-1-5-21-root domain-519
├── Schema Admins             — S-1-5-21-root domain-518
├── Administrators            — S-1-5-32-544
├── Account Operators         — S-1-5-32-548
├── Server Operators          — S-1-5-32-549
├── Backup Operators          — S-1-5-32-551
├── DNS Admins                — Custom SID
├── Group Policy Creator Owners — S-1-5-21-domain-520
└── Protected Users           — S-1-5-21-domain-525
```

---

## 3. Computer Objects

```
COMPUTER OBJECTS:

TYPES:
├── Workstations — User endpoints (WS-001$)
├── Servers — Application/file servers (SRV-WEB01$)
├── Domain Controllers — AD servers (DC01$)
└── Note: Computer names end with $ in AD

ATTRIBUTES:
├── dNSHostName        → ws-001.corp.local
├── operatingSystem    → Windows 10 Enterprise
├── operatingSystemVersion → 10.0 (19045)
├── lastLogonTimestamp → Last domain authentication
├── ms-Mcs-AdmPwd     → LAPS password (if configured)
└── servicePrincipalName → SPNs for the computer

MACHINE ACCOUNT:
├── Every domain computer has a machine account
├── Password: 120 random chars, auto-rotated every 30 days
├── Used for domain authentication
├── Can be exploited for lateral movement
└── Machine accounts have their own NTLM hash
```

---

## 4. Service Accounts

```
SERVICE ACCOUNT TYPES:

STANDARD USER ACCOUNTS (Used as service accounts):
├── Created by admins for applications
├── Example: svc_sql, svc_backup, svc_web
├── Often have SPN set → Kerberoastable!
├── Often have weak passwords (set once, never changed)
├── Often have excessive privileges
└── TOP TARGET for attackers!

MANAGED SERVICE ACCOUNTS (MSA):
├── Automatic password management
├── Cannot be used for interactive logon
├── Single computer use
└── More secure than standard accounts

GROUP MANAGED SERVICE ACCOUNTS (gMSA):
├── Like MSA but for multiple computers
├── 240-character auto-generated password
├── Rotated automatically
├── Very difficult to attack
└── Recommended by Microsoft

BUILT-IN SERVICE ACCOUNTS:
├── LOCAL SERVICE — Limited local privileges
├── NETWORK SERVICE — Network access with machine identity
├── LOCAL SYSTEM — Highest local privileges
└── krbtgt — Kerberos ticket granting service
```

---

## 5. Enumeration Commands

```bash
# === ENUMERATE USERS ===
# PowerView:
Get-DomainUser | select samaccountname, description, memberof
Get-DomainUser -AdminCount  # Privileged accounts

# LDAP:
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(objectClass=user)" sAMAccountName description

# === ENUMERATE GROUPS ===
Get-DomainGroup | select name
Get-DomainGroupMember "Domain Admins"

# === ENUMERATE COMPUTERS ===
Get-DomainComputer | select dnshostname, operatingsystem
Get-DomainComputer -Unconstrained  # Unconstrained delegation
```

---

## Summary Table

| Object | Key Attribute | Attacker Interest |
|--------|-------------|-------------------|
| **Users** | sAMAccountName, adminCount | Credentials, privilege |
| **Groups** | Domain Admins, Enterprise Admins | Escalation targets |
| **Computers** | operatingSystem, LAPS | Lateral movement |
| **Service Accounts** | SPN, password age | Kerberoasting |

---

## Quick Revision Questions

1. **What userAccountControl flag makes an account ASREProastable?**
2. **Why are service accounts high-value targets?**
3. **What is the difference between a gMSA and a standard service account?**
4. **What high-value groups should you enumerate first?**
5. **How do computer accounts authenticate to the domain?**

---

[← Previous: Domains, Forests, Trusts](02-domains-forests-trusts.md) | [Next: Group Policy Basics →](04-group-policy-basics.md)

---

*Unit 1 - Topic 3 of 5 | [Back to README](../README.md)*
