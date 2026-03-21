# DNS Record Analysis

## Unit 3 - Topic 5 | Domain and DNS Reconnaissance

---

## Overview

**DNS record analysis** involves deeply examining all DNS records associated with a target to extract intelligence about infrastructure, email configuration, service providers, and potential misconfigurations. Each record type tells a different story about the target's setup.

---

## 1. Comprehensive Record Analysis

```bash
# FULL DNS ANALYSIS WORKFLOW:

# Step 1: Get all records
dig target.com ANY +noall +answer
dig target.com A +short
dig target.com AAAA +short
dig target.com MX +short
dig target.com NS +short
dig target.com TXT +short
dig target.com SOA +short
dig target.com SRV +short
dig target.com CNAME +short
```

---

## 2. Record-by-Record Intelligence

| Record | Example | Intelligence |
|--------|---------|-------------|
| **A** | `203.0.113.10` | Server IP, hosting provider |
| **MX** | `aspmx.l.google.com` | Google Workspace for email |
| **NS** | `ns1.cloudflare.com` | Cloudflare DNS (likely WAF too) |
| **TXT** | `v=spf1 include:_spf.google.com` | Confirms Google email |
| **TXT** | `MS=ms12345` | Microsoft 365 verified |
| **TXT** | `docker-verification=abc` | Uses Docker Hub |
| **CNAME** | `blog → target.ghost.io` | Blog hosted on Ghost |
| **SRV** | `_sip._tcp → sipserver` | SIP/VoIP service |
| **SOA** | `admin.target.com` | Admin email address |

---

## 3. TXT Record Deep Dive

```bash
dig target.com TXT +short

# COMMON TXT RECORDS AND MEANING:

# SPF (email sending authorization)
"v=spf1 include:_spf.google.com include:sendgrid.net ~all"
# → Uses Google + SendGrid for email

# DKIM (email authentication)
dig default._domainkey.target.com TXT
# → Email signing configured

# DMARC (email policy)
dig _dmarc.target.com TXT
"v=DMARC1; p=reject; rua=mailto:dmarc@target.com"
# → Strict email policy (p=reject)

# SERVICE VERIFICATION TOKENS:
"google-site-verification=..."         → Google Search Console
"MS=ms..."                             → Microsoft 365
"facebook-domain-verification=..."     → Facebook Business
"docusign=..."                         → DocuSign
"atlassian-domain-verification=..."    → Atlassian (Jira/Confluence)
"docker-verification=..."              → Docker Hub
"stripe-verification=..."              → Stripe payments

# EACH TOKEN REVEALS A SERVICE THE TARGET USES!
```

---

## 4. Finding the Real IP Behind CDN

```bash
# Many targets hide behind CDN (Cloudflare, Akamai, etc.)
# Finding the origin IP bypasses CDN protections

# METHOD 1: Historical DNS records
# SecurityTrails, ViewDNS.info historical records
# Check IPs before CDN was added

# METHOD 2: DNS records that bypass CDN
dig target.com MX +short          # Mail server may be origin
dig ftp.target.com A +short       # FTP may point to origin
dig cpanel.target.com A +short    # cPanel direct

# METHOD 3: SSL certificate search
# Search Censys/Shodan for target's SSL cert on non-CDN IPs
# shodan search "ssl.cert.subject.cn:target.com"

# METHOD 4: Email headers
# Send email to target, check received headers
# "Received: from [origin-ip]"

# METHOD 5: Subdomain analysis
# Not all subdomains go through CDN
# Direct subdomains may reveal origin IP
```

---

## 5. DNS Misconfiguration Detection

```
COMMON DNS MISCONFIGURATIONS:

1. UNRESTRICTED ZONE TRANSFER
   → Test: dig @ns1.target.com target.com axfr

2. DANGLING CNAME (Subdomain Takeover)
   → CNAME points to decommissioned service
   → blog.target.com → target.herokuapp.com (deleted)

3. INTERNAL IP LEAKAGE
   → DNS records exposing 10.x.x.x, 172.16.x.x, 192.168.x.x
   → dev.target.com → 10.0.0.5 (private IP leaked!)

4. MISSING SPF/DMARC
   → No email authentication = spoofing possible
   → dig target.com TXT | grep spf
   → dig _dmarc.target.com TXT

5. WILDCARD DNS
   → *.target.com resolves to an IP
   → Any subdomain works = larger attack surface

6. STALE RECORDS
   → Records pointing to decommissioned servers
   → IPs no longer owned by target
```

---

## Summary Table

| Record | Reveals | Tool |
|--------|---------|:---:|
| **A/AAAA** | Server IPs | dig |
| **MX** | Email provider | dig |
| **NS** | DNS hosting | dig |
| **TXT** | SPF, services, verifications | dig |
| **CNAME** | Third-party services | dig |
| **SOA** | Admin email, zone info | dig |
| **SRV** | Internal services | dig |

---

## Quick Revision Questions

1. **What can TXT records reveal about a target?**
2. **How do service verification tokens help pen testers?**
3. **How can you find the real IP behind a CDN?**
4. **What DNS misconfigurations should you look for?**
5. **Why analyze SOA records?**

---

[← Previous: Subdomain Discovery](04-subdomain-discovery.md) | [Next: Historical DNS Data →](06-historical-dns-data.md)

---

*Unit 3 - Topic 5 of 6 | [Back to README](../README.md)*
