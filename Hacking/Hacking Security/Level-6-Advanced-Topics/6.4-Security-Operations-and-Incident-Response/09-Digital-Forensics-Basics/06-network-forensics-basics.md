# Unit 9: Digital Forensics Basics — Topic 6: Network Forensics Basics

## Overview

**Network forensics** captures, records, and analyzes network traffic to detect intrusions, reconstruct attacks, and gather evidence. Unlike endpoint forensics, network forensics provides visibility into communications between systems, revealing lateral movement, data exfiltration, command-and-control traffic, and other network-based attack indicators.

---

## 1. Network Forensics Fundamentals

```
NETWORK FORENSICS SCOPE:

  ┌────────────────────────────────────────────┐
  │        NETWORK FORENSICS                   │
  │                                            │
  │  WHAT IT REVEALS:                          │
  │  → Who communicated with whom             │
  │  → What data was transferred              │
  │  → When communications occurred           │
  │  → What protocols were used               │
  │  → Command-and-control traffic            │
  │  → Data exfiltration                      │
  │  → Lateral movement                       │
  │  → Malware downloads                      │
  │  → DNS tunneling                          │
  │  → Encrypted traffic anomalies            │
  │                                            │
  │  DATA TYPES:                               │
  │  Full Packet Capture (PCAP)               │
  │  → Complete network conversation          │
  │  → High storage, high detail              │
  │                                            │
  │  Flow Data (NetFlow/sFlow)                │
  │  → Metadata only (src, dst, port, bytes)  │
  │  → Lower storage, less detail             │
  │                                            │
  │  Log Data                                 │
  │  → Firewall, proxy, DNS, IDS logs         │
  │  → Structured, searchable                 │
  └────────────────────────────────────────────┘

NETWORK TAP vs SPAN:
  TAP (Test Access Point):
  → Hardware device, passive
  → Copies all traffic
  → No packet loss
  → Preferred for forensics

  SPAN (Switch Port Analyzer):
  → Software mirror port
  → May drop packets under load
  → Easier to implement
  → Good for monitoring
```

---

## 2. Packet Capture and Analysis

```
PACKET CAPTURE TOOLS:

tcpdump (CLI):
  # Capture all traffic on interface
  tcpdump -i eth0 -w capture.pcap

  # Capture specific host
  tcpdump -i eth0 host 203.0.113.50 -w suspect.pcap

  # Capture specific port
  tcpdump -i eth0 port 443 -w https.pcap

  # Capture with rotation (100MB files)
  tcpdump -i eth0 -w capture_%Y%m%d_%H%M.pcap \
    -C 100 -z gzip

  # Read and filter existing capture
  tcpdump -r capture.pcap host 10.0.1.50

Wireshark (GUI):
  → Real-time capture and analysis
  → Protocol dissection
  → Follow TCP/UDP streams
  → Export objects (files, images)
  → Statistics and conversations
  → Display and capture filters

WIRESHARK DISPLAY FILTERS:
  # Filter by IP
  ip.addr == 203.0.113.50

  # Filter by protocol
  http || dns || smtp

  # Filter HTTP POST requests
  http.request.method == "POST"

  # Filter DNS queries
  dns.qry.name contains "evil"

  # Filter by TCP port
  tcp.port == 4444

  # Filter for SYN packets (port scan)
  tcp.flags.syn == 1 && tcp.flags.ack == 0

  # Filter for data exfiltration indicators
  tcp.len > 1000 && ip.dst != 10.0.0.0/8

TSHARK (CLI Wireshark):
  # Extract HTTP URLs from PCAP
  tshark -r capture.pcap -Y "http.request" \
    -T fields -e http.host -e http.request.uri

  # Extract DNS queries
  tshark -r capture.pcap -Y "dns.qry.name" \
    -T fields -e dns.qry.name | sort | uniq -c

  # Extract file transfers
  tshark -r capture.pcap --export-objects http,/output/
```

---

## 3. Network Traffic Analysis

```
TRAFFIC ANALYSIS TECHNIQUES:

C2 DETECTION:
  Beaconing Pattern:
  → Regular interval connections to same host
  → Small, consistent packet sizes
  → Often over HTTP/HTTPS
  → May use DNS tunneling

  # Detect beaconing with tshark
  tshark -r capture.pcap -qz io,stat,60,"COUNT(frame)"
  
  # Look for regular intervals
  # Connections every 60s, 300s, etc.

  Indicators:
  → Regular time intervals (± jitter)
  → Consistent packet sizes
  → Long-lived sessions
  → Connections to uncommon domains
  → Unusual user-agent strings
  → Self-signed or expired certificates

DNS ANALYSIS:
  → Query for known bad domains
  → Unusually long subdomain names (tunneling)
  → High volume of queries to one domain
  → TXT record queries (data exfiltration)
  → Queries to non-standard DNS servers

  # DNS tunneling indicators
  tshark -r capture.pcap -Y "dns" -T fields \
    -e dns.qry.name | awk '{print length, $0}' | \
    sort -rn | head -20
  
  # Long domain names = possible tunneling
  # Normal: www.example.com (15 chars)
  # Tunneling: aGVsbG8gd29ybGQ.evil.com (25+ chars)

DATA EXFILTRATION:
  → Large outbound transfers
  → Transfers to cloud storage
  → Transfers outside business hours
  → Encrypted uploads to unknown hosts
  → DNS tunneling
  → ICMP tunneling

  # Identify large transfers
  tshark -r capture.pcap -qz conv,tcp | \
    sort -k 10 -rn | head -20

HTTP/HTTPS ANALYSIS:
  → Unusual user-agent strings
  → POST requests with large bodies
  → Connections to IP addresses (not domains)
  → Uncommon HTTP methods
  → Suspicious file downloads
```

