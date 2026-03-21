# Unit 12: Web Testing Tools - Topic 1: Burp Suite Fundamentals

## Overview

Burp Suite is the industry-standard web application security testing platform developed by PortSwigger. It serves as an intercepting proxy, scanner, and comprehensive toolkit for web penetration testing. Burp Suite Professional is used by the vast majority of professional web application penetration testers and bug bounty hunters. Mastering Burp Suite is **essential** for any web security professional, as it integrates proxy interception, active scanning, manual testing tools, and extensibility through the BApp Store.

---

## 1. Burp Suite Architecture

### Component Overview

```
┌──────────────────────────────────────────────────────────────┐
│                    BURP SUITE PLATFORM                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐ │
│  │  Proxy   │  │ Scanner  │  │ Intruder │  │  Repeater  │ │
│  │ Intercept│  │ Active/  │  │ Automated│  │  Manual    │ │
│  │ & Modify │  │ Passive  │  │ Attacks  │  │  Request   │ │
│  │ Traffic  │  │ Scanning │  │ Fuzzing  │  │  Editing   │ │
│  └──────────┘  └──────────┘  └──────────┘  └────────────┘ │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐ │
│  │ Decoder  │  │Comparer  │  │Sequencer │  │  Logger    │ │
│  │ Encode/  │  │ Diff     │  │ Token    │  │  HTTP      │ │
│  │ Decode   │  │ Responses│  │ Analysis │  │  History   │ │
│  └──────────┘  └──────────┘  └──────────┘  └────────────┘ │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────────┐  │
│  │Collabor- │  │Organizer │  │  Extensions (BApp Store) │  │
│  │  ator    │  │ Notes &  │  │  - Autorize             │  │
│  │ OOB      │  │ Findings │  │  - InQL                 │  │
│  │ Testing  │  │          │  │  - Turbo Intruder       │  │
│  └──────────┘  └──────────┘  │  - Upload Scanner       │  │
│                               │  - Param Miner          │  │
│                               │  - JWT Editor           │  │
│                               └──────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### Proxy Setup

```
Browser → Burp Proxy (127.0.0.1:8080) → Target Server

Setup Steps:
1. Burp Suite → Proxy → Options → Proxy Listeners
   - Running on: 127.0.0.1:8080

2. Browser (Firefox recommended):
   - Settings → Network → Manual Proxy
   - HTTP Proxy: 127.0.0.1  Port: 8080
   - Enable "Use this proxy for all protocols"
   
   OR use FoxyProxy extension:
   - Add new proxy: 127.0.0.1:8080
   - Pattern: *.target.com

3. Install Burp CA certificate:
   - Browse to: http://burpsuite
   - Download CA certificate
   - Import into browser's certificate store
   - Firefox: Settings → Privacy → Certificates → Import
   
4. For mobile testing:
   - Set phone's WiFi proxy to Burp IP:8080
   - Install CA cert on device
```

---

## 2. Proxy Tab — Intercepting Traffic

### Intercept Controls

```
Intercept Workflow:
┌─────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Browser │ ──► │  Burp    │ ──► │  Target  │ ──► │ Response │
│ Request │     │  Proxy   │     │  Server  │     │ Back to  │
│         │     │ (modify) │     │          │     │ Browser  │
└─────────┘     └──────────┘     └──────────┘     └──────────┘
                     │
              ┌──────┼──────┐
              │      │      │
           Forward  Drop  Action
            (→)     (✗)   (Send to...)
```

### Key Proxy Features

```
# Intercept Rules (Proxy → Options → Intercept Client Requests)
# Only intercept requests matching rules:
- URL: target.com
- File extension: NOT (jpg|gif|png|css|js|ico)
- Method: POST (only intercept form submissions)

# Match & Replace Rules (automate modifications)
Proxy → Options → Match and Replace:
- Replace header: User-Agent → Custom Agent
- Add header: X-Forwarded-For: 127.0.0.1
- Replace body: "role":"user" → "role":"admin"

