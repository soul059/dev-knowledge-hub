# Wireless Security Protocols

## Unit 9 - Topic 2 | Wireless Networks

---

## Overview

Wireless security protocols protect Wi-Fi communications through **encryption** and **authentication**. The evolution from WEP → WPA → WPA2 → WPA3 represents the ongoing battle between wireless security and attack techniques. Understanding each protocol's strengths and weaknesses is essential for both defending and testing wireless networks.

---

## 1. Protocol Evolution

```
┌──────────────────────────────────────────────────────────────┐
│              WIRELESS SECURITY EVOLUTION                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1999: WEP ─────── Broken! (RC4 + weak IV)                  │
│           │                                                  │
│  2003: WPA ─────── Temporary fix (TKIP over WEP hardware)   │
│           │                                                  │
│  2004: WPA2 ────── Strong (AES-CCMP), KRACK vuln in 2017    │
│           │                                                  │
│  2018: WPA3 ────── Current standard (SAE, 192-bit)           │
│                                                              │
│  ⚠️ WEP and WPA-TKIP should NEVER be used                   │
│  ✅ WPA2-AES minimum, WPA3 recommended                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Protocol Comparison

| Feature | WEP | WPA | WPA2 | WPA3 |
|---------|:---:|:---:|:----:|:----:|
| **Year** | 1999 | 2003 | 2004 | 2018 |
| **Encryption** | RC4 | TKIP (RC4) | AES-CCMP | AES-GCMP |
| **Key Length** | 40/104-bit | 128-bit | 128-bit | 128/192-bit |
| **Authentication** | Shared Key | PSK or 802.1X | PSK or 802.1X | SAE or 802.1X |
| **IV Length** | 24-bit | 48-bit | 48-bit | N/A |
| **Status** | ❌ Broken | ⚠️ Deprecated | ✅ Acceptable | ✅ Recommended |
| **Crack Time** | Minutes | Hours | Days-Years | Resistant |

---

## 3. WEP (Wired Equivalent Privacy) — BROKEN

```
WHY WEP IS BROKEN:
├── 24-bit IV → Only 16.7 million possibilities
├── IV reuse inevitable on busy networks
├── RC4 key scheduling weakness
├── Same key + IV = same keystream
└── Tools can crack in < 5 minutes with enough packets

# Cracking WEP (for authorized testing only):
airmon-ng start wlan0                    # Monitor mode
airodump-ng wlan0mon                      # Find target
airodump-ng -c 6 --bssid AA:BB:CC -w cap wlan0mon  # Capture
aireplay-ng -3 -b AA:BB:CC wlan0mon      # ARP replay (generate IVs)
aircrack-ng cap-01.cap                    # Crack key
```

---

## 4. WPA/WPA2 — PSK Mode

```
WPA2-PSK (Pre-Shared Key):
├── 4-Way Handshake for key exchange
├── PMK derived from passphrase + SSID
├── PTK (per-session key) derived from PMK + nonces
└── Vulnerable to offline dictionary/brute force attacks

4-WAY HANDSHAKE:
Client                          AP
  │──── ANonce ─────────────────►│   Message 1
  │◄─── SNonce + MIC ───────────│   Message 2
  │──── GTK + MIC ──────────────►│   Message 3
  │◄─── ACK ────────────────────│   Message 4

ATTACK:
1. Capture 4-way handshake
2. Run offline dictionary attack against handshake
3. If password is weak → cracked

# Capture and crack WPA2:
airodump-ng -c 6 --bssid AA:BB:CC -w handshake wlan0mon
aireplay-ng -0 5 -a AA:BB:CC wlan0mon    # Deauth to force handshake
aircrack-ng handshake-01.cap -w wordlist.txt
hashcat -m 22000 hash.hc22000 wordlist.txt  # GPU cracking
```

---

## 5. WPA3 — Current Standard

| Feature | Description |
|---------|-------------|
| **SAE (Simultaneous Authentication of Equals)** | Replaces PSK — resistant to offline dictionary attacks |
| **Forward Secrecy** | Past sessions can't be decrypted if key is compromised |
| **192-bit Security** | Enterprise mode with stronger cryptography |
| **Protected Management Frames** | Mandatory — prevents deauth attacks |
| **Opportunistic Wireless Encryption** | Encrypts open networks (no password needed) |

```
WPA3-SAE vs WPA2-PSK:
WPA2-PSK: Capture handshake → offline brute force → crack
WPA3-SAE: Each authentication attempt requires AP interaction
          → no offline attacks possible → brute force is online only
```

---

## 6. Enterprise Authentication (802.1X/EAP)

```
┌──────────────────────────────────────────────────────────────┐
│              802.1X ENTERPRISE AUTHENTICATION                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client ──► Access Point ──► RADIUS Server                   │
│  (Supplicant)  (Authenticator)  (Authentication Server)      │
│                                                              │
│  EAP Methods:                                                │
│  ├── EAP-TLS: Certificate on both sides (most secure)       │
│  ├── PEAP: Server cert + username/password                   │
│  ├── EAP-TTLS: Similar to PEAP                              │
│  └── EAP-FAST: Cisco proprietary                            │
│                                                              │
│  ✅ No shared password (unique credentials per user)         │
│  ✅ Centralized authentication and accounting                │
│  ✅ Can revoke individual user access                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Protocol | Encryption | Status | Key Vulnerability |
|----------|-----------|--------|-------------------|
| **WEP** | RC4 | ❌ Broken | IV reuse, cracked in minutes |
| **WPA** | TKIP | ⚠️ Deprecated | TKIP weaknesses |
| **WPA2-PSK** | AES-CCMP | ✅ Acceptable | Offline dictionary attacks on weak passwords |
| **WPA2-Enterprise** | AES-CCMP | ✅ Strong | Evil twin + RADIUS impersonation |
| **WPA3** | AES-GCMP/SAE | ✅ Recommended | Dragonblood (mostly patched) |

---

## Quick Revision Questions

1. **Why is WEP considered broken?**
2. **What is the 4-way handshake and why is it important for attackers?**
3. **How does WPA3-SAE prevent offline attacks?**
4. **What is the difference between PSK and Enterprise modes?**
5. **What EAP method is considered most secure and why?**

---

[← Previous: 802.11 Standards](01-802-11-standards.md) | [Next: Wireless Attacks →](03-wireless-attacks.md)

---

*Unit 9 - Topic 2 of 5 | [Back to README](../README.md)*
