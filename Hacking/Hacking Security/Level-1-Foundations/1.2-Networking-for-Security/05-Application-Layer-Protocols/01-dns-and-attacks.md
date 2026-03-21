# DNS and DNS Attacks

## Unit 5 - Topic 1 | Application Layer Protocols

---

## Overview

**DNS (Domain Name System)** translates domain names to IP addresses. It's one of the most critical internet services — and one of the most targeted. DNS attacks include **cache poisoning**, **DNS tunneling**, **zone transfer abuse**, **DNS hijacking**, and **amplification DDoS**.

---

## 1. How DNS Works

```
┌──────────────────────────────────────────────────────────────────┐
│                  DNS RESOLUTION PROCESS                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  User types: www.example.com                                    │
│                                                                  │
│  1. Browser → Local DNS Cache (cached? done!)                   │
│  2. → Recursive Resolver (ISP or 8.8.8.8)                      │
│  3. → Root DNS Server (.) → "Ask .com servers"                  │
│  4. → TLD Server (.com) → "Ask example.com's NS"               │
│  5. → Authoritative NS (example.com) → "IP: 93.184.216.34"     │
│  6. Answer cached and returned to browser                       │
│                                                                  │
│  Client ─► Resolver ─► Root ─► TLD ─► Authoritative            │
│    │           │                              │                  │
│    │◄──────────│◄─────────────────────────────┘                  │
│    │     93.184.216.34                                           │
│    ▼                                                             │
│  Connect to 93.184.216.34                                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. DNS Record Types (Security Relevant)

| Record | Purpose | Security Relevance |
|--------|---------|-------------------|
| **A** | Domain → IPv4 address | Target identification |
| **AAAA** | Domain → IPv6 address | IPv6 attack surface |
| **MX** | Mail server | Email infrastructure recon |
| **NS** | Nameserver | DNS infrastructure attacks |
| **CNAME** | Alias to another domain | Subdomain takeover |
| **TXT** | Text records | SPF, DKIM, DMARC; data exfil |
| **SOA** | Start of Authority | Zone transfer info |
| **PTR** | IP → Domain (reverse) | Reverse DNS recon |
| **SRV** | Service location | Service discovery |
| **AXFR** | Zone transfer | Full domain enumeration! |

---

## 3. DNS Attacks

### DNS Cache Poisoning

```
Attacker injects FAKE DNS records into resolver's cache:

Normal: example.com → 93.184.216.34 (real server)
Poisoned: example.com → 10.0.0.5 (attacker's server!)

All users of that resolver get sent to attacker's site.
→ Phishing, credential theft, malware delivery

DEFENSE: DNSSEC, randomized source ports, transaction IDs
```

### DNS Zone Transfer

```bash
# If misconfigured, reveals ALL DNS records for a domain
dig axfr @ns1.example.com example.com
host -t axfr example.com ns1.example.com
nslookup → set type=any → ls -d example.com

# Reveals: all subdomains, internal hostnames, IP addresses
# MASSIVE recon value — should NEVER be publicly accessible

# DEFENSE: Restrict zone transfers to authorized secondary DNS only
# BIND: allow-transfer { 10.0.0.2; };
```

### DNS Tunneling

```
Hide data inside DNS queries to bypass firewalls:

Normal query:  www.google.com
Tunnel query:  aGVsbG8gd29ybGQ=.tunnel.attacker.com
               ↑ Base64 encoded data in subdomain

Tools: dnscat2, iodine, dns2tcp

DEFENSE: Monitor DNS query length/frequency, restrict to
         trusted resolvers, DNS monitoring/analytics
```

### DNS Hijacking

```
Types:
• Local: Modify victim's /etc/hosts or DNS settings
• Router: Change router's DNS configuration
• Registrar: Compromise domain registrar account
• Rogue DNS: Set up fake DNS server (DHCP attack)

DEFENSE: DNSSEC, MFA on registrar accounts, DNS monitoring
```

---

## 4. DNS Reconnaissance Tools

```bash
# Basic DNS queries
nslookup example.com
dig example.com ANY
host example.com

# Subdomain enumeration
fierce --domain example.com
sublist3r -d example.com
amass enum -d example.com
dnsenum example.com

# Reverse DNS
dig -x 93.184.216.34
nmap -sL 192.168.1.0/24        # Reverse DNS for subnet
```

---

## 5. DNS Security Controls

| Control | Description |
|---------|-------------|
| **DNSSEC** | Cryptographic signatures on DNS records |
| **DoH (DNS over HTTPS)** | Encrypts DNS queries via HTTPS (port 443) |
| **DoT (DNS over TLS)** | Encrypts DNS queries via TLS (port 853) |
| **Split-horizon DNS** | Different answers for internal vs external queries |
| **DNS Firewall (RPZ)** | Block queries to known malicious domains |
| **Rate Limiting** | Prevent DNS amplification abuse |
| **Restrict Zone Transfers** | Only allow to authorized secondary DNS |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DNS** | Translates domains to IPs — critical infrastructure |
| **Cache Poisoning** | Inject fake records into resolver cache |
| **Zone Transfer** | AXFR reveals all domain records if misconfigured |
| **Tunneling** | Hides data in DNS queries to bypass firewalls |
| **DNSSEC** | Cryptographic authentication of DNS responses |
| **DoH/DoT** | Encrypts DNS queries for privacy |

---

## Quick Revision Questions

1. **Describe the DNS resolution process step by step.**
2. **What is DNS cache poisoning and what damage can it cause?**
3. **Why should zone transfers be restricted?**
4. **How does DNS tunneling work?**
5. **What is DNSSEC and what problem does it solve?**
6. **Name 3 DNS recon tools and what they discover.**

---

[Next: HTTP/HTTPS →](02-http-https.md)

---

*Unit 5 - Topic 1 of 6 | [Back to README](../README.md)*
