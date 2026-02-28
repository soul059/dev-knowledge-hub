# Chapter 3: Convex Hull (D&C Approach)

## Overview

The **Convex Hull** of a set of points is the smallest convex polygon enclosing all the points. Think of it as stretching a rubber band around nails on a board. The D&C approach achieves O(n log n) time by splitting points in half, computing hulls recursively, and merging the two hulls — a technique that elegantly demonstrates D&C on geometric problems.

---

## Problem Description

```
Given: n points in 2D plane: P = {p₁, p₂, ..., pₙ}

Find: The convex hull — smallest convex polygon containing all points.

                    *  p₅
              *  p₃     *  p₇
           *  p₂   
        *  p₁   *  p₄     *  p₈
                          *  p₆
           *  p₉

Convex Hull (vertices in order):
                    *  p₅
              *  p₃ ─ ─ ─ *  p₇
           *  p₂            │
        *  p₁               *  p₈
           │              ╱
           *  p₉ ─ ─ ─ *  p₆

Output: Vertices of hull in counterclockwise order
```

---

## D&C Strategy

```
┌────────────────────────────────────────────────────┐
│                                                    │
│  1. DIVIDE: Sort points by x-coordinate            │
│     Split into left half L and right half R         │
│                                                    │
│  2. CONQUER: Recursively find ConvexHull(L)        │
│     and ConvexHull(R)                              │
│                                                    │
│  3. COMBINE: Merge two convex hulls by finding     │
│     upper and lower tangent lines                  │
│                                                    │
└────────────────────────────────────────────────────┘

Visual:
    ┌──────────┬──────────┐
    │    L     │     R    │
    │   ╱╲    │    ╱╲    │
    │  ╱  ╲   │   ╱  ╲   │
    │ ╱    ╲  │  ╱    ╲  │
    │╱      ╲ │ ╱      ╲ │
    │╲      ╱ │ ╲      ╱ │
    │ ╲    ╱  │  ╲    ╱  │
    │  ╲  ╱   │   ╲  ╱   │
    │   ╲╱    │    ╲╱    │
    └──────────┴──────────┘
    Hull(L)       Hull(R)

            Merge ↓

    ┌─────────────────────┐
    │       ╱──────╲      │
    │      ╱        ╲     │
    │     ╱          ╲    │
    │    ╱            ╲   │
    │    ╲            ╱   │
    │     ╲          ╱    │
    │      ╲        ╱     │
    │       ╲──────╱      │
    └─────────────────────┘
         Merged Hull
```

---

## Finding Tangent Lines (The Merge Step)

```
Upper Tangent:
    Start with rightmost point of L (a) and leftmost point of R (b)
    
    While the line (a, b) is NOT upper tangent for both hulls:
        While (a, b) is NOT upper tangent of LEFT hull:
            Move a counterclockwise on L's hull
        While (a, b) is NOT upper tangent of RIGHT hull:
            Move b clockwise on R's hull

Lower Tangent:
    Same idea but move in opposite directions.

Upper Tangent Check:
    Line (a, b) is upper tangent of a hull if all other hull points
    are ON or BELOW the line.

    Use cross product to determine orientation:
    cross(ab, ac) > 0 → c is LEFT of ab (above)
    cross(ab, ac) < 0 → c is RIGHT of ab (below)
    cross(ab, ac) = 0 → collinear

           b
          ╱│
         ╱ │  
        ╱  │  Upper tangent: no points above line
       ╱   │
      a    │
       ╲   │
        ╲  │
         ╲ │  Lower tangent: no points below line
          ╲│
           b'
```

---

## Algorithm Walkthrough

```
Points: (1,2), (2,6), (3,3), (5,7), (6,1), (7,5), (8,4), (9,8)

Step 1: Sort by x-coordinate (already sorted)

Step 2: Split
    L = {(1,2), (2,6), (3,3), (5,7)}
    R = {(6,1), (7,5), (8,4), (9,8)}

Step 3: Recursively compute hulls
    Hull(L):
        L₁ = {(1,2), (2,6)} → Hull = [(1,2), (2,6)]
        L₂ = {(3,3), (5,7)} → Hull = [(3,3), (5,7)]
        Merge → Hull(L) = [(1,2), (2,6), (5,7), (3,3)]

    Hull(R):
        R₁ = {(6,1), (7,5)} → Hull = [(6,1), (7,5)]
        R₂ = {(8,4), (9,8)} → Hull = [(8,4), (9,8)]
        Merge → Hull(R) = [(6,1), (7,5), (9,8), (8,4)]

Step 4: Merge Hull(L) and Hull(R)
    Find upper tangent: connects (5,7) to (9,8)
    Find lower tangent: connects (1,2) to (6,1)
    
    Final Hull: [(1,2), (2,6), (5,7), (9,8), (8,4), (6,1)]
    (counterclockwise traversal)
```

