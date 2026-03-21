# Wireless Security Best Practices

## Unit 9 - Topic 5 | Wireless Networks

---

## Overview

Securing wireless networks requires a **layered approach** combining strong encryption, proper authentication, network architecture, monitoring, and user education. This topic consolidates all wireless security knowledge into actionable best practices for enterprise and home environments.

---

## 1. Enterprise Wireless Security Checklist

| Category | Best Practice |
|----------|--------------|
| **Encryption** | Use WPA3-Enterprise (or WPA2-Enterprise minimum) |
| **Authentication** | 802.1X with RADIUS — EAP-TLS (certificate-based) |
| **Passwords** | If PSK required, use 20+ character random passphrase |
| **Management Frames** | Enable 802.11w (PMF) — mandatory for WPA3 |
| **SSID** | Don't hide SSID (offers no real security), use descriptive name |
| **Guest Network** | Separate VLAN with internet-only access |
| **Segmentation** | Wireless on dedicated VLAN, isolated from sensitive resources |
| **Monitoring** | Deploy WIDS/WIPS for rogue AP detection |
| **AP Placement** | Position APs to minimize signal leakage outside building |
| **Firmware** | Keep AP firmware updated |

---

## 2. Network Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              SECURE WIRELESS ARCHITECTURE                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐      ┌──────────────┐      ┌────────────┐    │
│  │ Corporate│      │   FIREWALL   │      │  Internet  │    │
│  │ Network  │◄────►│   / ACLs     │◄────►│            │    │
│  │ (VLAN 10)│      └──────┬───────┘      └────────────┘    │
│  └──────────┘             │                                  │
│                    ┌──────┴───────┐                          │
│                    │   WIRELESS   │                          │
│                    │  CONTROLLER  │                          │
│                    └──────┬───────┘                          │
│               ┌───────────┼───────────┐                      │
│               ▼           ▼           ▼                      │
│         ┌──────────┐┌──────────┐┌──────────┐                │
│         │Corp WiFi ││Guest WiFi││IoT WiFi  │                │
│         │VLAN 20   ││VLAN 30   ││VLAN 40   │                │
│         │WPA3-Ent  ││Captive   ││Isolated  │                │
│         │802.1X    ││Portal    ││Restricted│                │
│         └──────────┘└──────────┘└──────────┘                │
│                                                              │
│  RULES:                                                      │
│  • Corp WiFi → Full access (authenticated users only)       │
│  • Guest WiFi → Internet only (no internal access)          │
│  • IoT WiFi → Specific services only (heavily restricted)   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Common Mistakes to Avoid

| Mistake | Why It's Bad | What to Do Instead |
|---------|-------------|-------------------|
| Using WEP or WPA-TKIP | Easily cracked | Use WPA2-AES or WPA3 |
| Hiding SSID | Easily discovered, causes client issues | Use strong encryption instead |
| MAC filtering only | MACs easily spoofed | Use 802.1X authentication |
| Short/weak PSK | Dictionary attack in hours | 20+ random characters |
| Same password for years | More time to crack, shared widely | Rotate quarterly or use 802.1X |
| No guest network | Guests on corporate network | Separate VLAN for guests |
| No monitoring | Can't detect rogue APs or attacks | Deploy WIDS/WIPS |
| Default AP credentials | Attacker reconfigures AP | Change immediately |

---

## 4. Wireless Security Monitoring

| What to Monitor | Tools |
|----------------|-------|
| **Rogue AP detection** | Kismet, AirMagnet, Cisco WIPS |
| **Deauth attacks** | WIDS with deauth alerting |
| **Client activity** | RADIUS accounting logs |
| **Signal strength** | RF heat mapping |
| **Failed authentications** | RADIUS/SIEM integration |
| **New devices** | NAC, 802.1X logs |

---

## 5. Home Wireless Security

| Best Practice | Implementation |
|--------------|----------------|
| Use WPA3 (or WPA2-AES) | Change in router settings |
| Strong password | 20+ characters, random |
| Change default admin credentials | Router admin page |
| Disable WPS | Often has PIN brute-force vulnerability |
| Update firmware | Check manufacturer website |
| Disable remote management | Prevent external access to router |
| Enable firewall | Usually built into router |
| Separate IoT devices | Use guest network for IoT |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Encryption** | WPA3 (preferred) or WPA2-AES (minimum) |
| **Authentication** | 802.1X/EAP-TLS for enterprise |
| **Segmentation** | Separate VLANs for corp, guest, IoT |
| **Monitoring** | WIDS/WIPS for rogue AP and attack detection |
| **PMF** | Enable 802.11w to protect management frames |
| **Avoid** | WEP, hidden SSID, MAC filtering as sole defense |

---

## Quick Revision Questions

1. **What is the recommended wireless security protocol for enterprises?**
2. **Why is hiding the SSID not an effective security measure?**
3. **How should guest wireless access be configured?**
4. **Why should WPS be disabled?**
5. **What three VLANs should a secure wireless architecture include?**

---

[← Previous: Rogue Access Points](04-rogue-access-points.md)

---

*Unit 9 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Network Segmentation →](../10-Network-Segmentation/01-micro-segmentation.md)*
