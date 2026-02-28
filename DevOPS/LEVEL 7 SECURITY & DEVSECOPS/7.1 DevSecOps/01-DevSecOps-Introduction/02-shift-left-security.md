# Chapter 1.2: Shift-Left Security

## Overview

"Shift Left" means moving security activities earlier in the software development lifecycle. Instead of testing for security only before deployment, you test as early as possible — ideally at the moment code is written.

---

## The Shift-Left Concept

### Traditional (Right-Heavy) Approach

```
   Security Effort
        ^
        |                                               ****
        |                                          ****
        |                                     ****
        |                                **** 
        |                           ****
        |
        |
        |  .  .  .  .  .  .  .                          
        +----+-----+-----+-----+-----+-----+-----+----->
           Plan   Code  Build  Test  Stage  Prod  Maint
           
   Problem: All security effort concentrated at the END
```

### Shift-Left Approach

```
   Security Effort
        ^
        |  ****
        |      ****
        |          ****
        |              ****
        |                  ****
        |                      ****
        |                          ****
        |                              ****
        +----+-----+-----+-----+-----+-----+-----+----->
           Plan   Code  Build  Test  Stage  Prod  Maint
           
   Goal: Security effort starts EARLY and stays continuous
```

---

## Security Activities at Each Stage

```
+------------------------------------------------------------------+
|                    SHIFT-LEFT SECURITY MAP                         |
+------------------------------------------------------------------+
|                                                                    |
|  PLANNING PHASE                                                    |
|  +------------------------------------------------------------+   |
|  | - Threat Modeling (STRIDE, DREAD)                          |   |
|  | - Security Requirements Definition                         |   |
|  | - Risk Assessment                                           |   |
|  | - Security User Stories                                     |   |
|  | - Architecture Security Review                              |   |
|  +------------------------------------------------------------+   |
|                          |                                         |
|                          v                                         |
|  CODING PHASE                                                      |
|  +------------------------------------------------------------+   |
|  | - IDE Security Plugins (real-time feedback)                |   |
|  | - Pre-commit Hooks (secret scanning, linting)              |   |
|  | - Pair Programming / Secure Code Review                     |   |
|  | - Secure Coding Standards Enforcement                       |   |
|  +------------------------------------------------------------+   |
|                          |                                         |
|                          v                                         |
|  BUILD PHASE                                                       |
|  +------------------------------------------------------------+   |
|  | - SAST (Static Analysis)                                   |   |
|  | - SCA (Dependency Scanning)                                |   |
|  | - Container Image Scanning                                  |   |
|  | - IaC Security Scanning                                     |   |
|  | - License Compliance Check                                  |   |
|  +------------------------------------------------------------+   |
|                          |                                         |
|                          v                                         |
|  TESTING PHASE                                                     |
|  +------------------------------------------------------------+   |
|  | - DAST (Dynamic Analysis)                                  |   |
|  | - IAST (Interactive Analysis)                              |   |
|  | - Security Unit Tests                                       |   |
|  | - Fuzz Testing                                              |   |
|  | - API Security Testing                                      |   |
|  +------------------------------------------------------------+   |
|                          |                                         |
|                          v                                         |
|  DEPLOYMENT & OPERATIONS                                           |
|  +------------------------------------------------------------+   |
|  | - Configuration Validation                                  |   |
|  | - Runtime Protection (RASP)                                |   |
|  | - Continuous Monitoring (SIEM)                              |   |
|  | - Vulnerability Management                                  |   |
|  | - Incident Response                                         |   |
|  +------------------------------------------------------------+   |
+------------------------------------------------------------------+
```

---

## Implementing Shift-Left in Practice

### 1. IDE-Level Security (Earliest Possible)

Developers get security feedback as they type code:

```
Developer's IDE
+--------------------------------------------------+
|  editor.py                                        |
|  ------------------------------------------------|
|  1  import sqlite3                                |
|  2                                                |
|  3  def get_user(username):                       |
|  4      conn = sqlite3.connect('db.sqlite')       |
|  5      cursor = conn.cursor()                    |
|  6      query = "SELECT * FROM users WHERE        |
|  7               name = '" + username + "'"       |
|  8  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~              |
|  |  [!] WARNING: SQL Injection vulnerability      |
|  |      detected. Use parameterized queries.      |
|  |      Severity: HIGH | CWE-89                   |
|  |                                                |
|  |      Fix: cursor.execute(                      |
|  |        "SELECT * FROM users WHERE name = ?",   |
|  |        (username,))                             |
|  +------------------------------------------------|
+--------------------------------------------------+
```

