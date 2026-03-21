# Unit 2: A02 - Cryptographic Failures — Topic 7: Prevention Strategies

## Overview

Preventing cryptographic failures requires a systematic approach: **classify data, choose the right algorithms, implement correctly using vetted libraries, manage keys securely, and verify through testing**. This topic provides a comprehensive crypto implementation guide, library recommendations, common implementation mistakes, a code review checklist, and compliance mapping to PCI DSS and NIST standards.

---

## 1. Cryptographic Decision Framework

```
┌─────────────────────────────────────────────────────────────────┐
│             CHOOSING THE RIGHT CRYPTO CONTROL                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  What do you need?                                             │
│  │                                                              │
│  ├─▶ CONFIDENTIALITY (hide data)                               │
│  │   ├─▶ Data at rest → AES-256-GCM                           │
│  │   ├─▶ Data in transit → TLS 1.3                             │
│  │   └─▶ Password storage → Argon2id (hash, NOT encrypt)      │
│  │                                                              │
│  ├─▶ INTEGRITY (detect tampering)                              │
│  │   ├─▶ Message authentication → HMAC-SHA256                  │
│  │   ├─▶ File integrity → SHA-256 checksums                   │
│  │   └─▶ Data + confidentiality → AES-GCM (AEAD)             │
│  │                                                              │
│  ├─▶ AUTHENTICATION (prove identity)                           │
│  │   ├─▶ Digital signatures → RSA-2048/Ed25519                │
│  │   ├─▶ API authentication → HMAC-SHA256 signatures          │
│  │   └─▶ Mutual authentication → mTLS                        │
│  │                                                              │
│  └─▶ KEY EXCHANGE (establish shared secret)                    │
│      ├─▶ TLS → ECDHE (X25519 or P-256)                       │
│      └─▶ Application → Diffie-Hellman or KMS                  │
└─────────────────────────────────────────────────────────────────┘
```

### Quick Reference: Recommended Algorithms

| Purpose | Algorithm | Min Key Size | Library |
|---------|-----------|-------------|---------|
| **Symmetric encryption** | AES-256-GCM | 256-bit | OpenSSL, libsodium |
| **Symmetric (alt)** | ChaCha20-Poly1305 | 256-bit | libsodium, NaCl |
| **Asymmetric encryption** | RSA-OAEP | 2048-bit | OpenSSL |
| **Digital signatures** | Ed25519 | 256-bit (fixed) | libsodium |
| **Digital signatures (alt)** | RSA-PSS | 2048-bit | OpenSSL |
| **Hashing** | SHA-256 / SHA-3 | — | Built-in |
| **Password hashing** | Argon2id | — | argon2-cffi |
| **Password hashing (alt)** | bcrypt | — | bcrypt |
| **HMAC** | HMAC-SHA256 | 256-bit | Built-in |
| **Key exchange** | X25519 / ECDHE | — | libsodium, OpenSSL |
| **Random generation** | CSPRNG | — | `os.urandom()` / `secrets` |

---

## 2. Recommended Crypto Libraries

| Language | Library | Notes |
|----------|---------|-------|
| **Python** | `cryptography` | Comprehensive, well-maintained |
| **Python** | `PyNaCl` / `libsodium` | Simple, misuse-resistant |
| **Python** | `argon2-cffi` | Password hashing |
| **Java** | Bouncy Castle | Comprehensive crypto |
| **Java** | Tink (Google) | Misuse-resistant, modern |
| **Node.js** | `crypto` (built-in) | Standard Node.js crypto |
| **Node.js** | `sodium-native` | libsodium bindings |
| **Go** | `crypto/*` (stdlib) | Built-in, high quality |
| **Rust** | `ring` | Fast, safe |
| **C/C++** | OpenSSL / LibreSSL | Industry standard |
| **C/C++** | libsodium (NaCl) | Easy-to-use, safe defaults |

### Using libsodium (Recommended — Hard to Misuse)

```python
# libsodium via PyNaCl — simple and safe
import nacl.secret
import nacl.utils

# Symmetric encryption (XSalsa20-Poly1305)
key = nacl.utils.random(nacl.secret.SecretBox.KEY_SIZE)  # 32 bytes
box = nacl.secret.SecretBox(key)

# Encrypt (nonce generated automatically)
encrypted = box.encrypt(b"sensitive data")

# Decrypt
plaintext = box.decrypt(encrypted)
```

---

## 3. Common Implementation Mistakes

### Top 10 Crypto Mistakes

