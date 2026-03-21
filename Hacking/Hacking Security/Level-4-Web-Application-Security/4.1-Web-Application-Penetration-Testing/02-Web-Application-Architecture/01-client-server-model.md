# Client-Server Model

## Unit 2: Web Application Architecture — Topic 1

## 🎯 Overview

The client-server model is the foundational architecture of web applications. Understanding how clients and servers interact, where data processing occurs, and how trust boundaries work is essential for identifying attack surfaces. This topic covers architecture models, trust boundaries, and security implications from a penetration testing perspective.

---

## 1. Basic Client-Server Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                  CLIENT-SERVER MODEL                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐       Internet        ┌──────────────────┐    │
│  │  Client  │ ═════════════════════ │     Server       │    │
│  │ (Browser)│    HTTP/HTTPS         │  ┌────────────┐  │    │
│  │          │    Requests &         │  │ Web Server │  │    │
│  │ HTML/CSS │    Responses          │  │  (nginx)   │  │    │
│  │ JavaScript│                      │  └─────┬──────┘  │    │
│  │          │                       │        │         │    │
│  └──────────┘                       │  ┌─────▼──────┐  │    │
│                                     │  │ App Server │  │    │
│  Client-side:                       │  │  (Node.js) │  │    │
│  • Rendering                        │  └─────┬──────┘  │    │
│  • UI logic                         │        │         │    │
│  • Form validation                  │  ┌─────▼──────┐  │    │
│  • Local storage                    │  │  Database  │  │    │
│                                     │  │  (MySQL)   │  │    │
│                                     │  └────────────┘  │    │
│                                     └──────────────────┘    │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Multi-Tier Architecture

```
┌─────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────┐
│ Client  │   │ Presentation │   │   Business   │   │   Data   │
│  Tier   │──→│    Tier       │──→│    Logic     │──→│   Tier   │
│         │   │              │   │    Tier       │   │          │
│ Browser │   │  Web Server  │   │  App Server  │   │ Database │
│ Mobile  │   │  nginx       │   │  Django      │   │ PostgreSQL│
│ API     │   │  Apache      │   │  Express     │   │ MongoDB  │
│ client  │   │  CDN         │   │  Spring Boot │   │ Redis    │
└─────────┘   └──────────────┘   └──────────────┘   └──────────┘

Attack Surface:     SSL/TLS           Server-side          SQL injection
                    Config issues      logic flaws          Data exposure
                    Info disclosure   Auth/AuthZ bugs       NoSQL injection
```

### Trust Boundaries

```
┌───────────────────────────────────────────────────────────────┐
│                     TRUST BOUNDARIES                           │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  UNTRUSTED            │  TRUST BOUNDARY  │  TRUSTED           │
│  ═══════════          │  ══════════════  │  ═══════           │
│                       │                  │                    │
│  • User input         │  ← Validation →  │  • Server logic    │
│  • Client-side code   │  ← Sanitization  │  • Database        │
│  • Cookies            │  ← Authentication│  • File system     │
│  • URL parameters     │  ← Authorization │  • Internal APIs   │
│  • HTTP headers       │                  │  • Config files    │
│  • File uploads       │                  │  • Secrets         │
│                       │                  │                    │
│  ⚠ NEVER trust       │                  │  ✓ Validate all    │
│    client-side data   │                  │    inputs here     │
└───────────────────────────────────────────────────────────────┘
```

---

## 3. Security Implications

### Client-Side vs Server-Side Validation

| Aspect | Client-Side | Server-Side |
|--------|------------|-------------|
| Location | Browser (JavaScript) | Server (backend code) |
| Bypassable? | ✅ Yes — always | ❌ No (if properly implemented) |
| Purpose | UX improvement | Security enforcement |
| Speed | Instant feedback | Requires round-trip |
| Trustworthy? | ❌ Never | ✅ Yes |

```bash
# Bypassing client-side validation
# 1. Disable JavaScript in browser
# 2. Use Burp Suite to modify request after validation
# 3. Use cURL to send requests directly

# Example: Client checks password length >= 8
# But server doesn't — send short password via cURL
curl -X POST https://target.com/register \
  -d "user=admin&pass=ab"
```

### Common Architecture Vulnerabilities

| Layer | Vulnerability | Example |
|-------|--------------|---------|
| Client | DOM XSS | `eval(location.hash)` |
| Presentation | Info disclosure | Server version in headers |
| Business Logic | Auth bypass | Broken access control |
| Data | SQL injection | Unsanitized queries |
| Network | MitM | HTTP instead of HTTPS |

---

## 4. Modern Architecture Patterns

### Single Page Application (SPA)

```
┌──────────────────────────────────────────────────────────────┐
│                  SPA ARCHITECTURE                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐     API Calls       ┌──────────────────┐  │
│  │  SPA Client  │ ══════════════════ │   Backend API    │  │
│  │              │   JSON/REST         │                  │  │
│  │  React       │←══════════════════ │  /api/users      │  │
│  │  Angular     │   JSON responses    │  /api/orders     │  │
│  │  Vue.js      │                     │  /api/auth       │  │
│  └──────────────┘                     └──────────────────┘  │
│                                                              │
│  Security concerns:                                          │
│  • All business logic exposed in JS                          │
│  • API endpoints directly accessible                         │
│  • Token storage (localStorage vs cookies)                   │
│  • Client-side routing may bypass auth checks                │
└──────────────────────────────────────────────────────────────┘
```

### Serverless Architecture

```
┌──────────┐    API Gateway    ┌─────────────────┐
│  Client  │──────────────────→│  AWS Lambda      │
│          │                   │  Azure Functions  │
│          │                   │  Cloud Functions   │
└──────────┘                   └────────┬──────────┘
                                       │
                               ┌───────▼──────────┐
                               │  Cloud Services   │
                               │  S3, DynamoDB     │
                               │  Cosmos DB        │
                               └──────────────────┘

# Attack surface:
# • Function injection (event data)
# • Misconfigured IAM roles
# • Excessive permissions
# • Insecure dependencies
# • Cold start timing attacks
```

---

## 5. Mapping Application Architecture

```bash
# 1. Identify technologies
whatweb https://target.com
wappalyzer (browser extension)

# 2. Map server infrastructure
nmap -sV -p 80,443,8080,8443 target.com
curl -sI https://target.com | grep -i server

# 3. Identify API endpoints
# Check JavaScript files for API URLs
curl -s https://target.com/app.js | grep -oE '/api/[a-zA-Z/]+'

# 4. Determine architecture type
# SPA: Heavy JS, API calls, single HTML page
# Traditional: Server-rendered HTML, form submissions
# Hybrid: Mix of both approaches
```

---

## 📊 Summary Table

| Architecture | Attack Surface | Key Testing Focus |
|-------------|---------------|-------------------|
| Monolithic | Single application | Traditional web vulns |
| Multi-tier | Each layer boundary | Input validation, AuthZ |
| SPA + API | Exposed API endpoints | API security, token storage |
| Microservices | Inter-service communication | Service-to-service auth |
| Serverless | Function inputs, IAM | Injection, permissions |

---

## ❓ Revision Questions

1. Why should server-side validation never be replaced by client-side validation?
2. What are the key trust boundaries in a web application?
3. How does an SPA architecture change the attack surface compared to traditional server-rendered apps?
4. What security concerns are specific to serverless architectures?
5. How do you identify the technology stack of a target web application?

---

*Next: [02-frontend-technologies.md](02-frontend-technologies.md)*

---

*[Back to README](../README.md)*
