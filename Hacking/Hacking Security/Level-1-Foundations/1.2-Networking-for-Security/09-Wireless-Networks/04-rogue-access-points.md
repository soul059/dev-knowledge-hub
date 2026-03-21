# Rogue Access Points

## Unit 9 - Topic 4 | Wireless Networks

---

## Overview

A **rogue access point (rogue AP)** is an unauthorized wireless access point connected to a network without the knowledge of the network administrator. Rogue APs can be deployed by **attackers** (for MITM attacks) or by **employees** (for convenience), both creating serious security vulnerabilities.

---

## 1. Types of Rogue APs

```
┌──────────────────────────────────────────────────────────────┐
│              ROGUE AP TYPES                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. UNAUTHORIZED EMPLOYEE AP:                                │
│     Employee plugs personal AP into corporate network        │
│     → Bypasses all wireless security controls                │
│     → No encryption, no authentication, no monitoring        │
│                                                              │
│  2. ATTACKER-DEPLOYED ROGUE AP:                              │
│     Attacker hides a small AP (e.g., Raspberry Pi)           │
│     inside the building, connected to corporate network      │
│     → Provides attacker remote wireless access               │
│                                                              │
│  3. EVIL TWIN:                                               │
│     Attacker creates AP with same SSID as legitimate         │
│     → Captures credentials and traffic                       │
│     (Not physically connected to target network)             │
│                                                              │
│  4. SOFTWARE AP:                                             │
│     Compromised laptop/device acting as AP (hostapd)         │
│     → Bridges wireless clients to wired network              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Rogue AP Threats

| Threat | Description |
|--------|-------------|
| **Bypasses perimeter security** | Provides wireless entry past firewalls |
| **Man-in-the-Middle** | All traffic through attacker-controlled AP |
| **Credential harvesting** | Captive portal captures passwords |
| **Lateral movement** | Bridge from wireless to wired network |
| **Data exfiltration** | Wireless channel to extract data |
| **Persistent access** | Hidden device maintains access even after patching |

---

## 3. Detection Methods

| Method | How It Works |
|--------|-------------|
| **Wireless IDS (WIDS)** | Monitors airspace for unauthorized APs |
| **802.1X/NAC** | Blocks unauthorized devices on switch ports |
| **DHCP monitoring** | Detect new DHCP servers (rogue AP often runs DHCP) |
| **Physical inspection** | Walk the facility checking for unknown devices |
| **Wired-side detection** | Monitor for MAC addresses not in inventory |
| **RF scanning** | Regular wireless surveys of facility |
| **Switch port security** | Limit MACs per port, shut on violation |

```bash
# Detection tools:
kismet                          # Wireless network detector
airodump-ng wlan0mon            # List all nearby APs
nmap -sP 10.0.0.0/24           # Find new devices on network
```

---

## 4. Prevention

| Control | Implementation |
|---------|---------------|
| **802.1X port authentication** | Only authorized devices can connect to switch |
| **NAC (Network Access Control)** | Validate device before network access |
| **Wireless IDS/IPS** | Detect and contain rogue APs automatically |
| **MAC filtering on switches** | Restrict known MACs per port |
| **Security awareness training** | Educate employees about rogue AP risks |
| **Regular wireless surveys** | Scheduled scans of the RF environment |
| **VLAN isolation** | Limit damage if rogue AP connects |
| **Disable unused switch ports** | Reduce physical connection opportunities |

---

## 5. Rogue AP Attack Scenario

```
ATTACK FLOW:
1. Attacker visits target office (interview, delivery, etc.)
2. Hides Raspberry Pi with battery + LTE behind printer
3. Pi connects to corporate Ethernet port
4. Pi broadcasts hidden SSID for attacker
5. Attacker parks outside, connects to hidden AP
6. Full access to internal network via wireless bridge
7. Can persist for weeks on battery + solar

DETECTION:
• 802.1X on all switch ports → Pi can't authenticate
• NAC → Unknown device blocked
• WIDS → Detects new AP in airspace
• Switch port security → Port shuts down on unknown MAC
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Rogue AP** | Unauthorized AP connected to network |
| **Employee Rogue** | Convenience AP that bypasses security |
| **Attacker Rogue** | Hidden device for persistent access |
| **Evil Twin** | Fake AP mimicking legitimate network |
| **Detection** | WIDS, 802.1X, NAC, physical surveys |
| **Prevention** | Port security, NAC, WIDS, awareness training |

---

## Quick Revision Questions

1. **What is a rogue access point?**
2. **How does a rogue AP differ from an evil twin?**
3. **What methods can detect rogue APs?**
4. **How does 802.1X help prevent rogue APs?**
5. **Describe a rogue AP attack scenario.**

---

[← Previous: Wireless Attacks](03-wireless-attacks.md) | [Next: Wireless Security Best Practices →](05-wireless-security-best-practices.md)

---

*Unit 9 - Topic 4 of 5 | [Back to README](../README.md)*
