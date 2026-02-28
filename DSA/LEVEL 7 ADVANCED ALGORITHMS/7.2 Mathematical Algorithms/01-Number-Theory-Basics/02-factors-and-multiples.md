# Chapter 2: Factors and Multiples

[← Previous: Divisibility](01-divisibility.md) | [Back to README](../README.md) | [Next: Prime Numbers →](03-prime-numbers.md)

---

## Overview

Factors and multiples are two sides of the same coin. Understanding how to efficiently find, count, and work with them is essential for number theory problems and is a building block for prime factorization, GCD, and combinatorics.

---

## 2.1 Definitions

```
  ┌──────────────────────────────────────────────────────────┐
  │  FACTOR (Divisor):                                       │
  │    f is a factor of n  if  n % f == 0                    │
  │    i.e., f divides n evenly                              │
  │                                                          │
  │  MULTIPLE:                                               │
  │    m is a multiple of n  if  m = n × k  for integer k    │
  │    i.e., m is divisible by n                             │
  │                                                          │
  │  RELATIONSHIP:                                           │
  │    f is a factor of n  ⟺  n is a multiple of f          │
  └──────────────────────────────────────────────────────────┘
```

### Visual Relationship

```
       Factor Relationship             Multiple Relationship
       
  "3 is a factor of 12"          "12 is a multiple of 3"
  
       3 ──── divides ────► 12        3 ◄──── is multiplied ── 12
       
  Factors of 12:                 Multiples of 3:
  {1, 2, 3, 4, 6, 12}           {3, 6, 9, 12, 15, 18, ...}
  (finite set)                   (infinite set)
```

---

## 2.2 Finding All Factors

### Naive Approach: O(n)

```
FUNCTION findFactorsNaive(n):
    factors = []
    FOR i = 1 TO n:
        IF n % i == 0:
            ADD i to factors
    RETURN factors
```

### Optimized Approach: O(√n)

```
FUNCTION findFactors(n):
    factors = []
    FOR i = 1 TO √n:
        IF n % i == 0:
            ADD i to factors
            IF i ≠ n / i:
                ADD (n / i) to factors
    SORT factors
    RETURN factors
```

### Trace: Find Factors of 60

```
  n = 60, √60 ≈ 7.74, check i = 1 to 7

  i=1: 60%1=0  → add 1, 60    factors: {1, 60}
  i=2: 60%2=0  → add 2, 30    factors: {1, 2, 30, 60}
  i=3: 60%3=0  → add 3, 20    factors: {1, 2, 3, 20, 30, 60}
  i=4: 60%4=0  → add 4, 15    factors: {1, 2, 3, 4, 15, 20, 30, 60}
  i=5: 60%5=0  → add 5, 12    factors: {1, 2, 3, 4, 5, 12, 15, 20, 30, 60}
  i=6: 60%6=0  → add 6, 10    factors: {1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30, 60}
  i=7: 60%7=4  → skip

  Sorted Result: {1, 2, 3, 4, 5, 6, 10, 12, 15, 20, 30, 60}
  Total: 12 factors
```

### Factor Pair Diagram

```
  Factors of 60 as pairs (each pair multiplies to 60):

       1  ×  60  =  60
       2  ×  30  =  60
       3  ×  20  =  60
       4  ×  15  =  60
       5  ×  12  =  60
       6  ×  10  =  60
       ▲         ▲
       │         │
    ≤ √60     ≥ √60
    (≈7.74)
    
  Mirror line at √n:
  ├──1──2──3──4──5──6──┼──10──12──15──20──30──60──►
                       √60
```

---

## 2.3 Factor Count from Prime Factorization

⚡ **Key Insight:** If $n = p_1^{a_1} \times p_2^{a_2} \times \ldots \times p_k^{a_k}$, then:

$$\text{Number of divisors} = (a_1 + 1)(a_2 + 1) \ldots (a_k + 1)$$

### Trace: Count Divisors of 360

```
  Step 1: Prime factorize 360
  360 = 2³ × 3² × 5¹

  Step 2: Apply formula
  τ(360) = (3+1)(2+1)(1+1) = 4 × 3 × 2 = 24

  Step 3: Verify — 360 has exactly 24 divisors:
  1, 2, 3, 4, 5, 6, 8, 9, 10, 12, 15, 18,
  20, 24, 30, 36, 40, 45, 60, 72, 90, 120, 180, 360
```

### Why Does This Formula Work?

```
  n = p₁^a₁ × p₂^a₂ × ... × pₖ^aₖ

  Every divisor d of n has the form:
  d = p₁^b₁ × p₂^b₂ × ... × pₖ^bₖ

  where 0 ≤ bᵢ ≤ aᵢ for each i

  For p₁: b₁ can be 0, 1, 2, ..., a₁  → (a₁+1) choices
  For p₂: b₂ can be 0, 1, 2, ..., a₂  → (a₂+1) choices
  ...
  By multiplication principle: total = Π(aᵢ+1)
```

---

## 2.4 Sum of Factors (Divisor Sum)

$$\sigma(n) = \prod_{i=1}^{k} \frac{p_i^{a_i+1} - 1}{p_i - 1}$$

### Trace: Sum of Divisors of 12

```
  12 = 2² × 3¹

  σ(12) = (2⁰ + 2¹ + 2²) × (3⁰ + 3¹)
        = (1 + 2 + 4)     × (1 + 3)
        = 7 × 4
        = 28

  Verify: 1 + 2 + 3 + 4 + 6 + 12 = 28  ✓
```

---

