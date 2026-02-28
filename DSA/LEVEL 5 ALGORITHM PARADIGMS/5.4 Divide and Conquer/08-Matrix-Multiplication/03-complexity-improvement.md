# Chapter 3: Complexity Improvement — Beyond Strassen

## Overview

Strassen's O(n^2.807) was the first sub-cubic matrix multiplication. This chapter examines the mathematical framework, subsequent improvements, and the fundamental question: **What is the fastest possible matrix multiplication?**

---

## The Exponent of Matrix Multiplication: ω

```
DEFINITION: ω (omega) is the infimum of all real numbers c 
such that n×n matrices can be multiplied in O(n^c) time.

Known bounds:
    2 ≤ ω < 2.3719        (as of 2024)

    ┌───────────────────────────────────────────────┐
    │             ω = ?                              │
    │                                                │
    │  ├──────────┼─────┼───┼─┼──────────────────┤  │
    │  2       2.3719  2.38 2.5  2.807       3.0  │  │
    │  │         │                  │          │   │  │
    │  lower    current           Strassen   naïve │  │
    │  bound    best                               │  │
    └───────────────────────────────────────────────┘
```

---

## Timeline of Matrix Multiplication Algorithms

```
┌──────┬───────────────────────────────┬────────────────┐
│ Year │ Algorithm                     │ Exponent ω     │
├──────┼───────────────────────────────┼────────────────┤
│ -    │ Standard (naïve)              │ 3.000          │
│ 1969 │ Strassen                      │ 2.807          │
│ 1979 │ Pan                           │ 2.796          │
│ 1980 │ Bini et al.                   │ 2.780          │
│ 1981 │ Schönhage                     │ 2.548          │
│ 1982 │ Romani                        │ 2.517          │
│ 1986 │ Strassen (laser method)       │ 2.479          │
│ 1990 │ Coppersmith-Winograd          │ 2.376          │
│ 2012 │ Williams                      │ 2.3729         │
│ 2014 │ Le Gall                       │ 2.3728642      │
│ 2024 │ Duan, Wu, Zhou                │ 2.371866       │
│  ?   │ Theoretical limit?            │ 2.000?         │
└──────┴───────────────────────────────┴────────────────┘
```

---

## Why ω ≥ 2?

```
Lower bound proof:

  Input: 2n² entries (matrices A and B)
  Output: n² entries (matrix C)

  Any algorithm must at least READ all input: Ω(n²)
  Any algorithm must WRITE the output: Ω(n²)

  Therefore: ω ≥ 2

  ┌─────────────────────────────────────────────┐
  │ The big open question:                      │
  │ Is ω = 2?                                   │
  │                                             │
  │ Many researchers conjecture ω = 2,          │
  │ but no algorithm achieving O(n²) is known.  │
  │                                             │
  │ Current gap: n^2.372 vs n^2.000            │
  │ This is still a significant difference.     │
  └─────────────────────────────────────────────┘
```

---

## General Framework: Reducing Multiplications

```
For 2×2 matrices:
  Standard: 8 multiplications → O(n³)
  Strassen: 7 multiplications → O(n^2.807)

General pattern: If we can multiply 2×2 block matrices
using k multiplications of (n/2)×(n/2) blocks:

  T(n) = k·T(n/2) + O(n²)
  T(n) = O(n^(log₂ k))

┌─────┬────────────────────────┐
│  k  │  Exponent log₂ k      │
├─────┼────────────────────────┤
│  8  │  3.000 (standard)      │
│  7  │  2.807 (Strassen)      │
│  6  │  2.585                 │
│  5  │  2.322                 │
│  4  │  2.000 (dream!)        │
└─────┴────────────────────────┘

With 4 multiplications, we'd get ω=2.
But it's proven that 2×2 requires at least 7!
(Hopcroft & Kerr, 1971)
```

---

## Larger Block Sizes

```
Instead of 2×2 blocks, use m×m blocks:

Standard: m³ multiplications of (n/m)×(n/m) blocks
  T(n) = m³ T(n/m) + O(n²)
  T(n) = O(n^(log_m m³)) = O(n³)  ← no improvement

Strassen-like: reduce m³ to some k < m³
  T(n) = k T(n/m) + O(n²)
  T(n) = O(n^(log_m k))

Example (3×3 blocks):
  Standard: 27 multiplications
  Best known: 21 multiplications (Makarov-Smirnov)
  T(n) = 21 T(n/3) + O(n²)
  T(n) = O(n^(log₃ 21)) ≈ O(n^2.771)  ← better than Strassen!
```

---

## The Coppersmith-Winograd Approach

```
The CW method uses a different strategy:

1. Construct a special "trilinear form" (tensor)
2. Find the "rank" of this tensor
3. Use "laser method" to amplify small improvements

Key idea: Instead of block decomposition, work with
algebraic tensors representing multiplication.

This approach led to all results below ω ≈ 2.5,
but the constants are astronomically large.
```

---

