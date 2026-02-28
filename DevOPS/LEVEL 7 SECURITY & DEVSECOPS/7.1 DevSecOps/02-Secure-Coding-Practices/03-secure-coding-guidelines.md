# Chapter 2.3: Secure Coding Guidelines

## Overview

Secure coding guidelines are a set of best practices that developers follow to write code resistant to known attack patterns. These guidelines should be enforced through automated tools, code reviews, and developer training.

---

## Core Secure Coding Principles

```
+====================================================================+
|                SECURE CODING PRINCIPLES                              |
+====================================================================+
|                                                                      |
|  1. INPUT VALIDATION         "Never trust user input"               |
|  2. OUTPUT ENCODING          "Encode data for its context"          |
|  3. LEAST PRIVILEGE          "Minimum permissions needed"           |
|  4. DEFENSE IN DEPTH         "Multiple layers of security"         |
|  5. FAIL SECURELY            "Errors should not leak info"         |
|  6. KEEP IT SIMPLE           "Complexity breeds vulnerabilities"   |
|  7. SECURE DEFAULTS          "Secure out of the box"               |
|  8. SEPARATION OF CONCERNS   "Isolate security-critical code"     |
|  9. DON'T ROLL YOUR OWN      "Use proven security libraries"      |
| 10. FIX IT PROPERLY          "Address root cause, not symptoms"    |
+====================================================================+
```

---

## 1. Input Validation

```
INPUT VALIDATION STRATEGY:

  +------------------+
  |   User Input     |
  +--------+---------+
           |
  +--------v---------+
  | 1. TYPE CHECK    |  Is it the expected data type?
  |    (string, int) |  (Reject if wrong type)
  +--------+---------+
           |
  +--------v---------+
  | 2. LENGTH CHECK  |  Is it within allowed length?
  |    (min/max)     |  (Reject if too long/short)
  +--------+---------+
           |
  +--------v---------+
  | 3. RANGE CHECK   |  Is the value in allowed range?
  |    (boundaries)  |  (e.g., age: 0-150)
  +--------+---------+
           |
  +--------v---------+
  | 4. WHITELIST     |  Does it match allowed patterns?
  |    (regex/enum)  |  (Prefer whitelist over blacklist)
  +--------+---------+
           |
  +--------v---------+
  | 5. SANITIZE      |  Remove/encode dangerous chars
  |    (encode/strip)|  (After validation)
  +--------+---------+
           |
  +--------v---------+
  | VALID INPUT      |  Safe to process
  +------------------+
```

```python
# Python - Input Validation Examples

import re
from datetime import datetime

# === TYPE VALIDATION ===
def validate_age(age_input):
    try:
        age = int(age_input)
    except (ValueError, TypeError):
        raise ValidationError("Age must be a number")
    
    if not 0 <= age <= 150:
        raise ValidationError("Age must be between 0 and 150")
    
    return age

# === WHITELIST VALIDATION ===
def validate_username(username):
    # Only allow alphanumeric and underscores, 3-30 chars
    pattern = r'^[a-zA-Z0-9_]{3,30}$'
    if not re.match(pattern, username):
        raise ValidationError("Invalid username format")
    return username

# === EMAIL VALIDATION ===
def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if not re.match(pattern, email):
        raise ValidationError("Invalid email format")
    if len(email) > 254:
        raise ValidationError("Email too long")
    return email.lower()

# === FILE UPLOAD VALIDATION ===
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB

def validate_upload(file):
    # Check extension (whitelist)
    ext = file.filename.rsplit('.', 1)[-1].lower()
    if ext not in ALLOWED_EXTENSIONS:
        raise ValidationError(f"Extension .{ext} not allowed")
    
    # Check file size
    if file.content_length > MAX_FILE_SIZE:
        raise ValidationError("File too large")
    
    # Check magic bytes (file signature)
    header = file.read(4)
    file.seek(0)
    magic_bytes = {
        b'\x89PNG': 'png',
        b'\xff\xd8\xff\xe0': 'jpg',
        b'\xff\xd8\xff\xe1': 'jpg',
        b'GIF8': 'gif'
    }
    if header[:4] not in magic_bytes and header[:3] not in [b'\xff\xd8\xff']:
        raise ValidationError("File content doesn't match extension")
    
    return file
```

---

## 2. Authentication & Session Management

