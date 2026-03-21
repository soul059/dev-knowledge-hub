# VLANs and Segmentation

## Unit 1 - Topic 4 | Network Fundamentals Review

---

## Overview

**VLANs (Virtual Local Area Networks)** create logically separate networks on the same physical switch infrastructure. From a security perspective, VLANs are a primary tool for **network segmentation** — isolating different types of traffic and limiting the blast radius of network attacks.

---

## 1. How VLANs Work

```
┌──────────────────────────────────────────────────────────────────┐
│                  WITHOUT VLANs (Flat Network)                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────┐                   │
│  │              SINGLE SWITCH               │                   │
│  │  Port1  Port2  Port3  Port4  Port5       │                   │
│  │   │      │      │      │      │          │                   │
│  └───┼──────┼──────┼──────┼──────┼──────────┘                   │
│      │      │      │      │      │                              │
│    User   User   Server  IoT  Guest                             │
│                                                                  │
│  ALL devices in SAME broadcast domain.                           │
│  ALL devices can see ALL traffic. ❌ BAD                         │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│                  WITH VLANs                                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────┐                   │
│  │              SAME SWITCH                 │                   │
│  │  Port1  Port2  Port3  Port4  Port5       │                   │
│  │  VLAN10 VLAN10 VLAN20 VLAN30 VLAN40      │                   │
│  └───┼──────┼──────┼──────┼──────┼──────────┘                   │
│      │      │      │      │      │                              │
│    User   User   Server  IoT  Guest                             │
│                                                                  │
│  Each VLAN is a SEPARATE broadcast domain.                      │
│  VLANs CANNOT communicate without a router/firewall. ✅         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. VLAN Types

| VLAN Type | Purpose | Example |
|-----------|---------|---------|
| **Data VLAN** | Regular user traffic | VLAN 10 — Employees |
| **Voice VLAN** | VoIP traffic (QoS priority) | VLAN 20 — IP Phones |
| **Management VLAN** | Switch/router management | VLAN 99 — Admin access |
| **Native VLAN** | Untagged traffic on trunks | VLAN 1 (change this!) |
| **Guest VLAN** | Visitor/guest access | VLAN 50 — Internet only |

---

## 3. VLAN Trunking

```
┌──────────────────────────────────────────────────────────────────┐
│                  VLAN TRUNKING (802.1Q)                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ACCESS PORT:                 TRUNK PORT:                        │
│  Belongs to ONE VLAN          Carries MULTIPLE VLANs             │
│  User devices connect here    Between switches/routers           │
│  No VLAN tag on frames        Adds 802.1Q VLAN tag               │
│                                                                  │
│  Switch A                              Switch B                  │
│  ┌────────────────┐  Trunk Link  ┌────────────────┐             │
│  │ VLAN10  VLAN20 │══════════════│ VLAN10  VLAN20 │             │
│  │ Port1   Port2  │  (carries    │ Port1   Port2  │             │
│  └────────────────┘   all VLANs) └────────────────┘             │
│                                                                  │
│  802.1Q TAG:                                                     │
│  ┌──────┬──────────┬─────────┬─────┐                            │
│  │ Dest │ Src MAC  │802.1Q   │Data │                            │
│  │ MAC  │          │VLAN Tag │     │                            │
│  └──────┴──────────┴─────────┴─────┘                            │
│                    ↑                                             │
│             4 bytes: TPID + VLAN ID (1-4094)                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. VLAN Security Threats

| Attack | Description | Defense |
|--------|-------------|---------|
| **VLAN Hopping (Switch Spoofing)** | Attacker pretends to be a trunk port | Disable DTP, set ports to access mode |
| **VLAN Hopping (Double Tagging)** | Adds two 802.1Q tags to jump VLANs | Change native VLAN from VLAN 1 |
| **DTP Exploitation** | Negotiates trunk link with switch | `switchport nonegotiate` |
| **VLAN Jumping** | Exploiting misconfigured trunks | Explicit trunk configuration |

### VLAN Hopping — Double Tagging Attack

```
Attacker (VLAN 10) sends frame with TWO tags:

┌──────┬──────────┬──────────┬─────┐
│ Dest │ Outer Tag│ Inner Tag│Data │
│ MAC  │ VLAN 1   │ VLAN 20  │     │  ← Attacker crafts this
└──────┴──────────┴──────────┴─────┘

Switch A strips outer tag (native VLAN 1) → forwards on trunk
Switch B sees inner tag (VLAN 20) → delivers to VLAN 20

Result: Attacker in VLAN 10 reaches VLAN 20! ❌

DEFENSE: Change native VLAN to an unused VLAN (not VLAN 1)
```

---

## 5. VLAN Security Best Practices

| Practice | Configuration |
|----------|--------------|
| **Change native VLAN** | Use unused VLAN (e.g., VLAN 999) as native |
| **Disable DTP** | `switchport nonegotiate` on all ports |
| **Access ports only** | Set user ports to `switchport mode access` |
| **Prune VLANs** | Only allow needed VLANs on trunks |
| **Shutdown unused ports** | `shutdown` on unused switch ports |
| **Assign unused ports** | Put unused ports in a "black hole" VLAN |
| **Use private VLANs** | Isolate hosts within the same VLAN |
| **Inter-VLAN firewall** | Route between VLANs through a firewall |

```cisco
! VLAN hardening example (Cisco)
! Access port configuration
interface GigabitEthernet0/1
  switchport mode access
  switchport access vlan 10
  switchport nonegotiate
  spanning-tree portfast
  spanning-tree bpduguard enable

! Trunk port configuration
interface GigabitEthernet0/24
  switchport mode trunk
  switchport trunk native vlan 999
  switchport trunk allowed vlan 10,20,30
  switchport nonegotiate
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **VLAN** | Logical network segment on a physical switch |
| **Purpose** | Segmentation, broadcast reduction, access control |
| **Trunk** | Link carrying multiple VLANs (802.1Q tagged) |
| **Key Threat** | VLAN hopping (switch spoofing, double tagging) |
| **Defense** | Disable DTP, change native VLAN, access mode on user ports |

---

## Quick Revision Questions

1. **What is a VLAN and what security benefit does it provide?**
2. **What is the difference between an access port and a trunk port?**
3. **Explain the double-tagging VLAN hopping attack.**
4. **Why should the native VLAN be changed from the default VLAN 1?**
5. **What is DTP and why should it be disabled?**
6. **List 4 VLAN security best practices.**

---

[← Previous: IP Addressing & Subnetting](03-ip-addressing-subnetting.md) | [Next: Network Protocols Overview →](05-network-protocols-overview.md)

---

*Unit 1 - Topic 4 of 5 | [Back to README](../README.md)*