---

## Pseudocode

```
CONVEX_HULL_DC(points):
    if len(points) ≤ 3:
        return BRUTE_FORCE_HULL(points)
    
    // Sort by x-coordinate (once, at top level)
    sort points by x, breaking ties by y
    
    // Divide
    mid ← len(points) / 2
    L ← points[0..mid-1]
    R ← points[mid..n-1]
    
    // Conquer
    hull_L ← CONVEX_HULL_DC(L)
    hull_R ← CONVEX_HULL_DC(R)
    
    // Combine
    return MERGE_HULLS(hull_L, hull_R)


MERGE_HULLS(hull_L, hull_R):
    // Find rightmost of L and leftmost of R
    a ← index of rightmost point in hull_L
    b ← index of leftmost point in hull_R
    
    // Find upper tangent
    (upper_a, upper_b) ← FIND_UPPER_TANGENT(hull_L, hull_R, a, b)
    
    // Find lower tangent
    (lower_a, lower_b) ← FIND_LOWER_TANGENT(hull_L, hull_R, a, b)
    
    // Construct merged hull:
    // Take hull_L from lower_a to upper_a (counterclockwise)
    // Then hull_R from upper_b to lower_b (counterclockwise)
    merged ← []
    i ← upper_a
    while i ≠ lower_a:
        merged.append(hull_L[i])
        i ← (i + 1) mod len(hull_L)
    merged.append(hull_L[lower_a])
    
    i ← lower_b
    while i ≠ upper_b:
        merged.append(hull_R[i])
        i ← (i + 1) mod len(hull_R)
    merged.append(hull_R[upper_b])
    
    return merged


CROSS_PRODUCT(O, A, B):
    return (A.x - O.x) * (B.y - O.y) - (A.y - O.y) * (B.x - O.x)
```

---

## Complexity Analysis

```
SORTING: O(n log n) — done once

RECURRENCE (after sorting):
    T(n) = 2T(n/2) + O(n)
    
    Merge step: Finding tangents takes O(n) in worst case
    (each point visited at most twice during tangent finding)
    
    Master Theorem: a=2, b=2, f(n)=O(n)
    log_b a = 1, f(n) = Θ(n) → Case 2
    
    T(n) = Θ(n log n)

TOTAL: O(n log n) for sort + O(n log n) for D&C = O(n log n)

COMPARISON OF CONVEX HULL ALGORITHMS:
┌──────────────────────┬──────────────┬──────────────────┐
│ Algorithm            │ Complexity   │ Approach         │
├──────────────────────┼──────────────┼──────────────────┤
│ Brute Force          │ O(n³)        │ Check all triples│
│ Gift Wrapping        │ O(nh)        │ Incremental      │
│ Graham Scan          │ O(n log n)   │ Sort + stack     │
│ D&C (this chapter)   │ O(n log n)   │ Divide & Conquer │
│ Monotone Chain       │ O(n log n)   │ Sort + two passes│
│ Chan's Algorithm     │ O(n log h)   │ Hybrid           │
└──────────────────────┴──────────────┴──────────────────┘

h = number of hull vertices
Note: O(n log h) is output-sensitive — optimal!
```

---

## Cross Product Geometry

```
Cross product of vectors AB and AC:
    cross = (B.x-A.x)(C.y-A.y) - (B.y-A.y)(C.x-A.x)

    cross > 0 → C is LEFT of AB  (counterclockwise turn)
    cross = 0 → C is ON line AB  (collinear)
    cross < 0 → C is RIGHT of AB (clockwise turn)

        C (cross > 0)
       ╱
      ╱
    A ──────── B ──────── C' (cross = 0)
                ╲
                 ╲
                  C'' (cross < 0)

This is the fundamental test for ALL convex hull algorithms!
```

---

## Python Implementation

