# Unit 12: Web Testing Tools - Topic 2: OWASP ZAP

## Overview

OWASP ZAP (Zed Attack Proxy) is a **free, open-source** web application security scanner maintained by the OWASP Foundation. It is the world's most popular free security tool and serves as an excellent alternative to Burp Suite Professional. ZAP provides automated scanning, manual testing tools, API support, and CI/CD integration. It is particularly valuable for **DevSecOps pipelines**, security training, and organizations that cannot afford commercial tools.

---

## 1. ZAP Architecture and Setup

### ZAP Components

```
┌─────────────────────────────────────────────────────────────┐
│                       OWASP ZAP                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐              │
│  │  Proxy    │  │  Spider   │  │  Active   │              │
│  │  (MitM)   │  │  (Crawl)  │  │  Scanner  │              │
│  └───────────┘  └───────────┘  └───────────┘              │
│                                                             │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐              │
│  │  Fuzzer   │  │  Forced   │  │  Ajax     │              │
│  │           │  │  Browse   │  │  Spider   │              │
│  └───────────┘  └───────────┘  └───────────┘              │
│                                                             │
│  ┌───────────┐  ┌───────────┐  ┌───────────────────────┐  │
│  │  Manual   │  │  Passive  │  │  Marketplace          │  │
│  │  Request  │  │  Scanner  │  │  (Add-ons/Extensions) │  │
│  │  Editor   │  │           │  │                       │  │
│  └───────────┘  └───────────┘  └───────────────────────┘  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  ZAP API (REST) — Full automation support             │  │
│  │  + CLI mode + Docker + GitHub Actions                 │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Installation

```bash
# Download from official site
# https://www.zaproxy.org/download/

# Docker (recommended for CI/CD)
docker pull zaproxy/zap-stable

# Run ZAP in Docker
docker run -t zaproxy/zap-stable zap-baseline.py \
  -t http://target.com

# Run with GUI (Docker)
docker run -u zap -p 8080:8080 -p 8090:8090 \
  -i zaproxy/zap-stable zap-webswing.sh

# Command line mode
zap.sh -daemon -port 8080 \
  -config api.key=your-api-key
```

### Proxy Configuration

```
Default proxy: localhost:8080

Browser Setup (same as Burp):
1. Set browser proxy to localhost:8080
2. Navigate to http://zap/ to get CA certificate
3. Import CA certificate into browser
4. Browse target application

HUD (Heads-Up Display):
- ZAP's unique feature
- Overlay in browser showing security info
- Enable: Toolbar → HUD button
- Shows alerts, attack tools directly in browser
```

---

## 2. Automated Scanning

### Spider (Traditional Crawler)

```
Spider Configuration:
┌──────────────────────────────────────────┐
│ Spider Options:                          │
│ - Max Depth: 10                         │
│ - Max Children: 0 (unlimited)            │
│ - Thread Count: 5                        │
│ - Max Duration: 0 (unlimited)            │
│ - Parse Comments: Yes                    │
│ - Parse Robots.txt: Yes                  │
│ - Handle OData Parameters: Yes           │
│ - POST Form: Yes                         │
└──────────────────────────────────────────┘

# Start Spider:
1. Sites tab → Right-click target
2. Attack → Spider
3. Configure scope and options
4. Start

# Spider results appear in Sites tree
# Shows all discovered URLs and parameters
```

### Ajax Spider

```
# For JavaScript-heavy applications (SPA, React, Angular)
# Uses a real browser (Firefox/Chrome) to crawl

Configuration:
- Browser: Firefox / Chrome / HtmlUnit
- Max Crawl Depth: 10
- Max Crawl States: 0 (unlimited)
- Max Duration: 60 minutes
- Click default elements: Yes
- Click elements once: Yes

# Start Ajax Spider:
1. Right-click target → Attack → Ajax Spider
2. Select browser type
3. Start

# Uses headless browser to:
# - Click buttons and links
# - Submit forms
# - Navigate JavaScript routes
# - Discover AJAX endpoints
```

### Active Scanner

```
Active Scan Policies:
┌──────────────────────────────────────────┐
│ Default Policy includes:                 │
│ ☑ SQL Injection                         │
│ ☑ Cross-Site Scripting (XSS)            │
│ ☑ Path Traversal                        │
│ ☑ Remote File Inclusion                 │
│ ☑ Command Injection                     │
│ ☑ CRLF Injection                        │
│ ☑ External Redirect                     │
│ ☑ Server-Side Template Injection        │
│ ☑ LDAP Injection                        │
│ ☑ XML External Entity                   │
│ ☑ And many more...                      │
└──────────────────────────────────────────┘

# Custom Scan Policy:
Analyze → Scan Policy Manager → Add
- Enable/disable specific test categories
- Set threshold: Low/Medium/High
- Set strength: Low/Medium/High/Insane

