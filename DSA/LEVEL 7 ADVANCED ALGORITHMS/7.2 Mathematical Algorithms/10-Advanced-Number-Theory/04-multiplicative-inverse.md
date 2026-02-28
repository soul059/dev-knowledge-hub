# Chapter 4: Multiplicative Inverse

[← Previous: Euler's Totient Function](03-eulers-totient-function.md) | [Back to README](../README.md) | [Next: Miller-Rabin Primality Test →](05-miller-rabin-test.md)

---

## Overview

The **modular multiplicative inverse** of $a$ modulo $m$ is the integer $x$ such that $a \cdot x \equiv 1 \pmod{m}$. This chapter consolidates all methods for computing inverses and their applications in competitive programming.

---

## 4.1 Definition & Existence

```
  ┌──────────────────────────────────────────────────────────┐
  │  a^(-1) mod m  is the integer x such that:              │
  │                                                          │
  │        a × x ≡ 1 (mod m)                                │
  │                                                          │
  │  EXISTS if and only if  gcd(a, m) = 1                   │
  │                                                          │
  │  When it exists, the inverse is UNIQUE modulo m          │
  └──────────────────────────────────────────────────────────┘
  
  Example: 3^(-1) mod 7 = 5, because 3×5 = 15 ≡ 1 (mod 7)
  
  No inverse: 4^(-1) mod 6 does not exist (gcd(4,6) = 2 ≠ 1)
```

---

## 4.2 Method 1: Extended Euclidean Algorithm

```
  The most general method — works for any m.
  
  Solve: a × x + m × y = gcd(a, m) = 1
  The coefficient x is the inverse.
  
  FUNCTION modInverse_extGCD(a, m):
      (g, x, _) = extGCD(a, m)
      IF g ≠ 1:
          RETURN -1           // no inverse
      RETURN (x % m + m) % m  // ensure positive
  
  FUNCTION extGCD(a, b):
      IF a == 0:
          RETURN (b, 0, 1)
      (g, x1, y1) = extGCD(b % a, a)
      RETURN (g, y1 - (b/a) × x1, x1)
```

### Trace: 3^(-1) mod 11

```
  extGCD(3, 11):
    extGCD(3, 11) → extGCD(2, 3) → extGCD(1, 2) → extGCD(0, 1)
    
    extGCD(0, 1) → (1, 0, 1)
    extGCD(1, 2) → (1, 1 - 2×0, 0) = (1, 1, 0)
    extGCD(2, 3) → (1, 0 - 1×1, 1) = (1, -1, 1)
    extGCD(3, 11)→ (1, 1 - 3×(-1), -1) = (1, 4, -1)
  
  x = 4, so 3^(-1) ≡ 4 (mod 11)
  Verify: 3 × 4 = 12 ≡ 1 (mod 11) ✓
```

**Time:** $O(\log m)$ | **Space:** $O(\log m)$ recursive / $O(1)$ iterative

---

## 4.3 Method 2: Fermat's Little Theorem (Prime Modulus)

```
  When m = p (prime):
  
    a^(p-1) ≡ 1 (mod p)
    a^(-1)  ≡ a^(p-2) (mod p)
  
  FUNCTION modInverse_fermat(a, p):
      RETURN fastPow(a, p - 2, p)
```

### Trace: 3^(-1) mod 11

```
  3^(11-2) = 3^9 mod 11
  
  3^1 = 3
  3^2 = 9
  3^4 = 81 mod 11 = 4
  3^8 = 16 mod 11 = 5
  3^9 = 3^8 × 3^1 = 5 × 3 = 15 mod 11 = 4
  
  3^(-1) ≡ 4 (mod 11) ✓
```

**Time:** $O(\log p)$ | **Space:** $O(1)$

---

## 4.4 Method 3: Euler's Theorem (Any Modulus)

```
  When gcd(a, m) = 1:
  
    a^φ(m) ≡ 1 (mod m)
    a^(-1) ≡ a^(φ(m)-1) (mod m)
  
  Requires computing φ(m) first → O(√m) + O(log m)
  
  Less common in CP than extGCD, but conceptually important.
```

---

## 4.5 Method 4: Precompute All Inverses (1 to n) mod p

```
  ┌──────────────────────────────────────────────────┐
  │  Useful when you need inverses of 1, 2, ..., n  │
  │  all at once (common for nCr precomputation).   │
  │                                                  │
  │  Recurrence (p must be prime):                   │
  │                                                  │
  │  inv[1] = 1                                      │
  │  inv[i] = -(p / i) × inv[p % i] mod p          │
  │                                                  │
  └──────────────────────────────────────────────────┘
  
  Derivation:
    p = (p/i)×i + (p%i)    (division algorithm)
    0 ≡ (p/i)×i + (p%i)    (mod p)
    i^(-1) ≡ -(p/i) × (p%i)^(-1)  (mod p)
```

### Pseudocode

```
FUNCTION precomputeInverses(n, p):
    inv = array[n+1]
    inv[1] = 1
    FOR i = 2 TO n:
        inv[i] = (p - (p / i) × inv[p % i] % p) % p
    RETURN inv
```

### Trace: Inverses mod 7

```
  p = 7
  inv[1] = 1
  inv[2] = (7 - (7/2) × inv[7%2] % 7) % 7 = (7 - 3×1) % 7 = 4
  inv[3] = (7 - (7/3) × inv[7%3] % 7) % 7 = (7 - 2×inv[1]) % 7 = 5
  inv[4] = (7 - (7/4) × inv[7%4] % 7) % 7 = (7 - 1×inv[3]) % 7 = 2
  inv[5] = (7 - (7/5) × inv[7%5] % 7) % 7 = (7 - 1×inv[2]) % 7 = 3
  inv[6] = (7 - (7/6) × inv[7%6] % 7) % 7 = (7 - 1×inv[1]) % 7 = 6
  
  Verify all:
  1×1=1  2×4=8≡1  3×5=15≡1  4×2=8≡1  5×3=15≡1  6×6=36≡1  ✓
```

**Time:** $O(n)$ | **Space:** $O(n)$

---

## 4.6 Method 5: Factorial Inverses (for nCr)

```
  Precompute:
    fact[i]    = i! mod p
    invFact[i] = (i!)^(-1) mod p
  
  FUNCTION precompute(n, p):
      fact[0] = 1
      FOR i = 1 TO n:
          fact[i] = fact[i-1] × i % p
      
      invFact[n] = fastPow(fact[n], p-2, p)    // one modular inverse
      FOR i = n-1 DOWNTO 0:
          invFact[i] = invFact[i+1] × (i+1) % p  // backward pass
  
  FUNCTION nCr(n, r, p):
      IF r < 0 OR r > n: RETURN 0
      RETURN fact[n] × invFact[r] % p × invFact[n-r] % p
  
  ★ Only ONE expensive modInverse call (for invFact[n])!
    Rest computed in O(1) each via backward multiplication.
```

**Time:** $O(n)$ precompute, $O(1)$ per query

---

## 4.7 Modular Division

```
  a / b mod m = a × b^(-1) mod m
  
  ┌──────────────────────────────────────────────────┐
  │  CAUTION:                                        │
  │  (a/b) mod m  ≠  (a mod m) / (b mod m)         │
  │                                                  │
  │  Correct way:                                    │
  │  (a mod m) × modInverse(b, m) % m               │
  │                                                  │
  │  This only works when gcd(b, m) = 1             │
  └──────────────────────────────────────────────────┘
  
  Example: 15 / 3 mod 7
    = 15 × 3^(-1) mod 7
    = 15 × 5 mod 7     (since 3^(-1) ≡ 5 mod 7)
    = 75 mod 7
    = 5
  
  Check: 15/3 = 5, and 5 mod 7 = 5 ✓
```

---

## 4.8 Comparison of Methods

```
  ┌──────────────────────────────────────────────────────────┐
  │  Method             │ Modulus     │ Time     │ Use Case  │
  ├─────────────────────┼────────────┼──────────┼───────────┤
  │  ExtGCD             │ Any m      │ O(log m) │ General   │
  │  Fermat (a^(p-2))   │ Prime p    │ O(log p) │ Simple    │
  │  Euler (a^(φ(m)-1)) │ Any m      │ O(√m)   │ Rare      │
  │  Linear precompute  │ Prime p    │ O(n)     │ Batch 1-n │
  │  Factorial inverse  │ Prime p    │ O(n)     │ nCr       │
  └─────────────────────┴────────────┴──────────┴───────────┘
  
  ★ In competitive programming with p = 10^9+7:
    Use Fermat for single inverses, factorial method for nCr.
```

---

## Summary Table

| Method | Works for | Time | Best when |
|--------|-----------|------|-----------|
| Extended GCD | Any coprime pair | O(log m) | Non-prime modulus |
| Fermat's theorem | Prime modulus | O(log p) | Single inverse, prime mod |
| Euler's theorem | Any coprime pair | O(√m + log m) | Educational |
| Linear sieve | 1..n mod prime | O(n) total | Batch inverses |
| Factorial inverse | nCr queries | O(n) + O(1)/query | Combinatorics |

---

## Quick Revision Questions

1. **When does** a modular inverse exist?
2. **Compute** 4^(-1) mod 13 using both Fermat and ExtGCD.
3. **Derive** the recurrence inv[i] = -(p/i) × inv[p%i] mod p.
4. **Why** do we compute invFact backwards instead of calling fastPow n times?
5. **Can** 6^(-1) mod 9 be computed? Why or why not?
6. **Write** code to compute nCr(20, 5) mod 10^9+7 efficiently.

---

[← Previous: Euler's Totient Function](03-eulers-totient-function.md) | [Back to README](../README.md) | [Next: Miller-Rabin Primality Test →](05-miller-rabin-test.md)
