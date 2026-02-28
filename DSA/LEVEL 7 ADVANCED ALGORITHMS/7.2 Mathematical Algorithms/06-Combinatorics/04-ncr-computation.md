# Chapter 4: nCr Computation

[← Previous: Pascal's Triangle](03-pascals-triangle.md) | [Back to README](../README.md) | [Next: nCr with Modulo →](05-ncr-with-modulo.md)

---

## Overview

Computing $\binom{n}{r}$ efficiently is a core competitive programming requirement. This chapter compares all major methods — recursive, DP table, multiplicative, and when each approach is appropriate.

---

## 4.1 Method 1: Naive Recursive

```
FUNCTION nCr(n, r):
    IF r == 0 OR r == n: RETURN 1
    RETURN nCr(n-1, r-1) + nCr(n-1, r)

Time:  O(2ⁿ) — exponential! Lots of repeated subproblems.
Space: O(n)  — recursion depth

  Recursion tree for nCr(5,2):
  
              nCr(5,2)
            /           \
       nCr(4,1)       nCr(4,2)
       /     \        /       \
   nCr(3,0) nCr(3,1) nCr(3,1) nCr(3,2)
            /   \     /   \     /    \
           ...  ...  ...  ...  ...   ...
  
  ⚠️ nCr(3,1) computed TWICE — and this gets much worse.
```

---

## 4.2 Method 2: Memoized Recursion

```
memo = {}

FUNCTION nCr(n, r):
    IF r == 0 OR r == n: RETURN 1
    IF (n, r) IN memo: RETURN memo[(n, r)]
    memo[(n, r)] = nCr(n-1, r-1) + nCr(n-1, r)
    RETURN memo[(n, r)]

Time:  O(n × r) — each (n, r) pair computed once
Space: O(n × r) — for the memo table
```

---

## 4.3 Method 3: DP Table (Pascal's Triangle)

```
FUNCTION nCr_dp(n, r):
    // Build partial Pascal's triangle
    dp[n+1][r+1]
    FOR i = 0 TO n:
        dp[i][0] = 1
        FOR j = 1 TO min(i, r):
            dp[i][j] = dp[i-1][j-1] + dp[i-1][j]
    RETURN dp[n][r]
```

### Trace: C(5, 3)

```
  Building dp table:
  
  i\j   0    1    2    3
  ──────────────────────
   0    1
   1    1    1
   2    1    2    1
   3    1    3    3    1
   4    1    4    6    4
   5    1    5   10   10  ← C(5,3) = 10 ✓
   
  Time:  O(n × r)
  Space: O(n × r) → can reduce to O(r) with 1D rolling
```

### 1D Rolling Array

```
FUNCTION nCr_1d(n, r):
    IF r > n - r: r = n - r
    dp = array of size r+1, all zeros
    dp[0] = 1
    FOR i = 1 TO n:
        FOR j = min(i, r) DOWNTO 1:  // RIGHT to LEFT!
            dp[j] = dp[j] + dp[j-1]
    RETURN dp[r]
    
  Time:  O(n × r)
  Space: O(r)
```

---

## 4.4 Method 4: Multiplicative Formula (Best for Single Queries)

```
  C(n, r) = n × (n-1) × ... × (n-r+1)
            ─────────────────────────────
                       r!

FUNCTION nCr_mult(n, r):
    IF r > n - r: r = n - r     // Optimization
    result = 1
    FOR i = 0 TO r - 1:
        result = result × (n - i)
        result = result / (i + 1)
    RETURN result

Time:  O(r) — or O(min(r, n-r)) with symmetry
Space: O(1)
```

### Trace: C(10, 3)

```
  r = min(3, 7) = 3
  
  i=0: result = 1 × 10 / 1 = 10
  i=1: result = 10 × 9 / 2  = 45
  i=2: result = 45 × 8 / 3  = 120
  
  C(10, 3) = 120 ✓
  
  Manual check: 10! / (3! × 7!) = 3628800 / (6 × 5040) = 120 ✓
```

### Why Division Always Yields an Integer

```
  After step i, the result equals C(n, i+1).
  
  Proof: At step i, we have computed:
    n × (n-1) × ... × (n-i)
    ────────────────────────
         (i+1)!
  
  This is exactly C(n, i+1), which is always an integer
  by definition (it counts combinations).
  
  Key insight: Do NOT accumulate numerator and denominator
  separately — divide at each step for safety!
```

---

## 4.5 Method 5: Using GCD for Safety

```
  When dividing at each step isn't clean enough
  (e.g., with non-standard types):

FUNCTION nCr_gcd(n, r):
    num = [n, n-1, ..., n-r+1]    // r terms
    den = [1, 2, ..., r]           // r terms
    FOR i = 0 TO r-1:
        FOR j = 0 TO r-1:
            g = gcd(num[i], den[j])
            num[i] /= g
            den[j] /= g
    result = 1
    FOR i = 0 TO r-1:
        result *= num[i]
    RETURN result
    
  Time:  O(r² log(max_value))
  Space: O(r)
  
  After GCD cancellation, all den[j] become 1,
  so the product of num[] is exactly C(n, r).
```

---

## 4.6 Method 6: Prime Factorization (Large n)

```
  Express C(n, r) as product of prime powers.
  
  Legendre's formula: exponent of prime p in n! is
    Σ ⌊n/pⁱ⌋  for i = 1, 2, ...

  Exponent of p in C(n, r) = Σ (⌊n/pⁱ⌋ - ⌊r/pⁱ⌋ - ⌊(n-r)/pⁱ⌋)
  
  Each term is 0 or 1 (equals carry at position i in base-p addition!)

FUNCTION nCr_prime_factorization(n, r):
    primes = sieve up to n
    result = 1
    FOR each prime p ≤ n:
        exp = 0
        pk = p
        WHILE pk ≤ n:
            exp += ⌊n/pk⌋ - ⌊r/pk⌋ - ⌊(n-r)/pk⌋
            pk *= p
        result *= p^exp
    RETURN result

  Time:  O(n / ln(n) × log_p(n))   — iterating over primes
  Space: O(n)                        — for the sieve
  
  Useful when you need exact value for VERY large n
  and can't fit n! in memory. Also useful for checking
  divisibility properties of C(n, r).
```

---

## 4.7 Comparison of Methods

```
  ┌─────────────────┬──────────┬─────────┬──────────────────────┐
  │ Method          │   Time   │  Space  │  Best When           │
  ├─────────────────┼──────────┼─────────┼──────────────────────┤
  │ Naive Recursive │ O(2ⁿ)   │ O(n)    │ Never in practice    │
  │ Memoized        │ O(n×r)  │ O(n×r)  │ Multiple queries     │
  │ DP Table        │ O(n×r)  │ O(n×r)  │ Need many C(i,j)     │
  │ 1D Rolling      │ O(n×r)  │ O(r)    │ Need one full row    │
  │ Multiplicative  │ O(r)    │ O(1)    │ Single query ★       │
  │ GCD-based       │ O(r²)   │ O(r)    │ Non-standard types   │
  │ Prime Factorize │ O(n)    │ O(n)    │ Exact large values   │
  │ Mod Precompute  │ O(1)    │ O(n)    │ Many queries mod p   │
  └─────────────────┴──────────┴─────────┴──────────────────────┘
  
  ★ For a SINGLE query without modular arithmetic,
    the multiplicative method is almost always best.
```

---

## 4.8 Edge Cases and Pitfalls

```
  ┌───────────────────────────────────────────────────────────┐
  │  Case             │  Result  │  Notes                     │
  ├───────────────────┼──────────┼────────────────────────────┤
  │  r < 0            │    0     │  Can't choose negative     │
  │  r > n            │    0     │  Not enough items           │
  │  r = 0            │    1     │  Empty set                  │
  │  r = n            │    1     │  Take everything            │
  │  r = 1            │    n     │  One item from n            │
  │  n = 0, r = 0     │    1     │  0! / (0! × 0!) = 1       │
  │  Large n (overflow)│  ⚠️    │  Use modular or big int     │
  └───────────────────┴──────────┴────────────────────────────┘
  
  Always add: IF r < 0 OR r > n: RETURN 0
  Always use: IF r > n - r: r = n - r  (symmetry optimization)
```

---

## 4.9 Practice Problem Analysis

```
  Problem: Count paths in grid from (0,0) to (n,m)
  
  Approach 1: DP — O(n×m) time, O(n×m) space
  dp[i][j] = dp[i-1][j] + dp[i][j-1]
  
  Approach 2: Combinatorics — O(n+m) time, O(1) space
  Answer = C(n+m, n)  — choose which n steps go down
  
  When n,m ≤ 1000: DP works fine
  When n,m ≤ 10⁶ with mod: Use factorial precompute + inverse
  When n,m ≤ 10¹⁸ with mod p < 10⁶: Use Lucas' theorem
```

---

## Summary Table

| Method | Time | Space | Overflow Safe | Best For |
|--------|------|-------|---------------|----------|
| Multiplicative | O(min(r,n-r)) | O(1) | Moderate | Single query |
| DP 1D | O(n×r) | O(r) | Use long long | Multiple C(n,*) |
| DP 2D | O(n×r) | O(n×r) | Use long long | Lookup table |
| Factorial+Inverse | O(n) pre / O(1) query | O(n) | Mod arithmetic | Many queries mod p |
| Prime Factor | O(n) | O(n) | Exact | Very large exact values |

---

## Quick Revision Questions

1. **Why is** the naive recursive approach O(2ⁿ)?
2. **Implement** C(n,r) using the multiplicative formula and trace C(8,3).
3. **When would** you choose the DP approach over the multiplicative formula?
4. **What is** the output of C(0,0)? C(5,0)? C(5,6)?
5. **Why must** the 1D rolling array iterate right to left?
6. **How does** Legendre's formula help compute C(n,r) for large n?

---

[← Previous: Pascal's Triangle](03-pascals-triangle.md) | [Back to README](../README.md) | [Next: nCr with Modulo →](05-ncr-with-modulo.md)
