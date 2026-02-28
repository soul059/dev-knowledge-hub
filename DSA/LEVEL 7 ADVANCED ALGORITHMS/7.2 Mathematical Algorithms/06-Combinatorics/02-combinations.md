# Chapter 2: Combinations

[← Previous: Permutations](01-permutations.md) | [Back to README](../README.md) | [Next: Pascal's Triangle →](03-pascals-triangle.md)

---

## Overview

A **combination** is a selection of items where order doesn't matter. This chapter covers the binomial coefficient $\binom{n}{r}$, its properties, and computation methods.

---

## 2.1 Definition

$$\binom{n}{r} = C(n, r) = \frac{n!}{r!(n-r)!}$$

```
  "Choose r items from n, order DOESN'T matter"

  C(5, 2) = 5! / (2! × 3!) = 120 / (2 × 6) = 10

  ┌──────┬────────────┬──────────┐
  │ n, r │  C(n, r)   │ Selections │
  ├──────┼────────────┼──────────┤
  │ 4, 0 │     1      │ {}        │
  │ 4, 1 │     4      │ A,B,C,D  │
  │ 4, 2 │     6      │ AB,AC,AD,│
  │      │            │ BC,BD,CD │
  │ 4, 3 │     4      │ ABC, ... │
  │ 4, 4 │     1      │ ABCD     │
  └──────┴────────────┴──────────┘
  
  Note: C(n,r) = P(n,r) / r!  (remove ordering)
```

---

## 2.2 Key Properties

```
  ┌──────────────────────────────────────────────────────────┐
  │  PROPERTIES OF C(n, r):                                  │
  │                                                          │
  │  1. Symmetry:    C(n,r) = C(n, n-r)                     │
  │  2. Boundary:    C(n,0) = C(n,n) = 1                    │
  │  3. Pascal's:    C(n,r) = C(n-1,r-1) + C(n-1,r)        │
  │  4. Sum of row:  Σ C(n,r) for r=0..n = 2ⁿ              │
  │  5. Vandermonde: C(m+n,r) = Σ C(m,k)×C(n,r-k)          │
  │  6. Absorption:  C(n,r) = (n/r) × C(n-1, r-1)          │
  │  7. C(n,r) × r! = P(n,r)                               │
  └──────────────────────────────────────────────────────────┘
```

---

## 2.3 Computation Methods

### Method 1: Using Factorials

```
FUNCTION nCr_factorial(n, r):
    IF r > n: RETURN 0
    IF r > n - r: r = n - r    // optimization: C(n,r) = C(n,n-r)
    RETURN factorial(n) / (factorial(r) * factorial(n - r))

  ⚠️ Overflows fast! Only for small n.
```

### Method 2: Iterative (Overflow-Safe for Moderate n)

```
FUNCTION nCr_iterative(n, r):
    IF r > n - r: r = n - r
    result = 1
    FOR i = 0 TO r - 1:
        result = result * (n - i)
        result = result / (i + 1)    // always divides evenly!
    RETURN result

  Why does it always divide evenly?
  At step i, result = C(n, i) × (n-i), and dividing by (i+1)
  gives C(n, i+1), which is always an integer.

Time: O(r)
Space: O(1)
```

### Trace: C(6, 2) Iteratively

```
  n=6, r=2 (already ≤ n-r=4)
  
  i=0: result = 1 × 6 / 1 = 6
  i=1: result = 6 × 5 / 2 = 15
  
  C(6, 2) = 15 ✓
```

### Method 3: Pascal's Triangle DP

```
FUNCTION nCr_pascal(n, r):
    dp[0][0] = 1
    FOR i = 1 TO n:
        dp[i][0] = 1
        FOR j = 1 TO min(i, r):
            dp[i][j] = dp[i-1][j-1] + dp[i-1][j]
    RETURN dp[n][r]

Time: O(n × r)
Space: O(n × r) or O(r) with rolling array
```

---

## 2.4 Combinations with Modular Arithmetic

```
  C(n, r) mod p:
  
  Precompute factorials and inverse factorials:
  
  fact[0] = 1
  FOR i = 1 TO n: fact[i] = fact[i-1] × i % MOD
  
  inv_fact[n] = modPow(fact[n], MOD-2, MOD)
  FOR i = n-1 DOWNTO 0: inv_fact[i] = inv_fact[i+1] × (i+1) % MOD
  
  C(n, r) = fact[n] × inv_fact[r] × inv_fact[n-r] % MOD

  Precompute: O(n)
  Per query: O(1)
```

---

## 2.5 Stars and Bars

```
  Count non-negative integer solutions to:
  x₁ + x₂ + ... + xₖ = n

  Answer: C(n + k - 1, k - 1)

  ┌──────────────────────────────────────────────────────────┐
  │  Visualization:                                          │
  │  Distribute n stars among k bins separated by k-1 bars   │
  │                                                          │
  │  n=5, k=3:  ★★|★★|★  means x₁=2, x₂=2, x₃=1          │
  │             ★★★||★★   means x₁=3, x₂=0, x₃=2          │
  │                                                          │
  │  Total arrangements: C(5+3-1, 3-1) = C(7,2) = 21       │
  └──────────────────────────────────────────────────────────┘

  For POSITIVE integer solutions (xᵢ ≥ 1):
  Substitute yᵢ = xᵢ - 1, then Σyᵢ = n - k
  Answer: C(n - 1, k - 1)
```

---

## 2.6 Inclusion-Exclusion with Combinations

```
  Problem: Count integers in [1, 1000] divisible by 2, 3, or 5.

  |A₂| = ⌊1000/2⌋ = 500
  |A₃| = ⌊1000/3⌋ = 333
  |A₅| = ⌊1000/5⌋ = 200
  |A₂∩A₃| = ⌊1000/6⌋ = 166
  |A₂∩A₅| = ⌊1000/10⌋ = 100
  |A₃∩A₅| = ⌊1000/15⌋ = 66
  |A₂∩A₃∩A₅| = ⌊1000/30⌋ = 33

  |A₂∪A₃∪A₅| = 500+333+200 - 166-100-66 + 33 = 734
```

---

## 2.7 Problem-Solving Patterns

### Pattern 1: Lattice Paths

```
  Grid paths from (0,0) to (m,n) using only right (R) and down (D):
  
  Total moves: m + n (m rights, n downs)
  Choose which m positions are R: C(m+n, m)

  Example: (0,0) to (3,2) → C(5, 3) = 10 paths
  
  ·──·──·──·
  |  |  |  |
  ·──·──·──·
  |  |  |  |
  ·──·──·──·
  
  One path: R R D R D = RRDRD
```

### Pattern 2: Subsets of Size r

```
  Choose r elements from n: C(n, r)
  Total subsets: 2ⁿ = Σ C(n, r) for r = 0 to n
```

### Pattern 3: Distributing Identical Items

```
  k identical items into n distinct boxes:
  Stars and bars: C(k+n-1, n-1)
```

---

## Summary Table

| Concept | Formula | When |
|---------|---------|------|
| Basic combination | n!/(r!(n-r)!) | Selection without order |
| Pascal's rule | C(n,r) = C(n-1,r-1)+C(n-1,r) | Recursive computation |
| Symmetry | C(n,r) = C(n,n-r) | Optimization |
| Stars and bars | C(n+k-1,k-1) | Distributing identical items |
| Lattice paths | C(m+n,m) | Grid path counting |
| Sum of row | Σ C(n,r) = 2ⁿ | Total subsets |
| Mod computation | fact × inv_fact × inv_fact | O(1) per query |

---

## Quick Revision Questions

1. **What is C(10, 3)?** Compute step by step.
2. **Prove** C(n, r) = C(n, n-r) using the formula.
3. **How many ways** to distribute 7 identical balls into 3 distinct boxes?
4. **How many lattice paths** from (0,0) to (4,3)?
5. **Compute C(100, 2) mod 10⁹+7** using the precomputation method.
6. **Why does** the iterative method always produce integer intermediate results?

---

[← Previous: Permutations](01-permutations.md) | [Back to README](../README.md) | [Next: Pascal's Triangle →](03-pascals-triangle.md)
