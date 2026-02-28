# Chapter 2: Divide and Conquer Approach to Matrix Multiplication

## Overview

This chapter details how the D&C framework applies to matrix multiplication, showing the block decomposition, the recursion structure, and why reducing from 8 to 7 recursive calls is so impactful.

---

## Block Matrix Multiplication

```
Any n×n matrix can be partitioned into four (n/2)×(n/2) blocks:

    A = ┌──────┬──────┐        B = ┌──────┬──────┐
        │ A₁₁  │ A₁₂  │            │ B₁₁  │ B₁₂  │
        │(n/2) │(n/2) │            │(n/2) │(n/2) │
        ├──────┼──────┤            ├──────┼──────┤
        │ A₂₁  │ A₂₂  │            │ B₂₁  │ B₂₂  │
        │(n/2) │(n/2) │            │(n/2) │(n/2) │
        └──────┴──────┘            └──────┴──────┘

Block multiplication follows the same rules as scalar:

    C₁₁ = A₁₁·B₁₁ + A₁₂·B₂₁     (2 mults, 1 add)
    C₁₂ = A₁₁·B₁₂ + A₁₂·B₂₂     (2 mults, 1 add)
    C₂₁ = A₂₁·B₁₁ + A₂₂·B₂₁     (2 mults, 1 add)
    C₂₂ = A₂₁·B₁₂ + A₂₂·B₂₂     (2 mults, 1 add)
    ──────────────────────────────
    Total: 8 multiplications, 4 additions of (n/2)×(n/2) matrices
```

---

## The Three Steps of D&C

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│  DIVIDE:                                               │
│    Split A, B into four (n/2)×(n/2) submatrices each  │
│    Cost: O(1) if using index manipulation              │
│          O(n²) if copying submatrices                  │
│                                                        │
│  CONQUER:                                              │
│    Standard: 8 recursive multiplications               │
│    Strassen: 7 recursive multiplications               │
│                                                        │
│  COMBINE:                                              │
│    Standard: 4 matrix additions → O(n²)                │
│    Strassen: 18 additions/subtractions → O(n²)         │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Recursion Tree: Standard (8 calls)

```
Level 0: 1 problem of size n²        Cost: n²
         ├─├─├─├─├─├─├─┤  (8 children)
Level 1: 8 problems of size (n/2)²    Cost: 8·(n/2)² = 2n²
         ├─ ... 64 children
Level 2: 64 problems of size (n/4)²   Cost: 64·(n/4)² = 4n²
         ...
Level k: 8ᵏ problems of size (n/2ᵏ)²  Cost: 8ᵏ·(n/2ᵏ)² = (8/4)ᵏ·n² = 2ᵏ·n²

Levels: log₂ n

Total = n² Σ(k=0 to log n) 2ᵏ = n²·(2^(log n + 1) - 1) = n²·(2n - 1) = O(n³)

Root-heavy? NO. Leaf-heavy!
  Leaf level cost: 8^(log₂ n) = n^(log₂ 8) = n³
  Root level cost: n²
  Growth factor: 8/4 = 2 > 1 → cost GROWS per level
```

---

## Recursion Tree: Strassen (7 calls)

```
Level 0: 1 problem of size n²           Cost: n²
         ├─├─├─├─├─├─┤  (7 children)
Level 1: 7 problems of size (n/2)²       Cost: 7·(n/2)² = 7n²/4
Level 2: 49 problems of size (n/4)²      Cost: 49·(n/4)² = 49n²/16
         ...
Level k: 7ᵏ problems of size (n/2ᵏ)²    Cost: 7ᵏ·n²/4ᵏ = (7/4)ᵏ·n²

Levels: log₂ n

Growth factor: 7/4 = 1.75 > 1 → still leaf-heavy

Total = n² · Σ(k=0 to log n) (7/4)ᵏ
      = n² · ((7/4)^(log₂ n + 1) - 1) / (7/4 - 1)
      = Θ((7/4)^(log₂ n) · n²)
      = Θ(n^(log₂ 7))            ← since (7/4)^(log₂ n) = n^(log₂ 7 - 2)
      ≈ Θ(n^2.807)
```

---

## Visual: 8 vs 7 Multiplications

```
    Standard D&C                        Strassen

    C₁₁  C₁₂  C₂₁  C₂₂                M₁ M₂ M₃ M₄ M₅ M₆ M₇
     │    │    │    │                    │  │  │  │  │  │  │
     ├──┤ ├──┤ ├──┤ ├──┤               Each Mᵢ = one mult
     │  │ │  │ │  │ │  │               of (n/2)×(n/2) matrices
    [8 multiplications]               [7 multiplications]

    Cost: 8·T(n/2) + 4·O((n/2)²)     Cost: 7·T(n/2) + 18·O((n/2)²)
         = 8·T(n/2) + O(n²)                = 7·T(n/2) + O(n²)
         ────────────────────                ────────────────────
         = O(n³)                             = O(n^2.807)

    SAVINGS: ~6.5% fewer multiplications per level
             → compounds to dramatic savings over log n levels!
```

---

## Why Saving 1 Multiplication Matters So Much

