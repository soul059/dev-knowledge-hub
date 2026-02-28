# Chapter 4: Composite Numbers

[← Previous: Prime Numbers](03-prime-numbers.md) | [Back to README](../README.md) | [Next: GCD and LCM →](05-gcd-and-lcm.md)

---

## Overview

Composite numbers are the counterpart to primes — integers greater than 1 that are NOT prime. Understanding composites is essential for factorization, divisor problems, and sieve algorithms.

---

## 4.1 Definition

```
  ┌──────────────────────────────────────────────────────────┐
  │  COMPOSITE NUMBER:                                       │
  │    A natural number n > 1 that has MORE than              │
  │    two distinct positive divisors.                        │
  │                                                          │
  │  Equivalently: n is composite if n = a × b               │
  │    where 1 < a ≤ b < n                                   │
  │                                                          │
  │  Classification of natural numbers:                      │
  │    • 1            → neither prime nor composite          │
  │    • primes       → exactly 2 divisors                   │
  │    • composites   → 3 or more divisors                   │
  └──────────────────────────────────────────────────────────┘
```

### Visual Classification

```
  Numbers 1 to 20:

  1   →  UNIT (neither prime nor composite)
  2   →  PRIME     (divisors: 1, 2)
  3   →  PRIME     (divisors: 1, 3)
  4   →  COMPOSITE (divisors: 1, 2, 4)
  5   →  PRIME     (divisors: 1, 5)
  6   →  COMPOSITE (divisors: 1, 2, 3, 6)
  7   →  PRIME     (divisors: 1, 7)
  8   →  COMPOSITE (divisors: 1, 2, 4, 8)
  9   →  COMPOSITE (divisors: 1, 3, 9)
  10  →  COMPOSITE (divisors: 1, 2, 5, 10)
  11  →  PRIME
  12  →  COMPOSITE (divisors: 1, 2, 3, 4, 6, 12)
  13  →  PRIME
  14  →  COMPOSITE
  15  →  COMPOSITE
  16  →  COMPOSITE
  17  →  PRIME
  18  →  COMPOSITE
  19  →  PRIME
  20  →  COMPOSITE
```

---

## 4.2 Factorization Trees

Every composite number can be broken down into prime factors using a factorization tree.

### Building a Factorization Tree

```
  Example: 120

            120
           /   \
          2    60
              /  \
             2   30
                /  \
               2   15
                  /  \
                 3    5

  Reading leaves (bottom-up): 2 × 2 × 2 × 3 × 5
  Result: 120 = 2³ × 3 × 5
```

```
  Example: 360

              360
             /    \
            2     180
                 /    \
                2     90
                     /  \
                    2   45
                       /  \
                      3   15
                         /  \
                        3    5

  Result: 360 = 2³ × 3² × 5
```

### Multiple Valid Trees, Same Result

```
  72 = 2³ × 3²

  Tree 1:            Tree 2:           Tree 3:
       72                72                72
      /  \             /   \             /   \
     8    9           4    18           2    36
    / \  / \         / \  /  \            /   \
   2  4 3  3        2  2 2   9           2    18
     / \                    / \              /  \
    2   2                  3   3            2    9
                                               / \
                                              3   3
  All give: 2³ × 3²  (Unique factorization!)
```

---

## 4.3 Composite Detection Algorithm

```
FUNCTION isComposite(n):
    IF n < 2: RETURN false    // 0, 1 are neither
    IF n == 2 OR n == 3: RETURN false  // primes
    IF n % 2 == 0: RETURN true  // even composite
    IF n % 3 == 0: RETURN true  // divisible by 3
    
    i = 5
    WHILE i * i ≤ n:
        IF n % i == 0 OR n % (i + 2) == 0:
            RETURN true
        i += 6
    RETURN false   // it's prime, not composite
```

### Trace: Is 91 Composite?

```
  n = 91
  91 ≥ 2  ✓
  91 is odd ✓
  91 % 3 = 1 ≠ 0  → continue
  
  i = 5:  5² = 25 ≤ 91?  YES
          91 % 5 = 1  → no
          91 % 7 = 0  → YES! 91 is COMPOSITE (91 = 7 × 13)
  
  Many people think 91 is prime — it's a common trap!
```

---

## 4.4 Smallest and Largest Prime Factors

### Smallest Prime Factor (SPF) Sieve

```
FUNCTION buildSPF(n):
    spf[0..n] = {0, 1, 2, 3, 4, ... n}  // initialize spf[i] = i
    FOR i = 2 TO √n:
        IF spf[i] == i:          // i is prime
            FOR j = i*i TO n STEP i:
                IF spf[j] == j:  // not yet assigned
                    spf[j] = i   // smallest prime factor
    RETURN spf
```

### Trace: Build SPF for n = 20

```
  Initial: spf = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20]

  i=2 (prime): mark multiples starting from 4
    j=4:  spf[4]=2    j=6: spf[6]=2    j=8: spf[8]=2
    j=10: spf[10]=2   j=12: spf[12]=2  j=14: spf[14]=2
    j=16: spf[16]=2   j=18: spf[18]=2  j=20: spf[20]=2

  i=3 (prime, spf[3]=3): mark multiples starting from 9
    j=9:  spf[9]=3    j=15: spf[15]=3  (others already marked)

  i=4: spf[4]=2 ≠ 4, skip (not prime)

  Final SPF:
  n:    0  1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20
  spf:  0  1  2  3  2  5  2  7  2  3   2  11   2  13   2   3   2  17   2  19   2
```