| # | Mistake | Correct Approach |
|---|---------|-----------------|
| 1 | **ECB mode** — patterns visible in ciphertext | Use GCM or CTR mode |
| 2 | **Reusing nonces/IVs** — breaks encryption | Generate unique nonce per operation |
| 3 | **Encrypt without authenticate** — CBC without HMAC | Use AEAD (GCM, Poly1305) |
| 4 | **Hardcoded keys** | Use KMS/Vault |
| 5 | **Using `random` instead of `secrets`** | Always use CSPRNG |
| 6 | **Custom crypto algorithms** | Use vetted standards |
| 7 | **Comparing hashes with `==`** | Use constant-time comparison |
| 8 | **No key rotation** | Rotate annually minimum |
| 9 | **Passwords encrypted, not hashed** | Use Argon2id/bcrypt |
| 10 | **Certificate verification disabled** | Always verify (default) |

### ECB Mode Weakness

```
ECB Mode (NEVER use):
  Same plaintext block → Same ciphertext block
  
  Plaintext:  [Block A][Block A][Block B][Block A]
  Ciphertext: [XXXX]   [XXXX]   [YYYY]   [XXXX]
              ↑ Same!            ↑ Diff    ↑ Same!
  
  Patterns in plaintext visible in ciphertext!
  Famous example: ECB penguin (encrypted image still shows penguin shape)

CBC/GCM Mode (Use these):
  Each block depends on previous (chained)
  Same plaintext → Different ciphertext each time
```

### Timing Attack Prevention

```python
# ❌ VULNERABLE to timing attack
def verify_signature(expected, actual):
    return expected == actual  # Short-circuits on first mismatch
    # Attacker can determine correct bytes one at a time

# ✅ SECURE — Constant-time comparison
import hmac
def verify_signature(expected, actual):
    return hmac.compare_digest(expected, actual)  # Always takes same time
```

---

## 4. Crypto Code Review Checklist

```
CRYPTOGRAPHIC CODE REVIEW CHECKLIST
════════════════════════════════════

ALGORITHM SELECTION:
□ No MD5, SHA-1, DES, 3DES, RC4, Blowfish
□ Symmetric: AES-256-GCM or ChaCha20-Poly1305
□ Asymmetric: RSA-2048+ or Ed25519/ECDSA P-256+
□ Hashing: SHA-256+ or SHA-3
□ Password hashing: Argon2id or bcrypt (NOT SHA/MD5)
□ Key exchange: ECDHE (X25519 or P-256)

IMPLEMENTATION:
□ No ECB mode — use GCM, CTR, or authenticated modes
□ Unique nonce/IV per encryption operation
□ Authenticated encryption (AEAD) — not encrypt-then-MAC
□ Constant-time comparison for signatures/MACs
□ CSPRNG for all random values (os.urandom, secrets)
□ No custom/homegrown crypto algorithms

KEY MANAGEMENT:
□ Keys not hardcoded in source code
□ Keys not committed to version control
□ Keys stored in KMS/HSM/Vault
□ Key rotation policy implemented
□ Separate keys per environment (dev/staging/prod)
□ Minimum key sizes met (AES-256, RSA-2048)

TLS CONFIGURATION:
□ TLS 1.2+ only (no SSL, TLS 1.0, TLS 1.1)
□ Strong cipher suites (ECDHE + AES-GCM)
□ HSTS header with includeSubDomains
□ Certificate validation NOT disabled
□ OCSP stapling enabled
□ Forward secrecy (ECDHE key exchange)

DATA PROTECTION:
□ Sensitive data classified and identified
□ PII/PCI encrypted at rest
□ All external communication over TLS
□ Passwords hashed with Argon2id/bcrypt
□ No sensitive data in URLs/query strings
□ No sensitive data in logs

COMPLIANCE:
□ PCI DSS requirements for payment data
□ GDPR requirements for personal data
□ HIPAA requirements for health data
□ FIPS 140-2 if required (government)
□ Regular vulnerability scans for crypto issues
```

---

## 5. Compliance Mapping

### PCI DSS Cryptographic Requirements

| PCI DSS Req | Control | Implementation |
|-------------|---------|---------------|
| **3.4** | Render PAN unreadable | AES-256, SHA-256 hash with salt, tokenization |
| **3.5** | Protect cryptographic keys | HSM, KMS, split knowledge, dual control |
| **3.6** | Key management procedures | Annual rotation, documented procedures |
| **4.1** | Strong cryptography for transmission | TLS 1.2+, no WEP/SSL |
| **6.5.3** | Protect against insecure crypto | Code review, SAST scanning |

### NIST SP 800-175B Recommendations

```
NIST Approved Algorithms (2024):

Symmetric Encryption:
  ✅ AES (128, 192, 256 bit)
  
Hashing:
  ✅ SHA-2 (SHA-256, SHA-384, SHA-512)
  ✅ SHA-3 (SHA3-256, SHA3-384, SHA3-512)
  
Digital Signatures:
  ✅ RSA (2048+ bit)
  ✅ ECDSA (P-256, P-384, P-521)
  ✅ EdDSA (Ed25519, Ed448)
  
Key Exchange:
  ✅ DH (2048+ bit)
  ✅ ECDH (P-256, P-384)
  ✅ X25519, X448

Message Authentication:
  ✅ HMAC (with SHA-2/SHA-3)
  ✅ CMAC (with AES)
  ✅ GMAC (with AES-GCM)
```

