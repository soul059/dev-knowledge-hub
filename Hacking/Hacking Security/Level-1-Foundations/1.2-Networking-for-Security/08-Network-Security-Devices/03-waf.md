# Web Application Firewalls (WAF)

## Unit 8 - Topic 3 | Network Security Devices

---

## Overview

A **Web Application Firewall (WAF)** protects web applications by filtering and monitoring HTTP/HTTPS traffic between a web application and the Internet. Unlike traditional firewalls that operate at layers 3-4, WAFs operate at **Layer 7** and understand web application logic, protecting against **SQLi**, **XSS**, **CSRF**, and other OWASP Top 10 attacks.

---

## 1. WAF vs Traditional Firewall

```
┌──────────────────────────────────────────────────────────────┐
│              WAF PLACEMENT                                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet → [Firewall] → [WAF] → Web Server                 │
│                                                              │
│  Firewall:                                                   │
│  ├─ Blocks by IP, port, protocol                            │
│  └─ "Allow TCP 443 from any"                                │
│                                                              │
│  WAF:                                                        │
│  ├─ Inspects HTTP request content                           │
│  ├─ Blocks: GET /page?id=1' OR 1=1--                        │
│  ├─ Blocks: <script>alert('XSS')</script>                   │
│  └─ Blocks: /../../../etc/passwd                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

| Feature | Network Firewall | WAF |
|---------|:---:|:---:|
| **Layer** | 3-4 | 7 |
| **Inspects** | IP, Port, Protocol | HTTP content, headers, cookies |
| **Blocks SQLi** | ❌ | ✅ |
| **Blocks XSS** | ❌ | ✅ |
| **Blocks DDoS** | Volumetric only | Application-layer DDoS |
| **SSL/TLS** | Pass-through | Terminates and inspects |

---

## 2. WAF Deployment Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **Inline (Reverse Proxy)** | All traffic passes through WAF | Active blocking, most common |
| **Out-of-Band (TAP/SPAN)** | WAF monitors copy of traffic | Detection only, no blocking |
| **Cloud-based** | WAF as a service (DNS redirect) | Quick deployment, scalable |
| **Embedded** | WAF module in web server | ModSecurity in Apache/Nginx |

---

## 3. What WAFs Protect Against

| Attack | How WAF Detects |
|--------|----------------|
| **SQL Injection** | Detects SQL keywords in parameters (`' OR`, `UNION SELECT`) |
| **Cross-Site Scripting (XSS)** | Detects `<script>` tags, event handlers in input |
| **Path Traversal** | Detects `../` sequences in URLs |
| **Command Injection** | Detects shell commands (`;`, `|`, `` ` ``) in input |
| **File Inclusion** | Detects include paths in parameters |
| **CSRF** | Validates tokens and referrer headers |
| **Rate Limiting** | Throttles excessive requests |
| **Bot Detection** | Fingerprints automated tools |

---

## 4. ModSecurity — Open Source WAF

```bash
# ModSecurity with OWASP Core Rule Set (CRS)
# Installed as Apache/Nginx module

# Example: Block SQL injection
SecRule ARGS "@rx (?i:(?:union\s+select|or\s+1\s*=\s*1))" \
  "id:1001,phase:2,deny,status:403,msg:'SQL Injection Detected'"

# Example: Block XSS
SecRule ARGS "@rx <script>" \
  "id:1002,phase:2,deny,status:403,msg:'XSS Attempt Detected'"

# OWASP CRS provides 100s of pre-built rules
```

---

## 5. WAF Solutions

| Solution | Type | Notes |
|----------|------|-------|
| **ModSecurity** | Open source | Apache/Nginx module with OWASP CRS |
| **AWS WAF** | Cloud | Integrated with CloudFront, ALB |
| **Cloudflare WAF** | Cloud | Global CDN + WAF |
| **Akamai** | Cloud | Enterprise CDN + WAF |
| **Imperva** | Commercial | On-prem and cloud |
| **F5 BIG-IP ASM** | Commercial | Advanced WAF features |

---

## 6. WAF Bypass Techniques

| Technique | Example | Countermeasure |
|-----------|---------|----------------|
| **Case variation** | `SeLeCt` instead of `SELECT` | Case-insensitive rules |
| **Encoding** | `%27` for `'` (URL encoding) | Decode before inspection |
| **Double encoding** | `%2527` for `'` | Recursive decoding |
| **Comments** | `SEL/**/ECT` | Remove comments before check |
| **Unicode** | Alternative character representations | Unicode normalization |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **WAF** | Layer 7 firewall for web applications |
| **Purpose** | Protect against SQLi, XSS, OWASP Top 10 |
| **Deployment** | Inline, out-of-band, cloud-based, embedded |
| **ModSecurity** | Leading open-source WAF engine |
| **OWASP CRS** | Community-maintained rule set for ModSecurity |
| **Bypass** | Encoding, case tricks, comments — defense in depth needed |

---

## Quick Revision Questions

1. **How does a WAF differ from a traditional firewall?**
2. **What types of attacks can a WAF detect?**
3. **What are the main WAF deployment modes?**
4. **What is ModSecurity and OWASP CRS?**
5. **Name 3 WAF bypass techniques.**

---

[← Previous: IDS/IPS](02-ids-ips.md) | [Next: Load Balancers →](04-load-balancers.md)

---

*Unit 8 - Topic 3 of 6 | [Back to README](../README.md)*
