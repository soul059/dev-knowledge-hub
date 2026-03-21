# MAC Addresses

## Unit 2 - Topic 1 | Layer 2 Security

---

## Overview

A **MAC (Media Access Control) address** is a unique hardware identifier assigned to every network interface card (NIC). Operating at **Layer 2** of the OSI model, MAC addresses are used for local network communication. Understanding how MAC addresses work — and how they can be spoofed — is critical for network security.

---

## 1. MAC Address Structure

```
┌──────────────────────────────────────────────────────────────────┐
│                  MAC ADDRESS FORMAT                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MAC Address: AA:BB:CC:DD:EE:FF                                 │
│               ├────────┤├────────┤                               │
│                 OUI       Device ID                              │
│              (Vendor)    (Unique per NIC)                        │
│                                                                  │
│  • 48 bits (6 bytes) total                                      │
│  • Written as 12 hex characters                                 │
│  • First 3 bytes = OUI (Organizationally Unique Identifier)     │
│  • Last 3 bytes = Device-specific                               │
│                                                                  │
│  FORMATS:                                                        │
│  Linux:    aa:bb:cc:dd:ee:ff                                    │
│  Windows:  AA-BB-CC-DD-EE-FF                                    │
│  Cisco:    aabb.ccdd.eeff                                       │
│                                                                  │
│  SPECIAL ADDRESSES:                                              │
│  FF:FF:FF:FF:FF:FF = Broadcast (all devices)                    │
│  01:00:5E:xx:xx:xx = IPv4 Multicast                             │
│  33:33:xx:xx:xx:xx = IPv6 Multicast                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. How MAC Addresses Work in Switching

```
┌──────────────────────────────────────────────────────────────┐
│              SWITCH MAC ADDRESS TABLE (CAM Table)             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Switch receives a frame on Port 1 from MAC AA:BB:CC:11:22:33│
│  → Learns: "AA:BB:CC:11:22:33 is on Port 1"                 │
│  → Stores in CAM (Content Addressable Memory) table          │
│                                                              │
│  ┌──────────────────────────┬────────┬─────────┐             │
│  │ MAC Address              │  Port  │  VLAN   │             │
│  ├──────────────────────────┼────────┼─────────┤             │
│  │ AA:BB:CC:11:22:33       │ Gi0/1  │  10     │             │
│  │ AA:BB:CC:44:55:66       │ Gi0/2  │  10     │             │
│  │ AA:BB:CC:77:88:99       │ Gi0/3  │  20     │             │
│  └──────────────────────────┴────────┴─────────┘             │
│                                                              │
│  When frame arrives for AA:BB:CC:44:55:66:                   │
│  Switch looks up table → forward to Port Gi0/2 ONLY         │
│                                                              │
│  If destination MAC NOT in table:                            │
│  Switch FLOODS frame to ALL ports (like a hub) ← Security risk│
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. MAC Address Security Concerns

| Concern | Description |
|---------|-------------|
| **MAC Spoofing** | Changing your MAC to impersonate another device |
| **MAC Flooding** | Overflowing CAM table to force switch into hub mode |
| **Bypass MAC Filtering** | MAC-based access control easily defeated by spoofing |
| **OUI Fingerprinting** | Identifying device vendor from MAC address |
| **Tracking** | MAC addresses used to track devices (Wi-Fi probes) |

### MAC Spoofing

```bash
# Linux — Change MAC address
ip link set eth0 down
ip link set eth0 address 00:11:22:33:44:55
ip link set eth0 up

# macchanger tool
macchanger -r eth0          # Random MAC
macchanger -m 00:11:22:33:44:55 eth0  # Specific MAC

# Windows (PowerShell) — Change MAC
Set-NetAdapter -Name "Ethernet" -MacAddress "00-11-22-33-44-55"

# WHY IT MATTERS:
# MAC-based access control (e.g., MAC filtering on Wi-Fi) 
# is NOT real security — it's trivially bypassed.
```

---

## 4. OUI Lookup (Vendor Identification)

```
MAC: B8:27:EB:xx:xx:xx → Raspberry Pi Foundation
MAC: 00:50:56:xx:xx:xx → VMware
MAC: 00:0C:29:xx:xx:xx → VMware
MAC: 08:00:27:xx:xx:xx → VirtualBox
MAC: 00:1A:2B:xx:xx:xx → Cisco

SECURITY USE:
During recon, OUI lookup reveals device types on the network.
Finding VirtualBox/VMware MACs may indicate virtual machines.
```

---

## 5. Useful Commands

```bash
# View MAC address
ip link show              # Linux
ipconfig /all             # Windows
ifconfig                  # macOS/Linux

# View switch CAM table (Cisco)
show mac address-table

# View ARP cache (MAC ↔ IP mapping)
arp -a                    # Windows/Linux
ip neigh show             # Linux

# OUI lookup
# https://macvendors.com or tools like Wireshark
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **MAC Address** | 48-bit hardware identifier, unique per NIC |
| **OUI** | First 3 bytes identify the vendor |
| **CAM Table** | Switch maps MAC addresses to ports |
| **MAC Spoofing** | Trivial to change — MAC filtering is weak security |
| **Security Role** | Layer 2 identification, ARP, port security |

---

## Quick Revision Questions

1. **What is a MAC address and how many bits does it have?**
2. **What is the OUI and what can it reveal?**
3. **How does a switch use its CAM table?**
4. **Why is MAC address filtering NOT strong security?**
5. **How can an attacker spoof a MAC address?**
6. **What happens when a switch receives a frame for an unknown MAC?**

---

[Next: ARP Protocol and ARP Spoofing →](02-arp-protocol-and-spoofing.md)

---

*Unit 2 - Topic 1 of 6 | [Back to README](../README.md)*
