# 2.5 Cryptographic Hash Functions

## Unit 2: Hash Functions

---

## What Are Cryptographic Hash Functions?

**Cryptographic hash functions** are a special class of hash functions designed for security applications. Unlike standard hash functions that prioritize speed and distribution, cryptographic hashes also guarantee that it's computationally infeasible to reverse the process or find collisions intentionally.

```
╔══════════════════════════════════════════════════════════════╗
║          CRYPTOGRAPHIC vs STANDARD HASH FUNCTIONS           ║
║                                                             ║
║   Standard Hash:     Speed + Distribution                   ║
║   Crypto Hash:       Speed + Distribution + SECURITY        ║
║                                                             ║
║   Standard:  h("password123") = 7                           ║
║              Given 7, can we find input? Maybe! (small      ║
║              output space → brute force is feasible)        ║
║                                                             ║
║   Crypto:    SHA256("password123") = ef92b778ba...          ║
║              Given ef92b778ba..., can we find input?        ║
║              Computationally INFEASIBLE! (2^256 space)      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Key Properties

```
╔══════════════════════════════════════════════════════════════════╗
║       CRYPTOGRAPHIC HASH FUNCTION PROPERTIES                    ║
║                                                                 ║
║   1. PRE-IMAGE RESISTANCE (One-way)                            ║
║      Given h(x), cannot find x                                 ║
║      hash → ??? → impossible to recover original              ║
║                                                                 ║
║   2. SECOND PRE-IMAGE RESISTANCE                               ║
║      Given x₁, cannot find x₂ ≠ x₁ where h(x₁) = h(x₂)     ║
║      Can't find another input with same hash                   ║
║                                                                 ║
║   3. COLLISION RESISTANCE                                      ║
║      Cannot find ANY pair (x₁, x₂) where h(x₁) = h(x₂)      ║
║      Even with birthday attack, should take ~2^(n/2) tries    ║
║                                                                 ║
║   4. AVALANCHE EFFECT (strict)                                 ║
║      1 bit change in input → ~50% bits change in output       ║
║      h("cat")  = a1b2c3d4e5f6...                              ║
║      h("bat")  = 7f8e9d0c1b2a...  (completely different)      ║
║                                                                 ║
║   5. DETERMINISTIC                                              ║
║      Same input always produces same hash                      ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## Common Cryptographic Hash Functions

| Algorithm | Output Size | Status | Speed |
|-----------|:-----------:|--------|:-----:|
| **MD5** | 128 bits | Broken (collisions found) | Fast |
| **SHA-1** | 160 bits | Broken (collisions found) | Fast |
| **SHA-256** | 256 bits | Secure | Moderate |
| **SHA-512** | 512 bits | Secure | Moderate |
| **SHA-3** | Variable | Secure | Moderate |
| **BLAKE3** | 256 bits | Secure | Very Fast |

```
╔════════════════════════════════════════════════════════════╗
║                    SHA-256 EXAMPLE                        ║
║                                                           ║
║   Input: "Hello"                                          ║
║   SHA-256: 185f8db32271fe25f561a6fc938b2e264306ec304eda   ║
║            386d8e20e5e4b7e97d5f1                          ║
║                                                           ║
║   Input: "Hello!"  (just added !)                         ║
║   SHA-256: 334d016f755cd6dc58c53a86e183882f8ec14f52fb05   ║
║            77d1c4ab7aa413ff34b9                            ║
║                                                           ║
║   Completely different output for tiny input change!      ║
╚════════════════════════════════════════════════════════════╝
```

---

## Applications in Computer Science

```
╔════════════════════════════════════════════════════════════════╗
║              APPLICATIONS OF CRYPTO HASHES                    ║
║                                                               ║
║   1. PASSWORD STORAGE                                         ║
║      Store hash(password), not password itself                ║
║      Login: compare hash(input) with stored hash             ║
║                                                               ║
║   2. DATA INTEGRITY                                           ║
║      File downloads: verify hash matches expected             ║
║      Git commits: SHA-1 identifies every commit              ║
║                                                               ║
║   3. DIGITAL SIGNATURES                                       ║
║      Hash document → sign the hash (not the full doc)        ║
║                                                               ║
║   4. BLOCKCHAIN                                               ║
║      Each block contains hash of previous block              ║
║      Changing any block breaks the chain                     ║
║                                                               ║
║   5. DEDUPLICATION                                            ║
║      Cloud storage: hash file → check if hash exists         ║
║      Same hash = same file (with high probability)           ║
╚════════════════════════════════════════════════════════════════╝
```

---

## Crypto Hashes vs Hash Table Hashes

| Feature | Hash Table Hash | Cryptographic Hash |
|---------|:---------------:|:-----------------:|
| Speed | **Very fast** (O(1)) | Slower (by design) |
| Output size | Small (table index) | Large (128-512 bits) |
| Collision resistance | Minimal | **Strong** |
| Reversibility | Not a concern | **Must be infeasible** |
| Use case | Data structures | Security |
| Example | `k % m` | SHA-256, BLAKE3 |

> **Key Point**: Don't use crypto hashes for hash tables (too slow) and don't use table hashes for security (too weak).

---

## The Birthday Paradox

```
╔═══════════════════════════════════════════════════════════════╗
║                  BIRTHDAY PARADOX                            ║
║                                                              ║
║   In a group of 23 people, there's a >50% chance            ║
║   that two share a birthday (out of 365 days).              ║
║                                                              ║
║   For hash functions with n-bit output:                      ║
║   Expected collisions after ~2^(n/2) hashes                 ║
║                                                              ║
║   Hash Size    │  Collision after ~                          ║
║   ─────────────┼──────────────────                           ║
║   32 bits      │  ~65,000 hashes (2^16)                     ║
║   64 bits      │  ~4 billion (2^32)                         ║
║   128 bits     │  ~18 quintillion (2^64)                    ║
║   256 bits     │  ~3.4 × 10^38 (2^128)                     ║
║                                                              ║
║   This is why MD5 (128-bit) is broken — 2^64 is             ║
║   now computationally feasible.                              ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: Why shouldn't you use SHA-256 as a hash function for a hash table? And why shouldn't you use `h(k) = k % m` for password storage?**

<details>
<summary>Click to reveal answer</summary>

**SHA-256 for hash tables**: SHA-256 is designed to be computationally expensive (relatively) to resist brute-force attacks. In a hash table, the hash function is called on every insert, search, and delete — it must be as fast as possible. Using SHA-256 would add unnecessary overhead, making O(1) operations significantly slower in practice.

**k % m for passwords**: The modulo hash function produces a tiny output space (0 to m-1, typically a few thousand values). An attacker could trivially brute-force all possible hash values. It also lacks pre-image resistance — many inputs map to the same output, and the function gives no security guarantees. Real password hashing uses algorithms like bcrypt or Argon2 that are intentionally slow and include salt values.

</details>

---

| [← Previous: Universal Hashing](04-universal-hashing.md) | [Next: String Hashing →](06-string-hashing.md) |
|:---|---:|
