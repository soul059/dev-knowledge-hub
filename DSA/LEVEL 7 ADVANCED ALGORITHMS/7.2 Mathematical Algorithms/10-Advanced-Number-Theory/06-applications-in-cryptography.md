# Chapter 6: Applications in Cryptography

[← Previous: Miller-Rabin Primality Test](05-miller-rabin-test.md) | [Back to README](../README.md)

---

## Overview

Modern cryptography is built on number theory. This chapter surveys how the mathematical tools from this entire course — primes, modular arithmetic, exponentiation, Euler's totient, and primality testing — combine to form real-world cryptographic systems.

---

## 6.1 The RSA Cryptosystem

```
  ┌──────────────────────────────────────────────────────────┐
  │  KEY GENERATION                                          │
  │  1. Choose two large primes p, q (e.g., 1024 bits each) │
  │  2. Compute n = p × q                                   │
  │  3. Compute φ(n) = (p-1)(q-1)                           │
  │  4. Choose e such that gcd(e, φ(n)) = 1                │
  │     (commonly e = 65537)                                 │
  │  5. Compute d = e^(-1) mod φ(n)  (extended GCD)         │
  │                                                          │
  │  Public key:  (e, n)                                     │
  │  Private key: (d, n)                                     │
  └──────────────────────────────────────────────────────────┘
  
  ENCRYPTION:   ciphertext = message^e mod n
  DECRYPTION:   message    = ciphertext^d mod n
  
  Why it works:
    m^(e×d) = m^(1 + k×φ(n)) = m × (m^φ(n))^k ≡ m × 1^k = m (mod n)
    (by Euler's theorem, since gcd(m,n) = 1)
```

### Trace: Toy RSA

```
  p = 11, q = 13
  n = 143
  φ(n) = 10 × 12 = 120
  
  Choose e = 7   (gcd(7, 120) = 1 ✓)
  d = 7^(-1) mod 120
  
  extGCD(7, 120):  7×103 + 120×(-6) = 1
  d = 103
  
  Encrypt message m = 9:
    c = 9^7 mod 143
    9^1  = 9
    9^2  = 81
    9^4  = 81² = 6561 mod 143 = 132
    9^7  = 9^4 × 9^2 × 9^1 = 132 × 81 × 9 mod 143
         = 132 × 81 = 10692 mod 143 = 114
           114 × 9 = 1026 mod 143 = 25
    c = 25
  
  Decrypt:
    m = 25^103 mod 143 = 9  ✓
    (use fast exponentiation)
  
  Security relies on: factoring n = p×q is HARD for large n
```

### Number Theory Concepts Used

```
  ┌──────────────────────────────────────────────────────────┐
  │  Concept                 │ Where in RSA                 │
  ├──────────────────────────┼──────────────────────────────┤
  │  Prime numbers           │ Choosing p, q               │
  │  Primality testing (M-R) │ Verifying p, q are prime    │
  │  Euler's totient         │ Computing φ(n) = (p-1)(q-1) │
  │  Extended GCD            │ Computing d = e^(-1)         │
  │  Fast exponentiation     │ Encryption & decryption     │
  │  Euler's theorem         │ Correctness proof           │
  │  CRT                     │ Faster decryption           │
  └──────────────────────────┴──────────────────────────────┘
```

---

## 6.2 Diffie-Hellman Key Exchange

```
  ┌──────────────────────────────────────────────────────────┐
  │  Allows two parties to agree on a shared secret         │
  │  over an insecure channel.                               │
  │                                                          │
  │  Setup: public prime p, generator g                      │
  │                                                          │
  │  Alice                    Bob                            │
  │  ─────                    ───                            │
  │  choose secret a          choose secret b                │
  │  compute A = g^a mod p    compute B = g^b mod p          │
  │                                                          │
  │          ──── send A ────▶                               │
  │          ◀──── send B ────                               │
  │                                                          │
  │  shared = B^a mod p       shared = A^b mod p             │
  │         = g^(ab) mod p           = g^(ab) mod p          │
  │                                                          │
  │  Both compute the SAME shared secret!                   │
  └──────────────────────────────────────────────────────────┘
  
  Security relies on: Discrete Logarithm Problem (DLP)
    Given g, p, and g^a mod p, finding a is computationally hard.
  
  Number theory used: modular exponentiation, primitive roots
```

### Trace

```
  p = 23, g = 5
  
  Alice: a = 6,  A = 5^6 mod 23 = 15625 mod 23 = 8
  Bob:   b = 15, B = 5^15 mod 23 = ... = 19
  
  Alice: shared = 19^6 mod 23 = 2
  Bob:   shared = 8^15 mod 23 = 2  ✓
  
  Both have shared secret = 2
```

---

## 6.3 Digital Signatures (RSA-based)

```
  Uses RSA in reverse:
  
  SIGNING (with private key d):
    signature = hash(message)^d mod n
  
  VERIFICATION (with public key e):
    recovered = signature^e mod n
    Check: recovered == hash(message)?
  
  ┌──────────────────────────────────────────────────┐
  │  Only the private key holder can create valid    │
  │  signatures, but anyone with the public key     │
  │  can verify them.                                │
  └──────────────────────────────────────────────────┘
```

---

## 6.4 Hash Functions & Number Theory

