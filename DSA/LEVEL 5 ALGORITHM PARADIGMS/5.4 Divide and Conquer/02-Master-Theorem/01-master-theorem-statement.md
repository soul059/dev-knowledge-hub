# Chapter 1: Master Theorem Statement

## Overview

The **Master Theorem** is a powerful tool for solving recurrence relations that arise from Divide and Conquer algorithms. Instead of manually expanding recurrences, the Master Theorem gives you the answer in one step — if the recurrence fits its standard form.

---

## The Standard Recurrence Form

Every D&C algorithm produces a recurrence of this form:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│           T(n) = a · T(n/b) + f(n)                         │
│                                                             │
│   Where:                                                    │
│     T(n) = total time for input of size n                  │
│     a    = number of subproblems (a ≥ 1)                   │
│     n/b  = size of each subproblem (b > 1)                 │
│     f(n) = cost of divide + combine steps                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Visualizing the Parameters

```
               T(n)                    ← Total cost
              / | \
             /  |  \
           T(n/b) T(n/b) ... T(n/b)   ← 'a' subproblems, each of size n/b
           
           + f(n)                       ← Work done outside recursive calls
           
Example: Merge Sort
  T(n) = 2·T(n/2) + O(n)
         ▲     ▲      ▲
         a=2   b=2    f(n)=O(n) [merge step]
```

---

## The Master Theorem Statement

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  MASTER THEOREM                                                     │
│                                                                     │
│  Given: T(n) = a·T(n/b) + f(n), where a ≥ 1, b > 1               │
│                                                                     │
│  Let c_crit = log_b(a)   (the critical exponent)                   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ CASE 1: If f(n) = O(n^c) where c < log_b(a)               │    │
│  │         Then T(n) = Θ(n^(log_b(a)))                        │    │
│  │         → "Leaves dominate"                                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ CASE 2: If f(n) = Θ(n^(log_b(a)) · log^k(n)) for k ≥ 0   │    │
│  │         Then T(n) = Θ(n^(log_b(a)) · log^(k+1)(n))        │    │
│  │         → "All levels contribute equally"                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ CASE 3: If f(n) = Ω(n^c) where c > log_b(a)               │    │
│  │         AND a·f(n/b) ≤ k·f(n) for some k < 1 (regularity) │    │
│  │         Then T(n) = Θ(f(n))                                │    │
│  │         → "Root dominates"                                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Understanding the Critical Exponent

The **critical exponent** `c_crit = log_b(a)` represents the rate at which work grows at the leaf level of the recursion tree.

```
c_crit = log_b(a)

Meaning: At each level, work multiplies by a/b^c_crit = 1
         (leaves contribute n^c_crit total work)

Examples:
  Binary Search:  a=1, b=2 → c_crit = log₂(1) = 0
  Merge Sort:     a=2, b=2 → c_crit = log₂(2) = 1
  Strassen:       a=7, b=2 → c_crit = log₂(7) ≈ 2.807
  Karatsuba:      a=3, b=2 → c_crit = log₂(3) ≈ 1.585
```

---

## Intuition Behind the Three Cases

The Master Theorem compares `f(n)` (work at the root level) vs `n^(log_b(a))` (work at the leaf level):

```
CASE 1: f(n) grows SLOWER than n^(log_b(a))
┌─────────────────────────────────────┐
│  Root:    small work                │
│  ...                                │
│  Leaves:  LARGE work  ← DOMINATES  │
│                                     │
│  Answer = Θ(n^(log_b(a)))          │
│  "Leaf-heavy"                       │
└─────────────────────────────────────┘

CASE 2: f(n) grows at the SAME rate as n^(log_b(a))
┌─────────────────────────────────────┐
│  Root:    moderate work             │
│  Middle:  moderate work             │
│  Leaves:  moderate work             │
│  ← ALL LEVELS CONTRIBUTE EQUALLY → │
│                                     │
│  Answer = Θ(n^(log_b(a)) · log n)  │
│  "Balanced"                         │
└─────────────────────────────────────┘

CASE 3: f(n) grows FASTER than n^(log_b(a))
┌─────────────────────────────────────┐
│  Root:    LARGE work  ← DOMINATES  │
│  ...                                │
│  Leaves:  small work                │
│                                     │
│  Answer = Θ(f(n))                  │
│  "Root-heavy"                       │
└─────────────────────────────────────┘
```