```python
# === SECURE PASSWORD HANDLING ===
import bcrypt
import secrets

# Hash password (NEVER store plaintext)
def hash_password(password):
    # bcrypt automatically handles salt
    salt = bcrypt.gensalt(rounds=12)
    return bcrypt.hashpw(password.encode('utf-8'), salt)

def verify_password(password, hashed):
    return bcrypt.checkpw(password.encode('utf-8'), hashed)

# === PASSWORD POLICY ===
def validate_password(password):
    errors = []
    if len(password) < 12:
        errors.append("Password must be at least 12 characters")
    if not re.search(r'[A-Z]', password):
        errors.append("Must contain uppercase letter")
    if not re.search(r'[a-z]', password):
        errors.append("Must contain lowercase letter")
    if not re.search(r'[0-9]', password):
        errors.append("Must contain a number")
    if not re.search(r'[!@#$%^&*(),.?":{}|<>]', password):
        errors.append("Must contain a special character")
    
    # Check against breached passwords
    if is_breached_password(password):
        errors.append("Password found in known data breaches")
    
    return errors

# === SECURE SESSION MANAGEMENT ===
def create_session(user_id):
    session_id = secrets.token_urlsafe(32)  # Cryptographically secure
    
    session = {
        'id': session_id,
        'user_id': user_id,
        'created_at': datetime.utcnow(),
        'expires_at': datetime.utcnow() + timedelta(hours=1),
        'ip_address': request.remote_addr,
    }
    
    store_session(session)
    
    response = make_response()
    response.set_cookie(
        'session_id',
        session_id,
        httponly=True,    # Prevents JavaScript access
        secure=True,      # HTTPS only
        samesite='Strict', # Prevents CSRF
        max_age=3600,     # 1 hour
        path='/'
    )
    return response
```

---

## 3. Error Handling & Logging

```
ERROR HANDLING FLOW:

  +------------------+
  |  Error Occurs    |
  +--------+---------+
           |
  +--------v---------+
  | Log DETAILED     |  Full stack trace, context
  | error internally |  (Server-side logs only)
  +--------+---------+
           |
  +--------v---------+
  | Return GENERIC   |  "An error occurred"
  | error to user    |  (No sensitive details!)
  +--------+---------+
           |
  +--------v---------+
  | Alert if needed  |  Critical errors trigger
  | (monitoring)     |  PagerDuty/Slack alerts
  +------------------+
```

```python
# === SECURE ERROR HANDLING ===
import logging

logger = logging.getLogger(__name__)

# WRONG: Exposes sensitive information
@app.errorhandler(500)
def handle_error_INSECURE(error):
    return f"""
    Error: {str(error)}
    Stack Trace: {traceback.format_exc()}
    Database: {app.config['DATABASE_URL']}
    """, 500  # NEVER DO THIS!

# RIGHT: Generic user message, detailed internal log
@app.errorhandler(500)
def handle_error_SECURE(error):
    # Log detailed error internally
    logger.error(
        "Internal error",
        extra={
            'error': str(error),
            'traceback': traceback.format_exc(),
            'request_id': request.headers.get('X-Request-ID'),
            'user_id': getattr(current_user, 'id', 'anonymous'),
            'endpoint': request.endpoint,
            'method': request.method,
        }
    )
    
    # Return generic message to user
    return jsonify({
        'error': 'An internal error occurred',
        'request_id': request.headers.get('X-Request-ID'),
        'support': 'Contact support with the request ID'
    }), 500

# === SECURE LOGGING ===
# What to log
logger.info("User login successful", extra={'user_id': user.id})
logger.warning("Failed login attempt", extra={
    'username': username,  # OK to log username
    'ip': request.remote_addr,
    'attempt_count': get_attempt_count(username)
})

# What NOT to log
logger.info(f"User {user.id} logged in with password {password}")  # NEVER!
logger.info(f"Session token: {session_token}")  # NEVER!
logger.info(f"Credit card: {card_number}")  # NEVER!
```

---

## 4. Cryptography Best Practices

