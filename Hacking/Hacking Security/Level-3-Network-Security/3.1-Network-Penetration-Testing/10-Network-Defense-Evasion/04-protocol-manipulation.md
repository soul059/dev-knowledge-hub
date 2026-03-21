# Protocol Manipulation

## Unit 10 - Topic 4 | Network Defense Evasion

---

## Overview

**Protocol manipulation** exploits the way network protocols handle edge cases, non-standard packets, and implementation differences between security tools and target systems to evade detection and bypass filtering.

---

## 1. IP Protocol Manipulation

```
IP HEADER MANIPULATION:

TTL MANIPULATION:
├── Set TTL to expire after IDS but before target
├── IDS sees the packet, target doesn't receive it
├── Follow up with real packet (IDS ignored it)
└── Confuses IDS traffic reassembly

IP FRAGMENTATION:
├── Split packet across multiple fragments
├── IDS must reassemble to detect signature
├── Different reassembly algorithms between IDS/target
├── Overlapping fragments → different interpretations
└── Tiny fragments → signatures don't match

IP OPTIONS:
├── Loose source routing
├── Record route
├── Timestamp
├── Some firewalls drop packets with options
└── Others pass them through

IP SPOOFING:
├── Forge source IP address
├── Useful for DoS, scanning evasion
├── Response goes to spoofed IP (blind attack)
└── Prevented by egress filtering (BCP38)
```

```bash
# === SCAPY (Python Packet Crafting) ===
from scapy.all import *

# Custom TTL:
pkt = IP(dst="target", ttl=1)/TCP(dport=80)
send(pkt)

# Fragmented packet:
pkt = IP(dst="target")/TCP(dport=80)/"GET / HTTP/1.1\r\nHost: target\r\n\r\n"
frags = fragment(pkt, fragsize=8)
for f in frags:
    send(f)

# Overlapping fragments:
# Fragment 1: bytes 0-15 (innocent)
# Fragment 2: bytes 8-23 (malicious, overlaps!)
# Target reassembles using Fragment 2's data
# IDS reassembles using Fragment 1's data
```

---

## 2. TCP Protocol Manipulation

```bash
# === TCP FLAGS MANIPULATION ===
# Non-standard flag combinations:
nmap -sF target.com    # FIN scan (no SYN)
nmap -sN target.com    # NULL scan (no flags)
nmap -sX target.com    # XMAS scan (FIN+PSH+URG)

# Custom flags with Scapy:
pkt = IP(dst="target")/TCP(dport=80, flags="SFPAU")
send(pkt)

# === TCP SEGMENTATION ===
# Split HTTP request across TCP segments:
# Segment 1: "GET /"
# Segment 2: "HTTP/1.1\r\n"
# Segment 3: "Host: target\r\n\r\n"
# IDS may fail to reassemble

# === TCP URGENT POINTER ===
# Use URG flag to hide data:
# ├── URG data processed differently
# ├── Some IDS ignore urgent data
# └── Target processes it normally

# === RST COOKIES ===
# Some firewalls use SYN cookies
# Craft specific RST patterns to bypass
# TCP handshake manipulation

# === OUT-OF-ORDER DELIVERY ===
# Send TCP segments out of order:
# Segment 3 → Segment 1 → Segment 2
# Target reassembles correctly
# Some IDS fail to handle reordering
```

---

## 3. HTTP Protocol Manipulation

```bash
# === HTTP EVASION TECHNIQUES ===

# CHUNKED ENCODING:
# Split payload across chunks:
# POST /upload HTTP/1.1
# Transfer-Encoding: chunked
# 
# 5\r\n
# <%ph\r\n
# 3\r\n
# p s\r\n
# 7\r\n
# ystem(\r\n
# ...

# REQUEST SMUGGLING:
# Exploit differences between proxy and backend:
# Content-Length: 13
# Transfer-Encoding: chunked
# 
# 0\r\n
# \r\n
# GET /admin HTTP/1.1

# CASE MANIPULATION:
# GeT /pAgE HTTP/1.1
# Some web servers are case-insensitive
# But WAF rules may be case-sensitive

# DOUBLE URL ENCODING:
# / → %2F → %252F
# ' → %27 → %2527
# WAF decodes once, app decodes twice

# UNICODE ENCODING:
# ../ → ..%c0%af
# ../ → ..%252f
# Bypass path traversal filters

# NULL BYTES:
# file.php%00.jpg
# WAF sees .jpg (image)
# App sees .php (executes)

# PARAMETER POLLUTION:
# ?id=1&id=2
# WAF checks first parameter
# App uses second parameter
```

---

## 4. DNS Protocol Manipulation

```bash
# === DNS EVASION ===
# Long subdomain names (tunneling detection):
# Split data across multiple labels:
# aGVsbG8.d29ybGQ.tunnel.attacker.com

# DNS record type abuse:
# TXT records — large data capacity
# CNAME records — redirect chains
# MX records — mail exchange abuse
# NULL records — arbitrary data

# DNS over HTTPS (DoH):
# Encrypts DNS queries inside HTTPS
# Network monitoring can't see DNS content
curl -H "accept: application/dns-json" \
  "https://cloudflare-dns.com/dns-query?name=target.com&type=A"

# === FAST FLUX DNS ===
# Rapidly change DNS records
# A record changes every 60 seconds
# Points to different compromised hosts
# Makes takedown very difficult
```

---

## 5. Protocol Manipulation Tools

| Tool | Purpose |
|------|---------|
| **Scapy** | Custom packet crafting (Python) |
| **hping3** | TCP/IP packet assembler |
| **Nmap** | Built-in evasion flags |
| **fragroute** | Fragment/manipulate traffic |
| **nping** | Custom packet generation |
| **tcpreplay** | Replay modified captures |

```bash
# === HPING3 EXAMPLES ===
# Custom SYN with spoofed source:
hping3 -S -p 80 -a spoofed_ip target.com

# Flood with random source:
hping3 --flood --rand-source -p 80 target.com

# Custom flags:
hping3 -F -P -U -p 80 target.com  # FIN+PSH+URG (XMAS)

# Fragment:
hping3 -f -p 80 target.com
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **IP fragmentation** | Split packets to break signatures |
| **TTL manipulation** | Packets expire after IDS |
| **TCP segmentation** | Split payload across TCP segments |
| **HTTP smuggling** | Exploit proxy/backend differences |
| **DNS abuse** | Tunnel data, use DoH for privacy |
| **Scapy/hping3** | Custom packet crafting tools |

---

## Quick Revision Questions

1. **How does IP fragmentation evade IDS?**
2. **What is HTTP request smuggling?**
3. **How does TTL manipulation work against IDS?**
4. **What is double URL encoding and why does it bypass WAFs?**
5. **How do overlapping IP fragments confuse security tools?**

---

[← Previous: Traffic Obfuscation](03-traffic-obfuscation.md) | [Next: Timing-Based Evasion →](05-timing-based-evasion.md)

---

*Unit 10 - Topic 4 of 5 | [Back to README](../README.md)*
