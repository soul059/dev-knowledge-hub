# Chapter 3: Modular Division and Modular Inverse

[← Previous: Modular Multiplication](02-modular-multiplication.md) | [Back to README](../README.md) | [Next: Modular Exponentiation →](04-modular-exponentiation.md)

---

## Overview

Unlike addition and multiplication, division does NOT directly work under modulo. We need the concept of a **modular multiplicative inverse**. This chapter covers why, how to find it, and when it exists.

---

## 3.1 The Division Problem

```
  ┌──────────────────────────────────────────────────────────┐
  │  WRONG:                                                  │
  │  (a / b) mod M ≠ (a mod M) / (b mod M)                 │
  │                                                          │
  │  Example: (14 / 7) mod 5                                │
  │  Correct: 14/7 = 2, 2 mod 5 = 2                        │
  │  Wrong way: (14%5) / (7%5) = 4/2 = 2 (lucky accident!) │
  │                                                          │
  │  Example: (10 / 5) mod 3                                │
  │  Correct: 10/5 = 2, 2 mod 3 = 2                        │
  │  Wrong way: (10%3) / (5%3) = 1/2 = 0 (integer div) ✗   │
  └──────────────────────────────────────────────────────────┘

  Instead: (a / b) mod M = (a × b⁻¹) mod M
  where b⁻¹ is the MODULAR INVERSE of b.
```

---

## 3.2 What Is a Modular Inverse?

```
  b⁻¹ mod M is the number x such that:
  
      b × x ≡ 1 (mod M)

  Example: inverse of 3 mod 7
  3 × 1 = 3 mod 7 ≠ 1
  3 × 2 = 6 mod 7 ≠ 1
  3 × 3 = 9 mod 7 = 2 ≠ 1
  3 × 4 = 12 mod 7 = 5 ≠ 1
  3 × 5 = 15 mod 7 = 1 ✓

  So 3⁻¹ ≡ 5 (mod 7)
  Verify: (a / 3) mod 7 = (a × 5) mod 7

  ┌──────────────────────────────────────────────────────────┐
  │  EXISTENCE: b⁻¹ mod M exists  ⟺  gcd(b, M) = 1       │
  │                                                          │
  │  If M is prime, EVERY b ∈ {1, ..., M-1} has an inverse │
  │  This is why M = 10⁹+7 (prime) is used everywhere!     │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.3 Method 1: Extended Euclidean Algorithm

```
  From Chapter 3.2: Extended GCD gives us x, y such that:
  b × x + M × y = gcd(b, M)
  
  If gcd(b, M) = 1:
  b × x + M × y = 1
  b × x ≡ 1 (mod M)     (since M × y ≡ 0 mod M)
  So x = b⁻¹ mod M

FUNCTION modInverse_extGCD(b, M):
    (g, x, _) = extendedGCD(b, M)
    IF g ≠ 1:
        RETURN -1     // inverse doesn't exist
    RETURN (x % M + M) % M   // ensure positive

Time: O(log M)
```

### Trace: Inverse of 3 mod 7

```
  extGCD(3, 7):
  7 = 3×2 + 1
  3 = 1×3 + 0
  
  Back-substitute:
  1 = 7 - 3×2
  1 = 7×1 + 3×(-2)
  
  x = -2
  x mod 7 = (-2 + 7) % 7 = 5
  
  3⁻¹ mod 7 = 5
  Verify: 3 × 5 = 15, 15 mod 7 = 1 ✓
```

---

## 3.4 Method 2: Fermat's Little Theorem

```
  ┌──────────────────────────────────────────────────────────┐
  │  FERMAT'S LITTLE THEOREM:                                │
  │  If p is PRIME and gcd(a, p) = 1:                       │
  │                                                          │
  │      a^(p-1) ≡ 1 (mod p)                                │
  │                                                          │
  │  Therefore:                                              │
  │      a × a^(p-2) ≡ 1 (mod p)                            │
  │      a⁻¹ ≡ a^(p-2) (mod p)                              │
  │                                                          │
  │  So: modular inverse = modular exponentiation!           │
  │      b⁻¹ mod p = power(b, p-2, p)                       │
  └──────────────────────────────────────────────────────────┘

FUNCTION modInverse_fermat(b, p):  // p must be prime!
    RETURN modPow(b, p - 2, p)

Time: O(log p)   (using fast exponentiation)
```

### Trace: Inverse of 3 mod 7

```
  3⁻¹ mod 7 = 3^(7-2) mod 7 = 3⁵ mod 7

  3¹ = 3
  3² = 9 mod 7 = 2
  3⁴ = 2² = 4
  3⁵ = 3⁴ × 3¹ = 4 × 3 = 12 mod 7 = 5

  3⁻¹ mod 7 = 5 ✓ (matches!)
```

---

## 3.5 Method 3: Iterative Formula (For All Inverses 1..n)

```
  Compute inverse of ALL numbers from 1 to n in O(n):

  inv[1] = 1
  FOR i = 2 TO n:
      inv[i] = -(M / i) × inv[M % i]  mod M

  Derivation:
  M = (M/i)×i + (M%i)
  0 ≡ (M/i)×i + (M%i)  (mod M)
  i × (M/i) ≡ -(M%i)   (mod M)
  i⁻¹ ≡ -(M/i) × (M%i)⁻¹ (mod M)
  i⁻¹ ≡ -(M/i) × inv[M%i] (mod M)
