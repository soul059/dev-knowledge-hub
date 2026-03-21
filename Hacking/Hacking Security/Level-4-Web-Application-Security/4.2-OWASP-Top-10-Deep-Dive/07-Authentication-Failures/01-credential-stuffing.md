# Unit 7: A07 - Identification and Authentication Failures — Topic 1: Credential Stuffing

## Overview

Credential stuffing is an **automated attack** that uses previously breached username/password pairs to gain unauthorized access to accounts on other services. It exploits the widespread practice of password reuse — when users use the same password across multiple sites, a breach on one site compromises their accounts everywhere. Billions of credentials are available on the dark web, making credential stuffing one of the most prevalent and impactful attack vectors.

---

## 1. How Credential Stuffing Works

```
┌─────────────────────────────────────────────────────────────────┐
│              CREDENTIAL STUFFING ATTACK FLOW                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Step 1: Attacker obtains breached credential list             │
│          └── Sources: dark web, paste sites, combo lists        │
│              Typical list: 1-100 million email:password pairs  │
│                                                                 │
│  Step 2: Attacker uses automated tools                         │
│          └── Tools: Sentry MBA, STORM, OpenBullet, custom bots │
│                                                                 │
│  Step 3: Credentials tested against target login               │
│          ┌──────────────────────────────────┐                   │
│          │ Login Request                     │                   │
│          │ email: user@example.com           │                   │
│          │ pass: Password123!                │                   │
│          └──────────┬───────────────────────┘                   │
│                     │                                           │
│          ┌──────────┴──────────┐                                │
│          │ Success (0.1-2%)    │ Failure (98-99.9%)             │
│          │ → Account takeover  │ → Try next credential         │
│          └─────────────────────┘                                │
│                                                                 │
│  Step 4: Successful logins → account takeover                  │
│          └── Financial fraud, data theft, spam, resale         │
│                                                                 │
│  Success rate: 0.1-2% (but at scale = thousands of accounts)  │
│  1 million credentials × 1% = 10,000 compromised accounts    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Scale of the Problem

| Statistic | Value |
|-----------|-------|
| Known breached credentials | 15+ billion |
| Password reuse rate | 52-65% of users |
| Average credential stuffing success rate | 0.1-2% |
| Cost per attempt | ~$0.001 (extremely cheap) |
| Credential stuffing attacks in 2023 | Billions per month |
| Largest combo lists | 3+ billion entries |

### Notable Credential Dumps

```
Collection #1-5: 2.2 billion unique email/password pairs
RockYou2021: 8.4 billion passwords (hashed)
COMB (Compilation of Many Breaches): 3.2 billion pairs
Anti Public Combo List: 562 million pairs
Exploit.in: 593 million email/password pairs
```

---

## 3. Detection Methods

```
INDICATORS OF CREDENTIAL STUFFING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  VOLUME-BASED:                                                 │
│  ├── Spike in failed login attempts                            │
│  ├── Unusual login attempt rates from single IP or subnet      │
│  ├── Many failed logins followed by success (account found!)   │
│  └── Login attempts from known proxy/VPN IP ranges             │
│                                                                 │
│  BEHAVIORAL:                                                   │
│  ├── Login attempts at non-human speed                         │
│  ├── Sequential usernames being tried                          │
│  ├── Missing or unusual User-Agent strings                     │
│  ├── No mouse/keyboard events (headless browser)               │
│  ├── Failed logins across many different accounts from same IP │
│  └── Geographic impossibility (login from 2 countries in 1hr)  │
│                                                                 │
│  TECHNICAL:                                                    │
│  ├── Missing or repeated session cookies                       │
│  ├── Consistent request timing patterns                        │
│  ├── Identical HTTP headers across attempts                    │
│  └── Known credential stuffing tool fingerprints               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Prevention Strategies

### Multi-Layered Defense

