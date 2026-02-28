# Chapter 4.2: DAST Tools

## Overview

This chapter covers the leading DAST tools — OWASP ZAP, Burp Suite, and Nuclei — with practical setup, configuration, and usage examples for each.

---

## Tool Comparison

```
DAST TOOL LANDSCAPE:

  ┌──────────────┬──────────────┬──────────────┬──────────────┐
  │ Feature       │ OWASP ZAP   │ Burp Suite   │ Nuclei       │
  ├──────────────┼──────────────┼──────────────┼──────────────┤
  │ Cost          │ Free (OSS)  │ $449+/yr     │ Free (OSS)   │
  │ Best For      │ CI/CD auto  │ Manual + auto│ Template     │
  │               │ scanning     │ testing       │ scanning     │
  │ API Testing  │ Good         │ Excellent    │ Excellent    │
  │ CI/CD        │ Excellent    │ Good         │ Excellent    │
  │ Ease of Use  │ Medium       │ Easy (GUI)   │ Easy (CLI)   │
  │ Extensibility│ Scripts/APIs │ Extensions   │ YAML tmpl    │
  │ Auth Support │ Scripts      │ Macros       │ Templates    │
  │ Speed         │ Medium       │ Medium       │ Fast         │
  └──────────────┴──────────────┴──────────────┴──────────────┘
```

---

## OWASP ZAP (Zed Attack Proxy)

### Architecture

```
ZAP ARCHITECTURE:

  ┌────────────────────────────────────────────────────────┐
  │                    OWASP ZAP                            │
  │                                                          │
  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  │
  │  │   Spider     │  │ Active       │  │ Passive      │  │
  │  │  (Crawler)   │  │ Scanner      │  │ Scanner      │  │
  │  │              │  │              │  │              │  │
  │  │ Traditional  │  │ SQL Injection│  │ Cookie flags │  │
  │  │ AJAX Spider  │  │ XSS          │  │ Headers      │  │
  │  │              │  │ Path Trav.   │  │ Info leaks   │  │
  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
  │         │                 │                  │           │
  │         ▼                 ▼                  ▼           │
  │  ┌──────────────────────────────────────────────────┐  │
  │  │                  RESULTS ENGINE                   │  │
  │  │  Alerts → Risk Assessment → Reports (HTML/JSON)  │  │
  │  └──────────────────────────────────────────────────┘  │
  └────────────────────────────────────────────────────────┘
```

### Docker Setup & Quick Start

```bash
# Pull ZAP Docker image
docker pull ghcr.io/zaproxy/zaproxy:stable

# Baseline scan (passive only — safe, fast)
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
    -t https://target-app.staging.com \
    -r report.html

# Full scan (active — staging only!)
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py \
    -t https://target-app.staging.com \
    -r full-report.html \
    -m 60  # max scan duration in minutes

# API scan (for REST/GraphQL APIs)
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py \
    -t https://target-app.staging.com/openapi.json \
    -f openapi \
    -r api-report.html
```

### ZAP Automation Framework

```yaml
# zap-automation.yaml
env:
  contexts:
    - name: "My App"
      urls:
        - "https://staging.myapp.com"
      includePaths:
        - "https://staging.myapp.com/.*"
      excludePaths:
        - "https://staging.myapp.com/logout.*"
      authentication:
        method: "form"
        parameters:
          loginPageUrl: "https://staging.myapp.com/login"
          loginRequestUrl: "https://staging.myapp.com/api/auth/login"
          loginRequestBody: "username={%username%}&password={%password%}"
        verification:
          method: "response"
          loggedInRegex: "\\Qdashboard\\E"
      users:
        - name: "test-user"
          credentials:
            username: "testuser@email.com"
            password: "TestPass123!"

jobs:
  - type: spider
    parameters:
      maxDuration: 5      # minutes
      maxDepth: 5
      maxChildren: 10
  
  - type: spiderAjax
    parameters:
      maxDuration: 5
      maxCrawlDepth: 3
  
  - type: passiveScan-wait
    parameters:
      maxDuration: 10
  
  - type: activeScan
    parameters:
      maxRuleDurationInMins: 5
      maxScanDurationInMins: 30
      policy: "API-Scan"
  
  - type: report
    parameters:
      template: "traditional-html"
      reportDir: "/zap/reports/"
      reportFile: "scan-report"
    risks:
      - high
      - medium
```

