# Historical DNS Data

## Unit 3 - Topic 6 | Domain and DNS Reconnaissance

---

## Overview

**Historical DNS data** shows how a domain's DNS records have changed over time. This reveals previous hosting providers, origin IPs before CDN adoption, abandoned subdomains, infrastructure migrations, and misconfigurations that may have been corrected but still provide useful intelligence.

---

## 1. Why Historical DNS Matters

```
WHAT HISTORICAL DNS REVEALS:

ORIGIN IP BEFORE CDN:
├── 2020: target.com → 203.0.113.10 (direct hosting)
├── 2021: target.com → 104.16.x.x  (moved to Cloudflare)
└── The origin IP (203.0.113.10) may still be active!

INFRASTRUCTURE CHANGES:
├── Old hosting providers
├── Previous nameservers
├── Migrated services
└── Abandoned but still-running servers

PAST MISCONFIGURATIONS:
├── Internal IPs that were briefly exposed
├── Test subdomains that were live temporarily
├── Old mail servers
└── Previously used services
```

---

## 2. Historical DNS Sources

| Source | URL | Data Type |
|--------|-----|-----------|
| **SecurityTrails** | securitytrails.com | Full DNS history, WHOIS |
| **ViewDNS.info** | viewdns.info | IP history, reverse IP |
| **DNSHistory** | dnshistory.org | Record changes |
| **CompleteDNS** | completedns.com | Zone file database |
| **RapidDNS** | rapiddns.io | DNS records database |
| **PassiveTotal** | community.riskiq.com | Passive DNS |
| **VirusTotal** | virustotal.com | DNS resolutions |
| **DNSDB** | dnsdb.info | Passive DNS database |

---

## 3. Practical Usage

```bash
# === SECURITYTRAILS API ===
curl "https://api.securitytrails.com/v1/history/target.com/dns/a" \
  -H "apikey: YOUR_API_KEY"

# Output shows historical A records:
# 2019-01: 198.51.100.5 (DigitalOcean)
# 2020-06: 203.0.113.10 (AWS)
# 2021-03: 104.16.132.229 (Cloudflare)

# === VIEWDNS.INFO ===
# Web: viewdns.info/iphistory/?domain=target.com
# Shows all historical IPs for a domain

# === VIRUSTOTAL ===
# Search: virustotal.com/gui/domain/target.com/relations
# Shows passive DNS resolutions over time

# === PASSIVE DNS WITH TOOLS ===
# Amass with passive DNS
amass intel -d target.com -whois

# DNSRecon
dnsrecon -d target.com -t std
```

---

## 4. Intelligence Extraction

```
HISTORICAL DNS ANALYSIS:

TARGET: target.com

TIMELINE:
2018 │ A: 192.0.2.50 (Hosting: Rackspace)
     │ MX: mail.target.com (self-hosted email)
     │ NS: ns1.rackspace.com
     │
2019 │ A: 203.0.113.10 (Hosting: AWS EC2)
     │ MX: aspmx.l.google.com (migrated to Google)
     │ NS: ns-1234.awsdns-12.org
     │
2020 │ A: 203.0.113.10 (still AWS)
     │ Added: dev.target.com → 203.0.113.40
     │ Added: staging.target.com → 203.0.113.41
     │
2021 │ A: 104.16.132.229 (moved behind Cloudflare)
     │ Removed: dev.target.com (but server still running?)
     │ NS: ns1.cloudflare.com
     │
2024 │ Current state (Cloudflare CDN)

ACTIONABLE INTEL:
├── Origin IP 203.0.113.10 — may still be accessible directly
├── dev.target.com removed from DNS — server may still be running
├── Old Rackspace IP 192.0.2.50 — may have been reused
└── AWS nameserver reveals cloud provider
```

---

## 5. Wayback Machine for Web History

```bash
# WEB ARCHIVE (web.archive.org):
# Not DNS-specific but complementary

# Check historical versions of target website
# web.archive.org/web/*/target.com

# INTELLIGENCE FROM WEB ARCHIVES:
# ├── Old pages that revealed internal info
# ├── Removed contact pages with employee details
# ├── Old technology stacks (jQuery, PHP versions)
# ├── Configuration files that were briefly public
# ├── robots.txt changes (what they started hiding)
# └── Removed subdomains that were linked

# WAYBACKURLS tool:
# waybackurls target.com | sort -u
# Extracts all URLs ever archived for the domain
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Historical DNS** | Record changes over time |
| **Origin IP** | Pre-CDN IPs may still be accessible |
| **Old subdomains** | Removed from DNS but servers may be running |
| **Tools** | SecurityTrails, ViewDNS, VirusTotal |
| **Web archive** | Historical website versions |
| **Value** | Bypass CDN, find forgotten infrastructure |

---

## Quick Revision Questions

1. **Why is historical DNS data valuable for recon?**
2. **How can you find the origin IP of a CDN-protected site?**
3. **Name 3 sources for historical DNS data.**
4. **What does a removed subdomain in DNS history indicate?**
5. **How does the Wayback Machine complement DNS history?**

---

[← Previous: DNS Record Analysis](05-dns-record-analysis.md)

---

*Unit 3 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Network Reconnaissance →](../04-Network-Reconnaissance/01-ip-range-identification.md)*
