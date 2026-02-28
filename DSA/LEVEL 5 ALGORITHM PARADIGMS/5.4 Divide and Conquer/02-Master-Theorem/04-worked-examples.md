# Chapter 4: Worked Examples

## Overview

This chapter is a comprehensive problem set with detailed solutions. Each example walks through the Master Theorem application step-by-step, building your confidence in recognizing and solving recurrences.

---

## Example 1: Standard Merge Sort

**T(n) = 2T(n/2) + cn** (where c is a constant)

```
┌──────────────────────────────────────────┐
│  a = 2,  b = 2,  f(n) = cn = Θ(n)      │
│  c_crit = log₂(2) = 1                   │
│  f(n) = n¹  →  c = 1 = c_crit           │
│  CASE 2  →  T(n) = Θ(n log n)           │
└──────────────────────────────────────────┘

Verification via recursion tree:
Level 0:  cn                                     = cn
Level 1:  2 · c(n/2)     = cn                    = cn
Level 2:  4 · c(n/4)     = cn                    = cn
Level 3:  8 · c(n/8)     = cn                    = cn
...
Level k:  2^k · c(n/2^k) = cn                    = cn

Number of levels = log₂(n)
Total = cn · log₂(n) = Θ(n log n) ✓
```

---

## Example 2: Karatsuba Multiplication

**T(n) = 3T(n/2) + cn**

```
┌──────────────────────────────────────────┐
│  a = 3,  b = 2,  f(n) = cn = Θ(n)      │
│  c_crit = log₂(3) ≈ 1.585               │
│  f(n) = n¹  →  c = 1 < 1.585 = c_crit   │
│  CASE 1  →  T(n) = Θ(n^1.585)           │
└──────────────────────────────────────────┘

Why? The leaves do more work:
Level 0:  cn
Level 1:  3·c(n/2) = 1.5cn              ← growing!
Level 2:  9·c(n/4) = 2.25cn             ← growing!
Level k:  3^k · c(n/2^k) = c·n·(3/2)^k  ← geometric growth

Leaf level dominates → Θ(n^(log₂ 3)) ≈ Θ(n^1.585)
```

---

## Example 3: Naive Matrix Multiply (D&C)

**T(n) = 8T(n/2) + Θ(n²)**

```
┌──────────────────────────────────────────┐
│  a = 8,  b = 2,  f(n) = n²              │
│  c_crit = log₂(8) = 3                   │
│  f(n) = n²  →  c = 2 < 3 = c_crit       │
│  CASE 1  →  T(n) = Θ(n³)                │
└──────────────────────────────────────────┘

This matches naive O(n³) — D&C doesn't help here!
Strassen reduces 8 → 7 subproblems to beat this.
```

---

## Example 4: Strassen's Matrix Multiply

**T(n) = 7T(n/2) + Θ(n²)**

```
┌──────────────────────────────────────────┐
│  a = 7,  b = 2,  f(n) = n²              │
│  c_crit = log₂(7) ≈ 2.807               │
│  f(n) = n²  →  c = 2 < 2.807 = c_crit   │
│  CASE 1  →  T(n) = Θ(n^2.807)           │
└──────────────────────────────────────────┘

Reducing from 8 to 7 recursive calls:
  Old: T(n) = Θ(n³)     →  n = 1000: ~10⁹ operations
  New: T(n) = Θ(n^2.807) →  n = 1000: ~4×10⁸ operations
  Speedup: ~2.5x for n = 1000!
```

---

## Example 5: Heavy Combine Step

**T(n) = T(n/2) + n**

```
┌──────────────────────────────────────────┐
│  a = 1,  b = 2,  f(n) = n               │
│  c_crit = log₂(1) = 0                   │
│  f(n) = n¹  →  c = 1 > 0 = c_crit       │
│  CASE 3                                  │
│  Regularity: 1·f(n/2) = n/2 ≤ ½·n ✓     │
│  T(n) = Θ(n)                             │
└──────────────────────────────────────────┘

Unrolling:
T(n) = n + n/2 + n/4 + n/8 + ... + 1
     = n(1 + 1/2 + 1/4 + ...) 
     = n · 2 = Θ(n) ✓
```

---

## Example 6: Many Subproblems, Light Work

**T(n) = 16T(n/4) + n**

```
┌──────────────────────────────────────────┐
│  a = 16,  b = 4,  f(n) = n              │
│  c_crit = log₄(16) = 2                  │
│  f(n) = n¹  →  c = 1 < 2 = c_crit       │
│  CASE 1  →  T(n) = Θ(n²)                │
└──────────────────────────────────────────┘
```

---

## Example 7: Balanced with Log Factor

**T(n) = 2T(n/2) + n log n**

```
┌──────────────────────────────────────────────┐
│  a = 2,  b = 2,  f(n) = n log n             │
│  c_crit = log₂(2) = 1                       │
│  f(n) = n log n = Θ(n¹ · log¹ n)            │
│  c = 1 = c_crit, with k = 1                 │
│  CASE 2 (extended)                           │
│  T(n) = Θ(n · log² n)                       │
└──────────────────────────────────────────────┘

Note: The log factor increases k by 1:
  f(n) = Θ(n^c_crit · log^k n) → T(n) = Θ(n^c_crit · log^(k+1) n)
  Here: k = 1 → T(n) = Θ(n · log² n)
```

---

## Example 8: Constant Work per Level

**T(n) = 4T(n/2) + n²**