## 2.5 Special Number Types

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    SPECIAL NUMBER TYPES                      │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  PERFECT NUMBER: σ(n) = 2n (sum of proper divisors = n)     │
  │    Examples: 6 (1+2+3=6), 28 (1+2+4+7+14=28)               │
  │                                                              │
  │  ABUNDANT NUMBER: σ(n) > 2n                                  │
  │    Example: 12 (1+2+3+4+6=16 > 12)                          │
  │                                                              │
  │  DEFICIENT NUMBER: σ(n) < 2n                                 │
  │    Example: 8 (1+2+4=7 < 8)                                  │
  │                                                              │
  │  AMICABLE NUMBERS: σ(a)-a = b AND σ(b)-b = a                │
  │    Example: (220, 284)                                       │
  │                                                              │
  │  HIGHLY COMPOSITE: More divisors than any smaller number     │
  │    Examples: 1, 2, 4, 6, 12, 24, 36, 48, 60, 120, ...       │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

---

## 2.6 Common Multiples

### Finding LCM (Least Common Multiple) — Preview

```
  Multiples of 4:  4,  8, 12, 16, 20, 24, 28, 32, 36, ...
  Multiples of 6:  6, 12, 18, 24, 30, 36, 42, ...
                       ↑           ↑       ↑
                 Common Multiples: 12, 24, 36, ...
                 
  LCM(4, 6) = 12  (the smallest common multiple)
  
  Formula: LCM(a, b) = (a × b) / GCD(a, b)
```

### Multiples in Programming

```
FUNCTION generateMultiples(n, count):
    result = []
    FOR i = 1 TO count:
        ADD (n × i) to result
    RETURN result

FUNCTION isMultiple(m, n):
    RETURN m % n == 0
```

---

## 2.7 Problem-Solving Patterns

### Pattern 1: Common Factor Counting

```
  Problem: Given array, count pairs with common factor > 1
  
  Approach:
  1. For each element, find its prime factors
  2. Group elements by their prime factors
  3. Count pairs within each group
  
  Array: [6, 12, 15, 10]
  Factors:
    6  → {2, 3}
    12 → {2, 3}
    15 → {3, 5}
    10 → {2, 5}
  
  Common factor 2: {6,12,10} → C(3,2) = 3 pairs
  Common factor 3: {6,12,15} → C(3,2) = 3 pairs
  Common factor 5: {15,10}   → C(2,2) = 1 pair
  Use inclusion-exclusion to avoid double counting
```

### Pattern 2: Maximum Number of Groups

```
  Problem: Divide n items into groups of equal size
  Answer: Any factor of n gives a valid group size
  
  Example: 24 items
  Valid group sizes: 1, 2, 3, 4, 6, 8, 12, 24
  Maximum equal groups: 24 (groups of 1)
  Minimum groups (>1 per group): 2 (groups of 12)
```

### Pattern 3: Factor Frequency with Sieve

```
  Problem: For each number 1..n, find its number of factors
  
  FUNCTION factorCountSieve(n):
      count[1..n] = all zeros
      FOR i = 1 TO n:
          FOR j = i, 2i, 3i, ..., n:    // all multiples of i
              count[j] += 1
      RETURN count
  
  Time: O(n log n)  [Harmonic series]
  
  Trace for n = 6:
  i=1: count[1]++, count[2]++, count[3]++, count[4]++, count[5]++, count[6]++
  i=2: count[2]++, count[4]++, count[6]++
  i=3: count[3]++, count[6]++
  i=4: count[4]++
  i=5: count[5]++
  i=6: count[6]++
  
  Result: count = [1, 2, 2, 3, 2, 4]
  Meaning: 1 has 1 factor, 2 has 2, 3 has 2, 4 has 3, 5 has 2, 6 has 4
```

---

## 2.8 Complexity Analysis

| Operation | Time Complexity | Space Complexity |
|-----------|----------------|-----------------|
| Find factors (naive) | O(n) | O(d) |
| Find factors (optimized) | O(√n) | O(d) |
| Factor count from prime factorization | O(√n) for factorization | O(log n) |
| Sum of factors | O(√n) | O(1) |
| Factor count sieve for [1..n] | O(n log n) | O(n) |
| Generate k multiples of n | O(k) | O(k) |

*d = number of divisors of n (typically O(n^(1/3)) on average)*

---

## 2.9 Real-World Applications

| Application | Usage |
|-------------|-------|
| Scheduling | LCM for repeating event synchronization |
| Tile layout | Factors determine valid tile sizes |
| Data partitioning | Factor determines equal partition sizes |
| Cryptography | Number of divisors impacts key strength |
| Competitive programming | Factor sieve for bulk queries |

---

## Summary Table

| Concept | Formula/Method | Key Point |
|---------|---------------|-----------|
| Factor of n | f where n % f == 0 | Factors come in pairs |
| Multiple of n | m = n × k | Infinite set |
| Factor count | Π(aᵢ + 1) from prime factorization | Efficient computation |
| Sum of factors | Π((pᵢ^(aᵢ+1) - 1)/(pᵢ - 1)) | Uses geometric series |
| Finding factors | Check 1 to √n | O(√n) |
| Factor sieve | Iterate multiples | O(n log n) for all [1,n] |

---

## Quick Revision Questions

1. **What is the relationship** between a factor and a multiple? Give an example.
2. **How many factors does 2⁴ × 3² × 7¹ have?** Apply the divisor count formula.
3. **Why do factors always come in pairs** around √n? Explain the proof.
4. **What is a perfect number?** Give the two smallest examples.
5. **Explain the factor sieve approach.** What is its time complexity and why?
6. **Given n = 90, find all its factor pairs.** Use the √n optimization.

---

[← Previous: Divisibility](01-divisibility.md) | [Back to README](../README.md) | [Next: Prime Numbers →](03-prime-numbers.md)
