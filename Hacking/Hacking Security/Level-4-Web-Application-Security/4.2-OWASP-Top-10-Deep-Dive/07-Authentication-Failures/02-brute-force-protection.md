# Unit 7: A07 - Identification and Authentication Failures — Topic 2: Brute Force Protection

## Overview

Brute force attacks systematically try every possible password combination until the correct one is found. While credential stuffing uses known credentials, brute force uses **exhaustive or dictionary-based guessing**. Without proper protection, even strong passwords can be cracked given enough time and computing power. This topic covers attack techniques and comprehensive defense mechanisms.

---

## 1. Types of Brute Force Attacks

```
┌─────────────────────────────────────────────────────────────────┐
│               BRUTE FORCE ATTACK TYPES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. SIMPLE BRUTE FORCE                                         │
│     Try every combination: a, b, c... aa, ab, ac...           │
│     Time: 8-char lowercase = 26^8 = 208B combinations         │
│     Speed: 1B/sec = 208 seconds (offline hash cracking)       │
│                                                                 │
│  2. DICTIONARY ATTACK                                          │
│     Try common words/passwords from a wordlist                 │
│     Wordlists: rockyou.txt (14M), SecLists, custom lists      │
│     Much faster than simple brute force                        │
│                                                                 │
│  3. HYBRID ATTACK                                              │
│     Dictionary + rules (password → P@ssw0rd, Password1!)      │
│     Combines dictionary words with common mutations            │
│                                                                 │
│  4. REVERSE BRUTE FORCE                                        │
│     Use one password, try many usernames                       │
│     Example: Try "Password123" against all accounts            │
│     Bypasses per-account lockout                               │
│                                                                 │
│  5. PASSWORD SPRAYING                                          │
│     Try small set of common passwords across many accounts     │
│     1-2 attempts per account to avoid lockout                  │
│     Common in Active Directory attacks                         │
│                                                                 │
│  6. RAINBOW TABLE ATTACK                                       │
│     Pre-computed hash → password lookup tables                 │
│     Defeated by salted hashing                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Attack Tools and Techniques

```bash
# Hydra — Network login brute forcer
# HTTP POST form
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
    target.com http-post-form \
    "/login:username=^USER^&password=^PASS^:Invalid credentials"

# SSH
hydra -l root -P wordlist.txt target.com ssh

# FTP
hydra -L users.txt -P passwords.txt target.com ftp

# Rate-limited targets
hydra -l admin -P wordlist.txt -t 1 -W 5 target.com http-post-form \
    "/login:user=^USER^&pass=^PASS^:F=Invalid"
# -t 1: single thread, -W 5: 5 second wait between attempts

# Burp Suite Intruder
# 1. Capture login request
# 2. Send to Intruder
# 3. Mark password field as payload position
# 4. Load wordlist as payload
# 5. Filter by response length/status for success

# Hashcat — Offline password hash cracking
# MD5
hashcat -m 0 hashes.txt wordlist.txt

# SHA-256
hashcat -m 1400 hashes.txt wordlist.txt

# bcrypt
hashcat -m 3200 hashes.txt wordlist.txt

# With rules (mutations)
hashcat -m 0 hashes.txt wordlist.txt -r rules/best64.rule

# John the Ripper
john --wordlist=rockyou.txt --format=bcrypt hashes.txt
john --incremental hashes.txt  # Pure brute force
```

---

## 3. Comprehensive Brute Force Protection

### Rate Limiting Implementation

```python
import time
import redis

r = redis.Redis()

class BruteForceProtection:
    def __init__(self):
        self.MAX_ATTEMPTS_PER_ACCOUNT = 5    # Per 15 minutes
        self.MAX_ATTEMPTS_PER_IP = 20        # Per 15 minutes
        self.LOCKOUT_DURATION = 900          # 15 minutes
        self.PROGRESSIVE_DELAYS = [0, 0, 0, 2, 4, 8, 16, 30]
    
    def check_and_limit(self, email, ip_address):
        """Multi-dimensional rate limiting"""
        
        # Check IP rate
        ip_key = f"login_attempts:ip:{ip_address}"
        ip_attempts = int(r.get(ip_key) or 0)
        
        if ip_attempts >= self.MAX_ATTEMPTS_PER_IP:
            return False, "Too many attempts from this IP", 0
        
        # Check account rate
        account_key = f"login_attempts:account:{email}"
        account_attempts = int(r.get(account_key) or 0)
        
        if account_attempts >= self.MAX_ATTEMPTS_PER_ACCOUNT:
            return False, "Account temporarily locked", 0
        
        # Progressive delay
        delay_index = min(account_attempts, len(self.PROGRESSIVE_DELAYS) - 1)
        delay = self.PROGRESSIVE_DELAYS[delay_index]
        
        return True, "OK", delay
    
    def record_failure(self, email, ip_address):
        """Record failed attempt"""
        ip_key = f"login_attempts:ip:{ip_address}"
        r.incr(ip_key)
        r.expire(ip_key, self.LOCKOUT_DURATION)
        
        account_key = f"login_attempts:account:{email}"
        r.incr(account_key)
        r.expire(account_key, self.LOCKOUT_DURATION)
    
    def record_success(self, email, ip_address):
        """Clear counters on successful login"""
        r.delete(f"login_attempts:account:{email}")
        # Don't clear IP counter (could be attacker who guessed right)

protection = BruteForceProtection()

