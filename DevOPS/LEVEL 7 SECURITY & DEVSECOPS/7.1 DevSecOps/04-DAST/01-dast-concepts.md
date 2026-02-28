# Chapter 4.1: DAST Concepts

## Overview

Dynamic Application Security Testing (DAST) tests running applications from the outside, simulating real-world attacks without access to source code. Unlike SAST which reads code, DAST interacts with live endpoints to find vulnerabilities only visible at runtime.

---

## How DAST Works

```
DAST SCANNING ARCHITECTURE:

  ┌──────────────┐        HTTP Requests        ┌──────────────────┐
  │              │ ──────────────────────────→  │                  │
  │  DAST TOOL   │   GET /login                 │  TARGET APP      │
  │  (Scanner)   │   POST /api/users            │  (Running)       │
  │              │   PUT /api/orders/1           │                  │
  │              │ ←──────────────────────────  │                  │
  │  Analyzes    │        HTTP Responses         │  Real responses  │
  │  Responses   │   200 OK + HTML               │  from live app   │
  └──────┬───────┘   500 Internal Error          └──────────────────┘
         │           302 Redirect
         ▼
  ┌──────────────┐
  │  FINDINGS    │
  │  REPORT      │
  │  - SQLi      │
  │  - XSS       │
  │  - CSRF      │
  │  - Auth bugs │
  └──────────────┘

  KEY PRINCIPLE: DAST is "black-box" testing
  ─ No source code needed
  ─ Tests the running application
  ─ Finds runtime-only vulnerabilities
  ─ Simulates real attacker behavior
```

---

## DAST Scanning Phases

```
DAST SCAN LIFECYCLE:

  Phase 1: CRAWLING (Discovery)
  ┌────────────────────────────────────────────────────────────┐
  │  Spider follows links, forms, and JavaScript to map        │
  │  the application's attack surface:                          │
  │                                                              │
  │  ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐             │
  │  │ /    │───→│/login│───→│/dash │───→│/api  │             │
  │  └──────┘    └──────┘    └──────┘    └──┬───┘             │
  │                                          │                  │
  │                                    ┌─────┴─────┐           │
  │                                    │/api/users │           │
  │                                    │/api/orders│           │
  │                                    └───────────┘           │
  │  Output: Site map with all discovered endpoints             │
  └────────────────────────────────────────────────────────────┘

  Phase 2: ATTACK (Active Scanning)
  ┌────────────────────────────────────────────────────────────┐
  │  For each discovered endpoint, send attack payloads:       │
  │                                                              │
  │  GET /api/users?id=1 OR 1=1           → SQLi test           │
  │  POST /login  {user:"<script>alert(1)</script>"}  → XSS    │
  │  PUT /api/users/2  (as user 1)        → IDOR test          │
  │  GET /admin  (no auth header)         → Auth bypass         │
  │  GET /api/users/../../../etc/passwd   → Path traversal      │
  └────────────────────────────────────────────────────────────┘

  Phase 3: ANALYSIS (Results)
  ┌────────────────────────────────────────────────────────────┐
  │  Analyze responses for vulnerability indicators:            │
  │                                                              │
  │  500 error with SQL syntax     → Confirmed SQLi             │
  │  Response contains <script>    → Reflected XSS              │
  │  200 OK without auth           → Broken access control      │
  │  Different response for ../    → Path traversal              │
  └────────────────────────────────────────────────────────────┘
```

---

## DAST vs Other Testing Methods

