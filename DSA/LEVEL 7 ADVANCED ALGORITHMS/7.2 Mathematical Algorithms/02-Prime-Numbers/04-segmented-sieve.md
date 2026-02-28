# Chapter 4: Segmented Sieve

[← Previous: Sieve of Eratosthenes](03-sieve-of-eratosthenes.md) | [Back to README](../README.md) | [Next: Prime Factorization →](05-prime-factorization.md)

---

## Overview

The standard Sieve of Eratosthenes requires O(n) memory, which becomes impractical for large n (e.g., n = 10⁹). The **Segmented Sieve** solves this by processing the range in small segments, using only O(√n) memory while still finding all primes up to n.

---

## 4.1 The Problem with Standard Sieve

```
  ┌──────────────────────────────────────────────────────────┐
  │  Standard sieve for n = 10⁹:                            │
  │    Memory needed: 10⁹ bytes ≈ 1 GB (boolean array)      │
  │    Or ~125 MB with bitset                                │
  │    → Too much for most competitive programming judges!   │
  │                                                          │
  │  Segmented sieve for n = 10⁹:                            │
  │    Memory: O(√n) ≈ 31,623 entries ≈ 32 KB               │
  │    → Fits in L1 cache! (very fast)                       │
  └──────────────────────────────────────────────────────────┘
```

---

## 4.2 Core Idea

```
  1. Find all primes up to √n using standard sieve
  2. Process the range [2, n] in segments of size Δ ≈ √n
  3. For each segment [lo, hi]:
     - Start with all numbers marked as prime
     - For each small prime p, cross out its multiples in [lo, hi]
     - Remaining unmarked numbers are prime

  ┌──────────────────────────────────────────────────────────┐
  │    [2 ........ √n]  [√n+1 ....... 2√n]  [2√n+1 ... 3√n]│
  │    Segment 0         Segment 1           Segment 2       │
  │    (standard sieve)  (use small primes)  (use small primes)
  │         ...                                              │
  │                              [... ... ... ... ... n]     │
  │                              Last segment                │
  └──────────────────────────────────────────────────────────┘
```

---

## 4.3 Algorithm (Pseudocode)

```
FUNCTION segmentedSieve(n):
    limit = √n
    
    // Step 1: Standard sieve for primes up to √n
    smallPrimes = sieveOfEratosthenes(limit)
    
    // Step 2: Process segments
    segSize = limit                    // segment size ≈ √n
    result = copy of smallPrimes       // primes from step 1
    
    FOR lo = limit + 1 TO n STEP segSize:
        hi = min(lo + segSize - 1, n)
        
        // Boolean array for this segment
        mark[0 .. hi-lo] = all true
        
        // Step 3: Cross out multiples of each small prime
        FOR each prime p in smallPrimes:
            // Find first multiple of p ≥ lo
            start = ⌈lo / p⌉ × p
            IF start == p:
                start = p * p  // don't cross out p itself
            
            // Cross out multiples in segment
            FOR j = start TO hi STEP p:
                mark[j - lo] = false
        
        // Step 4: Collect primes in this segment
        FOR i = 0 TO hi - lo:
            IF mark[i]:
                ADD (lo + i) to result
    
    RETURN result
```

---

## 4.4 Step-by-Step Trace (n = 30, segment size = 5)

```
  Step 1: √30 ≈ 5.47 → standard sieve up to 5
  Small primes: [2, 3, 5]

  Segment 0: [2, 5] — already found by standard sieve
  Primes so far: {2, 3, 5}

  ═══════════════════════════════════════════
  Segment 1: [6, 10]
  Initialize: mark = [T, T, T, T, T]  (indices 0-4 = numbers 6-10)

  Cross out multiples of 2 in [6,10]:
    start = ⌈6/2⌉ × 2 = 6
    j=6: mark[0]=F  j=8: mark[2]=F  j=10: mark[4]=F
    mark = [F, T, F, T, F]   (6,8,10 crossed)

  Cross out multiples of 3 in [6,10]:
    start = ⌈6/3⌉ × 3 = 6
    j=6: already F  j=9: mark[3]=F
    mark = [F, T, F, F, F]   (9 crossed)

  Cross out multiples of 5 in [6,10]:
    start = ⌈6/5⌉ × 5 = 10
    j=10: already F
    mark = [F, T, F, F, F]

  Primes in segment: 7
  Primes so far: {2, 3, 5, 7}

  ═══════════════════════════════════════════
  Segment 2: [11, 15]
  mark = [T, T, T, T, T]

  Multiples of 2: 12, 14 → mark = [T, F, T, F, T]
  Multiples of 3: 12, 15 → mark = [T, F, T, F, F]
  Multiples of 5: 15     → already F

  Primes: 11, 13
  Primes so far: {2, 3, 5, 7, 11, 13}

  ═══════════════════════════════════════════
  Segment 3: [16, 20]
  mark = [T, T, T, T, T]

  Multiples of 2: 16, 18, 20 → mark = [F, T, F, T, F]
  Multiples of 3: 18         → already F

  Primes: 17, 19
  Primes so far: {2, 3, 5, 7, 11, 13, 17, 19}

  ═══════════════════════════════════════════
  Segment 4: [21, 25]
  mark = [T, T, T, T, T]

  Multiples of 2: 22, 24     → mark = [T, F, T, F, T]
  Multiples of 3: 21, 24     → mark = [F, F, T, F, T]
  Multiples of 5: 25         → mark = [F, F, T, F, F]

  Primes: 23
  Primes so far: {2, 3, 5, 7, 11, 13, 17, 19, 23}

  ═══════════════════════════════════════════
  Segment 5: [26, 30]
  mark = [T, T, T, T, T]

  Multiples of 2: 26, 28, 30 → mark = [F, T, F, T, F]
  Multiples of 3: 27, 30     → mark = [F, T, F, F, F]

  Primes: 29
  Final: {2, 3, 5, 7, 11, 13, 17, 19, 23, 29}  ✓
```

