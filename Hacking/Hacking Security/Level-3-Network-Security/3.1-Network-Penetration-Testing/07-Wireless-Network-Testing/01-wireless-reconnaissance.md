# Wireless Reconnaissance

## Unit 7 - Topic 1 | Wireless Network Testing

---

## Overview

**Wireless reconnaissance** identifies and maps wireless networks in the target environment. This includes discovering SSIDs, access points, encryption types, channels, client devices, and hidden networks — essential intelligence before any wireless attack.

---

## 1. Wireless Adapter Setup

```bash
# === MONITOR MODE (Required for wireless testing) ===
# Check wireless interface:
iwconfig
ip link show

# Kill interfering processes:
sudo airmon-ng check kill

# Enable monitor mode:
sudo airmon-ng start wlan0
# Creates wlan0mon interface

# Verify monitor mode:
iwconfig wlan0mon
# Mode: Monitor ✅

# === RECOMMENDED ADAPTERS ===
# Must support monitor mode + packet injection:
# ├── Alfa AWUS036ACH (AC, 2.4/5GHz)
# ├── Alfa AWUS036AXML (AXE, Wi-Fi 6E)
# ├── Alfa AWUS1900 (AC, 4 antennas)
# ├── TP-Link TL-WN722N v1 (N, budget)
# └── Panda PAU09 (N, dual-band)

# Check chipset (Atheros/Realtek preferred):
lsusb
# Check injection support:
aireplay-ng --test wlan0mon
```

---

## 2. Passive Scanning

```bash
# === AIRODUMP-NG (Primary Scanner) ===
sudo airodump-ng wlan0mon
# Discovers all nearby networks

# OUTPUT COLUMNS:
# BSSID    — AP MAC address
# PWR      — Signal strength (higher = closer)
# Beacons  — Number of beacon frames
# #Data    — Number of data packets
# CH       — Channel
# MB       — Maximum speed
# ENC      — Encryption (WPA2, WEP, OPN)
# CIPHER   — Cipher (CCMP, TKIP)
# AUTH     — Authentication (PSK, MGT)
# ESSID    — Network name

# Focus on specific channel:
sudo airodump-ng wlan0mon -c 6

# Focus on specific network:
sudo airodump-ng wlan0mon -c 6 --bssid AA:BB:CC:DD:EE:FF

# Save output:
sudo airodump-ng wlan0mon -w scan_results --output-format csv,pcap

# === 5GHz SCANNING ===
sudo airodump-ng wlan0mon --band a      # 5GHz only
sudo airodump-ng wlan0mon --band abg    # All bands
```

---

## 3. Active Scanning and Discovery

```bash
# === PROBE REQUESTS (Discover Client Networks) ===
# Clients broadcast probe requests for known networks
# Passive collection reveals networks users connect to

# === HIDDEN NETWORK DISCOVERY ===
# Hidden SSIDs show as <length: 0> in airodump-ng
# Reveal by:
# 1. Wait for client to connect (probe request)
# 2. Deauth a connected client (forces reconnect)

sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF wlan0mon
# Client reconnects → SSID revealed in probe/association

# === WASH (WPS Detection) ===
wash -i wlan0mon
# Lists APs with WPS enabled
# Shows: WPS version, locked status

# === KISMET (Advanced Wireless Scanner) ===
sudo kismet -c wlan0mon
# Web UI: http://localhost:2501
# Features: GPS mapping, IDS, multi-protocol
# Supports: WiFi, Bluetooth, Zigbee, etc.

# === WIFITE (Automated Scanner + Attacker) ===
sudo wifite --kill
# Scans and presents attack options
# Automated WPA/WPA2/WPS attacks
```

---

## 4. Information to Gather

| Information | Why Important |
|-------------|--------------|
| **SSID** | Network name — target identification |
| **BSSID** | AP MAC — unique identifier |
| **Encryption** | WEP/WPA/WPA2/WPA3 — determines attack |
| **Channel** | Required for focused attacks |
| **Clients** | Connected devices — deauth targets |
| **Hidden SSIDs** | Indicates security awareness |
| **WPS status** | If enabled — PIN brute force possible |
| **Signal strength** | Physical proximity estimation |
| **Vendor** | AP manufacturer (OUI lookup) |
| **Enterprise auth** | 802.1X — evil twin potential |

---

## 5. Wireless Network Mapping

```
WIRELESS RECON REPORT EXAMPLE:

TARGET ENVIRONMENT:
├── Corporate Network
│   ├── SSID: CORP-WIFI (WPA2-Enterprise, 802.1X)
│   │   ├── Channel: 36 (5GHz)
│   │   ├── BSSID: AA:BB:CC:DD:EE:01
│   │   ├── Clients: 47 connected
│   │   └── Encryption: AES/CCMP ← Hardest target
│   │
│   ├── SSID: GUEST-WIFI (WPA2-PSK)
│   │   ├── Channel: 6 (2.4GHz)
│   │   ├── BSSID: AA:BB:CC:DD:EE:02
│   │   ├── Clients: 12 connected
│   │   └── Encryption: AES/CCMP ← PSK crackable
│   │
│   └── SSID: <hidden> (WPA2-PSK)
│       ├── Channel: 11
│       ├── BSSID: AA:BB:CC:DD:EE:03
│       ├── Clients: 3 connected
│       └── Note: Likely management/IoT network
│
└── Nearby Networks (not in scope)
    └── 15+ residential networks detected
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Monitor mode** | Required for passive wireless capture |
| **airodump-ng** | Primary wireless scanning tool |
| **Hidden SSIDs** | Revealed via deauth + probe capture |
| **WPS** | Wi-Fi Protected Setup — often bruteforceable |
| **Kismet** | Advanced multi-protocol wireless scanner |
| **Key info** | SSID, BSSID, encryption, channel, clients |

---

## Quick Revision Questions

1. **Why is monitor mode required for wireless testing?**
2. **How do you discover hidden wireless networks?**
3. **What information does airodump-ng display?**
4. **Why is WPS detection important during recon?**
5. **What wireless adapter chipsets support monitor mode?**

---

[Next: WPA/WPA2 Testing →](02-wpa-wpa2-testing.md)

---

*Unit 7 - Topic 1 of 6 | [Back to README](../README.md)*