```

### Trace: All inverses mod 7

```
  M = 7
  inv[1] = 1
  
  inv[2] = -(7/2) × inv[7%2] mod 7
         = -3 × inv[1] mod 7
         = -3 mod 7 = 4
  Check: 2 × 4 = 8 mod 7 = 1 ✓

  inv[3] = -(7/3) × inv[7%3] mod 7
         = -2 × inv[1] mod 7
         = -2 mod 7 = 5
  Check: 3 × 5 = 15 mod 7 = 1 ✓

  inv[4] = -(7/4) × inv[7%4] mod 7
         = -1 × inv[3] mod 7
         = -5 mod 7 = 2
  Check: 4 × 2 = 8 mod 7 = 1 ✓

  inv[5] = -(7/5) × inv[7%5] mod 7
         = -1 × inv[2] mod 7
         = -4 mod 7 = 3
  Check: 5 × 3 = 15 mod 7 = 1 ✓

  inv[6] = -(7/6) × inv[7%6] mod 7
         = -1 × inv[1] mod 7
         = -1 mod 7 = 6
  Check: 6 × 6 = 36 mod 7 = 1 ✓

  ┌───┬──────┬────────────────┐
  │ i │ i⁻¹  │ i × i⁻¹ mod 7 │
  ├───┼──────┼────────────────┤
  │ 1 │  1   │      1         │
  │ 2 │  4   │      1         │
  │ 3 │  5   │      1         │
  │ 4 │  2   │      1         │
  │ 5 │  3   │      1         │
  │ 6 │  6   │      1         │
  └───┴──────┴────────────────┘
```

---

## 3.6 Comparison of Methods

```
  ┌──────────────────┬──────────────┬──────────────┬──────────────┐
  │ Method           │ Time         │ Works When   │ Best For     │
  ├──────────────────┼──────────────┼──────────────┼──────────────┤
  │ Extended GCD     │ O(log M)     │ gcd(b,M)=1   │ Any M        │
  │ Fermat's         │ O(log M)     │ M is prime   │ M is prime   │
  │ Iterative        │ O(n) total   │ M is prime   │ All inv 1..n │
  └──────────────────┴──────────────┴──────────────┴──────────────┘

  Most common in CP: Fermat's method (simple, M is usually prime)
```

---

## 3.7 Modular Division

```
FUNCTION modDiv(a, b, M):
    b_inv = modInverse(b, M)     // using any method
    RETURN (a % M) * b_inv % M

  Example: (14 / 3) mod 7
  We don't mean integer division 14/3!
  We mean: find x such that 3x ≡ 14 (mod 7)

  3⁻¹ mod 7 = 5
  x = 14 × 5 mod 7 = 70 mod 7 = 0

  Verify: 3 × 0 = 0 ≡ 14 (mod 7)?
  14 mod 7 = 0. Yes! ✓
```

---

## 3.8 Factorial Inverse for nCr

```
  nCr mod p = n! / (r! × (n-r)!)  mod p
            = n! × (r!)⁻¹ × ((n-r)!)⁻¹  mod p

  Precompute:
  1. fact[i] = i! mod p         for i = 0..n
  2. inv_fact[n] = modPow(fact[n], p-2, p)
  3. inv_fact[i] = inv_fact[i+1] × (i+1)  for i = n-1..0

  Then: nCr = fact[n] × inv_fact[r] × inv_fact[n-r] mod p

  This is O(n) precomputation, O(1) per query!
```

---

## 3.9 When Inverse Doesn't Exist

```
  If gcd(b, M) ≠ 1, b⁻¹ mod M does NOT exist.

  Example: 4⁻¹ mod 6
  4 × 0 = 0 mod 6
  4 × 1 = 4 mod 6
  4 × 2 = 8 mod 6 = 2
  4 × 3 = 12 mod 6 = 0
  4 × 4 = 16 mod 6 = 4
  4 × 5 = 20 mod 6 = 2
  
  Products: {0, 4, 2, 0, 4, 2} → 1 never appears!
  gcd(4, 6) = 2 ≠ 1 → no inverse ✗

  If M is prime (like 10⁹+7), this NEVER happens
  (for b ≢ 0 mod M).
```

---

## Summary Table

| Concept | Formula | Key Condition |
|---------|---------|---------------|
| Modular inverse | b × b⁻¹ ≡ 1 (mod M) | gcd(b, M) = 1 |
| Via Extended GCD | extGCD gives x | Works for any M |
| Via Fermat | b⁻¹ = b^(p-2) mod p | M must be prime |
| Via Iterative | inv[i] = -(M/i)×inv[M%i] | O(n) for 1..n |
| Modular division | a/b = a × b⁻¹ mod M | b⁻¹ must exist |
| nCr mod p | fact[n] × inv_fact[r] × inv_fact[n-r] | p prime |

---

## Quick Revision Questions

1. **Why can't we** just do (a mod M) / (b mod M) for modular division?
2. **Find 4⁻¹ mod 13** using Fermat's Little Theorem.
3. **Find 5⁻¹ mod 11** using the Extended Euclidean Algorithm.
4. **When does** b⁻¹ mod M NOT exist? Give an example.
5. **Compute all inverses** mod 5 using the iterative method.
6. **How do you compute nCr mod p** efficiently? What's the precomputation cost?

---

[← Previous: Modular Multiplication](02-modular-multiplication.md) | [Back to README](../README.md) | [Next: Modular Exponentiation →](04-modular-exponentiation.md)
