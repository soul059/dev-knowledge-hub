# Network Scanning

## Unit 6 - Topic 1 | Scanning and Enumeration

---

## Overview

**Network scanning** is the process of discovering live hosts, open ports, and available services on a target network. It transitions from reconnaissance (gathering information) to active probing — sending packets to targets and analyzing responses.

---

## 1. Scanning Methodology

```
┌──────────────────────────────────────────────────────────────┐
│              SCANNING WORKFLOW                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  STEP 1: HOST DISCOVERY                                      │
│  "Which systems are alive?"                                  │
│  └── Ping sweep, ARP scan, TCP probe                        │
│                                                              │
│  STEP 2: PORT SCANNING                                       │
│  "Which ports are open on each host?"                        │
│  └── SYN scan, TCP connect, UDP scan                        │
│                                                              │
│  STEP 3: SERVICE DETECTION                                   │
│  "What services are running on each port?"                   │
│  └── Version detection, banner grabbing                      │
│                                                              │
│  STEP 4: OS DETECTION                                        │
│  "What operating system is each host running?"               │
│  └── TCP/IP fingerprinting                                   │
│                                                              │
│  STEP 5: VULNERABILITY SCANNING                              │
│  "What vulnerabilities exist?"                               │
│  └── CVE matching, exploit checking                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Host Discovery Techniques

```bash
# === ICMP PING SWEEP ===
nmap -sn 10.0.0.0/24                 # Standard ping sweep
nmap -sn -PE 10.0.0.0/24             # ICMP echo only
fping -a -g 10.0.0.0/24              # Fast ping sweep

# === TCP PROBE (when ICMP is blocked) ===
nmap -sn -PS22,80,443 10.0.0.0/24    # TCP SYN probe
nmap -sn -PA80 10.0.0.0/24           # TCP ACK probe

# === ARP SCAN (local network only) ===
arp-scan --interface=eth0 10.0.0.0/24
netdiscover -i eth0 -r 10.0.0.0/24
nmap -sn -PR 10.0.0.0/24             # ARP ping

# === COMPARISON ===
# ICMP: May be blocked by firewall
# TCP:  Often passes through firewalls
# ARP:  Most reliable on local network (can't be blocked)
```

---

## 3. Scan Optimization

```bash
# LARGE NETWORK SCANNING STRATEGY:

# Phase 1: Fast discovery (identify live hosts)
nmap -sn -T4 10.0.0.0/16 -oG alive_hosts.txt

# Phase 2: Quick port scan (top ports on live hosts)
nmap -sS --top-ports 100 -T4 -iL alive_hosts.txt -oA quick_scan

# Phase 3: Full scan on interesting hosts
nmap -sS -sV -p- -T4 interesting_host -oA full_scan

# Phase 4: Deep scan on high-value targets
nmap -sS -sV -sC -O -p- -T3 target -oA deep_scan

# === MASSCAN (for very large networks) ===
masscan 10.0.0.0/8 -p 80,443,22 --rate=10000 -oL results.txt
# ⚠️ Masscan is VERY fast but less accurate than nmap
# Use masscan for discovery, nmap for detailed scanning
```

---

## 4. Network Scanning Results

| Port State | Meaning |
|-----------|---------|
| **open** | Service is accepting connections |
| **closed** | Port is accessible but no service listening |
| **filtered** | Firewall is blocking probe packets |
| **unfiltered** | Port is accessible, state unknown |
| **open\|filtered** | Cannot determine if open or filtered |
| **closed\|filtered** | Cannot determine if closed or filtered |

---

## 5. Documentation

```
SCAN RESULTS DOCUMENTATION:

HOST: 10.0.0.5
STATUS: UP
OS: Ubuntu 22.04 (estimated)

PORT     STATE    SERVICE       VERSION
22/tcp   open     ssh           OpenSSH 8.9p1
80/tcp   open     http          nginx 1.22.1
443/tcp  open     https         nginx 1.22.1
3306/tcp filtered mysql         —
8080/tcp open     http-proxy    Apache Tomcat 9.0.68

NOTES:
• MySQL filtered (likely fw rule for internal only)
• Tomcat on 8080 — check for manager interface
• SSH may be vulnerable if not patched
```

---

## Summary Table

| Step | Purpose | Key Tool |
|------|---------|:--------:|
| **Host discovery** | Find live systems | nmap -sn |
| **Port scan** | Find open ports | nmap -sS |
| **Service detection** | Identify services | nmap -sV |
| **OS detection** | Identify operating system | nmap -O |
| **Large networks** | Fast initial scan | masscan |

---

## Quick Revision Questions

1. **What is the purpose of network scanning?**
2. **Why use TCP probes instead of ICMP for host discovery?**
3. **What does a "filtered" port state indicate?**
4. **When should you use masscan vs nmap?**
5. **What is the recommended scanning workflow for large networks?**

---

[Next: Port Scanning →](02-port-scanning.md)

---

*Unit 6 - Topic 1 of 6 | [Back to README](../README.md)*
