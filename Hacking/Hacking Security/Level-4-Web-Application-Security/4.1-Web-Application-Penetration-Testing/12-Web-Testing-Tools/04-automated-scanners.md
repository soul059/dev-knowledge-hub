# Unit 12: Web Testing Tools - Topic 4: Automated Scanners

## Overview

Automated web vulnerability scanners are tools that systematically test web applications for known security vulnerabilities. They combine crawling, fuzzing, and signature-based detection to identify issues like SQL injection, XSS, misconfigurations, and known CVEs. While no scanner can replace manual testing, automated scanners provide **broad coverage**, **consistent baseline testing**, and **CI/CD integration** that makes them essential components of any security testing program.

---

## 1. Scanner Categories

### Types of Automated Scanners

```
┌─────────────────────────────────────────────────────────────┐
│              Web Vulnerability Scanner Types                 │
├─────────────────┬───────────────────────────────────────────┤
│ DAST            │ Dynamic Application Security Testing      │
│ (Black-box)     │ Tests running application from outside    │
│                 │ e.g., Burp Scanner, ZAP, Nikto, Nuclei   │
├─────────────────┼───────────────────────────────────────────┤
│ SAST            │ Static Application Security Testing       │
│ (White-box)     │ Analyzes source code without running      │
│                 │ e.g., Semgrep, SonarQube, Checkmarx       │
├─────────────────┼───────────────────────────────────────────┤
│ IAST            │ Interactive Application Security Testing   │
│ (Gray-box)      │ Agent inside running app + external test  │
│                 │ e.g., Contrast Security, Seeker            │
├─────────────────┼───────────────────────────────────────────┤
│ SCA             │ Software Composition Analysis             │
│                 │ Checks dependencies for known CVEs        │
│                 │ e.g., Snyk, Dependabot, npm audit          │
└─────────────────┴───────────────────────────────────────────┘
```

### Scanner Comparison

| Tool | Type | Cost | Strengths | Weaknesses |
|------|------|------|-----------|------------|
| **Burp Scanner** | DAST | $$$ | Deep testing, low false positives | Cost, single-target |
| **OWASP ZAP** | DAST | Free | CI/CD, API support | Higher false positives |
| **Nikto** | DAST | Free | Fast, config checks | No deep vuln testing |
| **Nuclei** | DAST | Free | Template-based, fast | Requires template knowledge |
| **Nessus** | Vuln Scanner | $ | Broad coverage, CVE | Not web-focused |
| **Acunetix** | DAST | $$$ | Good JS handling | Cost |
| **Wapiti** | DAST | Free | Python-based, modular | Slower |
| **Arachni** | DAST | Free | Comprehensive | Discontinued |
| **Semgrep** | SAST | Free/$ | Pattern-based, fast | Requires source code |
| **SonarQube** | SAST | Free/$ | Multi-language | Complex setup |
| **Snyk** | SCA | Free/$ | Dependency scanning | Specific scope |

---

## 2. Nikto — Web Server Scanner

### Overview and Usage

```bash
# Nikto scans web servers for:
# - Misconfigurations
# - Default files and programs
# - Insecure files and programs
# - Outdated server software
# - Version-specific vulnerabilities

# Basic scan
nikto -h http://target.com

# Scan specific port
nikto -h target.com -p 8080

# Scan with SSL
nikto -h https://target.com -ssl

# Save output
nikto -h target.com -o report.html -Format html
nikto -h target.com -o report.xml -Format xml
nikto -h target.com -o report.csv -Format csv

# Specify tuning (test categories)
nikto -h target.com -Tuning 1234567890abcde

# Tuning options:
# 0 - File Upload
# 1 - Interesting File / Seen in logs
# 2 - Misconfiguration / Default File
# 3 - Information Disclosure
# 4 - Injection (XSS/Script/HTML)
# 5 - Remote File Retrieval - Inside Web Root
# 6 - Denial of Service
# 7 - Remote File Retrieval - Server Wide
# 8 - Command Execution / Remote Shell
# 9 - SQL Injection
# a - Authentication Bypass
# b - Software Identification
# c - Remote Source Inclusion
# d - WebService
# e - Administrative Console

# Use proxy (through Burp)
nikto -h target.com -useproxy http://127.0.0.1:8080

# Scan multiple hosts
nikto -h hosts.txt

# Evasion techniques
nikto -h target.com -evasion 1
# 1 - Random URI encoding
# 2 - Directory self-reference (/./)
# 3 - Premature URL ending
# 4 - Prepend long random string
# 5 - Fake parameter
# 6 - TAB as request spacer
# 7 - Change the case of the URL
# 8 - Use Windows directory separator (\)
# A - Use a carriage return (0x0d)
# B - Use binary value 0x0b as request spacer
```

### Nikto Output Analysis

