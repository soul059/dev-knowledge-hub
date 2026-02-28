# Chapter 2.5: Developer Security Training

## Overview

Developer security training is essential for building a security-first culture. Training should be continuous, practical, and relevant to developers' daily work — not theoretical lectures that are forgotten the next day.

---

## Training Program Architecture

```
+====================================================================+
|                SECURITY TRAINING PROGRAM                             |
+====================================================================+
|                                                                      |
|  ONBOARDING (Day 1)                                                 |
|  +--------------------------------------------------------------+  |
|  | - Security policies overview                                 |  |
|  | - Secrets management tools setup                             |  |
|  | - IDE security plugins installation                          |  |
|  | - Pre-commit hooks configuration                             |  |
|  +--------------------------------------------------------------+  |
|                           |                                          |
|                           v                                          |
|  FOUNDATIONS (Month 1)                                              |
|  +--------------------------------------------------------------+  |
|  | - OWASP Top 10 workshop (4-hour hands-on)                    |  |
|  | - Secure coding in your language (Python/JS/Java/Go)         |  |
|  | - Security tools walkthrough (SAST, SCA, DAST)               |  |
|  +--------------------------------------------------------------+  |
|                           |                                          |
|                           v                                          |
|  CONTINUOUS (Ongoing)                                               |
|  +--------------------------------------------------------------+  |
|  | - Monthly security lunch & learn sessions                    |  |
|  | - Quarterly CTF competitions                                 |  |
|  | - Annual security training refresh                           |  |
|  | - Security newsletter / Slack channel                        |  |
|  +--------------------------------------------------------------+  |
|                           |                                          |
|                           v                                          |
|  ADVANCED (Security Champions)                                      |
|  +--------------------------------------------------------------+  |
|  | - Threat modeling workshops                                  |  |
|  | - Penetration testing basics                                 |  |
|  | - Incident response drills                                   |  |
|  | - Security architecture review                               |  |
|  +--------------------------------------------------------------+  |
+====================================================================+
```

---

## Training Methods

### 1. Capture the Flag (CTF)

```
CTF PLATFORM SETUP:

  +--------------------------------------------------+
  |  INTERNAL CTF CHALLENGES                          |
  +--------------------------------------------------+
  |                                                    |
  |  Challenge 1: SQL Injection                        |
  |  Points: 100 | Difficulty: Easy                    |
  |  "Login to the admin panel without credentials"    |
  |                                                    |
  |  Challenge 2: XSS                                  |
  |  Points: 200 | Difficulty: Medium                  |
  |  "Steal the admin's cookie via stored XSS"         |
  |                                                    |
  |  Challenge 3: SSRF                                 |
  |  Points: 300 | Difficulty: Hard                    |
  |  "Access the internal metadata service"            |
  |                                                    |
  |  Challenge 4: Broken Auth                          |
  |  Points: 250 | Difficulty: Medium                  |
  |  "Bypass the password reset flow"                  |
  |                                                    |
  |  Challenge 5: Deserialization                      |
  |  Points: 400 | Difficulty: Hard                    |
  |  "Achieve RCE via insecure deserialization"        |
  +--------------------------------------------------+

  RECOMMENDED PLATFORMS:
  +--------------------------------------------------+
  | - OWASP Juice Shop (Free, self-hosted)            |
  | - HackTheBox (Online, CTF challenges)             |
  | - TryHackMe (Guided learning paths)               |
  | - PortSwigger Web Security Academy (Free)         |
  | - SANS Holiday Hack Challenge (Annual)             |
  | - Secure Code Warrior (Commercial)                 |
  +--------------------------------------------------+
```

### 2. Vulnerable Application Labs

```bash
# Deploy OWASP Juice Shop for training
docker run -d -p 3000:3000 bkimminich/juice-shop

# Deploy DVWA (Damn Vulnerable Web Application)
docker run -d -p 80:80 vulnerables/web-dvwa

# Deploy WebGoat (OWASP learning platform)
docker run -d -p 8080:8080 -p 9090:9090 webgoat/webgoat
```

```
JUICE SHOP TRAINING PATH:

  Level 1 (★)           Level 2 (★★)          Level 3 (★★★)
  +----------------+    +----------------+    +----------------+
  | - DOM XSS      |    | - CSRF         |    | - SSRF         |
  | - SQL Injection|    | - Broken Auth  |    | - XXE          |
  |   (basic)      |    | - File Upload  |    | - NoSQL Inject |
  | - Exposed      |    | - JWT Flaws    |    | - Prototype    |
  |   Admin page   |    | - Brute Force  |    |   Pollution    |
  +----------------+    +----------------+    +----------------+
```

