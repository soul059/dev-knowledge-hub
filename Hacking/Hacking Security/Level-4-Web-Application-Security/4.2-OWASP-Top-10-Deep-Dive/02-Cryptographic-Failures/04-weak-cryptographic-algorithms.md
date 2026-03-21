# Unit 2: A02 - Cryptographic Failures — Topic 4: Weak Cryptographic Algorithms

## Overview

Using deprecated or weak cryptographic algorithms is a critical failure that undermines all other security controls. Algorithms like **MD5, SHA-1, DES, 3DES, and RC4** have known mathematical weaknesses that allow practical attacks — collision generation, brute-force recovery, or statistical biases. Understanding why these algorithms are broken and what modern alternatives to use is essential for both offensive testing and secure development.

---

## 1. Deprecated Hash Algorithms

### MD5 (Message Digest 5)

```
Status: ❌ BROKEN — Collision attacks practical since 2004

MD5 Properties:
  Output: 128-bit (32 hex characters)
  Speed: ~5 billion hashes/second on GPU
  Collision: Can be generated in seconds

Timeline:
  1991: MD5 published
  1996: First collision weakness found
  2004: Full collision attack (Xiaoyun Wang)
  2008: Rogue CA certificate created using MD5 collision
  2012: Flame malware used MD5 collision for code signing

Example collision:
  File A: d131dd02c5e6eec4... → MD5: 79054025255fb1a26e4bc422aef54eb4
  File B: d131dd02c5e6eec4... → MD5: 79054025255fb1a26e4bc422aef54eb4
  (Different files, same MD5!)
```

### SHA-1 (Secure Hash Algorithm 1)

```
Status: ❌ BROKEN — Practical collision demonstrated in 2017 (SHAttered)

SHA-1 Properties:
  Output: 160-bit (40 hex characters)
  Speed: ~3 billion hashes/second on GPU
  Collision cost: ~$110,000 (2017), dropping

Timeline:
  1995: SHA-1 published
  2005: Theoretical collision attack (2^63 operations)
  2017: Google/CWI demonstrated practical collision (SHAttered)
  2020: Chosen-prefix collision ($45,000 cost)
  
Real-world impact:
  • Git uses SHA-1 for commit hashes (being migrated to SHA-256)
  • Certificate authorities stopped issuing SHA-1 certificates (2017)
  • Major browsers reject SHA-1 certificates
```

### Comparison Table

| Algorithm | Output Size | Collision Resistance | Speed (GPU) | Status |
|-----------|------------|---------------------|-------------|--------|
| **MD5** | 128-bit | ❌ Broken (seconds) | ~5B/s | NEVER USE |
| **SHA-1** | 160-bit | ❌ Broken ($45K) | ~3B/s | NEVER USE |
| **SHA-256** | 256-bit | ✅ Secure | ~1B/s | ✅ Recommended |
| **SHA-384** | 384-bit | ✅ Secure | ~700M/s | ✅ Recommended |
| **SHA-512** | 512-bit | ✅ Secure | ~500M/s | ✅ Recommended |
| **SHA-3** | Variable | ✅ Secure | ~400M/s | ✅ Recommended |
| **BLAKE2** | Variable | ✅ Secure | ~1.5B/s | ✅ Recommended (fast) |

---

## 2. Deprecated Encryption Algorithms

### DES (Data Encryption Standard)

```
Status: ❌ BROKEN — 56-bit key brute-forced in 22 hours (1999)

DES Properties:
  Key size: 56 bits (effective)
  Block size: 64 bits
  Brute force: ~2^56 operations (~72 quadrillion)
  
Timeline:
  1977: Published as federal standard
  1998: EFF's Deep Crack machine: 56 hours
  1999: Deep Crack + distributed.net: 22 hours
  2005: NIST withdrew DES as a standard

Key size is simply too small for modern computing
```

### 3DES (Triple DES)

```
Status: ⚠️ DEPRECATED — Sweet32 attack (2016), slow

3DES Properties:
  Key size: 112 or 168 bits
  Block size: 64 bits ← The problem
  Sweet32: After 2^32 blocks (~32 GB), birthday attack recovers plaintext
  
NIST deprecated 3DES in 2023 for all applications
```

### RC4 (Rivest Cipher 4)

```
Status: ❌ BROKEN — Statistical biases in output stream

RC4 Properties:
  Type: Stream cipher
  Key size: Variable (40-2048 bits)
  Issue: First bytes of keystream have detectable biases
  
Attacks:
  • 2013: Royal Holloway attack (recover plaintext from TLS)
  • 2015: Bar-mitzvah attack (recover cookies)
  • RFC 7465: Prohibits RC4 in TLS (2015)
```

### Modern Algorithm Recommendations

| Use Case | ❌ Avoid | ✅ Use |
|----------|---------|--------|
| **Symmetric encryption** | DES, 3DES, RC4, Blowfish | AES-256-GCM, ChaCha20-Poly1305 |
| **Hashing** | MD5, SHA-1 | SHA-256, SHA-3, BLAKE2 |
| **Password hashing** | MD5, SHA-*, plain | Argon2id, bcrypt, scrypt |
| **Key exchange** | RSA key transport, DH-768 | ECDHE (P-256, X25519) |
| **Digital signatures** | RSA-1024, DSA-1024 | RSA-2048+, ECDSA (P-256), Ed25519 |
| **MAC** | HMAC-MD5, HMAC-SHA1 | HMAC-SHA256, Poly1305 |

---

## 3. Why Weak Algorithms Are Exploitable

### Birthday Attack (Hash Collisions)