# Response Modification
- Enable "Intercept Server Responses"
- Modify response before browser renders
- Remove security headers
- Modify JavaScript
- Change hidden form field values

# HTTP History
- All requests/responses logged
- Filter by: domain, status, method, search term
- Color-code requests
- Add comments
- Send to other tools
```

---

## 3. Scanner — Automated Vulnerability Detection

### Active vs Passive Scanning

| Feature | Passive Scan | Active Scan |
|---------|-------------|-------------|
| **How it works** | Analyzes proxy traffic | Sends crafted requests |
| **Invasiveness** | Non-invasive | May modify data |
| **What it finds** | Info leaks, missing headers, sensitive data | SQLi, XSS, command injection, etc. |
| **When to use** | Always (runs automatically) | Targeted endpoints |
| **Performance** | No extra requests | Many additional requests |

### Running Scans

```
Active Scanning:
1. Right-click request in HTTP History
2. "Scan" or "Do active scan"
3. Configure scan options:
   - Audit: Check for common vulnerabilities
   - Crawl: Discover new content
   
Scan Configuration:
- Insert point types: URL params, body params, cookies, headers
- Scan speed: Fast / Normal / Thorough
- Issue types: All / Specific categories

Dashboard → Issue Activity:
- Shows discovered vulnerabilities
- Confidence: Certain / Firm / Tentative
- Severity: High / Medium / Low / Information
- Click for details, evidence, remediation advice
```

### Custom Scan Configurations

```
# Scan specific parameters only
1. Send request to Intruder
2. Mark specific parameter positions
3. Right-click marked area → "Scan defined insertion points"

# Scan with authentication
1. Project Options → Sessions → Session Handling Rules
2. Add rule: "Check session is valid"
3. Define login macro to re-authenticate
4. Run scan with session handling

# Scan schedule
1. New Scan → Crawl and Audit
2. Target URL
3. Schedule: Immediate / After previous / Custom time
```

---

## 4. Intruder — Automated Attacks

### Attack Types

```
┌────────────────────────────────────────────────────────────┐
│                    INTRUDER ATTACK TYPES                    │
├────────────┬───────────────────────────────────────────────┤
│  Sniper    │ Single payload set, one position at a time   │
│            │ §param1§=PAYLOAD, param2=fixed               │
│            │ param1=fixed, §param2§=PAYLOAD               │
│            │ Best for: Testing each parameter individually│
├────────────┼───────────────────────────────────────────────┤
│ Battering  │ Single payload set, all positions            │
│ Ram        │ §param1§=PAYLOAD, §param2§=PAYLOAD           │
│            │ Same payload in all positions simultaneously  │
│            │ Best for: Same value in multiple fields       │
├────────────┼───────────────────────────────────────────────┤
│ Pitchfork  │ Multiple payload sets, synchronized          │
│            │ §user§=payload1[0], §pass§=payload2[0]       │
│            │ §user§=payload1[1], §pass§=payload2[1]       │
│            │ Best for: Known username:password pairs       │
├────────────┼───────────────────────────────────────────────┤
│ Cluster    │ Multiple payload sets, all combinations      │
│ Bomb       │ §user§=payload1[0], §pass§=payload2[0]       │
│            │ §user§=payload1[0], §pass§=payload2[1]       │
│            │ §user§=payload1[1], §pass§=payload2[0]       │
│            │ Best for: Brute force (user × password)       │
└────────────┴───────────────────────────────────────────────┘
```

### Intruder Examples

```
# Brute Force Login
POST /api/login HTTP/1.1
Host: target.com
Content-Type: application/json

{"username":"§admin§","password":"§password§"}

Attack type: Cluster Bomb
Payload 1 (username): admin, administrator, root, test
Payload 2 (password): password, admin, 123456, letmein

# IDOR Testing
GET /api/users/§1§/profile HTTP/1.1
Host: target.com
Authorization: Bearer token123