```
┌──────────────────────────────────────────┐
│  a = 4,  b = 2,  f(n) = n²              │
│  c_crit = log₂(4) = 2                   │
│  f(n) = n²  →  c = 2 = c_crit           │
│  CASE 2  →  T(n) = Θ(n² log n)          │
└──────────────────────────────────────────┘

Recursion tree verification:
Level 0:  n²
Level 1:  4·(n/2)² = 4·n²/4 = n²          ← same!
Level 2:  16·(n/4)² = 16·n²/16 = n²       ← same!
Each of log₂(n) levels does n² work → Θ(n² log n) ✓
```

---

## Example 9: Three-Way Recursion

**T(n) = 27T(n/3) + n³**

```
┌──────────────────────────────────────────┐
│  a = 27,  b = 3,  f(n) = n³             │
│  c_crit = log₃(27) = 3                  │
│  f(n) = n³  →  c = 3 = c_crit           │
│  CASE 2  →  T(n) = Θ(n³ log n)          │
└──────────────────────────────────────────┘
```

---

## Example 10: Square Root f(n)

**T(n) = 2T(n/4) + √n**

```
┌──────────────────────────────────────────┐
│  a = 2,  b = 4,  f(n) = √n = n^(1/2)   │
│  c_crit = log₄(2) = 1/2 = 0.5           │
│  f(n) = n^(0.5)  →  c = 0.5 = c_crit    │
│  CASE 2  →  T(n) = Θ(√n · log n)        │
└──────────────────────────────────────────┘
```

---

## Example 11: Unequal but Valid

**T(n) = 3T(n/9) + √n**

```
┌──────────────────────────────────────────┐
│  a = 3,  b = 9,  f(n) = √n = n^(1/2)   │
│  c_crit = log₉(3) = 1/2 = 0.5           │
│  f(n) = n^(0.5)  →  c = 0.5 = c_crit    │
│  CASE 2  →  T(n) = Θ(√n · log n)        │
└──────────────────────────────────────────┘
```

---

## Example 12: Root Dominates

**T(n) = 3T(n/4) + n log n**

```
┌──────────────────────────────────────────────┐
│  a = 3,  b = 4,  f(n) = n log n             │
│  c_crit = log₄(3) ≈ 0.793                   │
│  f(n) = n log n = Ω(n^1)                    │
│  c = 1 > 0.793 = c_crit                     │
│  CASE 3                                      │
│  Regularity: 3·f(n/4) = 3·(n/4)·log(n/4)   │
│              = (3/4)·n·(log n - 2)          │
│              ≤ (3/4)·n·log n for large n ✓   │
│  T(n) = Θ(n log n)                          │
└──────────────────────────────────────────────┘
```

---

## Comprehensive Reference Table

| # | Recurrence | a | b | c_crit | c | Case | T(n) |
|---|-----------|---|---|--------|---|------|------|
| 1 | T(n) = 2T(n/2) + n | 2 | 2 | 1 | 1 | 2 | Θ(n log n) |
| 2 | T(n) = 3T(n/2) + n | 3 | 2 | 1.585 | 1 | 1 | Θ(n^1.585) |
| 3 | T(n) = 8T(n/2) + n² | 8 | 2 | 3 | 2 | 1 | Θ(n³) |
| 4 | T(n) = 7T(n/2) + n² | 7 | 2 | 2.807 | 2 | 1 | Θ(n^2.807) |
| 5 | T(n) = T(n/2) + n | 1 | 2 | 0 | 1 | 3 | Θ(n) |
| 6 | T(n) = T(n/2) + 1 | 1 | 2 | 0 | 0 | 2 | Θ(log n) |
| 7 | T(n) = 16T(n/4) + n | 16 | 4 | 2 | 1 | 1 | Θ(n²) |
| 8 | T(n) = 2T(n/2) + n log n | 2 | 2 | 1 | 1(k=1) | 2 | Θ(n log²n) |
| 9 | T(n) = 4T(n/2) + n² | 4 | 2 | 2 | 2 | 2 | Θ(n² log n) |
| 10 | T(n) = 2T(n/2) + n² | 2 | 2 | 1 | 2 | 3 | Θ(n²) |
| 11 | T(n) = 9T(n/3) + n | 9 | 3 | 2 | 1 | 1 | Θ(n²) |
| 12 | T(n) = 2T(n/4) + √n | 2 | 4 | 0.5 | 0.5 | 2 | Θ(√n log n) |

---

## Summary Table

| Difficulty | Example | Key Takeaway |
|------------|---------|-------------|
| Easy | Merge Sort, Binary Search | Classic Case 2 examples |
| Medium | Strassen, Karatsuba | Case 1 — reducing subproblems |
| Medium | T(n) = T(n/2) + n | Case 3 — root dominates |
| Hard | T(n) = 2T(n/2) + n log n | Case 2 with k > 0 |
| Tricky | Non-polynomial f(n) | May not fit Master Theorem |

---

## Quick Revision Questions

1. **Solve T(n) = 5T(n/5) + n using the Master Theorem.**
2. **For T(n) = 4T(n/2) + n³, what is the time complexity?**
3. **What case does T(n) = 64T(n/8) + n² fall under?**
4. **Solve T(n) = T(n/2) + log n. Hint: what is c_crit and c?**
5. **Why does T(n) = 8T(n/2) + n² give O(n³) — the same as brute force matrix multiplication?**
6. **Create your own recurrence for each of the three cases and solve them.**

---

[← Previous: Applying Master Theorem](03-applying-master-theorem.md) | [Next: Extended Master Theorem →](05-extended-master-theorem.md)