```
+================================================================+
|  CRYPTOGRAPHY DO's AND DON'Ts                                   |
+================================================================+
|                                                                  |
|  HASHING (One-way, for passwords/integrity)                     |
|  +----------------------------------------------------------+  |
|  |  DO:    bcrypt, Argon2id, scrypt (for passwords)          |  |
|  |         SHA-256, SHA-3 (for integrity)                    |  |
|  |  DON'T: MD5, SHA-1 (broken/weak)                         |  |
|  +----------------------------------------------------------+  |
|                                                                  |
|  ENCRYPTION (Two-way, for data protection)                      |
|  +----------------------------------------------------------+  |
|  |  SYMMETRIC:  AES-256-GCM (authenticated encryption)      |  |
|  |  ASYMMETRIC: RSA-2048+ or Ed25519 (key exchange)         |  |
|  |  DON'T:      DES, 3DES, RC4, ECB mode                    |  |
|  +----------------------------------------------------------+  |
|                                                                  |
|  TLS (Transport security)                                        |
|  +----------------------------------------------------------+  |
|  |  DO:    TLS 1.3 (preferred), TLS 1.2 (minimum)           |  |
|  |  DON'T: SSL 3.0, TLS 1.0, TLS 1.1 (deprecated)          |  |
|  +----------------------------------------------------------+  |
|                                                                  |
|  RANDOM NUMBERS                                                  |
|  +----------------------------------------------------------+  |
|  |  DO:    secrets module (Python), crypto.randomBytes (JS)  |  |
|  |  DON'T: random module (Python), Math.random() (JS)       |  |
|  +----------------------------------------------------------+  |
+================================================================+
```

```python
# === SECURE ENCRYPTION ===
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

# AES-256-GCM (Authenticated Encryption)
def encrypt_data(plaintext, key):
    aesgcm = AESGCM(key)
    nonce = os.urandom(12)  # MUST be unique per encryption
    ciphertext = aesgcm.encrypt(nonce, plaintext.encode(), None)
    return nonce + ciphertext  # Store nonce with ciphertext

def decrypt_data(data, key):
    aesgcm = AESGCM(key)
    nonce = data[:12]
    ciphertext = data[12:]
    return aesgcm.decrypt(nonce, ciphertext, None).decode()

# === SECURE TOKEN GENERATION ===
import secrets

api_key = secrets.token_urlsafe(32)      # URL-safe base64
session_id = secrets.token_hex(32)        # Hex string
reset_token = secrets.token_urlsafe(48)   # Password reset token
```

---

## 5. API Security

```python
# === RATE LIMITING ===
from flask_limiter import Limiter

limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route("/api/login", methods=["POST"])
@limiter.limit("5 per minute")  # Strict for auth endpoints
def login():
    pass

@app.route("/api/data")
@limiter.limit("100 per minute")
def get_data():
    pass

# === API KEY VALIDATION ===
from functools import wraps

def require_api_key(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        api_key = request.headers.get('X-API-Key')
        if not api_key:
            return jsonify({'error': 'API key required'}), 401
        
        # Use constant-time comparison to prevent timing attacks
        if not secrets.compare_digest(api_key, expected_key):
            return jsonify({'error': 'Invalid API key'}), 403
        
        return f(*args, **kwargs)
    return decorated

# === JWT VALIDATION ===
import jwt

def validate_jwt(token):
    try:
        payload = jwt.decode(
            token,
            app.config['JWT_SECRET'],
            algorithms=['HS256'],  # Specify algorithm explicitly!
            options={
                'require': ['exp', 'iat', 'sub'],
                'verify_exp': True,
            }
        )
        return payload
    except jwt.ExpiredSignatureError:
        raise AuthError("Token has expired")
    except jwt.InvalidTokenError:
        raise AuthError("Invalid token")
```

---

## 6. File & Path Security

```python
# === PATH TRAVERSAL PREVENTION ===
import os

UPLOAD_DIR = '/app/uploads'

def safe_file_access(filename):
    # VULNERABLE:
    # path = os.path.join(UPLOAD_DIR, filename)
    # open(path)  # filename = "../../etc/passwd" = disaster!
    
    # SECURE:
    # Resolve the full path and verify it's within allowed directory
    requested_path = os.path.realpath(
        os.path.join(UPLOAD_DIR, filename)
    )
    
    if not requested_path.startswith(os.path.realpath(UPLOAD_DIR)):
        raise SecurityError("Path traversal attempt detected")
    
    if not os.path.exists(requested_path):
        raise FileNotFoundError("File not found")
    
    return requested_path

# === SECURE FILE UPLOAD ===
import uuid

def secure_upload(file):
    # 1. Validate file (type, size, content)
    validate_upload(file)
    
    # 2. Generate safe filename (never use user-provided name)
    ext = file.filename.rsplit('.', 1)[-1].lower()
    safe_name = f"{uuid.uuid4()}.{ext}"
    
    # 3. Save to isolated directory
    filepath = os.path.join(UPLOAD_DIR, safe_name)
    file.save(filepath)
    
    # 4. Set restrictive permissions
    os.chmod(filepath, 0o644)  # Read-only for others
    
    return safe_name
```

---

## Language-Specific Guidelines

### Python Security Checklist

