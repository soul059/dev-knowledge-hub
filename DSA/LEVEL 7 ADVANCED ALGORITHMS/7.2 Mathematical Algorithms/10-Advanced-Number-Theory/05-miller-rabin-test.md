# Chapter 5: Miller-Rabin Primality Test

[← Previous: Multiplicative Inverse](04-multiplicative-inverse.md) | [Back to README](../README.md) | [Next: Applications in Cryptography →](06-applications-in-cryptography.md)

---

## Overview

The **Miller-Rabin test** is a probabilistic primality test that is stronger than Fermat's test — it correctly identifies Carmichael numbers as composite. With deterministic witness sets, it gives exact results for numbers up to specific bounds. It is the standard primality test in competitive programming.

---

## 5.1 Mathematical Foundation

```
  Start from Fermat: a^(n-1) ≡ 1 (mod n) for prime n.
  
  Write n-1 = 2^s × d   where d is odd.
  
  Then a^(n-1) = (a^d)^(2^s)
  
  ┌──────────────────────────────────────────────────────────┐
  │  For prime n, squaring a^d repeatedly must "land" at 1  │
  │  via the sequence:                                       │
  │                                                          │
  │  a^d,  a^(2d),  a^(4d), ..., a^(2^s × d) = a^(n-1)    │
  │                                                          │
  │  Since x² ≡ 1 (mod p) only if x ≡ ±1 (mod p):         │
  │                                                          │
  │  The sequence must be one of:                            │
  │    1, 1, 1, ..., 1                                       │
  │    ?, ?, ..., -1, 1, 1, ..., 1                          │
  │                                                          │
  │  If neither pattern → n is COMPOSITE                    │
  └──────────────────────────────────────────────────────────┘
```

---

## 5.2 The Test

```
  To test if n is prime using witness a:
  
  1. Write n-1 = 2^s × d  (d odd)
  2. Compute x = a^d mod n
  3. If x == 1 or x == n-1: PASS (probably prime)
  4. Repeat s-1 times:
       x = x² mod n
       If x == n-1: PASS (probably prime)
       If x == 1:   FAIL (definitely composite: found non-trivial √1)
  5. FAIL (definitely composite)
```

---

## 5.3 Pseudocode

```
FUNCTION millerRabinTest(n, a):
    IF n < 2: RETURN false
    IF n == 2 OR n == 3: RETURN true
    IF n % 2 == 0: RETURN false
    
    // Write n-1 = 2^s × d
    d = n - 1
    s = 0
    WHILE d % 2 == 0:
        d = d / 2
        s = s + 1
    
    // Compute a^d mod n
    x = fastPow(a, d, n)
    
    IF x == 1 OR x == n - 1:
        RETURN true    // probably prime
    
    FOR r = 1 TO s - 1:
        x = (x × x) % n
        IF x == n - 1:
            RETURN true    // probably prime
        IF x == 1:
            RETURN false   // non-trivial square root of 1
    
    RETURN false   // composite

FUNCTION isPrime(n):
    FOR each witness a IN {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37}:
        IF n == a: RETURN true
        IF NOT millerRabinTest(n, a):
            RETURN false
    RETURN true
```

---

## 5.4 Step-by-Step Trace

### Test: n = 561 (Carmichael number), a = 2

```
  561 - 1 = 560 = 2^4 × 35    (s=4, d=35)
  
  x = 2^35 mod 561
  
  2^35 mod 561:
    2^1  = 2
    2^2  = 4
    2^4  = 16
    2^8  = 256
    2^16 = 256² = 65536 mod 561 = 460
    2^32 = 460² = 211600 mod 561 = 67
    2^35 = 2^32 × 2^2 × 2^1 = 67 × 4 × 2 = 536 mod 561 = 536
  
  x = 536
  536 ≠ 1 and 536 ≠ 560 → continue
  
  Round 1: x = 536² mod 561 = 287296 mod 561 = 1
           x == 1 → COMPOSITE! (found non-trivial √1 = 536)
  
  ★ Fermat would say "probably prime" (2^560 ≡ 1 mod 561)
    but Miller-Rabin catches it!
```

### Test: n = 13 (prime), a = 2

```
  13 - 1 = 12 = 2^2 × 3    (s=2, d=3)
  
  x = 2^3 mod 13 = 8
  8 ≠ 1 and 8 ≠ 12 → continue
  
  Round 1: x = 8² mod 13 = 64 mod 13 = 12
           x == 12 → PROBABLY PRIME  ✓
```

---

