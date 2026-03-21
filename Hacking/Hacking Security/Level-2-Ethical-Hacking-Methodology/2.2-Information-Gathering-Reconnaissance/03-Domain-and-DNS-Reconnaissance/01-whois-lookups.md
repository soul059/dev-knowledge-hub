# WHOIS Lookups

## Unit 3 - Topic 1 | Domain and DNS Reconnaissance

---

## Overview

**WHOIS** is a protocol for querying databases that store information about registered domain names and IP addresses. WHOIS lookups reveal **registration details**, **contact information**, **nameservers**, and **registration dates** — all valuable for building a target profile.

---

## 1. What WHOIS Reveals

```
WHOIS QUERY RESULTS:

DOMAIN INFORMATION:
├── Domain name and status
├── Registration date (how long established?)
├── Expiration date (may lapse = opportunity)
├── Last updated date
├── Registrar (GoDaddy, Namecheap, etc.)
└── Nameservers (hosting provider clue)

REGISTRANT INFORMATION:
├── Name (may be real person or privacy service)
├── Organization
├── Street address
├── City, state, country
├── Phone number
├── Email address
└── ⚠️ Often hidden by privacy protection

TECHNICAL/ADMIN CONTACTS:
├── Technical contact (IT staff)
├── Admin contact (domain owner)
└── May differ from registrant
```

---

## 2. WHOIS Commands

```bash
# === DOMAIN WHOIS ===
whois target.com
whois -h whois.verisign-grs.com target.com  # Specific WHOIS server

# === IP ADDRESS WHOIS ===
whois 203.0.113.10
whois -h whois.arin.net 203.0.113.10        # ARIN (Americas)
whois -h whois.ripe.net 203.0.113.10        # RIPE (Europe)
whois -h whois.apnic.net 203.0.113.10       # APNIC (Asia-Pacific)

# === WEB-BASED WHOIS ===
# whois.domaintools.com — enhanced WHOIS with history
# who.is — clean interface
# whois.icann.org — ICANN WHOIS lookup

# === REVERSE WHOIS ===
# Find all domains registered by same person/org
# DomainTools reverse WHOIS
# whoxy.com — reverse WHOIS API
# viewdns.info/reversewhois
```

---

## 3. Interpreting WHOIS Data

```
EXAMPLE WHOIS OUTPUT:

Domain Name: TARGET.COM
Registry Domain ID: 12345678_DOMAIN_COM-VRSN
Registrar: GoDaddy.com, LLC
Creation Date: 2005-03-15T12:00:00Z      ← Established company
Updated Date: 2024-01-10T08:30:00Z       ← Recently updated
Expiry Date: 2025-03-15T12:00:00Z        ← Renewal coming up

Name Servers:
  NS1.CLOUDFLARE.COM                      ← Using Cloudflare DNS
  NS2.CLOUDFLARE.COM                      ← (may indicate CDN/WAF)

Registrant:
  Organization: Target Corporation
  State: California
  Country: US

INTELLIGENCE DERIVED:
├── Company is in California
├── Domain registered since 2005 (established)
├── Uses Cloudflare (CDN + potential WAF)
├── GoDaddy as registrar
└── Recently updated (active maintenance)
```

---

## 4. WHOIS Privacy and Limitations

| Situation | Impact on Recon |
|-----------|----------------|
| **Privacy protection** | Registrant info hidden behind proxy |
| **GDPR compliance** | EU registrations hide personal data |
| **Corporate registrar** | May use CSC, MarkMonitor for brand protection |
| **Expired domains** | May be re-registered by different owner |

```bash
# BYPASSING PRIVACY PROTECTION:
# 1. Check historical WHOIS (before privacy was added)
#    → DomainTools, SecurityTrails
# 2. Check other TLDs (.net, .org) — may lack privacy
# 3. Look at SSL certificate — may contain real info
# 4. Check DNS SOA record — may have admin email
# 5. Reverse WHOIS on known email/name
```

---

## 5. WHOIS for IP Addresses

```bash
# IP WHOIS shows network owner and allocation

whois 203.0.113.10

# OUTPUT:
# NetRange: 203.0.113.0 - 203.0.113.255
# CIDR: 203.0.113.0/24
# OrgName: Target Corporation
# OrgId: TARGETCORP
# Address: 123 Main Street
# City: San Francisco
# Country: US
# NetType: Direct Allocation

# REGIONAL INTERNET REGISTRIES (RIRs):
# ARIN    — Americas
# RIPE    — Europe, Middle East
# APNIC   — Asia-Pacific
# LACNIC  — Latin America
# AFRINIC — Africa
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **WHOIS** | Registration database for domains and IPs |
| **Domain info** | Dates, registrar, nameservers |
| **Contact info** | Often hidden by privacy services |
| **Reverse WHOIS** | Find all domains by same registrant |
| **IP WHOIS** | Network owner, allocation, address |
| **Limitations** | GDPR, privacy protection services |

---

## Quick Revision Questions

1. **What information does a domain WHOIS lookup reveal?**
2. **How does IP WHOIS differ from domain WHOIS?**
3. **What is reverse WHOIS and when is it useful?**
4. **How has GDPR affected WHOIS data availability?**
5. **How can nameservers in WHOIS data provide intelligence?**

---

[Next: DNS Enumeration →](02-dns-enumeration.md)

---

*Unit 3 - Topic 1 of 6 | [Back to README](../README.md)*
