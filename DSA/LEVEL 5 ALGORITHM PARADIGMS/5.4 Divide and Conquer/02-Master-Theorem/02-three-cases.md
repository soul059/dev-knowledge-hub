# Chapter 2: Three Cases of the Master Theorem

## Overview

The Master Theorem's three cases cover all possible relationships between the **work done at higher levels** (root) and **work done at lower levels** (leaves) of the recursion tree. This chapter provides deep intuition for each case with visual explanations.

---

## Recap: The Setup

```
Given: T(n) = a·T(n/b) + f(n)

Recursion tree:
Level 0:  1 problem of size n          → Work: f(n)
Level 1:  a problems of size n/b       → Work: a · f(n/b)
Level 2:  a² problems of size n/b²     → Work: a² · f(n/b²)
  ...
Level i:  aⁱ problems of size n/bⁱ     → Work: aⁱ · f(n/bⁱ)
  ...
Level L:  a^L problems of size 1        → Work: a^L · Θ(1) = Θ(n^(log_b a))

Where L = log_b(n) = depth of recursion tree
```

---

## Case 1: Leaf-Heavy (f(n) is Polynomially Smaller)

### Condition
`f(n) = O(n^c)` where `c < log_b(a)`

### Result
`T(n) = Θ(n^(log_b(a)))`

### Intuition

The work **increases** as you go deeper in the tree. The leaves do the most work.

```
Work distribution in the recursion tree:

Level 0:  ·                            ← Small work
Level 1:  · ·                          ← More work  
Level 2:  · · · ·                      ← Even more work
Level 3:  · · · · · · · ·              ← Even more work
   ...
Leaves:   · · · · · · · · · · · · · ·  ← MOST WORK (dominates!)

Total ≈ Work at leaves = n^(log_b(a))
```

### Why Does This Happen?

```
At each level, work ratio = a · f(n/b) / f(n)

If f(n) = n^c and c < log_b(a):
  Ratio = a · (n/b)^c / n^c = a / b^c > 1
  
  Since c < log_b(a) → b^c < a → a/b^c > 1
  
  Work INCREASES at each level! → Geometric series dominated by last term
```

### Example: T(n) = 8T(n/2) + n²

```
a = 8, b = 2, f(n) = n²
c_crit = log₂(8) = 3
f(n) = n² → c = 2 < 3 = c_crit

Case 1 applies!

T(n) = Θ(n³)

Recursion tree work per level:
Level 0: n²
Level 1: 8 · (n/2)² = 8 · n²/4 = 2n²       ← doubled!
Level 2: 8² · (n/4)² = 64 · n²/16 = 4n²     ← doubled again!
Level 3: 8n²                                   ← keeps growing!

Leaf level dominates → total ≈ n^(log₂ 8) = n³
```

---

## Case 2: Balanced (f(n) Matches the Critical Exponent)

### Condition
`f(n) = Θ(n^(log_b(a)) · log^k(n))` for `k ≥ 0`

### Result
`T(n) = Θ(n^(log_b(a)) · log^(k+1)(n))`

Most common sub-case: `k = 0` → `f(n) = Θ(n^(log_b(a)))` → `T(n) = Θ(n^(log_b(a)) · log n)`

### Intuition

Every level of the recursion tree does approximately the **same amount of work**.

```
Work distribution in the recursion tree:

Level 0:  ████████████████████████   ← Same work
Level 1:  ████████████████████████   ← Same work
Level 2:  ████████████████████████   ← Same work
Level 3:  ████████████████████████   ← Same work
   ...
Leaves:   ████████████████████████   ← Same work

Total = (work per level) × (number of levels)
      = Θ(n^(log_b(a))) × Θ(log n)
      = Θ(n^(log_b(a)) · log n)
```

### Why Does This Happen?

```
At each level, work ratio = a · f(n/b) / f(n)

If f(n) = n^c and c = log_b(a):
  Ratio = a · (n/b)^c / n^c = a / b^c = a / b^(log_b(a)) = a/a = 1
  
  Work is CONSTANT at each level!
  → Each of log_b(n) levels contributes Θ(n^c)
  → Total = Θ(n^c · log n)
```

### Example: Merge Sort — T(n) = 2T(n/2) + n

```
a = 2, b = 2, f(n) = n
c_crit = log₂(2) = 1
f(n) = n = Θ(n¹) = Θ(n^c_crit)

Case 2 applies (k = 0)!

T(n) = Θ(n · log n) = Θ(n log n)

Recursion tree work per level:
Level 0: n · 1 = n                      ← n work
Level 1: 2 · (n/2) = n                  ← n work  (same!)
Level 2: 4 · (n/4) = n                  ← n work  (same!)
Level 3: 8 · (n/8) = n                  ← n work  (same!)
   ...
Level log₂(n): n · 1 = n                ← n work  (same!)

Total = n × log₂(n) levels = n log n ✓
```

### Example: Binary Search — T(n) = T(n/2) + 1

```
a = 1, b = 2, f(n) = 1
c_crit = log₂(1) = 0
f(n) = 1 = Θ(n⁰) = Θ(n^c_crit)

Case 2 applies (k = 0)!

T(n) = Θ(n⁰ · log n) = Θ(log n)

Each level does O(1) work, log n levels → O(log n) ✓
```

---

## Case 3: Root-Heavy (f(n) is Polynomially Larger)

### Condition
`f(n) = Ω(n^c)` where `c > log_b(a)`, AND the **regularity condition** holds:
`a · f(n/b) ≤ k · f(n)` for some constant `k < 1`