# Targeted Active Scan:
1. Select specific request in History
2. Right-click → Active Scan
3. Choose scan policy
4. Start scan on just that endpoint
```

---

## 3. Manual Testing Features

### Manual Request Editor

```
# Similar to Burp's Repeater
Tools → Manual Request Editor

Features:
- Send/resend requests manually
- Modify method, headers, body
- View formatted response
- Cookie management
- Follow redirects option

# Breakpoints (like Burp Intercept)
1. Set breakpoint on specific URL pattern
2. When request matches, ZAP pauses it
3. Modify request before forwarding
4. Set breakpoints on:
   - URL regex
   - Request contains string
   - Response contains string
   - Specific HTTP methods
```

### Fuzzer

```
# ZAP's fuzzer (equivalent to Burp Intruder)
1. Select request in History
2. Right-click → Attack → Fuzz
3. Highlight parameter value
4. Click "Add" → Choose payload:

Payload Options:
┌────────────────────────────────────────────┐
│ - File: Load wordlist                     │
│ - File Fuzzers: Built-in fuzzing lists    │
│   - FuzzDB payloads                       │
│   - jbrofuzz payloads                     │
│   - dirbuster wordlists                  │
│ - Numberzz: Sequential numbers            │
│ - Regex: Pattern-based generation         │
│ - Strings: Custom string list             │
│ - Script: Custom payload generator        │
└────────────────────────────────────────────┘

# Example: SQL Injection fuzzing
1. Select parameter value in request
2. Add payload: File Fuzzers → FuzzDB → attack → sql-injection
3. Configure:
   - Threads: 10
   - Delay: 0ms
   - Follow redirects: Yes
4. Start Fuzzer
5. Analyze results by response code, size, time

# Add processors:
- URL Encode
- Base64 Encode
- MD5/SHA Hash
- Prefix/Suffix
- Script-based processing
```

---

## 4. ZAP Automation Framework

### CI/CD Integration

```bash
# ZAP Baseline Scan (passive only, fast)
docker run -t zaproxy/zap-stable zap-baseline.py \
  -t http://target.com \
  -r report.html \
  -J report.json

# ZAP Full Scan (spider + active scan)
docker run -t zaproxy/zap-stable zap-full-scan.py \
  -t http://target.com \
  -r report.html \
  -J report.json

# ZAP API Scan (OpenAPI/Swagger)
docker run -t zaproxy/zap-stable zap-api-scan.py \
  -t http://target.com/swagger.json \
  -f openapi \
  -r report.html

# ZAP GraphQL Scan
docker run -t zaproxy/zap-stable zap-api-scan.py \
  -t http://target.com/graphql \
  -f graphql \
  -r report.html
```

### Automation Framework (YAML)

```yaml
# ZAP Automation Framework plan (af-plan.yaml)
---
env:
  contexts:
    - name: "target"
      urls:
        - "http://target.com"
      includePaths:
        - "http://target.com/.*"
      excludePaths:
        - "http://target.com/logout.*"
      authentication:
        method: "form"
        parameters:
          loginUrl: "http://target.com/login"
          loginRequestData: "username={%username%}&password={%password%}"
        verification:
          method: "response"
          loggedInRegex: "\\QDashboard\\E"
      users:
        - name: "test-user"
          credentials:
            username: "testuser"
            password: "testpass"

jobs:
  - type: spider
    parameters:
      context: "target"
      user: "test-user"
      maxDuration: 5
  - type: spiderAjax
    parameters:
      context: "target"
      user: "test-user"
      maxDuration: 5
  - type: passiveScan-wait
    parameters:
      maxDuration: 10
  - type: activeScan
    parameters:
      context: "target"
      user: "test-user"
      maxRuleDuration: 60
  - type: report
    parameters:
      template: "traditional-html"
      reportDir: "/zap/reports/"
      reportFile: "zap-report"
```

### GitHub Actions Integration

```yaml
# .github/workflows/zap-scan.yml
name: ZAP Security Scan
on: [push, pull_request]

jobs:
  zap_scan:
    runs-on: ubuntu-latest
    steps:
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.9.0
        with:
          target: 'http://target.com'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
      
      - name: Upload Report
        uses: actions/upload-artifact@v3
        with:
          name: zap-report
          path: report_html.html
```

---

## 5. ZAP API

### REST API Usage

```bash
# ZAP API is accessible at http://localhost:8080/JSON/
# Requires API key

# Get alerts
curl "http://localhost:8080/JSON/core/view/alerts/?apikey=KEY&baseurl=http://target.com"

# Start spider
curl "http://localhost:8080/JSON/spider/action/scan/?apikey=KEY&url=http://target.com"

