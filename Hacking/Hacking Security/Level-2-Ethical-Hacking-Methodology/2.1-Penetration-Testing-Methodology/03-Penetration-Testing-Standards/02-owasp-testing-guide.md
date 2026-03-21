# OWASP Testing Guide

## Unit 3 - Topic 2 | Penetration Testing Standards

---

## Overview

The **OWASP Testing Guide** is the most comprehensive resource for **web application security testing**. Published by the Open Web Application Security Project, it provides a structured framework with **specific test cases** organized into categories covering every aspect of web app security.

---

## 1. OWASP Testing Framework

```
┌──────────────────────────────────────────────────────────────┐
│              OWASP TESTING GUIDE v4.2                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  11 TESTING CATEGORIES:                                      │
│                                                              │
│  01. Information Gathering         (10 tests)               │
│  02. Configuration and Deployment  (12 tests)               │
│  03. Identity Management           (5 tests)                │
│  04. Authentication                (10 tests)               │
│  05. Authorization                 (4 tests)                │
│  06. Session Management            (9 tests)                │
│  07. Input Validation              (19 tests)               │
│  08. Error Handling                (2 tests)                │
│  09. Cryptography                  (4 tests)                │
│  10. Business Logic                (9 tests)                │
│  11. Client-Side                   (13 tests)               │
│                                                              │
│  Total: ~91 specific test cases                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Key Testing Categories

| Category | ID | Example Tests |
|----------|:---:|-------------|
| **Info Gathering** | OTG-INFO | Search engine recon, fingerprint web server |
| **Config & Deploy** | OTG-CONFIG | Test HTTP methods, admin interfaces |
| **Identity Mgmt** | OTG-IDENT | Test user registration, account enumeration |
| **Authentication** | OTG-AUTHN | Credential transport, lockout, MFA bypass |
| **Authorization** | OTG-AUTHZ | Directory traversal, privilege escalation |
| **Session Mgmt** | OTG-SESS | Cookie attributes, session fixation, CSRF |
| **Input Validation** | OTG-INPVAL | XSS, SQLi, command injection, SSRF |
| **Error Handling** | OTG-ERR | Stack traces, error codes |
| **Cryptography** | OTG-CRYPST | Weak ciphers, padding oracle |
| **Business Logic** | OTG-BUSLOGIC | Workflow bypass, unlimited actions |
| **Client-Side** | OTG-CLIENT | DOM XSS, clickjacking, HTML injection |

---

## 3. Testing Methodology Flow

```
OWASP TESTING PHASES:

Phase 1: PASSIVE TESTING (Information Gathering)
├── Fingerprint web server and technologies
├── Review robots.txt, sitemap.xml
├── Map application architecture
└── Identify entry points

Phase 2: ACTIVE TESTING
├── Configuration testing
│   ├── HTTP methods (PUT, DELETE enabled?)
│   ├── File extensions handled
│   └── Admin interfaces exposed
├── Authentication testing
│   ├── Default credentials
│   ├── Brute force resistance
│   └── Password reset flaws
├── Authorization testing
│   ├── IDOR (Insecure Direct Object Reference)
│   ├── Horizontal privilege escalation
│   └── Vertical privilege escalation
├── Input validation testing
│   ├── SQL Injection
│   ├── Cross-Site Scripting (XSS)
│   ├── Command Injection
│   └── Server-Side Request Forgery (SSRF)
└── Business logic testing
    ├── Data validation
    ├── Integrity checks
    └── Workflow circumvention
```

---

## 4. OWASP Tools

| Tool | Purpose |
|------|---------|
| **ZAP (Zed Attack Proxy)** | Free web app scanner and proxy |
| **OWASP Amass** | Subdomain enumeration |
| **OWASP Dependency Check** | Find vulnerable libraries |
| **OWASP Juice Shop** | Intentionally vulnerable app for practice |
| **OWASP WebGoat** | Learning platform for web security |
| **OWASP ModSecurity CRS** | Web Application Firewall rules |

---

## 5. OWASP vs Other Standards

| Aspect | OWASP Testing Guide | PTES | NIST 800-115 |
|--------|:---:|:---:|:---:|
| **Focus** | Web applications | All pen testing | Technical guidance |
| **Detail level** | Very specific tests | Methodology | High-level |
| **Test cases** | 91+ named tests | General phases | General procedures |
| **Free** | ✅ | ✅ | ✅ |
| **Industry adoption** | Very high (web) | High (general) | Government |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **OWASP Testing Guide** | Most comprehensive web app security testing standard |
| **11 categories** | Info gathering through client-side testing |
| **91+ test cases** | Specific, named tests with procedures |
| **OWASP ZAP** | Free web application security scanner |
| **Best for** | Web application penetration testing |

---

## Quick Revision Questions

1. **How many testing categories does the OWASP Testing Guide have?**
2. **What does OTG-INPVAL cover?**
3. **Name 3 OWASP testing tools.**
4. **What is the difference between passive and active testing?**
5. **How does OWASP Testing Guide compare to PTES?**

---

[← Previous: PTES](01-ptes.md) | [Next: NIST SP 800-115 →](03-nist-sp-800-115.md)

---

*Unit 3 - Topic 2 of 5 | [Back to README](../README.md)*