```
The number of leaf-level operations determines asymptotic cost:

Standard: 8^(log₂ n) = n^(log₂ 8) = n^3.000
Strassen: 7^(log₂ n) = n^(log₂ 7) = n^2.807

                n        n³            n^2.807       Speedup
              ─────    ──────────    ──────────      ───────
                64      262,144       80,367          3.3×
               256      16.7 M       2.7 M           6.2×
              1024      1.07 B       90 M            11.9×
              4096      68.7 B       3.9 B           17.6×

The difference GROWS with n!

Key insight: each saved multiplication at level k
saves 7ᵏ⁻¹ multiplications in the subtree.
Over log n levels, this compounds exponentially.
```

---

## Implementation Details

### Handling Non-Power-of-2 Sizes

```
If n is not a power of 2, pad with zeros:

    ┌──────┬───┐         ┌──────┬───┐
    │      │   │         │      │ 0 │
    │  A   │ 0 │    →    │  A   │ 0 │  (next power of 2)
    │      │   │         │      │ 0 │
    ├──────┼───┤         ├──────┼───┤
    │  0   │ 0 │         │  0   │ 0 │
    └──────┴───┘         └──────┴───┘

At most doubles n → constant factor increase.
```

### Base Case Selection

```
For small n, standard O(n³) is faster due to overhead:

STRASSEN(A, B, n):
    if n ≤ CROSSOVER:        // typically 32-128
        return STANDARD(A, B, n)
    // ... Strassen recursion

This hybrid approach gives best practical performance.
```

---

## Memory Layout Considerations

```
Standard:
  ┌─────────────────────┐
  │ C = A × B           │
  │ In-place possible   │
  │ Good cache behavior │
  └─────────────────────┘

Strassen:
  ┌──────────────────────────────────┐
  │ Needs temporary matrices for:   │
  │   - 10 additions (A/B combos)   │
  │   - 7 products M₁..M₇          │
  │   - 4 result combinations       │
  │                                  │
  │ Extra space: O(n²) per level     │
  │ Total extra: O(n² log n)         │
  │   (can be reduced to O(n²))      │
  └──────────────────────────────────┘
```

---

## Python Implementation

```python
import numpy as np

def strassen(A, B):
    n = len(A)
    
    # Base case
    if n <= 64:
        return A @ B  # NumPy standard multiplication
    
    # Pad if not power of 2
    if n % 2 != 0:
        A = np.pad(A, ((0,1),(0,1)))
        B = np.pad(B, ((0,1),(0,1)))
        C = strassen(A, B)
        return C[:n, :n]
    
    mid = n // 2
    
    # Split into quadrants
    A11, A12 = A[:mid, :mid], A[:mid, mid:]
    A21, A22 = A[mid:, :mid], A[mid:, mid:]
    B11, B12 = B[:mid, :mid], B[:mid, mid:]
    B21, B22 = B[mid:, :mid], B[mid:, mid:]
    
    # 7 products
    M1 = strassen(A11 + A22, B11 + B22)
    M2 = strassen(A21 + A22, B11)
    M3 = strassen(A11, B12 - B22)
    M4 = strassen(A22, B21 - B11)
    M5 = strassen(A11 + A12, B22)
    M6 = strassen(A21 - A11, B11 + B12)
    M7 = strassen(A12 - A22, B21 + B22)
    
    # Combine
    C11 = M1 + M4 - M5 + M7
    C12 = M3 + M5
    C21 = M2 + M4
    C22 = M1 - M2 + M3 + M6
    
    # Assemble result
    C = np.zeros((n, n))
    C[:mid, :mid] = C11
    C[:mid, mid:] = C12
    C[mid:, :mid] = C21
    C[mid:, mid:] = C22
    
    return C
```

---

## D&C Structure Summary

```
┌──────────────────────────────────────────────────────┐
│          D&C for Matrix Multiplication               │
├──────────────┬───────────────────────────────────────┤
│ Problem      │ Multiply two n×n matrices             │
│ Divide       │ Split into 4 blocks of (n/2)×(n/2)   │
│ Conquer      │ 7 (not 8) recursive multiplications   │
│ Combine      │ 18 additions/subtractions → O(n²)     │
│ Base case    │ n=1 (scalar) or n≤64 (standard mult)  │
│ Recurrence   │ T(n) = 7T(n/2) + Θ(n²)               │
│ Complexity   │ Θ(n^log₂7) ≈ Θ(n^2.807)              │
└──────────────┴───────────────────────────────────────┘
```

---

## Summary Table

| D&C Step | Standard | Strassen |
|----------|----------|----------|
| Subproblems (a) | 8 | 7 |
| Size reduction (b) | 2 | 2 |
| Combine cost | O(n²) | O(n²) |
| log_b a | 3 | 2.807 |
| Total time | O(n³) | O(n^2.807) |

---

## Quick Revision Questions

1. **What are the four block equations for standard matrix D&C?**
2. **Why doesn't reducing additions help asymptotically?**
3. **Draw the first two levels of the recursion tree for Strassen.**
4. **How do you handle n that isn't a power of 2?**
5. **What base case size is typically used in practice?**
6. **Why is the recursion tree leaf-heavy for both approaches?**

---

[← Previous: Strassen's Algorithm](01-strassens-algorithm.md) | [Next: Complexity Improvement →](03-complexity-improvement.md)
