# Chapter 4: Complexity Analysis — Closest Pair

## Overview

This chapter formally analyzes the time and space complexity of the D&C Closest Pair algorithm, proving the O(n log n) result and showing it is optimal.

---

## Recurrence Relation

```
The algorithm performs:
  1. Pre-sort by x: O(n log n)       (one-time cost)
  2. Pre-sort by y: O(n log n)       (one-time cost)
  3. Recursive calls: 2T(n/2)
  4. Split Py into Pyl, Pyr: O(n)
  5. Build strip: O(n)
  6. Check strip (≤ 7 comparisons per point): O(n)

Combine cost = O(n) + O(n) + O(n) = O(n)

RECURRENCE:
  T(n) = 2T(n/2) + O(n)
  T(n ≤ 3) = O(1)

Plus O(n log n) for pre-sorting.
```

---

## Solving via Master Theorem

```
T(n) = 2T(n/2) + cn

Standard form: T(n) = aT(n/b) + f(n)
  a = 2, b = 2, f(n) = cn

Compare f(n) with n^(log_b a):
  log_b a = log₂ 2 = 1
  n^(log_b a) = n¹ = n
  f(n) = cn = Θ(n)

Case 2 of Master Theorem: f(n) = Θ(n^(log_b a))
  → T(n) = Θ(n log n)
```

---

## Solving via Recursion Tree

```
Level 0:          cn                        = cn
                /      \
Level 1:    cn/2      cn/2                  = cn
           /    \    /    \
Level 2: cn/4 cn/4 cn/4 cn/4               = cn
          ...
Level k: n nodes of c                      = cn

Number of levels: log₂ n
Cost per level: cn

TOTAL = cn × log₂ n = O(n log n)

           Level    Nodes    Cost/Level
           ─────    ─────    ──────────
             0        1         cn
             1        2         cn
             2        4         cn
             ...
           log n    n          cn
                              ─────────
                    Total:  cn log n
```

---

## Total Time Complexity

```
┌───────────────────────────────────────────────────┐
│                                                   │
│  T_total = T_presort + T_recurse                  │
│          = O(n log n) + O(n log n)                │
│          = O(n log n)                             │
│                                                   │
└───────────────────────────────────────────────────┘

Breakdown:
  ┌─────────────────────────┬───────────┐
  │ Component               │ Time      │
  ├─────────────────────────┼───────────┤
  │ Sort by x (once)        │ O(n log n)│
  │ Sort by y (once)        │ O(n log n)│
  │ Recursive D&C           │ O(n log n)│
  │   ├─ Divide (per level) │ O(n)      │
  │   ├─ Build strip        │ O(n)      │
  │   └─ Check strip        │ O(n)      │
  ├─────────────────────────┼───────────┤
  │ TOTAL                   │ O(n log n)│
  └─────────────────────────┴───────────┘
```

---

## Without Y Pre-sorting: O(n log²n)

```
If we sort the strip by y at every recursive level:

Combine cost = O(n) + O(s log s) where s = strip size

Worst case: s = n (all points in strip)
Combine = O(n log n)

T(n) = 2T(n/2) + O(n log n)

Solving:
  Level 0:  cn log n
  Level 1:  2 · c(n/2) log(n/2) = cn(log n - 1)
  Level 2:  4 · c(n/4) log(n/4) = cn(log n - 2)
  ...
  Level k:  cn(log n - k)

  Total = cn Σ(k=0 to log n) (log n - k)
        = cn · (log n)(log n + 1)/2
        = O(n log²n)

┌─────────────────┬────────────────┐
│ Approach         │ Complexity     │
├─────────────────┼────────────────┤
│ Re-sort strip    │ O(n log²n)    │
│ Pre-sort + split │ O(n log n)    │
│ Brute force      │ O(n²)         │
└─────────────────┴────────────────┘
```

---

## Space Complexity

