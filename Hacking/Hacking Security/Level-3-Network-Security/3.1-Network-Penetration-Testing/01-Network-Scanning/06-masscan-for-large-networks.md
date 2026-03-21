# Masscan for Large Networks

## Unit 1 - Topic 6 | Network Scanning

---

## Overview

**Masscan** is the fastest port scanner, capable of scanning the entire internet in under 6 minutes. It's designed for large-scale network scanning where Nmap would be too slow, then results are typically fed into Nmap for detailed service analysis.

---

## 1. Masscan vs Nmap

```
COMPARISON:

┌──────────────────┬──────────────┬──────────────────┐
│ Feature          │ Masscan      │ Nmap             │
├──────────────────┼──────────────┼──────────────────┤
│ Speed            │ ★★★★★       │ ★★★              │
│ Accuracy         │ ★★★          │ ★★★★★           │
│ Version detect   │ ❌ Limited   │ ✅ Full           │
│ Script engine    │ ❌ No        │ ✅ NSE            │
│ OS detection     │ ❌ No        │ ✅ Yes            │
│ Large networks   │ ✅ Designed  │ ⚠️ Very slow      │
│ Small targets    │ ⚠️ Overkill  │ ✅ Perfect        │
│ Stealth          │ ❌ Very noisy│ ✅ Evasion options │
└──────────────────┴──────────────┴──────────────────┘

WORKFLOW: Masscan (find open ports) → Nmap (detailed analysis)
```

---

## 2. Basic Masscan Usage

```bash
# === BASIC SCAN ===
masscan 10.0.0.0/24 -p 80,443
# Scan subnet for web ports

# === PORT RANGE ===
masscan 10.0.0.0/24 -p 1-65535
# All ports on subnet

# === RATE CONTROL ===
masscan 10.0.0.0/24 -p 80,443 --rate 1000
# 1000 packets per second

masscan 10.0.0.0/8 -p 80 --rate 100000
# 100K pps — very fast, very noisy!

# === MULTIPLE TARGETS ===
masscan 10.0.0.0/24 192.168.1.0/24 -p 80,443

# === COMMON PORTS ===
masscan 10.0.0.0/24 -p 21,22,23,25,53,80,110,111,135,139,143,443,445,993,995,1433,3306,3389,5432,5900,8080,8443

# === EXCLUDE HOSTS ===
masscan 10.0.0.0/24 -p 80 --excludefile exclude.txt

# === BANNER GRABBING ===
masscan 10.0.0.0/24 -p 80 --banners
# Basic banner grab (limited compared to Nmap)
```

---

## 3. Output and Integration

```bash
# === OUTPUT FORMATS ===
masscan 10.0.0.0/24 -p 80,443 -oL output.txt    # List format
masscan 10.0.0.0/24 -p 80,443 -oX output.xml     # XML (Nmap compatible!)
masscan 10.0.0.0/24 -p 80,443 -oG output.gnmap   # Grepable
masscan 10.0.0.0/24 -p 80,443 -oJ output.json    # JSON

# === LIST OUTPUT FORMAT ===
# open tcp 80 203.0.113.10 1707761337
# open tcp 443 203.0.113.10 1707761337
# open tcp 80 203.0.113.20 1707761338

# === MASSCAN → NMAP PIPELINE ===
# Step 1: Fast port discovery with Masscan
masscan 10.0.0.0/24 -p 1-65535 --rate 10000 -oL masscan_results.txt

# Step 2: Extract IPs and ports
cat masscan_results.txt | grep "open" | awk '{print $4}' | sort -u > alive_ips.txt
cat masscan_results.txt | grep "open" | awk '{print $4":"$3}' > ip_port_pairs.txt

# Step 3: Detailed Nmap scan on discovered ports
nmap -sV -sC -p $(cat masscan_results.txt | grep "open" | awk '{print $3}' | sort -un | paste -sd,) \
  -iL alive_ips.txt -oA detailed_scan

# === RUSTSCAN (Automated Pipeline) ===
rustscan -a 10.0.0.0/24 -- -sV -sC
# Automatically: fast port scan → feed to Nmap
```

---

## 4. Rate Selection Guide

| Network | Rate | Time for /24 |
|---------|:----:|:------------:|
| **Lab/CTF** | 100,000 pps | Seconds |
| **Internal pen test** | 10,000 pps | Minutes |
| **External target** | 1,000 pps | ~10 minutes |
| **Stealth (external)** | 100 pps | Hours |
| **ISP/production** | 500 pps | ~30 minutes |

```bash
# ⚠️ HIGH RATE WARNINGS:
# Rate too high → overwhelm target → DoS
# Rate too high → miss responses → false negatives
# Rate too high → trigger IDS → get blocked
# Rate too high → saturate your bandwidth

# SAFE DEFAULT:
masscan TARGET -p PORTS --rate 1000
```

---

## 5. Advanced Masscan Options

```bash
# === RESUME INTERRUPTED SCANS ===
masscan 10.0.0.0/8 -p 80 --rate 10000
# Press Ctrl+C to pause
# Creates: paused.conf

masscan --resume paused.conf
# Resumes from where it stopped

# === ADAPTER/INTERFACE ===
masscan 10.0.0.0/24 -p 80 -e eth0
# Specify network interface

# === SOURCE IP ===
masscan 10.0.0.0/24 -p 80 --adapter-ip 10.0.0.5
# Specify source IP address

# === RETRIES ===
masscan 10.0.0.0/24 -p 80 --retries 2
# Retransmit probes (more accurate, slower)

# === CONFIGURATION FILE ===
# Save settings:
masscan 10.0.0.0/24 -p 80 --echo > masscan.conf
# Run from config:
masscan -c masscan.conf
```

---

## Defensive Countermeasures

| Attack | Defense |
|--------|---------|
| High-rate scanning | Rate limiting at firewall |
| Port sweeping | IDS threshold alerts |
| Large-scale scan | Network flow monitoring |
| Banner grabbing | Minimize service banners |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Masscan** | Fastest scanner — entire internet in 6 min |
| **Rate control** | `--rate` controls packets per second |
| **Pipeline** | Masscan (ports) → Nmap (details) |
| **RustScan** | Automated Masscan → Nmap pipeline |
| **Resume** | `--resume paused.conf` for interrupted scans |
| **Output** | XML compatible with Nmap tools |

---

## Quick Revision Questions

1. **When should you use Masscan instead of Nmap?**
2. **What is the recommended Masscan → Nmap workflow?**
3. **How does rate selection affect scan results?**
4. **How do you resume an interrupted Masscan?**
5. **What is RustScan and how does it combine both tools?**

---

[← Previous: Output Formats and Parsing](05-output-formats-and-parsing.md)

---

*Unit 1 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Service Enumeration →](../02-Service-Enumeration/01-banner-grabbing.md)*