---

## 4.5 Finding First Multiple in Segment

```
  Given prime p and segment starting at lo:
  
  First multiple of p that is ≥ lo:
  
  Method 1: start = ⌈lo / p⌉ × p
  Method 2: start = lo + ((p - lo % p) % p)

  If start < p*p, use p*p instead (smaller multiples
  are already handled as composites of smaller primes).

  Example: p = 7, lo = 50
  ⌈50/7⌉ = 8 → start = 8 × 7 = 56

  Example: p = 3, lo = 100
  ⌈100/3⌉ = 34 → start = 34 × 3 = 102
```

---

## 4.6 Primes in a Given Range [L, R]

```
  Common problem: Find all primes in [L, R] where R can be large
  but R - L is small (≤ 10⁶).

FUNCTION primesInRange(L, R):
    limit = √R
    smallPrimes = sieveOfEratosthenes(limit)
    
    mark[0 .. R-L] = all true
    
    IF L == 1: mark[0] = false   // 1 is not prime
    
    FOR each prime p in smallPrimes:
        start = max(p*p, ⌈L/p⌉ × p)
        FOR j = start TO R STEP p:
            mark[j - L] = false
    
    primes = []
    FOR i = 0 TO R - L:
        IF mark[i]:
            ADD (L + i) to primes
    RETURN primes

  Example: Primes in [100, 120]
  Small primes ≤ √120 ≈ 10.9 → {2, 3, 5, 7}
  
  mark[0..20] = all true (representing 100-120)
  
  Cross multiples of 2: 100,102,104,106,108,110,112,114,116,118,120
  Cross multiples of 3: 102,105,108,111,114,117,120
  Cross multiples of 5: 100,105,110,115,120
  Cross multiples of 7: 105,112,119
  
  Remaining (primes): 101, 103, 107, 109, 113
```

---

## 4.7 Complexity Analysis

```
  ┌──────────────────────────────────────────────────────────┐
  │  SEGMENTED SIEVE COMPLEXITY:                             │
  │                                                          │
  │  Time:  O(n log log n)  — same as standard sieve         │
  │         (total work across all segments is identical)     │
  │                                                          │
  │  Space: O(√n)                                            │
  │         √n for small primes + √n for segment array       │
  │                                                          │
  │  RANGE SIEVE [L, R]:                                     │
  │  Time:  O((R-L+1) log log R + √R)                       │
  │  Space: O(R - L + √R)                                   │
  └──────────────────────────────────────────────────────────┘
```

### Memory Comparison

```
  n = 10⁹

  Standard sieve:     ~1 GB (10⁹ bytes)  ✗ Too much
  Segmented sieve:    ~32 KB (√10⁹ ≈ 31K) ✓ Fits in cache!

  Cache-friendliness makes segmented sieve faster in practice
  even though asymptotic complexity is the same.
```

---

## 4.8 Problem-Solving Patterns

### Pattern: Count Primes in [L, R]

```
  1. If R ≤ 10⁷: Use standard sieve + prefix count
  2. If R ≤ 10¹² but R-L ≤ 10⁶: Use range sieve
  3. If R ≤ 10¹⁸ but R-L ≤ 10⁶: Use range sieve (√R ≤ 10⁹ 
     but only ~31K primes needed up to √R)
```

### Pattern: kth Prime

```
  Iterate segments until k primes are found.
  No need to sieve all the way to n if we only need k primes.
```

---

## Summary Table

| Aspect | Standard Sieve | Segmented Sieve |
|--------|---------------|-----------------|
| Time | O(n log log n) | O(n log log n) |
| Space | O(n) | O(√n) |
| Max practical n | ~10⁷ | ~10⁹ or more |
| Cache friendly | No (for large n) | Yes |
| Range queries | Need full sieve | Supports [L, R] directly |

---

## Quick Revision Questions

1. **Why is the standard sieve impractical** for n = 10⁹?
2. **Explain the segmented sieve strategy** — how does it use O(√n) memory?
3. **How do you find the first multiple** of prime p in a range starting at lo?
4. **Trace the range sieve** for primes in [50, 70].
5. **Why is the segmented sieve faster in practice** despite the same asymptotic complexity?
6. **How would you find** all primes between 10⁹ and 10⁹ + 10⁶?

---

[← Previous: Sieve of Eratosthenes](03-sieve-of-eratosthenes.md) | [Back to README](../README.md) | [Next: Prime Factorization →](05-prime-factorization.md)
