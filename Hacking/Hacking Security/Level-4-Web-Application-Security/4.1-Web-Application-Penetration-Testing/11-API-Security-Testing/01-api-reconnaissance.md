# Unit 11: API Security Testing - Topic 1: API Reconnaissance

## Overview

API reconnaissance is the process of discovering, mapping, and understanding an application's API endpoints, parameters, authentication mechanisms, and data structures before testing for vulnerabilities. Modern web applications heavily rely on APIs (REST, GraphQL, SOAP, gRPC), making API reconnaissance a critical first step in web application penetration testing. Effective API recon reveals the application's **attack surface**, helps identify hidden or undocumented endpoints, and provides the information needed to conduct targeted security testing.

---

## 1. API Discovery Fundamentals

### Types of APIs

```
┌─────────────────────────────────────────────────────────────┐
│                     API Types                               │
├──────────────┬──────────────┬───────────┬──────────────────┤
│   REST API   │   GraphQL    │  SOAP API │    gRPC API      │
├──────────────┼──────────────┼───────────┼──────────────────┤
│ HTTP methods │ Single       │ XML-based │ Protocol Buffers │
│ JSON/XML     │ endpoint     │ WSDL      │ HTTP/2           │
│ Stateless    │ Query lang   │ Strict    │ Binary format    │
│ URL-based    │ Schema       │ contract  │ Streaming        │
│ resources    │ introspect   │           │                  │
└──────────────┴──────────────┴───────────┴──────────────────┘
```

### API Discovery Sources

| Source | What to Look For | Tools |
|--------|-----------------|-------|
| **Documentation** | Swagger/OpenAPI, Postman collections | Browser, curl |
| **JavaScript files** | API endpoints, keys, routes | LinkFinder, JSParser |
| **Mobile apps** | API calls, endpoints, tokens | APKTool, Frida |
| **Network traffic** | API requests/responses | Burp Suite, mitmproxy |
| **Source code** | Route definitions, controllers | grep, Semgrep |
| **Error messages** | Endpoint hints, stack traces | Manual testing |
| **Robots.txt/sitemap** | Disallowed API paths | Browser, curl |
| **WSDL/WADL** | SOAP service definitions | SoapUI |
| **GraphQL introspection** | Full schema | GraphQL Voyager |
| **Git repositories** | API code, configs | GitDorking |

---

## 2. Finding API Documentation

### Common Documentation Endpoints

```bash
# Swagger/OpenAPI documentation
/swagger.json
/swagger/v1/swagger.json
/api/swagger.json
/swagger-ui.html
/swagger-ui/
/api-docs
/api-docs/v1
/v1/api-docs
/v2/api-docs
/v3/api-docs
/openapi.json
/openapi.yaml
/api/openapi.json
/docs
/api/docs
/redoc

# GraphQL endpoints
/graphql
/graphiql
/graphql/console
/graphql/playground
/api/graphql
/v1/graphql

# WSDL (SOAP)
/service.wsdl
/services?wsdl
/?wsdl
/api?wsdl

# Postman collections
/collection.json
/api/collection

# API versioning discovery
/api/v1/
/api/v2/
/api/v3/
/v1/
/v2/
```

### Automated Documentation Discovery

```bash
# ffuf for API doc discovery
ffuf -u "http://target.com/FUZZ" \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-docs.txt \
  -mc 200 -t 50

# Wordlist for common API paths
ffuf -u "http://target.com/FUZZ" \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
  -mc 200,301,302 -fs 0

# kiterunner - API discovery tool
kr scan http://target.com -w routes-large.kite
kr scan http://target.com -A apiroutes-210228

# Check for swagger specifically
for path in swagger.json swagger/v1/swagger.json api-docs openapi.json api/swagger.json; do
    code=$(curl -s -o /dev/null -w "%{http_code}" "http://target.com/$path")
    echo "$code - /$path"
done
```

---

## 3. JavaScript Analysis for API Endpoints

### Extracting API Endpoints from JavaScript

```bash
# LinkFinder - Find endpoints in JS files
python3 linkfinder.py -i http://target.com -d -o results.html

# For specific JS file
python3 linkfinder.py -i http://target.com/static/app.js -o cli

# JSParser
python3 handler.py -u http://target.com

# Manual grep for API patterns
curl -s http://target.com/app.js | grep -oP '["'"'"'](/api/[^"'"'"']+)["'"'"']'

# Extract from all JS files
# Step 1: Find all JS files
gospider -s http://target.com -d 3 --js | grep "\.js$" > js_files.txt

# Step 2: Download and search
while read url; do
    curl -s "$url" | grep -oP '["'"'"'](\/api\/[^"'"'"'\s]+)["'"'"']'
done < js_files.txt | sort -u

# Step 3: Look for API keys and secrets in JS
curl -s http://target.com/app.js | grep -iE "(api[_-]?key|api[_-]?secret|token|authorization|bearer|password)"

# truffleHog for secrets in JS
trufflehog filesystem --directory ./js_files/
```

