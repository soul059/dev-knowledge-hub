# Chapter 1: Strassen's Algorithm

## Overview

**Strassen's Algorithm** (1969) was the first algorithm to multiply two n×n matrices faster than the standard O(n³) method. By cleverly reducing 8 recursive multiplications to **7**, it achieves O(n^2.807) — a breakthrough that proved the cubic barrier is not fundamental.

---

## Standard Matrix Multiplication

### Definition

```
Given two n×n matrices A and B, compute C = A × B where:

    C[i][j] = Σ(k=0 to n-1) A[i][k] × B[k][j]

Example (2×2):

    ┌       ┐   ┌       ┐   ┌                       ┐
    │ a  b  │ × │ e  f  │ = │ ae+bg    af+bh        │
    │ c  d  │   │ g  h  │   │ ce+dg    cf+dh        │
    └       ┘   └       ┘   └                       ┘
```

### Standard Algorithm: O(n³)

```
MATRIX_MULTIPLY(A, B, n):
    C ← new n×n matrix of zeros
    for i ← 0 to n-1:
        for j ← 0 to n-1:
            for k ← 0 to n-1:
                C[i][j] += A[i][k] * B[k][j]
    return C

    Three nested loops → O(n³) multiplications
```

---

## Naive D&C Approach

### Block Decomposition

```
Divide each n×n matrix into four (n/2)×(n/2) blocks:

    ┌──────┬──────┐   ┌──────┬──────┐   ┌──────┬──────┐
    │  A₁₁ │  A₁₂ │   │  B₁₁ │  B₁₂ │   │  C₁₁ │  C₁₂ │
    │      │      │ × │      │      │ = │      │      │
    ├──────┼──────┤   ├──────┼──────┤   ├──────┼──────┤
    │  A₂₁ │  A₂₂ │   │  B₂₁ │  B₂₂ │   │  C₂₁ │  C₂₂ │
    │      │      │   │      │      │   │      │      │
    └──────┴──────┘   └──────┴──────┘   └──────┴──────┘

Block formulas:
    C₁₁ = A₁₁·B₁₁ + A₁₂·B₂₁
    C₁₂ = A₁₁·B₁₂ + A₁₂·B₂₂
    C₂₁ = A₂₁·B₁₁ + A₂₂·B₂₁
    C₂₂ = A₂₁·B₁₂ + A₂₂·B₂₂

    → 8 multiplications of (n/2)×(n/2) matrices
    → 4 additions of (n/2)×(n/2) matrices
```

### Naive D&C Recurrence

```
T(n) = 8T(n/2) + O(n²)

Master Theorem: a=8, b=2, f(n)=n²
  log_b a = log₂ 8 = 3
  n^(log_b a) = n³
  f(n) = n² = O(n^(3-1)) → Case 1

  T(n) = Θ(n³)

No improvement! Same as brute force.
Need to reduce the number of multiplications!
```

---

## Strassen's Key Insight

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  Standard: 8 multiplications, 4 additions           │
│  Strassen: 7 multiplications, 18 additions/subs     │
│                                                     │
│  Trade 1 multiplication for extra additions.        │
│  Since multiplication is the expensive recursive    │
│  operation, this SAVES time overall!                │
│                                                     │
│  8 → 7 multiplications:                             │
│  T(n) = 8T(n/2) + O(n²) → T(n) = 7T(n/2) + O(n²) │
│       = O(n³)            → O(n^2.807)               │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## The Seven Products

```
Strassen defines 7 products (M₁ through M₇):

    M₁ = (A₁₁ + A₂₂) · (B₁₁ + B₂₂)
    M₂ = (A₂₁ + A₂₂) · B₁₁
    M₃ = A₁₁ · (B₁₂ - B₂₂)
    M₄ = A₂₂ · (B₂₁ - B₁₁)
    M₅ = (A₁₁ + A₁₂) · B₂₂
    M₆ = (A₂₁ - A₁₁) · (B₁₁ + B₁₂)
    M₇ = (A₁₂ - A₂₂) · (B₂₁ + B₂₂)

Then compute C blocks:

    C₁₁ = M₁ + M₄ - M₅ + M₇
    C₁₂ = M₃ + M₅
    C₂₁ = M₂ + M₄
    C₂₂ = M₁ - M₂ + M₃ + M₆

Operations count:
    Multiplications: 7 (recursive calls)
    Additions/Subtractions: 18
```

---

## Verification: C₁₁ = A₁₁·B₁₁ + A₁₂·B₂₁

