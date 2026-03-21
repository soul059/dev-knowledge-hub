# VLAN Hopping

## Unit 2 - Topic 4 | Layer 2 Security

---

## Overview

**VLAN hopping** is a network attack that allows an attacker to send traffic to or receive traffic from a VLAN they are not supposed to access. By exploiting switch configuration weaknesses, attackers can bypass VLAN segmentation — rendering this critical security boundary useless.

---

## 1. Two Types of VLAN Hopping

```
┌──────────────────────────────────────────────────────────────────┐
│                  VLAN HOPPING ATTACKS                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TYPE 1: SWITCH SPOOFING                                         │
│  Attacker negotiates a trunk link with the switch               │
│                                                                  │
│  TYPE 2: DOUBLE TAGGING                                          │
│  Attacker crafts frames with two 802.1Q tags                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Switch Spoofing Attack

```
┌──────────────────────────────────────────────────────────────────┐
│                  SWITCH SPOOFING                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  By default, many switches run DTP (Dynamic Trunking Protocol)  │
│  on all ports. DTP auto-negotiates trunk links.                 │
│                                                                  │
│  ATTACK:                                                         │
│  Attacker configures their NIC to act as a switch               │
│  and sends DTP negotiation frames.                              │
│                                                                  │
│  ┌──────────┐    DTP Negotiate    ┌──────────┐                  │
│  │ ATTACKER │━━━━━━━━━━━━━━━━━━━━│  SWITCH  │                  │
│  │ (fakes   │    Trunk formed!    │          │                  │
│  │  switch) │                     │          │                  │
│  └──────────┘                     └──────────┘                  │
│                                                                  │
│  Result: Attacker's port becomes a TRUNK                        │
│  → Can access ALL VLANs on the switch                           │
│                                                                  │
│  TOOL:                                                           │
│  yersinia -I   → Select DTP attack → "enabling trunking"       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Defense Against Switch Spoofing

```cisco
! Set ALL access ports to access mode explicitly
interface range GigabitEthernet0/1-20
  switchport mode access           ! Forces access mode
  switchport nonegotiate           ! Disables DTP completely

! Only configure trunks where NEEDED
interface GigabitEthernet0/24
  switchport mode trunk
  switchport nonegotiate
  switchport trunk allowed vlan 10,20,30
```

---

## 3. Double Tagging Attack

```
┌──────────────────────────────────────────────────────────────────┐
│                  DOUBLE TAGGING ATTACK                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONDITIONS REQUIRED:                                            │
│  • Attacker must be on the NATIVE VLAN (usually VLAN 1)         │
│  • Attack is ONE-WAY only (no return traffic)                   │
│  • Trunk must use 802.1Q (not ISL)                              │
│                                                                  │
│  Attacker on VLAN 1 wants to reach target on VLAN 20:           │
│                                                                  │
│  STEP 1: Attacker sends frame with TWO 802.1Q tags             │
│  ┌──────┬──────────┬──────────┬──────────┐                     │
│  │ Dest │ Tag: V1  │ Tag: V20 │ Payload  │                     │
│  │ MAC  │ (outer)  │ (inner)  │          │                     │
│  └──────┴──────────┴──────────┴──────────┘                     │
│                                                                  │
│  STEP 2: First switch (Switch A) sees outer tag = VLAN 1        │
│  Since VLAN 1 is native → strips the outer tag                  │
│  Forwards on trunk with only inner tag:                         │
│  ┌──────┬──────────┬──────────┐                                │
│  │ Dest │ Tag: V20 │ Payload  │                                │
│  │ MAC  │          │          │                                │
│  └──────┴──────────┴──────────┘                                │
│                                                                  │
│  STEP 3: Second switch (Switch B) sees tag = VLAN 20            │
│  Delivers frame to VLAN 20!                                     │
│                                                                  │
│  RESULT: Attacker from VLAN 1 reached VLAN 20 ❌                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Defense Against Double Tagging

```cisco
! DEFENSE 1: Change native VLAN to an UNUSED VLAN
interface GigabitEthernet0/24
  switchport trunk native vlan 999    ! Unused VLAN

! DEFENSE 2: Tag native VLAN traffic
vlan dot1q tag native                 ! Forces tagging on native

! DEFENSE 3: Don't put users in native VLAN
! Never assign user ports to VLAN 1

! DEFENSE 4: Prune VLANs on trunks
interface GigabitEthernet0/24
  switchport trunk allowed vlan 10,20,30
  ! Only allow VLANs that NEED to traverse this trunk
```

---

## 4. Complete VLAN Hopping Prevention Checklist

| Defense | Prevents | Configuration |
|---------|----------|---------------|
| **Disable DTP** | Switch spoofing | `switchport nonegotiate` |
| **Set access mode** | Switch spoofing | `switchport mode access` |
| **Change native VLAN** | Double tagging | `switchport trunk native vlan 999` |
| **Tag native VLAN** | Double tagging | `vlan dot1q tag native` |
| **Prune VLANs** | Both attacks | `switchport trunk allowed vlan X,Y,Z` |
| **Shutdown unused ports** | Physical access | `shutdown` |
| **Assign unused ports** | Physical access | Put in "black hole" VLAN |

---

## 5. Testing for VLAN Hopping

```bash
# Yersinia — Test DTP attacks
yersinia -G                     # GUI mode
# Select DTP → "enabling trunking"

# Scapy — Double tagging test
from scapy.all import *
pkt = Ether()/Dot1Q(vlan=1)/Dot1Q(vlan=20)/IP(dst="target")/ICMP()
sendp(pkt, iface="eth0")

# Frogger — VLAN hopping tool
# https://github.com/commonexploits/vlan-hopping
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **VLAN Hopping** | Bypassing VLAN segmentation to access other VLANs |
| **Switch Spoofing** | Attacker negotiates trunk via DTP |
| **Double Tagging** | Two 802.1Q tags — outer stripped, inner forwarded |
| **Key Defense** | Disable DTP + change native VLAN + access mode |
| **Double Tagging Requirement** | Attacker must be on native VLAN |

---

## Quick Revision Questions

1. **What are the two types of VLAN hopping attacks?**
2. **How does switch spoofing exploit DTP?**
3. **Explain the double tagging attack step by step.**
4. **Why does changing the native VLAN prevent double tagging?**
5. **What command disables DTP on a Cisco switch port?**
6. **List 5 defenses against VLAN hopping.**

---

[← Previous: MAC Flooding](03-mac-flooding.md) | [Next: STP Attacks →](05-stp-attacks.md)

---

*Unit 2 - Topic 4 of 6 | [Back to README](../README.md)*
