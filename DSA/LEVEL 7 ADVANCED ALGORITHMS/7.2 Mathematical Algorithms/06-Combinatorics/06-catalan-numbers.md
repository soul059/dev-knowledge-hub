# Chapter 6: Catalan Numbers

[← Previous: nCr with Modulo](05-ncr-with-modulo.md) | [Back to README](../README.md) | [Next: Unit 7 - Basic Probability →](../07-Probability-Basics/01-basic-probability.md)

---

## Overview

The **Catalan numbers** are a sequence of natural numbers that appear in countless combinatorial problems — from counting valid parenthesizations to binary search trees. The $n$-th Catalan number is:

$$C_n = \frac{1}{n+1}\binom{2n}{n} = \frac{(2n)!}{(n+1)!\,n!}$$

First few values: **1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, ...**

---

## 6.1 Closed-Form Formula

```
         1       ⎛ 2n ⎞      (2n)!
  Cₙ = ───── × ⎜    ⎟  =  ──────────
        n+1      ⎝  n ⎠    (n+1)! × n!

  Equivalent form:
  
  Cₙ = C(2n, n) - C(2n, n+1)

  ┌────────────────────────────────────────────────────┐
  │   n │  Cₙ    │  C(2n,n) │  C(2n,n)/(n+1)         │
  ├─────┼────────┼──────────┼────────────────────────-─┤
  │  0  │    1   │     1    │  1/1 = 1                 │
  │  1  │    1   │     2    │  2/2 = 1                 │
  │  2  │    2   │     6    │  6/3 = 2                 │
  │  3  │    5   │    20    │  20/4 = 5                │
  │  4  │   14   │    70    │  70/5 = 14               │
  │  5  │   42   │   252    │  252/6 = 42              │
  │  6  │  132   │   924    │  924/7 = 132             │
  └─────┴────────┴──────────┴──────────────────────────┘
```

---

## 6.2 Recurrence Relation

```
  C₀ = 1
  Cₙ = Σ Cᵢ × Cₙ₋₁₋ᵢ     for i = 0 to n-1

  ┌────────────────────────────────────────────────────────┐
  │  C₀ = 1                                               │
  │  C₁ = C₀×C₀ = 1                                      │
  │  C₂ = C₀×C₁ + C₁×C₀ = 1+1 = 2                       │
  │  C₃ = C₀×C₂ + C₁×C₁ + C₂×C₀ = 2+1+2 = 5            │
  │  C₄ = C₀×C₃ + C₁×C₂ + C₂×C₁ + C₃×C₀                │
  │     = 5 + 2 + 2 + 5 = 14                              │
  └────────────────────────────────────────────────────────┘

  Why this recurrence?
  Think of balanced parentheses: (A)B
  The first matched pair "(..)" encloses i pairs (Cᵢ ways)
  and the rest has n-1-i pairs (Cₙ₋₁₋ᵢ ways).
```

---

## 6.3 Classic Interpretations

### Interpretation 1: Balanced Parentheses

```
  Cₙ = number of valid arrangements of n pairs of parentheses
  
  n=3, C₃=5:
  
  ((()))    (()())    (())()    ()(())    ()()()
  
  Mapping to recurrence:
  (  [inner]  )  [remaining]
  ↑              ↑
  first match    rest after first matching pair
```

### Interpretation 2: Binary Search Trees

```
  Cₙ = number of structurally distinct BSTs with n nodes
  
  n=3 (keys 1,2,3), C₃=5:
  
    1        1        2        3      3
     \        \      / \      /      /
      2        3    1   3   1       2
       \      /              \     /
        3    2                2   1
```

### Interpretation 3: Lattice Paths (Ballot Problem)

```
  Cₙ = paths from (0,0) to (n,n) using R and U steps,
        NEVER crossing ABOVE the diagonal y = x
  
  For n=3:
  
  3 ·  ·  · ★     ★ = destination
    | ╱│  │╱      Paths stay ON or BELOW diagonal
  2 ·  ·  ·
    |  │╱ │
  1 ·  ·  ·       
    | ╱| ╱│
  0 ★──·──·──
    0  1  2  3
  
  These are called Dyck paths.
  Total unrestricted paths: C(2n, n)
  Paths that TOUCH above diagonal: C(2n, n+1)
  Valid paths: C(2n, n) - C(2n, n+1) = Cₙ ← Reflection principle
```

### Interpretation 4: Triangulations

```
  Cₙ = number of ways to triangulate a convex polygon with n+2 sides

  n=3 (pentagon, 5 sides):
  
    ·───·         ·───·         ·───·
   /|  /│        / \ / \       /│   │\
  · | / │       · ──·── ·     · │   │ ·
   \|/  │        \ / \ /       \│   │/
    ·───·         ·───·         ·───·
  
  (not all 5 shown — C₃ = 5 triangulations)
```

### Interpretation 5: Full Binary Trees

```
  Cₙ = number of full binary trees with n+1 leaves (n internal nodes)
  
  A full binary tree: every node has 0 or 2 children
  
  n=3 (3 internal, 4 leaves), C₃=5:
  
      ○          ○         ○        ○         ○
     / \        / \       / \      / \       / \  
    ○   △      ○   △     △   △    △   ○     △   ○
   / \        / \              / \         / \
  ○   △      △   ○            ○   △      △   ○
 / \            / \           / \            / \
△   △          △   △        △   △          △   △
```