## Practical vs Theoretical

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│  THEORETICAL ALGORITHMS (ω ≈ 2.37):                   │
│    • Use galactic constants                            │
│    • Only faster for n > 10²⁰ (estimated!)             │
│    • Never implemented in practice                     │
│    • Academic interest only                            │
│                                                        │
│  PRACTICAL ALGORITHMS:                                 │
│    • Standard O(n³): best for n < 100                  │
│    • Strassen O(n^2.81): useful for n > 100-500        │
│    • BLAS libraries: highly optimized O(n³)             │
│                                                        │
│  WHY? Because constants matter hugely:                 │
│                                                        │
│    Standard:  1 · n³                                   │
│    Strassen:  ~25 · n^2.807   (extra additions!)       │
│    CW:        10²⁰ · n^2.376  (galactic constant)     │
│                                                        │
│  At n = 1000:                                          │
│    Standard: 10⁹                                       │
│    Strassen: ~25 · 10⁸·⁴ ≈ 6.3 × 10⁹  (slower!)      │
│    Strassen breaks even around n ≈ 500-1000             │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Comparison at Various Sizes

```
Operations count (approximate):

┌──────┬──────────────┬──────────────┬───────────────┐
│  n   │ Standard n³  │ Strassen     │ Strassen wins?│
│      │              │ ~25·n^2.807  │               │
├──────┼──────────────┼──────────────┼───────────────┤
│   10 │ 1,000        │ 16,000       │ NO            │
│   50 │ 125,000      │ 261,000      │ NO            │
│  100 │ 1,000,000    │ 1,600,000    │ NO            │
│  500 │ 125,000,000  │ 96,000,000   │ YES           │
│ 1000 │ 1 × 10⁹     │ 6.1 × 10⁸   │ YES           │
│ 4096 │ 6.9 × 10¹⁰  │ 1.4 × 10¹⁰  │ YES (5×)      │
└──────┴──────────────┴──────────────┴───────────────┘

Crossover point depends on implementation and hardware.
Typically n ≈ 200-1000.
```

---

## Winograd's Variant

```
Winograd's variant of Strassen uses the same 7 multiplications
but reduces additions from 18 to 15:

Slightly faster in practice (fewer operations per call).
Same asymptotic complexity: O(n^2.807).

This is the variant most commonly implemented.

Additions comparison:
  Standard D&C: 4 additions
  Strassen:     18 additions/subtractions
  Winograd:     15 additions/subtractions
```

---

## Open Questions

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│  1. Is ω = 2?                                          │
│     Most researchers believe: probably yes.             │
│     But proving it is VERY hard.                        │
│                                                        │
│  2. Can we find a PRACTICAL sub-n^2.5 algorithm?       │
│     Currently: Strassen (~n^2.81) is the only          │
│     practical sub-cubic algorithm.                     │
│                                                        │
│  3. Are group-theoretic approaches the way forward?    │
│     Cohn-Umans framework suggests connections to       │
│     group theory and combinatorics.                    │
│                                                        │
│  4. What is the relationship between matrix            │
│     multiplication and other problems?                  │
│     Many problems reduce to matrix mult:               │
│     - Graph algorithms (transitive closure)            │
│     - Linear algebra (inversion, determinant)          │
│     - Parsing (CYK algorithm)                          │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Implications of ω

```
Many problems have complexity tied to ω:

┌────────────────────────────┬──────────────────┐
│ Problem                    │ Best Known Time   │
├────────────────────────────┼──────────────────┤
│ Matrix Multiplication      │ O(n^ω)           │
│ Matrix Inversion           │ O(n^ω)           │
│ Determinant                │ O(n^ω)           │
│ LU Decomposition           │ O(n^ω)           │
│ All-Pairs Shortest Paths   │ O(n^ω · polylog) │
│ Transitive Closure         │ O(n^ω)           │
│ Parsing (CFG)              │ O(n^ω)           │
└────────────────────────────┴──────────────────┘

If ω = 2, ALL these problems would be nearly quadratic!
```

---

## Summary Table

| Topic | Key Point |
|-------|-----------|
| ω definition | Infimum exponent for matrix multiplication |
| Current best | ω < 2.3719 (Duan, Wu, Zhou 2024) |
| Lower bound | ω ≥ 2 (must read all input) |
| Strassen | First sub-cubic: ω = log₂ 7 ≈ 2.807 |
| Practicality | Only Strassen is practical for large n |
| Crossover | Strassen faster roughly when n > 200-500 |
| Open question | Is ω = 2? (widely conjectured yes) |

---

## Quick Revision Questions

1. **What is ω and what are its known bounds?**
2. **Why must ω ≥ 2?**
3. **Why aren't CW-type algorithms used in practice?**
4. **At what size does Strassen typically become faster?**
5. **Name three problems whose complexity depends on ω.**
6. **How many multiplications are needed for 2×2 (lower bound)?**

---

[← Previous: D&C Approach](02-dc-approach.md) | [Next: Practical Considerations →](04-practical-considerations.md)
