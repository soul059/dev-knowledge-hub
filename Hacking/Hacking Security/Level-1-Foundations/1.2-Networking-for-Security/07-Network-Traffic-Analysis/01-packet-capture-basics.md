# Packet Capture Basics

## Unit 7 - Topic 1 | Network Traffic Analysis

---

## Overview

**Packet capture** (pcap) is the process of intercepting and recording network traffic for analysis. It's fundamental to **network forensics**, **troubleshooting**, **intrusion detection**, and **penetration testing**. Understanding how to capture, filter, and analyze packets is a core skill for security professionals.

---

## 1. How Packet Capture Works

```
┌──────────────────────────────────────────────────────────────────┐
│              PACKET CAPTURE METHODS                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  METHOD 1: PROMISCUOUS MODE                                     │
│  NIC captures ALL traffic on the segment (not just its own)     │
│  → Works on hubs, or after ARP spoofing/MAC flooding            │
│                                                                  │
│  METHOD 2: SPAN/MIRROR PORT                                     │
│  Switch copies traffic from one port to a monitoring port       │
│  ┌────────────────────────────────┐                             │
│  │ SWITCH                         │                             │
│  │  Port 1 (user) ──copy──► Port 24 (monitor/IDS)              │
│  └────────────────────────────────┘                             │
│                                                                  │
│  METHOD 3: NETWORK TAP                                          │
│  Hardware device that copies traffic inline                     │
│  ┌──────┐    ┌─────┐    ┌──────┐                               │
│  │ Host │────│ TAP │────│ Host │                               │
│  └──────┘    └──┬──┘    └──────┘                               │
│                 │ Copy to monitor                               │
│              ┌──┴──┐                                            │
│              │ IDS │                                            │
│              └─────┘                                            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Capture Tools

| Tool | Platform | Description |
|------|----------|-------------|
| **tcpdump** | Linux/macOS | Command-line packet capture |
| **Wireshark** | All | GUI packet analyzer (most popular) |
| **tshark** | All | Wireshark's CLI version |
| **dumpcap** | All | Lightweight capture (part of Wireshark) |
| **NetworkMiner** | Windows | Network forensic analyzer |
| **Netmon** | Windows | Microsoft's network monitor |

---

## 3. tcpdump — Essential Commands

```bash
# Basic capture
tcpdump -i eth0                        # Capture on eth0
tcpdump -i any                         # All interfaces
tcpdump -c 100                         # Capture 100 packets

# Save to file
tcpdump -i eth0 -w capture.pcap        # Save to pcap file
tcpdump -r capture.pcap                # Read from file

# Common filters
tcpdump -i eth0 host 10.0.0.5          # Traffic to/from IP
tcpdump -i eth0 port 80                # HTTP traffic
tcpdump -i eth0 src 10.0.0.5           # From specific source
tcpdump -i eth0 dst port 443           # To HTTPS
tcpdump -i eth0 tcp                    # TCP only
tcpdump -i eth0 udp port 53           # DNS traffic

# Verbose output
tcpdump -i eth0 -v                     # Verbose
tcpdump -i eth0 -vvv                   # Maximum verbosity
tcpdump -i eth0 -X                     # Hex + ASCII dump
tcpdump -i eth0 -A                     # ASCII dump

# Complex filters
tcpdump -i eth0 'tcp port 80 and host 10.0.0.5'
tcpdump -i eth0 'not port 22'         # Exclude SSH
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'  # SYN packets
```

---

## 4. Capture Best Practices

| Practice | Description |
|----------|-------------|
| **Capture full packets** | Use `-s 0` for full packet capture (not truncated) |
| **Save to file** | Always save raw pcap for later analysis |
| **Timestamp** | Ensure accurate timestamps for correlation |
| **Filter at capture** | Use BPF filters to reduce volume |
| **Rotate files** | Use ring buffers for continuous capture |
| **Legal considerations** | Ensure authorization before capturing traffic |
| **Encrypt storage** | Captured data may contain credentials |

```bash
# Rotating capture (10 files of 100MB each)
tcpdump -i eth0 -w capture.pcap -C 100 -W 10

# Capture with timestamps
tcpdump -i eth0 -tttt -w timestamped.pcap
```

---

## 5. BPF (Berkeley Packet Filter) Syntax

```
Primitives: host, net, port, src, dst, tcp, udp, icmp
Operators:  and, or, not (&&, ||, !)

Examples:
host 10.0.0.5                  # All traffic to/from IP
src host 10.0.0.5              # Source is IP
dst port 443                   # Destination port 443
tcp and port 80                # TCP port 80
not port 22                    # Everything except SSH
net 192.168.1.0/24             # Entire subnet
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Packet Capture** | Record network traffic for analysis |
| **Methods** | Promiscuous mode, SPAN port, network TAP |
| **tcpdump** | Primary CLI capture tool |
| **Wireshark** | Primary GUI analyzer |
| **BPF Filters** | Efficient filtering at capture time |
| **Legal** | Always get authorization before capturing |

---

## Quick Revision Questions

1. **What are three methods for capturing network traffic?**
2. **What is the difference between a SPAN port and a TAP?**
3. **Write a tcpdump command to capture HTTP traffic and save to file.**
4. **What is promiscuous mode?**
5. **Why should captured traffic be stored securely?**

---

[Next: Wireshark Fundamentals →](02-wireshark-fundamentals.md)

---

*Unit 7 - Topic 1 of 6 | [Back to README](../README.md)*
