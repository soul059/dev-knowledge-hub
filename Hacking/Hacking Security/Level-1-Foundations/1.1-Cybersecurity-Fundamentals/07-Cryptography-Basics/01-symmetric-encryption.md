# Symmetric Encryption

## Unit 7 - Topic 1 | Cryptography Basics

---

## Overview

**Symmetric encryption** uses the **same key** for both encrypting and decrypting data. It is fast, efficient, and used for bulk data encryption — making it the backbone of data protection for files, disks, databases, and network communications.

---

## 1. How Symmetric Encryption Works

```
┌──────────────────────────────────────────────────────────────────┐
│                 SYMMETRIC ENCRYPTION                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ENCRYPTION:                                                     │
│  ┌───────────┐    ┌────────────┐    ┌───────────────┐           │
│  │ Plaintext │───►│  ENCRYPT   │───►│  Ciphertext   │           │
│  │ "Hello"   │    │  with KEY  │    │  "xK9#mQ..."  │           │
│  └───────────┘    └────────────┘    └───────────────┘           │
│                        🔑                                        │
│                   Same Key (K)                                   │
│                        🔑                                        │
│  DECRYPTION:                                                     │
│  ┌───────────────┐  ┌────────────┐  ┌───────────┐              │
│  │  Ciphertext   │─►│  DECRYPT   │─►│ Plaintext │              │
│  │  "xK9#mQ..."  │  │  with KEY  │  │ "Hello"   │              │
│  └───────────────┘  └────────────┘  └───────────┘              │
│                                                                  │
│  ONE key does BOTH encryption and decryption.                    │
│  Key must be kept SECRET and shared securely.                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Types of Symmetric Ciphers

### Block Ciphers vs Stream Ciphers

| Type | How It Works | Examples | Best For |
|------|-------------|---------|----------|
| **Block Cipher** | Encrypts fixed-size blocks (e.g., 128 bits) | AES, DES, 3DES, Blowfish | File encryption, disk encryption |
| **Stream Cipher** | Encrypts one bit/byte at a time | RC4, ChaCha20, Salsa20 | Real-time communication, streaming |

```
BLOCK CIPHER:
  Plaintext:  [Block1][Block2][Block3][Block4]...
                 │        │        │        │
              ┌──┴──┐  ┌──┴──┐  ┌──┴──┐  ┌──┴──┐
              │Encr.│  │Encr.│  │Encr.│  │Encr.│
              └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘
                 │        │        │        │
  Ciphertext: [Ciph1][Ciph2][Ciph3][Ciph4]...

STREAM CIPHER:
  Plaintext:  H  e  l  l  o  ...
              │  │  │  │  │
  Key Stream: ⊕  ⊕  ⊕  ⊕  ⊕   (XOR with key stream)
              │  │  │  │  │
  Ciphertext: x  K  9  #  m  ...
```

---

## 3. Common Symmetric Algorithms

| Algorithm | Key Size | Block Size | Status | Use Case |
|-----------|----------|-----------|--------|----------|
| **AES** | 128/192/256 bit | 128 bit | ✅ Current standard | Everything (files, disk, VPN, TLS) |
| **DES** | 56 bit | 64 bit | ❌ Broken (too short) | Legacy only — do NOT use |
| **3DES (Triple DES)** | 112/168 bit | 64 bit | ⚠️ Deprecated | Legacy migration |
| **Blowfish** | 32-448 bit | 64 bit | ⚠️ Aging | Some legacy applications |
| **Twofish** | 128/192/256 bit | 128 bit | ✅ Secure | AES alternative |
| **ChaCha20** | 256 bit | Stream | ✅ Modern | TLS (mobile), WireGuard VPN |
| **RC4** | 40-2048 bit | Stream | ❌ Broken | Do NOT use |

---

## 4. AES (Advanced Encryption Standard)

```
┌──────────────────────────────────────────────────────────────┐
│                    AES — THE GOLD STANDARD                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  • Adopted by NIST in 2001 (replaced DES)                    │
│  • Used by US government for classified data                 │
│  • Block size: 128 bits                                      │
│  • Key sizes: 128, 192, or 256 bits                          │
│                                                              │
│  Key Size  │  Rounds  │  Security Level                      │
│  ──────────┼──────────┼──────────────────                    │
│  AES-128   │    10    │  Strong (commercial)                 │
│  AES-192   │    12    │  Very Strong                         │
│  AES-256   │    14    │  Maximum (government/military)       │
│                                                              │
│  WHERE IT'S USED:                                            │
│  • TLS/HTTPS (web traffic)                                   │
│  • VPN (IPsec, WireGuard)                                    │
│  • Full disk encryption (BitLocker, FileVault, LUKS)         │
│  • File encryption (7-Zip, VeraCrypt)                        │
│  • Database encryption (TDE)                                 │
│  • Wi-Fi (WPA2/WPA3)                                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Block Cipher Modes of Operation

| Mode | Name | Description | Use Case |
|------|------|-------------|----------|
| **ECB** | Electronic Codebook | Each block encrypted independently | ❌ NEVER use (reveals patterns) |
| **CBC** | Cipher Block Chaining | Each block XORed with previous ciphertext | File encryption |
| **CTR** | Counter Mode | Block cipher acts like stream cipher | High-performance encryption |
| **GCM** | Galois/Counter Mode | CTR + authentication tag | ✅ **Best choice** — TLS, IPsec |
| **CCM** | Counter with CBC-MAC | CTR + CBC-MAC authentication | Wi-Fi (WPA2) |

---

## 6. The Key Distribution Problem

```
┌──────────────────────────────────────────────────────────────┐
│              THE KEY DISTRIBUTION PROBLEM                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Alice wants to send encrypted message to Bob.               │
│                                                              │
│  PROBLEM: How does Alice share the key with Bob securely?    │
│                                                              │
│  ❌ Send key over internet (can be intercepted)              │
│  ❌ Send key by email (not encrypted)                        │
│  ⚠️ Physical delivery (slow, impractical)                    │
│                                                              │
│  SOLUTION: Use ASYMMETRIC encryption to exchange the         │
│  symmetric key, then use symmetric for bulk data.            │
│  This is exactly how TLS/HTTPS works!                        │
│                                                              │
│  Alice ──[RSA encrypts AES key]──► Bob                       │
│  Then both use AES key for fast symmetric encryption         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **How it works** | Same key encrypts and decrypts |
| **Speed** | Very fast — suitable for bulk data |
| **Current Standard** | AES (128/256 bit) |
| **Types** | Block ciphers (AES) and Stream ciphers (ChaCha20) |
| **Best Mode** | AES-GCM (encryption + authentication) |
| **Key Problem** | How to share the key securely (solved by asymmetric crypto) |
| **Avoid** | DES, RC4, ECB mode |

---

## Quick Revision Questions

1. **What is the defining characteristic of symmetric encryption?**
2. **What is the difference between block ciphers and stream ciphers?**
3. **Why is DES no longer considered secure?**
4. **What makes AES-GCM the preferred mode for modern encryption?**
5. **Explain the key distribution problem and how it's solved.**
6. **Where is AES used in everyday technology?**

---

[Next: Asymmetric Encryption →](02-asymmetric-encryption.md)

---

*Unit 7 - Topic 1 of 6 | [Back to README](../README.md)*
