# LDAP Queries for AD Enumeration

## Unit 2 - Topic 2 | AD Enumeration

---

## Overview

**LDAP (Lightweight Directory Access Protocol)** is the primary protocol for querying Active Directory. Mastering LDAP query syntax enables precise, targeted enumeration of users, groups, computers, and security-relevant attributes.

---

## 1. LDAP Query Syntax

```
LDAP FILTER SYNTAX:

BASIC OPERATORS:
├── (attribute=value)        — Equality
├── (attribute>=value)       — Greater than or equal
├── (attribute<=value)       — Less than or equal
├── (attribute=*value*)      — Substring (contains)
├── (attribute=value*)       — Starts with
├── (attribute=*)            — Presence (has attribute)
├── (!(attribute=value))     — NOT
├── (&(filter1)(filter2))    — AND
└── (|(filter1)(filter2))    — OR

EXAMPLES:
├── (sAMAccountName=admin)           — Find user "admin"
├── (&(objectClass=user)(adminCount=1)) — Privileged users
├── (|(name=*admin*)(name=*service*))   — Admin or service accounts
├── (&(objectClass=user)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))
│   → Enabled users only
└── (&(objectClass=user)(servicePrincipalName=*))
    → Users with SPNs (Kerberoastable)
```

---

## 2. Essential LDAP Queries

```bash
# === LDAPSEARCH (Linux) ===
# Base syntax:
ldapsearch -x -H ldap://DC_IP -D "user@corp.local" -w 'password' \
  -b "DC=corp,DC=local" "FILTER" ATTRIBUTES

# === DOMAIN INFORMATION ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" -s base "(objectClass=*)" \
  defaultNamingContext dnsHostName

# === ALL USERS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(objectClass=user)" \
  sAMAccountName userPrincipalName memberOf description

# === DOMAIN ADMINS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(cn=Domain Admins)" member

# === PRIVILEGED ACCOUNTS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(&(objectClass=user)(adminCount=1))" \
  sAMAccountName description memberOf

# === KERBEROASTABLE (SPNs) ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" \
  "(&(objectClass=user)(servicePrincipalName=*)(!(cn=krbtgt)))" \
  sAMAccountName servicePrincipalName

# === ASREP-ROASTABLE ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" \
  "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" \
  sAMAccountName

# === COMPUTERS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(objectClass=computer)" \
  dNSHostName operatingSystem operatingSystemVersion

# === FIND PASSWORDS IN DESCRIPTIONS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(&(objectClass=user)(description=*pass*))" \
  sAMAccountName description
```

---

## 3. Advanced LDAP Queries

```bash
# === UNCONSTRAINED DELEGATION ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" \
  "(&(objectClass=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" \
  dNSHostName

# === CONSTRAINED DELEGATION ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" \
  "(msDS-AllowedToDelegateTo=*)" \
  sAMAccountName msDS-AllowedToDelegateTo

# === DISABLED ACCOUNTS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" \
  "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2))" \
  sAMAccountName

# === PASSWORD NEVER EXPIRES ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" \
  "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=65536))" \
  sAMAccountName pwdLastSet

# === LAPS PASSWORDS (If readable) ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(objectClass=computer)" \
  dNSHostName ms-Mcs-AdmPwd ms-Mcs-AdmPwdExpirationTime

# === TRUST OBJECTS ===
ldapsearch -x -H ldap://dc01 -D "user@corp.local" -w pass \
  -b "DC=corp,DC=local" "(objectClass=trustedDomain)" \
  cn trustDirection trustType
```

---

## 4. LDAP Tools Comparison

| Tool | Platform | Feature |
|------|----------|---------|
| **ldapsearch** | Linux | Standard CLI LDAP client |
| **windapsearch** | Python | AD-specific LDAP queries |
| **ldapdomaindump** | Python | HTML report output |
| **ADFind** | Windows | Fast command-line queries |
| **PowerView** | PowerShell | Full AD enumeration |
| **dsquery** | Windows | Built-in AD query tool |

```bash
# === WINDAPSEARCH ===
python3 windapsearch.py -d corp.local --dc dc01 \
  -u 'user@corp.local' -p pass --da     # Domain Admins
python3 windapsearch.py --dc dc01 -d corp.local \
  -u user -p pass --users --full         # All user details

# === LDAPDOMAINDUMP ===
ldapdomaindump -u 'corp\user' -p pass dc01
# Creates: domain_users.html, domain_groups.html, etc.
```

---

## 5. LDAP Enumeration Checklist

| Query | Purpose | Priority |
|-------|---------|:--------:|
| Domain Admins members | Identify targets | 🔴 High |
| Users with SPNs | Kerberoasting candidates | 🔴 High |
| Users with description | Find passwords | 🔴 High |
| ASREProastable users | No-preauth hash capture | 🔴 High |
| Unconstrained delegation | TGT theft targets | 🟡 Medium |
| Constrained delegation | Delegation abuse | 🟡 Medium |
| LAPS passwords | Local admin access | 🟡 Medium |
| All computers + OS | Identify old/vulnerable | 🟢 Normal |
| Trust relationships | Cross-domain attacks | 🟢 Normal |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **LDAP filters** | AND (&), OR (|), NOT (!) operators |
| **adminCount=1** | Privileged accounts |
| **SPN presence** | Kerberoastable accounts |
| **4194304 flag** | DONT_REQ_PREAUTH (ASREProast) |
| **ldapdomaindump** | Creates readable HTML reports |
| **Descriptions** | Often contain plaintext passwords |

---

## Quick Revision Questions

1. **How do you write an LDAP filter for Kerberoastable accounts?**
2. **What does the adminCount attribute indicate?**
3. **How do you find ASREProastable accounts via LDAP?**
4. **What is the bitwise match syntax for userAccountControl?**
5. **What tools generate HTML reports from LDAP data?**

---

[← Previous: AD Enumeration Techniques](01-ad-enumeration-techniques.md) | [Next: PowerShell Enumeration →](03-powershell-enumeration.md)

---

*Unit 2 - Topic 2 of 6 | [Back to README](../README.md)*