@app.route('/login', methods=['POST'])
def login():
    email = request.json.get('email')
    password = request.json.get('password')
    ip = request.remote_addr
    
    # Check rate limits
    allowed, message, delay = protection.check_and_limit(email, ip)
    if not allowed:
        return jsonify({'error': message}), 429
    
    # Apply progressive delay
    if delay > 0:
        time.sleep(delay)
    
    # Attempt authentication
    if authenticate(email, password):
        protection.record_success(email, ip)
        return jsonify({'token': generate_token(email)})
    else:
        protection.record_failure(email, ip)
        return jsonify({'error': 'Invalid credentials'}), 401
```

---

## 4. Secure Password Hashing

```python
# Password hashing comparison
import bcrypt
import hashlib
import argon2

# ❌ INSECURE — Plain text
password_stored = "mysecretpassword"

# ❌ INSECURE — MD5 (fast, no salt)
hash_md5 = hashlib.md5(password.encode()).hexdigest()
# Cracking speed: 10+ billion/sec on GPU

# ❌ INSECURE — SHA-256 (fast, even with salt)
hash_sha256 = hashlib.sha256(password.encode()).hexdigest()
# Cracking speed: 5+ billion/sec on GPU

# ✅ SECURE — bcrypt (slow, built-in salt)
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
# bcrypt.checkpw(password.encode(), hashed)
# Cracking speed: ~25,000/sec on GPU (200,000x slower than MD5)

# ✅ MOST SECURE — Argon2id (memory-hard)
from argon2 import PasswordHasher
ph = PasswordHasher(
    time_cost=3,        # Iterations
    memory_cost=65536,  # 64 MB memory
    parallelism=4       # Threads
)
hashed = ph.hash(password)
# ph.verify(hashed, password)
# Most resistant to GPU/ASIC cracking
```

### Hashing Algorithm Comparison

| Algorithm | Speed (GPU) | Salted | Memory-Hard | Recommendation |
|-----------|------------|--------|-------------|----------------|
| MD5 | 10B+/sec | ❌ | ❌ | ❌ Never use |
| SHA-256 | 5B+/sec | Optional | ❌ | ❌ Never use for passwords |
| bcrypt | ~25K/sec | ✅ | ❌ | ✅ Good (cost ≥ 12) |
| scrypt | ~10K/sec | ✅ | ✅ | ✅ Good |
| Argon2id | ~1K/sec | ✅ | ✅ | ✅ Best (OWASP recommended) |

---

## 5. CAPTCHA Integration

```python
import requests

def verify_recaptcha_v3(token, action='login', threshold=0.5):
    """Verify reCAPTCHA v3 token"""
    resp = requests.post('https://www.google.com/recaptcha/api/siteverify', data={
        'secret': RECAPTCHA_SECRET_KEY,
        'response': token
    })
    
    result = resp.json()
    
    return (
        result.get('success', False) and
        result.get('score', 0) >= threshold and
        result.get('action') == action
    )

@app.route('/login', methods=['POST'])
def login():
    email = request.json.get('email')
    failed_attempts = get_failed_attempts(email)
    
    # Require CAPTCHA after 3 failures
    if failed_attempts >= 3:
        captcha_token = request.json.get('captcha_token')
        if not captcha_token or not verify_recaptcha_v3(captcha_token):
            return jsonify({
                'error': 'CAPTCHA verification required',
                'captcha_required': True
            }), 429
    
    # Continue with authentication...
```

---

## 6. Password Spraying Defense

```
PASSWORD SPRAYING is harder to detect because:
- Only 1-2 attempts per account (below lockout threshold)
- Distributed across many accounts
- May use different source IPs

DETECTION:
1. Monitor GLOBAL failed login rate (not just per-account)
2. Alert on same password used across multiple accounts
3. Track failed logins per time window across all accounts
4. Correlate with known common passwords

PREVENTION:
1. Ban common passwords (top 10,000 + context-specific)
2. Require MFA for all accounts
3. Password complexity that blocks "Season+Year" patterns
4. Monitor for impossible travel / suspicious locations
```

---

## Summary Table

| Attack Type | Target | Speed | Best Defense |
|------------|--------|-------|-------------|
| Simple Brute Force | Single account | Slow online, fast offline | Rate limiting + strong hashing |
| Dictionary | Single account | Fast online | Password policy + CAPTCHA |
| Credential Stuffing | Many accounts | Very fast | MFA + breached password check |
| Password Spraying | Many accounts | Slow per account | Ban common passwords + MFA |
| Reverse Brute Force | Many accounts, one password | Moderate | Global rate limiting |
| Offline Cracking | Stolen hashes | Billions/sec | Argon2id / bcrypt |

---

## Revision Questions

1. Compare all six types of brute force attacks. Which bypasses per-account lockout?
2. Implement multi-dimensional rate limiting that tracks attempts by IP, account, and global rate.
3. Why is bcrypt/Argon2id better than SHA-256 for password hashing? Include cracking speed comparisons.
4. How does password spraying differ from credential stuffing? What makes it harder to detect?
5. Design a CAPTCHA integration strategy that doesn't annoy legitimate users.
6. What is the cracking time for an 8-character password using different hashing algorithms?

---

*Previous: [01-credential-stuffing.md](01-credential-stuffing.md) | Next: [03-session-management.md](03-session-management.md)*

---

*[Back to README](../README.md)*
