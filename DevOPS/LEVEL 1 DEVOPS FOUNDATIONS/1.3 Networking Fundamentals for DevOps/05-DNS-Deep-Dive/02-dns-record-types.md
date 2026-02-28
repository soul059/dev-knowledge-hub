# Chapter 2: DNS Record Types

## Overview

DNS records are the instructions stored in authoritative nameservers. Each record type serves a specific purpose — from mapping domain names to IPs (A/AAAA) to email routing (MX), aliasing (CNAME), service discovery (SRV), and verification (TXT). Mastering record types is essential for managing domains, configuring services, and troubleshooting DNS issues.

---

## 2.1 Essential Record Types

```
┌──────────────────────────────────────────────────────────────┐
│              DNS RECORD TYPES                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  A Record (Address)                                          │
│  ┌─────────────────────────────────────────────┐             │
│  │ app.example.com  →  93.184.216.34           │             │
│  │ Maps hostname to IPv4 address                │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
│  AAAA Record (IPv6 Address)                                  │
│  ┌─────────────────────────────────────────────┐             │
│  │ app.example.com  →  2001:db8::1             │             │
│  │ Maps hostname to IPv6 address                │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
│  CNAME Record (Canonical Name / Alias)                       │
│  ┌─────────────────────────────────────────────┐             │
│  │ www.example.com  →  app.example.com         │             │
│  │ Alias: resolves to another domain name       │             │
│  │ ⚠ Cannot be at zone apex (example.com)       │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
│  MX Record (Mail Exchange)                                   │
│  ┌─────────────────────────────────────────────┐             │
│  │ example.com  →  10 mail1.example.com        │             │
│  │                  20 mail2.example.com        │             │
│  │ Routes email, priority (lower = preferred)   │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
│  TXT Record (Text)                                           │
│  ┌─────────────────────────────────────────────┐             │
│  │ example.com  →  "v=spf1 include:_spf..."    │             │
│  │ Verification, SPF, DKIM, DMARC, etc.         │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
│  NS Record (Nameserver)                                      │
│  ┌─────────────────────────────────────────────┐             │
│  │ example.com  →  ns1.example.com             │             │
│  │                  ns2.example.com             │             │
│  │ Delegates zone to nameservers                │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
│  SOA Record (Start of Authority)                             │
│  ┌─────────────────────────────────────────────┐             │
│  │ Zone metadata: primary NS, admin email,      │             │
│  │ serial, refresh, retry, expire, minimum TTL   │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 Complete Record Reference

| Record | Purpose | Example Value | TTL Typical |
|--------|---------|--------------|-------------|
| A | IPv4 address | 93.184.216.34 | 300-3600s |
| AAAA | IPv6 address | 2001:db8::1 | 300-3600s |
| CNAME | Alias to another name | app.example.com | 300-3600s |
| MX | Mail routing | 10 mail.example.com | 3600s |
| TXT | Text data (SPF, DKIM) | "v=spf1 ..." | 3600s |
| NS | Nameserver delegation | ns1.example.com | 86400s |
| SOA | Zone authority info | (serial, refresh, etc.) | 86400s |
| SRV | Service location | 10 5 5060 sip.example.com | 300s |
| PTR | Reverse lookup (IP→name) | 34.216.184.93.in-addr.arpa | 3600s |
| CAA | Certificate authority auth | 0 issue "letsencrypt.org" | 3600s |

---

## 2.3 CNAME vs ALIAS vs A Record

```
┌──────────────────────────────────────────────────────────────┐
│         CNAME vs ALIAS vs A RECORD                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  A Record:                                                   │
│  example.com → 93.184.216.34                                 │
│  ✓ Works at zone apex (naked domain)                         │
│  ✗ Must update when IP changes                               │
│                                                              │
│  CNAME Record:                                               │
│  www.example.com → app.example.com                           │
│  ✓ Follows target's IP changes automatically                 │
│  ✗ CANNOT be at zone apex (example.com)                      │
│  ✗ Extra DNS lookup required                                 │
│                                                              │
│  ALIAS Record (Route 53 specific):                           │
│  example.com → d123.cloudfront.net                           │
│  ✓ Works at zone apex!                                       │
│  ✓ Follows target's IP changes                               │
│  ✓ No extra DNS lookup (resolved server-side)                │
│  ✓ Free for AWS resources                                    │
│                                                              │
│  Common Pattern:                                             │
│  example.com    → ALIAS → ALB DNS name                       │
│  www.example.com → CNAME → example.com                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.4 SRV Records (Service Discovery)