# Start active scan
curl "http://localhost:8080/JSON/ascan/action/scan/?apikey=KEY&url=http://target.com"

# Get scan progress
curl "http://localhost:8080/JSON/ascan/view/status/?apikey=KEY&scanId=0"

# Generate report
curl "http://localhost:8080/OTHER/core/other/htmlreport/?apikey=KEY" > report.html
```

### Python API Client

```python
from zapv2 import ZAPv2

# Connect to ZAP
zap = ZAPv2(apikey='your-api-key', proxies={
    'http': 'http://127.0.0.1:8080',
    'https': 'http://127.0.0.1:8080'
})

target = 'http://target.com'

# Spider the target
print('Spidering target...')
scan_id = zap.spider.scan(target)
while int(zap.spider.status(scan_id)) < 100:
    print(f'Spider progress: {zap.spider.status(scan_id)}%')
    time.sleep(2)

# Active scan
print('Active scanning...')
scan_id = zap.ascan.scan(target)
while int(zap.ascan.status(scan_id)) < 100:
    print(f'Scan progress: {zap.ascan.status(scan_id)}%')
    time.sleep(5)

# Get results
alerts = zap.core.alerts(baseurl=target)
for alert in alerts:
    print(f"[{alert['risk']}] {alert['alert']}")
    print(f"  URL: {alert['url']}")
    print(f"  Description: {alert['description'][:100]}")
```

---

## 6. ZAP vs Burp Suite Comparison

| Feature | OWASP ZAP | Burp Suite Pro |
|---------|-----------|----------------|
| **Price** | Free & Open Source | $449/year |
| **Scanner Quality** | Good | Excellent |
| **Manual Testing** | Good | Excellent |
| **Extensions** | Marketplace | BApp Store |
| **CI/CD Integration** | Excellent (Docker, API) | Good (Enterprise) |
| **API Testing** | Good (OpenAPI, GraphQL) | Good |
| **HUD** | ✅ Unique feature | ❌ |
| **Collaborator** | ❌ (use alternatives) | ✅ Built-in |
| **Intruder Speed** | Unlimited | Throttled (Community) |
| **Learning Curve** | Moderate | Moderate-High |
| **Community** | Large, OWASP-backed | Large, commercial |
| **Best For** | DevSecOps, CI/CD, learning | Professional pentesting |

---

## 7. Add-ons and Extensions

### Essential ZAP Add-ons

| Add-on | Purpose |
|--------|---------|
| **Active Scan Rules** | Core vulnerability checks |
| **Passive Scan Rules** | Background analysis |
| **FuzzDB** | Comprehensive fuzzing payloads |
| **Ajax Spider** | JavaScript-heavy app crawling |
| **OpenAPI Support** | Swagger/OpenAPI import |
| **GraphQL Support** | GraphQL schema import & testing |
| **Selenium** | Browser automation |
| **Scripts** | Custom scripting (JS, Python, Zest) |
| **Report Generation** | HTML, XML, JSON reports |
| **Wappalyzer** | Technology fingerprinting |

### Installing Add-ons

```
GUI: Manage Add-ons → Marketplace → Search → Install

CLI:
zap.sh -cmd -addoninstall ascanrules
zap.sh -cmd -addoninstall pscanrules
zap.sh -cmd -addoninstall openapi
zap.sh -cmd -addoninstall graphql
```

---

## Summary Table

| Feature | Description |
|---------|-------------|
| **Free & Open Source** | No cost, community-driven, OWASP project |
| **Spider + Ajax Spider** | Traditional and JavaScript-aware crawling |
| **Active Scanner** | Comprehensive automated vulnerability testing |
| **Fuzzer** | Multi-payload fuzzing with processing |
| **CI/CD Ready** | Docker, GitHub Actions, REST API |
| **Automation Framework** | YAML-based scan orchestration |
| **HUD** | Heads-Up Display for in-browser testing |
| **API Support** | OpenAPI, GraphQL, SOAP import |

---

## Revision Questions

1. What are the key differences between ZAP's traditional spider and the Ajax spider? When would you use each?
2. How would you integrate ZAP into a CI/CD pipeline using Docker and GitHub Actions?
3. Describe ZAP's Automation Framework. How do you create an authenticated scan plan in YAML?
4. Compare OWASP ZAP and Burp Suite Professional — when would you choose each tool?
5. How does ZAP's HUD (Heads-Up Display) work and what advantage does it provide?
6. Write a Python script using ZAP's API to spider a target, perform an active scan, and export results.

---

*Previous: [01-burp-suite-fundamentals.md](01-burp-suite-fundamentals.md) | Next: [03-browser-developer-tools.md](03-browser-developer-tools.md)*

---

*[Back to README](../README.md)*
