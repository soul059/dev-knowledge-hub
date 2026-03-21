# DNS Enumeration

## Unit 3 - Topic 2 | Domain and DNS Reconnaissance

---

## Overview

**DNS enumeration** extracts as much information as possible from a target's DNS infrastructure — records, nameservers, mail servers, and associated IP addresses. DNS is the backbone of internet naming, and its records reveal the target's entire online footprint.

---

## 1. DNS Record Types

| Record | Purpose | Intelligence Value |
|--------|---------|:---:|
| **A** | Domain → IPv4 address | Server locations |
| **AAAA** | Domain → IPv6 address | IPv6 infrastructure |
| **MX** | Mail servers | Email infrastructure |
| **NS** | Nameservers | DNS hosting provider |
| **TXT** | Text records | SPF, DKIM, verification tokens |
| **CNAME** | Aliases | Service providers, CDN |
| **SOA** | Start of Authority | Admin email, zone info |
| **SRV** | Service records | Internal services (LDAP, SIP) |
| **PTR** | IP → Domain (reverse) | Discover hostnames |

---

## 2. DNS Enumeration Commands

```bash
# === DIG (Domain Information Groper) ===
dig target.com                         # Default A record
dig target.com ANY                     # All records
dig target.com MX                      # Mail servers
dig target.com NS                      # Nameservers
dig target.com TXT                     # Text records (SPF/DKIM)
dig target.com SOA                     # Start of Authority
dig target.com AAAA                    # IPv6 addresses

# Query specific nameserver
dig @ns1.target.com target.com ANY

# === NSLOOKUP ===
nslookup target.com
nslookup -type=MX target.com
nslookup -type=NS target.com
nslookup -type=TXT target.com

# === HOST ===
host target.com                        # Quick lookup
host -t MX target.com                  # MX records
host -t NS target.com                  # NS records
host -a target.com                     # All records

# === DNSRECON ===
dnsrecon -d target.com -t std          # Standard enumeration
dnsrecon -d target.com -t brt -D wordlist.txt  # Brute force
dnsrecon -d target.com -t rvl          # Reverse lookup

# === DNSENUM ===
dnsenum target.com                     # Comprehensive DNS enum
dnsenum --enum target.com              # Full enumeration
```

---

## 3. Reverse DNS Lookups

```bash
# Reverse DNS: IP → Hostname

# Single IP
dig -x 203.0.113.10
host 203.0.113.10
nslookup 203.0.113.10

# Scan entire range
dnsrecon -r 203.0.113.0/24 -t rvl

# Using nmap
nmap -sL 203.0.113.0/24               # List scan (DNS only)

# WHY REVERSE DNS MATTERS:
# ├── Discover hostnames on known IP ranges
# ├── Find shared hosting (multiple domains on one IP)
# ├── Identify internal naming conventions
# └── Map IP addresses to services
```

---

## 4. Interpreting DNS Results

```
EXAMPLE: target.com DNS Records

A Records:
  target.com         → 203.0.113.10     (main server)
  www.target.com     → 104.16.132.229   (Cloudflare CDN)
  mail.target.com    → 203.0.113.20     (mail server)
  vpn.target.com     → 203.0.113.30     (VPN endpoint)
  dev.target.com     → 10.0.0.5         (⚠️ internal IP leaked!)

MX Records:
  10 alt1.aspmx.l.google.com          (Google Workspace)
  20 alt2.aspmx.l.google.com

NS Records:
  ns1.cloudflare.com                   (Cloudflare DNS)
  ns2.cloudflare.com

TXT Records:
  "v=spf1 include:_spf.google.com ~all"  (SPF → uses Google)
  "google-site-verification=abc123"        (Google verified)
  "MS=ms12345678"                          (Microsoft 365 token)

INTELLIGENCE:
├── Main site behind Cloudflare CDN
├── Email via Google Workspace
├── Also uses Microsoft 365 (verification token)
├── VPN endpoint at .30 (potential target)
├── Dev server leaking internal IP (10.0.0.5)
└── Direct server IP: 203.0.113.10
```

---

## 5. DNS Enumeration Tools Summary

| Tool | Best For | Speed |
|------|---------|:---:|
| **dig** | Manual queries, specific records | Fast |
| **nslookup** | Quick lookups (Windows-friendly) | Fast |
| **dnsrecon** | Comprehensive enumeration | Medium |
| **dnsenum** | Automated full enum | Medium |
| **fierce** | DNS brute force | Slow |
| **dnsmap** | Subdomain brute force | Slow |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DNS records** | A, MX, NS, TXT, CNAME, SOA, SRV |
| **dig** | Most versatile DNS query tool |
| **Reverse DNS** | IP → hostname mapping |
| **TXT records** | Reveal email providers, services |
| **Leaked internal IPs** | Dev/staging may expose internal ranges |
| **MX records** | Identify email infrastructure |

---

## Quick Revision Questions

1. **What DNS record types are most valuable for recon?**
2. **What tool is best for manual DNS queries?**
3. **What can TXT records reveal about a target?**
4. **Why is reverse DNS useful?**
5. **How can DNS records leak internal IP addresses?**

---

[← Previous: WHOIS Lookups](01-whois-lookups.md) | [Next: Zone Transfers →](03-zone-transfers.md)

---

*Unit 3 - Topic 2 of 6 | [Back to README](../README.md)*
