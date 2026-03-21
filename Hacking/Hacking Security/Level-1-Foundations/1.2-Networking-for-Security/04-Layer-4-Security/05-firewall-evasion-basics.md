# Firewall Evasion Basics

## Unit 4 - Topic 5 | Layer 4 Security

---

## Overview

**Firewall evasion** techniques are used by penetration testers and attackers to bypass network firewalls and intrusion detection systems. Understanding these techniques is essential for both offense (pen testing) and defense (configuring firewalls that resist evasion).

---

## 1. Common Evasion Techniques

```
┌──────────────────────────────────────────────────────────────────┐
│                  FIREWALL EVASION METHODS                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. PACKET FRAGMENTATION — Split packets to avoid inspection    │
│  2. SOURCE PORT MANIPULATION — Use trusted port numbers         │
│  3. DECOY SCANNING — Hide real scan among fake sources          │
│  4. TIMING — Slow scans to avoid detection thresholds           │
│  5. TUNNELING — Hide traffic inside allowed protocols           │
│  6. ENCODING — Obfuscate payloads to bypass signatures          │
│  7. PROTOCOL ABUSE — Use allowed protocols for unintended use   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Technique Details

### Fragmentation

```bash
# Split scan packets into small fragments
nmap -f target.com             # 8-byte fragments
nmap --mtu 16 target.com       # Custom MTU

# Why it works: Stateless firewalls inspect individual packets
# Fragments may not contain enough header info for rules to match
# Modern NGFW reassembles before inspection → defeats this
```

### Source Port Manipulation

```bash
# Use trusted source ports that firewalls often allow
nmap --source-port 53 target.com    # Pretend to be DNS response
nmap --source-port 80 target.com    # Pretend to be HTTP response

# Common trusted ports:
# 53 (DNS) — often allowed through firewalls
# 80 (HTTP) — web traffic usually permitted
# 443 (HTTPS) — encrypted web usually permitted
# 88 (Kerberos) — AD traffic
```

### Decoy Scanning

```bash
# Mix your scan with decoy IPs
nmap -D 10.0.0.1,10.0.0.2,10.0.0.3,ME target.com
nmap -D RND:10 target.com      # 10 random decoy IPs

# Defender sees scans from multiple IPs → hard to identify real scanner
# Decoy IPs must be alive (or defender can filter them out)
```

### Timing Evasion

```bash
# Slow scan to avoid IDS detection
nmap -T0 target.com     # Paranoid: 5 min between probes
nmap -T1 target.com     # Sneaky: 15 sec between probes
nmap -T2 target.com     # Polite: 400ms between probes

# IDS often has threshold: "100+ SYNs in 10 seconds = scan"
# Slow timing stays below these thresholds
```

### Protocol Tunneling

```
┌──────────────────────────────────────────────────────────────┐
│              TUNNELING TECHNIQUES                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  DNS Tunneling:                                              │
│  Hide data inside DNS queries/responses                      │
│  → DNS (port 53) is almost always allowed                    │
│  Tools: dnscat2, iodine                                      │
│                                                              │
│  HTTP Tunneling:                                             │
│  Encapsulate traffic inside HTTP requests                    │
│  → HTTP/HTTPS traffic is almost always allowed               │
│  Tools: HTTPTunnel, Chisel                                   │
│                                                              │
│  ICMP Tunneling:                                             │
│  Hide data inside ICMP ping packets                          │
│  → Sometimes ICMP is allowed through firewalls               │
│  Tools: ptunnel, icmpsh                                      │
│                                                              │
│  SSH Tunneling:                                              │
│  Forward ports through SSH connections                       │
│  ssh -L 8080:internal:80 user@jumphost                       │
│  ssh -D 1080 user@jumphost    (SOCKS proxy)                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Advanced Evasion Techniques

| Technique | Description |
|-----------|-------------|
| **IP ID Header Manipulation** | Use idle scan (-sI) to scan through a zombie host |
| **TTL Manipulation** | Set TTL to expire after IDS but before target |
| **Payload Encoding** | Encode exploits (Base64, XOR) to bypass signatures |
| **TLS/SSL Encryption** | Hide malicious traffic inside encrypted channels |
| **IPv6** | Use IPv6 if only IPv4 is being monitored |
| **Application Layer** | Use allowed app protocols (DNS, HTTP) for C2 |

### Nmap Idle Scan (Zombie Scan)

```bash
# Use another host (zombie) to scan — ultimate stealth
nmap -sI zombie_host:80 target.com

# Target only sees scan from zombie, not from you
# Requires zombie with predictable IP ID values
```

---

## 4. Defending Against Evasion

| Defense | Counters |
|---------|----------|
| **NGFW with DPI** | Deep packet inspection defeats fragmentation, encoding |
| **Full packet reassembly** | Reassemble fragments before rule evaluation |
| **TLS inspection** | Decrypt and inspect HTTPS traffic |
| **Behavioral analysis** | Detect anomalous patterns regardless of encoding |
| **DNS monitoring** | Detect DNS tunneling (large/frequent queries) |
| **Egress filtering** | Block unexpected outbound connections |
| **Multiple detection points** | IDS at multiple network segments |
| **Log correlation** | SIEM correlates events across devices |

---

## Summary Table

| Technique | Purpose | Tool Example |
|-----------|---------|-------------|
| **Fragmentation** | Bypass stateless firewalls | `nmap -f` |
| **Source Port** | Appear as trusted traffic | `nmap --source-port 53` |
| **Decoys** | Hide among fake scanners | `nmap -D RND:10` |
| **Slow Timing** | Stay below IDS thresholds | `nmap -T0` |
| **Tunneling** | Hide in allowed protocols | dnscat2, SSH tunnels |
| **Idle Scan** | Scan through zombie host | `nmap -sI zombie` |

---

## Quick Revision Questions

1. **Why does fragmentation help evade firewalls?**
2. **Why is source port 53 effective for evasion?**
3. **How do decoy scans make it harder to identify the real attacker?**
4. **What is DNS tunneling and why is it hard to block?**
5. **What is an idle (zombie) scan and why is it the stealthiest?**
6. **How can NGFWs defeat most evasion techniques?**

---

[← Previous: Port Scanning Techniques](04-port-scanning-techniques.md)

---

*Unit 4 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Application Layer Protocols →](../05-Application-Layer-Protocols/01-dns-and-attacks.md)*
