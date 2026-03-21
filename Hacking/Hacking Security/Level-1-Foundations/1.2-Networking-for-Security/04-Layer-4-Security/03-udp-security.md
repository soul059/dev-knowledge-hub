# UDP Security Considerations

## Unit 4 - Topic 3 | Layer 4 Security

---

## Overview

**UDP (User Datagram Protocol)** is a connectionless, lightweight transport protocol. Its lack of handshake and state tracking makes it fast but also exploitable. UDP is heavily abused for **amplification DDoS attacks**, **UDP flooding**, and **data exfiltration** through services like DNS, NTP, and SNMP.

---

## 1. TCP vs UDP — Security Perspective

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Handshake required | No handshake |
| **State** | Stateful (tracked) | Stateless |
| **Reliability** | Guaranteed delivery | Best effort |
| **Spoofing** | Harder (need valid seq numbers) | Easy (no verification) |
| **Amplification** | Not possible | Major DDoS vector |
| **Scanning** | Clear open/closed responses | Ambiguous (no response = open OR filtered) |

---

## 2. UDP-Based Attacks

### UDP Flood

```
┌──────────────────────────────────────────────────────────────┐
│              UDP FLOOD (DoS)                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Attacker sends massive UDP packets to random ports          │
│  Target checks for listening service → none found            │
│  Target sends ICMP "Port Unreachable" for each              │
│  → Resource exhaustion from processing + ICMP responses      │
│                                                              │
│  ┌──────────┐   UDP to random ports   ┌──────────┐          │
│  │ Attacker │═════════════════════════►│  Target  │          │
│  └──────────┘                          │(exhausted)│         │
│                                        └──────────┘          │
│                                                              │
│  DEFENSE: Rate limiting, firewall UDP filtering,             │
│           disable unnecessary UDP services                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Amplification Attacks

| Protocol | Port | Amplification Factor | Abuse |
|----------|:----:|:-------------------:|-------|
| **DNS** | 53 | 28x–54x | Open resolvers |
| **NTP** | 123 | 556x | monlist command |
| **SNMP** | 161 | 6x | GetBulk queries |
| **SSDP** | 1900 | 30x | UPnP discovery |
| **Memcached** | 11211 | 51,000x! | Get requests |
| **CHARGEN** | 19 | Infinite | Character generator |
| **LDAP** | 389 | 46x–55x | Rootless queries |

```
AMPLIFICATION ATTACK PATTERN:
1. Attacker spoofs source IP to VICTIM's IP
2. Sends small query to UDP service (e.g., NTP)
3. Service sends LARGE reply to VICTIM
4. Multiply by thousands of servers = massive DDoS

Example (NTP monlist):
Request: 234 bytes → Response: 100x packets of 482 bytes
Amplification: ~556x
```

---

## 3. UDP Scanning Challenges

```bash
# UDP scanning is SLOW and UNRELIABLE
nmap -sU target.com                    # Full UDP scan (very slow)
nmap -sU --top-ports 20 target.com     # Top 20 UDP ports
nmap -sU -sV target.com               # With version detection

# UDP scan responses:
# Response received     → Port OPEN (service responded)
# ICMP unreachable      → Port CLOSED
# No response           → Port OPEN or FILTERED (ambiguous!)
# ICMP other error      → FILTERED

# Faster UDP scanning
unicornscan -mU -r 500 target.com:1-65535
```

---

## 4. Securing UDP Services

| Service | Port | Security Measure |
|---------|:----:|-----------------|
| **DNS** | 53 | Disable open resolver, rate limit, DNSSEC |
| **NTP** | 123 | Disable monlist (`restrict noquery`), rate limit |
| **SNMP** | 161 | Use SNMPv3, change community strings, restrict source IPs |
| **DHCP** | 67/68 | DHCP snooping on switch |
| **TFTP** | 69 | Disable if not needed, restrict to management VLAN |
| **SSDP** | 1900 | Disable UPnP on edge devices |

```bash
# NTP — Disable monlist (prevents amplification)
# /etc/ntp.conf
restrict default noquery nomodify notrap nopeer
restrict 127.0.0.1

# SNMP — Restrict access
# /etc/snmp/snmpd.conf
rocommunity MySecretString 10.0.0.0/24
# Better: Use SNMPv3 with authentication and encryption
```

---

## 5. UDP Defense Best Practices

| Practice | Description |
|----------|-------------|
| **Disable unused UDP services** | Reduce attack surface |
| **Rate-limit UDP** | Limit incoming UDP packets per source |
| **Ingress filtering (BCP38)** | Block spoofed source IPs |
| **Response rate limiting** | Limit outbound ICMP unreachable |
| **DDoS protection** | Cloud-based mitigation for amplification |
| **Stateful firewall** | Track UDP "connections" by source/dest/port |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **UDP** | Connectionless — no handshake, easy to spoof |
| **Amplification** | Small request → large response to spoofed victim |
| **Worst Amplifier** | Memcached (51,000x amplification!) |
| **UDP Scanning** | Slow and ambiguous — no response could mean open or filtered |
| **Defense** | Disable unused services, rate limit, BCP38, DDoS protection |

---

## Quick Revision Questions

1. **Why is UDP more susceptible to spoofing than TCP?**
2. **How do UDP amplification attacks work?**
3. **Which UDP service has the highest amplification factor?**
4. **Why is UDP scanning slow and unreliable?**
5. **How does NTP monlist enable amplification attacks?**
6. **What is BCP38 and how does it prevent UDP amplification?**

---

[← Previous: TCP Attacks](02-tcp-attacks.md) | [Next: Port Scanning Techniques →](04-port-scanning-techniques.md)

---

*Unit 4 - Topic 3 of 5 | [Back to README](../README.md)*
