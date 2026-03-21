# LDAP/Active Directory

## Unit 6 - Topic 4 | Network Services Security

---

## Overview

**LDAP (Lightweight Directory Access Protocol)** is used to access and manage directory services like **Active Directory (AD)**. AD is the backbone of identity management in most organizations. LDAP and AD are prime targets for attackers seeking **credential theft**, **privilege escalation**, and **lateral movement**.

---

## 1. LDAP Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                  LDAP / ACTIVE DIRECTORY                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LDAP (Port 389) — Directory query protocol                     │
│  LDAPS (Port 636) — LDAP over TLS (encrypted)                  │
│  Global Catalog (Port 3268/3269)                                │
│                                                                  │
│  Active Directory = Microsoft's directory service               │
│  Uses LDAP as the query protocol                                │
│  Uses Kerberos for authentication                               │
│                                                                  │
│  AD STORES:                                                      │
│  • User accounts and passwords                                  │
│  • Computer accounts                                            │
│  • Group memberships                                            │
│  • Group Policies (GPOs)                                        │
│  • DNS records                                                  │
│  • Security permissions                                         │
│                                                                  │
│  COMPROMISE AD = COMPROMISE THE ENTIRE ORGANIZATION             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. LDAP Security Concerns

| Concern | Description | Defense |
|---------|-------------|---------|
| **Cleartext LDAP** | Port 389 sends data unencrypted | Use LDAPS (636) or STARTTLS |
| **Anonymous Bind** | Query without authentication | Disable anonymous access |
| **LDAP Injection** | Manipulate LDAP queries | Input validation |
| **Credential Harvesting** | Enumerate users via LDAP queries | Restrict query permissions |
| **Password Spray** | Try common passwords across many accounts | Lockout policies, monitoring |
| **Kerberoasting** | Extract service ticket hashes for offline cracking | Strong service account passwords |

---

## 3. Active Directory Attacks

| Attack | Description |
|--------|-------------|
| **Kerberoasting** | Request service tickets, crack offline → service account passwords |
| **AS-REP Roasting** | Target accounts without pre-auth → crack password hashes |
| **Pass-the-Hash** | Use captured NTLM hash without cracking |
| **Pass-the-Ticket** | Use stolen Kerberos tickets |
| **Golden Ticket** | Forge Kerberos tickets with stolen KRBTGT hash |
| **DCSync** | Replicate domain controller to extract all password hashes |
| **LLMNR/NBT-NS Poisoning** | Capture credentials from broadcast name resolution |

---

## 4. LDAP Enumeration

```bash
# Anonymous LDAP enumeration
ldapsearch -x -H ldap://target.com -b "DC=corp,DC=com"

# Authenticated enumeration
ldapsearch -x -H ldap://target.com -D "user@corp.com" -w password \
  -b "DC=corp,DC=com" "(objectClass=user)" cn mail

# Nmap LDAP scripts
nmap -p 389 --script ldap-rootdse target.com
nmap -p 389 --script ldap-search target.com

# Enumerate users, groups, computers
# Tools: ldapdomaindump, BloodHound, PowerView
```

---

## 5. AD/LDAP Security Best Practices

| Practice | Description |
|----------|-------------|
| **Use LDAPS** | Encrypt all LDAP traffic (port 636) |
| **Disable anonymous** | Require authentication for all queries |
| **Tier Model** | Separate admin tiers (T0=DC, T1=Servers, T2=Workstations) |
| **Least Privilege** | Minimize admin group membership |
| **Service Account Security** | Long, complex passwords for service accounts |
| **LAPS** | Local Administrator Password Solution (unique per machine) |
| **Monitor** | Watch for Kerberoasting, DCSync, unusual replication |
| **Protected Users Group** | Enhanced security for privileged accounts |
| **Audit** | Log all authentication events and group changes |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **LDAP** | Directory query protocol — port 389 (cleartext) |
| **LDAPS** | LDAP over TLS — port 636 (encrypted) |
| **Active Directory** | Microsoft directory — identity backbone |
| **Top Threats** | Kerberoasting, Pass-the-Hash, DCSync, Golden Ticket |
| **Key Defense** | LDAPS, disable anonymous, tier model, monitoring |

---

## Quick Revision Questions

1. **What is LDAP and what port does it use?**
2. **Why should LDAPS be used instead of LDAP?**
3. **What is Kerberoasting and how is it performed?**
4. **Why is Active Directory such a high-value target?**
5. **What is the AD administrative tier model?**

---

[← Previous: NTP Security](03-ntp-security.md) | [Next: Certificate Services →](05-certificate-services.md)

---

*Unit 6 - Topic 4 of 5 | [Back to README](../README.md)*
