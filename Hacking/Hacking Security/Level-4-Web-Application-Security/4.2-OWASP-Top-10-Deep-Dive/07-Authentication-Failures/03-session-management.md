# Unit 7: A07 - Identification and Authentication Failures — Topic 3: Session Management

## Overview

Session management is how a web application **maintains state and identity** across multiple HTTP requests after authentication. Insecure session management can allow attackers to hijack authenticated sessions, bypass authentication entirely, or escalate privileges. This topic covers session security mechanisms, attack techniques, and secure implementation patterns.

---

## 1. Session Management Fundamentals

```
┌─────────────────────────────────────────────────────────────────┐
│              SESSION LIFECYCLE                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. USER AUTHENTICATES                                         │
│     ├── Provides credentials                                   │
│     └── Server verifies identity                               │
│                                                                 │
│  2. SESSION CREATED                                            │
│     ├── Server generates random session ID                     │
│     ├── Server stores session data (user ID, role, etc.)       │
│     └── Session ID sent to client (usually via cookie)         │
│                                                                 │
│  3. SUBSEQUENT REQUESTS                                        │
│     ├── Client sends session ID with every request             │
│     ├── Server looks up session data                           │
│     └── Server identifies user and applies authorization       │
│                                                                 │
│  4. SESSION TERMINATION                                        │
│     ├── User logs out (explicit)                               │
│     ├── Session times out (idle or absolute)                   │
│     └── Server invalidates session                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Session Attacks

### Session Hijacking

```
ATTACK VECTORS:
┌────────────────────────┬──────────────────────────────────────────┐
│ Vector                 │ Method                                   │
├────────────────────────┼──────────────────────────────────────────┤
│ Network sniffing       │ Capture session cookie over HTTP (WiFi)  │
│ XSS                   │ document.cookie sent to attacker          │
│ Session fixation       │ Force known session ID on victim          │
│ CSRF                  │ Trick browser into sending session cookie │
│ Man-in-the-middle     │ Intercept HTTPS (cert issues)             │
│ Malware/browser ext   │ Steal cookies from browser storage        │
│ Physical access       │ Copy cookies from unlocked computer       │
│ Predictable IDs       │ Guess/brute-force session identifiers    │
└────────────────────────┴──────────────────────────────────────────┘
```

### Session Fixation

```
ATTACK FLOW:

1. Attacker creates a session on target site
   → Gets session ID: ABC123

2. Attacker sends victim a link with that session ID:
   https://bank.com/login?JSESSIONID=ABC123

3. Victim clicks link and logs in
   → Server authenticates and KEEPS the same session ID

4. Attacker uses ABC123 to access victim's authenticated session
   → Full account access!

PREVENTION:
→ Regenerate session ID after authentication
→ Don't accept session IDs from URL parameters
→ Only accept server-generated session IDs
```

---

## 3. Secure Session Implementation

### Cookie Security Flags

```
Set-Cookie: session=abc123;
    Secure;          → Only send over HTTPS
    HttpOnly;        → JavaScript cannot access (prevents XSS theft)
    SameSite=Lax;    → CSRF protection (not sent on cross-site POST)
    Path=/;          → Available on all paths
    Domain=.example.com;  → Available on subdomains
    Max-Age=3600;    → Expires in 1 hour

SECURITY FLAG MATRIX:
┌────────────────┬──────────────────┬──────────────────────────────┐
│ Flag           │ Without          │ With                         │
├────────────────┼──────────────────┼──────────────────────────────┤
│ Secure         │ Sent over HTTP   │ HTTPS only                   │
│ HttpOnly       │ JS can read      │ JS cannot access             │
│ SameSite=Strict│ Sent everywhere  │ Only same-site requests      │
│ SameSite=Lax   │ Sent everywhere  │ Same-site + top-level GET    │
│ SameSite=None  │ N/A              │ Always sent (requires Secure)│
└────────────────┴──────────────────┴──────────────────────────────┘
```

### Session ID Generation

```python
import secrets
import hashlib

# ❌ INSECURE — Predictable session IDs
session_id = str(user_id)                    # Sequential
session_id = hashlib.md5(str(time.time()))    # Timestamp-based
session_id = str(random.randint(1, 999999))  # Low entropy

# ✅ SECURE — Cryptographically random
session_id = secrets.token_hex(32)  # 256 bits of entropy
session_id = secrets.token_urlsafe(32)  # URL-safe, 256 bits

# OWASP recommendation:
# - At least 128 bits of entropy
# - Generated by cryptographic PRNG
# - Not predictable or sequential
```

### Session Configuration

```python
# Flask secure session configuration
app.config.update(
    SECRET_KEY=os.environ['SECRET_KEY'],
    SESSION_COOKIE_SECURE=True,        # HTTPS only
    SESSION_COOKIE_HTTPONLY=True,       # No JS access
    SESSION_COOKIE_SAMESITE='Lax',     # CSRF protection
    SESSION_COOKIE_NAME='__Host-sid',  # Cookie prefix (requires Secure + Path=/)
    PERMANENT_SESSION_LIFETIME=timedelta(hours=8),  # Absolute timeout
    SESSION_COOKIE_PATH='/',
)

