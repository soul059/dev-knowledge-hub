# Tiered Administration Model

## Unit 9 - Topic 1 | AD Hardening

---

## Overview

The **Tiered Administration Model** (also called the Administrative Tier Model or Enhanced Security Admin Environment — ESAE) separates administrative privileges into distinct tiers to prevent credential theft from cascading across the entire environment. It's the most impactful single defense against AD attacks.

---

## 1. The Three Tiers

```
TIERED ADMINISTRATION MODEL:

TIER 0 — DOMAIN CONTROLLERS & IDENTITY
├── Assets: Domain Controllers, AD, PKI, ADFS
├── Accounts: Domain Admins, Enterprise Admins
├── Access: ONLY from Tier 0 PAWs
├── Policy: Never log into lower tiers
└── Impact if compromised: Entire domain

TIER 1 — SERVERS & APPLICATIONS
├── Assets: Member servers, SQL, Exchange, apps
├── Accounts: Server administrators
├── Access: Only from Tier 1 PAWs
├── Policy: Never log into Tier 2 or Tier 0
└── Impact if compromised: Server infrastructure

TIER 2 — WORKSTATIONS & END USERS
├── Assets: Workstations, laptops, printers
├── Accounts: Help desk, desktop admins
├── Access: Standard admin workstations
├── Policy: Never log into Tier 1 or Tier 0
└── Impact if compromised: End-user devices

ISOLATION PRINCIPLE:
┌─────────────────────────────────────────────┐
│              TIER 0 (IDENTITY)              │
│  DA accounts, DCs, PKI, ADFS               │
│  ❌ Never logs into Tier 1 or Tier 2        │
├─────────────────────────────────────────────┤
│  ↑ TIER 1 CAN'T ACCESS TIER 0 ↑           │
├─────────────────────────────────────────────┤
│              TIER 1 (SERVERS)               │
│  Server admin accounts, member servers      │
│  ❌ Never logs into Tier 2 or Tier 0        │
├─────────────────────────────────────────────┤
│  ↑ TIER 2 CAN'T ACCESS TIER 1 ↑           │
├─────────────────────────────────────────────┤
│           TIER 2 (WORKSTATIONS)             │
│  Help desk accounts, workstations           │
│  ❌ Never logs into Tier 1 or Tier 0        │
└─────────────────────────────────────────────┘
```

---

## 2. Why Tiering Prevents AD Attacks

```
WITHOUT TIERING (Typical Attack):

1. Attacker compromises workstation (Tier 2)
2. DA was logged into workstation for support
3. Attacker dumps LSASS → DA hash!
4. Attacker PtH to DC → DOMAIN COMPROMISED!

Workstation → DA credentials → Domain Controller
     💻          🔓              💀

WITH TIERING:

1. Attacker compromises workstation (Tier 2)
2. Only Tier 2 admin (helpdesk) credentials available
3. Tier 2 creds can't access servers or DCs
4. Attacker is CONTAINED at Tier 2!

Workstation → Tier 2 creds only → ❌ Can't reach DC
     💻          🔑              🛡️

THE KEY INSIGHT:
├── Credentials are left in memory wherever you log in
├── If DA logs into workstation → DA hash on workstation
├── If workstation is compromised → DA is compromised
├── Solution: DA NEVER logs into workstation
└── Each tier's credentials only exist at that tier
```

---

## 3. Implementing Tiered Administration