```
Nikto Results Example:
+ Server: Apache/2.4.49 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present.
+ /: The X-Content-Type-Options header is not set.
+ /: Directory indexing found.
+ /admin/: Admin area found.
+ /phpmyadmin/: phpMyAdmin found.
+ /server-status: Server status page found.
+ /test.php: Test file found with sensitive information.
+ Apache/2.4.49 - Path Traversal (CVE-2021-41773)

Priority Analysis:
1. [CRITICAL] CVE-2021-41773 - Known exploit available
2. [HIGH] phpMyAdmin exposed - Database management tool
3. [HIGH] Admin area accessible
4. [MEDIUM] Directory indexing enabled
5. [LOW] Missing security headers
```

---

## 3. Nuclei — Template-Based Scanner

### Overview and Usage

```bash
# Nuclei is a fast, customizable vulnerability scanner
# Uses YAML templates for detection logic
# Community maintains 6000+ templates

# Install
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Update templates
nuclei -update-templates

# Basic scan (all templates)
nuclei -u http://target.com

# Scan with specific template category
nuclei -u http://target.com -t cves/
nuclei -u http://target.com -t misconfiguration/
nuclei -u http://target.com -t exposures/
nuclei -u http://target.com -t vulnerabilities/
nuclei -u http://target.com -t technologies/

# Scan with severity filter
nuclei -u http://target.com -s critical,high
nuclei -u http://target.com -s medium

# Scan multiple targets
nuclei -l targets.txt -s critical,high

# Scan with specific tags
nuclei -u http://target.com -tags sqli,xss,rce
nuclei -u http://target.com -tags cve,misconfig
nuclei -u http://target.com -tags oast  # Out-of-band tests

# Scan with authentication
nuclei -u http://target.com \
  -H "Authorization: Bearer TOKEN" \
  -H "Cookie: session=abc123"

# Output formats
nuclei -u http://target.com -o results.txt
nuclei -u http://target.com -je results.json  # JSON export
nuclei -u http://target.com -me results/       # Markdown export

# Rate limiting
nuclei -u http://target.com -rl 50   # 50 requests/second
nuclei -u http://target.com -c 10    # 10 concurrent templates

# Headless mode (for JavaScript-rendered pages)
nuclei -u http://target.com -headless

# Interactsh (OOB testing)
nuclei -u http://target.com -iserver https://interact.sh
```

### Writing Custom Nuclei Templates

```yaml
# Custom template: check-admin-panel.yaml
id: admin-panel-detection
info:
  name: Admin Panel Detection
  author: pentester
  severity: medium
  description: Detects exposed admin panels
  tags: admin,exposure

http:
  - method: GET
    path:
      - "{{BaseURL}}/admin"
      - "{{BaseURL}}/admin/"
      - "{{BaseURL}}/administrator"
      - "{{BaseURL}}/wp-admin"
      - "{{BaseURL}}/admin/login"
    
    matchers-condition: or
    matchers:
      - type: word
        words:
          - "admin"
          - "login"
          - "dashboard"
        condition: or
      - type: status
        status:
          - 200
          - 302

---
# Custom template: sensitive-file-exposure.yaml
id: env-file-exposure
info:
  name: Environment File Exposure
  author: pentester
  severity: high
  description: Checks for exposed .env files

http:
  - method: GET
    path:
      - "{{BaseURL}}/.env"
    
    matchers-condition: and
    matchers:
      - type: status
        status:
          - 200
      - type: word
        words:
          - "DB_PASSWORD"
          - "APP_KEY"
          - "SECRET_KEY"
          - "API_KEY"
        condition: or
    
    extractors:
      - type: regex
        regex:
          - "(?i)(DB_PASSWORD|APP_KEY|SECRET_KEY|API_KEY)=(.+)"
```

---

## 4. Wapiti — Python Web Scanner

```bash
# Install
pip install wapiti3

# Basic scan
wapiti -u http://target.com

# Scan specific modules
wapiti -u http://target.com -m sql,xss,exec,file

# Available modules:
# sql      - SQL injection
# xss      - Cross-Site Scripting
# exec     - Command execution
# file     - File handling (LFI/RFI)
# crlf     - CRLF injection
# ssrf     - Server-Side Request Forgery
# xxe      - XML External Entity
# redirect - Open redirects
# htaccess - .htaccess bypass
# csp      - Content Security Policy
# nikto    - Known vulnerabilities (uses nikto DB)

# Authenticated scan
wapiti -u http://target.com \
  --cookie "session=abc123; token=xyz"

# Scan with form login
wapiti -u http://target.com \
  --form-cred "username%password" \
  --form-url "http://target.com/login"

# Output
wapiti -u http://target.com -f html -o report/
wapiti -u http://target.com -f json -o report.json
wapiti -u http://target.com -f xml -o report.xml

# Scope control
wapiti -u http://target.com -s http://target.com  # Stay on domain
wapiti -u http://target.com --max-links-per-page 50
wapiti -u http://target.com --max-scan-time 3600   # 1 hour max
```

---

## 5. Specialized Scanners

### SSL/TLS Scanners

```bash
# testssl.sh - Comprehensive TLS testing
./testssl.sh https://target.com
./testssl.sh --severity HIGH https://target.com
./testssl.sh --jsonfile results.json https://target.com

# Checks:
# - Protocol support (SSLv3, TLS 1.0-1.3)
# - Cipher suite strength
# - Certificate validity
# - Known vulnerabilities (BEAST, POODLE, Heartbleed, etc.)
# - HSTS, HPKP headers
# - Certificate chain

# sslyze - Python TLS scanner
sslyze target.com
sslyze --regular target.com
sslyze --json_out results.json target.com
```

