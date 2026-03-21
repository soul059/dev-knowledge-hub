# Intrusion Detection and Prevention Systems (IDS/IPS)

## Unit 8 - Topic 2 | Network Security Devices

---

## Overview

**IDS** (Intrusion Detection System) monitors network traffic and alerts on suspicious activity. **IPS** (Intrusion Prevention System) does the same but can also **block** malicious traffic in real-time. Together, they provide critical visibility and protection against network attacks.

---

## 1. IDS vs IPS

```
┌──────────────────────────────────────────────────────────────┐
│              IDS vs IPS DEPLOYMENT                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  IDS (Detection Only — Passive):                             │
│  Traffic ─────────────────────► Destination                  │
│                │                                             │
│           Copy to IDS                                        │
│                │                                             │
│           ┌────┴────┐                                        │
│           │  IDS    │──► Alert! (no blocking)                │
│           └─────────┘                                        │
│                                                              │
│  IPS (Detection + Prevention — Inline):                      │
│  Traffic ────► ┌─────┐ ────► Destination                    │
│                │ IPS │                                       │
│                └──┬──┘                                       │
│                   │                                          │
│             Block + Alert!                                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

| Feature | IDS | IPS |
|---------|:---:|:---:|
| **Monitors traffic** | ✅ | ✅ |
| **Generates alerts** | ✅ | ✅ |
| **Blocks attacks** | ❌ | ✅ |
| **Inline deployment** | ❌ (passive/SPAN) | ✅ (inline) |
| **Latency impact** | None | Minimal |
| **False positive risk** | Alert only | May block legitimate traffic |

---

## 2. Detection Methods

| Method | How It Works | Pros | Cons |
|--------|-------------|------|------|
| **Signature-Based** | Match traffic against known attack patterns | Low false positives, fast | Can't detect unknown attacks |
| **Anomaly-Based** | Compare traffic to baseline | Detects zero-days | Higher false positives |
| **Policy-Based** | Alert on policy violations | Customizable | Requires manual rule creation |
| **Heuristic** | Use algorithms to detect suspicious behavior | Adaptive | Resource intensive |

---

## 3. Deployment Types

| Type | Description | Example |
|------|-------------|---------|
| **NIDS/NIPS** | Network-based — monitors network segments | Snort, Suricata at perimeter |
| **HIDS/HIPS** | Host-based — monitors individual hosts | OSSEC, Wazuh on servers |
| **Wireless IDS** | Monitors wireless traffic | Kismet, AirMagnet |
| **NBA** | Network Behavior Analysis | Darktrace, Vectra |

---

## 4. Snort — Open Source IDS/IPS

```bash
# Snort rule format
# action protocol src_ip src_port -> dst_ip dst_port (options)

# Detect SSH brute force
alert tcp any any -> $HOME_NET 22 (msg:"SSH Brute Force Attempt"; \
  flow:to_server,established; threshold:type both, track by_src, \
  count 5, seconds 60; sid:1000001; rev:1;)

# Detect SQL injection in HTTP
alert tcp any any -> $HOME_NET 80 (msg:"SQL Injection Attempt"; \
  flow:to_server,established; content:"' OR 1=1"; nocase; \
  sid:1000002; rev:1;)

# Detect ICMP flood
alert icmp any any -> $HOME_NET any (msg:"ICMP Flood"; \
  threshold:type both, track by_src, count 100, seconds 10; \
  sid:1000003; rev:1;)

# Running Snort
snort -c /etc/snort/snort.conf -i eth0        # IDS mode
snort -c /etc/snort/snort.conf -i eth0 -Q     # IPS mode (inline)
```

---

## 5. Alert Management

| Concept | Description |
|---------|-------------|
| **True Positive** | Correctly detected real attack ✅ |
| **False Positive** | Alert on legitimate traffic (noise) ⚠️ |
| **True Negative** | Correctly passed legitimate traffic ✅ |
| **False Negative** | Missed a real attack (most dangerous) ❌ |

---

## 6. Major IDS/IPS Solutions

| Solution | Type | Notes |
|----------|------|-------|
| **Snort** | Open source NIDS/NIPS | Most widely deployed, owned by Cisco |
| **Suricata** | Open source NIDS/NIPS | Multi-threaded, faster than Snort |
| **OSSEC/Wazuh** | Open source HIDS | Log analysis, file integrity, rootkit detection |
| **Zeek (Bro)** | Open source NSM | Network monitoring (not traditional IDS) |
| **Cisco Firepower** | Commercial | Integrated NGFW + IPS |
| **Palo Alto** | Commercial | Threat prevention module |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **IDS** | Detects and alerts only (passive) |
| **IPS** | Detects and blocks (inline, active) |
| **Signature** | Known attack patterns — low false positive |
| **Anomaly** | Deviation from normal — detects zero-days |
| **Snort** | Most popular open-source IDS/IPS |
| **False Negative** | Missed attack — worst outcome |

---

## Quick Revision Questions

1. **What is the key difference between IDS and IPS?**
2. **Why are false negatives more dangerous than false positives?**
3. **Write a Snort rule to detect traffic to port 4444.**
4. **What is the difference between NIDS and HIDS?**
5. **Name 3 open-source IDS/IPS solutions.**

---

[← Previous: Firewalls](01-firewalls.md) | [Next: Web Application Firewalls →](03-waf.md)

---

*Unit 8 - Topic 2 of 6 | [Back to README](../README.md)*
