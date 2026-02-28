# Chapter 3: Sieve of Eratosthenes

[← Previous: Optimized Primality](02-optimized-primality-check.md) | [Back to README](../README.md) | [Next: Segmented Sieve →](04-segmented-sieve.md)

---

## Overview

The Sieve of Eratosthenes is the most efficient way to find **all** primes up to a given limit n. Instead of testing each number individually, it eliminates composites in bulk, achieving O(n log log n) time — nearly linear!

---

## 3.1 Core Idea

```
  ┌──────────────────────────────────────────────────────────┐
  │  SIEVE OF ERATOSTHENES:                                  │
  │                                                          │
  │  1. Write all numbers from 2 to n                        │
  │  2. Start with the first uncrossed number (2)            │
  │  3. Cross out all its multiples (4, 6, 8, 10, ...)       │
  │  4. Move to the next uncrossed number                    │
  │  5. Repeat until you've processed all numbers ≤ √n       │
  │  6. All remaining uncrossed numbers are PRIME            │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.2 Step-by-Step Visualization (n = 30)

```
  Initial: all numbers 2-30 are unmarked (potential primes)
  
  [2] [3] [4] [5] [6] [7] [8] [9] [10] [11] [12] [13] [14] [15]
  [16] [17] [18] [19] [20] [21] [22] [23] [24] [25] [26] [27] [28] [29] [30]
```

### Pass 1: p = 2 (cross out multiples of 2)

```
  [2] [3]  ×  [5]  ×  [7]  ×  [9]  ×   [11]  ×   [13]  ×    ×
   ×  [17]  ×  [19]  ×  [21]  ×  [23]  ×   [25]  ×   [27]  ×  [29]  ×

  Crossed out: 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30
```

### Pass 2: p = 3 (cross out multiples of 3, starting from 9)

```
  [2] [3]  ×  [5]  ×  [7]  ×   ×   ×   [11]  ×   [13]  ×    ×
   ×  [17]  ×  [19]  ×    ×   ×  [23]  ×   [25]  ×    ×   ×  [29]  ×

  Newly crossed: 9, 15, 21, 27  (6,12,18,24,30 already crossed by 2)
```

### Pass 3: p = 5 (cross out multiples of 5, starting from 25)

```
  5² = 25 ≤ 30? YES
  
  [2] [3]  ×  [5]  ×  [7]  ×   ×   ×   [11]  ×   [13]  ×    ×
   ×  [17]  ×  [19]  ×    ×   ×  [23]  ×    ×   ×    ×   ×  [29]  ×

  Newly crossed: 25  (10,15,20,30 already crossed)
```

### Pass 4: p = 7 → 7² = 49 > 30 → STOP

```
  Final primes ≤ 30:
  2, 3, 5, 7, 11, 13, 17, 19, 23, 29
  
  Total: 10 primes
```

---

## 3.3 Algorithm (Pseudocode)

```
FUNCTION sieveOfEratosthenes(n):
    isPrime[0..n] = all true
    isPrime[0] = false
    isPrime[1] = false
    
    FOR p = 2 TO √n:
        IF isPrime[p]:                    // p is prime
            FOR j = p*p TO n STEP p:      // cross out multiples
                isPrime[j] = false
    
    primes = []
    FOR i = 2 TO n:
        IF isPrime[i]:
            ADD i to primes
    RETURN primes
```

### Why Start From p²?

```
  When crossing out multiples of p, we start from p² because:
  
  Multiples of p less than p² have already been crossed out
  by smaller primes.
  
  For p = 5:
    5 × 2 = 10  → already crossed by 2
    5 × 3 = 15  → already crossed by 3
    5 × 4 = 20  → already crossed by 2
    5 × 5 = 25  → NOT yet crossed! Start here.
    5 × 6 = 30  → already crossed by 2 (but we check anyway)
  
  General: p × k for any k < p has a prime factor < p,
  so it's already been eliminated.
```

---

## 3.4 Memory Layout (Boolean Array)

```
  Index:    0  1  2  3  4  5  6  7  8  9  10  11  12  13  14  15
  isPrime:  F  F  T  T  F  T  F  T  F  F   F   T   F   T   F   F

  Memory: each element = 1 byte (or 1 bit with bitset optimization)
  
  For n = 10⁷:
    Boolean array: 10 MB
    Bitset:        ~1.25 MB (8× compression)
```

---

## 3.5 Optimized Sieve (Skip Evens)

```
FUNCTION sieveOptimized(n):
    IF n < 2: RETURN []
    
    // Only store odd numbers: index i represents number 2i+1
    size = (n - 1) / 2
    isOddPrime[0..size] = all true    // index 0 = number 1 (mark false)
    isOddPrime[0] = false
    
    FOR i = 1 TO size:
        IF isOddPrime[i]:
            p = 2*i + 1              // actual number
            IF p*p > n: BREAK
            // Cross out odd multiples of p, starting from p²
            start = (p*p - 1) / 2
            FOR j = start TO size STEP p:
                isOddPrime[j] = false
    
    primes = [2]                     // 2 is special
    FOR i = 1 TO size:
        IF isOddPrime[i]:
            ADD (2*i + 1) to primes
    RETURN primes

  Space: n/2 instead of n (50% savings!)
