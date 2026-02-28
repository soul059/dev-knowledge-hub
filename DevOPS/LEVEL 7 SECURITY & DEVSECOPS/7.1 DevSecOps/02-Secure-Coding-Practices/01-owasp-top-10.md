# Chapter 2.1: OWASP Top 10

## Overview

The **OWASP (Open Web Application Security Project) Top 10** is the most widely recognized standard for web application security risks. Updated periodically, it represents a consensus on the most critical security risks to web applications. Every developer and security professional should understand these risks.

---

## OWASP Top 10 (2021 Edition)

```
+====================================================================+
|                      OWASP TOP 10 (2021)                            |
+====================================================================+
|                                                                      |
|  +---+  A01: Broken Access Control              [Previously #5]    |
|  | 1 |  94% of apps tested had some form of broken access control  |
|  +---+                                                              |
|                                                                      |
|  +---+  A02: Cryptographic Failures              [Previously #3]   |
|  | 2 |  Formerly "Sensitive Data Exposure"                         |
|  +---+                                                              |
|                                                                      |
|  +---+  A03: Injection                           [Previously #1]   |
|  | 3 |  SQL, NoSQL, OS, LDAP injection attacks                     |
|  +---+                                                              |
|                                                                      |
|  +---+  A04: Insecure Design                     [NEW in 2021]     |
|  | 4 |  Flaws in design and architecture                           |
|  +---+                                                              |
|                                                                      |
|  +---+  A05: Security Misconfiguration           [Previously #6]   |
|  | 5 |  Default configs, incomplete setup, open cloud storage      |
|  +---+                                                              |
|                                                                      |
|  +---+  A06: Vulnerable & Outdated Components    [Previously #9]   |
|  | 6 |  Using components with known vulnerabilities                |
|  +---+                                                              |
|                                                                      |
|  +---+  A07: Identification & Auth Failures      [Previously #2]   |
|  | 7 |  Broken authentication, session management                  |
|  +---+                                                              |
|                                                                      |
|  +---+  A08: Software & Data Integrity Failures  [NEW in 2021]     |
|  | 8 |  CI/CD pipeline integrity, unsigned updates                 |
|  +---+                                                              |
|                                                                      |
|  +---+  A09: Security Logging & Monitoring Failures [Previously #10]|
|  | 9 |  Insufficient logging, detection, and response              |
|  +---+                                                              |
|                                                                      |
|  +---+  A10: Server-Side Request Forgery (SSRF)  [NEW in 2021]    |
|  |10 |  Server fetches attacker-controlled URLs                    |
|  +---+                                                              |
+====================================================================+
```

---

## A01: Broken Access Control

The #1 most common vulnerability. Occurs when users can act outside their intended permissions.

```
ATTACK SCENARIO:

  Normal User (Role: viewer)
       |
       | GET /api/users/123/profile    <-- Own profile (OK)
       |
       | GET /api/users/456/profile    <-- Other user's profile!
       |                                    (IDOR - Insecure Direct
       |                                     Object Reference)
       |
       | POST /api/admin/delete-user   <-- Admin function!
       |                                    (Privilege escalation)
       |
       | GET /api/users/123/profile?role=admin  <-- Parameter tampering!

  +------------------------------------------------------------+
  |  VULNERABLE CODE (Python/Flask):                            |
  +------------------------------------------------------------+
  |  @app.route('/api/users/<user_id>/profile')                |
  |  def get_profile(user_id):                                  |
  |      # NO authorization check!                              |
  |      user = db.get_user(user_id)                           |
  |      return jsonify(user.profile)                           |
  +------------------------------------------------------------+

  +------------------------------------------------------------+
  |  SECURE CODE:                                               |
  +------------------------------------------------------------+
  |  @app.route('/api/users/<user_id>/profile')                |
  |  @login_required                                            |
  |  def get_profile(user_id):                                  |
  |      if current_user.id != user_id and                      |
  |         not current_user.has_role('admin'):                  |
  |          abort(403)  # Forbidden                            |
  |      user = db.get_user(user_id)                           |
  |      return jsonify(user.profile)                           |
  +------------------------------------------------------------+
```