# Session regeneration on privilege change
@app.route('/login', methods=['POST'])
def login():
    if authenticate(username, password):
        session.clear()                     # Clear old session data
        session.regenerate()                # Generate new session ID
        session['user_id'] = user.id
        session['role'] = user.role
        session['login_time'] = datetime.utcnow().isoformat()
        session['ip'] = request.remote_addr
        return redirect('/dashboard')

# Logout — proper session destruction
@app.route('/logout', methods=['POST'])
def logout():
    session.clear()      # Clear session data
    session.destroy()    # Destroy session on server
    response = redirect('/login')
    response.delete_cookie('__Host-sid')  # Delete cookie
    return response
```

---

## 4. Session Timeout Policies

```
TWO TYPES OF TIMEOUT:

IDLE TIMEOUT (Inactivity):
  → Session expires after X minutes of no activity
  → Recommended: 15-30 minutes for standard apps
  → Financial apps: 5-15 minutes
  → Reset timer on each request

ABSOLUTE TIMEOUT:
  → Session expires after X hours regardless of activity
  → Recommended: 4-8 hours
  → Prevents indefinite session use
  → User must re-authenticate

IMPLEMENTATION:
session_data = {
    'user_id': user.id,
    'created_at': datetime.utcnow(),      # For absolute timeout
    'last_activity': datetime.utcnow(),   # For idle timeout
}

def check_session_validity(session):
    now = datetime.utcnow()
    
    # Absolute timeout (8 hours)
    if (now - session['created_at']).total_seconds() > 28800:
        return False, "Session expired"
    
    # Idle timeout (30 minutes)
    if (now - session['last_activity']).total_seconds() > 1800:
        return False, "Session timed out due to inactivity"
    
    # Update last activity
    session['last_activity'] = now
    return True, "Valid"
```

---

## 5. JWT Session Security

```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = os.environ['JWT_SECRET']

# Generate JWT
def create_token(user):
    payload = {
        'sub': user.id,
        'email': user.email,
        'role': user.role,
        'iat': datetime.utcnow(),
        'exp': datetime.utcnow() + timedelta(hours=1),
        'jti': secrets.token_hex(16),  # Unique ID for revocation
    }
    return jwt.encode(payload, SECRET_KEY, algorithm='HS256')

# Verify JWT
def verify_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        
        # Check if token is revoked
        if is_token_revoked(payload['jti']):
            raise jwt.InvalidTokenError("Token revoked")
        
        return payload
    except jwt.ExpiredSignatureError:
        raise AuthError("Token expired")
    except jwt.InvalidTokenError as e:
        raise AuthError(f"Invalid token: {e}")

# JWT SECURITY RULES:
# 1. Use strong secret (256+ bits)
# 2. Always set expiration (short-lived: 15min-1hr)
# 3. Use refresh tokens for long sessions
# 4. Validate algorithm (prevent 'none' attack)
# 5. Include jti for revocation capability
# 6. Store in HttpOnly cookie (not localStorage)
```

---

## 6. Concurrent Session Control

```python
# Limit active sessions per user
MAX_SESSIONS = 3

def create_session(user_id):
    sessions = get_active_sessions(user_id)
    
    if len(sessions) >= MAX_SESSIONS:
        # Option 1: Reject new login
        # return error("Too many active sessions")
        
        # Option 2: Invalidate oldest session (preferred)
        oldest = min(sessions, key=lambda s: s['created_at'])
        invalidate_session(oldest['session_id'])
    
    new_session = {
        'session_id': secrets.token_hex(32),
        'user_id': user_id,
        'created_at': datetime.utcnow(),
        'ip': request.remote_addr,
        'user_agent': request.user_agent.string
    }
    store_session(new_session)
    return new_session
```

---

## Summary Table

| Security Control | Prevents | Implementation |
|-----------------|----------|----------------|
| Secure flag | Session theft via HTTP | `Secure` cookie flag |
| HttpOnly flag | XSS session theft | `HttpOnly` cookie flag |
| SameSite flag | CSRF attacks | `SameSite=Lax` or `Strict` |
| Session regeneration | Session fixation | Regenerate after auth |
| Idle timeout | Abandoned session hijack | 15-30 min inactivity |
| Absolute timeout | Long-term session abuse | 4-8 hour maximum |
| Cryptographic IDs | Session prediction | `secrets.token_hex(32)` |

---

## Revision Questions

1. Explain the session lifecycle from authentication to termination.
2. What are all the cookie security flags and what does each prevent?
3. How does session fixation work? Write code that prevents it.
4. Compare idle timeout vs absolute timeout. What values are recommended for banking vs social media?
5. What are the security considerations for JWT-based sessions vs server-side sessions?
6. How would you implement concurrent session control to limit users to 3 active sessions?

---

*Previous: [02-brute-force-protection.md](02-brute-force-protection.md) | Next: [04-password-requirements.md](04-password-requirements.md)*

---

*[Back to README](../README.md)*