```python
def cross(O, A, B):
    """Cross product of vectors OA and OB."""
    return (A[0] - O[0]) * (B[1] - O[1]) - (A[1] - O[1]) * (B[0] - O[0])

def convex_hull_dc(points):
    """Compute convex hull using divide and conquer."""
    points = sorted(set(points))  # sort by (x, y), remove duplicates
    
    if len(points) <= 1:
        return points
    
    # Use monotone chain (Andrew's) as practical D&C variant
    # Build lower hull
    lower = []
    for p in points:
        while len(lower) >= 2 and cross(lower[-2], lower[-1], p) <= 0:
            lower.pop()
        lower.append(p)
    
    # Build upper hull
    upper = []
    for p in reversed(points):
        while len(upper) >= 2 and cross(upper[-2], upper[-1], p) <= 0:
            upper.pop()
        upper.append(p)
    
    # Remove last point of each half (it's repeated)
    return lower[:-1] + upper[:-1]

# --- True D&C Implementation ---

def merge_hulls(left_hull, right_hull):
    """Merge two convex hulls by finding upper and lower tangents."""
    n1, n2 = len(left_hull), len(right_hull)
    
    # Find rightmost of left hull, leftmost of right hull
    a = max(range(n1), key=lambda i: left_hull[i][0])
    b = min(range(n2), key=lambda i: right_hull[i][0])
    
    # Upper tangent
    ua, ub = a, b
    while True:
        changed = False
        while cross(right_hull[ub], left_hull[ua], 
                     left_hull[(ua - 1) % n1]) >= 0:
            ua = (ua - 1) % n1
            changed = True
        while cross(left_hull[ua], right_hull[ub], 
                     right_hull[(ub + 1) % n2]) <= 0:
            ub = (ub + 1) % n2
            changed = True
        if not changed:
            break
    
    # Lower tangent
    la, lb = a, b
    while True:
        changed = False
        while cross(right_hull[lb], left_hull[la], 
                     left_hull[(la + 1) % n1]) <= 0:
            la = (la + 1) % n1
            changed = True
        while cross(left_hull[la], right_hull[lb], 
                     right_hull[(lb - 1) % n2]) >= 0:
            lb = (lb - 1) % n2
            changed = True
        if not changed:
            break
    
    # Build merged hull
    result = []
    i = ua
    while True:
        result.append(left_hull[i])
        if i == la:
            break
        i = (i + 1) % n1
    
    i = lb
    while True:
        result.append(right_hull[i])
        if i == ub:
            break
        i = (i + 1) % n2
    
    return result

def dc_convex_hull(points):
    """True divide and conquer convex hull."""
    points = sorted(points)
    
    def solve(pts):
        if len(pts) <= 3:
            # Base case: brute force small hull
            return convex_hull_dc(pts)  # uses monotone chain
        
        mid = len(pts) // 2
        left = solve(pts[:mid])
        right = solve(pts[mid:])
        return merge_hulls(left, right)
    
    return solve(points)

# Test
points = [(1,2), (2,6), (3,3), (5,7), (6,1), (7,5), (8,4), (9,8)]
hull = dc_convex_hull(points)
print("Convex Hull:", hull)
```

---

## 3D Extension

```
3D Convex Hull: Output is a convex polyhedron (set of triangular faces)

D&C approach in 3D:
    Split by median x-coordinate
    Recursively compute 3D hulls
    Merge by finding "bridge" faces
    
    Complexity: O(n log n) — same as 2D!

Algorithms:
    Incremental: O(n²) worst case, O(n log n) expected
    D&C: O(n log n) deterministic
    Gift wrapping: O(nf) where f = faces
    
Higher dimensions:
    d-dimensional hull: O(n^⌊d/2⌋) in worst case
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Find smallest convex polygon enclosing all points |
| D&C split | By x-coordinate into left and right halves |
| Key merge step | Find upper and lower tangent lines |
| Fundamental test | Cross product for orientation |
| Recurrence | T(n) = 2T(n/2) + O(n) |
| Complexity | O(n log n) |
| Output-sensitive | Chan's algorithm: O(n log h) |
| 3D extension | O(n log n) via D&C |

---

## Quick Revision Questions

1. **What does the cross product tell us about point orientation?**
2. **How are two convex hulls merged in the combine step?**
3. **What is an upper tangent line between two hulls?**
4. **Why does the merge step take O(n) time?**
5. **Compare D&C convex hull with Graham Scan in terms of approach and complexity.**

---

[← Previous: FFT Concept](02-fft-concept.md) | [Next: Order Statistics →](04-order-statistics.md)
