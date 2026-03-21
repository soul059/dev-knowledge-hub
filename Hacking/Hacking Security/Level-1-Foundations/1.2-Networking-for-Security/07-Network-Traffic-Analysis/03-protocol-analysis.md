# Protocol Analysis

## Unit 7 - Topic 3 | Network Traffic Analysis

---

## Overview

**Protocol analysis** is the systematic examination of network protocols in captured traffic to identify normal behavior, detect anomalies, and discover attacks. It goes beyond simple filtering — it requires understanding protocol structure, expected behavior, and indicators of compromise.

---

## 1. Analyzing Key Protocols

### TCP Analysis

```
NORMAL TCP HANDSHAKE:
Packet 1: SYN (Client → Server)
Packet 2: SYN-ACK (Server → Client)
Packet 3: ACK (Client → Server)

SUSPICIOUS PATTERNS:
• Many SYN without SYN-ACK → Port scan or SYN flood
• RST immediately after SYN-ACK → Port closed or firewall
• Many RST packets → Connection failures or scan
• Retransmissions → Network issues or DoS
• Out-of-order packets → Network problems or injection

Wireshark filters:
tcp.analysis.retransmission     # Retransmissions
tcp.analysis.duplicate_ack      # Duplicate ACKs
tcp.analysis.zero_window        # Zero window (resource issues)
```

### DNS Analysis

```
NORMAL DNS:
• Query: A record for legitimate domain
• Response: IP address, reasonable TTL

SUSPICIOUS:
• Very long subdomain names → DNS tunneling
  aGVsbG8gd29ybGQ=.tunnel.evil.com
• High query frequency to single domain → C2 or exfiltration
• TXT record queries with encoded data → Data exfiltration
• Queries to unusual TLDs → Potentially malicious
• NXDOMAIN responses (many) → Domain generation algorithm (DGA)

Wireshark filters:
dns.qry.name contains "evil"
dns.flags.rcode == 3            # NXDOMAIN
dns.qry.type == 16              # TXT record queries
```

### HTTP Analysis

```
NORMAL HTTP:
• GET requests for web resources
• POST for form submissions
• 200 OK responses

SUSPICIOUS:
• User-Agent anomalies → Bot or malware
• Unusual HTTP methods (PUT, DELETE) → Web exploitation
• Encoded/obfuscated parameters → SQLi, XSS attempts
• Beaconing pattern (regular intervals) → C2 communication
• Large POST to external IP → Data exfiltration

Wireshark filters:
http.request.method == "POST" && http.content_length > 10000
http.user_agent contains "curl"
http.response.code >= 400       # Error responses
```

---

## 2. Identifying Attack Traffic

| Attack | Protocol Indicators |
|--------|-------------------|
| **Port Scan** | Many SYN to different ports from same source |
| **SYN Flood** | Thousands of SYN without completing handshake |
| **ARP Spoofing** | Multiple IPs resolving to same MAC address |
| **DNS Tunneling** | Long subdomain names, high query rate, TXT records |
| **Brute Force** | Repeated failed authentication attempts |
| **Data Exfiltration** | Large outbound transfers, DNS TXT queries |
| **C2 Beaconing** | Regular-interval connections to external host |
| **MITM** | ARP anomalies, certificate warnings |

---

## 3. Protocol Analysis Tools

| Tool | Purpose |
|------|---------|
| **Wireshark** | Interactive protocol analysis |
| **tshark** | CLI protocol analysis (scriptable) |
| **NetworkMiner** | Extract files, images, credentials from pcap |
| **Zeek (Bro)** | Network security monitoring framework |
| **Suricata** | IDS/IPS with protocol analysis |
| **ngrep** | Network grep — search packet content |

```bash
# tshark — CLI protocol analysis
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri

# ngrep — Search for strings in network traffic
ngrep -q -d eth0 "password"
ngrep -q -d eth0 "GET|POST" tcp port 80
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Protocol Analysis** | Examine protocol behavior for anomalies |
| **TCP Anomalies** | Retransmissions, RST floods, half-open connections |
| **DNS Anomalies** | Long names, high frequency, TXT records = tunneling |
| **HTTP Anomalies** | Beaconing, large POSTs, unusual user agents |
| **Tools** | Wireshark, tshark, Zeek, NetworkMiner |

---

## Quick Revision Questions

1. **What TCP patterns indicate a port scan?**
2. **How can you identify DNS tunneling in a capture?**
3. **What HTTP traffic patterns suggest C2 communication?**
4. **What does a high rate of RST packets indicate?**
5. **Name 3 tools for protocol analysis.**

---

[← Previous: Wireshark Fundamentals](02-wireshark-fundamentals.md) | [Next: Traffic Patterns →](04-traffic-patterns.md)

---

*Unit 7 - Topic 3 of 6 | [Back to README](../README.md)*