```
C₁₁ = M₁ + M₄ - M₅ + M₇

M₁ = (A₁₁+A₂₂)(B₁₁+B₂₂)
   = A₁₁B₁₁ + A₁₁B₂₂ + A₂₂B₁₁ + A₂₂B₂₂

M₄ = A₂₂(B₂₁-B₁₁)
   = A₂₂B₂₁ - A₂₂B₁₁

M₅ = (A₁₁+A₁₂)B₂₂
   = A₁₁B₂₂ + A₁₂B₂₂

M₇ = (A₁₂-A₂₂)(B₂₁+B₂₂)
   = A₁₂B₂₁ + A₁₂B₂₂ - A₂₂B₂₁ - A₂₂B₂₂

M₁ + M₄ - M₅ + M₇:
  = (A₁₁B₁₁ + A₁₁B₂₂ + A₂₂B₁₁ + A₂₂B₂₂)
  + (A₂₂B₂₁ - A₂₂B₁₁)
  - (A₁₁B₂₂ + A₁₂B₂₂)
  + (A₁₂B₂₁ + A₁₂B₂₂ - A₂₂B₂₁ - A₂₂B₂₂)

Cancellations:
  A₁₁B₂₂ cancels with -A₁₁B₂₂     ✓
  A₂₂B₁₁ cancels with -A₂₂B₁₁     ✓
  A₂₂B₂₂ cancels with -A₂₂B₂₂     ✓
  A₂₂B₂₁ cancels with -A₂₂B₂₁     ✓
  A₁₂B₂₂ cancels with -A₁₂B₂₂     ✓

Remaining: A₁₁B₁₁ + A₁₂B₂₁  ✓ CORRECT!
```

---

## Worked Example (2×2)

```
    A = ┌ 1  3 ┐    B = ┌ 5  7 ┐
        │ 2  4 │        │ 6  8 │
        └      ┘        └      ┘

Standard: C₁₁ = 1·5 + 3·6 = 5+18 = 23
          C₁₂ = 1·7 + 3·8 = 7+24 = 31
          C₂₁ = 2·5 + 4·6 = 10+24 = 34
          C₂₂ = 2·7 + 4·8 = 14+32 = 46

Strassen:
  M₁ = (1+4)·(5+8) = 5·13 = 65
  M₂ = (2+4)·5     = 6·5  = 30
  M₃ = 1·(7-8)     = 1·(-1) = -1
  M₄ = 4·(6-5)     = 4·1  = 4
  M₅ = (1+3)·8     = 4·8  = 32
  M₆ = (2-1)·(5+7) = 1·12 = 12
  M₇ = (3-4)·(6+8) = (-1)·14 = -14

  C₁₁ = 65 + 4 - 32 + (-14) = 23  ✓
  C₁₂ = -1 + 32              = 31  ✓
  C₂₁ = 30 + 4               = 34  ✓
  C₂₂ = 65 - 30 + (-1) + 12  = 46  ✓
```

---

## Pseudocode

```
STRASSEN(A, B, n):
    if n == 1:
        return A[0][0] * B[0][0]
    
    // Split into quadrants
    A₁₁, A₁₂, A₂₁, A₂₂ ← split A into four (n/2)×(n/2)
    B₁₁, B₁₂, B₂₁, B₂₂ ← split B into four (n/2)×(n/2)
    
    // Compute the 7 products
    M₁ ← STRASSEN(A₁₁+A₂₂, B₁₁+B₂₂, n/2)
    M₂ ← STRASSEN(A₂₁+A₂₂, B₁₁, n/2)
    M₃ ← STRASSEN(A₁₁, B₁₂-B₂₂, n/2)
    M₄ ← STRASSEN(A₂₂, B₂₁-B₁₁, n/2)
    M₅ ← STRASSEN(A₁₁+A₁₂, B₂₂, n/2)
    M₆ ← STRASSEN(A₂₁-A₁₁, B₁₁+B₁₂, n/2)
    M₇ ← STRASSEN(A₁₂-A₂₂, B₂₁+B₂₂, n/2)
    
    // Combine results
    C₁₁ ← M₁ + M₄ - M₅ + M₇
    C₁₂ ← M₃ + M₅
    C₂₁ ← M₂ + M₄
    C₂₂ ← M₁ - M₂ + M₃ + M₆
    
    return combine(C₁₁, C₁₂, C₂₁, C₂₂)
```

---

## Cost Diagram

```
         Standard D&C              Strassen
         ━━━━━━━━━━━              ━━━━━━━━
    8 × T(n/2) + Θ(n²)      7 × T(n/2) + Θ(n²)
         │                        │
    8 recursive calls         7 recursive calls
    4 matrix additions       18 matrix add/subs
         │                        │
    T(n) = Θ(n³)             T(n) = Θ(n^lg 7)
                                   ≈ Θ(n^2.807)
```

---

## Summary Table

| Aspect | Standard | Naive D&C | Strassen |
|--------|----------|-----------|----------|
| Multiplications | n³ | 8T(n/2) | 7T(n/2) |
| Additions | — | 4·(n/2)² | 18·(n/2)² |
| Time | O(n³) | O(n³) | O(n^2.807) |
| Space | O(n²) | O(n²) | O(n² log n) |
| Year | Ancient | — | 1969 |

---

## Quick Revision Questions

1. **How many multiplications does standard matrix multiply use?**
2. **How does Strassen reduce 8 multiplications to 7?**
3. **What is the recurrence for Strassen's algorithm?**
4. **Verify that C₂₁ = A₂₁·B₁₁ + A₂₂·B₂₁ using M₂ + M₄.**
5. **Why do extra additions not affect the asymptotic complexity?**

---

[Next: D&C Approach in Detail →](02-dc-approach.md)
