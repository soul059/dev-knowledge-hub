# Network Access Control (NAC)

## Unit 10 - Topic 4 | Network Segmentation

---

## Overview

**Network Access Control (NAC)** is a security solution that enforces policies on devices seeking to access the network. NAC verifies the **identity**, **security posture**, and **compliance** of devices before granting access — ensuring only authorized, healthy devices connect to appropriate network segments.

---

## 1. How NAC Works

```
┌──────────────────────────────────────────────────────────────┐
│              NAC WORKFLOW                                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. DEVICE CONNECTS                                          │
│     User plugs into switch or connects to Wi-Fi              │
│                                                              │
│  2. AUTHENTICATION                                           │
│     802.1X: User provides credentials                        │
│     Device presents certificate or MAC address               │
│                                                              │
│  3. POSTURE ASSESSMENT                                       │
│     NAC checks device health:                                │
│     ✅ Antivirus installed and updated?                       │
│     ✅ OS patches current?                                    │
│     ✅ Firewall enabled?                                      │
│     ✅ Approved device type?                                  │
│     ✅ No prohibited software?                                │
│                                                              │
│  4. AUTHORIZATION (Decision)                                 │
│     ┌───────────────┐                                        │
│     │ Compliant?    │                                        │
│     └───────┬───────┘                                        │
│        YES  │   NO                                           │
│        ▼        ▼                                            │
│     Full      Quarantine VLAN                                │
│     Access    (remediation)                                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. NAC Components

| Component | Description |
|-----------|-------------|
| **NAC Server** | Central policy engine (decides access) |
| **NAC Agent** | Software on endpoint for posture assessment |
| **802.1X Supplicant** | Client software for authentication |
| **RADIUS Server** | Backend authentication (integrates with AD) |
| **Network Infrastructure** | Switches/APs that enforce NAC decisions |
| **Remediation Server** | Provides updates/patches for non-compliant devices |

---

## 3. NAC Deployment Modes

| Mode | Description | Pros | Cons |
|------|-------------|------|------|
| **Pre-admission** | Check BEFORE granting any access | Maximum security | Complex setup |
| **Post-admission** | Check AFTER initial access, then restrict | Easier deployment | Brief exposure window |
| **Agent-based** | Software installed on endpoint | Thorough assessment | Requires agent install |
| **Agentless** | Scans device remotely (no install) | BYOD friendly | Less thorough |
| **Inline** | NAC device sits in traffic path | Full control | Single point of failure |
| **Out-of-band** | NAC communicates with switches via SNMP/RADIUS | No bottleneck | Slightly less control |

---

## 4. Posture Assessment Checks

| Check | What It Verifies |
|-------|-----------------|
| **Antivirus** | AV installed, running, signatures current |
| **Patches** | OS and application patches up to date |
| **Firewall** | Host firewall enabled and configured |
| **Encryption** | Disk encryption active (BitLocker/FileVault) |
| **Software** | No prohibited apps (P2P, hacking tools) |
| **Certificate** | Valid device certificate present |
| **Domain membership** | Device joined to corporate domain |
| **Jailbreak/root** | Mobile device not jailbroken or rooted |

---

## 5. NAC Solutions

| Solution | Type | Notes |
|----------|------|-------|
| **Cisco ISE** | Commercial | Market leader, full 802.1X + posture |
| **Aruba ClearPass** | Commercial | Multi-vendor support, BYOD focus |
| **FortiNAC** | Commercial | Fortinet ecosystem integration |
| **PacketFence** | Open source | Full-featured, free NAC |
| **FreeRADIUS** | Open source | 802.1X authentication backend |
| **Microsoft NPS** | Built-in | Windows Server RADIUS/NAC |

---

## 6. NAC + Segmentation

```
NAC DECISION → VLAN ASSIGNMENT:

Employee (compliant, domain-joined):
→ Corporate VLAN (full access)

Employee (non-compliant — outdated AV):
→ Quarantine VLAN (update AV, then reassess)

Guest (no agent):
→ Guest VLAN (internet only)

IoT Device (printer, camera):
→ IoT VLAN (restricted access)

Unknown Device:
→ Blocked (no network access)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **NAC** | Verify identity and health before network access |
| **802.1X** | Port-based authentication standard |
| **Posture Check** | Verify AV, patches, firewall, encryption |
| **Non-compliant** | Quarantine VLAN for remediation |
| **BYOD** | Agentless NAC for personal devices |
| **Key Solutions** | Cisco ISE, ClearPass, PacketFence |

---

## Quick Revision Questions

1. **What does NAC verify before granting network access?**
2. **What happens to a non-compliant device?**
3. **What is the difference between agent-based and agentless NAC?**
4. **How does NAC integrate with VLANs?**
5. **Name 3 NAC solutions.**

---

[← Previous: Software-Defined Networking](03-software-defined-networking.md) | [Next: Air-Gapped Networks →](05-air-gapped-networks.md)

---

*Unit 10 - Topic 4 of 5 | [Back to README](../README.md)*
