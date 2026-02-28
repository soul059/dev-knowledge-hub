# Chapter 1: Fermat's Little Theorem

[← Previous Unit: Rectangle Overlap](../09-Computational-Geometry/06-rectangle-overlap.md) | [Back to README](../README.md) | [Next: Chinese Remainder Theorem →](02-chinese-remainder-theorem.md)

---

## Overview

**Fermat's Little Theorem** is one of the most important results in number theory. It provides a shortcut for modular exponentiation and forms the basis for primality testing, modular inverses, and cryptographic protocols.

---

## 1.1 Statement

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  If p is PRIME and gcd(a, p) = 1, then:                │
  │                                                          │
  │              a^(p-1) ≡ 1  (mod p)                       │
  │                                                          │
  │  Equivalently, for any integer a:                       │
  │                                                          │
  │              a^p ≡ a  (mod p)                            │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.2 Intuitive Understanding

```
  Consider p = 7, a = 3:
  
  Powers of 3 mod 7:
    3^1 ≡ 3
    3^2 ≡ 2
    3^3 ≡ 6
    3^4 ≡ 4
    3^5 ≡ 5
    3^6 ≡ 1  ← a^(p-1) ≡ 1 ✓
    3^7 ≡ 3  ← a^p ≡ a ✓  (cycle restarts)
  
  The residues {3,2,6,4,5,1} are a permutation of {1,2,3,4,5,6}.
  
  ┌──────────────────────────────────────────────┐
  │  Why does this work?                         │
  │                                              │
  │  {1·a, 2·a, 3·a, ..., (p-1)·a} mod p       │
  │  is a permutation of {1, 2, 3, ..., p-1}    │
  │                                              │
  │  So their products are equal:                │
  │  a^(p-1) · (p-1)! ≡ (p-1)!  (mod p)        │
  │                                              │
  │  Cancel (p-1)!  (coprime to p):              │
  │        a^(p-1) ≡ 1  (mod p)   □             │
  └──────────────────────────────────────────────┘
```

---

## 1.3 Proof (Detailed)

```
  Given: p is prime, gcd(a, p) = 1
  
  Step 1: Consider the set S = {1·a, 2·a, ..., (p-1)·a}
  
  Step 2: Claim — all elements of S are distinct mod p.
    Proof by contradiction:
    Suppose i·a ≡ j·a (mod p) for 1 ≤ i < j ≤ p-1
    Then (j-i)·a ≡ 0 (mod p)
    Since gcd(a,p)=1 and p is prime, p | (j-i)
    But 0 < j-i < p, contradiction! □
  
  Step 3: Since S has p-1 elements, all distinct, and none ≡ 0 (mod p),
    S mod p is a permutation of {1, 2, ..., p-1}
  
  Step 4: Multiply all elements:
    (1·a)(2·a)...(p-1)·a ≡ 1·2·...·(p-1) (mod p)
    a^(p-1) · (p-1)! ≡ (p-1)! (mod p)
  
  Step 5: Since gcd((p-1)!, p) = 1 (p is prime, so no factor of p in (p-1)!):
    a^(p-1) ≡ 1 (mod p)  □
```

---

## 1.4 Application 1: Modular Inverse

```
  From a^(p-1) ≡ 1 (mod p):
  
    a · a^(p-2) ≡ 1 (mod p)
    
    Therefore: a^(-1) ≡ a^(p-2) (mod p)
  
  ┌──────────────────────────────────────────────────┐
  │  To find modular inverse of a mod prime p:       │
  │                                                  │
  │       inverse(a) = power(a, p-2, p)             │
  │                                                  │
  │  Uses fast exponentiation → O(log p)             │
  └──────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION modInverse(a, p):
    // p must be prime and gcd(a, p) = 1
    RETURN fastPow(a, p - 2, p)

FUNCTION fastPow(base, exp, mod):
    result = 1
    base = base % mod
    WHILE exp > 0:
        IF exp is odd:
            result = (result × base) % mod
        exp = exp >> 1
        base = (base × base) % mod
    RETURN result
```

### Trace

```
  Find inverse of 3 mod 7:
  
  3^(7-2) = 3^5 mod 7
  
  3^1 = 3
  3^2 = 9 mod 7 = 2
  3^4 = 4 mod 7 = 4
  3^5 = 3^4 × 3^1 = 4 × 3 = 12 mod 7 = 5
  
  Verify: 3 × 5 = 15 mod 7 = 1  ✓
  
  So 3^(-1) ≡ 5 (mod 7)
```