**Preventions**: Deny by default, implement RBAC, validate object-level permissions, disable directory listing.

---

## A02: Cryptographic Failures

Failure to properly protect sensitive data (in transit and at rest).

```
COMMON FAILURES:

  +------------------------------------------------------------+
  | - Transmitting data in cleartext (HTTP instead of HTTPS)   |
  | - Using weak algorithms (MD5, SHA1 for passwords)          |
  | - Using default or weak encryption keys                     |
  | - Not enforcing encryption                                 |
  | - Improper certificate validation                          |
  +------------------------------------------------------------+

  VULNERABLE:                          SECURE:
  +---------------------------+        +---------------------------+
  | password = md5(input)     |        | password = bcrypt.hash(   |
  |                           |        |   input, rounds=12)       |
  +---------------------------+        +---------------------------+
  | conn_string = "http://    |        | conn_string = "https://   |
  |   api.example.com"        |        |   api.example.com"        |
  +---------------------------+        +---------------------------+
  | key = "hardcoded_key_123" |        | key = vault.get_secret(   |
  |                           |        |   "encryption_key")       |
  +---------------------------+        +---------------------------+
```

---

## A03: Injection

Untrusted data sent to an interpreter as part of a command or query.

```
SQL INJECTION EXAMPLE:

  User Input: ' OR '1'='1' --

  VULNERABLE QUERY:
  +----------------------------------------------------------+
  | SELECT * FROM users                                       |
  | WHERE username = '' OR '1'='1' --' AND password = '...'  |
  |                    ^^^^^^^^^^^^                            |
  |                    Always TRUE! Returns ALL users          |
  +----------------------------------------------------------+

  PARAMETERIZED QUERY (SAFE):
  +----------------------------------------------------------+
  | cursor.execute(                                           |
  |   "SELECT * FROM users WHERE username = ? AND pwd = ?",  |
  |   (username, password)                                    |
  | )                                                         |
  +----------------------------------------------------------+
```

---

## A04: Insecure Design

Design-level flaws that cannot be fixed by perfect implementation.

```
EXAMPLE: Password Reset Flow

  INSECURE DESIGN:
  +------------------------------------------------------------+
  |  1. User clicks "Forgot Password"                          |
  |  2. System asks: "What is your pet's name?"                |
  |  3. User answers correctly                                  |
  |  4. System shows new password on screen                     |
  |                                                              |
  |  Problems:                                                   |
  |  - Security questions are guessable (social media)          |
  |  - Password displayed in browser (shoulder surfing)          |
  |  - No rate limiting on guesses                               |
  +------------------------------------------------------------+

  SECURE DESIGN:
  +------------------------------------------------------------+
  |  1. User clicks "Forgot Password"                          |
  |  2. System sends time-limited token to registered email     |
  |  3. User clicks link, enters new password                   |
  |  4. Token expires after 15 minutes / single use             |
  |  5. Rate limited to 3 requests per hour                     |
  |  6. Old sessions invalidated                                |
  +------------------------------------------------------------+
```

---

## A05: Security Misconfiguration

```
COMMON MISCONFIGURATIONS:

  +------+--------------------------------------------------+
  | Area | Examples                                          |
  +------+--------------------------------------------------+
  | App  | Debug mode enabled in production                  |
  |      | Default admin credentials (admin/admin)           |
  |      | Verbose error messages showing stack traces        |
  +------+--------------------------------------------------+
  | Web  | Directory listing enabled                         |
  |      | Unnecessary HTTP methods (PUT, DELETE)             |
  |      | Missing security headers                          |
  +------+--------------------------------------------------+
  |Cloud | S3 buckets public by default                      |
  |      | Overly permissive security groups (0.0.0.0/0)     |
  |      | Unused ports open                                 |
  +------+--------------------------------------------------+
  | DB   | Default ports exposed to internet                 |
  |      | No authentication required                        |
  |      | Excessive privileges for app accounts             |
  +------+--------------------------------------------------+
```