```
  Polynomial rolling hash:
    H(s) = (s[0]×p^(n-1) + s[1]×p^(n-2) + ... + s[n-1]) mod m
  
  Number theory connections:
  - Choose m as a large prime (reduces collisions)
  - Choose p coprime to m
  - Use modular arithmetic for computation
  - Multi-mod hashing + CRT for collision resistance
  
  ┌──────────────────────────────────────────────────┐
  │  Birthday Paradox:                               │
  │  With m possible hash values, expect collision   │
  │  after ≈ √m values.                             │
  │                                                  │
  │  For m = 10^9+7:  collision after ~31623 strings │
  │  Double hashing (two mods): √(10^18) ≈ 10^9     │
  │  → Use at least 2 mod values in CP!             │
  └──────────────────────────────────────────────────┘
```

---

## 6.5 Discrete Logarithm Problem

```
  Given g, h, p (prime), find x such that g^x ≡ h (mod p)
  
  Baby-Step Giant-Step Algorithm:
  
  Let m = ⌈√(p-1)⌉
  
  Baby steps: compute g^j mod p for j = 0, 1, ..., m-1
              Store in hash table: {g^j → j}
  
  Giant steps: compute h × (g^(-m))^i mod p for i = 0, 1, ...
               If found in table → x = i×m + j
  
  Time: O(√p)   Space: O(√p)
  
  ★ This is feasible for small p but infeasible for cryptographic sizes
    (p ≈ 2^2048 → √p ≈ 2^1024 steps)
```

---

## 6.6 Pollard's Rho Factorisation

```
  Finds a non-trivial factor of composite n.
  
  Idea: random walk in Z_n with collision detection.
  
  FUNCTION pollardRho(n):
      x = random(2, n-1)
      y = x
      c = random(1, n-1)
      d = 1
      
      WHILE d == 1:
          x = (x² + c) mod n           // tortoise
          y = (y² + c) mod n           // hare (two steps)
          y = (y² + c) mod n
          d = gcd(|x - y|, n)
      
      IF d ≠ n: RETURN d               // found factor!
      ELSE: retry with different c
  
  Expected time: O(n^(1/4))
  
  Combined with Miller-Rabin:
  1. Test if n is prime (M-R)
  2. If composite, find factor with Pollard's rho
  3. Recurse on factors
  → Complete factorisation in expected O(n^(1/4) × log)
```

---

## 6.7 Number Theory in Competitive Programming Cryptography Problems

```
  ┌──────────────────────────────────────────────────────────┐
  │  Common CP problem types involving crypto concepts:      │
  │                                                          │
  │  1. "Compute x^(huge tower) mod m"                      │
  │     → Euler's theorem + recursive φ reduction           │
  │                                                          │
  │  2. "Given a^e mod n = c, find a"                       │
  │     → RSA decryption: compute d, then c^d mod n         │
  │                                                          │
  │  3. "Find x: g^x ≡ h (mod p)"                          │
  │     → Baby-step giant-step                               │
  │                                                          │
  │  4. "Factor n given some constraints"                    │
  │     → Pollard's rho or algebraic structure              │
  │                                                          │
  │  5. "Reconstruct number from remainders"                │
  │     → Chinese Remainder Theorem                          │
  │                                                          │
  │  6. "Find primitive root / order of element"             │
  │     → Euler's totient + factorisation of φ(p)           │
  └──────────────────────────────────────────────────────────┘
```

---

## 6.8 Course Recap: Connecting All Topics

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                                                                 │
  │  Unit 1: Number Theory Basics ──┐                              │
  │  Unit 2: Prime Numbers ─────────┼─▶ Foundation                 │
  │  Unit 3: GCD and LCM ──────────┘                               │
  │                                                                 │
  │  Unit 4: Modular Arithmetic ────┐                              │
  │  Unit 5: Fast Exponentiation ───┼─▶ Core Techniques            │
  │                                                                 │
  │  Unit 6: Combinatorics ─────────┼─▶ Counting                   │
  │  Unit 7: Probability ──────────┘                               │
  │                                                                 │
  │  Unit 8: Geometry Basics ───────┐                              │
  │  Unit 9: Computational Geometry ┼─▶ Spatial Algorithms         │
  │                                                                 │
  │  Unit 10: Advanced Number Theory ─▶ Synthesis & Applications   │
  │    Fermat → CRT → Euler → Inverse → Miller-Rabin → Crypto     │
  │                                                                 │
  │  Everything connects:                                           │
  │  Primes → Factorisation → Totient → Exponentiation → RSA      │
  │  GCD → ExtGCD → Mod Inverse → Division → Combinatorics        │
  │  Sieve → Factorisation → Multiplicative Functions → Counting  │
  │                                                                 │
  └─────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Cryptographic System | Number Theory Foundation | Security Basis |
|---------------------|-------------------------|----------------|
| RSA | Euler's theorem, ExtGCD, primes | Integer factorisation |
| Diffie-Hellman | Modular exponentiation | Discrete logarithm |
| Digital signatures | RSA + hashing | Factorisation |
| Polynomial hashing | Modular arithmetic, primes | Collision resistance |
| Secret sharing | CRT, Lagrange interpolation | Information-theoretic |

---

## Quick Revision Questions

1. **Trace** RSA key generation, encryption, and decryption for p=5, q=11, e=3, m=4.
2. **Why** does RSA decryption work (prove using Euler's theorem)?
3. **Explain** the Diffie-Hellman key exchange and what problem it relies on.
4. **What is** Baby-Step Giant-Step and what is its time complexity?
5. **How does** Pollard's rho find factors and why is it expected O(n^(1/4))?
6. **Connect** at least 5 topics from this course to show how they build upon each other.

---

[← Previous: Miller-Rabin Primality Test](05-miller-rabin-test.md) | [Back to README](../README.md)
