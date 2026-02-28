# Chapter 1: Divisibility

[← Back to README](../README.md) | [Next: Factors and Multiples →](02-factors-and-multiples.md)

---

## Overview

Divisibility is the most fundamental concept in number theory. It defines when one integer divides another without a remainder, forming the foundation for primes, GCD, modular arithmetic, and virtually every mathematical algorithm.

---

## 1.1 What is Divisibility?

**Definition:** An integer `a` **divides** integer `b` (written `a | b`) if there exists an integer `k` such that `b = a × k`.

```
  a | b  means  b = a × k   for some integer k

  Example:  3 | 12  because  12 = 3 × 4   (k = 4)
            5 | 35  because  35 = 5 × 7   (k = 7)
            4 ∤ 10  because  10 / 4 = 2.5  (not an integer)
```

### ASCII Visualization: Division on Number Line

```
  Multiples of 3 on a number line:
  
  0     3     6     9    12    15    18    21    24
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────►
  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓
  
  Is 3 | 14?
  0     3     6     9    12  14 15
  ├─────┼─────┼─────┼─────┼──✗──┼────►
                              ↑
                         14 is NOT a multiple of 3
                         14 = 3 × 4 + 2  (remainder = 2)
```

---

## 1.2 The Division Algorithm

For any integers `a` (dividend) and `b` (divisor, b > 0), there exist **unique** integers `q` (quotient) and `r` (remainder) such that:

$$a = b \times q + r, \quad 0 \le r < b$$

```
  ┌──────────────────────────────────────┐
  │        DIVISION ALGORITHM            │
  │                                      │
  │   a = b × q + r                      │
  │                                      │
  │   where: a = dividend                │
  │          b = divisor   (b > 0)       │
  │          q = quotient  (⌊a/b⌋)      │
  │          r = remainder (a mod b)     │
  │          0 ≤ r < b                   │
  │                                      │
  │   a is divisible by b  ⟺  r = 0    │
  └──────────────────────────────────────┘
```

### Step-by-Step Trace

```
  Example: a = 23, b = 5

  Step 1: Compute q = ⌊23/5⌋ = ⌊4.6⌋ = 4
  Step 2: Compute r = 23 - 5 × 4 = 23 - 20 = 3
  Step 3: Verify: 23 = 5 × 4 + 3  ✓
  Step 4: 0 ≤ 3 < 5  ✓
  
  Result: quotient = 4, remainder = 3
  Since r ≠ 0, we say 5 does NOT divide 23.
```

---

## 1.3 Divisibility Rules (Quick Tests)

These rules help determine divisibility without performing full division:

| Divisor | Rule | Example |
|---------|------|---------|
| **2** | Last digit is even (0,2,4,6,8) | 438 → 8 is even ✓ |
| **3** | Sum of digits divisible by 3 | 621 → 6+2+1=9, 3\|9 ✓ |
| **4** | Last two digits divisible by 4 | 1324 → 24/4=6 ✓ |
| **5** | Last digit is 0 or 5 | 735 → ends in 5 ✓ |
| **6** | Divisible by both 2 and 3 | 312 → even, 3+1+2=6 ✓ |
| **7** | Double last digit, subtract from rest | 343 → 34-2×3=28, 7\|28 ✓ |
| **8** | Last three digits divisible by 8 | 1160 → 160/8=20 ✓ |
| **9** | Sum of digits divisible by 9 | 729 → 7+2+9=18, 9\|18 ✓ |
| **11** | Alternating sum of digits divisible by 11 | 1023 → 1-0+2-3=0, 11\|0 ✓ |

---

## 1.4 Properties of Divisibility

### Fundamental Properties

```
  ┌─────────────────────────────────────────────────────────┐
  │              DIVISIBILITY PROPERTIES                     │
  ├─────────────────────────────────────────────────────────┤
  │                                                         │
  │  1. Reflexivity:    a | a  (every number divides itself)│
  │                                                         │
  │  2. Transitivity:   if a | b  and  b | c  then  a | c  │
  │                                                         │
  │  3. Linearity:      if a | b  and  a | c               │
  │                     then  a | (b×m + c×n)               │
  │                     for any integers m, n                │
  │                                                         │
  │  4. Multiplication: if a | b  then  a | (b × c)        │
  │                     for any integer c                    │
  │                                                         │
  │  5. Division bound:  if a | b  and  b ≠ 0              │
  │                      then  |a| ≤ |b|                    │
  │                                                         │
  │  6. Unity:           1 | a  for every integer a         │
  │                                                         │
  │  7. Zero:            a | 0  for every nonzero a         │
  │                                                         │
  └─────────────────────────────────────────────────────────┘
```

### Transitivity Proof Trace

```
  Given: a | b  and  b | c
  Prove: a | c

  Step 1: a | b  means  b = a × k₁  for some integer k₁
  Step 2: b | c  means  c = b × k₂  for some integer k₂
  Step 3: Substitute: c = (a × k₁) × k₂ = a × (k₁ × k₂)
  Step 4: Let k = k₁ × k₂ (an integer)
  Step 5: c = a × k  ⟹  a | c  ∎

  Example: 2 | 6  and  6 | 24  ⟹  2 | 24  ✓
```

---

## 1.5 Algorithmic Divisibility Check

### Pseudocode

