# ARP Protocol and ARP Spoofing

## Unit 2 - Topic 2 | Layer 2 Security

---

## Overview

**ARP (Address Resolution Protocol)** maps IP addresses to MAC addresses on a local network. Because ARP has **no authentication**, it's vulnerable to **ARP spoofing** (also called ARP poisoning) — one of the most common and powerful Layer 2 attacks, enabling man-in-the-middle (MITM) interception of network traffic.

---

## 1. How ARP Works

```
┌──────────────────────────────────────────────────────────────────┐
│                  ARP RESOLUTION PROCESS                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Alice (10.0.0.5) wants to communicate with Bob (10.0.0.10)     │
│  Alice knows Bob's IP but NOT his MAC address.                  │
│                                                                  │
│  STEP 1: ARP Request (Broadcast)                                │
│  Alice → FF:FF:FF:FF:FF:FF (everyone on LAN):                   │
│  "Who has 10.0.0.10? Tell 10.0.0.5"                            │
│                                                                  │
│  ┌───────┐          ┌───────┐          ┌───────┐               │
│  │ Alice │──BROAD──►│  ALL  │◄─────────│  Bob  │               │
│  │10.0.0.5│  CAST   │devices│          │10.0.0.10│              │
│  └───────┘          └───────┘          └───┬───┘               │
│                                            │                    │
│  STEP 2: ARP Reply (Unicast)               │                    │
│  Bob → Alice:                              │                    │
│  "10.0.0.10 is at MAC BB:BB:BB:BB:BB:BB"  │                    │
│                                            │                    │
│  ┌───────┐◄───────────────────────────────┘                    │
│  │ Alice │                                                      │
│  │ ARP   │ Caches: 10.0.0.10 → BB:BB:BB:BB:BB:BB              │
│  │ Cache │                                                      │
│  └───────┘                                                      │
│                                                                  │
│  NOW Alice can send frames directly to Bob's MAC.               │
│                                                                  │
│  ⚠️ PROBLEM: ARP has NO AUTHENTICATION                          │
│  Anyone can send ARP replies — even unsolicited ones!           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. ARP Spoofing / Poisoning Attack

```
┌──────────────────────────────────────────────────────────────────┐
│                  ARP SPOOFING ATTACK                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BEFORE ATTACK (Normal):                                         │
│  Alice ──────────────────────────► Gateway                      │
│  10.0.0.5                          10.0.0.1                     │
│                                                                  │
│  ATTACK:                                                         │
│  Attacker sends FAKE ARP replies to BOTH Alice and Gateway:     │
│                                                                  │
│  To Alice:   "10.0.0.1 (Gateway) is at ATTACKER's MAC"         │
│  To Gateway: "10.0.0.5 (Alice) is at ATTACKER's MAC"           │
│                                                                  │
│  AFTER ATTACK (MITM):                                            │
│  Alice ───► ATTACKER ───► Gateway ───► Internet                 │
│       (thinks attacker        (thinks attacker                   │
│        is the gateway)         is Alice)                         │
│                                                                  │
│  ┌───────┐         ┌──────────┐         ┌─────────┐            │
│  │ Alice │────────►│ ATTACKER │────────►│ Gateway │            │
│  │       │◄────────│  (MITM)  │◄────────│         │            │
│  └───────┘         └──────────┘         └─────────┘            │
│                    ↑                                            │
│                    Reads, modifies, or                           │
│                    records ALL traffic                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### ARP Spoofing Tools

```bash
# arpspoof (dsniff suite) — Linux
echo 1 > /proc/sys/net/ipv4/ip_forward     # Enable forwarding
arpspoof -i eth0 -t 10.0.0.5 10.0.0.1     # Tell Alice: I'm the gateway
arpspoof -i eth0 -t 10.0.0.1 10.0.0.5     # Tell Gateway: I'm Alice

# Ettercap — GUI/CLI MITM tool
ettercap -T -M arp:remote /10.0.0.5// /10.0.0.1//

# Bettercap — Modern MITM framework
bettercap -iface eth0
> set arp.spoof.targets 10.0.0.5
> arp.spoof on
> net.sniff on
```

---

## 3. What Attackers Can Do After ARP Spoofing

| Capability | Description |
|-----------|-------------|
| **Sniff Traffic** | Read all unencrypted traffic (passwords, emails) |
| **SSL Stripping** | Downgrade HTTPS to HTTP |
| **DNS Spoofing** | Redirect domain lookups to malicious servers |
| **Session Hijacking** | Steal authentication cookies/tokens |
| **Credential Theft** | Capture login credentials in cleartext |
| **Data Modification** | Alter data in transit |

---

## 4. Defenses Against ARP Spoofing

| Defense | Description |
|---------|-------------|
| **Dynamic ARP Inspection (DAI)** | Switch validates ARP packets against DHCP snooping table |
| **Static ARP Entries** | Manually set IP-MAC mappings (impractical at scale) |
| **DHCP Snooping** | Validates DHCP traffic, builds trusted IP-MAC bindings |
| **802.1X (Port Authentication)** | Authenticates devices before network access |
| **Use Encryption** | HTTPS, SSH, VPN — ARP spoof can't read encrypted data |
| **ARP Monitoring Tools** | arpwatch, XArp — detect ARP anomalies |
| **Network Segmentation** | Limits scope of ARP attacks to single VLAN |

### Cisco DAI Configuration

```
! Enable DHCP Snooping (prerequisite for DAI)
ip dhcp snooping
ip dhcp snooping vlan 10,20

! Trust uplink port (to DHCP server/router)
interface GigabitEthernet0/24
  ip dhcp snooping trust

! Enable Dynamic ARP Inspection
ip arp inspection vlan 10,20

! Trust uplink port for ARP inspection
interface GigabitEthernet0/24
  ip arp inspection trust
```

---

## 5. Detecting ARP Spoofing

```bash
# Check ARP cache for duplicates
arp -a | sort                  # Look for duplicate MACs

# Linux — Monitor ARP changes
arpwatch -i eth0               # Alerts on ARP changes

# Wireshark filter for ARP
arp                            # Show all ARP traffic
arp.duplicate-address-detected # Detect duplicate MACs

# Signs of ARP spoofing:
# ✅ Multiple IPs mapped to same MAC
# ✅ Gateway MAC suddenly changes
# ✅ Unusual ARP broadcast storms
# ✅ Duplicate IP/MAC warnings
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **ARP** | Maps IP addresses to MAC addresses on LAN |
| **Vulnerability** | No authentication — any device can send fake ARP replies |
| **ARP Spoofing** | MITM attack by poisoning ARP caches |
| **Impact** | Traffic interception, credential theft, session hijacking |
| **Best Defense** | DAI + DHCP Snooping on switches |
| **User Defense** | Use encrypted protocols (HTTPS, SSH, VPN) |

---

## Quick Revision Questions

1. **What does ARP do and why is it necessary?**
2. **Why is ARP vulnerable to spoofing attacks?**
3. **Describe how an ARP spoofing MITM attack works step by step.**
4. **What is Dynamic ARP Inspection (DAI) and how does it work?**
5. **What tools can be used to perform ARP spoofing?**
6. **How can you detect ARP spoofing on your network?**

---

[← Previous: MAC Addresses](01-mac-addresses.md) | [Next: MAC Flooding →](03-mac-flooding.md)

---

*Unit 2 - Topic 2 of 6 | [Back to README](../README.md)*
