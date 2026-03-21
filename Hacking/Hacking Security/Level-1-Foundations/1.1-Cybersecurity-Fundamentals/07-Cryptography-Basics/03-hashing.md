# Hashing

## Unit 7 - Topic 3 | Cryptography Basics

---

## Overview

**Hashing** is a one-way mathematical function that converts input data of any size into a fixed-size output (called a **hash**, **digest**, or **fingerprint**). Unlike encryption, hashing is **irreversible** — you cannot recover the original data from the hash. Hashing is used for integrity verification, password storage, and digital signatures.

---

## 1. How Hashing Works

```
┌──────────────────────────────────────────────────────────────────┐
│                      HASHING                                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Input (any size)         Hash Function        Output (fixed)    │
│  ┌──────────────┐        ┌──────────┐        ┌────────────────┐ │
│  │ "Hello"      │───────►│  SHA-256 │───────►│ 2cf24dba5fb0.. │ │
│  └──────────────┘        └──────────┘        │ (256 bits)     │ │
│                                              └────────────────┘ │
│  ┌──────────────┐        ┌──────────┐        ┌────────────────┐ │
│  │ "Hello!"     │───────►│  SHA-256 │───────►│ 334d016f7556.. │ │
│  └──────────────┘        └──────────┘        │ (completely    │ │
│                                              │  different!)   │ │
│  KEY PROPERTIES:                             └────────────────┘ │
│  ✅ Same input = Always same output (deterministic)             │
│  ✅ One-way — cannot reverse hash to get input                  │
│  ✅ Fixed output size regardless of input size                  │
│  ✅ Tiny input change = completely different hash (avalanche)   │
│  ✅ Collision-resistant — hard to find two inputs with same hash│
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Hashing vs Encryption

| Aspect | Hashing | Encryption |
|--------|---------|-----------|
| **Direction** | One-way (irreversible) | Two-way (reversible with key) |
| **Purpose** | Verify integrity | Protect confidentiality |
| **Key Required** | No key | Key required |
| **Output Size** | Fixed (e.g., 256 bits) | Same size as input (roughly) |
| **Use Case** | Password storage, file integrity | Data protection, communication |
| **Can Recover Data?** | ❌ No | ✅ Yes (with key) |

---

## 3. Common Hash Algorithms

| Algorithm | Output Size | Status | Use Case |
|-----------|-----------|--------|----------|
| **MD5** | 128 bit | ❌ Broken (collisions found) | Legacy checksum only |
| **SHA-1** | 160 bit | ❌ Broken (collisions found) | Deprecated everywhere |
| **SHA-256** | 256 bit | ✅ Secure | File integrity, certificates, blockchain |
| **SHA-384** | 384 bit | ✅ Secure | High-security applications |
| **SHA-512** | 512 bit | ✅ Secure | High-security applications |
| **SHA-3** | Variable | ✅ Secure | Alternative to SHA-2 family |
| **bcrypt** | 184 bit | ✅ Secure | Password hashing (with salt + work factor) |
| **Argon2** | Variable | ✅ Best | Password hashing (winner of PHC) |
| **BLAKE2** | Variable | ✅ Secure | Fast hashing, file integrity |

---

## 4. Use Cases for Hashing

### Password Storage

```
┌──────────────────────────────────────────────────────────────┐
│              PASSWORD HASHING                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  WRONG ❌: Store passwords in plaintext                       │
│  Database: | user  | password    |                           │
│            | alice | MyP@ss123   |  ← If DB stolen, ALL      │
│            | bob   | S3cret!     |    passwords exposed!      │
│                                                              │
│  RIGHT ✅: Store salted hashes                                │
│  Database: | user  | salt       | hash                |      │
│            | alice | x9f2a...   | bcrypt(salt+pass)   |      │
│            | bob   | k3m1b...   | bcrypt(salt+pass)   |      │
│                                                              │
│  Login verification:                                         │
│  1. User enters password                                     │
│  2. System retrieves salt                                    │
│  3. Hash(salt + entered password)                            │
│  4. Compare with stored hash                                 │
│  5. Match ✅ = authenticated                                  │
│                                                              │
│  SALT: Random value added to password before hashing         │
│  Prevents rainbow table attacks (pre-computed hashes)        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### File Integrity Verification

```
Download a file and verify its integrity:

Website says SHA-256: a3f2b8c9d1e4f5a6b7c8d9e0f1a2b3c4...
You download the file and compute:
  sha256sum file.iso → a3f2b8c9d1e4f5a6b7c8d9e0f1a2b3c4...

Match ✅ = File is authentic, not tampered with
No Match ❌ = File modified or corrupted — DO NOT TRUST
```

### Digital Forensics

```
Evidence preservation:
1. Image the hard drive
2. Hash the image: SHA-256 = abc123...
3. Store hash in chain-of-custody log
4. Later: Re-hash the image
5. Compare: If hashes match → evidence integrity maintained ✅
```

---

## 5. Attacks on Hashes

| Attack | Description | Defense |
|--------|-------------|---------|
| **Brute Force** | Try every possible input | Use slow hash functions (bcrypt, Argon2) |
| **Dictionary Attack** | Try common passwords | Salt + slow hash |
| **Rainbow Table** | Pre-computed hash lookup table | Salt makes rainbow tables useless |
| **Collision Attack** | Find two inputs with same hash | Use SHA-256+ (collision-resistant) |
| **Length Extension** | Extend a hash without knowing the input | Use HMAC instead of plain hashing |

---

## 6. HMAC (Hash-based Message Authentication Code)

```
HMAC = Hash(Key + Message)

Combines hashing with a secret key to provide:
✅ Integrity (message not tampered)
✅ Authentication (sender has the key)

Used in: API authentication, JWT tokens, TLS
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Definition** | One-way function producing fixed-size output |
| **Key Property** | Irreversible — cannot recover input from hash |
| **Current Standard** | SHA-256 (general), bcrypt/Argon2 (passwords) |
| **Avoid** | MD5, SHA-1 (both broken) |
| **Salt** | Random value preventing rainbow table attacks |
| **HMAC** | Hash + secret key = integrity + authentication |

---

## Quick Revision Questions

1. **What is the difference between hashing and encryption?**
2. **Why are MD5 and SHA-1 considered broken?**
3. **What is a salt and why is it important for password hashing?**
4. **Why should passwords be stored with bcrypt/Argon2 instead of SHA-256?**
5. **How is hashing used in digital forensics?**
6. **What is HMAC and what two properties does it provide?**

---

[← Previous: Asymmetric Encryption](02-asymmetric-encryption.md) | [Next: Digital Signatures →](04-digital-signatures.md)

---

*Unit 7 - Topic 3 of 6 | [Back to README](../README.md)*
