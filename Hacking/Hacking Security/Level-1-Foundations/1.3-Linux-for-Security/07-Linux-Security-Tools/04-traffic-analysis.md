# Traffic Analysis

## Unit 7 - Topic 4 | Linux Security Tools

---

## Overview

**Traffic analysis** involves capturing and inspecting network packets to detect anomalies, attacks, malware communication, and data exfiltration. Linux provides powerful tools for packet capture and analysis — from command-line tools like **tcpdump** to GUI tools like **Wireshark**.

---

## 1. tcpdump — Command-Line Packet Capture

```bash
# === BASIC CAPTURE ===
sudo tcpdump -i eth0                 # Capture on interface
sudo tcpdump -i any                  # Capture on all interfaces
sudo tcpdump -c 100                  # Capture 100 packets
sudo tcpdump -w capture.pcap         # Save to file
sudo tcpdump -r capture.pcap         # Read from file

# === FILTERS (BPF Syntax) ===
sudo tcpdump host 10.0.0.1           # Traffic to/from host
sudo tcpdump src 10.0.0.1            # Traffic from host
sudo tcpdump dst 10.0.0.1            # Traffic to host
sudo tcpdump net 10.0.0.0/24         # Entire subnet
sudo tcpdump port 80                 # HTTP traffic
sudo tcpdump port 443                # HTTPS traffic
sudo tcpdump tcp                     # TCP only
sudo tcpdump udp                     # UDP only
sudo tcpdump icmp                    # ICMP only

# === COMPLEX FILTERS ===
# HTTP traffic from specific host
sudo tcpdump -i eth0 'src 10.0.0.5 and port 80'

# DNS queries
sudo tcpdump -i eth0 'port 53'

# SYN packets (port scan detection)
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0'

# Traffic NOT from a specific host
sudo tcpdump -i eth0 'not host 10.0.0.1'

# === DISPLAY OPTIONS ===
sudo tcpdump -A                      # Print ASCII payload
sudo tcpdump -X                      # Print hex + ASCII
sudo tcpdump -v                      # Verbose
sudo tcpdump -vv                     # More verbose
sudo tcpdump -nn                     # Don't resolve names/ports
sudo tcpdump -tttt                   # Full timestamp
```

---

## 2. Wireshark / tshark

```bash
# tshark — Wireshark's CLI version
# Capture
sudo tshark -i eth0 -c 100
sudo tshark -i eth0 -w capture.pcap

# Read and filter
tshark -r capture.pcap
tshark -r capture.pcap -Y 'http'                     # HTTP only
tshark -r capture.pcap -Y 'dns'                      # DNS only
tshark -r capture.pcap -Y 'tcp.port == 443'          # HTTPS
tshark -r capture.pcap -Y 'ip.addr == 10.0.0.1'     # Specific IP

# === USEFUL DISPLAY FILTERS ===
# tcp.flags.syn == 1 && tcp.flags.ack == 0     — SYN scans
# http.request.method == "POST"                 — HTTP POST
# dns.qry.name contains "evil"                  — Suspicious DNS
# tcp.analysis.retransmission                   — Retransmissions
# frame.len > 1500                              — Jumbo frames

# Extract specific fields
tshark -r capture.pcap -Y 'http' -T fields -e http.host -e http.request.uri

# Statistics
tshark -r capture.pcap -z io,stat,1             # I/O stats per second
tshark -r capture.pcap -z conv,tcp              # TCP conversations
tshark -r capture.pcap -z endpoints,ip          # IP endpoints
```

---

## 3. Security-Focused Analysis

```bash
# === DETECT PORT SCANNING ===
# Look for many SYN packets from one source
sudo tcpdump -i eth0 'tcp[tcpflags] == tcp-syn' | \
    awk '{print $3}' | cut -d. -f1-4 | sort | uniq -c | sort -rn

# === DETECT ARP SPOOFING ===
sudo tcpdump -i eth0 arp | grep -i "is-at"
# Multiple MACs claiming same IP = ARP spoof!

# === DETECT DNS TUNNELING ===
# Look for unusually long DNS queries
tshark -r capture.pcap -Y 'dns.qry.name.len > 50' \
    -T fields -e dns.qry.name

# === DETECT DATA EXFILTRATION ===
# Large outbound transfers
tshark -r capture.pcap -z conv,tcp | sort -k 10 -rn | head

# === EXTRACT FILES FROM PCAP ===
# Export HTTP objects
tshark -r capture.pcap --export-objects http,exported_files/

# Extract with NetworkMiner or foremost
foremost -i capture.pcap -o extracted/
```

---

## 4. Other Traffic Analysis Tools

| Tool | Purpose |
|------|---------|
| **tcpdump** | CLI packet capture and basic analysis |
| **Wireshark** | GUI packet analysis (gold standard) |
| **tshark** | CLI Wireshark (scripting-friendly) |
| **tcpflow** | Reconstruct TCP streams |
| **ngrep** | Grep for network traffic |
| **NetworkMiner** | Forensic analysis of pcap files |
| **Zeek (Bro)** | Network security monitoring framework |
| **Suricata** | IDS/IPS with protocol analysis |

```bash
# ngrep — grep for network traffic
sudo ngrep -d eth0 'password'         # Search for "password" in traffic
sudo ngrep -d eth0 'GET|POST' port 80 # HTTP methods

# tcpflow — reconstruct streams
sudo tcpflow -i eth0 port 80          # Capture HTTP streams as files
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **tcpdump** | CLI packet capture, BPF filters |
| **tshark** | CLI Wireshark, display filters |
| **BPF** | Berkeley Packet Filter syntax |
| **pcap** | Standard packet capture file format |
| **Display filters** | Wireshark/tshark filter syntax |
| **Security** | Detect scans, spoofing, exfiltration |

---

## Quick Revision Questions

1. **What is the difference between tcpdump and tshark filters?**
2. **How do you capture only DNS traffic with tcpdump?**
3. **How do you detect a port scan in a packet capture?**
4. **What command saves tcpdump output to a file?**
5. **How do you extract files from a pcap?**

---

[← Previous: Password Crackers](03-password-crackers.md) | [Next: Forensic Tools →](05-forensic-tools.md)

---

*Unit 7 - Topic 4 of 6 | [Back to README](../README.md)*