### 3. Security Code Labs

```
CODE LAB: Fixing SQL Injection

  STEP 1: Understand the vulnerability
  +----------------------------------------------------------+
  | Read: SQL Injection explanation and real-world impact     |
  | Time: 10 minutes                                          |
  +----------------------------------------------------------+
  
  STEP 2: Exploit it
  +----------------------------------------------------------+
  | Exercise: Use SQLMap to demonstrate the attack            |
  | $ sqlmap -u "http://lab.local/search?q=test" --tables     |
  | Time: 15 minutes                                          |
  +----------------------------------------------------------+
  
  STEP 3: Find it in code
  +----------------------------------------------------------+
  | Review the source code and identify the vulnerable line   |
  | File: app/routes/search.py                                |
  | Time: 10 minutes                                          |
  +----------------------------------------------------------+
  
  STEP 4: Fix it
  +----------------------------------------------------------+
  | Rewrite using parameterized queries                       |
  | Write a unit test to verify the fix                       |
  | Time: 20 minutes                                          |
  +----------------------------------------------------------+
  
  STEP 5: Verify
  +----------------------------------------------------------+
  | Re-run SQLMap to confirm the fix works                    |
  | Run SAST scan to verify clean results                     |
  | Time: 10 minutes                                          |
  +----------------------------------------------------------+
```

---

## Training Curriculum by Role

```
+================================================================+
|  ROLE-BASED TRAINING MATRIX                                     |
+================================================================+
|                                                                  |
|  Role              | Required Courses        | Hours/Year       |
|  ------------------+-------------------------+------------------+
|  All Developers    | OWASP Top 10            | 8                |
|                    | Secure Coding Basics     |                  |
|                    | Secrets Management       |                  |
|  ------------------+-------------------------+------------------+
|  Frontend Dev      | + XSS Prevention         | 12               |
|                    | + CSP Configuration      |                  |
|                    | + Client-side Security   |                  |
|  ------------------+-------------------------+------------------+
|  Backend Dev       | + SQL Injection          | 12               |
|                    | + API Security           |                  |
|                    | + Auth/Authz Patterns    |                  |
|  ------------------+-------------------------+------------------+
|  DevOps/SRE        | + Container Security     | 16               |
|                    | + K8s Security           |                  |
|                    | + IaC Security           |                  |
|                    | + Network Security       |                  |
|  ------------------+-------------------------+------------------+
|  Security Champion | + Threat Modeling        | 24               |
|                    | + Pen Testing Basics     |                  |
|                    | + Incident Response      |                  |
|                    | + Security Architecture  |                  |
|  ------------------+-------------------------+------------------+
|  Tech Lead / Mgr   | + Risk Assessment       | 8                |
|                    | + Security Governance    |                  |
|                    | + Compliance Overview    |                  |
+================================================================+
```

---

## Measuring Training Effectiveness

```
TRAINING EFFECTIVENESS METRICS:

  LEADING INDICATORS (Predictive):
  +----------------------------------------------------------+
  | Training completion rate:        Target: 100%             |
  | CTF participation rate:          Target: > 80%            |
  | Security champion coverage:     Target: 100% of teams    |
  | Training satisfaction score:     Target: > 4.0/5.0        |
  +----------------------------------------------------------+

  LAGGING INDICATORS (Results):
  +----------------------------------------------------------+
  | Vulnerabilities per release:     Target: Decreasing       |
  | MTTR (Mean Time to Remediate):   Target: Decreasing       |
  | Security bugs found by developers: Target: Increasing     |
  | Production security incidents:   Target: Decreasing       |
  +----------------------------------------------------------+

  BEFORE vs AFTER TRAINING:
  
  Vulns per release     Developer-found bugs
       |                     |
   20  |  *                  |                    *
   15  |     *          5    |               *
   10  |        *       4    |          *
    5  |           *    3    |     *
    0  |              * 2    |  *
       +---+---+---+-- 1    +---+---+---+---
        Q1  Q2  Q3  Q4       Q1  Q2  Q3  Q4
       
       (Going DOWN = Good)   (Going UP = Good)
```

---

## Security Knowledge Base

### Internal Security Wiki Structure