---

## The Recursion Tree Perspective

```
Level 0:       ─────── f(n) ───────           Work: f(n)
              /                    \
Level 1:   f(n/b)    ...    f(n/b)            Work: a·f(n/b)
            / \                / \
Level 2: f(n/b²) ...      f(n/b²)            Work: a²·f(n/b²)
           ...                                 ...
Level k:                                       Work: a^k · f(n/b^k)
           ...
Level log_b(n):  T(1) T(1) ... T(1)          Work: a^(log_b(n)) = n^(log_b(a))
                 └── n^(log_b(a)) leaves ──┘

Total Work = Σ (work at each level)
           = f(n) + a·f(n/b) + a²·f(n/b²) + ... + n^(log_b(a))·T(1)
```

The three cases depend on whether this sum is dominated by the first term, the last term, or is a balanced geometric series.

---

## Quick Identification Guide

```
Step 1: Identify a, b, f(n) from T(n) = a·T(n/b) + f(n)
Step 2: Compute c_crit = log_b(a)
Step 3: Compare f(n) with n^c_crit:

  f(n) = O(n^c) with c < c_crit  →  CASE 1: T(n) = Θ(n^c_crit)
  f(n) = Θ(n^c_crit)             →  CASE 2: T(n) = Θ(n^c_crit · log n)
  f(n) = Ω(n^c) with c > c_crit  →  CASE 3: T(n) = Θ(f(n))
```

---

## Examples at a Glance

| Recurrence | a | b | f(n) | c_crit | Case | T(n) |
|-----------|---|---|------|--------|------|------|
| T(n) = 2T(n/2) + n | 2 | 2 | n | 1 | Case 2 | Θ(n log n) |
| T(n) = 2T(n/2) + 1 | 2 | 2 | 1 | 1 | Case 1 | Θ(n) |
| T(n) = T(n/2) + 1 | 1 | 2 | 1 | 0 | Case 2 | Θ(log n) |
| T(n) = 4T(n/2) + n | 4 | 2 | n | 2 | Case 1 | Θ(n²) |
| T(n) = 2T(n/2) + n² | 2 | 2 | n² | 1 | Case 3 | Θ(n²) |
| T(n) = 7T(n/2) + n² | 7 | 2 | n² | 2.807 | Case 1 | Θ(n^2.807) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Standard form | T(n) = a·T(n/b) + f(n) |
| Critical exponent | c_crit = log_b(a) |
| Case 1 | Leaves dominate → T(n) = Θ(n^(log_b(a))) |
| Case 2 | Balanced → T(n) = Θ(n^(log_b(a)) · log n) |
| Case 3 | Root dominates → T(n) = Θ(f(n)) |
| Key comparison | Compare growth of f(n) vs n^(log_b(a)) |

---

## Quick Revision Questions

1. **What is the standard form of a D&C recurrence that the Master Theorem applies to?**
2. **What does the critical exponent `log_b(a)` represent intuitively?**
3. **In which case does the leaf-level work dominate the total complexity?**
4. **For Merge Sort T(n) = 2T(n/2) + O(n), what is a, b, f(n), and c_crit?**
5. **What additional condition (regularity) must be checked in Case 3?**
6. **If a = 4, b = 2, and f(n) = n, which case applies and what is the result?**

---

[← Back to Unit 1](../01-DC-Fundamentals/06-classic-examples-overview.md) | [Next: Three Cases →](02-three-cases.md)
