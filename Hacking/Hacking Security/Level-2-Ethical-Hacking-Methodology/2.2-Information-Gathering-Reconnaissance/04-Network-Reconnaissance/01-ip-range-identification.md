# IP Range Identification

## Unit 4 - Topic 1 | Network Reconnaissance

---

## Overview

**IP range identification** determines all IP addresses and network blocks owned or used by a target organization. This defines the boundaries of the network attack surface and ensures comprehensive scanning coverage during active reconnaissance.

---

## 1. Methods for Identifying IP Ranges

```bash
# === WHOIS (IP Range from Domain) ===
# Step 1: Find IP of domain
dig target.com A +short
# Result: 203.0.113.10

# Step 2: WHOIS on the IP
whois 203.0.113.10
# NetRange: 203.0.113.0 - 203.0.113.255
# CIDR: 203.0.113.0/24
# OrgName: Target Corporation

# === ASN LOOKUP ===
# Step 1: Find ASN for the organization
whois -h whois.radb.net -- '-i origin AS12345'
# Step 2: List all prefixes advertised by that ASN
whois -h whois.radb.net AS12345

# === ARIN/RIPE/APNIC ===
# ARIN: whois.arin.net (Americas)
whois -h whois.arin.net "n Target Corporation"
# Lists all network allocations for the organization

# === BGP TOOLKIT ===
# bgp.he.net — Hurricane Electric BGP Toolkit
# Search by company name, ASN, or IP
```

---

## 2. IP Range Discovery Tools

```bash
# === AMASS ===
amass intel -org "Target Corporation"
# Returns ASNs and IP ranges associated with org

amass intel -asn 12345
# Lists all domains and IPs for an ASN

# === SHODAN ===
shodan search "org:Target Corporation"
# Shows all internet-facing devices for the org

# === CENSYS ===
# Search: autonomous_system.name: "Target Corporation"

# === SPYSE / SECURITYTRAILS ===
# IP ranges and associated domains

# === DNSDUMPSTER ===
# dnsdumpster.com — visual map of target DNS/IP infrastructure

# === MX RECORDS → Additional IPs ===
dig target.com MX +short
# Mail servers may be on different IP ranges
```

---

## 3. Organizing IP Ranges

```
TARGET IP RANGES:

OWNED RANGES:
├── 203.0.113.0/24   (Primary — main servers)
├── 198.51.100.0/24  (Secondary — DR site)
└── 192.0.2.0/25     (Office network)

CLOUD RANGES (shared infrastructure):
├── AWS: Various (check security groups)
├── Azure: Various
└── GCP: Various

CDN IPs (not target-owned):
├── Cloudflare: 104.16.0.0/12
└── Akamai: Various
⚠️ Do NOT scan CDN IPs — they're shared!

THIRD-PARTY HOSTING:
├── GitHub Pages: 185.199.108.0/22
├── Heroku: Various
└── DigitalOcean: Various
```

---

## 4. Scope Validation

| IP Type | In Scope? | Notes |
|---------|:---------:|-------|
| **Owned ranges** | ✅ Yes | Confirmed in authorization |
| **Cloud instances** | ⚠️ Check | May need cloud provider approval |
| **CDN IPs** | ❌ No | Shared infrastructure |
| **Third-party hosted** | ⚠️ Check | Need separate authorization |
| **Partner networks** | ❌ Usually no | Unless explicitly scoped |

---

## 5. Reverse DNS on IP Ranges

```bash
# Discover hostnames across entire IP range
nmap -sL 203.0.113.0/24 | grep "scan report"
# Lists all reverse DNS entries

# Detailed reverse lookup
dnsrecon -r 203.0.113.0/24 -t rvl

# This reveals:
# 203.0.113.10 → www.target.com
# 203.0.113.11 → admin.target.com
# 203.0.113.20 → mail.target.com
# 203.0.113.50 → unknown (no reverse DNS)
# ...
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **IP ranges** | Network blocks owned by target |
| **WHOIS** | Query IP for network allocation |
| **ASN** | Autonomous System Number identifies all routes |
| **Amass** | Automated org → ASN → IP range discovery |
| **Scope** | Only scan authorized ranges |
| **Reverse DNS** | Map IPs to hostnames across ranges |

---

## Quick Revision Questions

1. **How do you find all IP ranges for an organization?**
2. **What is an ASN and how does it help?**
3. **Why should you NOT scan CDN IP ranges?**
4. **How does reverse DNS help map an IP range?**
5. **What tools automate IP range discovery?**

---

[Next: BGP and ASN Information →](02-bgp-and-asn-information.md)

---

*Unit 4 - Topic 1 of 5 | [Back to README](../README.md)*