---

## 4. Network Forensic Platforms

```
FULL PACKET CAPTURE PLATFORMS:

ARKIME (formerly Moloch):
  → Open-source full PCAP capture
  → Indexed, searchable
  → Web-based interface
  → Session reconstruction
  → Can handle high traffic volumes
  → Integrates with Elasticsearch

SECURITY ONION:
  → Full network security monitoring platform
  → Includes: Suricata, Zeek, Arkime
  → Log management with Elasticsearch
  → Kibana dashboards
  → Free and open source

ZEEK (formerly Bro):
  → Network analysis framework
  → Generates structured logs
  → Protocol analysis
  → File extraction
  → Custom scripting

  Key Zeek Logs:
  Log File    | Content
  conn.log    | All connections
  http.log    | HTTP requests
  dns.log     | DNS queries
  files.log   | File transfers
  ssl.log     | SSL/TLS connections
  notice.log  | Anomalies detected
  weird.log   | Unusual traffic

  # Zeek analysis example
  cat conn.log | zeek-cut id.orig_h id.resp_h \
    id.resp_p proto | sort | uniq -c | sort -rn

NETFLOW ANALYSIS:
  → Metadata (no payload)
  → Source/dest IP, ports, bytes
  → Useful for identifying patterns
  → Lower storage requirements

  Tools:
  → nfdump: CLI analysis
  → SiLK: Network analysis suite
  → ntopng: Web-based visualization
  → Elastic Stack: Visualization
```

---

## 5. Network Evidence Collection

```
EVIDENCE COLLECTION:

WHAT TO COLLECT:
  → Full packet captures (if available)
  → NetFlow/sFlow data
  → Firewall logs
  → Proxy/web filter logs
  → DNS logs
  → IDS/IPS alerts
  → DHCP logs (IP-to-MAC mapping)
  → VPN logs
  → Email gateway logs
  → Load balancer logs
  → WAF logs

COLLECTION CHECKLIST:
  [ ] Identify relevant time window
  [ ] Identify relevant network segments
  [ ] Collect PCAP from capture platform
  [ ] Export firewall logs for time window
  [ ] Export proxy logs for relevant IPs
  [ ] Export DNS query logs
  [ ] Export IDS alerts
  [ ] Hash all collected evidence
  [ ] Document collection method
  [ ] Maintain chain of custody

STORAGE CONSIDERATIONS:
  Full PCAP Storage Requirements:
  → 1 Gbps link ≈ 10 TB/day
  → 10 Gbps link ≈ 100 TB/day
  → Compression helps but limited
  → Retention: typically 3-30 days
  → Targeted capture more practical

ANALYSIS REPORT TEMPLATE:
  ┌──────────────────────────────────────────┐
  │ NETWORK FORENSIC FINDINGS               │
  │                                          │
  │ Analysis Period: Jan 10-15, 2024        │
  │ Systems: 10.0.1.50, 203.0.113.50       │
  │                                          │
  │ FINDINGS:                               │
  │ 1. C2 beaconing detected               │
  │    → 10.0.1.50 → 203.0.113.50:443      │
  │    → Every 60 seconds ± 5s jitter       │
  │    → Total: 7,200 connections (5 days)  │
  │                                          │
  │ 2. Data exfiltration detected           │
  │    → 10.0.1.50 → mega.nz               │
  │    → Upload: 1.8 GB via HTTPS           │
  │    → Jan 12, 03:00-04:30 UTC            │
  │                                          │
  │ 3. DNS tunneling suspected              │
  │    → Queries to data.evil.example.com   │
  │    → 500+ unique subdomains             │
  │    → Average subdomain length: 40 chars  │
  └──────────────────────────────────────────┘
```

---

## Summary Table

| Data Type | Detail Level | Storage | Best For |
|-----------|-------------|---------|---------|
| Full PCAP | Complete | Very High | Detailed reconstruction |
| NetFlow | Metadata | Low | Pattern analysis |
| Firewall Logs | Connection data | Medium | Access tracking |
| Proxy Logs | URL/content | Medium | Web activity |
| DNS Logs | Query/response | Low | Domain analysis |
| IDS Alerts | Threat matches | Low | Known threats |

---

## Revision Questions

1. What is the difference between full PCAP and NetFlow data?
2. How can network beaconing be detected in packet captures?
3. What indicators suggest DNS tunneling?
4. What tools are used for full-scale network forensics?
5. What network evidence should be collected during an incident?

---

*Previous: [05-memory-forensics-basics.md](05-memory-forensics-basics.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
