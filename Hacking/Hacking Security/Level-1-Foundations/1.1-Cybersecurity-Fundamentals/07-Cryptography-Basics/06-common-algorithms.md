# Common Algorithms Overview

## Unit 7 - Topic 6 | Cryptography Basics

---

## Overview

This topic provides a comprehensive reference of the most important cryptographic algorithms, their characteristics, use cases, and current security status. Use this as a quick-reference guide for algorithm selection and security assessments.

---

## 1. Algorithm Categories

```
┌──────────────────────────────────────────────────────────────────┐
│                CRYPTOGRAPHIC ALGORITHMS                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────────┐  ┌───────────────────┐                   │
│  │ SYMMETRIC          │  │ ASYMMETRIC         │                   │
│  │ (Same key)         │  │ (Key pair)         │                   │
│  │ AES, ChaCha20,     │  │ RSA, ECC, DH,      │                   │
│  │ 3DES, Blowfish     │  │ ECDH, Ed25519      │                   │
│  └───────────────────┘  └───────────────────┘                   │
│                                                                  │
│  ┌───────────────────┐  ┌───────────────────┐                   │
│  │ HASHING            │  │ KEY EXCHANGE       │                   │
│  │ (One-way)          │  │ (Shared secret)    │                   │
│  │ SHA-256, SHA-3,    │  │ Diffie-Hellman,    │                   │
│  │ bcrypt, Argon2     │  │ ECDH               │                   │
│  └───────────────────┘  └───────────────────┘                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Master Algorithm Reference Table

### Symmetric Encryption

| Algorithm | Key Size | Block Size | Status | Notes |
|-----------|----------|-----------|--------|-------|
| **AES-128** | 128 bit | 128 bit | ✅ Secure | Standard for commercial use |
| **AES-256** | 256 bit | 128 bit | ✅ Secure | Government/military grade |
| **ChaCha20** | 256 bit | Stream | ✅ Secure | Mobile-optimized, used in TLS 1.3 |
| **3DES** | 168 bit | 64 bit | ❌ Deprecated | Retired 2023, do not use |
| **DES** | 56 bit | 64 bit | ❌ Broken | Cracked in 1999, never use |
| **Blowfish** | 32-448 bit | 64 bit | ⚠️ Aging | Replaced by Twofish |
| **Twofish** | 256 bit | 128 bit | ✅ Secure | AES finalist, good alternative |
| **RC4** | 40-2048 bit | Stream | ❌ Broken | Banned from TLS |

### Asymmetric Encryption / Signatures

| Algorithm | Key Size | Use | Status | Notes |
|-----------|----------|-----|--------|-------|
| **RSA** | 2048-4096 | Encrypt + Sign | ✅ Use 2048+ | Most widely used |
| **ECDSA** | 256-384 | Signatures | ✅ Modern | Smaller keys than RSA |
| **Ed25519** | 256 | Signatures | ✅ Best | Fast, modern, SSH default |
| **X25519** | 256 | Key exchange | ✅ Best | Used in TLS 1.3 |
| **DSA** | 2048-3072 | Signatures | ⚠️ Phasing out | Being replaced by ECDSA |
| **ElGamal** | 2048+ | Encrypt + Sign | ⚠️ Rare | Used in PGP |

### Hashing

| Algorithm | Output | Status | Use Case |
|-----------|--------|--------|----------|
| **SHA-256** | 256 bit | ✅ Standard | File integrity, certificates, blockchain |
| **SHA-384** | 384 bit | ✅ Secure | High-security applications |
| **SHA-512** | 512 bit | ✅ Secure | High-security applications |
| **SHA-3** | Variable | ✅ Secure | Alternative to SHA-2 |
| **BLAKE2** | Variable | ✅ Secure | Fast hashing |
| **bcrypt** | 184 bit | ✅ Secure | Password hashing |
| **Argon2** | Variable | ✅ Best | Password hashing (PHC winner) |
| **scrypt** | Variable | ✅ Secure | Password hashing, cryptocurrency |
| **MD5** | 128 bit | ❌ Broken | Never use for security |
| **SHA-1** | 160 bit | ❌ Broken | Deprecated everywhere |

### Key Exchange

| Algorithm | Type | Status | Notes |
|-----------|------|--------|-------|
| **Diffie-Hellman** | Classic | ✅ Use 2048+ | Original key exchange protocol |
| **ECDH** | Elliptic curve | ✅ Modern | Efficient key exchange |
| **X25519** | Curve25519 | ✅ Best | TLS 1.3 default |

---

## 3. Algorithm Selection Guide

```
┌──────────────────────────────────────────────────────────────┐
│              WHICH ALGORITHM TO USE?                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ENCRYPTING DATA AT REST:                                    │
│  └── AES-256-GCM                                             │
│                                                              │
│  ENCRYPTING DATA IN TRANSIT:                                 │
│  └── TLS 1.3 (AES-256-GCM or ChaCha20-Poly1305)             │
│                                                              │
│  STORING PASSWORDS:                                          │
│  └── Argon2id (best) or bcrypt (widely supported)            │
│                                                              │
│  FILE INTEGRITY:                                             │
│  └── SHA-256 or SHA-512                                      │
│                                                              │
│  DIGITAL SIGNATURES:                                         │
│  └── Ed25519 (best) or ECDSA P-256                           │
│                                                              │
│  SSH KEYS:                                                   │
│  └── Ed25519 (best) or RSA 4096                              │
│                                                              │
│  KEY EXCHANGE:                                               │
│  └── X25519 (best) or ECDH P-256                             │
│                                                              │
│  VPN:                                                        │
│  └── WireGuard (ChaCha20 + Curve25519) or IPsec (AES-GCM)   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Algorithms to AVOID

