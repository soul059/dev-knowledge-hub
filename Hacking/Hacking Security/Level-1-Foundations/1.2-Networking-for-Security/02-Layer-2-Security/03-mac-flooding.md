# MAC Flooding

## Unit 2 - Topic 3 | Layer 2 Security

---

## Overview

**MAC flooding** is a Layer 2 attack that overwhelms a switch's **CAM (Content Addressable Memory) table** with thousands of fake MAC addresses. When the CAM table is full, the switch can no longer learn new MAC-to-port mappings and falls back to **hub mode** — broadcasting all traffic to all ports, allowing an attacker to sniff all network traffic.

---

## 1. How MAC Flooding Works

```
┌──────────────────────────────────────────────────────────────────┐
│                  MAC FLOODING ATTACK                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  NORMAL OPERATION:                                               │
│  Switch knows which MAC is on which port → unicast forwarding   │
│  Traffic only goes to the intended recipient.                   │
│                                                                  │
│  ┌────────────────────────────┐                                 │
│  │ CAM Table (8,000 entries)  │                                 │
│  │ MAC AA:AA → Port 1        │                                 │
│  │ MAC BB:BB → Port 2        │                                 │
│  │ MAC CC:CC → Port 3        │                                 │
│  │ (space for more...)       │                                 │
│  └────────────────────────────┘                                 │
│                                                                  │
│  ATTACK:                                                         │
│  Attacker floods switch with FAKE random MAC addresses          │
│                                                                  │
│  ┌────────────────────────────┐                                 │
│  │ CAM Table (8,000 entries)  │                                 │
│  │ FAKE1 → Port 4            │                                 │
│  │ FAKE2 → Port 4            │  ← ALL 8,000 entries filled     │
│  │ FAKE3 → Port 4            │     with fake MACs from         │
│  │ ...                       │     attacker on Port 4          │
│  │ FAKE8000 → Port 4         │                                 │
│  └────────────────────────────┘                                 │
│                                                                  │
│  RESULT: Table full → switch becomes a HUB                      │
│  ALL traffic broadcast to ALL ports → Attacker sees EVERYTHING  │
│                                                                  │
│  ┌───┐  ┌───┐  ┌───┐  ┌──────────┐                            │
│  │PC1│  │PC2│  │PC3│  │ ATTACKER │                             │
│  └─┬─┘  └─┬─┘  └─┬─┘  └────┬─────┘                            │
│    │      │      │         │                                    │
│  ┌─┴──────┴──────┴─────────┴──┐                                │
│  │    SWITCH (in HUB mode)    │  Every frame goes EVERYWHERE   │
│  └────────────────────────────┘                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Attack Tools

```bash
# macof (dsniff suite) — Primary MAC flooding tool
macof -i eth0                    # Flood with random MACs
macof -i eth0 -n 100000         # Send 100,000 fake MACs

# Yersinia — Advanced Layer 2 attack tool
yersinia -I                      # Interactive mode
# Select CDP/STP/DTP/DHCP attacks

# Scapy (Python) — Custom MAC flooding
from scapy.all import *
def mac_flood():
    while True:
        pkt = Ether(src=RandMAC(), dst=RandMAC()) / \
              IP(src=RandIP(), dst=RandIP())
        sendp(pkt, iface="eth0", verbose=0)
```

---

## 3. Impact of MAC Flooding

| Impact | Description |
|--------|-------------|
| **Traffic Sniffing** | Attacker sees all traffic on the switch segment |
| **Credential Capture** | Cleartext passwords, cookies intercepted |
| **Network Degradation** | Switch performance drops significantly |
| **DoS Effect** | High volume of broadcast traffic overwhelms network |
| **Follow-up Attacks** | Enables ARP spoofing, session hijacking |

---

## 4. Defense: Port Security

```
┌──────────────────────────────────────────────────────────────┐
│              PORT SECURITY (Primary Defense)                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Limits the number of MAC addresses per switch port.         │
│  If exceeded → port shuts down or restricts traffic.         │
│                                                              │
│  Cisco Configuration:                                        │
│  interface GigabitEthernet0/1                                │
│    switchport mode access                                    │
│    switchport port-security                                  │
│    switchport port-security maximum 2                        │
│    switchport port-security violation shutdown               │
│    switchport port-security mac-address sticky               │
│                                                              │
│  VIOLATION MODES:                                            │
│  ┌───────────┬─────────────────────────────────────┐         │
│  │ Mode      │ Action                              │         │
│  ├───────────┼─────────────────────────────────────┤         │
│  │ Protect   │ Drops unknown MACs, no log          │         │
│  │ Restrict  │ Drops + logs + increments counter   │         │
│  │ Shutdown  │ Disables port entirely (default) ✅  │         │
│  └───────────┴─────────────────────────────────────┘         │
│                                                              │
│  "sticky" = learns MACs dynamically and saves them           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Additional Defenses

| Defense | Description |
|---------|-------------|
| **Port Security** | Limit MAC addresses per port (primary defense) |
| **802.1X** | Authenticate devices before granting port access |
| **DHCP Snooping** | Validates DHCP and builds trusted MAC-IP database |
| **Storm Control** | Limits broadcast/multicast/unicast flooding rate |
| **Private VLANs** | Isolate hosts within same VLAN |
| **Network Monitoring** | IDS alerts on unusual MAC activity |

### Storm Control Configuration

```cisco
! Limit broadcast traffic to 20% of port bandwidth
interface GigabitEthernet0/1
  storm-control broadcast level 20
  storm-control action shutdown
```

---

## 6. Detection

```bash
# Check switch port-security status (Cisco)
show port-security
show port-security interface gi0/1
show port-security address

# Check for violation
show port-security interface gi0/1
# Look for: "Security Violation Count" > 0
# Look for: "Port Status: Secure-shutdown"

# Recovery after shutdown
interface GigabitEthernet0/1
  shutdown
  no shutdown
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **MAC Flooding** | Overflows CAM table → switch acts as hub |
| **Tool** | macof (dsniff suite) |
| **Impact** | Attacker sniffs all traffic on the segment |
| **Primary Defense** | Port security (limit MACs per port) |
| **Best Violation Mode** | Shutdown (disables port completely) |
| **Additional** | 802.1X, storm control, DHCP snooping |

---

## Quick Revision Questions

1. **What happens when a switch's CAM table is full?**
2. **How does MAC flooding enable traffic sniffing?**
3. **What tool is commonly used for MAC flooding?**
4. **How does port security prevent MAC flooding?**
5. **What are the three port security violation modes?**
6. **What is storm control and how does it help?**

---

[← Previous: ARP Protocol & Spoofing](02-arp-protocol-and-spoofing.md) | [Next: VLAN Hopping →](04-vlan-hopping.md)

---

*Unit 2 - Topic 3 of 6 | [Back to README](../README.md)*