```
TESTING METHOD COMPARISON:

  ┌──────────┬────────────┬───────────┬───────────┬───────────┐
  │ Aspect   │ SAST       │ DAST      │ IAST      │ Pen Test  │
  ├──────────┼────────────┼───────────┼───────────┼───────────┤
  │ Analyzes │ Source code│ Running   │ Both code │ Running   │
  │          │            │ app       │ + runtime │ app       │
  ├──────────┼────────────┼───────────┼───────────┼───────────┤
  │ Box Type │ White-box  │ Black-box │ Gray-box  │ Black-box │
  ├──────────┼────────────┼───────────┼───────────┼───────────┤
  │ Finds    │ Code-level │ Runtime   │ Both      │ Logic +   │
  │          │ flaws      │ flaws     │           │ runtime   │
  ├──────────┼────────────┼───────────┼───────────┼───────────┤
  │ Speed    │ Fast       │ Slow      │ Medium    │ Very slow │
  │          │ (minutes)  │ (hours)   │ (minutes) │ (days)    │
  ├──────────┼────────────┼───────────┼───────────┼───────────┤
  │ Stage    │ Build      │ Test/     │ Test      │ Pre-prod  │
  │          │            │ Staging   │           │           │
  ├──────────┼────────────┼───────────┼───────────┼───────────┤
  │ FP Rate  │ Medium-    │ Low       │ Very low  │ Very low  │
  │          │ High       │           │           │           │
  ├──────────┼────────────┼───────────┼───────────┼───────────┤
  │ Language │ Yes        │ No        │ Yes       │ No        │
  │ Specific │            │           │           │           │
  └──────────┴────────────┴───────────┴───────────┴───────────┘
```

---

## What DAST Finds (That SAST Cannot)

```
VULNERABILITIES UNIQUE TO DAST:

  1. MISCONFIGURATION
  ┌────────────────────────────────────────────────────┐
  │ Missing security headers (CSP, HSTS, X-Frame)     │
  │ TLS/SSL misconfigurations                          │
  │ Directory listing enabled                          │
  │ Default credentials still active                   │
  └────────────────────────────────────────────────────┘

  2. AUTHENTICATION / SESSION ISSUES
  ┌────────────────────────────────────────────────────┐
  │ Session fixation                                    │
  │ Session not invalidated on logout                  │
  │ Missing account lockout                             │
  │ Token not rotating on privilege change             │
  └────────────────────────────────────────────────────┘

  3. BUSINESS LOGIC FLAWS
  ┌────────────────────────────────────────────────────┐
  │ Price manipulation (change $100 to $1)             │
  │ Step skipping in multi-step workflows              │
  │ Race conditions in concurrent requests             │
  │ IDOR (access other users' data)                    │
  └────────────────────────────────────────────────────┘

  4. DEPLOYMENT ISSUES
  ┌────────────────────────────────────────────────────┐
  │ Exposed admin panels                                │
  │ Debug mode enabled in production                   │
  │ Backup files accessible (.bak, .old)               │
  │ Source maps exposed                                 │
  └────────────────────────────────────────────────────┘
```

---

## DAST Scan Types

```
  PASSIVE SCAN                       ACTIVE SCAN
  ┌────────────────────┐            ┌────────────────────┐
  │ Observes traffic    │            │ Sends attack        │
  │ only — no attack   │            │ payloads to app     │
  │ payloads sent       │            │                      │
  │                      │            │ May cause:           │
  │ Safe for production │            │ - Data modification  │
  │                      │            │ - Session issues     │
  │ Finds:               │            │ - Performance impact │
  │ - Missing headers   │            │                      │
  │ - Cookie flags      │            │ Use on:              │
  │ - Info disclosure   │            │ - Staging/QA only    │
  │ - SSL issues        │            │ - Never production   │
  └────────────────────┘            └────────────────────┘

  AUTHENTICATED SCAN                 UNAUTHENTICATED SCAN
  ┌────────────────────┐            ┌────────────────────┐
  │ Scanner logs in     │            │ Tests only public   │
  │ with valid creds    │            │ endpoints            │
  │                      │            │                      │
  │ Tests:               │            │ Finds:               │
  │ - Post-login pages  │            │ - Login page vulns  │
  │ - IDOR across roles │            │ - Public API issues │
  │ - Privilege issues  │            │ - Auth bypass        │
  │                      │            │                      │
  │ Coverage: 80-90%    │            │ Coverage: 20-30%     │
  └────────────────────┘            └────────────────────┘
```

---

## DAST in the Pipeline

