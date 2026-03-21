# Parameter Discovery

## Unit 3: Web Reconnaissance — Topic 4

## 🎯 Overview

Parameter discovery is the process of finding hidden or undocumented parameters that a web application accepts. These parameters often control functionality like debug modes, admin features, or internal configurations. Discovering hidden parameters can unlock new attack vectors including privilege escalation, information disclosure, and access control bypass.

---

## 1. Why Parameters Matter

```
┌──────────────────────────────────────────────────────────────┐
│              PARAMETER TYPES                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  URL Parameters    │  /page?id=1&debug=true&role=admin       │
│  POST Body         │  username=test&admin=1&bypass=true      │
│  Headers           │  X-Debug: true, X-Custom-Auth: admin    │
│  Cookies           │  role=user; debug=0; internal=false     │
│  JSON Body         │  {"user":"test","admin":true}           │
│  Path Parameters   │  /api/users/123/orders                  │
│  Fragment          │  #section=admin (DOM-based)             │
│                                                              │
│  Hidden parameters may enable:                               │
│  • Debug/verbose mode                                        │
│  • Admin functionality                                       │
│  • Rate limit bypass                                         │
│  • Authentication bypass                                     │
│  • Different application behavior                            │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Parameter Discovery Tools

### Arjun — Automated Parameter Discovery

```bash
# Find hidden GET parameters
arjun -u https://target.com/page --get

# Find hidden POST parameters
arjun -u https://target.com/api/login --post

# Find JSON parameters
arjun -u https://target.com/api/user --json

# Custom wordlist
arjun -u https://target.com/page -w custom-params.txt

# Multiple URLs from file
arjun -i urls.txt --get
```

### ParamSpider

```bash
# Mine parameters from web archives
paramspider -d target.com

# Output: URLs with parameters from web archives
# https://target.com/page?id=FUZZ
# https://target.com/search?q=FUZZ&category=FUZZ
```

### ffuf Parameter Fuzzing

```bash
# Fuzz GET parameters
ffuf -u "https://target.com/page?FUZZ=test" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -fs 4242

# Fuzz POST parameters
ffuf -u https://target.com/api/login \
  -X POST -d "username=admin&FUZZ=true" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -fs 4242

# Fuzz parameter values
ffuf -u "https://target.com/page?debug=FUZZ" \
  -w /usr/share/seclists/Fuzzing/true-false-values.txt \
  -fs 4242

# Fuzz JSON keys
ffuf -u https://target.com/api/user \
  -X POST -H "Content-Type: application/json" \
  -d '{"username":"test","FUZZ":"true"}' \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
```

---

## 3. Common Hidden Parameters

```bash
# Debug and development parameters
debug=true, debug=1, verbose=true, test=true
_debug=1, development=1, env=dev, mode=debug

# Authentication and authorization
admin=true, role=admin, is_admin=1, isAdmin=true
access=admin, user_type=admin, auth=bypass
token=, api_key=, key=, secret=

# Rate limiting bypass
no_limit=1, bypass_rate=true, internal=true
X-Forwarded-For: 127.0.0.1 (header)

# Output control
format=json, output=xml, response_type=full
fields=all, include=password, expand=true

# Pagination manipulation
limit=99999, offset=0, page_size=10000
per_page=1000, count=all
```

---

## 4. Source-Based Parameter Discovery

```bash
# Extract parameters from JavaScript files
curl -s https://target.com/app.js | grep -oE '[?&][a-zA-Z_]+=' | sort -u

# Extract from HTML forms
curl -s https://target.com/page | grep -oE 'name="[^"]*"' | sort -u

# Extract from API documentation
curl -s https://target.com/swagger.json | python3 -m json.tool

# Check web archives for old parameters
curl "https://web.archive.org/web/*/target.com/page?*" 2>/dev/null

# LinkFinder for JS endpoint parameters
python3 linkfinder.py -i https://target.com/app.js -o cli
```

---

## 5. Parameter Tampering Techniques

```
┌──────────────────────────────────────────────────────────────┐
│              PARAMETER TAMPERING                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Mass Assignment / Object Injection:                         │
│  POST /api/register                                          │
│  {"username":"hacker","email":"h@test.com","role":"admin"}   │
│  # Server may accept "role" even if not in the form         │
│                                                              │
│  HTTP Parameter Pollution (HPP):                             │
│  /transfer?amount=100&to=victim&amount=99999                │
│  # Which "amount" does the server use?                      │
│  # Backend may use first, last, or concatenation            │
│                                                              │
│  Type Juggling:                                              │
│  /api/verify?code=true                                       │
│  /api/verify?code[]=                                         │
│  # PHP: true == "any_string" is TRUE                        │
│  # Sending array may bypass string comparison               │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Test mass assignment
curl -X POST https://target.com/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test","email":"t@test.com","role":"admin","isAdmin":true}'

# Test HTTP Parameter Pollution
curl "https://target.com/action?user=normal&user=admin"
curl -X POST https://target.com/action \
  -d "user=normal&user=admin"

# Test array parameter injection (PHP)
curl "https://target.com/verify?code[]=&code[]=admin"
```

---

## 6. Burp Suite Parameter Analysis

```
┌──────────────────────────────────────────────────────────────┐
│           BURP SUITE PARAMETER WORKFLOW                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Proxy → HTTP History                                     │
│     View all parameters in each request                      │
│                                                              │
│  2. Target → Site Map → right-click → "Engagement tools"    │
│     → "Analyze Target" → shows all parameters                │
│                                                              │
│  3. Intruder → select parameter → add payload positions     │
│     Fuzz parameter names and values                          │
│                                                              │
│  4. Param Miner extension                                    │
│     Automatically discovers hidden parameters                │
│     Right-click → Extensions → Param Miner → Guess params   │
│                                                              │
│  5. Active Scan                                              │
│     Automatically tests all discovered parameters            │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Method | Approach | Tool |
|--------|----------|------|
| Automated fuzzing | Brute-force parameter names | Arjun, ffuf |
| Source analysis | Extract from JS/HTML | grep, LinkFinder |
| Archive mining | Historical parameters | ParamSpider, Wayback |
| Proxy analysis | Observe during browsing | Burp Suite, ZAP |
| Documentation | API docs, Swagger | Manual review |
| Mass assignment | Add extra fields | cURL, Burp Repeater |

---

## ❓ Revision Questions

1. What types of hidden parameters are commonly found in web applications?
2. How does Arjun discover hidden parameters?
3. What is HTTP Parameter Pollution and how can it be exploited?
4. How can mass assignment lead to privilege escalation?
5. Why should JavaScript files be analyzed for parameter discovery?
6. What is the difference between parameter fuzzing and parameter tampering?

---

*Previous: [03-directory-enumeration.md](03-directory-enumeration.md) | Next: [05-functionality-mapping.md](05-functionality-mapping.md)*

---

*[Back to README](../README.md)*