---

## 1.5 Application 2: Simplifying Large Exponents

```
  Compute a^n mod p (p prime, gcd(a,p)=1):
  
  Since a^(p-1) ≡ 1 (mod p):
    a^n ≡ a^(n mod (p-1)) (mod p)
  
  Example: 2^100 mod 13
    p-1 = 12
    100 mod 12 = 4
    2^100 ≡ 2^4 = 16 mod 13 = 3
  
  ★ Reduces exponent from n to n mod (p-1) before fast exponentiation!
```

---

## 1.6 Application 3: Fermat Primality Test

```
  If p is prime, then a^(p-1) ≡ 1 (mod p) for all 1 ≤ a < p.
  
  CONTRAPOSITIVE: If a^(p-1) ≢ 1 (mod p) for some a, then p is NOT prime.
  
  ┌──────────────────────────────────────────────────┐
  │  Fermat Primality Test:                          │
  │                                                  │
  │  1. Pick random a ∈ [2, n-2]                    │
  │  2. Compute a^(n-1) mod n using fast power      │
  │  3. If result ≠ 1 → n is DEFINITELY composite   │
  │  4. If result = 1 → n is PROBABLY prime          │
  │  5. Repeat k times for confidence                │
  │                                                  │
  │  False positive probability: ≈ (1/2)^k          │
  └──────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION isProbablyPrime(n, k):
    IF n < 2: RETURN false
    IF n <= 3: RETURN true
    IF n is even: RETURN false
    
    FOR i = 1 TO k:
        a = random(2, n-2)
        IF fastPow(a, n-1, n) ≠ 1:
            RETURN false   // definitely composite
    
    RETURN true  // probably prime
```

---

## 1.7 Carmichael Numbers (Fermat Liars)

```
  ┌──────────────────────────────────────────────────┐
  │  WARNING: Some composites pass Fermat's test     │
  │  for ALL bases coprime to n!                     │
  │                                                  │
  │  These are CARMICHAEL NUMBERS:                   │
  │  561 = 3 × 11 × 17                              │
  │  1105 = 5 × 13 × 17                             │
  │  1729 = 7 × 13 × 19                             │
  │                                                  │
  │  For these, a^(n-1) ≡ 1 (mod n) for all         │
  │  gcd(a,n) = 1, even though n is composite!      │
  │                                                  │
  │  → This is why Miller-Rabin is preferred         │
  │    (it detects Carmichael numbers)               │
  └──────────────────────────────────────────────────┘
  
  Korselt's criterion: n is Carmichael iff
  1. n is square-free
  2. For every prime p | n: (p-1) | (n-1)
```

---

## 1.8 Relationship to Euler's Theorem

```
  Fermat's Little Theorem is a special case of Euler's Theorem:
  
  ┌──────────────────────────────────────────────────┐
  │  Euler's Theorem:                                │
  │  If gcd(a, n) = 1, then a^φ(n) ≡ 1 (mod n)    │
  │                                                  │
  │  When n = p (prime):                             │
  │  φ(p) = p - 1                                    │
  │  So a^(p-1) ≡ 1 (mod p)   ← Fermat's!          │
  └──────────────────────────────────────────────────┘
  
  Euler's version works for any n, not just primes.
  (Covered in Chapter 3: Euler's Totient Function)
```

---

## Summary Table

| Application | Formula | Requirement |
|-------------|---------|-------------|
| Modular inverse | a⁻¹ ≡ a^(p-2) mod p | p prime |
| Exponent reduction | a^n ≡ a^(n mod (p-1)) mod p | p prime, gcd(a,p)=1 |
| Primality test | a^(n-1) ≡ 1 mod n? | Probabilistic |
| Discrete log link | ord(a) divides p-1 | p prime |

---

## Quick Revision Questions

1. **State** Fermat's Little Theorem in both forms.
2. **Compute** 5^(-1) mod 13 using Fermat's theorem.
3. **What is** a Carmichael number and why is it problematic?
4. **Why** can we reduce a^n mod p to a^(n mod (p-1)) mod p?
5. **Compute** 2^1000 mod 17 using Fermat's theorem.
6. **How** is Fermat's theorem a special case of Euler's theorem?

---

[← Previous Unit: Rectangle Overlap](../09-Computational-Geometry/06-rectangle-overlap.md) | [Back to README](../README.md) | [Next: Chinese Remainder Theorem →](02-chinese-remainder-theorem.md)
