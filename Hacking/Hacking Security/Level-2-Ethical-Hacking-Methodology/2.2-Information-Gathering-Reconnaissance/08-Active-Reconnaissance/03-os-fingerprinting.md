# OS Fingerprinting

## Unit 8 - Topic 3 | Active Reconnaissance

---

## Overview

**OS fingerprinting** identifies the operating system running on target hosts by analyzing how they respond to network probes. Knowing the OS version helps select appropriate exploits, predict system behavior, and understand the target environment.

---

## 1. Active vs Passive OS Fingerprinting

```
ACTIVE OS FINGERPRINTING:
├── Sends specially crafted packets to target
├── Analyzes responses (TCP/IP stack behavior)
├── More accurate but detectable
├── Tools: Nmap (-O), xprobe2
└── Example: TCP window size, TTL, DF bit

PASSIVE OS FINGERPRINTING:
├── Observes existing network traffic
├── No packets sent to target
├── Less accurate but undetectable
├── Tools: p0f, NetworkMiner
└── Example: Sniff packets and analyze TTL/window

COMPARISON:
┌──────────┬──────────────┬──────────────┐
│ Aspect   │ Active       │ Passive      │
├──────────┼──────────────┼──────────────┤
│ Accuracy │ ★★★★★       │ ★★★          │
│ Stealth  │ ★★           │ ★★★★★       │
│ Speed    │ Fast         │ Depends      │
│ Position │ Remote       │ Must sniff   │
│ Detection│ IDS alerts   │ Undetectable │
└──────────┴──────────────┴──────────────┘
```

---

## 2. Nmap OS Detection

```bash
# === BASIC OS DETECTION ===
sudo nmap -O target.com
# Requires at least one open and one closed port
# Reports: OS family, version, accuracy percentage

# === AGGRESSIVE OS DETECTION ===
sudo nmap -O --osscan-guess target.com
# Attempts to guess even when uncertain

sudo nmap -O --osscan-limit target.com
# Only attempts on ideal targets (1 open + 1 closed)

# === COMBINED SCANNING ===
sudo nmap -sS -sV -O target.com
# SYN scan + service versions + OS detection

# === EXAMPLE OUTPUT ===
# OS detection:
# Running: Linux 5.X
# OS CPE: cpe:/o:linux:linux_kernel:5
# OS details: Linux 5.4 - 5.15
# Accuracy: 96%

# === OS DETECTION + VERSION ===
sudo nmap -A target.com
# -A = aggressive: OS detection + version + scripts + traceroute
```

---

## 3. How OS Fingerprinting Works

```
TCP/IP STACK DIFFERENCES:

Each OS implements TCP/IP slightly differently:

┌─────────────────────────┬─────────┬─────────┬─────────┐
│ TCP/IP Parameter        │ Linux   │ Windows │ macOS   │
├─────────────────────────┼─────────┼─────────┼─────────┤
│ Initial TTL             │ 64      │ 128     │ 64      │
│ TCP Window Size         │ 29200   │ 65535   │ 65535   │
│ DF (Don't Fragment)     │ Set     │ Set     │ Set     │
│ TCP Options Order       │ MSS,NOP │ MSS,NOP │ MSS,NOP │
│ Response to FIN         │ No resp │ RST     │ No resp │
│ ICMP Response           │ Yes     │ Varies  │ Yes     │
│ TCP Timestamp           │ Yes     │ No*     │ Yes     │
└─────────────────────────┴─────────┴─────────┴─────────┘

NMAP SENDS 16 PROBES:
├── TCP probes to open ports (various flag combos)
├── TCP probes to closed ports
├── UDP probe to closed port
├── ICMP echo request
└── Compares responses to database of 5,000+ OS signatures
```

---

## 4. Passive OS Fingerprinting

```bash
# === P0F (Passive OS Fingerprinting) ===
sudo p0f -i eth0
# Listens to network traffic passively
# Identifies OS from SYN packets observed

# Output:
# .-[ 203.0.113.10/80 -> 192.168.1.100/45678 (syn+ack) ]-
# | client   = 192.168.1.100/45678
# | os       = Linux 3.11 and newer
# | dist     = 0
# | params   = none

# === P0F FROM PCAP FILE ===
p0f -r capture.pcap
# Analyze pre-captured traffic

# === QUICK TTL-BASED GUESS ===
ping target.com
# TTL=64  → Linux/macOS
# TTL=128 → Windows
# TTL=255 → Network device (Cisco, etc.)

# Not definitive — TTL decrements per hop:
# Received TTL=121 = started at 128, 7 hops = Windows
# Received TTL=56  = started at 64, 8 hops = Linux
```

---

## 5. Evasion and Countermeasures

| Technique | Purpose |
|-----------|---------|
| **TTL spoofing** | Change default TTL to mislead |
| **TCP stack hardening** | Normalize TCP responses |
| **OS fingerprint scrubbing** | Proxy modifies responses |
| **Decoy scanning** | `-D decoy1,decoy2,ME` confuses IDS |
| **Fragmentation** | `-f` fragments packets |
| **Timing** | `-T0` or `-T1` slow scanning |

```bash
# === NMAP EVASION TECHNIQUES ===
# Decoy scan (hide among fake source IPs):
sudo nmap -O -D 192.168.1.100,192.168.1.101,ME target.com

# Fragmented packets:
sudo nmap -O -f target.com

# Source port manipulation:
sudo nmap -O --source-port 53 target.com
# Appear as DNS traffic
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **nmap -O** | Active OS fingerprinting |
| **p0f** | Passive OS fingerprinting (undetectable) |
| **TTL** | 64=Linux, 128=Windows, 255=network device |
| **TCP stack** | Each OS responds differently to probes |
| **Accuracy** | Active is more accurate than passive |
| **Evasion** | Decoys, fragmentation, timing |

---

## Quick Revision Questions

1. **What is the difference between active and passive OS fingerprinting?**
2. **What Nmap flag enables OS detection?**
3. **How does TTL help identify the operating system?**
4. **What tool performs passive OS fingerprinting?**
5. **What nmap technique uses decoy IPs to evade IDS?**

---

[← Previous: Service Detection](02-service-detection.md) | [Next: Vulnerability Identification →](04-vulnerability-identification.md)

---

*Unit 8 - Topic 3 of 5 | [Back to README](../README.md)*