```
┌───────────────────────────┬──────────────┐
│ Storage                   │ Space        │
├───────────────────────────┼──────────────┤
│ Px (x-sorted copy)       │ O(n)         │
│ Py (y-sorted copy)       │ O(n)         │
│ Pyl, Pyr (per level)     │ O(n) total   │
│ Strip array               │ O(n)         │
│ Recursion stack           │ O(log n)     │
├───────────────────────────┼──────────────┤
│ TOTAL                     │ O(n)         │
└───────────────────────────┴──────────────┘
```

---

## Optimality Proof

```
THEOREM: Any comparison-based algorithm for Closest Pair
requires Ω(n log n) time.

PROOF (Reduction from Element Uniqueness):

  1. ELEMENT UNIQUENESS PROBLEM:
     Given n numbers, determine if all are distinct.
     Known lower bound: Ω(n log n) in comparison model.

  2. REDUCTION:
     Given numbers x₁, x₂, ..., xₙ
     Create points: (x₁, 0), (x₂, 0), ..., (xₙ, 0)
     
     Run Closest Pair algorithm.
     
     If min distance = 0 → duplicates exist
     If min distance > 0 → all distinct

  3. This solves Element Uniqueness using Closest Pair.
     If Closest Pair were o(n log n), Element Uniqueness
     would also be o(n log n) — a contradiction.

  4. Therefore: CLOSEST PAIR = Ω(n log n)               □

Combined with our O(n log n) algorithm:
  CLOSEST PAIR = Θ(n log n) — tight bound!
```

---

## Comparison of Approaches

```
┌──────────────────────┬──────────┬──────────┬──────────┐
│ Algorithm            │ Time     │ Space    │ Notes    │
├──────────────────────┼──────────┼──────────┼──────────┤
│ Brute Force          │ O(n²)    │ O(1)     │ Simple   │
│ D&C (no y-presort)   │O(n log²n)│ O(n)     │ Simpler  │
│ D&C (with y-presort) │O(n log n)│ O(n)     │ Optimal  │
│ Randomized           │ O(n) exp │ O(n)     │ Expected │
│ Lower bound          │Ω(n log n)│ —        │ Tight    │
└──────────────────────┴──────────┴──────────┴──────────┘
```

---

## Randomized O(n) Algorithm (Brief Mention)

```
Rabin's randomized algorithm achieves O(n) expected time:

1. Pick a random point pair and compute distance d
2. Build a grid with cell size d
3. Hash points into grid cells
4. Check only neighboring cells
5. Expected O(n) because grid lookups are O(1)

This is theoretically faster but:
  - More complex to implement
  - Expected, not worst-case
  - D&C O(n log n) is preferred in practice
```

---

## Practical Performance

```
n = 1,000,000 points

┌──────────────────┬────────────┬────────────────┐
│ Algorithm        │ Operations │ Approx Time    │
├──────────────────┼────────────┼────────────────┤
│ Brute Force      │ 5 × 10¹¹  │ ~500 seconds   │
│ D&C O(n log²n)   │ 4 × 10⁸   │ ~0.4 seconds   │
│ D&C O(n log n)   │ 2 × 10⁷   │ ~0.02 seconds  │
│ Randomized O(n)  │ ~10⁶      │ ~0.001 seconds │
└──────────────────┴────────────┴────────────────┘

The O(n log n) D&C is the standard choice:
fastest deterministic algorithm, practical, elegant.
```

---

## Summary Table

| Analysis | Result |
|----------|--------|
| Recurrence | T(n) = 2T(n/2) + O(n) |
| Master Theorem | Case 2: Θ(n log n) |
| Pre-sort overhead | O(n log n) — absorbed |
| Total time | O(n log n) |
| Space | O(n) |
| Lower bound | Ω(n log n) via Element Uniqueness |
| Optimality | Θ(n log n) — tight |

---

## Quick Revision Questions

1. **What is the recurrence for the D&C Closest Pair?**
2. **Which case of the Master Theorem applies?**
3. **How does pre-sorting improve from O(n log²n) to O(n log n)?**
4. **What problem reduces to Closest Pair for the lower bound?**
5. **What is the space complexity and why?**
6. **Can Closest Pair be solved faster than O(n log n)?**

---

[← Previous: Strip Optimization](03-strip-optimization.md) | [Next: 3D Extension →](05-3d-extension.md)
