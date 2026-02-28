# Chapter 1: The Closest Pair Problem

## Overview

Given **n points** in a 2D plane, find the pair of points with the **smallest Euclidean distance** between them. This classic computational geometry problem is a beautiful example of Divide and Conquer improving upon the brute-force approach.

---

## Problem Statement

```
INPUT:  A set P of n points: {p₁, p₂, ..., pₙ}
        Each point pᵢ = (xᵢ, yᵢ) in 2D space

OUTPUT: Two points pᵢ, pⱼ such that
        dist(pᵢ, pⱼ) is MINIMIZED over all pairs

DISTANCE: Euclidean distance
          dist(p, q) = √((p.x - q.x)² + (p.y - q.y)²)
```

---

## Visual Example

```
        y
        │
    8   │           ● E(7,8)
        │
    6   │   ● B(2,6)
        │                     ● F(9,5)
    4   │         ● C(4,4)
        │
    2   │ ● A(1,2)       ● D(6,2)
        │
    0   └──────────────────────────── x
        0  1  2  3  4  5  6  7  8  9

    All pairwise distances:
    ┌───────┬──────┬──────┬──────┬──────┬──────┐
    │       │  A   │  B   │  C   │  D   │  E   │  F
    ├───────┼──────┼──────┼──────┼──────┼──────┤
    │ A(1,2)│  -   │ 4.12 │ 3.61 │ 5.00 │ 7.81 │ 8.54
    │ B(2,6)│      │  -   │ 2.83 │ 5.66 │ 5.39 │ 7.07
    │ C(4,4)│      │      │  -   │ 2.83 │ 5.00 │ 5.10
    │ D(6,2)│      │      │      │  -   │ 6.08 │ 4.24
    │ E(7,8)│      │      │      │      │  -   │ 3.61
    │ F(9,5)│      │      │      │      │      │  -
    └───────┴──────┴──────┴──────┴──────┴──────┘

    ANSWER: B-C or C-D, distance = 2.83
```

---

## Brute Force Approach

### Algorithm

Check **every pair** of points and track the minimum distance.

```
BRUTE_FORCE_CLOSEST_PAIR(P):
    n ← |P|
    min_dist ← ∞
    best ← (null, null)
    
    for i ← 0 to n-2:
        for j ← i+1 to n-1:
            d ← DISTANCE(P[i], P[j])
            if d < min_dist:
                min_dist ← d
                best ← (P[i], P[j])
    
    return best, min_dist
```

### Trace

```
Points: A(1,2), B(2,6), C(4,4), D(6,2)

i=0, j=1: dist(A,B) = √(1+16)  = 4.12  → min = 4.12
i=0, j=2: dist(A,C) = √(9+4)   = 3.61  → min = 3.61
i=0, j=3: dist(A,D) = √(25+0)  = 5.00  → min = 3.61
i=1, j=2: dist(B,C) = √(4+4)   = 2.83  → min = 2.83  ← new best!
i=1, j=3: dist(B,D) = √(16+16) = 5.66  → min = 2.83
i=2, j=3: dist(C,D) = √(4+4)   = 2.83  → min = 2.83

Answer: (B,C) with distance 2.83
Total comparisons: C(4,2) = 6
```

### Complexity of Brute Force

```
Number of pairs = C(n, 2) = n(n-1)/2

Time:  O(n²)
Space: O(1)
```

---

## Why Brute Force Isn't Good Enough

```
┌──────────┬──────────────┬────────────────┐
│ n        │ Pairs (n²/2) │ At 10⁹ ops/sec │
├──────────┼──────────────┼────────────────┤
│ 1,000    │ 499,500      │ < 1 ms         │
│ 10,000   │ ~50 million  │ ~50 ms         │
│ 100,000  │ ~5 billion   │ ~5 seconds     │
│ 1,000,000│ ~500 billion │ ~8 minutes     │
└──────────┴──────────────┴────────────────┘

For large datasets (GPS coordinates, nearest-neighbor queries,
collision detection), O(n²) is far too slow.

D&C solution: O(n log n) — orders of magnitude faster!
```

---

## 1D Closest Pair: Warm-Up

Before tackling 2D, let's solve the simpler 1D version.

```
1D Problem: Given n numbers on a line, find two with smallest gap.

Solution: SORT them, then check adjacent pairs!

Example:
  Points: 7, 1, 4, 9, 2
  Sorted: 1, 2, 4, 7, 9
  Gaps:    1  2  3  2
  Answer: (1, 2), gap = 1

Time: O(n log n) for sort + O(n) scan = O(n log n)
```

### Why 1D Is Easy

```
After sorting:   ──•──•──────•────────•──•──
                   1  2      4        7  9

Closest pair MUST be adjacent in sorted order!
Proof: If a < b < c, then dist(a,c) = dist(a,b) + dist(b,c)
       So dist(a,c) > dist(a,b) and dist(a,c) > dist(b,c)
```

### Why 2D Is Harder

```
In 2D, there's no natural "adjacent" order:

           •
    •                •
         •
                •

Sorting by x doesn't guarantee closest pair is adjacent.
Need a more sophisticated approach → D&C!
```

---

## Applications

```
┌────────────────────────────┬──────────────────────────────────┐
│ Application                │ Details                          │
├────────────────────────────┼──────────────────────────────────┤
│ Collision detection        │ Nearest objects in simulations   │
│ Computer graphics          │ Closest vertices in meshes       │
│ Geographic systems         │ Nearest GPS coordinates          │
│ Clustering (single linkage)│ Finding closest clusters         │
│ Air traffic control        │ Detecting near-miss planes       │
│ Facility placement         │ Minimum separation constraints   │
│ VLSI design                │ Minimum wire spacing             │
│ Pattern recognition        │ Nearest neighbor in feature space│
└────────────────────────────┴──────────────────────────────────┘
```

---

## Road Map

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  BRUTE FORCE │     │  D&C BASIC   │     │  D&C + STRIP │
│    O(n²)     │ ──► │ O(n log²n)   │ ──► │  O(n log n)  │
│  (this file) │     │ (next file)  │     │  (file 3)    │
└──────────────┘     └──────────────┘     └──────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Find closest pair among n points in 2D |
| Brute force | Check all C(n,2) pairs → O(n²) |
| 1D solution | Sort + scan adjacent → O(n log n) |
| 2D challenge | No natural ordering in 2D |
| D&C goal | O(n log n) via clever divide step |

---

## Quick Revision Questions

1. **How many pairs exist among n points?**
2. **Why is the 1D closest pair problem easy after sorting?**
3. **What makes the 2D version harder than 1D?**
4. **What is the Euclidean distance formula?**
5. **Name three real-world applications of closest pair.**

---

[Next: D&C Approach →](02-dc-approach.md)
