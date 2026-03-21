# STP Attacks

## Unit 2 - Topic 5 | Layer 2 Security

---

## Overview

**STP (Spanning Tree Protocol)** prevents Layer 2 loops in switched networks by electing a **Root Bridge** and blocking redundant paths. Attackers can exploit STP to become the Root Bridge, enabling them to intercept all network traffic — a powerful man-in-the-middle attack at Layer 2.

---

## 1. How STP Works

```
┌──────────────────────────────────────────────────────────────────┐
│                  SPANNING TREE PROTOCOL                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WITHOUT STP:                       WITH STP:                    │
│                                                                  │
│  SW1 ─── SW2                        SW1 ─── SW2                  │
│   │  \ /  │    Broadcast            │       │    One path        │
│   │   X   │    storm! ❌            │       │    blocked ✅      │
│   │  / \  │    (infinite loop)      │    ╳  │    (no loop)       │
│  SW3 ─── SW4                        SW3 ─── SW4                  │
│                                                                  │
│  STP PROCESS:                                                    │
│  1. Elect ROOT BRIDGE (lowest Bridge ID wins)                   │
│  2. Calculate shortest path to Root Bridge                      │
│  3. BLOCK redundant paths to prevent loops                      │
│  4. Only ONE active path between any two switches               │
│                                                                  │
│  Bridge ID = Priority (default 32768) + MAC Address             │
│  LOWEST Bridge ID becomes ROOT BRIDGE                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. STP Attack — Root Bridge Takeover

```
┌──────────────────────────────────────────────────────────────────┐
│                  ROOT BRIDGE ATTACK                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BEFORE ATTACK:                                                  │
│  Legitimate Root Bridge (Priority 32768, good MAC)              │
│                                                                  │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐            │
│  │  SW1     │───────│  SW2     │───────│  SW3     │            │
│  │ ROOT ⭐  │       │          │       │          │            │
│  │ Pri:32768│       │ Pri:32768│       │ Pri:32768│            │
│  └──────────┘       └──────────┘       └──────────┘            │
│                                                                  │
│  ATTACK:                                                         │
│  Attacker sends BPDUs with LOWER priority (e.g., 0)            │
│                                                                  │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐            │
│  │  SW1     │───────│  SW2     │───────│  SW3     │            │
│  │ old root │       │          │       │          │            │
│  │ Pri:32768│       │ Pri:32768│       │ Pri:32768│            │
│  └──────────┘       └────┬─────┘       └──────────┘            │
│                          │                                      │
│                    ┌─────┴──────┐                                │
│                    │  ATTACKER  │                                │
│                    │  NEW ROOT ⭐│  Priority: 0 (lowest wins!)   │
│                    │  Pri: 0    │                                │
│                    └────────────┘                                │
│                                                                  │
│  RESULT: Network topology recalculates!                         │
│  All traffic now flows THROUGH the attacker's device.           │
│  → Man-in-the-Middle at Layer 2                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### STP Attack Tools

```bash
# Yersinia — Primary STP attack tool
yersinia -G                      # GUI mode
# Select STP → "claiming root role"

yersinia stp -attack 4           # Become root bridge

# Scapy — Custom BPDU crafting
from scapy.all import *
bpdu = LLC()/STP(bpdutype=0x00, rootid=0, bridgeid=0)
sendp(Ether(dst="01:80:c2:00:00:00")/bpdu, iface="eth0")
```

---

## 3. Impact of STP Attacks

| Impact | Description |
|--------|-------------|
| **MITM** | All traffic routed through attacker |
| **Network Disruption** | Topology recalculation causes downtime |
| **DoS** | Continuous BPDU storms prevent convergence |
| **Traffic Sniffing** | Attacker positioned to capture all frames |
| **Cascading Failures** | Repeated topology changes can crash switches |

---

## 4. STP Variants

| Protocol | Standard | Convergence | Notes |
|----------|----------|:-----------:|-------|
| **STP** | 802.1D | 30-50 sec | Original, slow |
| **RSTP** | 802.1w | 1-3 sec | Rapid, modern |
| **MSTP** | 802.1s | 1-3 sec | Multiple instances per VLAN |
| **PVST+** | Cisco | 30-50 sec | Per-VLAN STP (Cisco proprietary) |
| **Rapid-PVST+** | Cisco | 1-3 sec | Rapid + per-VLAN (Cisco) |

---

## 5. Defenses Against STP Attacks

### BPDU Guard

```cisco
! BPDU Guard — Shuts down port if BPDU is received
! Use on ACCESS ports (where end devices connect)
interface GigabitEthernet0/1
  spanning-tree portfast
  spanning-tree bpduguard enable

! Enable globally for all PortFast interfaces
spanning-tree portfast bpduguard default
```

### Root Guard

```cisco
! Root Guard — Prevents port from becoming Root Port
! Use on ports that should NEVER receive superior BPDUs
interface GigabitEthernet0/24
  spanning-tree guard root
```

### All STP Defenses

| Defense | Purpose | Where to Apply |
|---------|---------|---------------|
| **BPDU Guard** | Disables port receiving BPDUs | Access ports (user-facing) |
| **Root Guard** | Prevents unauthorized root bridge | Ports facing other switches |
| **BPDU Filter** | Suppresses BPDU sending/receiving | Edge ports (use carefully) |
| **Loop Guard** | Prevents loops from unidirectional link failures | Redundant links |
| **PortFast** | Skips STP states for faster port-up | Access ports only |
| **Set Root Priority** | Explicitly set root bridge priority low | Core switches |

```cisco
! Explicitly set your core switch as root bridge
spanning-tree vlan 1-100 root primary
! Or set specific low priority
spanning-tree vlan 10 priority 4096
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **STP** | Prevents L2 loops by blocking redundant paths |
| **Root Bridge** | Lowest Bridge ID = Root (controls topology) |
| **STP Attack** | Attacker sends low-priority BPDUs to become Root |
| **Impact** | MITM, network disruption, DoS |
| **Key Defense** | BPDU Guard on access ports, Root Guard on switch ports |
| **Tool** | Yersinia |

---

## Quick Revision Questions

1. **What problem does STP solve?**
2. **How is the Root Bridge elected?**
3. **How can an attacker become the Root Bridge?**
4. **What is BPDU Guard and where should it be applied?**
5. **What is the difference between BPDU Guard and Root Guard?**
6. **What tool is used to perform STP attacks?**

---

[← Previous: VLAN Hopping](04-vlan-hopping.md) | [Next: Layer 2 Security Controls →](06-layer-2-security-controls.md)

---

*Unit 2 - Topic 5 of 6 | [Back to README](../README.md)*
