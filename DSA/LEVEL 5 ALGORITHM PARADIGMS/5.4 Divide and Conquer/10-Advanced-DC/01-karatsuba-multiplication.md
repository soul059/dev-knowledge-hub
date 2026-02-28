# Chapter 1: Karatsuba Multiplication

## Overview

**Karatsuba multiplication** (1960) was the first algorithm to multiply two n-digit numbers in sub-quadratic time. It reduces three multiplications of n/2-digit numbers instead of four, achieving O(n^1.585) — a beautiful application of D&C to arithmetic.

---

## The Problem

```
Multiply two n-digit numbers:
    X = 1234
    Y = 5678
    X × Y = ?

Standard long multiplication:
    Each digit of X multiplied by each digit of Y → O(n²) 
    
    Can we do better? YES — Karatsuba!
```

---

## Standard Multiplication Recap

```
          1 2 3 4
        × 5 6 7 8
        ─────────
          9 8 7 2    (1234 × 8)
        8 6 3 8      (1234 × 7)
      7 4 0 4        (1234 × 6)
    6 1 7 0          (1234 × 5)
    ─────────────
    7 0 0 6 6 5 2

Each of n digits multiplied by each of n digits:
n² single-digit multiplications → O(n²)
```

---

## D&C Setup

```
Split each n-digit number into two halves:

    X = X_H · 10^(n/2) + X_L
    Y = Y_H · 10^(n/2) + Y_L

Example (n=4):
    X = 1234 → X_H = 12, X_L = 34
    Y = 5678 → Y_H = 56, Y_L = 78

    X = 12 · 10² + 34
    Y = 56 · 10² + 78

Product:
    X · Y = (X_H · 10^(n/2) + X_L)(Y_H · 10^(n/2) + Y_L)
          = X_H·Y_H · 10^n + (X_H·Y_L + X_L·Y_H) · 10^(n/2) + X_L·Y_L
```

---

## Naive D&C: 4 Multiplications

```
X · Y = X_H·Y_H · 10^n + (X_H·Y_L + X_L·Y_H) · 10^(n/2) + X_L·Y_L

Four multiplications of (n/2)-digit numbers:
    P₁ = X_H · Y_H
    P₂ = X_H · Y_L
    P₃ = X_L · Y_H
    P₄ = X_L · Y_L

    X · Y = P₁ · 10^n + (P₂ + P₃) · 10^(n/2) + P₄

Recurrence: T(n) = 4T(n/2) + O(n)
    log₂ 4 = 2 → T(n) = O(n²)  ← same as standard!
```

---

## Karatsuba's Trick: 3 Multiplications

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  KEY INSIGHT: We can compute X_H·Y_L + X_L·Y_H          │
│  WITHOUT computing them separately!                      │
│                                                          │
│  Define three products:                                  │
│    z₀ = X_L · Y_L                                        │
│    z₂ = X_H · Y_H                                        │
│    z₁ = (X_H + X_L) · (Y_H + Y_L) - z₂ - z₀            │
│                                                          │
│  Then:                                                   │
│    z₁ = X_H·Y_H + X_H·Y_L + X_L·Y_H + X_L·Y_L - z₂ - z₀│
│       = X_H·Y_L + X_L·Y_H                               │
│                                                          │
│  Result: X · Y = z₂ · 10^n + z₁ · 10^(n/2) + z₀        │
│                                                          │
│  Only 3 multiplications! (z₀, z₁, z₂)                   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Worked Example

```
X = 1234, Y = 5678, n = 4

Split:
    X_H = 12, X_L = 34
    Y_H = 56, Y_L = 78

Calculate:
    z₀ = X_L · Y_L = 34 × 78 = 2652
    z₂ = X_H · Y_H = 12 × 56 = 672
    z₁ = (X_H + X_L)(Y_H + Y_L) - z₂ - z₀
       = (12 + 34)(56 + 78) - 672 - 2652
       = 46 × 134 - 672 - 2652
       = 6164 - 672 - 2652
       = 2840

Result:
    X · Y = z₂ · 10⁴ + z₁ · 10² + z₀
          = 672 × 10000 + 2840 × 100 + 2652
          = 6720000 + 284000 + 2652
          = 7006652

Verify: 1234 × 5678 = 7006652 ✓
```

---

## Pseudocode

