# BGP and ASN Information

## Unit 4 - Topic 2 | Network Reconnaissance

---

## Overview

**BGP (Border Gateway Protocol)** is the routing protocol of the internet, and **ASN (Autonomous System Number)** identifies organizations that control their own IP routing. Understanding BGP/ASN data reveals an organization's complete internet presence — all IP prefixes, peering relationships, and network architecture.

---

## 1. Key Concepts

| Term | Definition |
|------|-----------|
| **ASN** | Unique number identifying an autonomous system |
| **BGP** | Protocol for exchanging routing info between ASNs |
| **Prefix** | IP address range advertised via BGP (e.g., 203.0.113.0/24) |
| **Peering** | Direct interconnection between two ASNs |
| **Transit** | Paying an ISP to carry your traffic |
| **Route** | Path through multiple ASNs to reach a destination |

---

## 2. ASN Discovery

```bash
# === FIND ASN FOR AN ORGANIZATION ===

# Method 1: From an IP address
whois 203.0.113.10 | grep -i "originas\|origin"
# Output: OriginAS: AS12345

# Method 2: BGP toolkit
# bgp.he.net — search by company name
# "Target Corporation" → AS12345

# Method 3: RADB
whois -h whois.radb.net "Target Corporation"

# Method 4: Amass
amass intel -org "Target Corporation"
# Returns: AS12345 — Target Corporation

# === LIST ALL PREFIXES FOR AN ASN ===
whois -h whois.radb.net -- '-i origin AS12345'
# route: 203.0.113.0/24
# route: 198.51.100.0/24
# route: 192.0.2.0/25

# === BGP LOOKING GLASS TOOLS ===
# bgp.he.net/AS12345           — Hurricane Electric
# bgpview.io/asn/12345         — BGPView
# stat.ripe.net/AS12345        — RIPE Stat
```

---

## 3. Intelligence from BGP Data

```
ASN ANALYSIS FOR AS12345 (Target Corporation):

PREFIXES ANNOUNCED:
├── 203.0.113.0/24    (256 IPs — main data center)
├── 198.51.100.0/24   (256 IPs — secondary site)
├── 192.0.2.0/25      (128 IPs — office network)
└── Total: 640 IP addresses

PEERING RELATIONSHIPS:
├── AS13335 (Cloudflare)        — CDN provider
├── AS16509 (Amazon/AWS)        — Cloud provider
├── AS15169 (Google)            — Email/Cloud
├── AS3356  (Lumen/CenturyLink) — Transit provider
└── AS174   (Cogent)            — Transit provider

INTELLIGENCE:
├── Uses Cloudflare and AWS
├── Email via Google Workspace
├── Two transit providers (redundancy)
├── Three separate IP allocations (3 locations?)
└── Total 640 IPs to potentially scan
```

---

## 4. BGP History and Anomalies

```bash
# === BGP STREAM (real-time monitoring) ===
# bgpstream.com — real-time BGP updates

# === BGP HISTORY ===
# RIPE Stat: stat.ripe.net
# Shows: routing history, visibility, prefix changes

# ANOMALIES TO LOOK FOR:
# ├── New prefix announcements (new infrastructure)
# ├── Withdrawn prefixes (decommissioned but still running?)
# ├── More specific announcements (potential hijack)
# ├── Origin AS changes (migration or attack)
# └── Unstable routes (flapping = network issues)

# === TRACEROUTE (trace the BGP path) ===
traceroute -A target.com
# Shows each hop with ASN information
# Reveals transit providers and network path
```

---

## 5. Practical Application

| Use Case | BGP/ASN Tool | Result |
|----------|:---:|--------|
| Find all target IPs | ASN prefix list | Complete IP inventory |
| Identify hosting | Peering data | Cloud/CDN providers |
| Map network | Traceroute + ASN | Network topology |
| Find related orgs | ASN neighbors | Partner networks |
| Detect changes | BGP history | Infrastructure changes |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **ASN** | Identifies organization's network |
| **BGP prefixes** | All IP ranges announced by target |
| **Peering** | Shows providers and partners |
| **Tools** | bgp.he.net, BGPView, RIPE Stat |
| **History** | Track infrastructure changes over time |
| **Value** | Complete IP inventory for scanning |

---

## Quick Revision Questions

1. **What is an ASN and how do you find one?**
2. **How do you list all IP prefixes for an ASN?**
3. **What intelligence can peering relationships reveal?**
4. **Name 3 BGP looking glass tools.**
5. **Why monitor BGP history for a target?**

---

[← Previous: IP Range Identification](01-ip-range-identification.md) | [Next: Network Mapping →](03-network-mapping.md)

---

*Unit 4 - Topic 2 of 5 | [Back to README](../README.md)*
