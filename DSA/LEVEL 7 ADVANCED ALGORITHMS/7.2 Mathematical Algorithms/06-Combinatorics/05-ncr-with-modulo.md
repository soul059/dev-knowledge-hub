# Chapter 5: nCr with Modulo

[← Previous: nCr Computation](04-ncr-computation.md) | [Back to README](../README.md) | [Next: Catalan Numbers →](06-catalan-numbers.md)

---

## Overview

Since $\binom{n}{r}$ grows astronomically, competitive programming problems almost always ask for the answer **modulo a prime** $p$ (usually $10^9 + 7$). This chapter covers all core techniques: factorial precomputation, modular inverse, and **Lucas' theorem** for huge $n$.

---

## 5.1 Why Modulo?

```
  C(1000, 500) has 299 digits!
  C(10⁶, 5×10⁵) is astronomically large.
  
  We cannot store such values, so we compute:
  C(n, r) mod p    where p = 10⁹ + 7 (a prime)

  ⚠️ Modulo does NOT distribute over division:
     (a / b) mod p ≠ (a mod p) / (b mod p)
  
  Instead: (a / b) mod p = (a mod p) × (b⁻¹ mod p) mod p
  where b⁻¹ is the modular inverse of b.
```

---

## 5.2 Precomputing Factorials and Inverse Factorials

```
  MOD = 10⁹ + 7
  MAXN = 10⁶ + 5

  // Step 1: Precompute factorials
  fact[0] = 1
  FOR i = 1 TO MAXN:
      fact[i] = fact[i-1] × i % MOD

  // Step 2: Compute inverse of last factorial using Fermat's Little Theorem
  inv_fact[MAXN] = modPow(fact[MAXN], MOD - 2, MOD)

  // Step 3: Backfill inverse factorials
  FOR i = MAXN - 1 DOWNTO 0:
      inv_fact[i] = inv_fact[i+1] × (i + 1) % MOD

  ┌─────────────────────────────────────────────────────────┐
  │  Why does backfilling work?                             │
  │                                                         │
  │  inv_fact[i] = 1/i!                                     │
  │  inv_fact[i+1] = 1/(i+1)!                               │
  │  inv_fact[i] = inv_fact[i+1] × (i+1)                   │
  │             = (i+1) / (i+1)! = 1/i!  ✓                 │
  └─────────────────────────────────────────────────────────┘

  Precompute: O(n) time + O(log p) for one modPow
  Per query:  O(1)
```

---

## 5.3 Query: C(n, r) mod p in O(1)

```
FUNCTION nCr_mod(n, r):
    IF r < 0 OR r > n: RETURN 0
    RETURN fact[n] × inv_fact[r] % MOD × inv_fact[n - r] % MOD

  ┌────────────────────────────────────────────────────────┐
  │  C(n, r) = n! / (r! × (n-r)!)                        │
  │          = fact[n] × inv_fact[r] × inv_fact[n-r]      │
  │          (all mod p)                                   │
  └────────────────────────────────────────────────────────┘
```

### Trace: C(5, 2) mod 10⁹+7

```
  fact[5] = 120
  fact[2] = 2
  fact[3] = 6
  
  inv_fact[2] = modPow(2, 10⁹+5, 10⁹+7) = 500000004
  inv_fact[3] = modPow(6, 10⁹+5, 10⁹+7) = 166666668
  
  C(5,2) = 120 × 500000004 % MOD × 166666668 % MOD
         = ... = 10 ✓
```

---

## 5.4 Modular Inverse Methods

### Method 1: Fermat's Little Theorem (p is prime)

```
  a⁻¹ ≡ aᵖ⁻² (mod p)

FUNCTION modInverse(a, p):
    RETURN modPow(a, p - 2, p)

  Time: O(log p) per call
  
  ⚠️ Only works when p is prime AND gcd(a, p) = 1
```

### Method 2: Extended Euclidean Algorithm

```
FUNCTION modInverse(a, m):
    (g, x, y) = extGCD(a, m)
    IF g ≠ 1: RETURN -1    // no inverse exists
    RETURN (x % m + m) % m

  Time: O(log(min(a, m)))
  Works for ANY modulus m (not just primes), as long as gcd(a, m) = 1
```

### Method 3: Iterative Formula (1 to n in O(n))

```
  inv[1] = 1
  FOR i = 2 TO n:
      inv[i] = -(MOD / i) × inv[MOD % i] % MOD
      inv[i] = (inv[i] % MOD + MOD) % MOD

  Why this works:
  Let MOD = q × i + r  where q = ⌊MOD/i⌋, r = MOD % i
  Then: q × i + r ≡ 0 (mod MOD)
        i⁻¹ ≡ -q × r⁻¹ ≡ -(MOD/i) × inv[MOD%i] (mod MOD)
  
  Time: O(n) for all inverses from 1 to n
```

---

## 5.5 Lucas' Theorem (Huge n, Small Prime p)