```
FUNCTION isDivisible(a, b):
    IF b == 0:
        RETURN Error  // Division by zero
    IF a % b == 0:
        RETURN true
    ELSE:
        RETURN false
```

### Finding All Divisors of N

```
FUNCTION findDivisors(n):
    divisors = empty list
    FOR i = 1 TO √n:
        IF n % i == 0:
            ADD i to divisors
            IF i ≠ n / i:
                ADD (n / i) to divisors
    SORT divisors
    RETURN divisors
```

### Step-by-Step Trace: Find Divisors of 36

```
  n = 36, √36 = 6

  i = 1: 36 % 1 == 0  → add 1 and 36/1 = 36  → {1, 36}
  i = 2: 36 % 2 == 0  → add 2 and 36/2 = 18  → {1, 2, 18, 36}
  i = 3: 36 % 3 == 0  → add 3 and 36/3 = 12  → {1, 2, 3, 12, 18, 36}
  i = 4: 36 % 4 == 0  → add 4 and 36/4 = 9   → {1, 2, 3, 4, 9, 12, 18, 36}
  i = 5: 36 % 5 == 1  → skip
  i = 6: 36 % 6 == 0  → add 6 and 36/6 = 6   → same! add only 6
                                                → {1, 2, 3, 4, 6, 9, 12, 18, 36}
  
  Result: 9 divisors of 36
```

### Why Only Check Up to √n?

```
  Key Insight:
  ┌──────────────────────────────────────────────────────┐
  │  If d | n, then both d and n/d are divisors.         │
  │  One of them must be ≤ √n and the other ≥ √n.       │
  │                                                      │
  │  Proof: If both d > √n AND n/d > √n, then           │
  │         d × (n/d) > √n × √n = n                     │
  │         But d × (n/d) = n. Contradiction!            │
  └──────────────────────────────────────────────────────┘

  Divisors of 36 in pairs:
  
  d:     1    2    3    4    6 ← √36
  n/d:  36   18   12    9    6
        ▲                    ▲
        large partner    meets at √n
```

---

## 1.6 Complexity Analysis

| Operation | Time Complexity | Space Complexity |
|-----------|----------------|-----------------|
| Single divisibility check `a % b` | O(1) | O(1) |
| Find all divisors (naive) | O(n) | O(√n) divisors on average |
| Find all divisors (optimized) | O(√n) | O(√n) |
| Divisibility test using rules | O(d) where d = # digits | O(1) |

---

## 1.7 Common Patterns in Problems

### Pattern 1: Count Divisors in a Range

```
  Problem: Count numbers in [1, n] divisible by d
  Answer:  ⌊n / d⌋
  
  Example: Count multiples of 7 in [1, 50]
  Answer:  ⌊50/7⌋ = 7
  They are: 7, 14, 21, 28, 35, 42, 49
```

### Pattern 2: Inclusion-Exclusion for Multiple Divisors

```
  Problem: Count numbers in [1, n] divisible by a OR b
  
  |A ∪ B| = |A| + |B| - |A ∩ B|
           = ⌊n/a⌋ + ⌊n/b⌋ - ⌊n/lcm(a,b)⌋

  Example: Count numbers in [1, 30] divisible by 3 or 5
  = ⌊30/3⌋ + ⌊30/5⌋ - ⌊30/15⌋
  = 10 + 6 - 2
  = 14
```

### Pattern 3: Pair Sum Divisibility

```
  Problem: Given array, count pairs (i,j) where (a[i]+a[j]) % k == 0
  
  Key Insight: (a[i]+a[j]) % k == 0
               ⟺ (a[i]%k + a[j]%k) % k == 0
  
  Approach: Group elements by their remainder when divided by k
            Count complementary pairs
```

---

## 1.8 Real-World Applications

| Application | How Divisibility Is Used |
|-------------|------------------------|
| Hash tables | `index = key % table_size` |
| Calendar calculations | Day of week = days % 7 |
| Circular buffers | `pos = index % buffer_size` |
| Check digits (ISBN, UPC) | Weighted digit sum divisibility |
| Time conversion | seconds → hours: seconds / 3600, remainder for minutes |
| Memory alignment | Address must be divisible by word size |

---

## Summary Table

| Concept | Key Formula/Rule | Complexity |
|---------|-----------------|------------|
| Divisibility | a \| b ⟺ b mod a = 0 | O(1) |
| Division algorithm | a = bq + r, 0 ≤ r < b | O(1) |
| Find all divisors | Check 1 to √n | O(√n) |
| Count multiples in [1,n] | ⌊n/d⌋ | O(1) |
| Inclusion-exclusion | \|A∪B\| = \|A\|+\|B\|-\|A∩B\| | O(1) |
| Divisor pair symmetry | d and n/d come in pairs | — |

---

## Quick Revision Questions

1. **What does `a | b` mean mathematically?** State the formal definition.
2. **Why do we only need to check up to √n** when finding all divisors of n?
3. **How many numbers between 1 and 100 are divisible by 6?** Compute using the floor formula.
4. **If 4 | 20 and 20 | 60, what can we conclude?** Which property is this?
5. **How would you count numbers from 1 to 1000 divisible by 3 or 7 but not both?** Write the inclusion-exclusion formula.
6. **What is the time complexity** of finding all divisors of n using the optimized approach?

---

[← Back to README](../README.md) | [Next: Factors and Multiples →](02-factors-and-multiples.md)
