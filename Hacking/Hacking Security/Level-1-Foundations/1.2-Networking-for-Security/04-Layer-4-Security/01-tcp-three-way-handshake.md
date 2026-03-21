# TCP Three-Way Handshake

## Unit 4 - Topic 1 | Layer 4 Security

---

## Overview

The **TCP three-way handshake** establishes a reliable connection between two hosts before data transfer begins. Understanding this process is critical for security professionals because many attacks — SYN floods, session hijacking, port scanning — target or exploit the handshake mechanism.

---

## 1. The Three-Way Handshake

```
┌──────────────────────────────────────────────────────────────────┐
│                  TCP THREE-WAY HANDSHAKE                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Client                                     Server               │
│    │                                           │                 │
│    │ 1. SYN (seq=100)                          │                 │
│    │──────────────────────────────────────────►│                 │
│    │  "I want to connect"                      │                 │
│    │  Client → SYN_SENT state                  │                 │
│    │                                           │                 │
│    │ 2. SYN-ACK (seq=300, ack=101)             │                 │
│    │◄──────────────────────────────────────────│                 │
│    │  "OK, I acknowledge"                      │                 │
│    │  Server → SYN_RECEIVED state              │                 │
│    │                                           │                 │
│    │ 3. ACK (seq=101, ack=301)                 │                 │
│    │──────────────────────────────────────────►│                 │
│    │  "Connection established"                 │                 │
│    │  Both → ESTABLISHED state                 │                 │
│    │                                           │                 │
│    │═══════ DATA TRANSFER BEGINS ═════════════│                 │
│    │                                           │                 │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. TCP Connection States

| State | Description | Security Note |
|-------|-------------|---------------|
| **LISTEN** | Server waiting for connections | Open port — attack target |
| **SYN_SENT** | Client sent SYN, waiting for SYN-ACK | Normal client behavior |
| **SYN_RECEIVED** | Server received SYN, sent SYN-ACK | SYN flood fills this state |
| **ESTABLISHED** | Connection fully open, data flowing | Session hijacking target |
| **FIN_WAIT_1/2** | Connection closing (initiator side) | FIN scan reconnaissance |
| **TIME_WAIT** | Connection closed, waiting to ensure clean shutdown | Resource exhaustion possible |
| **CLOSE_WAIT** | Waiting for application to close | Indicates application bug if many |

---

## 3. TCP Connection Termination (Four-Way)

```
Client                              Server
  │                                    │
  │ 1. FIN ───────────────────────────►│
  │    "I'm done sending"             │
  │                                    │
  │ 2. ACK ◄──────────────────────────│
  │    "OK, acknowledged"             │
  │                                    │
  │ 3. FIN ◄──────────────────────────│
  │    "I'm done too"                 │
  │                                    │
  │ 4. ACK ───────────────────────────►│
  │    "Connection closed"            │
  │                                    │
```

---

## 4. TCP Flags and Security

| Flag | Name | Purpose | Scan/Attack Use |
|:----:|------|---------|----------------|
| **SYN** | Synchronize | Initiate connection | SYN scan, SYN flood |
| **ACK** | Acknowledge | Confirm receipt | ACK scan (firewall mapping) |
| **FIN** | Finish | Close connection | FIN scan (stealth recon) |
| **RST** | Reset | Abort connection | RST injection, connection reset |
| **PSH** | Push | Deliver data immediately | Normal data transfer |
| **URG** | Urgent | Priority data | Xmas scan (with FIN+PSH) |

### Scan Types Based on Flags

```
SYN Scan (Half-open):     SYN → SYN-ACK = open, RST = closed
Connect Scan:             Full handshake → reliable but logged
FIN Scan:                 FIN → no response = open, RST = closed
Xmas Scan:                FIN+PSH+URG → no response = open
NULL Scan:                No flags → no response = open
ACK Scan:                 ACK → RST = unfiltered (firewall mapping)
```

---

## 5. Viewing TCP Connections

```bash
# Windows — View TCP connections
netstat -ano | findstr ESTABLISHED
netstat -ano | findstr LISTENING

# Linux — View TCP connections
ss -tuln                     # Listening ports
ss -tupn                     # Established connections
netstat -tupan               # All connections

# View TCP states
netstat -an | awk '{print $6}' | sort | uniq -c | sort -rn

# Wireshark filter
tcp.flags.syn == 1 && tcp.flags.ack == 0    # SYN only (new connections)
tcp.stream eq 5                              # Follow specific stream
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Handshake** | SYN → SYN-ACK → ACK (three packets to connect) |
| **SYN_RECEIVED** | State exploited by SYN flood attacks |
| **TCP Flags** | SYN, ACK, FIN, RST, PSH, URG — each has security implications |
| **Scanning** | Different flag combinations reveal port states |
| **Termination** | FIN → ACK → FIN → ACK (four-way close) |

---

## Quick Revision Questions

1. **Describe the three steps of the TCP handshake.**
2. **What TCP state is exploited in a SYN flood attack?**
3. **What is the difference between a SYN scan and a Connect scan?**
4. **What flags are set in a Xmas scan?**
5. **How can you view active TCP connections on Windows and Linux?**

---

[Next: TCP Attacks →](02-tcp-attacks.md)

---

*Unit 4 - Topic 1 of 5 | [Back to README](../README.md)*
