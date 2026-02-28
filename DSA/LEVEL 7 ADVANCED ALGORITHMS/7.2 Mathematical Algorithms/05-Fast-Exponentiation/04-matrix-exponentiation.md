# Chapter 4: Matrix Exponentiation

[← Previous: Recursive Fast Power](03-recursive-fast-power.md) | [Back to README](../README.md) | [Next: Fibonacci in O(log n) →](05-fibonacci-in-log-n.md)

---

## Overview

Matrix exponentiation is one of the most powerful techniques in competitive programming. By encoding a linear recurrence as a matrix multiplication, we can compute the n-th term in $O(k^3 \log n)$ where $k$ is the matrix size — usually small (2×2 or 3×3).

---

## 4.1 Why Matrices?

```
  Linear recurrences like:
    F(n) = F(n-1) + F(n-2)        (Fibonacci)
    f(n) = 2f(n-1) + 3f(n-2)     (general 2nd order)
    f(n) = af(n-1) + bf(n-2) + cf(n-3)  (3rd order)

  ┌──────────────────────────────────────────────────────────┐
  │  Can be computed in O(n) via iteration.                  │
  │  But what if n = 10¹⁸?                                  │
  │                                                          │
  │  Matrix exponentiation: O(k³ × log n)                   │
  │  For k=2: O(8 × 60) = O(480) operations for n=10¹⁸!    │
  └──────────────────────────────────────────────────────────┘
```

---

## 4.2 Matrix Multiplication Review

```
  For 2×2 matrices:

  ┌     ┐   ┌     ┐   ┌                 ┐
  │ a b │ × │ e f │ = │ ae+bg   af+bh   │
  │ c d │   │ g h │   │ ce+dg   cf+dh   │
  └     ┘   └     ┘   └                 ┘

  Properties:
  ✓ Associative: (AB)C = A(BC)
  ✗ NOT commutative: AB ≠ BA in general
  ✓ Identity matrix: AI = IA = A

  Time: O(k³) for k×k matrices
  
  ┌     ┐
  │ 1 0 │  ← Identity matrix I₂
  │ 0 1 │
  └     ┘
```

---

## 4.3 Encoding Fibonacci as Matrix

```
  Fibonacci: F(n) = F(n-1) + F(n-2)

  ┌        ┐   ┌     ┐   ┌        ┐
  │ F(n+1) │ = │ 1 1 │ × │  F(n)  │
  │  F(n)  │   │ 1 0 │   │ F(n-1) │
  └        ┘   └     ┘   └        ┘

  Let T = ┌     ┐
          │ 1 1 │
          │ 1 0 │
          └     ┘

  Then: ┌        ┐       ┌      ┐
        │ F(n+1) │ = T^n │ F(1) │
        │  F(n)  │       │ F(0) │
        └        ┘       └      ┘

  F(1) = 1, F(0) = 0

  So F(n) is the bottom-left (or top-right) element of T^n!
```

### Verification for small n:

```
  T^1 = ┌     ┐    F(1) = 1 ✓
        │ 1 1 │
        │ 1 0 │    F(0) = 0 ✓
        └     ┘

  T^2 = ┌     ┐    F(2) = 1 ✓
        │ 2 1 │
        │ 1 1 │    F(1) = 1 ✓
        └     ┘

  T^3 = ┌     ┐    F(3) = 2 ✓
        │ 3 2 │
        │ 2 1 │    F(2) = 1 ✓
        └     ┘

  T^6 = ┌       ┐  F(6) = 8 ✓
        │ 13  8 │
        │  8  5 │  F(5) = 5 ✓
        └       ┘
```

---

## 4.4 Matrix Fast Power Algorithm

```
FUNCTION matMul(A, B, MOD):     // Multiply two k×k matrices
    k = rows(A)
    C = new k×k matrix of zeros
    FOR i = 0 TO k-1:
        FOR j = 0 TO k-1:
            FOR p = 0 TO k-1:
                C[i][j] = (C[i][j] + A[i][p] * B[p][j]) % MOD
    RETURN C

FUNCTION matPow(M, n, MOD):    // Matrix M raised to power n
    k = rows(M)
    result = identity(k)        // I_k
    WHILE n > 0:
        IF n & 1:
            result = matMul(result, M, MOD)
        M = matMul(M, M, MOD)
        n >>= 1
    RETURN result

Time: O(k³ × log n)
Space: O(k²)
```

---

## 4.5 General Linear Recurrence → Matrix

```
  Given: f(n) = c₁f(n-1) + c₂f(n-2) + ... + cₖf(n-k)

  Transformation matrix (k × k):

  ┌                          ┐
  │ c₁  c₂  c₃  ...  cₖ₋₁ cₖ│
  │  1   0   0  ...   0    0 │
  │  0   1   0  ...   0    0 │
  │  0   0   1  ...   0    0 │
  │  ⋮   ⋮   ⋮  ⋱    ⋮    ⋮ │
  │  0   0   0  ...   1    0 │
  └                          ┘

  State vector: ┌        ┐
                │ f(n)   │
                │ f(n-1) │
                │  ⋮     │
                │ f(n-k+1)│
                └        ┘

  Example: f(n) = 2f(n-1) + 3f(n-2)

  T = ┌     ┐    State = ┌      ┐
      │ 2 3 │            │ f(n) │
      │ 1 0 │            │f(n-1)│
      └     ┘            └      ┘
```