### Using SPF for Fast Factorization

```
FUNCTION factorize(n, spf):
    factors = {}
    WHILE n > 1:
        p = spf[n]
        count = 0
        WHILE n % p == 0:
            n = n / p
            count += 1
        factors[p] = count
    RETURN factors
```

### Trace: Factorize 180 using SPF

```
  n = 180
  
  Step 1: spf[180] = 2, count 2's
          180/2 = 90, 90/2 = 45, 45%2 ≠ 0 → 2² recorded, n = 45
  
  Step 2: spf[45] = 3, count 3's
          45/3 = 15, 15/3 = 5, 5%3 ≠ 0 → 3² recorded, n = 5
  
  Step 3: spf[5] = 5, count 5's
          5/5 = 1 → 5¹ recorded, n = 1
  
  Result: 180 = 2² × 3² × 5¹  ✓
  Time: O(log n) per factorization!
```

---

## 4.5 Highly Composite Numbers

Numbers with more divisors than any smaller positive integer.

```
  n     d(n)   Prime factorization
  ─────────────────────────────────
  1       1     -
  2       2     2
  4       3     2²
  6       4     2 × 3
  12      6     2² × 3
  24      8     2³ × 3
  36      9     2² × 3²
  48     10     2⁴ × 3
  60     12     2² × 3 × 5
  120    16     2³ × 3 × 5
  180    18     2² × 3² × 5
  240    20     2⁴ × 3 × 5
  360    24     2³ × 3² × 5
  720    30     2⁴ × 3² × 5
  
  Pattern: Exponents in factorization are non-increasing
           Uses consecutive primes starting from 2
```

---

## 4.6 Semi-Primes

```
  ┌──────────────────────────────────────────────────────────┐
  │  SEMI-PRIME:                                             │
  │    A number that is the product of exactly TWO primes    │
  │    (not necessarily distinct)                             │
  │                                                          │
  │    n = p × q  where p, q are prime                       │
  │                                                          │
  │  Examples:                                               │
  │    4 = 2 × 2    6 = 2 × 3    9 = 3 × 3                  │
  │    10 = 2 × 5   14 = 2 × 7   15 = 3 × 5                 │
  │    21 = 3 × 7   22 = 2 × 11  25 = 5 × 5                 │
  │                                                          │
  │  Importance: RSA cryptography uses large semi-primes     │
  │  (product of two large primes → hard to factor)          │
  └──────────────────────────────────────────────────────────┘
```

---

## 4.7 Problem-Solving Patterns

### Pattern: Count Composites in Range

```
  Composites in [1, n] = n - 1 - π(n)
  
  Where π(n) = number of primes ≤ n
  Subtract 1 because 1 is neither prime nor composite
  
  Example: Composites in [1, 20]
  Primes: 2,3,5,7,11,13,17,19 → π(20) = 8
  Composites = 20 - 1 - 8 = 11
```

### Pattern: Check if n has Exactly k Prime Factors

```
FUNCTION countPrimeFactors(n):
    count = 0
    d = 2
    WHILE d * d ≤ n:
        WHILE n % d == 0:
            count += 1
            n = n / d
        d += 1
    IF n > 1:
        count += 1  // remaining prime factor
    RETURN count

  Trace for n = 12 (= 2² × 3):
  d=2: 12/2=6 (count=1), 6/2=3 (count=2), 3%2≠0
  d=3: 3²=9 > 3, exit loop
  n=3 > 1: count=3
  Result: 3 prime factors (with multiplicity: 2, 2, 3)
```

### Pattern: Distinct vs Total Prime Factors

```
  n = 360 = 2³ × 3² × 5

  Ω(360) = 3 + 2 + 1 = 6  (total prime factors, with multiplicity)
  ω(360) = 3               (distinct prime factors: {2, 3, 5})
```

---

## 4.8 Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Composite check | O(√n) | O(1) |
| Build SPF sieve | O(n log log n) | O(n) |
| Factorize using SPF | O(log n) | O(log n) |
| Count prime factors | O(√n) | O(1) |
| Trial division factorization | O(√n) | O(log n) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Composite definition | More than 2 divisors (n > 1, not prime) |
| Factorization tree | Visual breakdown into prime factors |
| Unique factorization | Fundamental Theorem of Arithmetic |
| SPF sieve | Precompute smallest prime factor for O(log n) factorization |
| Highly composite | More divisors than any smaller number |
| Semi-prime | Product of exactly two primes |
| Ω(n) vs ω(n) | Total vs distinct prime factor count |

---

## Quick Revision Questions

1. **What distinguishes 1, primes, and composites?** How many divisors does each have?
2. **Why is 91 composite?** What are its factors?
3. **Explain the SPF sieve** — how does it enable O(log n) factorization?
4. **What is a highly composite number?** Give 5 examples.
5. **How are semi-primes used in cryptography?**
6. **Given n = 504, build its factorization tree** and express as prime powers.

---

[← Previous: Prime Numbers](03-prime-numbers.md) | [Back to README](../README.md) | [Next: GCD and LCM →](05-gcd-and-lcm.md)
