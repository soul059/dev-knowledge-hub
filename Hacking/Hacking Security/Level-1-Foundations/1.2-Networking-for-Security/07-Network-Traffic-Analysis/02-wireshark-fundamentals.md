# Wireshark Fundamentals

## Unit 7 - Topic 2 | Network Traffic Analysis

---

## Overview

**Wireshark** is the world's most popular network protocol analyzer. It captures and interactively analyzes network traffic in real-time, providing deep visibility into hundreds of protocols. For security professionals, Wireshark is essential for **forensics**, **incident response**, **malware analysis**, and **penetration testing**.

---

## 1. Wireshark Interface

```
┌──────────────────────────────────────────────────────────────┐
│ WIRESHARK INTERFACE                                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────┐              │
│  │ DISPLAY FILTER BAR                         │              │
│  │ tcp.port == 80 && ip.addr == 10.0.0.5     │              │
│  └────────────────────────────────────────────┘              │
│                                                              │
│  ┌────────────────────────────────────────────┐              │
│  │ PACKET LIST PANE                           │              │
│  │ #   Time    Src        Dst        Proto Len│              │
│  │ 1   0.000   10.0.0.5   10.0.0.1   TCP   66│              │
│  │ 2   0.001   10.0.0.1   10.0.0.5   TCP   66│              │
│  │ 3   0.002   10.0.0.5   10.0.0.1   HTTP 350│              │
│  └────────────────────────────────────────────┘              │
│                                                              │
│  ┌────────────────────────────────────────────┐              │
│  │ PACKET DETAILS PANE                        │              │
│  │ ▶ Ethernet II (Src: aa:bb, Dst: cc:dd)    │              │
│  │ ▶ Internet Protocol (Src: 10.0.0.5)       │              │
│  │ ▶ TCP (Src Port: 45123, Dst: 80)          │              │
│  │ ▶ HTTP (GET /index.html)                   │              │
│  └────────────────────────────────────────────┘              │
│                                                              │
│  ┌────────────────────────────────────────────┐              │
│  │ PACKET BYTES PANE (Hex + ASCII)            │              │
│  │ 0000  aa bb cc dd ee ff 11 22 33 44 55 66  │              │
│  └────────────────────────────────────────────┘              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Essential Display Filters

```
# PROTOCOL FILTERS
http                          # HTTP traffic
dns                           # DNS traffic
tcp                           # TCP traffic
arp                           # ARP traffic
icmp                          # ICMP traffic
tls                           # TLS/SSL traffic
smb                           # SMB traffic

# IP FILTERS
ip.addr == 10.0.0.5           # Traffic to/from IP
ip.src == 10.0.0.5            # Source IP
ip.dst == 10.0.0.1            # Destination IP

# PORT FILTERS
tcp.port == 80                # TCP port 80
tcp.dstport == 443            # Destination port 443
udp.port == 53                # UDP port 53 (DNS)

# TCP FLAGS
tcp.flags.syn == 1            # SYN packets
tcp.flags.syn == 1 && tcp.flags.ack == 0   # SYN only (new conns)
tcp.flags.rst == 1            # RST packets (connection refused)
tcp.flags.fin == 1            # FIN packets (connection close)

# SECURITY-FOCUSED FILTERS
http.request.method == "POST"  # POST requests (credential submission)
http.authbasic                 # Basic auth (cleartext credentials!)
ftp.request.command == "PASS"  # FTP password
smtp.req.command == "AUTH"     # SMTP authentication
dns.qry.name contains "evil"  # DNS queries for suspicious domains

# EXCLUSION
!(arp || dns)                 # Exclude ARP and DNS
!tcp.port == 22               # Exclude SSH

# COMPLEX
tcp.port == 80 && ip.addr == 10.0.0.5 && http.request.method == "POST"
```

---

## 3. Key Wireshark Features for Security

| Feature | How to Access | Security Use |
|---------|--------------|-------------|
| **Follow TCP Stream** | Right-click → Follow → TCP Stream | Read full conversation (HTTP, FTP) |
| **Expert Info** | Analyze → Expert Information | Find errors, warnings, anomalies |
| **Protocol Hierarchy** | Statistics → Protocol Hierarchy | See protocol distribution |
| **Conversations** | Statistics → Conversations | Identify top talkers |
| **IO Graphs** | Statistics → I/O Graphs | Detect traffic spikes |
| **Export Objects** | File → Export Objects → HTTP | Extract downloaded files |
| **Coloring Rules** | View → Coloring Rules | Highlight suspicious traffic |

---

## 4. Credential Capture Examples

```
# FTP Credentials (Follow TCP Stream)
220 FTP server ready
USER admin
331 Password required
PASS secretpassword123        ← Cleartext password!

# HTTP Basic Auth (base64 decoded)
Authorization: Basic YWRtaW46cGFzc3dvcmQ=
Decoded: admin:password       ← Cleartext credentials!

# HTTP POST Form (Follow TCP Stream)
POST /login HTTP/1.1
username=admin&password=secret ← Form credentials!
```

---

## 5. Useful Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| **Ctrl+E** | Start/Stop capture |
| **Ctrl+F** | Find packet |
| **Ctrl+G** | Go to packet number |
| **Ctrl+Shift+E** | Export specified packets |
| **Ctrl+/** | Apply display filter |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Wireshark** | GUI-based packet analyzer — industry standard |
| **Display Filters** | Filter captured traffic for analysis |
| **Follow Stream** | Reconstruct full TCP/HTTP conversations |
| **Export Objects** | Extract files transferred over HTTP |
| **Security Use** | Credential capture, malware traffic, forensics |

---

## Quick Revision Questions

1. **What are the three main panes in Wireshark's interface?**
2. **Write a display filter to show only HTTP POST requests.**
3. **How do you extract files from an HTTP capture?**
4. **What does "Follow TCP Stream" show you?**
5. **How can Wireshark capture cleartext credentials?**

---

[← Previous: Packet Capture Basics](01-packet-capture-basics.md) | [Next: Protocol Analysis →](03-protocol-analysis.md)

---

*Unit 7 - Topic 2 of 6 | [Back to README](../README.md)*
