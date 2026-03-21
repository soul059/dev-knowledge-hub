# AD Architecture

## Unit 1 - Topic 1 | Active Directory Fundamentals

---

## Overview

**Active Directory (AD)** is Microsoft's directory service for Windows networks. It centralizes authentication, authorization, and management for all network resources — users, computers, groups, and policies. Understanding AD architecture is the foundation for both attacking and defending enterprise environments.

---

## 1. Active Directory Components

```
ACTIVE DIRECTORY ARCHITECTURE:

┌─────────────────────────────────────────────────┐
│                    FOREST                        │
│  (Security boundary — highest level)            │
│                                                  │
│  ┌──────────────────┐  ┌──────────────────┐     │
│  │    DOMAIN 1       │  │    DOMAIN 2       │     │
│  │  corp.local       │  │  dev.corp.local   │     │
│  │                   │  │                   │     │
│  │  ┌─────────────┐ │  │  ┌─────────────┐ │     │
│  │  │     DC1      │ │  │  │     DC3      │ │     │
│  │  │ (Domain      │ │  │  │ (Domain      │ │     │
│  │  │  Controller) │ │  │  │  Controller) │ │     │
│  │  └─────────────┘ │  │  └─────────────┘ │     │
│  │  ┌─────────────┐ │  │                   │     │
│  │  │     DC2      │ │  │                   │     │
│  │  │ (Backup DC)  │ │  │                   │     │
│  │  └─────────────┘ │  │                   │     │
│  └──────────────────┘  └──────────────────┘     │
│           ↕ Trust                                │
│  ┌──────────────────┐                            │
│  │    DOMAIN 3       │                            │
│  │  dmz.corp.local   │                            │
│  └──────────────────┘                            │
└─────────────────────────────────────────────────┘

KEY COMPONENTS:
├── Forest    — Collection of domains sharing schema
├── Domain    — Administrative boundary
├── Domain Controller (DC) — Server running AD DS
├── OU (Organizational Unit) — Container for objects
├── Site      — Physical network location
├── Schema    — Object class definitions
└── Global Catalog — Forest-wide search service
```

---

## 2. Domain Controller (DC) Services

| Service | Port | Description |
|---------|:----:|-------------|
| **LDAP** | 389 | Directory queries |
| **LDAPS** | 636 | Encrypted LDAP |
| **Kerberos** | 88 | Authentication |
| **DNS** | 53 | Name resolution |
| **SMB** | 445 | File sharing, group policy |
| **RPC** | 135 | Remote procedure calls |
| **Global Catalog** | 3268 | Forest-wide LDAP |
| **GC SSL** | 3269 | Encrypted Global Catalog |
| **WinRM** | 5985 | Remote management |

```
DOMAIN CONTROLLER ROLES:

FSMO ROLES (Flexible Single Master Operations):
├── Forest-Level (one per forest):
│   ├── Schema Master — controls schema changes
│   └── Domain Naming Master — add/remove domains
│
└── Domain-Level (one per domain):
    ├── PDC Emulator — time sync, password changes, GPO
    ├── RID Master — allocates RID pools for SIDs
    └── Infrastructure Master — cross-domain references
```

---

## 3. AD Database (NTDS.dit)

```
NTDS.DIT — The Crown Jewels:

Location: C:\Windows\NTDS\ntds.dit

CONTAINS:
├── All user accounts and password hashes
├── All computer accounts
├── All group memberships
├── All security policies
├── All GPO information
├── Schema information
├── Trust relationships
└── DNS records (AD-integrated)

SIZE: Typically 100MB - 10GB+ depending on domain size

WHY ATTACKERS WANT IT:
├── Contains EVERY user's NTLM password hash
├── Contains the krbtgt hash (Golden Ticket!)
├── Contains domain trust keys
├── Offline access = crack at leisure
└── One file = complete domain compromise!

HOW TO EXTRACT:
├── secretsdump.py (Impacket — remote DCSync)
├── ntdsutil (from Domain Controller)
├── Volume Shadow Copy (VSS)
├── Mimikatz DCSync
└── Physical disk access
```

---

## 4. AD Object Hierarchy

```
ACTIVE DIRECTORY HIERARCHY:

DC=corp,DC=local                    ← Domain root
├── OU=Corporate                     ← Organizational Unit
│   ├── OU=Users
│   │   ├── CN=John Smith           ← User object
│   │   ├── CN=Jane Doe             ← User object
│   │   └── CN=svc_backup           ← Service account
│   ├── OU=Computers
│   │   ├── CN=WS-001               ← Workstation
│   │   └── CN=LAPTOP-042           ← Laptop
│   ├── OU=Servers
│   │   ├── CN=SRV-WEB01            ← Web server
│   │   └── CN=SRV-DB01             ← Database server
│   └── OU=Groups
│       ├── CN=IT-Admins             ← Security group
│       └── CN=HR-Users              ← Security group
├── CN=Users                         ← Built-in container
│   ├── CN=Administrator             ← Default admin
│   ├── CN=Guest                     ← Default guest
│   └── CN=krbtgt                    ← Kerberos service account
├── CN=Computers                     ← Default computer container
└── CN=Builtin                       ← Built-in groups
    ├── CN=Administrators
    ├── CN=Domain Admins
    ├── CN=Enterprise Admins
    └── CN=Schema Admins
```

---

## 5. Why AD Is a Primary Target

| Reason | Impact |
|--------|--------|
| **Centralized auth** | One compromise = access to everything |
| **Ubiquitous** | 95%+ of enterprises use AD |
| **Legacy support** | Old protocols enabled by default |
| **Complex** | Misconfigurations are common |
| **Trust** | Trusts create attack paths |
| **Credentials** | Hashes stored on DCs, cached on workstations |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Forest** | Security boundary — collection of domains |
| **Domain** | Administrative boundary — manages resources |
| **DC** | Server running AD DS — stores NTDS.dit |
| **NTDS.dit** | AD database — all hashes and objects |
| **FSMO** | Special DC roles (Schema, PDC, RID, etc.) |
| **Why target** | Central auth = one-stop-shop for attackers |

---

## Quick Revision Questions

1. **What is the difference between a forest and a domain?**
2. **What does NTDS.dit contain and why is it valuable?**
3. **What are the five FSMO roles?**
4. **What ports does a Domain Controller use?**
5. **Why is Active Directory the primary target for attackers?**

---

[Next: Domains, Forests, Trusts →](02-domains-forests-trusts.md)

---

*Unit 1 - Topic 1 of 5 | [Back to README](../README.md)*