```
KARATSUBA(X, Y):
    n ← max(digits(X), digits(Y))
    
    if n == 1:
        return X × Y    // single-digit multiplication
    
    // Split
    m ← ⌈n/2⌉
    X_H ← X / 10^m      // high digits
    X_L ← X mod 10^m    // low digits
    Y_H ← Y / 10^m
    Y_L ← Y mod 10^m
    
    // Three recursive multiplications
    z0 ← KARATSUBA(X_L, Y_L)
    z2 ← KARATSUBA(X_H, Y_H)
    z1 ← KARATSUBA(X_H + X_L, Y_H + Y_L) - z2 - z0
    
    return z2 × 10^(2m) + z1 × 10^m + z0
```

---

## Recursion Tree

```
             1234 × 5678
            /     |      \
      34×78   46×134   12×56
      /|\      /|\       /|\
    ...        ...      ...

Standard: 4 branches → 4^(log₂ n) = n² leaves
Karatsuba: 3 branches → 3^(log₂ n) = n^1.585 leaves

Level 0: 1 problem × O(n) work
Level 1: 3 problems × O(n/2) work  
Level 2: 9 problems × O(n/4) work
...
Level k: 3ᵏ problems × O(n/2ᵏ) work
```

---

## Complexity Analysis

```
RECURRENCE:
    T(n) = 3T(n/2) + O(n)

Master Theorem: a=3, b=2, f(n)=O(n)
    log_b a = log₂ 3 ≈ 1.585
    n^(log_b a) = n^1.585
    f(n) = O(n) = O(n^(1.585 - ε)) → Case 1

    T(n) = Θ(n^(log₂ 3)) ≈ Θ(n^1.585)

Comparison:
┌────────────────────┬──────────────────┐
│ Algorithm          │ Complexity       │
├────────────────────┼──────────────────┤
│ Grade school       │ O(n²)            │
│ Karatsuba          │ O(n^1.585)       │
│ Toom-Cook-3        │ O(n^1.465)       │
│ Schönhage-Strassen │ O(n log n loglogn)│
│ Harvey-Hoeven 2019 │ O(n log n)       │
└────────────────────┴──────────────────┘
```

---

## Analogy with Strassen

```
Strassen for matrices:    Karatsuba for integers:
    8 mults → 7 mults         4 mults → 3 mults
    O(n³) → O(n^2.807)       O(n²) → O(n^1.585)
    
    Same key idea:
    Trade multiplications for additions/subtractions!
    
    One fewer multiplication at each level →
    exponential savings over the recursion tree.
```

---

## Python Implementation

```python
def karatsuba(x, y):
    """Multiply two integers using Karatsuba algorithm."""
    # Base case
    if x < 10 or y < 10:
        return x * y
    
    # Determine size
    n = max(len(str(abs(x))), len(str(abs(y))))
    m = n // 2
    
    # Split
    power = 10 ** m
    x_h, x_l = x // power, x % power
    y_h, y_l = y // power, y % power
    
    # Three recursive multiplications
    z0 = karatsuba(x_l, y_l)
    z2 = karatsuba(x_h, y_h)
    z1 = karatsuba(x_h + x_l, y_h + y_l) - z2 - z0
    
    return z2 * (10 ** (2 * m)) + z1 * (10 ** m) + z0

# Test
print(karatsuba(1234, 5678))    # 7006652
print(karatsuba(314159, 271828)) # 85397341652
```

---

## Practical Considerations

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  1. Base case: Switch to standard for small numbers  │
│     → Karatsuba overhead not worth it for n < 20-40  │
│                                                      │
│  2. Used in practice for big integer libraries:      │
│     Python's int, GMP, Java BigInteger               │
│     Typically switch from grade-school → Karatsuba   │
│     → Toom-Cook → Schönhage-Strassen as n grows      │
│                                                      │
│  3. Works in any base, not just base 10              │
│     In computers: use base 2³² or 2⁶⁴               │
│                                                      │
│  4. Sign handling:                                   │
│     Track signs separately, multiply magnitudes      │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Multiply two n-digit numbers |
| Key trick | 4 mults → 3 mults via algebraic identity |
| z₁ formula | (X_H+X_L)(Y_H+Y_L) - X_H·Y_H - X_L·Y_L |
| Recurrence | T(n) = 3T(n/2) + O(n) |
| Complexity | O(n^log₂3) ≈ O(n^1.585) |
| Improvement over | O(n²) grade-school multiplication |
| Used in practice | GMP, Python, Java BigInteger |

---

## Quick Revision Questions

1. **What algebraic identity does Karatsuba exploit?**
2. **How does reducing 4 multiplications to 3 change the recurrence?**
3. **What is the analogy between Karatsuba and Strassen?**
4. **Calculate 56 × 78 using Karatsuba (splitting into single digits).**
5. **When is Karatsuba slower than grade-school multiplication?**

---

[Next: FFT Concept →](02-fft-concept.md)