```
┌──────────────────────────────────────────────────────────────┐
│              SRV RECORD FORMAT                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  _service._protocol.name  TTL  IN  SRV  priority weight port target │
│                                                              │
│  Example:                                                    │
│  _http._tcp.example.com  300  IN  SRV  10 60 8080 web1.example.com │
│  _http._tcp.example.com  300  IN  SRV  10 40 8080 web2.example.com │
│  _http._tcp.example.com  300  IN  SRV  20 0  8080 web3.example.com │
│                                                              │
│  Fields:                                                     │
│  - Priority: Lower = preferred (like MX)                     │
│  - Weight: Relative load distribution (60/40 split above)    │
│  - Port: Service port number                                 │
│  - Target: Hostname running the service                      │
│                                                              │
│  Used by: Kubernetes, SIP, LDAP, XMPP, Consul               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.5 TXT Records for Verification

```bash
# SPF (Sender Policy Framework) - Email authentication
example.com  TXT  "v=spf1 include:_spf.google.com include:amazonses.com ~all"

# DKIM (DomainKeys Identified Mail)
selector._domainkey.example.com  TXT  "v=DKIM1; k=rsa; p=MIGfMA0..."

# DMARC (Domain Message Authentication)
_dmarc.example.com  TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"

# Domain verification (Google, AWS, etc.)
example.com  TXT  "google-site-verification=abc123..."
example.com  TXT  "amazonses:abc123-def456-..."

# CAA (Certificate Authority Authorization)
example.com  CAA  0 issue "letsencrypt.org"
example.com  CAA  0 issue "amazon.com"
example.com  CAA  0 issuewild "letsencrypt.org"

# Check TXT records
dig TXT example.com +short
```

---

## 2.6 DevOps Record Configuration Examples

```bash
# Typical web application DNS setup:

# Zone apex → ALB (using ALIAS in Route 53)
example.com        ALIAS  my-alb-123.us-east-1.elb.amazonaws.com

# WWW redirect
www.example.com    CNAME  example.com

# API subdomain → separate ALB
api.example.com    CNAME  api-alb-456.us-east-1.elb.amazonaws.com

# Static assets → CloudFront
cdn.example.com    CNAME  d123abc.cloudfront.net

# Staging environment
staging.example.com  A     10.0.1.50

# Email (Google Workspace)
example.com        MX    1  aspmx.l.google.com
example.com        MX    5  alt1.aspmx.l.google.com
example.com        MX    5  alt2.aspmx.l.google.com

# Monitoring / status page
status.example.com  CNAME  mycompany.statuspage.io

# Verify domain ownership for various services
example.com        TXT   "v=spf1 include:_spf.google.com ~all"
example.com        TXT   "google-site-verification=abcdef..."
```

---

## 2.7 Query Records with dig

```bash
# A record
dig A example.com +short
# Output: 93.184.216.34

# AAAA record
dig AAAA example.com +short

# CNAME record
dig CNAME www.example.com +short
# Output: example.com

# MX records
dig MX example.com +short
# Output: 10 mail.example.com

# NS records
dig NS example.com +short
# Output: ns1.example.com  ns2.example.com

# TXT records
dig TXT example.com +short

# SOA record
dig SOA example.com

# All records
dig ANY example.com

# Specific server
dig @8.8.8.8 A example.com +short
```

---

## 2.8 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| CNAME at zone apex fails | CNAME not allowed at apex | Use ALIAS (Route 53) or A record |
| Email not delivered | Missing or wrong MX records | Check MX records with `dig MX` |
| Email marked as spam | Missing SPF/DKIM/DMARC | Add TXT records for email auth |
| SSL cert issuance fails | CAA record blocking CA | Add CAA record for your CA |
| CNAME chain too long | CNAME → CNAME → CNAME | Flatten to A or single CNAME |
| Record not resolving | TTL caching old value | Wait for TTL or flush DNS cache |

---

## Summary Table

| Record | Purpose | Zone Apex? | Key DevOps Use |
|--------|---------|-----------|----------------|
| A | IPv4 mapping | Yes | Point domain to server IP |
| AAAA | IPv6 mapping | Yes | IPv6 support |
| CNAME | Alias | No | Point to ALB/CloudFront DNS |
| ALIAS | AWS alias | Yes | Zone apex to AWS resources |
| MX | Email routing | Yes | Google Workspace, SES |
| TXT | Text/verification | Yes | SPF, DKIM, domain verify |
| NS | Delegation | Yes | Subdomain delegation |
| SRV | Service location | N/A | K8s, Consul discovery |
| CAA | CA authorization | Yes | Restrict SSL cert issuers |
| PTR | Reverse lookup | N/A | Email deliverability |

---

## Quick Revision Questions

1. **Why can't you use a CNAME record at the zone apex (e.g., `example.com`)?**
2. **What's the difference between a CNAME and an ALIAS record in Route 53?**
3. **Set up DNS records for: `example.com` → ALB, `www` → redirect, `api` → separate ALB, email via Google Workspace.**
4. **Explain the purpose of SPF, DKIM, and DMARC TXT records.**
5. **When would you use an SRV record? Give a real-world example.**
6. **How do CAA records improve your SSL/TLS security?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← DNS Hierarchy](01-dns-hierarchy.md) | [README](../README.md) | [DNS Resolution Process →](03-dns-resolution-process.md) |
