# Bluetooth Security

## Unit 7 - Topic 5 | Wireless Network Testing

---

## Overview

**Bluetooth** is a short-range wireless protocol used in headphones, keyboards, IoT devices, medical equipment, and automotive systems. Security vulnerabilities in Bluetooth can lead to eavesdropping, unauthorized access, data theft, and device control.

---

## 1. Bluetooth Versions and Security

| Version | Feature | Security Level |
|---------|---------|:--------------:|
| **Classic (BR/EDR)** | Audio, file transfer | ⚠️ Varies |
| **BLE (4.0+)** | Low energy IoT | ⚠️ Often weak |
| **BT 5.0** | Longer range, faster | ✅ Improved |
| **BT 5.2** | LE Audio, Isochronous | ✅ Better |

```
BLUETOOTH SECURITY MODES:
├── Mode 1: No security (no auth, no encryption)
├── Mode 2: Service-level security
├── Mode 3: Link-level security
└── Mode 4: Secure Simple Pairing (SSP) — Current

PAIRING METHODS:
├── Just Works — No authentication (vulnerable!)
├── Numeric Comparison — Confirm 6-digit code
├── Passkey Entry — Enter PIN
└── Out of Band — NFC or other channel
```

---

## 2. Bluetooth Reconnaissance

```bash
# === HCITOOL (Linux Built-in) ===
# Scan for discoverable devices:
hcitool scan
# Shows: MAC address and device name

# Get device info:
hcitool info AA:BB:CC:DD:EE:FF

# BLE scanning:
hcitool lescan

# === BLUETOOTHCTL (Modern Linux) ===
bluetoothctl
> scan on       # Start scanning
> devices       # List found devices
> info AA:BB:CC:DD:EE:FF  # Device details

# === BETTERCAP (BLE Enumeration) ===
sudo bettercap
ble.recon on
# Discovers BLE devices with services

# === BTLEJACK (BLE Sniffing) ===
btlejack -c AA:BB:CC:DD:EE:FF
# Sniff BLE connections (requires Micro:Bit)

# === UBERTOOTH (Advanced Bluetooth Sniffing) ===
ubertooth-scan         # Discover devices
ubertooth-btle -f -t AA:BB:CC:DD:EE:FF  # Follow BLE device
ubertooth-btbb -l      # Sniff classic Bluetooth (LAP)
# Requires Ubertooth One hardware (~$120)
```

---

## 3. Common Bluetooth Attacks

```
BLUETOOTH ATTACK TYPES:

BLUEJACKING:
├── Send unsolicited messages via Bluetooth
├── Low risk — annoyance, social engineering
└── Uses OBEX push protocol

BLUESNARFING:
├── Unauthorized access to device data
├── Contacts, calendar, messages, files
├── Exploits OBEX vulnerabilities
└── Can be done without pairing!

BLUEBUGGING:
├── Full control of target device
├── Make calls, send messages
├── Access internet connection
└── Most serious — requires exploit

BLUEBORNE (CVE-2017-0781):
├── Over-the-air attack — no pairing needed!
├── Bluetooth doesn't even need to be discoverable
├── Affects Android, iOS, Windows, Linux
├── RCE without user interaction
└── Billions of devices affected

KNOB ATTACK (Key Negotiation of Bluetooth):
├── Force low-entropy encryption key
├── Downgrade key to 1 byte (8 bits)
├── Brute force 256 possibilities
└── Decrypt all Bluetooth traffic

BIAS ATTACK:
├── Bluetooth Impersonation Attack
├── Impersonate previously paired device
├── Bypass authentication
└── No knowledge of pairing key needed
```

---

## 4. BLE (Bluetooth Low Energy) Attacks

```bash
# === BLE GATT ENUMERATION ===
# GATT = Generic Attribute Profile
# BLE devices expose services and characteristics

# gatttool:
gatttool -b AA:BB:CC:DD:EE:FF -I
> connect
> primary         # List primary services
> characteristics # List characteristics
> char-read-hnd 0x000e  # Read a characteristic

# === BTLE (BLE Tool) ===
# Read all characteristics:
bettercap
ble.recon on
ble.enum AA:BB:CC:DD:EE:FF

# === COMMON BLE VULNERABILITIES ===
# ├── No encryption (data in plaintext)
# ├── Static passkey (000000, 123456)
# ├── No authentication on characteristics
# ├── Writable characteristics (change device behavior)
# ├── Hardcoded credentials in firmware
# └── Replay attacks on commands

# === SMART LOCK ATTACK EXAMPLE ===
# 1. Sniff BLE communication when owner unlocks
# 2. Capture unlock command
# 3. Replay command → door unlocks!
```

---

## 5. Bluetooth Defense

| Defense | Description |
|---------|-------------|
| **Disable when not in use** | Reduces attack surface |
| **Non-discoverable mode** | Don't broadcast presence |
| **Reject unknown pairing** | Don't accept random requests |
| **Update firmware** | Patch BlueBorne, KNOB, BIAS |
| **Use BT 5.0+** | Better security features |
| **Secure Simple Pairing** | Use Numeric Comparison (not Just Works) |
| **BLE authentication** | Implement proper GATT security |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Bluejacking** | Unsolicited messages (low risk) |
| **Bluesnarfing** | Unauthorized data access |
| **BlueBorne** | Over-the-air RCE — no pairing needed |
| **KNOB** | Downgrade encryption key strength |
| **BLE** | Low energy IoT — often no security |
| **Defense** | Disable Bluetooth when not in use |

---

## Quick Revision Questions

1. **What is the difference between Bluejacking and Bluesnarfing?**
2. **Why is BlueBorne considered a critical vulnerability?**
3. **How does the KNOB attack work?**
4. **What BLE security issues are common in IoT devices?**
5. **What hardware is needed for advanced Bluetooth testing?**

---

[← Previous: Wireless Client Attacks](04-wireless-client-attacks.md) | [Next: Wireless Tools →](06-wireless-tools.md)

---

*Unit 7 - Topic 5 of 6 | [Back to README](../README.md)*
