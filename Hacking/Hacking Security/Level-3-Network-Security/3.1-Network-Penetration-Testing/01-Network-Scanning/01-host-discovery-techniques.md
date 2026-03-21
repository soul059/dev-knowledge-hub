# Host Discovery Techniques

## Unit 1 - Topic 1 | Network Scanning

---

## Overview

**Host discovery** (also called ping sweeping or host enumeration) determines which IP addresses on a network have active hosts. This is the first step in network penetration testing — before scanning ports, you need to know which systems are alive.

---

## 1. Host Discovery Methods

```
HOST DISCOVERY HIERARCHY:

LAYER 2 (Data Link) — Local Network Only:
├── ARP requests (most reliable on LAN)
└── Tool: arp-scan, nmap -PR

LAYER 3 (Network):
├── ICMP Echo Request (ping)
├── ICMP Timestamp
├── ICMP Address Mask
└── Tool: nmap -PE, -PP, -PM

LAYER 4 (Transport):
├── TCP SYN to common ports (80, 443)
├── TCP ACK to common ports
├── UDP to common ports
└── Tool: nmap -PS, -PA, -PU

APPLICATION LAYER:
├── HTTP request
├── DNS query
└── SNMP query
```

---

## 2. Nmap Host Discovery Commands

```bash
# === ARP DISCOVERY (Layer 2 — LAN only) ===
sudo nmap -sn -PR 192.168.1.0/24
# Most reliable on local network
# Uses ARP: "Who has 192.168.1.x?"

# === ICMP ECHO (Ping) ===
sudo nmap -sn -PE 10.0.0.0/24
# Traditional ping sweep
# ⚠️ Often blocked by firewalls

# === TCP SYN DISCOVERY ===
sudo nmap -sn -PS80,443,22,445 10.0.0.0/24
# Send SYN to common ports
# Host alive if SYN/ACK or RST received
# Works even when ICMP is blocked!

# === TCP ACK DISCOVERY ===
sudo nmap -sn -PA80,443 10.0.0.0/24
# Send ACK to ports
# Host alive if RST received
# Bypasses stateless firewalls

# === UDP DISCOVERY ===
sudo nmap -sn -PU53,161,137 10.0.0.0/24
# Send UDP to common services
# Host alive if UDP response or ICMP port unreachable

# === COMBINED (DEFAULT for root) ===
sudo nmap -sn 10.0.0.0/24
# Default: ICMP echo + TCP SYN:443 + TCP ACK:80 + ICMP timestamp
# Best coverage with multiple probes

# === NO PING (Skip Discovery) ===
nmap -Pn 10.0.0.1
# Treat all hosts as alive — scan ports directly
# Use when discovery is blocked but you know host exists

# === LIST SCAN (No Probes) ===
nmap -sL 10.0.0.0/24
# Lists targets with reverse DNS — sends NO packets
```

---

## 3. Other Host Discovery Tools

```bash
# === ARP-SCAN (Layer 2 — Fast and Reliable) ===
sudo arp-scan 192.168.1.0/24
# Sends ARP requests to all IPs in range
# Shows: IP, MAC address, vendor
# Only works on local subnet

# === NETDISCOVER (Passive + Active ARP) ===
sudo netdiscover -i eth0 -r 192.168.1.0/24     # Active
sudo netdiscover -i eth0 -p                      # Passive (listen)

# === FPING (Fast Ping Sweep) ===
fping -asg 10.0.0.0/24 2>/dev/null
# -a: show alive
# -s: show stats
# -g: generate range

# === PING SWEEP (Native) ===
# Linux:
for i in $(seq 1 254); do
    ping -c 1 -W 1 192.168.1.$i &>/dev/null && echo "192.168.1.$i is alive"
done

# === MASSCAN (Large Networks) ===
masscan 10.0.0.0/8 -p 80,443 --rate 10000
# Discovers hosts by port responsiveness
# Extremely fast for large networks
```

---

## 4. Discovery Decision Matrix

| Scenario | Best Method | Command |
|----------|-------------|---------|
| **Local LAN** | ARP scan | `nmap -sn -PR 192.168.1.0/24` |
| **ICMP allowed** | Ping sweep | `nmap -sn -PE 10.0.0.0/24` |
| **ICMP blocked** | TCP SYN probe | `nmap -sn -PS80,443 target` |
| **Firewall present** | Combined probes | `nmap -sn target/24` |
| **Stealth needed** | Passive ARP / slow scan | `netdiscover -p` |
| **Known live host** | Skip discovery | `nmap -Pn target` |
| **Large network** | Masscan | `masscan 10.0.0.0/8 -p 80` |

---

## 5. Network Diagram

```
HOST DISCOVERY ON NETWORK 10.0.0.0/24:

Attacker (10.0.0.5)
    │
    ├──── ARP Request → 10.0.0.1 [Gateway]    → ALIVE ✅
    ├──── ARP Request → 10.0.0.2              → No response ❌
    ├──── ARP Request → 10.0.0.10 [Web Server] → ALIVE ✅
    ├──── ARP Request → 10.0.0.20 [DB Server]  → ALIVE ✅
    ├──── ARP Request → 10.0.0.30 [Printer]    → ALIVE ✅
    ├──── ARP Request → 10.0.0.50 [Workstation] → ALIVE ✅
    └──── ...remaining IPs → No response ❌

RESULTS: 5 alive hosts out of 254 possible
Next step: Port scan alive hosts
```

---

## Defensive Countermeasures

| Attack | Defense |
|--------|---------|
| ARP scanning | Port security, VLAN segmentation |
| ICMP sweep | Block ICMP at firewall |
| TCP SYN probe | Stateful firewall with drop rules |
| Host discovery | IDS alert on sweep patterns |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **ARP discovery** | Most reliable on LAN |
| **ICMP ping** | Often blocked by firewalls |
| **TCP SYN probe** | Works when ICMP is blocked |
| **Combined probes** | Default nmap -sn with root |
| **-Pn flag** | Skip discovery, assume alive |
| **arp-scan** | Fast Layer 2 discovery |

---

## Quick Revision Questions

1. **Why is ARP discovery the most reliable on a local network?**
2. **What does the `-sn` flag do in Nmap?**
3. **How does TCP SYN discovery work when ICMP is blocked?**
4. **When would you use the `-Pn` flag?**
5. **What tools perform passive host discovery?**

---

[Next: Port Scanning →](02-port-scanning.md)

---

*Unit 1 - Topic 1 of 6 | [Back to README](../README.md)*