---

## 6.4 Computation Methods

### Method 1: Direct Formula with Modular Inverse

```
FUNCTION catalan(n):
    // Cₙ = C(2n, n) / (n+1)
    RETURN nCr(2*n, n) × modInverse(n + 1, MOD) % MOD

  Using precomputed factorials:
    = fact[2*n] × inv_fact[n] % MOD × inv_fact[n] % MOD
      × modInverse(n+1, MOD) % MOD

  Time: O(1) after O(n) precomputation
```

### Method 2: Alternative Formula (Avoids Separate Inverse)

```
FUNCTION catalan(n):
    // Cₙ = C(2n, n) - C(2n, n+1)
    RETURN (nCr(2*n, n) - nCr(2*n, n+1) + MOD) % MOD

  No extra inverse needed — just two nCr calls!
```

### Method 3: DP Using Recurrence

```
FUNCTION catalan_dp(n):
    C[0] = 1
    FOR i = 1 TO n:
        C[i] = 0
        FOR j = 0 TO i - 1:
            C[i] = (C[i] + C[j] × C[i-1-j]) % MOD
    RETURN C[n]

  Time:  O(n²)
  Space: O(n)
  
  Useful when n is small and you need ALL Catalan numbers up to n.
```

---

## 6.5 Trace: Computing C₄

```
  Method 1 (Direct):
  C₄ = C(8, 4) / 5 = 70 / 5 = 14 ✓

  Method 2 (Subtraction):
  C₄ = C(8, 4) - C(8, 5) = 70 - 56 = 14 ✓

  Method 3 (Recurrence):
  C₀ = 1
  C₁ = C₀×C₀ = 1
  C₂ = C₀×C₁ + C₁×C₀ = 1 + 1 = 2
  C₃ = C₀×C₂ + C₁×C₁ + C₂×C₀ = 2 + 1 + 2 = 5
  C₄ = C₀×C₃ + C₁×C₂ + C₂×C₁ + C₃×C₀
     = 5 + 2 + 2 + 5 = 14 ✓
```

---

## 6.6 More Applications

```
  ┌────────────────────────────────────────────────────────────┐
  │  Problem                              │  Catalan Number    │
  ├───────────────────────────────────────┼────────────────────┤
  │  Balanced parentheses with n pairs    │  Cₙ               │
  │  BSTs with n nodes                    │  Cₙ               │
  │  Full binary trees with n+1 leaves    │  Cₙ               │
  │  Dyck paths on n×n grid              │  Cₙ               │
  │  Triangulations of (n+2)-gon         │  Cₙ               │
  │  Non-crossing partitions of n+1 pts  │  Cₙ               │
  │  Stack-sortable permutations of n    │  Cₙ               │
  │  Ways to multiply n+1 matrices       │  Cₙ               │
  │  Mountain ranges with n upstrokes    │  Cₙ               │
  │  Paths in grid avoiding diagonal     │  Cₙ               │
  └───────────────────────────────────────┴────────────────────┘
```

---

## 6.7 Growth Rate

```
  Asymptotic formula (Stirling):
  
  Cₙ ~ 4ⁿ / (n^(3/2) × √π)
  
  Growth is exponential (~4ⁿ) but slower than C(2n,n) ~ 4ⁿ/√(πn).
  
  For competitive programming: always use modular arithmetic
  for n > 30 (C₃₀ ≈ 5.9 × 10⁸ already near 10⁹+7).
```

---

## 6.8 Extended: Ballot Problem

```
  In an election, candidate A gets a votes, B gets b votes (a > b).
  Probability that A is STRICTLY ahead throughout the count:
  
  P = (a - b) / (a + b)
  
  Number of such vote sequences:
  (a - b) / (a + b) × C(a + b, a)

  Connection to Catalan:
  When a = b + 1, count of valid sequences = C_{b} (Catalan!)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Formula | Cₙ = C(2n,n)/(n+1) |
| Alt Formula | Cₙ = C(2n,n) - C(2n,n+1) |
| Recurrence | Cₙ = Σ Cᵢ × Cₙ₋₁₋ᵢ |
| First values | 1, 1, 2, 5, 14, 42, 132, 429 |
| Growth | ~4ⁿ / (n√(πn)) |
| Key interpretation | Balanced parentheses |
| DP complexity | O(n²) time, O(n) space |
| Direct formula | O(1) per query after O(n) precomputation |

---

## Quick Revision Questions

1. **Compute** C₅ using both the direct formula and the recurrence.
2. **How many** structurally distinct BSTs can be formed with 5 nodes?
3. **Why does** the parentheses interpretation lead to the recurrence?
4. **Derive** Cₙ = C(2n,n) - C(2n,n+1) from the lattice path model.
5. **List** five different combinatorial objects counted by Catalan numbers.
6. **What is** C₁₀ mod 10⁹+7? Compute using the direct formula.

---

[← Previous: nCr with Modulo](05-ncr-with-modulo.md) | [Back to README](../README.md) | [Next: Unit 7 - Basic Probability →](../07-Probability-Basics/01-basic-probability.md)
