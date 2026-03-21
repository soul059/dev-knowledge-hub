# Traffic Patterns

## Unit 7 - Topic 4 | Network Traffic Analysis

---

## Overview

**Traffic patterns** describe the normal and abnormal behaviors observed in network traffic. Recognizing these patterns is critical for detecting attacks, identifying compromised hosts, and understanding network behavior. Security analysts must distinguish between **normal baselines** and **malicious indicators**.

---

## 1. Normal Traffic Patterns

| Pattern | Characteristics |
|---------|----------------|
| **Web Browsing** | HTTP/HTTPS to various IPs, variable intervals, mixed sizes |
| **Email** | SMTP/IMAP/POP3 to mail servers, periodic checks |
| **DNS** | Short queries followed by responses, distributed servers |
| **File Transfers** | Large data flows, sustained bandwidth |
| **Video/Streaming** | Sustained high-bandwidth UDP flows |
| **Business Hours** | Traffic peaks during working hours, drops at night |

---

## 2. Malicious Traffic Patterns

### C2 Beaconing

```
┌──────────────────────────────────────────────────────────────┐
│              C2 BEACONING PATTERN                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Malware checks in with C2 server at regular intervals:      │
│                                                              │
│  Time: ─┬──────┬──────┬──────┬──────┬──────┬─               │
│         │      │      │      │      │      │                 │
│       Beacon Beacon Beacon Beacon Beacon Beacon              │
│       60s    60s    60s    60s    60s    60s                  │
│                                                              │
│  INDICATORS:                                                 │
│  • Regular interval connections (e.g., every 60 seconds)     │
│  • Small data size (keep-alive or command poll)              │
│  • Same destination IP/domain                                │
│  • May use HTTP/HTTPS/DNS to blend in                        │
│  • Jitter: Smart malware adds randomness (55-65 sec)        │
│                                                              │
│  DETECTION:                                                  │
│  • Statistical analysis of connection intervals              │
│  • Unusual consistent outbound connections                   │
│  • Connections during off-hours                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Data Exfiltration

```
INDICATORS:
• Large outbound data transfers (unusual volume)
• Transfers to external/unknown IPs
• Transfers during off-hours
• Encrypted traffic to non-standard ports
• DNS queries with encoded data (DNS exfil)
• Slow, steady trickle of data over long period
```

### Lateral Movement

```
INDICATORS:
• Internal host scanning other internal hosts
• SMB/RDP connections between workstations (unusual)
• PsExec, WMI, WinRM connections
• Authentication attempts across multiple hosts
• NTLM relay traffic
```

### DDoS Patterns

```
INDICATORS:
• Sudden massive traffic spike
• Many sources → single destination
• SYN flood: thousands of SYN, no ACK
• Amplification: large UDP responses from DNS/NTP
• Traffic from geographically dispersed sources
```

---

## 3. Pattern Detection Tools

| Tool | Detection Capability |
|------|---------------------|
| **SIEM** | Correlate logs and traffic patterns across sources |
| **Zeek (Bro)** | Network behavior analysis, connection logs |
| **RITA** | Beaconing detection from Zeek logs |
| **Wireshark I/O Graphs** | Visual traffic patterns |
| **NetFlow/sFlow** | Traffic flow statistics without full capture |
| **Darktrace / Vectra** | AI-based anomaly detection |

---

## Summary Table

| Pattern | Indicator | Concern |
|---------|-----------|---------|
| **Beaconing** | Regular-interval outbound connections | C2 communication |
| **Exfiltration** | Large/unusual outbound transfers | Data theft |
| **Lateral Movement** | Internal host-to-host scanning | Attacker spreading |
| **DDoS** | Massive inbound traffic spike | Service disruption |
| **DNS Tunnel** | Long/frequent DNS queries | Covert channel |

---

## Quick Revision Questions

1. **What is C2 beaconing and how is it detected?**
2. **What traffic patterns indicate data exfiltration?**
3. **How does lateral movement appear in network traffic?**
4. **What tool specifically detects beaconing patterns?**
5. **How can an analyst distinguish DDoS from legitimate traffic spikes?**

---

[← Previous: Protocol Analysis](03-protocol-analysis.md) | [Next: Baseline Establishment →](05-baseline-establishment.md)

---

*Unit 7 - Topic 4 of 6 | [Back to README](../README.md)*