**Popular IDE Security Plugins**:
- **SonarLint** — Real-time SAST in IDE
- **Snyk** — Vulnerability scanning for dependencies
- **Semgrep** — Custom rule-based scanning
- **Checkov** — IaC security scanning

### 2. Pre-Commit Hooks

Run security checks before code is even committed:

```bash
# .pre-commit-config.yaml
repos:
  # Secret Detection
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']

  # Security Linting
  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.5
    hooks:
      - id: bandit
        args: ['-r', '--severity-level', 'medium']

  # Dockerfile Linting
  - repo: https://github.com/hadolint/hadolint
    rev: v2.12.0
    hooks:
      - id: hadolint

  # Terraform Security
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.83.0
    hooks:
      - id: terraform_tfsec
      - id: terraform_checkov
```

```bash
# Install pre-commit hooks
pip install pre-commit
pre-commit install

# Now every commit triggers security checks:
$ git commit -m "Add user endpoint"
Detect Secrets.......................................................Passed
Bandit...............................................................Failed
- hook id: bandit
  B105: Possible hardcoded password: 'admin123'
  Severity: LOW  Confidence: MEDIUM
  Location: ./config.py:12
```

### 3. Security in CI Pipeline

```yaml
# .github/workflows/security.yml
name: Security Pipeline

on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      # SAST - Static Analysis
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/owasp-top-ten

      # SCA - Dependency Check
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

      # Secret Scanning
      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # Container Scanning
      - name: Run Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:latest'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'

      # IaC Scanning
      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: terraform/
          framework: terraform
```

---

## Threat Modeling (Shift-Left at Planning)

### STRIDE Model

```
+---------------------+------------------------------------------+
|  Threat Category    |  Description                              |
+---------------------+------------------------------------------+
| (S) Spoofing        | Impersonating a user or system            |
| (T) Tampering       | Modifying data or code                    |
| (R) Repudiation     | Denying an action occurred                |
| (I) Info Disclosure | Exposing sensitive information            |
| (D) Denial of Svc   | Making service unavailable               |
| (E) Elevation       | Gaining unauthorized privileges           |
+---------------------+------------------------------------------+
```

### Example: Threat Model for a Web Application

```
                    +----------------+
                    |   Internet     |
                    +-------+--------+
                            |
                   [HTTPS / TLS 1.3]
                            |
                    +-------+--------+
                    |   WAF / CDN    |  <-- DDoS Protection (D)
                    +-------+--------+
                            |
                    +-------+--------+
                    |  Load Balancer |
                    +-------+--------+
                            |
                +-----------+-----------+
                |                       |
        +-------+--------+    +--------+-------+
        |  Web Server    |    |  API Server    |
        |  (Frontend)    |    |  (Backend)     |
        +-------+--------+    +--------+-------+
                |                       |
                |    +------------------+
                |    |
        +-------+--------+
        |   Database     |  <-- SQL Injection (T)
        |   (Encrypted)  |  <-- Data Exposure (I)
        +----------------+

  THREATS IDENTIFIED:
  +--+------------------+-------------------+------------------+
  | # | Threat           | Category          | Mitigation       |
  +--+------------------+-------------------+------------------+
  | 1 | XSS Attack       | Tampering         | Input validation |
  | 2 | SQL Injection    | Tampering         | Parameterized Q  |
  | 3 | Credential Theft | Spoofing          | MFA, rate limit  |
  | 4 | Data Leak        | Info Disclosure   | Encryption, ACL  |
  | 5 | DDoS             | Denial of Service | WAF, CDN, limits |
  | 6 | Priv Escalation  | Elevation         | RBAC, least priv |
  +--+------------------+-------------------+------------------+
```

---

## Security User Stories

Shift-left means adding security requirements during sprint planning:

```
TRADITIONAL USER STORY:
  "As a user, I want to log in so I can access my account."

SECURITY-ENHANCED USER STORIES:
  "As a user, I want to log in with MFA so my account is protected 
   even if my password is compromised."
  
  "As a system, I should lock an account after 5 failed login 
   attempts to prevent brute force attacks."
  
  "As a system, I should log all authentication attempts for 
   audit and anomaly detection."
  
  "As a user, I want my session to expire after 30 minutes of 
   inactivity to protect against session hijacking."
```

---

## Real-World Scenario: Implementing Shift-Left

### Before Shift-Left

