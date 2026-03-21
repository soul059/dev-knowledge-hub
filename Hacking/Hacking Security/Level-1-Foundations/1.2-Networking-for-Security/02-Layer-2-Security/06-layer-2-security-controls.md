# Layer 2 Security Controls

## Unit 2 - Topic 6 | Layer 2 Security

---

## Overview

Layer 2 security controls are switch-based features that protect against the attacks covered in this unit: ARP spoofing, MAC flooding, VLAN hopping, STP attacks, and DHCP abuse. This topic consolidates all Layer 2 defenses into a comprehensive security checklist.

---

## 1. Layer 2 Security Control Map

```
┌──────────────────────────────────────────────────────────────────┐
│                  LAYER 2 SECURITY CONTROLS                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ATTACK                  │  PRIMARY DEFENSE                      │
│  ──────────────────────  │  ──────────────────────               │
│  MAC Flooding            │  Port Security                       │
│  ARP Spoofing            │  Dynamic ARP Inspection (DAI)        │
│  VLAN Hopping (DTP)      │  Disable DTP, access mode            │
│  VLAN Hopping (Double)   │  Change native VLAN                  │
│  STP Attack              │  BPDU Guard, Root Guard              │
│  DHCP Starvation         │  DHCP Snooping                       │
│  Rogue DHCP Server       │  DHCP Snooping                       │
│  Unauthorized Devices    │  802.1X Port Authentication          │
│  Broadcast Storm         │  Storm Control                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Port Security

```cisco
! Limits MACs per port — prevents MAC flooding
interface GigabitEthernet0/1
  switchport mode access
  switchport access vlan 10
  switchport port-security
  switchport port-security maximum 2
  switchport port-security violation shutdown
  switchport port-security mac-address sticky

! Verify
show port-security interface gi0/1
show port-security address
```

---

## 3. DHCP Snooping

```
┌──────────────────────────────────────────────────────────────┐
│              DHCP SNOOPING                                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Creates a TRUSTED database of IP-MAC-Port bindings.         │
│  Only TRUSTED ports can respond to DHCP requests.            │
│                                                              │
│  Trusted Port: Connected to legitimate DHCP server/uplink   │
│  Untrusted Port: All user-facing ports (default)             │
│                                                              │
│  Prevents:                                                   │
│  • Rogue DHCP servers                                        │
│  • DHCP starvation attacks                                   │
│  • Provides binding table for DAI and IP Source Guard        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```cisco
! Enable DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10,20,30

! Trust the uplink to DHCP server
interface GigabitEthernet0/24
  ip dhcp snooping trust

! Rate-limit DHCP on untrusted ports
interface range GigabitEthernet0/1-20
  ip dhcp snooping limit rate 10

! Verify
show ip dhcp snooping
show ip dhcp snooping binding
```

---

## 4. Dynamic ARP Inspection (DAI)

```cisco
! Prerequisites: DHCP Snooping must be enabled
! DAI validates ARP against DHCP snooping binding table

ip arp inspection vlan 10,20,30

! Trust uplink ports
interface GigabitEthernet0/24
  ip arp inspection trust

! Rate limit ARP on untrusted ports
ip arp inspection limit rate 15

! Verify
show ip arp inspection
show ip arp inspection statistics
```

---

## 5. 802.1X Port-Based Authentication

```
┌──────────────────────────────────────────────────────────────┐
│              802.1X AUTHENTICATION                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Three components:                                           │
│                                                              │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │Supplicant│───►│Authenticator │───►│ Auth Server  │       │
│  │ (Client) │    │  (Switch)    │    │ (RADIUS)     │       │
│  └──────────┘    └──────────────┘    └──────────────┘       │
│                                                              │
│  1. Client connects to switch port                           │
│  2. Switch blocks all traffic except 802.1X (EAP)            │
│  3. Client sends credentials via EAP                         │
│  4. Switch forwards to RADIUS server                         │
│  5. RADIUS authenticates → switch opens port                 │
│                                                              │
│  Until authenticated: port is CLOSED                         │
│  After authenticated: port is OPEN                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```cisco
! Enable 802.1X globally
aaa new-model
aaa authentication dot1x default group radius
dot1x system-auth-control

! Configure port
interface GigabitEthernet0/1
  switchport mode access
  authentication port-control auto
  dot1x pae authenticator

! RADIUS server
radius server AUTH-SERVER
  address ipv4 10.0.0.50
  key SecretKey123
```

---

## 6. IP Source Guard

```cisco
! Prevents IP spoofing at Layer 2
! Uses DHCP snooping binding table
interface GigabitEthernet0/1
  ip verify source port-security
```

---

## 7. Storm Control

```cisco
! Prevents broadcast/multicast/unicast storms
interface GigabitEthernet0/1
  storm-control broadcast level 20.00
  storm-control multicast level 20.00
  storm-control action shutdown
```

---

## 8. Complete Layer 2 Hardening Checklist

| # | Control | Command/Action |
|:-:|---------|---------------|
| 1 | **Port Security** | Limit MACs per port, violation shutdown |
| 2 | **DHCP Snooping** | Enable on all VLANs, trust only uplinks |
| 3 | **Dynamic ARP Inspection** | Enable on all VLANs, trust only uplinks |
| 4 | **IP Source Guard** | Prevent IP spoofing on access ports |
| 5 | **BPDU Guard** | Enable on all access ports |
| 6 | **Root Guard** | Enable on designated switch-facing ports |
| 7 | **PortFast** | Enable on access ports |
| 8 | **Disable DTP** | `switchport nonegotiate` on all ports |
| 9 | **Access mode** | `switchport mode access` on user ports |
| 10 | **Change native VLAN** | Use unused VLAN (e.g., 999) |
| 11 | **Prune trunk VLANs** | Only allow needed VLANs |
| 12 | **Shutdown unused ports** | Disable and assign to black hole VLAN |
| 13 | **Storm Control** | Limit broadcast/multicast rates |
| 14 | **802.1X** | Authenticate devices before access |
| 15 | **Logging** | Enable SNMP traps, syslog for violations |

---

## Summary Table

| Control | Prevents |
|---------|----------|
| **Port Security** | MAC flooding |
| **DHCP Snooping** | Rogue DHCP, starvation |
| **DAI** | ARP spoofing |
| **BPDU Guard** | STP manipulation |
| **DTP Disable** | VLAN hopping (switch spoofing) |
| **Native VLAN Change** | VLAN hopping (double tagging) |
| **802.1X** | Unauthorized device access |
| **Storm Control** | Broadcast storms |

---

## Quick Revision Questions

1. **What is the relationship between DHCP Snooping and DAI?**
2. **Name the three components of 802.1X authentication.**
3. **What does IP Source Guard prevent?**
4. **List 5 Layer 2 security controls and what they protect against.**
5. **Why should unused switch ports be shut down and placed in a separate VLAN?**
6. **What is the complete order for enabling Layer 2 security on a new switch?**

---

[← Previous: STP Attacks](05-stp-attacks.md)

---

*Unit 2 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Layer 3 Security →](../03-Layer-3-Security/01-ip-protocol-security.md)*
