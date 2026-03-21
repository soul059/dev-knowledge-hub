# LDAP Enumeration

## Unit 6 - Topic 2 | Network Protocol Attacks

---

## Overview

**LDAP (Lightweight Directory Access Protocol)** on ports 389/636 provides access to directory services like Active Directory. LDAP enumeration reveals users, groups, computers, organizational units, and security policies — critical intelligence for penetration testing.

---

## 1. LDAP Basics

```
LDAP HIERARCHY (Active Directory):

DC=corp,DC=local          ← Domain root
├── OU=Users              ← Organizational Unit
│   ├── CN=John Smith     ← User object
│   ├── CN=Jane Doe       ← User object
│   └── CN=svc_backup     ← Service account
├── OU=Computers
│   ├── CN=WS01           ← Workstation
│   └── CN=SRV01          ← Server
├── OU=Groups
│   ├── CN=Domain Admins  ← Security group
│   └── CN=IT Staff       ← Security group
└── CN=Administrator      ← Built-in admin

LDAP PORTS:
├── 389:  LDAP (unencrypted)
├── 636:  LDAPS (SSL/TLS encrypted)
├── 3268: Global Catalog
└── 3269: Global Catalog over SSL
```

---

## 2. LDAP Enumeration Tools

```bash
# === LDAPSEARCH (Linux built-in) ===
# Anonymous bind (test if allowed):
ldapsearch -x -H ldap://dc01.corp.local -b "DC=corp,DC=local"

# Authenticated:
ldapsearch -x -H ldap://dc01.corp.local \
  -D "CN=user,DC=corp,DC=local" -w 'Password123' \
  -b "DC=corp,DC=local"

# Enumerate users:
ldapsearch -x -H ldap://dc01.corp.local \
  -D "user@corp.local" -w 'Password123' \
  -b "DC=corp,DC=local" "(objectClass=user)" \
  sAMAccountName description memberOf

# Enumerate groups:
ldapsearch -x -H ldap://dc01.corp.local \
  -D "user@corp.local" -w 'Password123' \
  -b "DC=corp,DC=local" "(objectClass=group)" \
  cn member

# Domain Admins:
ldapsearch -x -H ldap://dc01.corp.local \
  -D "user@corp.local" -w 'Password123' \
  -b "DC=corp,DC=local" "(cn=Domain Admins)" member

# === WINDAPSEARCH (Python) ===
python3 windapsearch.py -d corp.local --dc dc01.corp.local \
  -u 'user@corp.local' -p 'Password123' \
  --users --groups --da

# Enumerate Domain Admins:
windapsearch.py --dc dc01 -d corp.local -u user -p pass --da

# === LDAPDOMAINDUMP ===
ldapdomaindump -u 'corp\user' -p 'Password123' dc01.corp.local
# Creates HTML report with all AD objects
# Outputs: domain_users.html, domain_groups.html, etc.
```

---

## 3. Valuable LDAP Attributes

| Attribute | Contains |
|-----------|----------|
| **sAMAccountName** | Username (login name) |
| **userPrincipalName** | user@domain.com format |
| **memberOf** | Group memberships |
| **description** | Often contains passwords! |
| **adminCount** | 1 = privileged account |
| **servicePrincipalName** | SPN — Kerberoastable |
| **lastLogon** | Last login timestamp |
| **pwdLastSet** | Password age |
| **userAccountControl** | Account flags (disabled, no preauth, etc.) |
| **msDS-AllowedToDelegateTo** | Delegation targets |
| **operatingSystem** | Computer OS version |

```bash
# === FIND PASSWORDS IN DESCRIPTIONS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(description=*)" \
  sAMAccountName description

# === FIND KERBEROASTABLE ACCOUNTS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" \
  "(&(objectClass=user)(servicePrincipalName=*))" \
  sAMAccountName servicePrincipalName

# === FIND ASREP-ROASTABLE ACCOUNTS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" \
  "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" \
  sAMAccountName
```

---

## 4. LDAP Attack Techniques

```
LDAP ATTACK TREE:

UNAUTHENTICATED:
├── Anonymous bind → enumerate everything
├── NULL bind → some DCs allow this
└── LDAP injection (web apps using LDAP)

AUTHENTICATED:
├── Full AD enumeration
├── Find passwords in descriptions
├── Identify Kerberoastable accounts (SPN)
├── Find ASREProastable accounts
├── Map trust relationships
├── Discover delegation configurations
├── Find Group Policy Preferences (GPP)
└── BloodHound data collection (SharpHound)

LDAP INJECTION (Web Applications):
├── Input: *)(uid=*))(|(uid=*
├── Bypass LDAP authentication
├── Extract directory information
└── Similar to SQL injection concept
```

---

## 5. Defense and Hardening

| Defense | Description |
|---------|-------------|
| **Disable anonymous bind** | Require authentication for LDAP |
| **Use LDAPS (636)** | Encrypt LDAP traffic |
| **LDAP signing** | Require signed LDAP requests |
| **Channel binding** | Prevent relay attacks |
| **Audit queries** | Log and monitor LDAP queries |
| **Least privilege** | Limit what authenticated users can read |
| **No passwords in descriptions** | Enforce policy |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **LDAP** | Port 389/636 — directory service protocol |
| **ldapsearch** | Linux command-line LDAP query tool |
| **Description field** | Often contains plaintext passwords |
| **SPN** | Service Principal Name — Kerberoastable |
| **BloodHound** | Visual AD attack path analysis |
| **Defense** | Disable anonymous bind, use LDAPS |

---

## Quick Revision Questions

1. **What ports does LDAP use?**
2. **What is anonymous bind and why is it dangerous?**
3. **What LDAP attribute indicates a Kerberoastable account?**
4. **Why should you check the description field of user objects?**
5. **How does LDAP channel binding improve security?**

---

[← Previous: SMB Attacks](01-smb-attacks.md) | [Next: Kerberos Attacks Overview →](03-kerberos-attacks-overview.md)

---

*Unit 6 - Topic 2 of 6 | [Back to README](../README.md)*