```bash
# Run with automation framework
docker run -v $(pwd):/zap/wrk/:rw \
    ghcr.io/zaproxy/zaproxy:stable \
    zap.sh -cmd -autorun /zap/wrk/zap-automation.yaml
```

---

## Burp Suite

### Architecture

```
BURP SUITE ARCHITECTURE:

  ┌──────────────────────────────────────────────────────────┐
  │                     BURP SUITE                            │
  │                                                            │
  │  Browser ──→ ┌──────────┐ ──→ Target Application         │
  │              │  PROXY    │                                  │
  │  Intercept ←─┤  (MitM)  │←── Responses                    │
  │              └────┬─────┘                                  │
  │                   │                                         │
  │    ┌──────────────┼──────────────┐                         │
  │    ▼              ▼              ▼                          │
  │ ┌────────┐  ┌──────────┐  ┌──────────┐                   │
  │ │Scanner │  │Repeater  │  │Intruder  │                    │
  │ │(Auto)  │  │(Manual)  │  │(Fuzzer)  │                    │
  │ └────────┘  └──────────┘  └──────────┘                    │
  │                                                            │
  │  Enterprise CI/CD: Burp Suite Enterprise Edition           │
  │  REST API for pipeline integration                         │
  └──────────────────────────────────────────────────────────┘
```

### Burp Suite CI/CD Integration (Enterprise)

```bash
# Start scan via Burp Enterprise API
curl -X POST "https://burp-enterprise.company.com/api/scans" \
    -H "Authorization: Bearer $BURP_API_TOKEN" \
    -H "Content-Type: application/json" \
    -d '{
        "scan_configuration_ids": ["full-audit"],
        "urls": ["https://staging.myapp.com"],
        "scope": {
            "include": ["https://staging.myapp.com/.*"],
            "exclude": ["https://staging.myapp.com/logout"]
        }
    }'

# Poll for results
curl "https://burp-enterprise.company.com/api/scans/{scan_id}/issues" \
    -H "Authorization: Bearer $BURP_API_TOKEN"
```

---

## Nuclei

### Overview & Setup

```
NUCLEI ARCHITECTURE:

  ┌────────────────────────────────────────────────────────┐
  │                      NUCLEI                             │
  │                                                          │
  │  ┌──────────────┐                                       │
  │  │ YAML Template │ → Defines: Request + Matcher         │
  │  │ Library       │   (6,000+ community templates)       │
  │  └──────┬───────┘                                       │
  │         │                                                │
  │         ▼                                                │
  │  ┌──────────────┐    ┌──────────────┐                   │
  │  │ HTTP Engine  │───→│ Target App   │                   │
  │  │ (concurrent) │←───│              │                   │
  │  └──────┬───────┘    └──────────────┘                   │
  │         │                                                │
  │         ▼                                                │
  │  ┌──────────────┐                                       │
  │  │ Matcher      │ Response matched template?             │
  │  │ Engine       │ → YES = Finding, NO = Clean            │
  │  └──────────────┘                                       │
  └────────────────────────────────────────────────────────┘
```

```bash
# Install Nuclei
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
# Or via package manager
brew install nuclei        # macOS
apt install nuclei         # Debian/Ubuntu
choco install nuclei       # Windows

# Update templates
nuclei -ut

# Basic scan
nuclei -u https://staging.myapp.com -as  # auto-select templates

# Scan with specific templates
nuclei -u https://staging.myapp.com \
    -t cves/ \
    -t vulnerabilities/ \
    -t misconfiguration/ \
    -severity critical,high \
    -o results.txt

# Scan multiple targets
nuclei -l targets.txt -t http/ -severity critical,high

# Output in JSON for CI/CD parsing
nuclei -u https://staging.myapp.com \
    -t vulnerabilities/ \
    -json -o results.json
```

### Custom Nuclei Template

```yaml
# custom-templates/exposed-admin.yaml
id: exposed-admin-panel
info:
  name: Exposed Admin Panel
  author: security-team
  severity: high
  description: Admin panel accessible without authentication
  tags: admin,exposure,misconfig

http:
  - method: GET
    path:
      - "{{BaseURL}}/admin"
      - "{{BaseURL}}/admin/"
      - "{{BaseURL}}/administrator"
      - "{{BaseURL}}/wp-admin"
      - "{{BaseURL}}/dashboard"

    matchers-condition: and
    matchers:
      - type: status
        status:
          - 200
      - type: word
        words:
          - "admin"
          - "dashboard"
          - "login"
        condition: or
```

