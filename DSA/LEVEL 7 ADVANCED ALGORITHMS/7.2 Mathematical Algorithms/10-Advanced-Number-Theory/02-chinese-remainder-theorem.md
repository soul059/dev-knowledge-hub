# Chapter 2: Chinese Remainder Theorem

[← Previous: Fermat's Little Theorem](01-fermats-little-theorem.md) | [Back to README](../README.md) | [Next: Euler's Totient Function →](03-eulers-totient-function.md)

---

## Overview

The **Chinese Remainder Theorem (CRT)** provides a way to reconstruct an integer from its remainders modulo several pairwise coprime numbers. It is fundamental in competitive programming for combining modular constraints and in cryptography (RSA optimisation).

---

## 2.1 Statement

```
  ┌──────────────────────────────────────────────────────────┐
  │  Given pairwise coprime moduli m1, m2, ..., mk           │
  │  and residues r1, r2, ..., rk                            │
  │                                                          │
  │  The system:                                             │
  │     x ≡ r1 (mod m1)                                     │
  │     x ≡ r2 (mod m2)                                     │
  │     ...                                                  │
  │     x ≡ rk (mod mk)                                     │
  │                                                          │
  │  has a UNIQUE solution modulo M = m1·m2·...·mk           │
  │                                                          │
  │  i.e., exactly one x in [0, M)                           │
  └──────────────────────────────────────────────────────────┘
  
  "Pairwise coprime" means gcd(mi, mj) = 1 for all i ≠ j
```

---

## 2.2 Construction Formula

```
  Let M = m1 × m2 × ... × mk
  Let Mi = M / mi   (product of all moduli except mi)
  Let yi = Mi^(-1) mod mi   (modular inverse of Mi modulo mi)
  
  Then:
            x = (Σ ri × Mi × yi) mod M
  
  ┌──────────────────────────────────────────────────┐
  │  Step-by-step:                                   │
  │  1. Compute M = product of all moduli            │
  │  2. For each i:                                  │
  │     a. Mi = M / mi                               │
  │     b. yi = modInverse(Mi, mi)                   │
  │     c. term_i = ri × Mi × yi                     │
  │  3. x = sum of all terms, mod M                  │
  └──────────────────────────────────────────────────┘
```

---

## 2.3 Trace Example

```
  Solve:
    x ≡ 2 (mod 3)
    x ≡ 3 (mod 5)
    x ≡ 2 (mod 7)
  
  Step 1: M = 3 × 5 × 7 = 105
  
  Step 2:
    M1 = 105/3 = 35    y1 = 35^(-1) mod 3
                         35 mod 3 = 2
                         2^(-1) mod 3 = 2  (since 2×2=4≡1)
                         y1 = 2
    
    M2 = 105/5 = 21    y2 = 21^(-1) mod 5
                         21 mod 5 = 1
                         1^(-1) mod 5 = 1
                         y2 = 1
    
    M3 = 105/7 = 15    y3 = 15^(-1) mod 7
                         15 mod 7 = 1
                         1^(-1) mod 7 = 1
                         y3 = 1
  
  Step 3:
    x = (2×35×2 + 3×21×1 + 2×15×1) mod 105
    x = (140 + 63 + 30) mod 105
    x = 233 mod 105
    x = 23
  
  Verify:
    23 mod 3 = 2 ✓
    23 mod 5 = 3 ✓
    23 mod 7 = 2 ✓
```

---

## 2.4 Pseudocode

```
FUNCTION chineseRemainderTheorem(remainders[], moduli[], k):
    M = 1
    FOR i = 0 TO k-1:
        M *= moduli[i]
    
    x = 0
    FOR i = 0 TO k-1:
        Mi = M / moduli[i]
        yi = modInverse(Mi, moduli[i])     // extended Euclidean
        x = (x + remainders[i] × Mi × yi) % M
    
    RETURN x

FUNCTION modInverse(a, m):
    // Using extended Euclidean algorithm
    (g, x, _) = extGCD(a, m)
    IF g ≠ 1: ERROR "inverse doesn't exist"
    RETURN (x % m + m) % m

FUNCTION extGCD(a, b):
    IF a == 0: RETURN (b, 0, 1)
    (g, x1, y1) = extGCD(b % a, a)
    RETURN (g, y1 - (b/a)×x1, x1)
```

---

## 2.5 Two-Moduli Simplified Version

