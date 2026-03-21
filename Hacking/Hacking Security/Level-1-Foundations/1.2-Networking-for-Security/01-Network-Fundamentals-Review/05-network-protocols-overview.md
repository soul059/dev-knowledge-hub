# Network Protocols Overview

## Unit 1 - Topic 5 | Network Fundamentals Review

---

## Overview

Network protocols define the rules for communication between devices. From a security perspective, understanding these protocols reveals their **weaknesses**, how attackers **exploit** them, and what **defenses** to apply. This topic provides a security-focused overview of the most important protocols.

---

## 1. Protocol Map by OSI Layer

```
┌──────────────────────────────────────────────────────────────────┐
│                PROTOCOLS BY OSI LAYER                             │
├──────┬───────────────────────────────────────────────────────────┤
│  L7  │ HTTP, HTTPS, FTP, SSH, DNS, SMTP, SNMP, DHCP, Telnet,   │
│      │ LDAP, RDP, SMB/CIFS, NTP                                │
├──────┼───────────────────────────────────────────────────────────┤
│  L6  │ TLS/SSL, MIME, ASCII, JPEG, compression                 │
├──────┼───────────────────────────────────────────────────────────┤
│  L5  │ NetBIOS, RPC, SOCKS, SIP                                │
├──────┼───────────────────────────────────────────────────────────┤
│  L4  │ TCP, UDP                                                 │
├──────┼───────────────────────────────────────────────────────────┤
│  L3  │ IP (IPv4/IPv6), ICMP, IGMP, IPsec, ARP                  │
├──────┼───────────────────────────────────────────────────────────┤
│  L2  │ Ethernet, 802.1Q, PPP, ARP, Wi-Fi (802.11)              │
├──────┼───────────────────────────────────────────────────────────┤
│  L1  │ Electrical signals, light, radio waves                   │
└──────┴───────────────────────────────────────────────────────────┘
```

---

## 2. Critical Protocols — Security Reference

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Guaranteed delivery (ACK, retransmit) | Best effort — no guarantee |
| **Order** | Ordered delivery | No ordering |
| **Speed** | Slower (overhead) | Faster (minimal overhead) |
| **Attack Surface** | SYN flood, session hijacking | UDP flood, amplification |
| **Use Cases** | HTTP, SSH, FTP, SMTP | DNS queries, VoIP, streaming, DHCP |

### Key Protocol Security Summary

| Protocol | Port | Transport | Secure? | Security Concern | Secure Alternative |
|----------|:----:|:---------:|:-------:|-----------------|-------------------|
| **HTTP** | 80 | TCP | ❌ | Cleartext — sniffable | HTTPS (443) |
| **HTTPS** | 443 | TCP | ✅ | Certificate validation | — |
| **FTP** | 20/21 | TCP | ❌ | Cleartext credentials | SFTP (22), FTPS |
| **Telnet** | 23 | TCP | ❌ | Cleartext everything | SSH (22) |
| **SSH** | 22 | TCP | ✅ | Brute force, key mgmt | Key-based auth |
| **DNS** | 53 | UDP/TCP | ❌ | Cache poisoning, tunneling | DNSSEC, DoH, DoT |
| **SMTP** | 25 | TCP | ❌ | Spoofing, relay | STARTTLS, SPF/DKIM/DMARC |
| **SNMP** | 161/162 | UDP | ⚠️ | v1/v2c use cleartext community strings | SNMPv3 |
| **DHCP** | 67/68 | UDP | ❌ | Starvation, rogue server | DHCP snooping |
| **NTP** | 123 | UDP | ⚠️ | Amplification attacks | NTP authentication |
| **RDP** | 3389 | TCP | ⚠️ | Brute force, BlueKeep | VPN + MFA + NLA |
| **SMB** | 445 | TCP | ⚠️ | EternalBlue, WannaCry | SMBv3, disable v1 |
| **LDAP** | 389 | TCP | ❌ | Cleartext credentials | LDAPS (636) |
| **IMAP** | 143 | TCP | ❌ | Cleartext | IMAPS (993) |
| **POP3** | 110 | TCP | ❌ | Cleartext | POP3S (995) |

---

## 3. Protocol Communication Flow

```
┌──────────────────────────────────────────────────────────────┐
│           HTTPS REQUEST FLOW                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client                              Server                  │
│    │                                    │                    │
│    │ 1. DNS Query (port 53/UDP)         │                    │
│    │───────────────────────────────────► │ DNS Server         │
│    │ ◄── IP: 93.184.216.34             │                    │
│    │                                    │                    │
│    │ 2. TCP 3-Way Handshake (port 443)  │                    │
│    │──── SYN ──────────────────────────►│                    │
│    │◄─── SYN-ACK ─────────────────────│                    │
│    │──── ACK ──────────────────────────►│                    │
│    │                                    │                    │
│    │ 3. TLS Handshake                   │                    │
│    │──── ClientHello ─────────────────►│                    │
│    │◄─── ServerHello + Certificate ───│                    │
│    │──── Key Exchange ────────────────►│                    │
│    │◄─── Finished ────────────────────│                    │
│    │                                    │                    │
│    │ 4. Encrypted HTTP Data             │                    │
│    │◄═══════════════════════════════════│                    │
│    │                                    │                    │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Cleartext vs Encrypted Protocols

```
┌──────────────────────────────────────────────────────────────┐
│          CLEARTEXT PROTOCOLS = SECURITY RISK                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CLEARTEXT ❌ (Sniffable)     ENCRYPTED ✅ (Secure)          │
│  ─────────────────────        ──────────────────             │
│  HTTP (80)          →         HTTPS (443)                    │
│  FTP (21)           →         SFTP (22) / FTPS               │
│  Telnet (23)        →         SSH (22)                       │
│  SMTP (25)          →         SMTPS (465) / STARTTLS (587)   │
│  IMAP (143)         →         IMAPS (993)                    │
│  POP3 (110)         →         POP3S (995)                    │
│  LDAP (389)         →         LDAPS (636)                    │
│  SNMP v1/v2c        →         SNMPv3 (auth + encryption)    │
│  DNS (53)           →         DoH (443) / DoT (853)          │
│                                                              │
│  RULE: If a protocol sends data in cleartext,                │
│  an attacker on the network can READ everything.             │
│  Always use encrypted alternatives.                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **TCP** | Connection-oriented, reliable — vulnerable to SYN flood |
| **UDP** | Connectionless, fast — vulnerable to amplification attacks |
| **Cleartext Protocols** | HTTP, FTP, Telnet, SMTP — ALWAYS replace with secure alternatives |
| **Key Ports** | 22 (SSH), 443 (HTTPS), 53 (DNS), 445 (SMB), 3389 (RDP) |
| **Security Principle** | Always prefer encrypted protocols over cleartext |

---

## Quick Revision Questions

1. **What is the difference between TCP and UDP?**
2. **Name 5 cleartext protocols and their encrypted alternatives.**
3. **Why is SNMP v1/v2c considered insecure?**
4. **What port does HTTPS use and what protocol provides the encryption?**
5. **Why is SMB port 445 a significant security concern?**
6. **Explain the flow of an HTTPS connection from DNS to data transfer.**

---

[← Previous: VLANs & Segmentation](04-vlans-and-segmentation.md)

---

*Unit 1 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Layer 2 Security →](../02-Layer-2-Security/01-mac-addresses.md)*
