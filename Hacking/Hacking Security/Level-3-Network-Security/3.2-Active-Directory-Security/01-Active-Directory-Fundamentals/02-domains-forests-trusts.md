# Domains, Forests, and Trusts

## Unit 1 - Topic 2 | Active Directory Fundamentals

---

## Overview

**Domains, forests, and trusts** form the structural framework of Active Directory. Understanding these concepts is essential for mapping attack paths, exploiting trust relationships, and understanding the boundaries of compromise.

---

## 1. Domain Structure

```
DOMAIN TYPES:

ROOT DOMAIN:
├── First domain created in a forest
├── Contains Enterprise Admins and Schema Admins
├── Example: corp.local
└── Forest root domain = most privileged

CHILD DOMAIN:
├── Created under an existing domain
├── Example: dev.corp.local (child of corp.local)
├── Automatic two-way trust with parent
└── Shares same forest schema

TREE:
├── Hierarchy of domains sharing a contiguous namespace
├── corp.local → dev.corp.local → test.dev.corp.local
├── Parent-child trust relationships
└── All in same forest

DOMAIN NAMING:
├── NetBIOS: CORP
├── DNS: corp.local
├── Distinguished Name: DC=corp,DC=local
└── SID: S-1-5-21-xxxxxxxxxx-xxxxxxxxxx-xxxxxxxxxx
```

---

## 2. Forest Architecture

```
FOREST STRUCTURE:

FOREST: corp.local
├── TREE 1: corp.local
│   ├── corp.local (root domain)
│   ├── dev.corp.local (child)
│   └── dmz.corp.local (child)
│
└── TREE 2: partner.local
    ├── partner.local (tree root)
    └── sales.partner.local (child)

WHAT A FOREST SHARES:
├── Schema — same object definitions
├── Configuration — same forest topology
├── Global Catalog — forest-wide search
├── Enterprise Admins — forest-wide admin group
└── Schema Admins — can modify schema

WHAT A FOREST DOES NOT SHARE:
├── Domain Admins — per-domain only
├── User accounts — per-domain
├── Group policies — per-domain
├── Security policies — per-domain
└── Password policies — per-domain
```

---

## 3. Trust Relationships

```
TRUST TYPES:

PARENT-CHILD (Automatic):
corp.local ←→ dev.corp.local
├── Two-way transitive
├── Created automatically
└── Cannot be removed

TREE-ROOT (Automatic):
corp.local ←→ partner.local
├── Two-way transitive
├── Between tree roots in same forest
└── Created automatically

EXTERNAL (Manual):
corp.local → external.com
├── One-way or two-way
├── Non-transitive
├── Between domains in different forests
└── Created manually

FOREST (Manual):
Forest A ←→ Forest B
├── One-way or two-way
├── Transitive between all domains in both forests
├── Created manually
└── Enables cross-forest access

TRUST DIRECTION:
┌─────────┐    Trusts    ┌─────────┐
│ Domain A │ ───────────→ │ Domain B │
│(Trusting)│              │(Trusted) │
└─────────┘              └─────────┘
Users in B can access resources in A
"A trusts B" = B's users can access A's resources
```

---

## 4. Trust Enumeration

```bash
# === ENUMERATE TRUSTS ===
# PowerShell:
Get-ADTrust -Filter *
([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetAllTrustRelationships()

# PowerView:
Get-DomainTrust
Get-ForestTrust
Get-DomainTrustMapping   # Map all trusts

# Impacket:
python3 getTGT.py corp.local/user:pass -dc-ip dc01

# BloodHound:
# Automatically maps all trust relationships
# Query: "Find all domain trusts"

# net commands:
nltest /domain_trusts
nltest /trusted_domains
```

---

## 5. Trust Attack Implications

| Trust Type | Attack Path |
|-----------|------------|
| **Parent-Child** | SID History attack → escalate to parent |
| **Forest** | Golden ticket + SID History → cross-forest |
| **External** | Limited — non-transitive, harder to abuse |
| **One-Way** | Only flows in trust direction |
| **Bidirectional** | Attack both directions |

```
TRUST ABUSE SCENARIO:

CHILD DOMAIN COMPROMISE → FOREST ROOT COMPROMISE:

1. Compromise child domain: dev.corp.local
2. Get Domain Admin of dev.corp.local
3. Dump krbtgt hash of dev.corp.local
4. Create Golden Ticket with:
   ├── krbtgt hash from dev.corp.local
   ├── SID of dev.corp.local
   └── Extra SID: Enterprise Admins SID from corp.local
5. Ticket accepted by corp.local (parent trusts child)
6. FULL FOREST COMPROMISE!

DEFENSE: SID Filtering (enabled by default on external trusts)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Domain** | Administrative boundary with own policies |
| **Forest** | Security boundary — collection of domains |
| **Trust** | Allows cross-domain authentication |
| **Parent-Child** | Automatic two-way transitive trust |
| **External** | Manual, non-transitive, cross-forest |
| **Trust abuse** | SID History → escalate across trusts |

---

## Quick Revision Questions

1. **What is the difference between a domain and a forest?**
2. **What trust is automatically created between parent and child domains?**
3. **What does "A trusts B" mean in terms of access?**
4. **How can a child domain compromise lead to forest compromise?**
5. **What is SID Filtering and how does it protect trusts?**

---

[← Previous: AD Architecture](01-ad-architecture.md) | [Next: Objects (Users, Groups, Computers) →](03-objects-users-groups-computers.md)

---

*Unit 1 - Topic 2 of 5 | [Back to README](../README.md)*