Attack type: Sniper
Payload: Numbers 1-1000

# Analyze results:
- Sort by Status Code (200 = accessible)
- Sort by Response Length (different length = different user)
- Grep for specific keywords in responses

# Directory Enumeration
GET /api/§path§ HTTP/1.1
Host: target.com

Attack type: Sniper
Payload: Load from wordlist file

# Filter results:
- Exclude 404 responses
- Sort by response length
```

### Payload Processing

```
Payload processing rules:
1. Prefix: Add string before payload
2. Suffix: Add string after payload
3. Hash: MD5, SHA1, SHA256
4. Encode: URL, Base64, HTML
5. Match/Replace: Regex substitution
6. Case: Upper, Lower
7. Skip if matches regex

Example: Base64 encode credentials
Payload: admin:password123
Process: Base64 encode
Result: YWRtaW46cGFzc3dvcmQxMjM=
```

---

## 5. Repeater — Manual Testing

### Using Repeater Effectively

```
Workflow:
1. Capture request in Proxy
2. Right-click → "Send to Repeater" (Ctrl+R)
3. Modify request in Repeater
4. Click "Send" to see response
5. Compare responses for different inputs

Key Features:
- Multiple tabs for different test cases
- Request/Response side-by-side
- Render tab (see page as browser would)
- Auto-highlight differences
- Send group in parallel (race conditions)

# Tips:
- Use Ctrl+Shift+U to URL-encode selected text
- Use Ctrl+Shift+B to Base64-encode selected text
- Right-click → "Change request method" (GET ↔ POST)
- Inspector panel for structured parameter editing
```

### Repeater Testing Scenarios

```http
# SQL Injection testing
POST /api/search HTTP/1.1
Content-Type: application/json

{"query": "test' OR '1'='1"}     # Test 1
{"query": "test' UNION SELECT 1--"}  # Test 2
{"query": "test' AND SLEEP(5)--"}    # Test 3

# XSS testing
PUT /api/profile HTTP/1.1
Content-Type: application/json

{"name": "<script>alert(1)</script>"}        # Test 1
{"name": "<img src=x onerror=alert(1)>"}     # Test 2
{"name": "{{7*7}}"}                          # SSTI Test

# SSRF testing
POST /api/fetch HTTP/1.1
Content-Type: application/json

{"url": "http://127.0.0.1:80"}               # Test 1
{"url": "http://169.254.169.254/latest/"}    # Test 2
{"url": "file:///etc/passwd"}                # Test 3
```

---

## 6. Collaborator — Out-of-Band Testing

### Burp Collaborator Usage

```
Burp Collaborator provides:
- Unique subdomain for OOB testing
- DNS, HTTP, HTTPS, SMTP interaction logging
- Polls for interactions automatically

Use Cases:
┌──────────────────────────────────────────────┐
│ 1. Blind SSRF Detection                     │
│    Payload: http://xyz.oastify.com           │
│    If server makes request → confirmed SSRF  │
│                                              │
│ 2. Blind XSS Detection                       │
│    Payload: <img src=http://xyz.oastify.com> │
│    If admin views → confirmed stored XSS     │
│                                              │
│ 3. Blind SQLi (OOB)                          │
│    Payload: LOAD_FILE('\\\\xyz.oastify.com') │
│    DNS query confirms SQLi                   │
│                                              │
│ 4. Blind XXE                                 │
│    DTD fetched from: http://xyz.oastify.com  │
│    Confirms XXE processing                   │
│                                              │
│ 5. Email Header Injection                    │
│    Test SMTP interaction logging              │
└──────────────────────────────────────────────┘