### Result
`T(n) = Θ(f(n))`

### Intuition

The work **decreases** as you go deeper. The root does the most work.

```
Work distribution in the recursion tree:

Level 0:  ████████████████████████████   ← MOST WORK (dominates!)
Level 1:  ██████████████████             ← Less work
Level 2:  ██████████████                 ← Even less
Level 3:  ██████████                     ← Even less
   ...
Leaves:   ····                           ← Negligible

Total ≈ Work at root = Θ(f(n))
```

### Why Does This Happen?

```
At each level, work ratio = a · f(n/b) / f(n)

If f(n) = n^c and c > log_b(a):
  Ratio = a · (n/b)^c / n^c = a / b^c < 1  (since b^c > a)
  
  Work DECREASES at each level!
  → Geometric series dominated by first term (root)
```

### Example: T(n) = 2T(n/2) + n²

```
a = 2, b = 2, f(n) = n²
c_crit = log₂(2) = 1
f(n) = n² → c = 2 > 1 = c_crit

Check regularity: 2 · f(n/2) = 2 · (n/2)² = n²/2 ≤ ½ · n² ✓ (k = ½)

Case 3 applies!

T(n) = Θ(n²)

Recursion tree:
Level 0: n²
Level 1: 2 · (n/2)² = n²/2                ← halved!
Level 2: 4 · (n/4)² = n²/4                ← halved again!
Level 3: n²/8

Total = n²(1 + 1/2 + 1/4 + ...) ≈ 2n² = Θ(n²)
```

---

## Visual Comparison of All Three Cases

```
          Work per level          Case 1       Case 2       Case 3
          
Level 0:  f(n)                    small        medium       LARGE
Level 1:  a·f(n/b)               medium       medium       medium
Level 2:  a²·f(n/b²)            large        medium       small
  ...
Leaves:   n^(log_b a)            LARGE        medium       small

                                  ▲            ▲            ▲
                                  │            │            │
                              Leaves       All levels     Root
                              dominate     equal          dominates
                                  │            │            │
                              Θ(n^(log_b a)) Θ(n^(log_b a)·log n)  Θ(f(n))
```

---

## Decision Flowchart

```
START: T(n) = a·T(n/b) + f(n)
          │
    Compute c_crit = log_b(a)
          │
    Compare f(n) with n^c_crit
          │
    ┌─────┼─────────────┐
    │     │              │
    ▼     ▼              ▼
  c < c_crit   c = c_crit    c > c_crit
    │          │              │
    │    f(n) has log^k?    Regularity
    │     │                  holds?
    │    YES                  │
    │     │               ┌───┴───┐
    ▼     ▼              YES     NO
 CASE 1  CASE 2           │       │
    │     │             CASE 3   Can't
    ▼     ▼               │      apply
Θ(n^c_crit) Θ(n^c_crit·log^(k+1)n)  │  MT
              │               ▼
              │           Θ(f(n))
```

---

## Common Recurrences: Quick Reference

| Recurrence | a | b | c_crit | f(n) | Case | T(n) |
|-----------|---|---|--------|------|------|------|
| T(n) = T(n/2) + 1 | 1 | 2 | 0 | 1 | 2 | Θ(log n) |
| T(n) = 2T(n/2) + 1 | 2 | 2 | 1 | 1 | 1 | Θ(n) |
| T(n) = 2T(n/2) + n | 2 | 2 | 1 | n | 2 | Θ(n log n) |
| T(n) = 2T(n/2) + n² | 2 | 2 | 1 | n² | 3 | Θ(n²) |
| T(n) = 4T(n/2) + n | 4 | 2 | 2 | n | 1 | Θ(n²) |
| T(n) = 4T(n/2) + n² | 4 | 2 | 2 | n² | 2 | Θ(n² log n) |
| T(n) = 3T(n/2) + n | 3 | 2 | 1.585 | n | 1 | Θ(n^1.585) |
| T(n) = 7T(n/2) + n² | 7 | 2 | 2.807 | n² | 1 | Θ(n^2.807) |
| T(n) = T(n/2) + n | 1 | 2 | 0 | n | 3 | Θ(n) |
| T(n) = 2T(n/4) + √n | 2 | 4 | 0.5 | √n | 2 | Θ(√n · log n) |

---

## Summary Table

| Aspect | Case 1 | Case 2 | Case 3 |
|--------|--------|--------|--------|
| **Condition** | c < log_b(a) | c = log_b(a) | c > log_b(a) |
| **f(n) vs leaves** | f(n) smaller | f(n) matches | f(n) larger |
| **Work distribution** | Bottom-heavy | Uniform | Top-heavy |
| **Dominates** | Leaves | All levels equally | Root |
| **Result** | Θ(n^(log_b(a))) | Θ(n^(log_b(a))·log n) | Θ(f(n)) |
| **Extra condition** | None | Check for log factor | Regularity check |

---

## Quick Revision Questions

1. **What does "leaf-heavy" mean intuitively, and which case does it correspond to?**
2. **In Case 2, why does a `log n` factor appear in the result?**
3. **What is the regularity condition in Case 3, and why is it needed?**
4. **For T(n) = 4T(n/2) + n³, identify the case and solve.**
5. **Why do all levels contribute equally in Case 2? Show with an example.**
6. **Draw the recursion tree work distribution for T(n) = 2T(n/2) + 1 and identify the case.**

---

[← Previous: Master Theorem Statement](01-master-theorem-statement.md) | [Next: Applying Master Theorem →](03-applying-master-theorem.md)
