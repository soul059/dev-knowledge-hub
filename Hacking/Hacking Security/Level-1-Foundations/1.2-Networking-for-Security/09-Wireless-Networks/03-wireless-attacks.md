# Wireless Attacks

## Unit 9 - Topic 3 | Wireless Networks

---

## Overview

Wireless networks are inherently more vulnerable than wired networks because the **radio signals travel through the air** — any device within range can intercept traffic. Wireless attacks target the **encryption protocols**, **management frames**, **access points**, and **clients**.

---

## 1. Attack Categories

```
┌──────────────────────────────────────────────────────────────┐
│              WIRELESS ATTACK TAXONOMY                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  RECONNAISSANCE:                                             │
│  ├── Wardriving (scanning for networks)                      │
│  ├── Passive sniffing (capturing traffic)                    │
│  └── Identifying targets and security posture                │
│                                                              │
│  AUTHENTICATION ATTACKS:                                     │
│  ├── WEP cracking                                            │
│  ├── WPA/WPA2 handshake capture + dictionary                 │
│  ├── PMKID attack (no handshake needed)                      │
│  └── Evil twin (fake AP)                                     │
│                                                              │
│  DENIAL OF SERVICE:                                          │
│  ├── Deauthentication flood                                  │
│  ├── RF jamming                                              │
│  └── Authentication flood                                    │
│                                                              │
│  MAN-IN-THE-MIDDLE:                                          │
│  ├── Evil twin with captive portal                           │
│  ├── KARMA attack                                            │
│  └── SSL stripping on wireless                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Key Wireless Attacks

### Deauthentication Attack

```
PURPOSE: Force client to disconnect and reconnect
         (used to capture WPA handshake or as DoS)

Attacker sends spoofed deauth frames:
┌──────────┐  Deauth (spoofed)  ┌─────┐
│ Attacker │──────────────────►│ AP  │
└──────────┘                    └──┬──┘
     │                             │
     │  Deauth (spoofed)           │ Client disconnected!
     └──────────────────►┌────────┐│
                         │ Client │◄┘ Reconnects → handshake captured!
                         └────────┘

# aireplay-ng deauth attack:
aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0mon      # Deauth all
aireplay-ng -0 5 -a AA:BB:CC -c 11:22:33 wlan0mon     # Deauth specific client

# DEFENSE: 802.11w (Protected Management Frames / PMF)
```

### Evil Twin Attack

```
SETUP: Create fake AP with same SSID as target

┌─────────┐ "FreeWiFi"    ┌──────────┐
│ Real AP │ ◄────────────  │  Client  │ (auto-connects to strongest)
└─────────┘                └────┬─────┘
                                │
┌──────────────┐ "FreeWiFi"     │
│ Evil Twin AP │ ◄──────────────┘ (connects to attacker instead!)
│ (Attacker)   │
└──────┬───────┘
       │
   Intercept all traffic, serve phishing pages,
   capture credentials

# Tools: hostapd-wpe, Fluxion, WiFi-Pumpkin
```

### PMKID Attack (WPA2)

```
# No need to wait for handshake — request PMKID from AP
hcxdumptool -i wlan0mon --enable_status=1 -o capture.pcapng
hcxpcapngtool -o hash.hc22000 capture.pcapng
hashcat -m 22000 hash.hc22000 wordlist.txt

# Advantage: No client needed — just AP
# Works against most WPA2 routers with PMKID enabled
```

### KARMA Attack

```
Attacker's AP responds to ALL probe requests:

Client: "Is 'HomeWiFi' available?"
KARMA AP: "Yes, I am 'HomeWiFi'!"

Client: "Is 'OfficeNet' available?"  
KARMA AP: "Yes, I am 'OfficeNet' too!"

→ Client auto-connects to attacker's AP
→ All traffic intercepted

# Tools: hostapd-mana, WiFi-Pumpkin
```

---

## 3. Wireless Attack Tools

| Tool | Purpose |
|------|---------|
| **aircrack-ng suite** | WEP/WPA cracking, packet injection, monitoring |
| **Kismet** | Wireless network detector, sniffer, IDS |
| **Wireshark** | Packet capture and analysis |
| **hashcat** | GPU-based password cracking |
| **Fluxion** | Automated evil twin + captive portal |
| **WiFi-Pumpkin** | Rogue AP framework |
| **Bettercap** | MITM framework with wireless modules |
| **hcxtools** | PMKID capture and conversion |

---

## 4. Wireless Pentesting Methodology

| Step | Action | Tool |
|:----:|--------|------|
| 1 | Enable monitor mode | `airmon-ng start wlan0` |
| 2 | Scan for networks | `airodump-ng wlan0mon` |
| 3 | Target specific network | `airodump-ng -c CH --bssid MAC -w cap wlan0mon` |
| 4 | Force handshake (deauth) | `aireplay-ng -0 5 -a MAC wlan0mon` |
| 5 | Crack captured handshake | `aircrack-ng cap-01.cap -w wordlist.txt` |
| 6 | Or use GPU cracking | `hashcat -m 22000 hash.hc22000 wordlist.txt` |

---

## Summary Table

| Attack | Target | Impact | Defense |
|--------|--------|--------|---------|
| **Deauth** | Management frames | DoS, handshake capture | 802.11w/PMF |
| **Evil Twin** | Client trust | MITM, credential theft | Certificate pinning, VPN |
| **WPA Crack** | Weak passwords | Network access | Strong passphrase (20+ chars) |
| **PMKID** | WPA2 AP | Offline cracking | WPA3, strong password |
| **KARMA** | Client probes | Auto-connect to rogue AP | Forget saved networks |

---

## Quick Revision Questions

1. **How does a deauthentication attack work?**
2. **What is an evil twin attack?**
3. **How does the PMKID attack differ from traditional WPA cracking?**
4. **What is a KARMA attack?**
5. **What defense prevents deauthentication attacks?**

---

[← Previous: Wireless Security Protocols](02-wireless-security-protocols.md) | [Next: Rogue Access Points →](04-rogue-access-points.md)

---

*Unit 9 - Topic 3 of 5 | [Back to README](../README.md)*