```
  For just two equations (very common in CP):
  
    x ≡ r1 (mod m1)
    x ≡ r2 (mod m2)
  
  x = r1 + m1 × t, where t satisfies:
    m1 × t ≡ (r2 - r1) (mod m2)
    t = (r2 - r1) × m1^(-1) mod m2
    
  x = r1 + m1 × t
  x ranges in [0, m1×m2)
  
  ★ Can be extended iteratively for multiple moduli!
```

### Iterative CRT

```
FUNCTION iterativeCRT(remainders[], moduli[], k):
    curR = remainders[0]
    curM = moduli[0]
    
    FOR i = 1 TO k-1:
        // Merge: x ≡ curR (mod curM) and x ≡ r[i] (mod m[i])
        diff = remainders[i] - curR
        inv = modInverse(curM, moduli[i])
        t = (diff × inv) % moduli[i]
        IF t < 0: t += moduli[i]
        
        curR = curR + curM × t
        curM = curM × moduli[i]
        curR = curR % curM
    
    RETURN curR
```

---

## 2.6 Extended CRT (Non-Coprime Moduli)

```
  When moduli are NOT pairwise coprime:
  
    x ≡ r1 (mod m1)
    x ≡ r2 (mod m2)
  
  Let g = gcd(m1, m2)
  
  ┌──────────────────────────────────────────────────┐
  │  Solution exists ⟺ (r2 - r1) mod g == 0       │
  │                                                  │
  │  If solvable:                                    │
  │    Merged: x ≡ r (mod lcm(m1, m2))              │
  │    where r is computed via extended GCD           │
  └──────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION extendedCRT(r1, m1, r2, m2):
    g = gcd(m1, m2)
    IF (r2 - r1) % g ≠ 0:
        RETURN NO_SOLUTION
    
    lcm_val = m1 / g × m2
    (_, p, _) = extGCD(m1/g, m2/g)
    diff = (r2 - r1) / g
    t = diff × p % (m2/g)
    r = r1 + m1 × t
    r = ((r % lcm_val) + lcm_val) % lcm_val
    
    RETURN (r, lcm_val)
```

---

## 2.7 Applications

```
  ┌──────────────────────────────────────────────────┐
  │  1. RSA decryption optimization (Garner's alg)   │
  │  2. Solving systems of modular equations in CP   │
  │  3. Reconstructing large numbers from residues   │
  │  4. Hash collision avoidance (multi-mod hashing) │
  │  5. Calendar / clock arithmetic                  │
  │  6. Secret sharing schemes                       │
  └──────────────────────────────────────────────────┘
  
  CP Pattern: Compute answer mod p1, mod p2, mod p3 separately
              (each fits in 64-bit), then use CRT to combine.
              
  Common mod set: {10^9+7, 10^9+9, 998244353}
```

---

## 2.8 Garner's Algorithm

```
  Efficient representation when M is huge:
  
  Instead of computing x = Σ ri × Mi × yi (may overflow),
  represent x in mixed-radix form:
  
    x = a1 + a2·m1 + a3·m1·m2 + a4·m1·m2·m3 + ...
  
  Coefficients a_i computed incrementally — avoids big numbers.
  
  Useful when combining answers mod several primes to get
  the true (huge) answer or its value mod yet another number.
```

---

## Summary Table

| Variant | Requirement | Time | Key Step |
|---------|-------------|------|----------|
| Standard CRT | Pairwise coprime | O(k log M) | Mod inverse via extGCD |
| Iterative CRT | Pairwise coprime | O(k log M) | Merge two at a time |
| Extended CRT | Any moduli | O(k log M) | Check divisibility by gcd |
| Garner's algorithm | Pairwise coprime | O(k²) | Mixed-radix conversion |

---

## Quick Revision Questions

1. **State** the Chinese Remainder Theorem and its uniqueness condition.
2. **Solve** x ≡ 1 (mod 3), x ≡ 4 (mod 5), x ≡ 6 (mod 7) using CRT.
3. **When does** the system fail to have a solution with non-coprime moduli?
4. **Why** is CRT useful for multi-mod hashing?
5. **Explain** the iterative merging approach for multiple congruences.
6. **What is** Garner's algorithm and when would you use it?

---

[← Previous: Fermat's Little Theorem](01-fermats-little-theorem.md) | [Back to README](../README.md) | [Next: Euler's Totient Function →](03-eulers-totient-function.md)
