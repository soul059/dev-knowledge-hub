# Credential Guard

## Unit 9 - Topic 3 | AD Hardening

---

## Overview

**Windows Credential Guard** uses Virtualization-Based Security (VBS) to isolate LSASS secrets in a protected memory space that is inaccessible even to kernel-level attackers. It prevents extraction of NTLM hashes, Kerberos tickets, and other credentials from memory — blocking the most common credential theft attacks.

---

## 1. How Credential Guard Works

```
WITHOUT CREDENTIAL GUARD:
┌───────────────────────────────┐
│         Windows OS            │
│                               │
│  ┌─────────┐  ┌────────────┐ │
│  │  LSASS  │  │  Attacker  │ │
│  │         │  │  (Admin)   │ │
│  │ Hashes  │←─│  Mimikatz  │ │
│  │ Tickets │  │  reads     │ │
│  │ Plaintext│  │  memory    │ │
│  └─────────┘  └────────────┘ │
│  (Same kernel space!)        │
└───────────────────────────────┘

WITH CREDENTIAL GUARD:
┌────────────────────────────────────────┐
│  Hypervisor (VBS)                      │
│                                        │
│  ┌──────────────┐  ┌───────────────┐  │
│  │ VTL 1        │  │ VTL 0         │  │
│  │ (Isolated)   │  │ (Normal OS)   │  │
│  │              │  │               │  │
│  │  LSA Isolated│  │  LSASS       │  │
│  │  (LSAIso)    │  │  (Limited)   │  │
│  │              │  │               │  │
│  │  ✅ Hashes   │  │  ❌ No hashes│  │
│  │  ✅ Tickets  │  │  ❌ No tickets│  │
│  │              │  │               │  │
│  │  Attacker    │  │  Attacker     │  │
│  │  CAN'T reach!│  │  CAN reach,  │  │
│  │              │  │  but nothing  │  │
│  │              │  │  to steal!    │  │
│  └──────────────┘  └───────────────┘  │
└────────────────────────────────────────┘
```

---

## 2. Enabling Credential Guard

```powershell
# === REQUIREMENTS ===
# OS: Windows 10 Enterprise/Education, Server 2016+
# Hardware: UEFI firmware, Secure Boot, TPM (recommended)
# Virtualization: Intel VT-x or AMD-V
# Domain functional level: Any (but 2012 R2+ recommended)

# === METHOD 1: GROUP POLICY (Recommended) ===
# Computer Config → Admin Templates →
# System → Device Guard →
# "Turn On Virtualization Based Security"
# → Enabled
# Credential Guard Configuration:
# → "Enabled with UEFI lock" (most secure, hard to disable)
# → "Enabled without lock" (can be disabled remotely)

# === METHOD 2: REGISTRY ===
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard" \
  /v EnableVirtualizationBasedSecurity /t REG_DWORD /d 1 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\LSA" \
  /v LsaCfgFlags /t REG_DWORD /d 1 /f
# 1 = Enabled with UEFI lock
# 2 = Enabled without lock

# === METHOD 3: POWERSHELL (Windows 10/11) ===
# Enable required features:
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-Hypervisor
# Reboot required

# === VERIFY ===
# Check if running:
Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard
# VirtualizationBasedSecurityStatus:
# 0 = Not running
# 1 = Enabled but not running
# 2 = Running ✅

# Or: msinfo32.exe → System Summary
# "Virtualization-based Security" = Running
# "Credential Guard" = Running
```

---

## 3. What Credential Guard Blocks

| Attack | Without CG | With CG |
|--------|:----------:|:-------:|
| **Mimikatz sekurlsa::logonpasswords** | ✅ Works | ❌ Blocked |
| **Pass-the-Hash (NTLM)** | ✅ Works | ❌ Blocked |
| **Pass-the-Ticket** | ✅ Works | ❌ Blocked |
| **WDigest plaintext** | ✅ Works | ❌ Blocked |
| **Kerberoasting** | ✅ Works | ✅ Still works* |
| **DCSync** | ✅ Works | ✅ Still works* |
| **Golden Ticket creation** | ✅ Works | ✅ Still works* |
| **SAM dump** | ✅ Works | ✅ Still works** |
| **LSASS process dump** | ✅ Useful | ⚠️ Empty/Limited |
| **Skeleton Key** | ✅ Works | ❌ Blocked |

```
* Kerberoasting, DCSync, Golden Ticket still work because
  they don't rely on extracting credentials from LSASS
  — they use network protocols or forged tickets

** SAM dump still works because SAM is registry-based,
   not in LSASS isolated memory
```

---

## 4. Credential Guard Limitations

```
LIMITATIONS AND BYPASSES:

1. DOESN'T PROTECT SAM:
   ├── Local account hashes still extractable
   ├── reg save HKLM\SAM still works
   └── Use LAPS for local admin passwords

2. DOESN'T PREVENT KEYLOGGING:
   ├── Credentials at time of entry can be captured
   ├── Keystroke capture still viable
   └── Screen capture still viable

3. DOESN'T BLOCK NETWORK ATTACKS:
   ├── Kerberoasting still works
   ├── NTLM relay still possible (for non-CG machines)
   ├── AS-REP Roasting unaffected
   └── DCSync (if you have rights) unaffected

4. COMPATIBILITY ISSUES:
   ├── Some applications can't work with CG
   ├── WiFi/VPN using MSCHAPv2 may fail
   ├── NTLMv1 authentication breaks
   ├── Some smartcard drivers incompatible
   └── CredSSP (used by some RDP) may fail

5. UEFI LOCK BYPASS:
   ├── "Without UEFI lock" → can be disabled via registry
   ├── "With UEFI lock" → requires physical access to disable
   └── Always use UEFI lock in production!

6. HYPERVISOR ATTACKS:
   ├── Theoretical hypervisor-level attacks
   ├── Very sophisticated (nation-state level)
   └── Practical risk is very low
```

---

## 5. Credential Guard Best Practices

```
DEPLOYMENT RECOMMENDATIONS:

1. DEPLOY TO ALL TIER 0 SYSTEMS FIRST:
   ├── Domain Controllers
   ├── PAWs
   └── Certificate Authorities

2. EXPAND TO TIER 1:
   ├── Member servers
   └── Application servers

3. EXPAND TO TIER 2:
   ├── Workstations (where hardware supports)
   └── Laptops

COMBINE WITH:
├── Protected Users group
├── LAPS (for local admin)
├── LSA Protection (PPL)
├── Admin tiering
├── Network segmentation
└── Strong monitoring

TESTING:
├── Test in lab first
├── Monitor for authentication failures
├── Check application compatibility
├── Have rollback plan (use "without lock" initially)
└── Graduate to UEFI lock after validation
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Technology** | Virtualization-Based Security (VBS) |
| **Protects** | NTLM hashes, Kerberos tickets in LSASS |
| **Blocks** | Mimikatz, PtH, PtT, WDigest, Skeleton Key |
| **Doesn't block** | SAM dump, Kerberoast, DCSync, keylogging |
| **Requirement** | UEFI, Secure Boot, VT-x/AMD-V |
| **Best practice** | Enable with UEFI lock + Protected Users |

---

## Quick Revision Questions

1. **How does Credential Guard isolate LSASS credentials?**
2. **What attacks does Credential Guard NOT prevent?**
3. **Why should UEFI lock be used with Credential Guard?**
4. **What is LSAIso and what is its role?**
5. **How do you verify Credential Guard is running?**

---

[← Previous: Protected Users Group](02-protected-users-group.md) | [Next: LAPS →](04-laps.md)

---

*Unit 9 - Topic 3 of 6 | [Back to README](../README.md)*