```python
# Layer 1: Rate Limiting
from flask_limiter import Limiter

limiter = Limiter(app)

@app.route('/login', methods=['POST'])
@limiter.limit("5/minute", key_func=lambda: request.json.get('email'))  # Per account
@limiter.limit("20/minute")  # Per IP
def login():
    pass

# Layer 2: Breached Password Check
import hashlib
import requests

def is_password_breached(password):
    """Check against Have I Been Pwned API (k-anonymity)"""
    sha1 = hashlib.sha1(password.encode()).hexdigest().upper()
    prefix = sha1[:5]
    suffix = sha1[5:]
    
    resp = requests.get(f"https://api.pwnedpasswords.com/range/{prefix}")
    
    for line in resp.text.splitlines():
        hash_suffix, count = line.split(':')
        if hash_suffix == suffix:
            return True, int(count)  # Password found in breaches
    
    return False, 0

# Layer 3: Progressive Delays
def login_with_delay(email, password):
    failed_attempts = get_failed_attempts(email)
    
    if failed_attempts >= 3:
        delay = min(2 ** (failed_attempts - 3), 30)  # Exponential backoff
        time.sleep(delay)
    
    if authenticate(email, password):
        reset_failed_attempts(email)
        return success_response()
    else:
        increment_failed_attempts(email)
        return error_response("Invalid credentials")

# Layer 4: CAPTCHA after failures
@app.route('/login', methods=['POST'])
def login():
    failed = get_failed_attempts(request.json['email'])
    
    if failed >= 3:
        if not verify_captcha(request.json.get('captcha_token')):
            return error_response("CAPTCHA required"), 429
```

### Account Lockout Strategy

```
SMART LOCKOUT (NOT simple lockout):

Simple lockout (BAD):
  → Lock account after 5 failures
  → Problem: Attacker can lock out legitimate users (DoS)

Smart lockout (GOOD):
  → Track failed attempts by IP AND account separately
  → IP-based: Block IP after 20 failures across ANY account
  → Account-based: Require CAPTCHA after 3 failures
  → Device-based: Flag new devices for additional verification
  → Never fully lock the account (allow legitimate users with CAPTCHA)
  → Exponential backoff on failed attempts
```

---

## 5. Breached Credential Monitoring

```
PROACTIVE DEFENSE — Check if your users' credentials are breached:

1. HAVE I BEEN PWNED API
   → Check passwords at registration and login
   → k-anonymity model (only send first 5 chars of hash)
   → Free for operational use

2. ENZOIC (formerly PasswordPing)
   → Real-time credential monitoring
   → Enterprise API
   → Alert when user credentials appear in new breaches

3. SPYCLOUD
   → Automated credential monitoring
   → Dark web intelligence
   → Account takeover prevention

4. INTERNAL MONITORING
   → Hash all passwords in your database
   → Compare hashes against known breach databases
   → Force password reset for affected accounts
   → Notify affected users
```

---

## 6. Bot Detection and Mitigation

| Solution | Type | How It Works |
|----------|------|-------------|
| **reCAPTCHA v3** | Invisible scoring | Risk score 0.0-1.0 based on behavior |
| **hCaptcha** | Challenge-based | Privacy-focused CAPTCHA |
| **Cloudflare Bot Management** | Network-level | ML-based bot detection |
| **Akamai Bot Manager** | Enterprise | Behavioral analysis |
| **Device Fingerprinting** | Client-side | Canvas, WebGL, font fingerprints |
| **Proof of Work** | Computational | Client solves puzzle before request |

---

## Summary Table

| Defense Layer | Mechanism | Effectiveness |
|--------------|-----------|---------------|
| Rate limiting | Throttle login attempts | High (slows attacks) |
| Breached password check | Block known compromised passwords | High (prevents reuse) |
| CAPTCHA | Challenge automation | High (blocks bots) |
| MFA | Second authentication factor | Very High (blocks automated login) |
| Smart lockout | Progressive delays and challenges | High (limits attempts) |
| Bot detection | Behavioral analysis | High (identifies automation) |

---

## Revision Questions

1. Explain how credential stuffing works and why password reuse makes it effective.
2. What is the typical success rate of credential stuffing, and why is it still profitable at that rate?
3. List five indicators that a credential stuffing attack is in progress.
4. Implement a multi-layered defense against credential stuffing in Python/Node.js.
5. How does the Have I Been Pwned API use k-anonymity to check passwords securely?
6. Compare simple account lockout vs smart lockout — why is simple lockout actually a vulnerability?

---

*Previous: None (First topic in this unit) | Next: [02-brute-force-protection.md](02-brute-force-protection.md)*

---

*[Back to README](../README.md)*