```
+--------------------------------------------------------------+
| [x] Use virtual environments (isolate dependencies)          |
| [x] Pin dependency versions (pip freeze > requirements.txt)  |
| [x] Run bandit for security linting                          |
| [x] Use secrets module for random tokens (not random)        |
| [x] Avoid eval(), exec(), pickle with untrusted data         |
| [x] Use subprocess with shell=False                          |
| [x] Set DEBUG=False in production                            |
| [x] Use parameterized queries with SQLAlchemy                |
+--------------------------------------------------------------+
```

### JavaScript/Node.js Security Checklist

```
+--------------------------------------------------------------+
| [x] Use npm audit regularly                                  |
| [x] Enable strict mode ('use strict')                        |
| [x] Use helmet.js for security headers                       |
| [x] Sanitize user input (validator.js, DOMPurify)            |
| [x] Avoid eval(), new Function() with user data             |
| [x] Use crypto.randomBytes() for tokens                      |
| [x] Set secure cookie flags (httpOnly, secure, sameSite)     |
| [x] Use parameterized queries (not string concatenation)     |
+--------------------------------------------------------------+
```

### Java Security Checklist

```
+--------------------------------------------------------------+
| [x] Use PreparedStatement (never Statement with user data)   |
| [x] Enable CSRF protection in Spring Security               |
| [x] Use BCryptPasswordEncoder for passwords                  |
| [x] Validate all deserialization (avoid Java native)         |
| [x] Use SecureRandom (not Random) for security tokens        |
| [x] Configure security headers in Spring Security            |
| [x] Enable HTTPS with proper TLS configuration               |
| [x] Use OWASP Java Encoder for output encoding              |
+--------------------------------------------------------------+
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Input validation bypassed | Client-side only validation | Always validate server-side; client validation is UX only |
| Password hashing too slow | bcrypt rounds too high | Use 10-12 rounds; benchmark on your hardware |
| JWT "none" algorithm attack | Algorithm not pinned | Always specify algorithm explicitly in verification |
| File upload exploits | Only checking extension | Check magic bytes, use antivirus scanning, sandboxing |
| Timing attack on auth | Using `==` for comparison | Use `secrets.compare_digest()` (constant-time) |

---

## Summary Table

| Principle | Implementation |
|-----------|---------------|
| **Input Validation** | Type, length, range, whitelist, sanitize |
| **Output Encoding** | HTML entity, URL, JavaScript, CSS encoding |
| **Authentication** | bcrypt/Argon2, MFA, secure sessions |
| **Error Handling** | Generic user messages, detailed internal logs |
| **Cryptography** | AES-256-GCM, bcrypt, TLS 1.3, secrets module |
| **API Security** | Rate limiting, API keys, JWT validation |
| **File Security** | Path traversal prevention, safe filenames, size limits |
| **Least Privilege** | Minimum permissions, deny by default |
| **Defense in Depth** | Multiple layers: validation + WAF + encoding + CSP |

---

## Quick Revision Questions

1. **Why should you prefer whitelist validation over blacklist validation?**
   > Whitelisting defines what IS allowed (safer, smaller surface). Blacklisting defines what ISN'T allowed and can miss new attack patterns. Attackers constantly discover new payloads that bypass blacklists.

2. **Why should you never use `random.random()` (Python) or `Math.random()` (JS) for security tokens?**
   > They use pseudo-random number generators (PRNGs) that are predictable if the seed is known. Use `secrets` module (Python) or `crypto.randomBytes()` (Node.js) which use cryptographically secure random numbers.

3. **What is a timing attack and how do you prevent it in authentication?**
   > A timing attack measures how long a comparison takes to determine how many characters match. Standard `==` comparison short-circuits on first mismatch. Use `secrets.compare_digest()` which takes constant time regardless of input.

4. **Why should error messages shown to users be generic?**
   > Detailed errors (stack traces, DB info, file paths) help attackers understand your system. Log details internally for debugging but return only generic messages to users with a request ID for support reference.

5. **What is defense in depth and give an example?**
   > Using multiple security layers so that if one fails, others still protect. Example: SQL injection protection = input validation + parameterized queries + WAF + least-privilege DB user + network segmentation.

6. **Why should you never use `eval()` or `exec()` with user input?**
   > They execute arbitrary code. If user input reaches `eval()`, attackers can execute any command on your server (Remote Code Execution). Always use safer alternatives like JSON.parse() for data parsing.

---

[← Previous: 2.2 Common Vulnerabilities](02-common-vulnerabilities.md) | [Next: 2.4 Code Review for Security →](04-code-review-for-security.md)

[Back to Table of Contents](../README.md)