```

---

## 3.6 Complexity Analysis

```
  ┌──────────────────────────────────────────────────────────────┐
  │  TIME COMPLEXITY: O(n log log n)                            │
  │                                                              │
  │  Proof sketch:                                               │
  │  Work done = Σ (n/p) for all primes p ≤ √n                 │
  │            = n × Σ (1/p) for primes p ≤ √n                 │
  │            = n × O(log log n)   [Mertens' theorem]          │
  │                                                              │
  │  This is nearly linear! log log n grows EXTREMELY slowly:   │
  │    n = 10⁶  → log log n ≈ 3                                │
  │    n = 10⁹  → log log n ≈ 3.3                              │
  │    n = 10¹⁸ → log log n ≈ 3.8                              │
  │                                                              │
  │  SPACE COMPLEXITY: O(n)                                      │
  │  With bitset: O(n/8) bytes                                   │
  └──────────────────────────────────────────────────────────────┘
```

### Comparison with Trial Division

```
  Task: Find all primes up to n = 10⁶

  Trial division (per number):       O(√n) per test
  Total for all numbers:             O(n × √n) = O(n^1.5)
                                     = 10⁹ operations 😰

  Sieve of Eratosthenes:             O(n log log n)
                                     ≈ 3 × 10⁶ operations 😊

  Speedup: ~333×
```

---

## 3.7 Building a Smallest Prime Factor (SPF) Sieve

```
FUNCTION buildSPF(n):
    spf[0..n] = {0, 1, 2, 3, ..., n}   // initialize spf[i] = i
    
    FOR i = 2 TO √n:
        IF spf[i] == i:                  // i is prime
            FOR j = i*i TO n STEP i:
                IF spf[j] == j:          // not yet assigned
                    spf[j] = i
    RETURN spf

  Result enables O(log n) factorization of ANY number ≤ n.
  
  spf[12] = 2  →  12/2 = 6  →  spf[6] = 2  →  6/2 = 3  →  spf[3] = 3  →  done
  12 = 2² × 3
```

---

## 3.8 Sieve Variations

### Linear Sieve (O(n) strict)

```
  Each composite is marked exactly ONCE by its smallest prime factor.

FUNCTION linearSieve(n):
    spf[0..n] = all 0
    primes = []
    
    FOR i = 2 TO n:
        IF spf[i] == 0:          // i is prime
            spf[i] = i
            ADD i to primes
        FOR each p in primes:
            IF p > spf[i] OR i*p > n:
                BREAK
            spf[i*p] = p         // p is smallest prime factor of i*p
    RETURN primes, spf

  Time: O(n) — each number visited exactly once!
  Space: O(n)
```

---

## 3.9 Common Problem Patterns

### Pattern 1: Count Primes Up to n

```
  Simply count true values in sieve array.
  
  FUNCTION countPrimes(n):
      sieve = sieveOfEratosthenes(n)
      RETURN count of true values in isPrime[2..n]
```

### Pattern 2: Sum of Primes Up to n

```
  Run sieve, then sum primes.
  Useful for Project Euler problems.
```

### Pattern 3: Prime Queries (Multiple Range Queries)

```
  Precompute prefix count:
  prefixPrimeCount[i] = number of primes ≤ i
  
  Primes in range [a, b] = prefixPrimeCount[b] - prefixPrimeCount[a-1]
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Time complexity | O(n log log n) — nearly linear |
| Space complexity | O(n) or O(n/8) with bitset |
| When to use | Need ALL primes up to n ≤ ~10⁷ |
| Start crossing from | p² (smaller multiples already handled) |
| Skip evens variant | Halves memory, ~2× faster |
| Linear sieve | O(n) strict, builds SPF simultaneously |
| SPF sieve | Enables O(log n) factorization |

---

## Quick Revision Questions

1. **Explain why we start crossing multiples from p²**, not 2p.
2. **What is the time complexity** of the Sieve of Eratosthenes and why?
3. **Trace the sieve for n = 20** — show which numbers get crossed at each step.
4. **How much memory** does a sieve for n = 10⁷ require (boolean vs bitset)?
5. **What advantage does the linear sieve** have over the standard sieve?
6. **How would you answer** "how many primes between 100 and 500?" using a sieve?

---

[← Previous: Optimized Primality](02-optimized-primality-check.md) | [Back to README](../README.md) | [Next: Segmented Sieve →](04-segmented-sieve.md)