### CMS-Specific Scanners

```bash
# WPScan - WordPress scanner
wpscan --url http://target.com --api-token YOUR_TOKEN
wpscan --url http://target.com --enumerate u,p,t
# u = users, p = plugins, t = themes

# Joomscan - Joomla scanner
joomscan -u http://target.com

# Droopescan - Drupal/Silverstripe/WordPress
droopescan scan drupal -u http://target.com

# CMSmap - Multi-CMS scanner
cmsmap http://target.com
```

### Infrastructure Scanners

```bash
# Nmap with NSE scripts for web
nmap -sV --script=http-* target.com -p 80,443,8080

# Useful NSE scripts:
nmap --script http-enum target.com          # Directory enumeration
nmap --script http-vuln-* target.com        # Known vulnerabilities
nmap --script http-headers target.com       # Security headers
nmap --script http-methods target.com       # Allowed methods
nmap --script http-sql-injection target.com # Basic SQLi check
nmap --script ssl-enum-ciphers target.com   # TLS analysis
```

---

## 6. Scan Strategy and Methodology

### Layered Scanning Approach

```
┌──────────────────────────────────────────────────────────┐
│ Layer 1: Passive Reconnaissance                          │
│ - Technology fingerprinting (Wappalyzer, BuiltWith)     │
│ - SSL/TLS analysis (testssl.sh, sslyze)                 │
│ - Header analysis (SecurityHeaders.com)                  │
├──────────────────────────────────────────────────────────┤
│ Layer 2: Quick Active Scan                               │
│ - Nikto (server misconfigurations)                      │
│ - Nuclei with critical/high templates                   │
│ - CMS-specific scanner (if applicable)                  │
├──────────────────────────────────────────────────────────┤
│ Layer 3: Deep Automated Scan                             │
│ - ZAP or Burp Scanner (full scan with auth)             │
│ - Nuclei with all templates                             │
│ - Wapiti for additional coverage                        │
├──────────────────────────────────────────────────────────┤
│ Layer 4: Manual Testing                                  │
│ - Business logic (no scanner finds these)               │
│ - Complex auth/authz testing                            │
│ - Chained vulnerabilities                               │
│ - Race conditions                                       │
└──────────────────────────────────────────────────────────┘
```

### Scanner Limitations

| Limitation | Description |
|-----------|-------------|
| **False positives** | Scanners report issues that don't exist |
| **False negatives** | Scanners miss real vulnerabilities |
| **Business logic** | Cannot understand business rules |
| **Complex auth** | Struggle with MFA, CAPTCHA, custom auth |
| **Client-side** | Limited JavaScript analysis (improving) |
| **Context** | Don't understand data sensitivity |
| **Chained vulns** | Can't chain multiple findings |
| **Rate limiting** | May be blocked by WAF/rate limits |

---

## 7. CI/CD Integration

### Pipeline Integration Example

```yaml
# GitHub Actions with multiple scanners
name: Security Scan Pipeline
on: [push]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # SAST - Static analysis
      - name: Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: p/owasp-top-ten
      
      # SCA - Dependency check
      - name: Snyk
        uses: snyk/actions@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      
      # DAST - Dynamic testing
      - name: ZAP Baseline
        uses: zaproxy/action-baseline@v0.9.0
        with:
          target: ${{ env.APP_URL }}
      
      # Nuclei scan
      - name: Nuclei
        uses: projectdiscovery/nuclei-action@main
        with:
          target: ${{ env.APP_URL }}
          flags: "-s critical,high"
```

---

## Summary Table

| Scanner | Best For | Speed | Accuracy |
|---------|----------|-------|----------|
| **Nikto** | Server misconfigs, quick recon | Fast | Medium |
| **Nuclei** | Known CVEs, template-based testing | Very Fast | High |
| **ZAP** | Full DAST scanning, CI/CD | Medium | Medium-High |
| **Burp Scanner** | Deep vulnerability testing | Slow | Very High |
| **Wapiti** | Python-based comprehensive scan | Medium | Medium |
| **testssl.sh** | TLS configuration analysis | Fast | Very High |
| **WPScan** | WordPress-specific testing | Fast | High |
| **Semgrep** | Source code analysis (SAST) | Very Fast | High |

---

## Revision Questions

1. Compare DAST, SAST, IAST, and SCA scanning approaches. What are the strengths and weaknesses of each?
2. How would you use Nuclei templates to create a custom security check? Write a template example.
3. What are the key limitations of automated scanners? Why is manual testing still essential?
4. Describe a layered scanning strategy for a web application penetration test.
5. How would you integrate automated security scanning into a CI/CD pipeline?
6. Compare Nikto and Nuclei — when would you choose each tool and why?

---

*Previous: [03-browser-developer-tools.md](03-browser-developer-tools.md) | Next: [05-custom-scripts.md](05-custom-scripts.md)*

---

*[Back to README](../README.md)*
