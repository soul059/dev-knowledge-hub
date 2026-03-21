# 802.11 Standards

## Unit 9 - Topic 1 | Wireless Networks

---

## Overview

The **IEEE 802.11** family of standards defines wireless LAN (Wi-Fi) communication. Understanding these standards is critical for security professionals because **each generation introduced new security features** while older standards contain known vulnerabilities that attackers exploit.

---

## 1. 802.11 Standards Evolution

| Standard | Year | Frequency | Max Speed | Range (Indoor) | Key Feature |
|----------|:----:|:---------:|:---------:|:--------------:|-------------|
| **802.11a** | 1999 | 5 GHz | 54 Mbps | ~35m | First 5 GHz standard |
| **802.11b** | 1999 | 2.4 GHz | 11 Mbps | ~38m | First widely adopted |
| **802.11g** | 2003 | 2.4 GHz | 54 Mbps | ~38m | Backward compatible with b |
| **802.11n (Wi-Fi 4)** | 2009 | 2.4/5 GHz | 600 Mbps | ~70m | MIMO, channel bonding |
| **802.11ac (Wi-Fi 5)** | 2013 | 5 GHz | 6.9 Gbps | ~35m | MU-MIMO, beamforming |
| **802.11ax (Wi-Fi 6)** | 2019 | 2.4/5 GHz | 9.6 Gbps | ~35m | OFDMA, WPA3, BSS coloring |
| **802.11be (Wi-Fi 7)** | 2024 | 2.4/5/6 GHz | 46 Gbps | ~35m | MLO, 320 MHz channels |

---

## 2. Frequency Bands

```
┌──────────────────────────────────────────────────────────────┐
│              WIRELESS FREQUENCY BANDS                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  2.4 GHz Band:                                               │
│  ├── Channels 1-14 (only 1, 6, 11 non-overlapping)          │
│  ├── Longer range, better wall penetration                   │
│  ├── More interference (Bluetooth, microwaves)               │
│  └── Crowded — fewer non-overlapping channels                │
│                                                              │
│  5 GHz Band:                                                 │
│  ├── More channels (23+ non-overlapping)                     │
│  ├── Shorter range, less wall penetration                    │
│  ├── Less interference                                       │
│  └── DFS channels (shared with radar)                        │
│                                                              │
│  6 GHz Band (Wi-Fi 6E/7):                                    │
│  ├── 59 additional channels                                  │
│  ├── Requires WPA3 (no legacy support)                       │
│  └── Least interference, newest spectrum                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Key Wireless Concepts

| Concept | Description |
|---------|-------------|
| **SSID** | Service Set Identifier — network name |
| **BSSID** | MAC address of the access point |
| **Channel** | Specific frequency band for communication |
| **MIMO** | Multiple Input Multiple Output — multiple antennas |
| **Beamforming** | Directs signal toward specific clients |
| **OFDMA** | Allows multiple users to share a channel simultaneously |
| **BSS Coloring** | Reduces interference between nearby APs |

---

## 4. 802.11 Frame Types

| Frame Type | Purpose | Security Relevance |
|-----------|---------|-------------------|
| **Management** | Association, authentication, beacons | Deauth attacks target these |
| **Control** | RTS, CTS, ACK — flow control | Can be spoofed |
| **Data** | Actual payload transmission | Contains encrypted/unencrypted data |

```
Management Frames (CRITICAL for security):
├── Beacon         — AP announces its presence
├── Probe Request  — Client searches for networks
├── Probe Response — AP responds to probe
├── Authentication — Client authenticates
├── Association    — Client joins network
├── Deauthentication — Disconnect (spoofable!)
└── Disassociation — Leave network (spoofable!)

⚠️ Management frames were UNPROTECTED until 802.11w (PMF)
```

---

## 5. Security Implications per Standard

| Standard | Default Security | Vulnerability |
|----------|:---:|------|
| **802.11b/g** | WEP | Completely broken, crackable in minutes |
| **802.11n** | WPA/WPA2 | WPA-TKIP has weaknesses |
| **802.11ac** | WPA2 | KRACK attack (patched) |
| **802.11ax** | WPA3 | Dragonblood (mostly patched) |
| **802.11w** | PMF | Protected Management Frames (prevents deauth) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **802.11** | IEEE wireless LAN standard family |
| **2.4 GHz** | Longer range, more interference, 3 non-overlapping channels |
| **5 GHz** | Shorter range, less interference, more channels |
| **Wi-Fi 6** | Latest widely deployed, supports WPA3 |
| **Management Frames** | Unprotected without 802.11w (PMF) |
| **802.11w** | Protected Management Frames — prevents deauth attacks |

---

## Quick Revision Questions

1. **What are the three non-overlapping channels in the 2.4 GHz band?**
2. **Which 802.11 standard introduced WPA3 support?**
3. **What is the security significance of 802.11w?**
4. **What are the three types of 802.11 frames?**
5. **Why is the 5 GHz band preferred for security?**

---

[Next: Wireless Security Protocols →](02-wireless-security-protocols.md)

---

*Unit 9 - Topic 1 of 5 | [Back to README](../README.md)*
