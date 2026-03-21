# Asymmetric Encryption

## Unit 7 - Topic 2 | Cryptography Basics

---

## Overview

**Asymmetric encryption** (also called **public-key cryptography**) uses a **pair of mathematically related keys**: a **public key** (shared openly) and a **private key** (kept secret). What one key encrypts, only the other can decrypt. This solves the key distribution problem of symmetric encryption.

---

## 1. How Asymmetric Encryption Works

```
┌──────────────────────────────────────────────────────────────────┐
│                  ASYMMETRIC ENCRYPTION                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  KEY PAIR:  🔑 Public Key (share with everyone)                  │
│             🔐 Private Key (keep SECRET)                         │
│                                                                  │
│  FOR ENCRYPTION (Confidentiality):                               │
│  ┌───────────┐  Bob's Public Key  ┌───────────────┐             │
│  │ Plaintext │──────🔑──────────►│  Ciphertext   │             │
│  │ "Hello"   │     ENCRYPT        │  "xK9#mQ..."  │             │
│  └───────────┘                    └───────┬───────┘             │
│                                           │                     │
│                           Bob's Private Key│                     │
│                                    🔐      │                     │
│                                           ▼                     │
│                                   ┌───────────┐                 │
│                                   │ Plaintext │                 │
│                                   │ "Hello"   │                 │
│                                   └───────────┘                 │
│                                                                  │
│  ONLY Bob's private key can decrypt what Bob's public key        │
│  encrypted. Anyone can encrypt TO Bob, only Bob can READ it.     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Symmetric vs Asymmetric

| Feature | Symmetric | Asymmetric |
|---------|-----------|-----------|
| **Keys** | 1 shared key | 2 keys (public + private) |
| **Speed** | Very fast | Slow (1000x slower) |
| **Key Distribution** | Difficult (must share securely) | Easy (public key is... public) |
| **Use Case** | Bulk data encryption | Key exchange, digital signatures, small data |
| **Key Length** | 128-256 bits | 2048-4096 bits (RSA), 256 bits (ECC) |
| **Examples** | AES, ChaCha20 | RSA, ECC, Diffie-Hellman |
| **Analogy** | Same key for lock and unlock | Mailbox: anyone drops in, only owner opens |

---

## 3. Common Asymmetric Algorithms

| Algorithm | Key Size | Use Case | Status |
|-----------|----------|----------|--------|
| **RSA** | 2048-4096 bit | Encryption, signatures, key exchange | ✅ Widely used (use 2048+ bit) |
| **ECC** | 256-384 bit | Same as RSA but with smaller keys | ✅ Modern, efficient |
| **Diffie-Hellman** | 2048+ bit | Key exchange only (not encryption) | ✅ Used in TLS |
| **ECDH** | 256+ bit | ECC-based key exchange | ✅ Modern TLS |
| **Ed25519** | 256 bit | Digital signatures | ✅ Fast, modern (SSH keys) |
| **ElGamal** | 2048+ bit | Encryption and signatures | ⚠️ Less common |

### RSA vs ECC Key Size Comparison

| Security Level | RSA Key Size | ECC Key Size | Advantage |
|:-:|:-:|:-:|:-:|
| 80-bit | 1024 | 160 | ECC 6x smaller |
| 128-bit | 3072 | 256 | ECC 12x smaller |
| 192-bit | 7680 | 384 | ECC 20x smaller |
| 256-bit | 15360 | 521 | ECC 30x smaller |

ECC provides **equivalent security with much smaller keys** = faster, less bandwidth.

---

## 4. Two Uses of Asymmetric Encryption

```
┌──────────────────────────────────────────────────────────────────┐
│            TWO USES OF PUBLIC/PRIVATE KEYS                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  USE 1: ENCRYPTION (Confidentiality)                             │
│  ──────────────────────────────────                              │
│  Encrypt with RECIPIENT'S PUBLIC key                             │
│  Decrypt with RECIPIENT'S PRIVATE key                            │
│  → Only the recipient can read the message                       │
│                                                                  │
│  USE 2: DIGITAL SIGNATURE (Authentication + Integrity)           │
│  ─────────────────────────────────────────────────               │
│  Sign with SENDER'S PRIVATE key                                  │
│  Verify with SENDER'S PUBLIC key                                 │
│  → Proves the sender's identity + message wasn't modified        │
│                                                                  │
│  KEY INSIGHT:                                                    │
│  Encrypt with PUBLIC = only private key holder can read          │
│  Sign with PRIVATE = anyone with public key can verify           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5. How TLS Uses Both Symmetric and Asymmetric

```
┌──────────────────────────────────────────────────────────────────┐
│            TLS HANDSHAKE (Simplified)                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Client                                    Server                │
│    │                                          │                  │
│    │ 1. "Hello, I support these ciphers"      │                  │
│    │─────────────────────────────────────────►│                  │
│    │                                          │                  │
│    │ 2. "Here's my certificate (public key)"  │                  │
│    │◄─────────────────────────────────────────│                  │
│    │                                          │                  │
│    │ 3. Generate random AES session key       │  ASYMMETRIC      │
│    │    Encrypt with server's PUBLIC key       │  (Key exchange)  │
│    │─────────────────────────────────────────►│                  │
│    │                                          │                  │
│    │ 4. Server decrypts with PRIVATE key      │                  │
│    │    Both now have the AES session key      │                  │
│    │                                          │                  │
│    │ 5. All further data encrypted with AES   │  SYMMETRIC       │
│    │◄════════════════════════════════════════►│  (Bulk data)     │
│    │         Fast symmetric encryption        │                  │
│    │                                          │                  │
│  HYBRID APPROACH: Asymmetric for key exchange,                   │
│  Symmetric for actual data — best of both worlds!                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **How it works** | Two keys: public (encrypt/verify) + private (decrypt/sign) |
| **Speed** | Slow — used for key exchange, not bulk data |
| **Current Standard** | RSA (2048+), ECC (256+) |
| **Solves** | Key distribution problem of symmetric encryption |
| **Two Uses** | Encryption (confidentiality) + Digital signatures (auth + integrity) |
| **In Practice** | Hybrid with symmetric (TLS handshake) |

---

## Quick Revision Questions

1. **What is the key difference between symmetric and asymmetric encryption?**
2. **Why is asymmetric encryption slower than symmetric?**
3. **Explain how TLS uses BOTH symmetric and asymmetric encryption.**
4. **Why is ECC considered more efficient than RSA?**
5. **If you encrypt with someone's public key, who can decrypt it?**
6. **If you sign with your private key, what does the recipient's verification prove?**

---

[← Previous: Symmetric Encryption](01-symmetric-encryption.md) | [Next: Hashing →](03-hashing.md)

---

*Unit 7 - Topic 2 of 6 | [Back to README](../README.md)*
