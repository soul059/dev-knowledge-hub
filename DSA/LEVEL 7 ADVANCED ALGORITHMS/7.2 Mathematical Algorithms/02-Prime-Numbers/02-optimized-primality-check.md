# Chapter 2: Optimized Primality Check

[← Previous: Naive Primality](01-primality-check-naive.md) | [Back to README](../README.md) | [Next: Sieve of Eratosthenes →](03-sieve-of-eratosthenes.md)

---

## Overview

Building on the basic trial division, we apply the **6k ± 1 optimization** — exploiting the fact that all primes greater than 3 must be of the form 6k-1 or 6k+1. This reduces trial divisions by ~3× while keeping the algorithm simple.

---

## 2.1 The 6k ± 1 Insight

```
  Every integer falls into one of these residue classes mod 6:

  6k + 0 = 6k       → divisible by 6 (2 and 3)  → COMPOSITE
  6k + 1             → POSSIBLY PRIME ✓
  6k + 2 = 2(3k+1)  → divisible by 2            → COMPOSITE
  6k + 3 = 3(2k+1)  → divisible by 3            → COMPOSITE
  6k + 4 = 2(3k+2)  → divisible by 2            → COMPOSITE
  6k + 5 = 6(k+1)-1 → POSSIBLY PRIME ✓  (same as 6k-1)

  ┌──────────────────────────────────────────────────────────┐
  │  After eliminating multiples of 2 and 3, only numbers    │
  │  of the form 6k ± 1 remain as prime CANDIDATES.          │
  │                                                          │
  │  So we only need to test divisors of the form 6k ± 1!   │
  └──────────────────────────────────────────────────────────┘
```

### Visual: Which Numbers We Check

```
  Numbers 1-30, showing forms:
  
  1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30
        P     P     P           P       P           P       P           P       P
  
  6k-1 positions (5, 11, 17, 23, 29):     ●     ●      ●      ●      ●
  6k+1 positions (7, 13, 19, 25):            ●      ●      ●      ●
  
  We ONLY check these as potential divisors (and 2, 3 separately)
  
  Naive:  Check 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, ...  (ALL numbers)
  6k±1:   Check 2, 3, 5, 7, 11, 13, 17, 19, 23, 25, ... (1/3 of numbers)
```

---

## 2.2 Optimized Algorithm

```
FUNCTION isPrime(n):
    IF n ≤ 1: RETURN false
    IF n ≤ 3: RETURN true       // 2 and 3 are prime
    IF n % 2 == 0: RETURN false // even number
    IF n % 3 == 0: RETURN false // multiple of 3
    
    i = 5
    WHILE i * i ≤ n:
        IF n % i == 0:          // check 6k - 1
            RETURN false
        IF n % (i + 2) == 0:   // check 6k + 1
            RETURN false
        i = i + 6              // jump to next 6k - 1
    RETURN true

Time:  O(√n / 3) — about 3× faster than checking all odds
Space: O(1)
```

### How the Loop Works

```
  i starts at 5 and increments by 6:
  
  i = 5:   check 5 (6×1-1) and 7 (6×1+1)
  i = 11:  check 11 (6×2-1) and 13 (6×2+1)
  i = 17:  check 17 (6×3-1) and 19 (6×3+1)
  i = 23:  check 23 (6×4-1) and 25 (6×4+1)
  i = 29:  check 29 (6×5-1) and 31 (6×5+1)
  ...

  Pattern of divisors checked:
  2, 3, 5, 7, 11, 13, 17, 19, 23, 25, 29, 31, ...
              └─────────────────────────────────────┘
              These come from the 6k±1 loop
```

---

## 2.3 Detailed Traces

### Trace: Is 127 Prime?

```
  n = 127
  127 > 3              ✓
  127 % 2 = 1 ≠ 0     ✓  (odd)
  127 % 3 = 1 ≠ 0     ✓  (not mult of 3)
  
  √127 ≈ 11.27
  
  i = 5:   5² = 25 ≤ 127?  YES
           127 % 5  = 2 ≠ 0   ✓
           127 % 7  = 1 ≠ 0   ✓
  i = 11:  11² = 121 ≤ 127? YES
           127 % 11 = 6 ≠ 0   ✓
           127 % 13 = 10 ≠ 0  ✓
  i = 17:  17² = 289 ≤ 127? NO → EXIT
  
  Return true → 127 is PRIME ✓
  
  Only 6 mod operations (vs 10 with odd-only, vs 125 naive)
```

### Trace: Is 221 Prime?