```powershell
# === OU STRUCTURE ===
# Create Tier-based OU structure:
# OU=Tier 0,DC=corp,DC=local
#   OU=Accounts   → Tier 0 admin accounts
#   OU=Servers    → Domain Controllers (moved from default)
#   OU=PAWs       → Privileged Access Workstations
#
# OU=Tier 1,DC=corp,DC=local
#   OU=Accounts   → Server admin accounts
#   OU=Servers    → Member servers
#
# OU=Tier 2,DC=corp,DC=local
#   OU=Accounts   → Helpdesk accounts
#   OU=Workstations → End-user computers

# === GPO: RESTRICT TIER 0 LOGON ===
# Apply to Tier 2 & Tier 1 computers:
# Computer Config → Windows Settings → Security Settings →
# Local Policies → User Rights Assignment →
# "Deny log on locally" → Add Tier 0 admin groups
# "Deny log on through RDP" → Add Tier 0 admin groups
# "Deny access from network" → Add Tier 0 admin groups

# Result: Tier 0 accounts CANNOT log into lower tier machines
# Even if credentials are known!

# === ACCOUNT NAMING CONVENTION ===
# Regular account: jsmith
# Tier 2 admin:    t2-jsmith
# Tier 1 admin:    t1-jsmith
# Tier 0 admin:    t0-jsmith
# Each has different password, MFA, and access rights

# === AUTHENTICATION POLICY SILOS ===
# Windows Server 2012 R2+:
# Restrict where accounts can authenticate
New-ADAuthenticationPolicySilo -Name "Tier0Silo" \
  -UserAllowedToAuthenticateFrom "O:SYG:SYD:(XA;OICI;CR;;;WD;(@USER.ad://ext/AuthenticationSilo == `"Tier0Silo`"))"
```

---

## 4. Privileged Access Workstations (PAWs)

```
PAW DEPLOYMENT:

WHAT IS A PAW:
├── Hardened workstation dedicated to admin tasks
├── No internet access, no email, no browsing
├── Only connects to the corresponding tier
├── Clean install, fully patched, monitored
└── Physical or virtual separation

PAW TYPES:
├── Tier 0 PAW → Manages DCs, AD, PKI
│   ├── Connects ONLY to Tier 0 resources
│   ├── Most restricted (no internet at all)
│   └── Stored in secure location
│
├── Tier 1 PAW → Manages servers, applications
│   ├── Connects ONLY to Tier 1 resources
│   └── Moderate restrictions
│
└── Tier 2 PAW → Manages workstations
    ├── Connects to Tier 2 resources
    └── Less restricted but still hardened

PAW HARDENING:
├── Credential Guard enabled
├── Device Guard / WDAC
├── AppLocker (whitelist applications)
├── BitLocker full disk encryption
├── Sysmon + EDR
├── USB restrictions
├── No local admin for users
├── Automatic updates
├── Network isolation (VLAN)
└── MFA for all logons
```

---

## 5. Tiering Maturity Levels

| Level | Implementation | Effort |
|-------|---------------|:------:|
| **Level 1** | Deny Tier 0 logon to Tier 1/2 GPO | 🟢 Low |
| **Level 2** | Separate admin accounts per tier | 🟡 Medium |
| **Level 3** | PAWs for Tier 0 administration | 🟡 Medium |
| **Level 4** | Authentication Policy Silos | 🔴 High |
| **Level 5** | Full ESAE with bastion forest | 🔴 Very High |

```
QUICK WIN (Start Here):
├── Create separate DA account (t0-admin)
├── GPO: Deny t0-admin logon to workstations/servers
├── Only use t0-admin from DC console or PAW
├── This alone blocks most credential theft!
└── Takes: 1-2 hours to implement
```

---

## Summary Table

| Tier | Assets | Accounts | Never Logs Into |
|:----:|--------|----------|:--------------:|
| **0** | DCs, AD, PKI | DA, EA | Tier 1, Tier 2 |
| **1** | Servers | Server admins | Tier 0, Tier 2 |
| **2** | Workstations | Helpdesk | Tier 0, Tier 1 |

---

## Quick Revision Questions

1. **What problem does the tiered administration model solve?**
2. **Why should Domain Admins never log into workstations?**
3. **What is a Privileged Access Workstation (PAW)?**
4. **What is the easiest first step to implement tiering?**
5. **What are Authentication Policy Silos?**

---

[Next: Protected Users Group →](02-protected-users-group.md)

---

*Unit 9 - Topic 1 of 6 | [Back to README](../README.md)*