### Common Patterns in JavaScript

```javascript
// API endpoints in JavaScript code
fetch('/api/v1/users')
axios.get('/api/users/profile')
$.ajax({ url: '/api/orders' })
XMLHttpRequest.open('GET', '/api/data')

// API base URL
const API_BASE = 'https://api.target.com/v2'
const BASE_URL = process.env.REACT_APP_API_URL

// API keys (should NOT be in client-side JS)
const API_KEY = 'sk_live_abc123def456'
headers: { 'Authorization': 'Bearer eyJhbGciOiJIUzI1NiJ9...' }

// Route definitions (React/Angular/Vue)
path: '/admin/dashboard'
path: '/api/internal/debug'
path: '/api/admin/users'
```

---

## 4. API Endpoint Enumeration

### Directory Brute-Forcing for APIs

```bash
# Gobuster for API paths
gobuster dir -u http://target.com/api/ \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
  -t 50 --no-error

# ffuf with method testing
ffuf -u "http://target.com/api/FUZZ" \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
  -mc 200,201,204,301,401,403,405 \
  -X GET

# Test multiple HTTP methods
for method in GET POST PUT DELETE PATCH OPTIONS HEAD; do
    echo "=== $method ==="
    curl -s -o /dev/null -w "%{http_code}" -X $method http://target.com/api/users
done

# Arjun - parameter discovery
arjun -u http://target.com/api/users -m GET POST
arjun -u http://target.com/api/search --stable

# Param Miner (Burp Extension) for hidden parameters
# Automatically discovers hidden parameters in requests
```

### API Versioning Enumeration

```bash
# Test for different API versions
for v in v1 v2 v3 v4 v5; do
    code=$(curl -s -o /dev/null -w "%{http_code}" "http://target.com/api/$v/users")
    echo "$code - /api/$v/users"
done

# Check header-based versioning
curl -H "Accept: application/vnd.api.v1+json" http://target.com/api/users
curl -H "Accept: application/vnd.api.v2+json" http://target.com/api/users
curl -H "Api-Version: 1" http://target.com/api/users
curl -H "Api-Version: 2" http://target.com/api/users

# Older API versions may lack security controls!
# v1 might not have rate limiting or auth checks
# that were added in v2
```

### Parameter Discovery

```bash
# Arjun for hidden parameters
arjun -u http://target.com/api/users -m GET
# Output: Found parameters: id, name, email, role, admin, debug

# x8 - Hidden parameter discovery
x8 -u "http://target.com/api/search" -w params.txt

# Common hidden parameters to test
?debug=true
?test=true
?admin=true
?internal=true
?verbose=true
?format=json
?callback=jsonp
?_method=PUT
?fields=*
?include=all
?expand=*
```

---

## 5. Traffic Analysis and API Mapping

### Intercepting API Traffic with Burp Suite

```
Methodology:
1. Configure browser/app to use Burp proxy
2. Browse/use the application normally
3. Review all API calls in HTTP History
4. Note:
   - Endpoints and HTTP methods
   - Authentication headers
   - Request/response formats
   - Parameter names and types
   - Error responses and codes
   
5. Use Target → Site Map for visual overview
6. Export API map for documentation
```

### Using mitmproxy for API Capture

```bash
# Start mitmproxy
mitmproxy -p 8080

# For mobile app traffic
mitmproxy -p 8080 --mode transparent

# Dump all API calls to file
mitmdump -p 8080 -w api_traffic.flow

# Filter for API requests only
mitmdump -p 8080 --set flow_detail=3 \
  "~u /api/"

# Export as HAR
mitmdump -p 8080 --set hardump=api_traffic.har
```

### Building an API Map

```
API Endpoint Map Example:

Method  Endpoint                    Auth    Description
─────── ─────────────────────────── ─────── ───────────────────────
GET     /api/v1/users               Token   List all users
GET     /api/v1/users/{id}          Token   Get specific user
POST    /api/v1/users               Admin   Create user
PUT     /api/v1/users/{id}          Token   Update user
DELETE  /api/v1/users/{id}          Admin   Delete user
GET     /api/v1/users/{id}/orders   Token   User's orders
POST    /api/v1/auth/login          None    Authenticate
POST    /api/v1/auth/register       None    Register
POST    /api/v1/auth/reset          None    Password reset
GET     /api/v1/products            None    List products
GET     /api/v1/products/{id}       None    Product details
POST    /api/v1/orders              Token   Create order
GET     /api/v1/orders/{id}         Token   Order details
GET     /api/v1/admin/stats         Admin   Admin statistics
POST    /api/v1/admin/config        Admin   Update config
GET     /api/internal/health        None    Health check
GET     /api/internal/debug         None!   Debug info (EXPOSED!)
```

---

## 6. OpenAPI/Swagger Analysis

