# Chapter 1: Primality Check — Naive Approach

[← Previous: Euclidean Algorithm](../01-Number-Theory-Basics/06-euclidean-algorithm.md) | [Back to README](../README.md) | [Next: Optimized Primality Check →](02-optimized-primality-check.md)

---

## Overview

Primality testing — determining whether a number is prime — is a foundational problem in computer science. We start with the simplest brute-force approach, understand its limitations, and build intuition for optimizations.

---

## 1.1 The Brute Force Approach

### Idea

Check every number from 2 to n-1. If any divides n, it's composite.

```
FUNCTION isPrimeNaive(n):
    IF n < 2:
        RETURN false
    FOR i = 2 TO n - 1:
        IF n % i == 0:
            RETURN false    // found a divisor → composite
    RETURN true             // no divisor found → prime

Time Complexity:  O(n)
Space Complexity: O(1)
```

### Trace: Is 17 Prime?

```
  n = 17
  Check: 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16
  
  i=2:  17%2 = 1 ≠ 0  → continue
  i=3:  17%3 = 2 ≠ 0  → continue
  i=4:  17%4 = 1 ≠ 0  → continue
  i=5:  17%5 = 2 ≠ 0  → continue
  ...
  i=16: 17%16 = 1 ≠ 0 → continue
  
  No divisor found → 17 is PRIME ✓
  
  Total checks: 15 (way too many!)
```

### Trace: Is 15 Prime?

```
  n = 15
  i=2:  15%2 = 1 ≠ 0  → continue
  i=3:  15%3 = 0       → COMPOSITE! Return false
  
  Early exit after 2 checks (found divisor 3)
  15 = 3 × 5
```

---

## 1.2 First Optimization: Check up to √n

```
  ┌──────────────────────────────────────────────────────────┐
  │  KEY INSIGHT:                                            │
  │  If n = a × b and a ≤ b, then a ≤ √n                   │
  │                                                          │
  │  So we only need to check divisors from 2 to √n         │
  │  If no divisor is found in that range, n is prime.       │
  └──────────────────────────────────────────────────────────┘
```

```
FUNCTION isPrimeSqrt(n):
    IF n < 2:
        RETURN false
    FOR i = 2 TO √n:          // i * i ≤ n
        IF n % i == 0:
            RETURN false
    RETURN true

Time Complexity:  O(√n)
Space Complexity: O(1)
```

### Comparison: Checks Required

```
  n          Naive O(n)    √n Optimized     Speedup
  ────────────────────────────────────────────────────
  100            98              10           ~10×
  10,000       9,998           100          ~100×
  1,000,000   999,998         1,000       ~1000×
  10⁹          ~10⁹           ~31,623    ~31,623×

  Visual:
  
  n = 10,000
  Naive:     ████████████████████████████████████████ (9998 checks)
  √n:        ████ (100 checks)
```

### Trace: Is 97 Prime? (√n method)

```
  n = 97, √97 ≈ 9.85 → check i = 2 to 9

  i=2: 97%2 = 1  → continue
  i=3: 97%3 = 1  → continue
  i=4: 97%4 = 1  → continue
  i=5: 97%5 = 2  → continue
  i=6: 97%6 = 1  → continue
  i=7: 97%7 = 6  → continue
  i=8: 97%8 = 1  → continue
  i=9: 97%9 = 7  → continue
  
  No divisor found → 97 is PRIME ✓
  Only 8 checks instead of 95!
```

---

## 1.3 Second Optimization: Skip Even Numbers

```
FUNCTION isPrimeBetter(n):
    IF n < 2: RETURN false
    IF n == 2: RETURN true
    IF n % 2 == 0: RETURN false     // eliminate all even numbers
    
    FOR i = 3 TO √n STEP 2:        // only check odd numbers
        IF n % i == 0:
            RETURN false
    RETURN true

Time: O(√n / 2) = O(√n) but ~2× faster in practice
```

### Why This Works

```
  All even numbers > 2 are composite.
  If n is odd, its factors must also be odd.
  
  So after checking 2, we only need to check: 3, 5, 7, 9, 11, ...
  
  Checks for n = 97:
  Before: 2, 3, 4, 5, 6, 7, 8, 9  (8 checks)
  After:  2, 3, 5, 7, 9            (5 checks)
```

---

## 1.4 Decision Flow

```
                    START
                      │
                      ▼
                ┌───────────┐
                │  n < 2 ?  │──YES──► return false
                └─────┬─────┘
                      │ NO
                      ▼
                ┌───────────┐
                │ n == 2 ?  │──YES──► return true
                └─────┬─────┘
                      │ NO
                      ▼
                ┌───────────┐
                │ n even ?  │──YES──► return false
                └─────┬─────┘
                      │ NO
                      ▼
              ┌──────────────┐
              │ i = 3        │
              │ while i²≤ n  │
              └──────┬───────┘
                     │
              ┌──────▼───────┐
              │ n % i == 0 ? │──YES──► return false
              └──────┬───────┘
                     │ NO
                     ▼
                  i += 2
                     │
              (loop back)
                     │
                     ▼
              return true (PRIME)
```

---

## 1.5 Edge Cases

```
  ┌─────────────────────────────────────────────────┐
  │  EDGE CASES TO HANDLE:                          │
  ├─────────────────────────────────────────────────┤
  │  n = 0   → false (not prime)                   │
  │  n = 1   → false (not prime)                   │
  │  n = 2   → true  (smallest prime)              │
  │  n = 3   → true  (prime)                       │
  │  n = 4   → false (2 × 2)                       │
  │  n < 0   → false (primes are positive by defn) │
  │  n = 10⁹ → need √n = ~31623 checks             │
  └─────────────────────────────────────────────────┘
```

---

## 1.6 When to Use Naive Methods

| Scenario | Recommended Method |
|----------|-------------------|
| Single number, n ≤ 10⁶ | √n trial division |
| Single number, n ≤ 10¹² | √n with 6k±1 (next chapter) |
| Many numbers, all ≤ 10⁷ | Sieve of Eratosthenes |
| Single number, n > 10¹⁸ | Miller-Rabin (Unit 10) |

---

## 1.7 Practice Problem Types

```
  Type 1: "Is n prime?"
  → Direct primality check

  Type 2: "Count primes in range [a, b]"
  → Loop + isPrime for each (or use sieve)

  Type 3: "Find the next prime after n"
  → Increment and test: n+1, n+2, ...
     (By Bertrand's postulate, next prime < 2n)

  Type 4: "Find all prime factors"
  → Trial division (covered in Unit 2, Chapter 5)
```

---

## Summary Table

| Method | Time | Space | Practical Use |
|--------|------|-------|---------------|
| Check 2 to n-1 | O(n) | O(1) | Never (too slow) |
| Check 2 to √n | O(√n) | O(1) | Single queries, n ≤ 10¹² |
| Skip evens | O(√n / 2) | O(1) | Better constant factor |
| 6k±1 trick | O(√n / 3) | O(1) | Next chapter |

---

## Quick Revision Questions

1. **Why is O(n) too slow** for primality testing of large numbers?
2. **Prove that checking up to √n is sufficient** — walk through the logic.
3. **How many mod operations** does the √n method need for n = 1,000,000?
4. **Why do we skip even numbers** after checking 2?
5. **What is the time complexity** of checking if all numbers from 1 to n are prime using trial division?
6. **Is 1 prime?** Why or why not?

---

[← Previous: Euclidean Algorithm](../01-Number-Theory-Basics/06-euclidean-algorithm.md) | [Back to README](../README.md) | [Next: Optimized Primality Check →](02-optimized-primality-check.md)