```
  When n, r can be up to 10¹⁸ but p is SMALL (e.g., p ≤ 10⁵):

  Lucas' Theorem:
  C(n, r) mod p = Π C(nᵢ, rᵢ) mod p
  
  where n = nₖnₖ₋₁...n₁n₀ in base p
        r = rₖrₖ₋₁...r₁r₀ in base p

  If any rᵢ > nᵢ, the result is 0.

FUNCTION lucas(n, r, p):
    result = 1
    WHILE n > 0 OR r > 0:
        ni = n % p
        ri = r % p
        IF ri > ni: RETURN 0
        result = result × nCr_small(ni, ri, p) % p
        n = n / p
        r = r / p
    RETURN result

FUNCTION nCr_small(n, r, p):
    // n, r < p, so precompute factorials up to p-1
    IF r > n: RETURN 0
    RETURN fact[n] × inv_fact[r] % p × inv_fact[n-r] % p

  Time: O(p + log_p(n))
  Space: O(p)
```

### Trace: C(10, 3) mod 3

```
  Convert to base 3:
  10 = (1·9 + 0·3 + 1) → digits [1, 0, 1]
   3 = (0·9 + 1·3 + 0) → digits [0, 1, 0]
  
  Step 1: C(1, 0) mod 3 = 1
  Step 2: C(0, 1) mod 3 = 0  ← r_i > n_i → entire result = 0
  
  C(10, 3) mod 3 = 0
  
  Check: C(10,3) = 120, 120 mod 3 = 0 ✓
```

---

## 5.6 Complete Template Code

```
// Pseudocode template for competitive programming

CONST MOD = 1000000007
CONST MAXN = 1000005

fact[MAXN], inv_fact[MAXN]

FUNCTION modPow(base, exp, mod):
    result = 1
    base = base % mod
    WHILE exp > 0:
        IF exp is odd: result = result × base % mod
        exp = exp >> 1
        base = base × base % mod
    RETURN result

FUNCTION precompute():
    fact[0] = 1
    FOR i = 1 TO MAXN - 1:
        fact[i] = fact[i-1] × i % MOD
    inv_fact[MAXN-1] = modPow(fact[MAXN-1], MOD - 2, MOD)
    FOR i = MAXN - 2 DOWNTO 0:
        inv_fact[i] = inv_fact[i+1] × (i + 1) % MOD

FUNCTION nCr(n, r):
    IF r < 0 OR r > n: RETURN 0
    RETURN fact[n] × inv_fact[r] % MOD × inv_fact[n-r] % MOD

// Call precompute() once before any nCr queries
```

---

## 5.7 Common Pitfalls

```
  ┌──────────────────────────────────────────────────────────┐
  │  Pitfall                      │  Fix                     │
  ├───────────────────────────────┼──────────────────────────┤
  │  Division instead of inverse  │  Use × modInverse(b)     │
  │  Negative modulo result       │  Add MOD before final %  │
  │  Overflow in multiplication   │  Use 64-bit integers     │
  │  Forgetting r > n check       │  Return 0 immediately    │
  │  Non-prime modulus            │  Can't use Fermat's      │
  │  p divides denominator        │  Use Lucas' or lift       │
  │  MAXN too small               │  Set ≥ max possible n    │
  └───────────────────────────────┴──────────────────────────┘
  
  Multiplication overflow check:
  If MOD ~ 10⁹, then a × b can be up to ~10¹⁸
  → Fits in 64-bit integer (max ~9.2 × 10¹⁸)
  
  If MOD ~ 10¹⁸ (rare), use 128-bit or __int128 in C++.
```

---

## 5.8 Decision Guide

```
  ┌─────────────────────────────────────────────────────────┐
  │                  What method to use?                    │
  │                                                         │
  │  n ≤ 60?                                                │
  │  ├── YES → Multiplicative formula (exact, fits 64-bit)  │
  │  └── NO                                                 │
  │      n ≤ 10⁶? and modulus is prime?                     │
  │      ├── YES → Factorial precompute + inverse ★         │
  │      └── NO                                             │
  │          n > 10⁶ but p is small?                      │
  │          ├── YES → Lucas' Theorem                       │
  │          └── NO                                         │
  │              Modulus not prime?                         │
  │              ├── Use CRT + Lucas on prime factors       │
  │              └── Or extended GCD for inverse            │
  └─────────────────────────────────────────────────────────┘
  
  ★ Most common in competitive programming
```

---

## Summary Table

| Technique | Constraint | Precompute | Per Query |
|-----------|-----------|------------|-----------|
| Factorial + Inverse | n ≤ 10⁶, p prime | O(n) | O(1) |
| Fermat's Inverse | p prime | — | O(log p) |
| Extended GCD | gcd(a,m)=1 | — | O(log m) |
| Iterative Inverses | 1..n | O(n) | O(1) |
| Lucas' Theorem | Large n, small p | O(p) | O(log_p n) |
| CRT + Lucas | Composite modulus | O(max pᵢ) | O(Σ log pᵢ) |

---

## Quick Revision Questions

1. **Why can't** we simply divide factorials and then take mod?
2. **How does** backfilling inverse factorials work? Derive the recurrence.
3. **Compute** C(10, 4) mod 7 using Lucas' theorem step by step.
4. **What is** the time complexity of the full precomputation pipeline?
5. **When would** you use Extended GCD over Fermat's method?
6. **Write** the complete code template for computing nCr mod 10⁹+7.

---

[← Previous: nCr Computation](04-ncr-computation.md) | [Back to README](../README.md) | [Next: Catalan Numbers →](06-catalan-numbers.md)