```
SECURITY KNOWLEDGE BASE:

  /security-wiki
  ├── getting-started/
  │   ├── onboarding-checklist.md
  │   ├── tool-setup-guide.md
  │   └── first-week-security.md
  │
  ├── secure-coding/
  │   ├── python-guidelines.md
  │   ├── javascript-guidelines.md
  │   ├── java-guidelines.md
  │   ├── go-guidelines.md
  │   └── common-pitfalls.md
  │
  ├── vulnerability-guides/
  │   ├── sql-injection.md
  │   ├── xss-prevention.md
  │   ├── csrf-protection.md
  │   ├── ssrf-mitigation.md
  │   └── auth-best-practices.md
  │
  ├── tools/
  │   ├── sast-setup.md
  │   ├── dast-runbook.md
  │   ├── sca-configuration.md
  │   └── secrets-management.md
  │
  ├── incident-response/
  │   ├── playbooks/
  │   ├── post-mortems/
  │   └── contacts.md
  │
  └── compliance/
      ├── soc2-requirements.md
      ├── gdpr-checklist.md
      └── pci-dss-guide.md
```

---

## Real-World Scenario: Google's Security Training

```
GOOGLE'S APPROACH:

  +------------------------------------------------------------+
  | 1. SECURITY-FOCUSED CODE REVIEWS                           |
  |    Every code change requires security-aware reviewers      |
  +------------------------------------------------------------+
  | 2. BUG BOUNTY PROGRAM                                      |
  |    External researchers find bugs Google pays for           |
  +------------------------------------------------------------+
  | 3. VULNERABILITY REWARD PROGRAM                            |
  |    Incentivizes internal finding and reporting              |
  +------------------------------------------------------------+
  | 4. SECURITY BOOTCAMPS                                      |
  |    Multi-day intensive training for new engineers           |
  +------------------------------------------------------------+
  | 5. FUZZING INFRASTRUCTURE                                  |
  |    ClusterFuzz / OSS-Fuzz for continuous fuzzing            |
  +------------------------------------------------------------+
  | 6. SECURITY KEYS FOR ALL EMPLOYEES                        |
  |    Hardware 2FA eliminated phishing attacks                 |
  +------------------------------------------------------------+
  
  RESULT: Zero successful phishing attacks on employees
          since mandatory hardware security keys
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Low training attendance | Mandatory training feels like burden | Make it hands-on, gamified, and fun (CTF) |
| Knowledge not applied | Theory-only training | Use real codebase examples, pair with SAST findings |
| Training quickly outdated | Security evolves fast | Quarterly updates, subscribe to threat intel feeds |
| Different skill levels | One-size-fits-all | Role-based and level-based training tracks |
| No time allocated | Competing priorities | Get management buy-in, dedicate % of sprint capacity |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Onboarding Security** | Security tools and policies from Day 1 |
| **CTF Competitions** | Hands-on learning through challenges |
| **Vulnerable Labs** | Practice environments (Juice Shop, WebGoat, DVWA) |
| **Code Labs** | Step-by-step exercises with real vulnerabilities |
| **Role-Based Training** | Different tracks for frontend, backend, DevOps, leads |
| **Security Knowledge Base** | Central wiki with guidelines, playbooks, and references |
| **Effectiveness Metrics** | Track vulns/release, MTTR, developer-found bugs |
| **Continuous Learning** | Monthly sessions, newsletters, and quarterly CTFs |

---

## Quick Revision Questions

1. **Why is hands-on training (CTF, labs) more effective than lecture-based security training?**
   > Developers learn by doing. Hands-on training lets them experience attacks firsthand, understand impact, and practice fixes in a safe environment. Retention rates are significantly higher with experiential learning.

2. **Name three platforms for practicing application security.**
   > OWASP Juice Shop (self-hosted, comprehensive), PortSwigger Web Security Academy (free, browser-based), and TryHackMe (guided paths). Others: HackTheBox, WebGoat, DVWA.

3. **How do you measure whether security training is actually reducing vulnerabilities?**
   > Track lagging indicators: vulnerabilities per release (should decrease), MTTR (should decrease), production incidents (should decrease), and developer-found security bugs (should increase over time).

4. **What security training should be included in developer onboarding?**
   > Security policies overview, secrets management tool setup, IDE security plugin installation, pre-commit hooks configuration, and introduction to the security knowledge base.

5. **Why is role-based security training important?**
   > Different roles face different threats. Frontend devs need XSS/CSP training, backend devs need SQLi/auth training, DevOps needs container/IaC security. Generic training misses role-specific attack surfaces.

6. **How did Google eliminate employee phishing attacks?**
   > By mandating hardware security keys (FIDO2/WebAuthn) for all employees. Hardware keys are phishing-resistant because they verify the domain cryptographically, unlike OTP codes that can be intercepted.

---

[← Previous: 2.4 Code Review for Security](04-code-review-for-security.md) | [Next: Unit 3 - 3.1 SAST Concepts →](../03-SAST/01-sast-concepts.md)

[Back to Table of Contents](../README.md)