```
Birthday Paradox:
  In a room of 23 people, there's a 50% chance two share a birthday
  
Applied to cryptography:
  For an n-bit hash, collision expected after ~2^(n/2) hashes
  
  MD5 (128-bit):  Collision after ~2^64 operations ← feasible
  SHA-1 (160-bit): Collision after ~2^80 operations ← feasible
  SHA-256 (256-bit): Collision after ~2^128 operations ← infeasible
```

### Brute Force Feasibility

| Key Size | Combinations | Time (10B keys/sec) | Status |
|----------|-------------|---------------------|--------|
| 40-bit | 1.1 × 10^12 | 110 seconds | ❌ Trivial |
| 56-bit (DES) | 7.2 × 10^16 | 83 days | ❌ Broken |
| 64-bit | 1.8 × 10^19 | 58 years | ⚠️ Weak |
| 128-bit (AES) | 3.4 × 10^38 | 10^21 years | ✅ Secure |
| 256-bit (AES) | 1.2 × 10^77 | 10^60 years | ✅ Very secure |

---

## 4. Real-World Breaches from Weak Crypto

### LinkedIn (2012) — SHA-1 Unsalted

```
What happened:
  • 6.5 million password hashes leaked
  • SHA-1 without salt
  • 90%+ cracked within days using GPU hashcat

Impact:
  • Credential stuffing attacks across other services
  • Class-action settlement: $1.25 million
  
Root cause: SHA-1 is fast → easy to brute force passwords
Fix: Switch to bcrypt/Argon2id with salt
```

### Adobe (2013) — 3DES ECB Mode

```
What happened:
  • 153 million user records stolen
  • Passwords encrypted (not hashed!) with 3DES in ECB mode
  • Same password → same ciphertext (ECB pattern leakage)
  • Common passwords easily identified by frequency analysis

Impact:
  • Cross-site credential stuffing
  • $1.1 million settlement
  
Root cause: Encryption instead of hashing + ECB mode
Fix: Use Argon2id/bcrypt for passwords, never ECB mode
```

---

## 5. Detecting Weak Crypto in Code

```bash
# Search for weak algorithms in codebase
grep -rn "MD5\|md5\|SHA1\|sha1\|DES\|des\|RC4\|rc4" --include="*.py" --include="*.java" --include="*.js" .

# Specific patterns
grep -rn "hashlib.md5\|hashlib.sha1" --include="*.py" .
grep -rn "MessageDigest.getInstance.*MD5\|MessageDigest.getInstance.*SHA-1" --include="*.java" .
grep -rn "createHash.*md5\|createHash.*sha1" --include="*.js" .

# Check SSL/TLS configuration files
grep -rn "SSLv2\|SSLv3\|TLSv1\.0\|TLSv1\.1\|RC4\|DES\|3DES" --include="*.conf" --include="*.yml" .

# Semgrep rules for weak crypto
semgrep --config=p/owasp-top-ten --lang python .
```

```python
# Automated weak crypto scanner
import ast
import os

WEAK_PATTERNS = {
    'python': [
        'hashlib.md5', 'hashlib.sha1',
        'DES.new', 'DES3.new', 'ARC4.new',
        'Blowfish.new',
    ],
    'java': [
        'MessageDigest.getInstance("MD5")',
        'MessageDigest.getInstance("SHA-1")',
        'Cipher.getInstance("DES")',
    ]
}

def scan_file(filepath):
    findings = []
    with open(filepath, 'r', errors='ignore') as f:
        for i, line in enumerate(f, 1):
            for pattern in WEAK_PATTERNS.get('python', []):
                if pattern in line:
                    findings.append({
                        'file': filepath,
                        'line': i,
                        'pattern': pattern,
                        'severity': 'HIGH',
                        'recommendation': get_recommendation(pattern)
                    })
    return findings

def get_recommendation(pattern):
    recommendations = {
        'hashlib.md5': 'Use hashlib.sha256() or hashlib.sha3_256()',
        'hashlib.sha1': 'Use hashlib.sha256() or hashlib.sha3_256()',
        'DES.new': 'Use AES.new() with MODE_GCM',
        'DES3.new': 'Use AES.new() with MODE_GCM',
    }
    return recommendations.get(pattern, 'Use a modern algorithm')
```

---

## Summary Table

| Algorithm | Type | Weakness | Modern Alternative |
|-----------|------|----------|-------------------|
| **MD5** | Hash | Collision in seconds | SHA-256, SHA-3 |
| **SHA-1** | Hash | Collision demonstrated | SHA-256, SHA-3 |
| **DES** | Cipher | 56-bit key brute-forced | AES-256 |
| **3DES** | Cipher | Sweet32 (64-bit block) | AES-256 |
| **RC4** | Stream | Statistical biases | ChaCha20-Poly1305 |
| **RSA-1024** | Asymmetric | Factorable with effort | RSA-2048+ or ECDSA |
| **Blowfish** | Cipher | 64-bit block size | AES-256 |

---

## Revision Questions

1. Why is MD5 considered broken? Describe the collision attack and give a real-world example of its exploitation.
2. What is the Sweet32 attack against 3DES? Why does 64-bit block size create this vulnerability?
3. Compare SHA-1 and SHA-256 in terms of security, output size, and current status.
4. Explain the birthday attack and how it applies to hash collision resistance.
5. How would you scan a codebase for use of weak cryptographic algorithms? What tools would you use?
6. Describe the LinkedIn and Adobe breaches — what weak crypto was used and what should have been used instead?

---

*Previous: [03-data-at-rest-protection.md](03-data-at-rest-protection.md) | Next: [05-key-management.md](05-key-management.md)*

---

*[Back to README](../README.md)*