---

## 6. Testing and Verification

### Automated Crypto Testing

```bash
# SAST for crypto issues
semgrep --config=p/secrets .                    # Find hardcoded secrets
semgrep --config=p/owasp-top-ten .              # OWASP checks
semgrep --config=r/python.cryptography .        # Python crypto checks

# TLS configuration testing
testssl.sh https://target.com
sslyze --regular target.com

# Check for weak crypto in dependencies
pip-audit                                        # Python dependencies
npm audit                                        # Node.js dependencies
trivy fs .                                       # Multi-language scanner

# Password hash analysis (detect weak hashing in database dumps)
hashid 'hash_value'                              # Identify hash type
hashcat -m 0 hashes.txt wordlist.txt             # MD5 (if found)
```

### Crypto Verification Script

```python
#!/usr/bin/env python3
"""Verify cryptographic configuration of a web application"""
import ssl
import socket
import subprocess
import json

def check_tls_config(hostname, port=443):
    """Check TLS configuration"""
    results = {'hostname': hostname, 'issues': []}
    
    context = ssl.create_default_context()
    
    try:
        with socket.create_connection((hostname, port), timeout=10) as sock:
            with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                cert = ssock.getpeercert()
                protocol = ssock.version()
                cipher = ssock.cipher()
                
                # Check protocol version
                if protocol in ('SSLv2', 'SSLv3', 'TLSv1', 'TLSv1.1'):
                    results['issues'].append(f'Weak protocol: {protocol}')
                
                # Check cipher
                if cipher and ('RC4' in cipher[0] or 'DES' in cipher[0] or 'NULL' in cipher[0]):
                    results['issues'].append(f'Weak cipher: {cipher[0]}')
                
                # Check certificate expiry
                import datetime
                not_after = datetime.datetime.strptime(cert['notAfter'], '%b %d %H:%M:%S %Y %Z')
                if not_after < datetime.datetime.utcnow():
                    results['issues'].append('Certificate EXPIRED')
                elif (not_after - datetime.datetime.utcnow()).days < 30:
                    results['issues'].append(f'Certificate expires in {(not_after - datetime.datetime.utcnow()).days} days')
                
                results['protocol'] = protocol
                results['cipher'] = cipher[0] if cipher else 'unknown'
                results['expires'] = cert.get('notAfter', 'unknown')
                
    except ssl.SSLError as e:
        results['issues'].append(f'SSL Error: {e}')
    except Exception as e:
        results['issues'].append(f'Connection Error: {e}')
    
    return results

# Test
result = check_tls_config('example.com')
if result['issues']:
    print(f"[!] Issues found for {result['hostname']}:")
    for issue in result['issues']:
        print(f"    - {issue}")
else:
    print(f"[✓] {result['hostname']}: TLS config looks good")
    print(f"    Protocol: {result['protocol']}, Cipher: {result['cipher']}")
```

---

## Summary Table

| Strategy | Implementation | Priority |
|----------|---------------|----------|
| **Classify data** | Identify and label all sensitive data | Critical |
| **Use modern algorithms** | AES-256-GCM, SHA-256, Argon2id | Critical |
| **Use vetted libraries** | cryptography, libsodium, Tink | Critical |
| **Manage keys properly** | KMS/HSM, no hardcoding, rotation | Critical |
| **Enforce TLS** | TLS 1.2+, HSTS, strong ciphers | Critical |
| **Validate certificates** | Never disable verification | High |
| **Avoid common mistakes** | No ECB, no nonce reuse, CSPRNG | High |
| **Code review** | Crypto-focused review checklist | Medium |
| **Compliance** | Map to PCI DSS/NIST/GDPR | Medium |
| **Test regularly** | SAST, TLS scanning, pen testing | Ongoing |

---

## Revision Questions

1. Given a new web application, walk through the complete decision framework for choosing cryptographic controls for each data type.
2. What are the top 5 most common cryptographic implementation mistakes? How would you detect each in a code review?
3. Compare libsodium and OpenSSL as crypto libraries. When would you choose each?
4. Why is ECB mode dangerous? Show with a diagram how patterns leak through ECB encryption.
5. Create a comprehensive crypto code review checklist organized by category (algorithms, keys, TLS, data).
6. How do PCI DSS and NIST SP 800-175B crypto requirements overlap? Create a mapping table.

---

*Previous: [06-certificate-validation.md](06-certificate-validation.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