```
Timeline: Feature Development (4 weeks)

Week 1-2: Development
  |================================|
  
Week 3: Testing  
          |================|
          
Week 4: Security Review (1 week before release)
                   |========|
                          ^
                          |
                   "Found 47 critical 
                    vulnerabilities!
                    Release DELAYED 
                    by 3 weeks."
```

### After Shift-Left

```
Timeline: Feature Development (4 weeks)

Week 1: Plan + Code
  |  Threat Model  |  IDE Plugins  |  Pre-commit hooks  |
  |  (2 vulns      |  (5 vulns     |  (3 secrets        |
  |   prevented)   |   caught)     |   blocked)         |
  
Week 2: Code + Build
  |  Code Review   |  SAST Scan    |  SCA Scan          |
  |  (3 vulns      |  (4 vulns     |  (2 vulnerable     |
  |   caught)      |   found)      |   deps found)      |
  
Week 3: Test
  |  DAST Scan     |  Security     |  Fuzz Testing      |
  |  (1 vuln       |  Unit Tests   |  (0 vulns)         |
  |   found)       |  (all pass)   |                    |
  
Week 4: Deploy
  |  IaC Scan      |  Config       |  RELEASE ON TIME!  |
  |  (1 misconfig  |  Validation   |                    |
  |   fixed)       |  (pass)       |                    |

  Total: 21 issues found and fixed incrementally
         0 release delays
```

---

## Cost of Fixing Vulnerabilities

| Stage Found | Relative Cost | Example ($) | Effort |
|-------------|--------------|-------------|--------|
| Requirements/Design | 1x | $100 | Hours |
| Coding (IDE) | 2x | $200 | Hours |
| Build (SAST/SCA) | 5x | $500 | Hours-Days |
| Testing (DAST) | 15x | $1,500 | Days |
| Staging/Pre-prod | 30x | $3,000 | Days-Weeks |
| Production | 100x | $10,000 | Weeks |
| Post-breach | 1000x+ | $100,000+ | Months |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Pre-commit hooks too slow | Scanning entire codebase | Use incremental scanning (only changed files) |
| IDE plugins flooding warnings | Tool sensitivity too high | Tune severity thresholds, start with CRITICAL/HIGH |
| Developers bypassing hooks | Using `--no-verify` flag | Server-side hooks in CI/CD as backup |
| Threat modeling feels abstract | No threat modeling training | Use tools like OWASP Threat Dragon, Microsoft TMT |
| Security stories not prioritized | Product team focus on features | Include security in Definition of Done |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Shift Left** | Move security testing earlier in the SDLC |
| **IDE Security** | Real-time vulnerability detection while coding |
| **Pre-commit Hooks** | Automated security checks before code is committed |
| **Threat Modeling** | Identify potential threats during planning phase |
| **STRIDE** | Framework for categorizing security threats |
| **Security Stories** | User stories that include security requirements |
| **Cost Multiplier** | Later detection = exponentially higher fix cost |
| **Incremental Scanning** | Only scan changed files for speed |

---

## Quick Revision Questions

1. **What does "Shift Left" mean in the context of DevSecOps?**
   > Moving security activities earlier in the SDLC — from the planning and coding phases — rather than leaving security testing until late stages like staging or production.

2. **Name three security activities you can perform during the coding phase.**
   > (1) IDE security plugins for real-time feedback, (2) Pre-commit hooks for secret scanning/linting, (3) Secure code review / pair programming.

3. **What is STRIDE and why is it used in shift-left security?**
   > STRIDE is a threat modeling framework (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of privilege) used during the planning phase to proactively identify and mitigate threats before any code is written.

4. **How do pre-commit hooks help with shift-left security?**
   > They run automated security checks (secret detection, linting, security scanning) locally before code is committed to the repository, catching vulnerabilities at the earliest possible point.

5. **Why is fixing a vulnerability in production ~100x more expensive than during coding?**
   > Production fixes require emergency patching, rollback procedures, incident response, potential downtime, customer notifications, compliance reporting, and may involve data breach costs.

6. **How would you handle developers who bypass pre-commit hooks using `--no-verify`?**
   > Implement server-side CI/CD pipeline security checks as a mandatory backup. Pre-commit hooks are a convenience, not the sole defense; the pipeline must also enforce security gates.

---

[← Previous: 1.1 What is DevSecOps?](01-what-is-devsecops.md) | [Next: 1.3 Security in the CI/CD Pipeline →](03-security-in-cicd.md)

[Back to Table of Contents](../README.md)
