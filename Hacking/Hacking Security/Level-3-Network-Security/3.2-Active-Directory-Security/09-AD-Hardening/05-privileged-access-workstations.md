# Privileged Access Workstations (PAWs)

## Unit 9 - Topic 5 | AD Hardening

---

## Overview

**Privileged Access Workstations (PAWs)** are dedicated, hardened machines used exclusively for administrative tasks. By isolating administrative activities to clean, monitored systems, PAWs prevent credential theft from compromised workstations — the most common initial vector for AD attacks.

---

## 1. PAW Concept and Architecture

```
PAW vs NORMAL ADMIN WORKFLOW:

WITHOUT PAW (Dangerous):
┌──────────────────────────────────┐
│ Admin's Daily Workstation        │
│                                  │
│ ✉ Email    🌐 Browser  📂 Files │ ← Attack surface!
│ 🎮 Apps    📱 Personal          │
│                                  │
│ ALSO: RDP to DC, manage AD      │ ← Admin creds here!
│ DA credentials in LSASS!        │
│                                  │
│ Malware from email              │
│ → Dump LSASS → Steal DA creds  │ ← GAME OVER
└──────────────────────────────────┘

WITH PAW (Secure):
┌──────────────────┐  ┌──────────────────┐
│ Daily Workstation│  │ PAW (Clean)      │
│                  │  │                  │
│ ✉ Email         │  │ ❌ No email      │
│ 🌐 Browser      │  │ ❌ No browser    │
│ 📂 Files        │  │ ❌ No internet   │
│ 📱 Apps         │  │                  │
│                  │  │ ✅ RDP to DC     │
│ No admin creds! │  │ ✅ AD management │
│ If compromised: │  │ ✅ DA creds ONLY │
│ → No DA creds   │  │                  │
│ → Contained!    │  │ Heavily hardened │
└──────────────────┘  └──────────────────┘
```

---

## 2. PAW Security Controls

```
PAW HARDENING CHECKLIST:

HARDWARE:
├── ✅ TPM 2.0 for Secure Boot and BitLocker
├── ✅ UEFI firmware (not legacy BIOS)
├── ✅ Hardware token for MFA
└── ✅ Dedicated physical device (or hardened VM)

OS HARDENING:
├── ✅ Clean OS install (not imaged from regular workstation)
├── ✅ Latest OS version, fully patched
├── ✅ BitLocker full disk encryption
├── ✅ Credential Guard enabled
├── ✅ Device Guard / WDAC (application whitelisting)
├── ✅ Windows Firewall — restrictive rules
├── ✅ Disable USB storage devices
├── ✅ Disable Bluetooth
└── ✅ LSASS Protected Process Light (PPL)

NETWORK:
├── ✅ Dedicated admin VLAN
├── ✅ No internet access (or heavily restricted)
├── ✅ Only connects to managed tier resources
├── ✅ DNS filtering (if internet needed)
├── ✅ No access to email servers
└── ✅ No access to file shares (general)

ACCESS CONTROL:
├── ✅ MFA required for all logons
├── ✅ Only tier-appropriate admin accounts can log in
├── ✅ No regular user accounts on PAW
├── ✅ Local admin disabled or LAPS-managed
├── ✅ AppLocker: Only approved applications
└── ✅ PowerShell Constrained Language Mode

MONITORING:
├── ✅ EDR agent deployed
├── ✅ Sysmon with comprehensive config
├── ✅ PowerShell script block logging
├── ✅ PowerShell transcription
├── ✅ Windows Event Forwarding to SIEM
└── ✅ File integrity monitoring
```

---

## 3. PAW Deployment Models

```
PAW DEPLOYMENT OPTIONS:

MODEL 1: DEDICATED HARDWARE (Most Secure)
├── Separate physical machine for admin tasks
├── Stored in secure location
├── Never used for daily tasks
├── Pros: Strongest isolation
└── Cons: Expensive, inconvenient

MODEL 2: PRIVILEGED VM ON SECURE HOST (Balanced)
├── Hardened host OS (no network access)
├── Admin VM for tier management
├── Daily-use VM for email/browsing
├── Hyper-V isolation between VMs
├── Pros: Single device, good isolation
└── Cons: Complex setup, hardware requirements

MODEL 3: JUMP SERVER / BASTION HOST (Simpler)
├── Centralized hardened server
├── Admins RDP to jump server for admin tasks
├── Jump server has restricted access
├── Pros: Easier to manage, centralized
└── Cons: Single point of compromise

RECOMMENDED APPROACH:
├── Tier 0: Dedicated hardware PAW (Model 1)
├── Tier 1: VM-based PAW or jump server (Model 2/3)
└── Tier 2: Jump server (Model 3)
```