| Algorithm | Why | Replace With |
|-----------|-----|-------------|
| **DES** | 56-bit key cracked easily | AES |
| **3DES** | Deprecated, 64-bit block | AES |
| **RC4** | Multiple vulnerabilities | AES-GCM or ChaCha20 |
| **MD5** | Collision attacks trivial | SHA-256 |
| **SHA-1** | Collision attacks demonstrated | SHA-256 |
| **RSA < 2048** | Too small, can be factored | RSA 2048+ or ECC |
| **DH < 2048** | Vulnerable to Logjam attack | DH 2048+ or ECDH |

---

## 5. Post-Quantum Cryptography (Future)

```
┌──────────────────────────────────────────────────────────────┐
│             POST-QUANTUM THREAT                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Quantum computers will eventually break:                    │
│  ❌ RSA (Shor's algorithm)                                   │
│  ❌ ECC (Shor's algorithm)                                   │
│  ❌ Diffie-Hellman                                           │
│                                                              │
│  Quantum-safe algorithms (NIST PQC standardization):         │
│  ✅ CRYSTALS-Kyber (key exchange)                            │
│  ✅ CRYSTALS-Dilithium (signatures)                          │
│  ✅ FALCON (signatures)                                      │
│  ✅ SPHINCS+ (signatures)                                    │
│                                                              │
│  Symmetric (AES) and hashing (SHA-256) remain safe           │
│  with doubled key sizes (AES-256 vs quantum).                │
│                                                              │
│  ACTION: Start planning migration to PQC algorithms.         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Use Case | Recommended Algorithm | Avoid |
|----------|----------------------|-------|
| **Symmetric Encryption** | AES-256-GCM, ChaCha20 | DES, 3DES, RC4 |
| **Asymmetric Encryption** | RSA 2048+, ECC | RSA < 2048 |
| **Digital Signatures** | Ed25519, ECDSA | DSA |
| **Hashing (general)** | SHA-256, SHA-3 | MD5, SHA-1 |
| **Password Hashing** | Argon2id, bcrypt | MD5, SHA-256 (unsalted) |
| **Key Exchange** | X25519, ECDH | DH < 2048 |

---

## Quick Revision Questions

1. **Why is AES considered the gold standard for symmetric encryption?**
2. **What algorithms should NEVER be used and why?**
3. **For password storage, why use bcrypt/Argon2 instead of SHA-256?**
4. **What is the post-quantum threat and which algorithms are affected?**
5. **Which algorithm would you choose for SSH keys and why?**
6. **What is the recommended algorithm for each: encryption at rest, transit, passwords, signatures?**

---

[← Previous: PKI Fundamentals](05-pki-fundamentals.md)

---

*Unit 7 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Security Architecture →](../08-Security-Architecture/01-network-security-zones.md)*