---

## 4.6 Trace: f(n) = 2f(n-1) + 3f(n-2), f(0)=1, f(1)=2

```
  T = ┌     ┐    initial = ┌     ┐
      │ 2 3 │              │ f(1)│ = ┌   ┐
      │ 1 0 │              │ f(0)│   │ 2 │
      └     ┘              └     ┘   │ 1 │
                                     └   ┘

  Compute f(5) via T^4 × initial:

  T^1 = ┌     ┐
        │ 2 3 │
        │ 1 0 │
        └     ┘

  T^2 = ┌       ┐
        │ 7  6  │     → f(2) = T^1 × [2,1]ᵀ = [7, 2]
        │ 2  3  │        f(2) = 2×2+3×1 = 7 ✓
        └       ┘

  T^4 = T^2 × T^2 = ┌         ┐
                     │ 61  60  │
                     │ 20  21  │
                     └         ┘

  T^4 × [2, 1]ᵀ = [61×2+60×1, 20×2+21×1] = [182, 61]

  f(5) = 182

  Verify iteratively:
  f(0)=1, f(1)=2
  f(2) = 2×2+3×1 = 7
  f(3) = 2×7+3×2 = 20
  f(4) = 2×20+3×7 = 61
  f(5) = 2×61+3×20 = 182 ✓
```

---

## 4.7 Recurrence with Constant Term

```
  f(n) = af(n-1) + bf(n-2) + c  (constant addend)

  Trick: Augment the matrix with the constant!

  ┌      ┐   ┌       ┐   ┌        ┐
  │ f(n)  │   │ a b c │   │ f(n-1) │
  │f(n-1) │ = │ 1 0 0 │ × │ f(n-2) │
  │   1   │   │ 0 0 1 │   │   1    │
  └       ┘   └       ┘   └        ┘

  The "1" row stays constant → carries the additive term.

  Example: f(n) = f(n-1) + f(n-2) + 1

  T = ┌       ┐
      │ 1 1 1 │
      │ 1 0 0 │
      │ 0 0 1 │
      └       ┘
```

---

## 4.8 When to Use Matrix Exponentiation

```
  ┌──────────────────────────────────────────────────────────┐
  │  USE WHEN:                                               │
  │  ✓ Linear recurrence with constant coefficients         │
  │  ✓ n is very large (up to 10¹⁸)                         │
  │  ✓ k (order of recurrence) is small (≤ 10)             │
  │  ✓ Need exact value mod M                               │
  │                                                          │
  │  DON'T USE WHEN:                                         │
  │  ✗ Non-linear recurrence (f(n) = f(n-1)²)              │
  │  ✗ Coefficients depend on n (f(n) = n × f(n-1))        │
  │  ✗ k is large (matrix mult becomes expensive)           │
  │  ✗ n is small enough for simple iteration               │
  └──────────────────────────────────────────────────────────┘
```

---

## 4.9 Classic Problems

| Problem | Matrix | Size |
|---------|--------|------|
| Fibonacci | [[1,1],[1,0]] | 2×2 |
| Tribonacci | [[1,1,1],[1,0,0],[0,1,0]] | 3×3 |
| f(n) = af(n-1)+bf(n-2) | [[a,b],[1,0]] | 2×2 |
| With constant +c | [[a,b,c],[1,0,0],[0,0,1]] | 3×3 |
| Number of paths (k steps in graph) | Adjacency matrix^k | V×V |
| Tiling problems | Transfer matrix | varies |

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Idea | Encode recurrence as matrix multiplication |
| Matrix size | k × k for k-th order recurrence |
| Time | O(k³ log n) |
| Space | O(k²) |
| Key requirement | Linear recurrence with constant coefficients |
| Matrix pow | Same as number pow (squaring method) |
| Identity matrix | Plays role of "1" in number exponentiation |

---

## Quick Revision Questions

1. **Encode the recurrence** f(n) = 3f(n-1) + 2f(n-2) as a matrix.
2. **What is the time complexity** of computing F(10¹⁸) via matrix exponentiation?
3. **Why does the matrix method** fail for f(n) = n × f(n-1)?
4. **How do you handle** a constant term c in the recurrence?
5. **Compute T² for** T = [[1,1],[1,0]] and verify F(2).
6. **How would you count** paths of length k in a graph using matrices?

---

[← Previous: Recursive Fast Power](03-recursive-fast-power.md) | [Back to README](../README.md) | [Next: Fibonacci in O(log n) →](05-fibonacci-in-log-n.md)
