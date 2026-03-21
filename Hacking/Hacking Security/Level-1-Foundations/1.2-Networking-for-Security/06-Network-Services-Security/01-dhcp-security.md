# DHCP Security

## Unit 6 - Topic 1 | Network Services Security

---

## Overview

**DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP addresses, subnet masks, gateways, and DNS servers to network devices. Because DHCP operates without authentication, it's vulnerable to **rogue DHCP servers**, **DHCP starvation**, and **DHCP spoofing** attacks.

---

## 1. How DHCP Works (DORA Process)

```
┌──────────────────────────────────────────────────────────────────┐
│                  DHCP DORA PROCESS                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Client                                    DHCP Server          │
│    │                                           │                │
│    │ 1. DISCOVER (broadcast)                   │                │
│    │═══════════════════════════════════════════►│                │
│    │  "I need an IP address!"                  │                │
│    │                                           │                │
│    │ 2. OFFER (unicast/broadcast)              │                │
│    │◄═══════════════════════════════════════════│                │
│    │  "Here's 192.168.1.100 available"         │                │
│    │                                           │                │
│    │ 3. REQUEST (broadcast)                    │                │
│    │═══════════════════════════════════════════►│                │
│    │  "I'll take 192.168.1.100 please"         │                │
│    │                                           │                │
│    │ 4. ACKNOWLEDGE (unicast)                  │                │
│    │◄═══════════════════════════════════════════│                │
│    │  "Confirmed! IP: .100, GW: .1, DNS: .53" │                │
│    │                                           │                │
│    │  D = Discover                             │                │
│    │  O = Offer                                │                │
│    │  R = Request                              │                │
│    │  A = Acknowledge                          │                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. DHCP Attacks

### Rogue DHCP Server

```
┌──────────────────────────────────────────────────────────────────┐
│              ROGUE DHCP SERVER ATTACK                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Attacker sets up fake DHCP server on the network.              │
│  Responds to DISCOVER before legitimate server.                 │
│                                                                  │
│  Victim's DHCP config (from attacker):                          │
│  • Gateway: ATTACKER'S IP  → All traffic through attacker (MITM)│
│  • DNS: ATTACKER'S DNS    → DNS hijacking (phishing)            │
│                                                                  │
│  Result: Man-in-the-Middle for ALL victim traffic               │
│                                                                  │
│  Tool: Ettercap, Metasploit dhcp module, custom dnsmasq         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### DHCP Starvation

```
Attacker sends thousands of DHCP DISCOVER messages
with random MAC addresses → exhausts entire IP pool.

Legitimate devices can't get IP addresses → DoS

Tool: yersinia, dhcpstarv, Gobbler

Often combined with rogue DHCP:
1. Starve legitimate DHCP server
2. Set up rogue DHCP to serve clients
3. Hand out attacker-controlled gateway/DNS
```

---

## 3. DHCP Snooping (Primary Defense)

```
┌──────────────────────────────────────────────────────────────────┐
│              DHCP SNOOPING                                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Switch feature that validates DHCP traffic.                    │
│                                                                  │
│  TRUSTED PORTS: Connected to legitimate DHCP server/uplinks     │
│  → DHCP offers allowed                                          │
│                                                                  │
│  UNTRUSTED PORTS: All user-facing ports                         │
│  → DHCP offers BLOCKED (prevents rogue DHCP)                   │
│  → DHCP requests rate-limited (prevents starvation)            │
│                                                                  │
│  BINDING TABLE (built automatically):                           │
│  ┌──────────────┬──────────────────┬────────┬──────┐           │
│  │ IP Address   │ MAC Address      │ Port   │ VLAN │           │
│  ├──────────────┼──────────────────┼────────┼──────┤           │
│  │ 192.168.1.10 │ AA:BB:CC:11:22:33│ Gi0/1  │ 10   │           │
│  │ 192.168.1.11 │ AA:BB:CC:44:55:66│ Gi0/2  │ 10   │           │
│  └──────────────┴──────────────────┴────────┴──────┘           │
│                                                                  │
│  This table is used by DAI and IP Source Guard.                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

```cisco
! Enable DHCP Snooping
ip dhcp snooping
ip dhcp snooping vlan 10,20,30

! Trust legitimate DHCP server port
interface GigabitEthernet0/24
  ip dhcp snooping trust

! Rate-limit on user ports
interface range GigabitEthernet0/1-20
  ip dhcp snooping limit rate 10

! Verify
show ip dhcp snooping
show ip dhcp snooping binding
```

---

## 4. Additional DHCP Security

| Defense | Description |
|---------|-------------|
| **DHCP Snooping** | Validate DHCP messages, build binding table |
| **Port Security** | Limit MACs per port (prevents starvation) |
| **IP Source Guard** | Validates source IP against DHCP binding table |
| **802.1X** | Authenticate before network access |
| **DHCP Logging** | Monitor DHCP assignments for anomalies |
| **Static Reservations** | Assign fixed IPs to critical devices |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DHCP** | Auto-assigns IP config — no authentication |
| **Rogue DHCP** | Fake DHCP server → MITM via malicious gateway/DNS |
| **DHCP Starvation** | Exhaust IP pool → DoS |
| **DHCP Snooping** | Switch validates DHCP, builds trusted binding table |
| **Binding Table** | IP-MAC-Port mapping used by DAI and IP Source Guard |

---

## Quick Revision Questions

1. **What are the four steps of the DHCP DORA process?**
2. **How does a rogue DHCP server enable MITM?**
3. **What is DHCP starvation and what tool performs it?**
4. **How does DHCP snooping prevent rogue DHCP servers?**
5. **What is the DHCP snooping binding table used for?**

---

[Next: DNS Security (DNSSEC) →](02-dns-security-dnssec.md)

---

*Unit 6 - Topic 1 of 5 | [Back to README](../README.md)*