**Secure Headers Example:**
```
# nginx.conf security headers
add_header X-Frame-Options "DENY" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self'" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
```

---

## A06: Vulnerable & Outdated Components

```
SUPPLY CHAIN RISK:

  Your Application
       |
       +-- Express.js v4.17.1
       |      |
       |      +-- body-parser v1.19.0
       |      +-- cookie v0.4.1
       |      +-- qs v6.7.0 (VULNERABLE! CVE-2022-24999)
       |
       +-- lodash v4.17.15 (VULNERABLE! CVE-2021-23337)
       |
       +-- jsonwebtoken v8.5.1
              |
              +-- jws v3.2.2
              +-- ms v2.1.2

  Even if YOUR code is perfect, vulnerable dependencies
  create attack surface.

  DETECTION:
  $ npm audit
  $ snyk test
  $ pip audit
  $ mvn dependency-check:check
```

---

## A07: Identification & Authentication Failures

```
  ATTACK TYPES:
  
  1. CREDENTIAL STUFFING
  +------------------------------------------------------------+
  |  Attacker has list of breached credentials                  |
  |  Tries them all against your login                          |
  |  Many users reuse passwords across sites                    |
  +------------------------------------------------------------+
  
  2. BRUTE FORCE
  +------------------------------------------------------------+
  |  Attacker tries all possible passwords                      |
  |  No rate limiting = unlimited attempts                      |
  +------------------------------------------------------------+
  
  3. SESSION HIJACKING
  +------------------------------------------------------------+
  |  Session ID in URL: example.com?session=abc123              |
  |  Session ID not rotated after login                         |
  |  Session doesn't expire                                     |
  +------------------------------------------------------------+

  PREVENTIONS:
  +------------------------------------------------------------+
  | [x] Multi-factor authentication (MFA)                      |
  | [x] Rate limiting (e.g., 5 attempts per minute)            |
  | [x] Account lockout after N failures                       |
  | [x] Bcrypt/Argon2 for password hashing                     |
  | [x] Session timeout and rotation                           |
  | [x] Password complexity requirements                       |
  | [x] Check against breached password databases              |
  +------------------------------------------------------------+
```

---

## A08: Software & Data Integrity Failures

```
  EXAMPLES:
  
  1. UNSIGNED UPDATES
  +------------------------------------------------------------+
  |  Application auto-updates without verifying signature       |
  |  Attacker MITM's the update = malware installed             |
  +------------------------------------------------------------+
  
  2. CI/CD PIPELINE INTEGRITY (SolarWinds-type)
  +------------------------------------------------------------+
  |  Pipeline modified to inject malicious code                 |
  |  Build artifacts not signed or verified                     |
  +------------------------------------------------------------+
  
  3. INSECURE DESERIALIZATION
  +------------------------------------------------------------+
  |  Untrusted data deserialized without validation             |
  |  Can lead to Remote Code Execution (RCE)                    |
  +------------------------------------------------------------+
  
  PREVENTIONS:
  +------------------------------------------------------------+
  | [x] Use digital signatures for software updates             |
  | [x] SLSA framework for build integrity                     |
  | [x] SCA tools to verify dependency integrity               |
  | [x] Avoid deserializing untrusted data                     |
  | [x] Use integrity checks (checksums, SBOM)                 |
  +------------------------------------------------------------+
```

---

## A09: Security Logging & Monitoring Failures

```
  WHAT TO LOG:
  +------------------------------------------------------------+
  | [x] Login attempts (success and failure)                   |
  | [x] Access control failures (403s)                         |
  | [x] Input validation failures                              |
  | [x] Server-side errors (500s)                              |
  | [x] Admin activities                                       |
  | [x] Data access and modifications                          |
  +------------------------------------------------------------+
  
  WHAT NOT TO LOG:
  +------------------------------------------------------------+
  | [x] Passwords (even hashed)                                |
  | [x] Credit card numbers                                    |
  | [x] Social security numbers                                |
  | [x] Personal health information                            |
  | [x] Session tokens                                         |
  +------------------------------------------------------------+
```

