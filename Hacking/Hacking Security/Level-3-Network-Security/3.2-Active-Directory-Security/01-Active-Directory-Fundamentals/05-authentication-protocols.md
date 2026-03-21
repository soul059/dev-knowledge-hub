# Authentication Protocols (NTLM and Kerberos)

## Unit 1 - Topic 5 | Active Directory Fundamentals

---

## Overview

Active Directory uses two primary authentication protocols: **NTLM** (legacy) and **Kerberos** (default since Windows 2000). Understanding how each works is critical because most AD attacks exploit weaknesses in these authentication mechanisms.

---

## 1. NTLM Authentication

```
NTLM CHALLENGE-RESPONSE:

CLIENT                                SERVER/DC
  │                                      │
  │  1. NEGOTIATE (request auth)         │
  │ ────────────────────────────────────→│
  │                                      │
  │  2. CHALLENGE (random 8-byte nonce)  │
  │ ←────────────────────────────────────│
  │                                      │
  │  3. RESPONSE                         │
  │  (NTLM hash + challenge = response)  │
  │ ────────────────────────────────────→│
  │                                      │
  │  4. Server sends to DC for verify    │
  │                           DC verifies│
  │                                      │
  │  5. SUCCESS / FAILURE                │
  │ ←────────────────────────────────────│

NTLM HASH:
├── Computed: MD4(UTF-16LE(password))
├── No salt — same password = same hash everywhere
├── Enables Pass-the-Hash attacks!
├── Example: aad3b435b51404eeaad3b435b51404ee
└── Stored in SAM (local) and NTDS.dit (domain)

NTLMv1 vs NTLMv2:
├── NTLMv1: Weak, easily cracked, relay-friendly
├── NTLMv2: Stronger, includes timestamp + target info
├── Both vulnerable to relay attacks!
└── NTLMv2 is the minimum recommended version
```

---

## 2. Kerberos Authentication

```
KERBEROS FLOW:

CLIENT              KDC (Domain Controller)         SERVICE
  │                        │                           │
  │  1. AS-REQ             │                           │
  │  (username + timestamp │                           │
  │   encrypted with       │                           │
  │   user's hash)         │                           │
  │───────────────────────→│                           │
  │                        │ Verifies against          │
  │                        │ NTDS.dit                  │
  │  2. AS-REP             │                           │
  │  (TGT encrypted       │                           │
  │   with krbtgt hash)    │                           │
  │←───────────────────────│                           │
  │                        │                           │
  │  3. TGS-REQ            │                           │
  │  (TGT + service name)  │                           │
  │───────────────────────→│                           │
  │                        │                           │
  │  4. TGS-REP            │                           │
  │  (Service Ticket       │                           │
  │   encrypted with       │                           │
  │   service acct hash)   │                           │
  │←───────────────────────│                           │
  │                        │                           │
  │  5. AP-REQ (Service Ticket)                        │
  │────────────────────────────────────────────────────→│
  │                        │                           │
  │  6. Access Granted ✅                               │
  │←────────────────────────────────────────────────────│

KEY ELEMENTS:
├── KDC (Key Distribution Center) — runs on every DC
├── TGT (Ticket Granting Ticket) — encrypted with krbtgt hash
├── TGS (Ticket Granting Service) — service ticket
├── PAC (Privilege Attribute Certificate) — user's groups/rights
└── Realm — Kerberos term for domain (CORP.LOCAL)
```

---

## 3. NTLM vs Kerberos Comparison

| Feature | NTLM | Kerberos |
|---------|:----:|:--------:|
| **Default** | Fallback | Primary (since Win2K) |
| **Speed** | Slower | Faster (ticket caching) |
| **Mutual auth** | ❌ No | ✅ Yes |
| **Delegation** | ❌ Limited | ✅ Full delegation |
| **Replay protection** | ❌ Limited | ✅ Timestamps |
| **Pass-the-Hash** | ✅ Vulnerable | N/A (different attack) |
| **Pass-the-Ticket** | N/A | ✅ Vulnerable |
| **Relay attacks** | ✅ Vulnerable | ❌ Protected |
| **Offline cracking** | N/A | Kerberoasting |
| **Used when** | IP address, fallback | Domain name, default |

---

## 4. Authentication Attacks Summary

```
NTLM ATTACKS:
├── Pass-the-Hash — reuse NTLM hash without cracking
├── NTLM Relay — forward captured auth to another target
├── Responder — capture NTLMv2 hashes via LLMNR/NBT-NS
├── Brute force — crack captured NTLMv2 hashes
└── NTLM downgrade — force NTLMv1 (weaker)

KERBEROS ATTACKS:
├── Kerberoasting — request service tickets, crack offline
├── AS-REP Roasting — no pre-auth required, get hash
├── Golden Ticket — forge TGT with krbtgt hash
├── Silver Ticket — forge service ticket with service hash
├── Pass-the-Ticket — reuse stolen Kerberos tickets
├── Overpass-the-Hash — NTLM hash → Kerberos ticket
├── Diamond Ticket — modify legitimate TGT
├── Skeleton Key — patch LSASS, master password
├── Unconstrained Delegation — steal TGTs
└── Constrained Delegation — S4U abuse
```

---

## 5. When NTLM Is Used (Fallback)

```
KERBEROS IS USED WHEN:
├── Client specifies server by hostname (DNS name)
├── Client is domain-joined
├── Target server is in same forest
├── Port 88 (Kerberos) is reachable
└── Default for all domain authentication

NTLM FALLBACK OCCURS WHEN:
├── Client uses IP address (not hostname)
├── Client is not domain-joined
├── Target is not in the domain/forest
├── Kerberos fails (network issues, time sync)
├── Application explicitly requests NTLM
└── External/workgroup authentication

WHY THIS MATTERS:
├── Attackers can force NTLM (use IP instead of hostname)
├── NTLM relay attacks exploit this
├── Disable NTLM where possible!
└── Monitor NTLM usage with auditing
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **NTLM** | Challenge-response, no salt, PtH vulnerable |
| **Kerberos** | Ticket-based, default, Kerberoasting vulnerable |
| **TGT** | Encrypted with krbtgt hash (Golden Ticket) |
| **Service Ticket** | Encrypted with service hash (Kerberoasting) |
| **NTLM fallback** | When IP is used instead of hostname |
| **krbtgt** | Most important hash in the domain |

---

## Quick Revision Questions

1. **How does NTLM challenge-response authentication work?**
2. **What are the main steps in Kerberos authentication?**
3. **Why is the krbtgt hash the most valuable hash in AD?**
4. **When does NTLM fallback occur instead of Kerberos?**
5. **What attacks target NTLM vs Kerberos?**

---

[← Previous: Group Policy Basics](04-group-policy-basics.md)

---

*Unit 1 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: AD Enumeration →](../02-AD-Enumeration/01-ad-enumeration-techniques.md)*