## 5.5 Deterministic Witness Sets

```
  ┌──────────────────────────────────────────────────────────┐
  │  Range of n              │ Sufficient witnesses         │
  ├──────────────────────────┼──────────────────────────────┤
  │  n < 2,047               │ {2}                          │
  │  n < 1,373,653           │ {2, 3}                       │
  │  n < 3,215,031,751       │ {2, 3, 5, 7}                │
  │  n < 3,317,044,064,679,887,385,961,981                 │
  │                          │ {2,3,5,7,11,13,17,19,23,29, │
  │                          │  31,37}                      │
  │  n < 3.3 × 10^24        │ First 12 primes             │
  │  n < 2^64               │ {2,3,5,7,11,13,17,19,23,29, │
  │                          │  31,37} (proven sufficient)  │
  └──────────────────────────┴──────────────────────────────┘
  
  ★ With {2,3,5,7,11,13,17,19,23,29,31,37}:
    DETERMINISTIC for all n < 3.3 × 10^24 (covers 64-bit integers!)
```

---

## 5.6 Implementation Notes for CP

```
  ┌──────────────────────────────────────────────────────────┐
  │  Key implementation details:                             │
  │                                                          │
  │  1. Use __int128 or modular multiplication to avoid     │
  │     overflow when n > 2^31 (since x*x can overflow)     │
  │                                                          │
  │  2. Handle a % n == 0 case (skip this witness)          │
  │                                                          │
  │  3. Use the 12-prime witness set for 64-bit numbers     │
  │                                                          │
  │  4. For n ≤ 10^9, witnesses {2, 3, 5, 7} suffice       │
  │                                                          │
  │  5. Combine with trial division for small factors       │
  │     (check primes < 1000 first for speed)               │
  └──────────────────────────────────────────────────────────┘
```

### Safe Modular Multiplication (for large n)

```
FUNCTION mulmod(a, b, m):
    // For languages without 128-bit integers:
    result = 0
    a = a % m
    WHILE b > 0:
        IF b is odd:
            result = (result + a) % m
        a = (a + a) % m         // double a
        b = b >> 1
    RETURN result
    
    // Time: O(log b) — slower but safe from overflow
    // In C++: use __int128 instead for O(1)
```

---

## 5.7 Comparison with Other Tests

```
  ┌──────────────────────────────────────────────────────────┐
  │  Test          │ Type          │ Handles       │ Speed   │
  │                │               │ Carmichael?   │         │
  ├────────────────┼───────────────┼───────────────┼─────────┤
  │  Trial div     │ Deterministic │ Yes           │ O(√n)   │
  │  Fermat        │ Probabilistic │ NO            │ O(k·log)│
  │  Miller-Rabin  │ Probabilistic*│ Yes           │ O(k·log)│
  │  AKS           │ Deterministic │ Yes           │ O(log⁶) │
  │  Baillie-PSW   │ Probabilistic │ Yes           │ O(log²) │
  └────────────────┴───────────────┴───────────────┴─────────┘
  
  * Deterministic with fixed witness sets for bounded n
  
  ★ Miller-Rabin is the gold standard for competitive programming.
```

---

## 5.8 Applications

```
  1. Primality testing in competitive programming
  2. Generating large random primes
  3. RSA key generation (find large primes p, q)
  4. Pollard's rho factorisation (uses M-R as subroutine)
  5. Verifying prime-related properties
  6. Number theory problem constraints (n up to 10^18)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Base idea | Strengthened Fermat test via square root check |
| Write | n-1 = 2^s × d (d odd) |
| Check | a^d, a^(2d), ..., a^(2^s·d) sequence |
| Composite if | Sequence reaches 1 without passing through -1 |
| Error probability | ≤ 1/4 per random witness |
| Deterministic | 12 witnesses for all 64-bit numbers |
| Time per witness | O(log² n) with safe multiplication |

---

## Quick Revision Questions

1. **Why** is Miller-Rabin stronger than Fermat's test?
2. **Decompose** 100 as 2^s × d and trace Miller-Rabin with a=2 on n=101.
3. **What witnesses** suffice for deterministic testing up to 10^9?
4. **Explain** why x² ≡ 1 (mod p) implies x ≡ ±1 when p is prime.
5. **How** does Miller-Rabin detect 561 as composite?
6. **What is** the per-witness error probability and how do multiple rounds help?

---

[← Previous: Multiplicative Inverse](04-multiplicative-inverse.md) | [Back to README](../README.md) | [Next: Applications in Cryptography →](06-applications-in-cryptography.md)