```
  n = 221
  221 > 3              ✓
  221 % 2 = 1          ✓
  221 % 3 = 2          ✓
  
  √221 ≈ 14.87
  
  i = 5:   5² = 25 ≤ 221?  YES
           221 % 5  = 1 ≠ 0    ✓
           221 % 7  = 4 ≠ 0    ✓
  i = 11:  11² = 121 ≤ 221? YES
           221 % 11 = 1 ≠ 0    ✓
           221 % 13 = 0         ✗ → COMPOSITE!

  221 = 13 × 17
  
  Found in 6 mod operations.
```

---

## 2.4 Performance Comparison

```
  Number of mod operations to test n = 1,000,003 (prime):

  Method              Checks    Time (relative)
  ──────────────────────────────────────────────
  Naive (2..n-1)     1,000,001     1000×
  √n (2..√n)             1,000       3×
  Skip evens         ~500            1.5×
  6k±1               ~333            1×

  The 6k±1 method checks only 1/3 of numbers up to √n.
```

### Fraction of Numbers Checked

```
  All numbers:        ████████████████████████  100%
  Odd numbers only:   ████████████             50%
  6k±1 only:          ████████                 33%
  
  Note: Not all 6k±1 numbers are prime (e.g., 25 = 5²)
  But ALL primes > 3 ARE of the form 6k±1.
  The optimization is about divisors we test, not about them being prime.
```

---

## 2.5 Further Optimizations (Conceptual)

### Wheel Factorization

```
  6k±1 is a "wheel" of size 6 that skips multiples of 2 and 3.
  
  We can extend this to skip multiples of 2, 3, AND 5:
  
  Wheel of size 30 (= 2×3×5):
  Check only positions: 1, 7, 11, 13, 17, 19, 23, 29  (mod 30)
  That's 8/30 = 26.7% of numbers  (vs 33% for 6k±1)

  Wheel of size 210 (= 2×3×5×7):
  Check only 48/210 = 22.9% of numbers

  Diminishing returns: each larger wheel adds complexity
  for marginal speedup.
  
  ┌───── Wheel Sizes ─────────────────────────────┐
  │  Size 2:   50.0% checked  (skip evens)        │
  │  Size 6:   33.3% checked  (6k±1)              │
  │  Size 30:  26.7% checked                      │
  │  Size 210: 22.9% checked                      │
  │  Optimal:  ~0% (check only primes — but we    │
  │            need primes to find primes!)         │
  └────────────────────────────────────────────────┘
```

---

## 2.6 Common Pitfalls

```
  ⚠️ Pitfall 1: Forgetting i = 2 check
     n = 4 → without even check, loop starts at 5, 5²=25 > 4
     → incorrectly returns true!

  ⚠️ Pitfall 2: Using i < √n instead of i*i ≤ n
     Floating point √n can be inaccurate!
     Always use i*i ≤ n for exact comparison.

  ⚠️ Pitfall 3: Not handling n = 2 and n = 3
     These need special cases before the main loop.

  ⚠️ Pitfall 4: Integer overflow in i*i
     For very large n, i*i might overflow. Use:
     i ≤ n/i  (but be careful with negative division)
     or cast to long/64-bit integer.
```

---

## 2.7 Problem-Solving Applications

### Application 1: Next Prime

```
FUNCTION nextPrime(n):
    candidate = n + 1
    IF candidate % 2 == 0:
        candidate += 1    // start from odd
    WHILE NOT isPrime(candidate):
        candidate += 2    // only check odds
    RETURN candidate

  Trace: nextPrime(14)
  candidate = 15 → 15%3=0, not prime
  candidate = 17 → isPrime(17) = true
  Return 17
```

### Application 2: Prime Counting in Range

```
FUNCTION countPrimesInRange(a, b):
    count = 0
    FOR n = a TO b:
        IF isPrime(n):
            count += 1
    RETURN count

  For large ranges, use Sieve (next chapter) instead!
```

---

## Summary Table

| Optimization | What It Skips | Checks/√n | Speedup |
|-------------|---------------|-----------|---------|
| None | Nothing | √n | 1× |
| Skip evens | 50% of candidates | √n/2 | 2× |
| 6k±1 | 67% of candidates | √n/3 | 3× |
| Wheel(30) | 73% of candidates | √n/3.75 | 3.75× |

---

## Quick Revision Questions

1. **Explain the 6k±1 rule** — why are all primes > 3 of this form?
2. **Walk through the optimized isPrime** for n = 341. Is it prime?
3. **Why do we use `i*i ≤ n`** instead of `i ≤ √n`?
4. **What is wheel factorization?** How does the wheel of size 30 work?
5. **How would you find all primes** between 100 and 200 using this method?
6. **Compare the number of mod operations** for testing n = 10⁶ + 3 with naive vs 6k±1.

---

[← Previous: Naive Primality](01-primality-check-naive.md) | [Back to README](../README.md) | [Next: Sieve of Eratosthenes →](03-sieve-of-eratosthenes.md)
