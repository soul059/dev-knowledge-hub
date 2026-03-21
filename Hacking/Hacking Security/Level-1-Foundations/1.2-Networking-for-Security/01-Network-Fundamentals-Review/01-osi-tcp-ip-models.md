# OSI and TCP/IP Models (Security Perspective)

## Unit 1 - Topic 1 | Network Fundamentals Review

---

## Overview

The **OSI (Open Systems Interconnection)** and **TCP/IP** models describe how data travels across networks. Understanding each layer from a **security perspective** reveals where attacks occur, what vulnerabilities exist, and what defenses to deploy at each level.

---

## 1. The OSI Model — 7 Layers

```
┌──────────────────────────────────────────────────────────────────┐
│                    OSI MODEL (Top to Bottom)                     │
├──────┬──────────────┬──────────────────┬────────────────────────┤
│ Layer│    Name      │    Function       │  Security Concerns     │
├──────┼──────────────┼──────────────────┼────────────────────────┤
│  7   │ Application  │ User interface    │ XSS, SQLi, phishing   │
│  6   │ Presentation │ Encryption/format │ SSL stripping, crypto  │
│  5   │ Session      │ Session mgmt      │ Session hijacking      │
│  4   │ Transport    │ End-to-end comms  │ SYN flood, port scan   │
│  3   │ Network      │ Routing/addressing│ IP spoofing, MITM      │
│  2   │ Data Link    │ Frame delivery    │ ARP spoofing, MAC flood│
│  1   │ Physical     │ Bits on wire      │ Wiretapping, jamming   │
├──────┴──────────────┴──────────────────┴────────────────────────┤
│                                                                  │
│  MEMORY AID: "All People Seem To Need Data Processing"           │
│  (Layer 7 → Layer 1)                                             │
│                                                                  │
│  REVERSE:    "Please Do Not Throw Sausage Pizza Away"            │
│  (Layer 1 → Layer 7)                                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. The TCP/IP Model — 4 Layers

```
┌──────────────────────────────────────────────────────────────────┐
│              TCP/IP MODEL vs OSI MODEL                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TCP/IP Model          OSI Model             Protocols           │
│  ┌──────────────┐  ┌──────────────┐                             │
│  │              │  │ Application  │  HTTP, DNS, FTP, SSH,       │
│  │ Application  │  │ Presentation │  SMTP, SNMP, DHCP,         │
│  │              │  │ Session      │  TLS/SSL                    │
│  ├──────────────┤  ├──────────────┤                             │
│  │ Transport    │  │ Transport    │  TCP, UDP                   │
│  ├──────────────┤  ├──────────────┤                             │
│  │ Internet     │  │ Network      │  IP, ICMP, ARP, IGMP       │
│  ├──────────────┤  ├──────────────┤                             │
│  │ Network      │  │ Data Link    │  Ethernet, Wi-Fi,          │
│  │ Access       │  │ Physical     │  PPP, Frame Relay           │
│  └──────────────┘  └──────────────┘                             │
│                                                                  │
│  TCP/IP is what the internet ACTUALLY uses.                      │
│  OSI is the REFERENCE model for understanding layers.            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Security at Each OSI Layer

### Layer 1 — Physical

| Attack | Description | Defense |
|--------|-------------|---------|
| Wiretapping | Physical cable interception | Fiber optic, physical security |
| Jamming | Radio frequency interference (Wi-Fi) | Frequency hopping, detection |
| Physical access | Plugging in rogue devices | Port security, NAC |

### Layer 2 — Data Link

| Attack | Description | Defense |
|--------|-------------|---------|
| ARP Spoofing | Poisoning ARP cache to intercept traffic | Dynamic ARP Inspection (DAI) |
| MAC Flooding | Overflowing switch CAM table | Port security limits |
| VLAN Hopping | Jumping between VLANs | Disable DTP, prune VLANs |

### Layer 3 — Network

| Attack | Description | Defense |
|--------|-------------|---------|
| IP Spoofing | Faking source IP address | Ingress/egress filtering |
| ICMP Attacks | Ping flood, Smurf attack | Rate limiting, ICMP filtering |
| Route Hijacking | Manipulating routing tables | Route authentication |

### Layer 4 — Transport

| Attack | Description | Defense |
|--------|-------------|---------|
| SYN Flood | Exhausting server connections | SYN cookies, rate limiting |
| Port Scanning | Discovering open ports/services | Firewall, IDS |
| Session Hijacking | Stealing active TCP sessions | Encryption (TLS) |

### Layers 5-7 — Session/Presentation/Application

| Attack | Description | Defense |
|--------|-------------|---------|
| Session Hijacking | Stealing session tokens | Secure cookies, HTTPS |
| SSL Stripping | Downgrading HTTPS to HTTP | HSTS |
| SQL Injection | Injecting malicious queries | Input validation, WAF |
| XSS | Injecting client-side scripts | CSP, output encoding |

---

## 4. Data Encapsulation

```
APPLICATION:  [ Data                              ]
TRANSPORT:    [ TCP Header | Data                  ]  ← Segment
NETWORK:      [ IP Header  | TCP Header | Data     ]  ← Packet
DATA LINK:    [ Frame Hdr  | IP | TCP | Data | FCS ]  ← Frame
PHYSICAL:     [ 1010110110101110010101011010110...  ]  ← Bits

Each layer WRAPS the data from the layer above.
Attackers can target headers at ANY layer.
```

---

## 5. Key Ports to Know (Security)

| Port | Protocol | Service | Security Note |
|:----:|:--------:|---------|---------------|
| 20/21 | TCP | FTP | Sends credentials in cleartext |
| 22 | TCP | SSH | Secure remote access |
| 23 | TCP | Telnet | Cleartext — never use |
| 25 | TCP | SMTP | Email — often abused for spam |
| 53 | TCP/UDP | DNS | DNS tunneling, cache poisoning |
| 80 | TCP | HTTP | Unencrypted web traffic |
| 443 | TCP | HTTPS | Encrypted web traffic (TLS) |
| 445 | TCP | SMB | WannaCry, EternalBlue exploits |
| 3389 | TCP | RDP | Brute force target |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **OSI Model** | 7 layers — reference model for understanding networks |
| **TCP/IP Model** | 4 layers — the actual internet protocol stack |
| **Security View** | Every layer has unique attack vectors and defenses |
| **Encapsulation** | Data wrapped with headers at each layer — each a target |
| **Key Principle** | Defense in depth — protect at EVERY layer |

---

## Quick Revision Questions

1. **Name all 7 OSI layers and one attack per layer.**
2. **How does the TCP/IP model differ from the OSI model?**
3. **What is data encapsulation and why does it matter for security?**
4. **At which layer do ARP spoofing and MAC flooding occur?**
5. **Why is understanding the OSI model important for penetration testing?**

---

[Next: Network Devices →](02-network-devices.md)

---

*Unit 1 - Topic 1 of 5 | [Back to README](../README.md)*