---

## A10: Server-Side Request Forgery (SSRF)

```
  ATTACK FLOW:

  Attacker                    Web Server                Internal Network
     |                            |                           |
     | POST /api/fetch-url        |                           |
     | url=http://169.254.169.254 |                           |
     |  (AWS metadata endpoint)   |                           |
     |--------------------------->|                           |
     |                            |  GET /latest/meta-data/   |
     |                            |  iam/security-credentials |
     |                            |-------------------------->|
     |                            |                           |
     |                            |  { AccessKeyId: "AKIA..", |
     |                            |    SecretAccessKey: "..."} |
     |                            |<--------------------------|
     |                            |                           |
     | { AWS credentials leaked } |                           |
     |<---------------------------|                           |

  PREVENTIONS:
  +------------------------------------------------------------+
  | [x] Whitelist allowed URLs/domains                         |
  | [x] Block requests to private IP ranges                   |
  | [x] Use IMDSv2 (requires token for metadata access)       |
  | [x] Don't expose raw HTTP client to user input            |
  | [x] Network segmentation                                   |
  +------------------------------------------------------------+
```

---

## OWASP Top 10 Quick Reference

| # | Risk | CWE | Key Prevention |
|---|------|-----|----------------|
| A01 | Broken Access Control | CWE-284 | RBAC, deny by default, object-level checks |
| A02 | Cryptographic Failures | CWE-311 | Strong algorithms, key management, TLS |
| A03 | Injection | CWE-89 | Parameterized queries, input validation |
| A04 | Insecure Design | CWE-209 | Threat modeling, secure design patterns |
| A05 | Security Misconfiguration | CWE-16 | Automated config scanning, hardening |
| A06 | Vulnerable Components | CWE-1035 | SCA tools, dependency updates |
| A07 | Auth Failures | CWE-287 | MFA, rate limiting, secure sessions |
| A08 | Integrity Failures | CWE-502 | Signing, SLSA, integrity checks |
| A09 | Logging Failures | CWE-778 | Centralized logging, alerting, SIEM |
| A10 | SSRF | CWE-918 | URL whitelist, block private IPs |

---

## Quick Revision Questions

1. **What is the OWASP Top 10 and why is it important?**
   > It's a standard awareness document listing the 10 most critical web application security risks, updated periodically by OWASP. It's important because it provides a consensus baseline for security testing and developer training.

2. **Which vulnerability moved from #1 (2017) to #3 (2021), and why?**
   > Injection. It moved down because frameworks with built-in parameterized queries became more common, and Broken Access Control (now #1) was found in 94% of tested applications.

3. **What is the difference between A04 (Insecure Design) and A05 (Security Misconfiguration)?**
   > Insecure Design is a flaw in the architecture/design that cannot be fixed by correct implementation. Security Misconfiguration is a correctly designed system that was improperly configured — it can be fixed by changing settings.

4. **How does SSRF (A10) differ from other injection attacks?**
   > SSRF tricks the server into making requests to internal resources on behalf of the attacker. Unlike SQL injection which targets databases, SSRF targets the server's network access to reach internal services, cloud metadata, or private networks.

5. **Name three new entries in the OWASP Top 10 (2021) compared to 2017.**
   > (1) A04 Insecure Design, (2) A08 Software & Data Integrity Failures, (3) A10 Server-Side Request Forgery (SSRF).

6. **What type of hashing algorithm should be used for passwords, and why?**
   > Adaptive hashing algorithms like bcrypt, Argon2, or scrypt. They are intentionally slow and use salting, making brute-force and rainbow table attacks impractical. Never use MD5 or SHA-1/SHA-256 for passwords.

---

[← Previous: 1.5 Shared Responsibility Model](../01-DevSecOps-Introduction/05-shared-responsibility-model.md) | [Next: 2.2 Common Vulnerabilities →](02-common-vulnerabilities.md)

[Back to Table of Contents](../README.md)