```yaml
# custom-templates/missing-security-headers.yaml
id: missing-security-headers
info:
  name: Missing Security Headers
  author: security-team
  severity: medium
  tags: headers,misconfig

http:
  - method: GET
    path:
      - "{{BaseURL}}"

    matchers-condition: or
    matchers:
      - type: regex
        part: header
        name: missing-csp
        regex:
          - "^(?!.*content-security-policy).*$"
        condition: and

      - type: regex
        part: header
        name: missing-hsts
        regex:
          - "^(?!.*strict-transport-security).*$"
        condition: and
```

---

## Tool Selection Decision

```
WHICH DAST TOOL TO USE?

                    Start Here
                        │
                        ▼
              ┌───────────────────┐
              │ Need CI/CD auto   │
              │ scanning?         │
              └────────┬──────────┘
                  Yes  │     No
          ┌────────────┤     │
          ▼            │     ▼
  ┌──────────────┐     │  ┌──────────────┐
  │ Budget?       │     │  │ Manual pen   │
  └──────┬───────┘     │  │ testing?      │
   Free  │  Paid       │  └──────┬───────┘
         │     │       │    Yes  │
    ┌────┘     │       │         ▼
    ▼          ▼       │   BURP SUITE PRO
  ZAP or    BURP      │   (Best manual
  Nuclei    ENT.      │    testing GUI)
                       │
  RECOMMENDED:         │
  Use ZAP + Nuclei     │
  together (free)      │
  ZAP = full scanning  │
  Nuclei = fast checks │
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| ZAP can't crawl SPA | JavaScript rendering needed | Use AJAX Spider instead of traditional spider |
| Authentication keeps failing | Session expires during scan | Configure longer session timeouts on staging; use re-authentication |
| Scan produces 0 results | Scanner blocked by WAF/CDN | Point scanner at direct staging URL, bypass CDN |
| Nuclei templates outdated | Template library not updated | Run `nuclei -ut` regularly; pin to specific release in CI |
| Scan takes 8+ hours | Too many endpoints, all checks | Scope to critical paths only; use `maxDuration` limits |

---

## Summary Table

| Tool | Type | Best For | Cost |
|------|------|----------|------|
| **OWASP ZAP** | Full DAST scanner | CI/CD integration, automated scanning | Free (OSS) |
| **Burp Suite Pro** | Full DAST scanner | Manual testing, expert analysis | $449/yr |
| **Burp Enterprise** | Full DAST scanner | Enterprise CI/CD scanning | $$$ |
| **Nuclei** | Template scanner | Fast checks, CVEs, misconfigs | Free (OSS) |
| **ZAP + Nuclei** | Combined approach | Best free coverage | Free |

---

## Quick Revision Questions

1. **What are the three scan modes available in OWASP ZAP Docker?**
   > Baseline scan (passive only, safe for production), Full scan (active scanning, staging only), and API scan (specifically for REST/GraphQL APIs using OpenAPI/Swagger specs).

2. **How does Nuclei differ from traditional DAST scanners like ZAP?**
   > Nuclei uses YAML-based templates to define specific checks, making it extremely fast and focused. It excels at checking for known CVEs, misconfigurations, and exposures. Traditional scanners like ZAP do comprehensive crawling and fuzzing of all parameters.

3. **When would you choose Burp Suite over ZAP?**
   > For manual penetration testing (Burp's Repeater/Intruder GUI is superior), when expert-level analysis is needed, or at enterprise scale with Burp Enterprise for CI/CD integration. ZAP is preferred for free automated CI/CD scanning.

4. **What is the ZAP Automation Framework?**
   > A YAML-based configuration system for ZAP that defines scan jobs (spider, AJAX spider, passive scan, active scan, reporting) in a declarative file. It allows consistent, repeatable scans ideal for CI/CD integration.

5. **How do you write a custom Nuclei template?**
   > Define a YAML file with `id`, `info` (name, severity, tags), and `http` sections. The HTTP section specifies the request method, paths, and matchers that identify vulnerable responses (status codes, words, regex patterns).

---

[← Previous: 4.1 DAST Concepts](01-dast-concepts.md) | [Next: 4.3 Automated Scanning →](03-automated-scanning.md)

[Back to Table of Contents](../README.md)