```
WHERE DAST FITS:

  Code → Build → SAST → Deploy to Staging → DAST → Deploy to Prod
                                               │
                         ┌─────────────────────┤
                         │                     │
                    ┌────┴─────┐         ┌────┴─────┐
                    │ Passive  │         │ Active   │
                    │ Scan     │         │ Scan     │
                    │ (5 min)  │         │ (1-4 hr) │
                    └──────────┘         └──────────┘
                         │                     │
                         ▼                     ▼
                    PR Feedback           Nightly/Weekly
                    Quick gates           Full assessment

  TIMING STRATEGY:
  ┌──────────────┬────────────────────┬──────────────────┐
  │ Trigger       │ Scan Type          │ Duration          │
  ├──────────────┼────────────────────┼──────────────────┤
  │ Every PR     │ Passive only       │ 2-5 minutes       │
  │ Merge to main│ Targeted active    │ 15-30 minutes     │
  │ Nightly      │ Full active scan   │ 2-8 hours         │
  │ Weekly       │ Full + auth scan   │ 4-12 hours        │
  └──────────────┴────────────────────┴──────────────────┘
```

---

## Real-World Scenario: DAST Catches What SAST Missed

```
SCENARIO: E-commerce Application

  SAST scan: 0 findings (clean code using framework defaults)

  DAST scan finds:
  ┌─────────────────────────────────────────────────────────┐
  │ 1. Missing CSP header                                    │
  │    → SAST can't check HTTP headers                       │
  │                                                          │
  │ 2. Session cookie without Secure flag                    │
  │    → Server configuration, not in app code               │
  │                                                          │
  │ 3. IDOR: GET /api/orders/123 returns other user's data  │
  │    → Business logic flaw, not detectable in code alone   │
  │                                                          │
  │ 4. Admin panel exposed at /admin (default Django admin) │
  │    → Deployment misconfiguration                         │
  │                                                          │
  │ 5. TLS 1.0 still enabled on server                      │
  │    → Infrastructure config, outside application code     │
  └─────────────────────────────────────────────────────────┘

  LESSON: SAST + DAST together provide comprehensive coverage
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Scanner can't find pages | Modern SPA (React/Angular) | Use browser-based crawling (AJAX Spider) instead of traditional spider |
| Low coverage (< 30%) | Unauthenticated scan | Configure authentication scripts for logged-in scanning |
| Scan takes too long | Scanning everything at max depth | Limit scope, reduce spider depth, exclude non-critical paths |
| Too many false positives | Generic scan policy | Customize scan policy; disable checks irrelevant to your stack |
| Scanner crashes/hangs | Memory exhaustion or loops | Set max scan duration, limit concurrent threads, exclude infinite scroll endpoints |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **DAST** | Dynamic testing of running applications (black-box) |
| **Crawling** | Spider phase discovering all application endpoints |
| **Active Scan** | Sending attack payloads to test for vulnerabilities |
| **Passive Scan** | Observing responses without sending attack payloads |
| **Authenticated Scan** | Scanner logs in to test post-authentication pages |
| **IAST** | Hybrid approach combining code instrumentation + runtime |
| **Attack Surface** | All endpoints, parameters, and inputs available to attackers |

---

## Quick Revision Questions

1. **What is the fundamental difference between SAST and DAST?**
   > SAST analyzes source code without running the application (white-box), while DAST tests the running application from the outside without source code access (black-box). SAST finds code-level flaws; DAST finds runtime and configuration vulnerabilities.

2. **What are the three phases of a DAST scan?**
   > Crawling (spider discovers endpoints and builds site map), Attack (sends malicious payloads to each endpoint), and Analysis (examines responses for vulnerability indicators like SQL errors, reflected scripts, or unauthorized access).

3. **Why should active DAST scans never run against production?**
   > Active scans send attack payloads (SQL injection, XSS) that can modify data, create accounts, delete records, or cause performance issues. They should only target staging/QA environments that mirror production.

4. **What types of vulnerabilities can DAST find that SAST cannot?**
   > Server misconfigurations (missing headers, TLS issues), session management flaws, business logic vulnerabilities (IDOR, price manipulation), deployment issues (exposed admin panels, debug mode), and infrastructure configurations.

5. **Why is authenticated scanning important?**
   > Unauthenticated scans only cover ~20-30% of the application (public pages). Authenticated scans test post-login functionality, role-based access control, IDOR between users, and privilege escalation — covering 80-90% of the attack surface.

---

[← Previous: 3.5 Fixing Vulnerabilities](../03-SAST/05-fixing-vulnerabilities.md) | [Next: 4.2 DAST Tools →](02-dast-tools.md)

[Back to Table of Contents](../README.md)