---

## 4. PAW GPO Configuration

```powershell
# === GPO FOR PAW MACHINES ===

# Computer Config → Windows Settings → Security Settings:

# USER RIGHTS ASSIGNMENT:
# "Allow log on locally" → Only PAW admin accounts
# "Deny log on locally" → All regular users, lower-tier admins
# "Allow log on through RDP" → Only authorized admin accounts

# SECURITY OPTIONS:
# "Interactive logon: Require smart card" → Enabled (if using MFA)
# "Network access: Do not allow storage of passwords" → Enabled
# "Accounts: Block Microsoft accounts" → Enabled

# APPLOCKER POLICIES:
# Default rules: Allow Windows system files
# Custom rules:
#   Allow: RSAT tools, AD management tools, RDP
#   Allow: Approved PowerShell scripts only
#   Block: Everything else

# WINDOWS FIREWALL:
# Inbound: Block all (except management)
# Outbound: 
#   Allow: DNS (to DCs only)
#   Allow: Kerberos (port 88 to DCs)
#   Allow: LDAP (port 389/636 to DCs)
#   Allow: RDP (port 3389 to managed tier only)
#   Allow: WinRM (port 5985 to managed tier only)
#   Block: HTTP/HTTPS (80/443)
#   Block: SMTP (25/587)
#   Block: Everything else

# CREDENTIAL GUARD:
# "Turn On Virtualization Based Security" → Enabled
# "Credential Guard" → Enabled with UEFI lock

# DEVICE GUARD:
# Code Integrity policies for application whitelisting
```

---

## 5. PAW Operational Guidelines

| Rule | Description |
|------|-------------|
| **No personal use** | PAW is exclusively for admin tasks |
| **No email** | Never access email from PAW |
| **No browsing** | No web browsing from PAW |
| **MFA always** | Multi-factor for all PAW logons |
| **Physical security** | Lock PAW when not in use |
| **Regular patching** | Highest priority for security updates |
| **Monitoring** | All PAW activity logged and alerted |
| **Incident response** | Compromised PAW = assume tier compromised |
| **Regular review** | Audit PAW access and configurations monthly |

```
COMMON MISTAKES TO AVOID:
├── ❌ Using PAW for email "just this once"
├── ❌ Allowing USB drives for file transfer
├── ❌ Connecting PAW to general network
├── ❌ Allowing internet access "for updates"
│   (Use WSUS/SCCM instead)
├── ❌ Not monitoring PAW activities
├── ❌ Shared PAW accounts
└── ❌ PAW without MFA
```

---

## Summary Table

| Aspect | Tier 0 PAW | Tier 1 PAW | Jump Server |
|--------|:----------:|:----------:|:-----------:|
| **Isolation** | 🟢 Maximum | 🟡 High | 🟡 Medium |
| **Internet** | ❌ None | ❌ None | ⚠️ Limited |
| **Hardware** | Dedicated | Dedicated/VM | Shared VM |
| **Cost** | 🔴 High | 🟡 Medium | 🟢 Low |
| **Convenience** | 🔴 Low | 🟡 Medium | 🟢 High |
| **Security** | 🟢 Best | 🟡 Good | 🟡 Acceptable |

---

## Quick Revision Questions

1. **Why should administrative tasks be isolated to PAWs?**
2. **What is the biggest difference between PAW models?**
3. **Why must PAWs have no internet access?**
4. **What security controls are essential on every PAW?**
5. **What should happen if a PAW is suspected of compromise?**

---

[← Previous: LAPS](04-laps.md) | [Next: AD Security Best Practices →](06-ad-security-best-practices.md)

---

*Unit 9 - Topic 5 of 6 | [Back to README](../README.md)*