### Parsing Swagger Documentation

```bash
# Download and analyze Swagger file
curl -s http://target.com/swagger.json | python3 -m json.tool

# Extract all endpoints
curl -s http://target.com/swagger.json | \
  python3 -c "
import json, sys
spec = json.load(sys.stdin)
for path, methods in spec.get('paths', {}).items():
    for method in methods:
        if method.upper() in ['GET','POST','PUT','DELETE','PATCH']:
            print(f'{method.upper():7s} {path}')
"

# Import into Postman
# 1. Open Postman
# 2. Import → Link → paste swagger URL
# 3. Full API collection with all endpoints

# Import into Burp Suite
# Use "OpenAPI Parser" extension
# File → Import → select swagger.json
```

### Converting Swagger to Wordlists

```bash
# Generate endpoint wordlist from Swagger
curl -s http://target.com/swagger.json | \
  python3 -c "
import json, sys
spec = json.load(sys.stdin)
for path in spec.get('paths', {}).keys():
    # Replace path parameters with common values
    fuzzed = path.replace('{id}', '1').replace('{userId}', '1')
    print(fuzzed)
" > api_endpoints.txt

# Use with ffuf
ffuf -u "http://target.com/FUZZ" \
  -w api_endpoints.txt \
  -mc 200,201,204,401,403
```

---

## 7. Mobile App API Discovery

### Decompiling Mobile Apps

```bash
# Android APK Analysis
# Decompile APK
apktool d target_app.apk -o decompiled/

# Search for API endpoints
grep -rn "http://" decompiled/ --include="*.smali"
grep -rn "https://" decompiled/ --include="*.smali"
grep -rn "api" decompiled/ --include="*.xml"

# Use jadx for Java source
jadx target_app.apk -d java_source/
grep -rn "/api/" java_source/ --include="*.java"

# iOS IPA Analysis
# Unzip IPA
unzip target_app.ipa -d extracted/

# Search binary for strings
strings extracted/Payload/App.app/App | grep -i "api\|http\|endpoint"

# Use Frida for runtime API interception
frida -U -n target_app -l api_trace.js
```

### Runtime API Interception

```javascript
// Frida script to intercept API calls (api_trace.js)
Java.perform(function() {
    // Hook OkHttp
    var Builder = Java.use("okhttp3.Request$Builder");
    Builder.build.implementation = function() {
        var request = this.build();
        console.log("URL: " + request.url().toString());
        console.log("Method: " + request.method());
        return request;
    };
    
    // Hook Retrofit
    var Retrofit = Java.use("retrofit2.Retrofit");
    Retrofit.create.implementation = function(cls) {
        console.log("Retrofit API: " + cls.getName());
        return this.create(cls);
    };
});
```

---

## 8. API Reconnaissance Tools Summary

| Tool | Purpose | Type |
|------|---------|------|
| **Kiterunner** | API endpoint discovery | Active scanning |
| **ffuf** | Fuzzing API paths and parameters | Active scanning |
| **Arjun** | Hidden parameter discovery | Active scanning |
| **LinkFinder** | Extract endpoints from JS | Passive analysis |
| **Postman** | API testing and documentation | Interactive |
| **Burp Suite** | Proxy, intercept, analyze | Comprehensive |
| **mitmproxy** | Traffic interception | Passive capture |
| **Swagger Editor** | OpenAPI analysis | Documentation |
| **GraphQL Voyager** | GraphQL schema visualization | Documentation |
| **APKTool/jadx** | Mobile app decompilation | Static analysis |
| **Frida** | Runtime API interception | Dynamic analysis |
| **gospider** | Web spidering for JS/URLs | Active crawling |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Documentation Discovery** | Swagger, OpenAPI, WSDL at predictable paths |
| **JavaScript Analysis** | Extract endpoints, keys, routes from client JS |
| **Endpoint Enumeration** | Brute force with API-specific wordlists |
| **Version Discovery** | Test v1, v2, v3; older versions may lack security |
| **Parameter Discovery** | Arjun, x8 for hidden parameters |
| **Traffic Analysis** | Burp/mitmproxy to capture and map all API calls |
| **Mobile App Recon** | Decompile APK/IPA, intercept with Frida |
| **API Mapping** | Build complete endpoint map with methods, auth, params |

---

## Revision Questions

1. What are the common paths where Swagger/OpenAPI documentation can be found? Why is exposed API documentation a security concern?
2. How would you use LinkFinder to extract API endpoints from JavaScript files? What should you look for?
3. Explain the importance of testing older API versions (v1 vs v2). What security differences might exist?
4. Describe how Kiterunner differs from traditional directory brute-forcing tools for API discovery.
5. How would you intercept and analyze API traffic from a mobile application?
6. What is the process for converting discovered API documentation into a comprehensive attack surface map?

---

*Next: [02-api-authentication-testing.md](02-api-authentication-testing.md)*

---

*[Back to README](../README.md)*