# Generate Collaborator payload:
Burp → Collaborator → Copy to clipboard
# Result: abc123.oastify.com
# Use this domain in payloads
# Monitor Collaborator tab for interactions
```

---

## 7. Essential Burp Extensions

### Must-Have Extensions

| Extension | Purpose | Use Case |
|-----------|---------|----------|
| **Autorize** | Authorization testing | IDOR/BOLA automated testing |
| **Turbo Intruder** | High-speed attacks | Race conditions, mass fuzzing |
| **Logger++** | Enhanced logging | Advanced filtering, export |
| **Param Miner** | Hidden parameter discovery | Find secret params |
| **JWT Editor** | JWT manipulation | Token attacks |
| **InQL** | GraphQL testing | Schema discovery, attacks |
| **Upload Scanner** | File upload testing | Extension/MIME bypass |
| **Active Scan++** | Enhanced scanning | Additional scan checks |
| **Hackvertor** | Encoding/decoding | Payload encoding |
| **HTTP Request Smuggler** | Request smuggling | HTTP desync attacks |
| **Collaborator Everywhere** | OOB everywhere | Inject Collaborator payloads |
| **Software Vulnerability Scanner** | CVE detection | Known vulnerability checks |

### Installing Extensions

```
BApp Store:
1. Extender → BApp Store
2. Search for extension
3. Click "Install"
4. Some require Jython (for Python extensions):
   - Download jython-standalone.jar
   - Extender → Options → Python Environment
   - Set path to jython jar
```

---

## 8. Burp Suite Tips and Tricks

### Productivity Tips

```
# Keyboard Shortcuts
Ctrl+R         Send to Repeater
Ctrl+I         Send to Intruder
Ctrl+Shift+R   Send to active scan
Ctrl+Space     Send request (in Repeater)
Ctrl+Z         Undo
Ctrl+F         Find in current view
Ctrl+Shift+U   URL-encode selected text
Ctrl+Shift+B   Base64-encode selected text
Ctrl+U         URL-decode selected text
Tab            Switch between request/response

# Project Settings
- Save project files for persistence
- Use session handling rules for authenticated scanning
- Configure scope to focus on target only
- Use bambdas for custom filtering (Burp 2024+)

# Match and Replace Rules (automate testing)
Proxy → Options → Match and Replace:
- Auto-add testing headers to every request
- Auto-modify parameters
- Strip security headers from responses
- Add debug parameters

# Macro Recording (for complex workflows)
1. Project Options → Sessions → Macros → Add
2. Record sequence: login → navigate → action
3. Use in session handling rules
4. Scanner/Intruder auto-replays macro when session expires
```

### Scope Configuration

```
Target → Scope:
- Include: *.target.com
- Exclude: *.google-analytics.com
- Exclude: *.cdn.com

Benefits of proper scope:
- HTTP History only shows relevant traffic
- Scanner only tests in-scope URLs
- Reduces noise significantly
- Prevents testing out-of-scope assets
```

---

## Summary Table

| Component | Purpose | Key Usage |
|-----------|---------|-----------|
| **Proxy** | Intercept and modify traffic | Foundation of all testing |
| **Scanner** | Automated vulnerability detection | Active + passive scanning |
| **Intruder** | Automated customized attacks | Brute force, fuzzing, IDOR |
| **Repeater** | Manual request manipulation | Precise vulnerability testing |
| **Decoder** | Encoding/decoding | Payload crafting |
| **Sequencer** | Token randomness analysis | Session token strength |
| **Collaborator** | Out-of-band testing | Blind vulnerabilities |
| **Extensions** | Extended functionality | Autorize, Turbo Intruder, etc. |

---

## Revision Questions

1. Explain the four Intruder attack types (Sniper, Battering Ram, Pitchfork, Cluster Bomb). When would you use each?
2. How does Burp Collaborator help detect blind vulnerabilities? Give three examples.
3. Describe the process of setting up Burp Suite as a proxy for mobile application testing.
4. What is the difference between active and passive scanning? When should you use each?
5. List five essential Burp extensions and explain what each does.
6. How would you use session handling rules and macros to maintain authentication during automated scanning?

---

*Next: [02-owasp-zap.md](02-owasp-zap.md)*

---

*[Back to README](../README.md)*
