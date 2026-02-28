# Chapter 3: The Strip Optimization

## Overview

The strip optimization is the **key insight** that makes the Closest Pair algorithm run in O(n log n). Without it, checking pairs across the dividing line would take O(n²), making D&C no better than brute force. The trick: each point in the strip needs to be compared with **at most 7 other points**.

---

## The Problem with the Strip

```
After the recursive step, we have δ = min(δ_L, δ_R).

Strip = all points within distance δ of the dividing line.

       │←── δ ──→│←── δ ──→│
       │         │         │
       │    •    │    •    │
       │  •      │      •  │
       │    •    │   •     │
       │       • │         │
       │  •      │    •    │
       │         │         │

WORST CASE: All n points could be in the strip!
Naively checking all pairs in strip: O(n²)

T(n) = 2T(n/2) + O(n²) → T(n) = O(n²)  ← No improvement!
```

---

## The Key Geometric Insight

```
THEOREM: In the strip, each point needs to be compared
with at most 7 points ahead in y-sorted order.

WHY? Consider a δ × 2δ rectangle centered at point p:

    ┌──────────┬──────────┐  ─┐
    │          │          │   │
    │  Left    │  Right   │   │ δ
    │  half    │  half    │   │
    ├──────────┤──────────┤  ─┤
    │          │     ●    │   │
    │  Left    │ p  Right │   │ δ
    │  half    │  half    │   │
    └──────────┴──────────┘  ─┘
    │←── δ ──→│←── δ ──→│

    Rectangle: 2δ wide × δ tall (above p's y)
    
    Any point q with dist(p,q) < δ must be in this box!
    (Because |p.x - q.x| < δ and |p.y - q.y| < δ)
```

---

## Why At Most 8 Points in the Box

```
Divide the δ × 2δ rectangle into 8 cells of size δ/2 × δ/2:

    ┌─────┬─────┬─────┬─────┐
    │  5  │  6  │  7  │  8  │  ← each cell: δ/2 × δ/2
    ├─────┼─────┼─────┼─────┤
    │  1  │  2  │  3  │  4  │
    └─────┴─────┴─────┴─────┘
    │←─ δ ──→│←─── δ ──→│
        Left      Right

Each cell has diagonal = √((δ/2)² + (δ/2)²) = δ/√2 < δ

Since the minimum distance within each half is ≥ δ,
each cell can contain AT MOST 1 point!
(Two points in the same cell on the same side 
 would have distance < δ, contradicting our δ.)

8 cells → at most 8 points → at most 7 comparisons per point!
```

### Formal Proof

```
Claim: At most 8 points can exist in the δ × 2δ rectangle.

Proof:
  1. The rectangle spans the dividing line, with δ on each side.
  
  2. Left side: δ × δ square. Divide into 4 cells of (δ/2 × δ/2).
     - Max diagonal of each cell = δ√2/2 ≈ 0.707δ < δ
     - Two points in same cell on left side → distance < δ
     - But δ_L ≥ δ, so at most 1 point per cell
     - Left side: ≤ 4 points
  
  3. Right side: same argument → ≤ 4 points
  
  4. Total: ≤ 4 + 4 = 8 points (including p itself)
  
  5. Comparisons needed for p: ≤ 8 - 1 = 7   □
```

---

## The Optimized Strip Check

```
CLOSEST_IN_STRIP(strip, δ):
    // strip is SORTED BY Y-COORDINATE
    min_dist ← δ
    
    for i ← 0 to |strip| - 1:
        j ← i + 1
        while j < |strip| AND strip[j].y - strip[i].y < δ:
            d ← DISTANCE(strip[i], strip[j])
            if d < min_dist:
                min_dist ← d
            j ← j + 1
    
    return min_dist
```

### Why the Inner Loop Is O(1) Per Point

```
For each point strip[i]:
  - We check strip[i+1], strip[i+2], ... 
  - We STOP when y-difference ≥ δ
  - At most 7 points can be within y-distance δ
  - So inner loop runs ≤ 7 iterations per i

Total strip check: O(7n) = O(n)
```

---

## Visual Trace of Strip Check

```
Strip (sorted by y):
     y
     │
  12 │     ● G            G(6,12)       outside δ range of A
  10 │   ● F              F(4,10)       outside δ range of A
   8 │     ● E  ● D       E(6,8) D(7,8)
   6 │   ● C              C(4,6)
   4 │ ● A   ● B          A(3,4) B(5,4)
     │
     └────────────── x

Let δ = 3.0

Processing A(3,4):
  Check B(5,4): |4-4|=0 < 3 → dist = 2.0 ✓ (new min = 2.0)
  Check C(4,6): |6-4|=2 < 3 → dist = √(1+4) = 2.24
  Check E(6,8): |8-4|=4 ≥ 3 → STOP

Processing B(5,4):
  Check C(4,6): |6-4|=2 < 3 → dist = √(1+4) = 2.24
  Check E(6,8): |8-4|=4 ≥ 3 → STOP

Processing C(4,6):
  Check E(6,8): |8-6|=2 < 3 → dist = √(4+4) = 2.83
  Check D(7,8): |8-6|=2 < 3 → dist = √(9+4) = 3.61
  Check F(4,10): |10-6|=4 ≥ 3 → STOP

Processing E(6,8):
  Check D(7,8): |8-8|=0 < 3 → dist = 1.0 ✓ (new min = 1.0!)
  Check F(4,10): |10-8|=2 < 3 → dist = √(4+4) = 2.83
  Check G(6,12): |12-8|=4 ≥ 3 → STOP

Answer from strip: (E, D), distance = 1.0
Each point checked at most 3-4 others (well under 7)
```

---

## The Constant 7: A Tighter Bound

```
Can we do better than 7?

In practice, at most 6 points need checking.
Some analyses show 5 is sufficient.

The exact constant doesn't matter for Big-O:
  It's O(1) comparisons per point regardless.

┌───────────────┬────────────────────────┐
│ Bound         │ Source                 │
├───────────────┼────────────────────────┤
│ 7             │ Standard textbook      │
│ 6             │ Tighter analysis       │
│ 5             │ With careful argument  │
│ O(1) anyway   │ All give O(n) strip    │
└───────────────┴────────────────────────┘
```

---

## Complete Algorithm with Strip Optimization

```
CLOSEST_PAIR(P):
    Px ← sort P by x-coordinate        // O(n log n)
    Py ← sort P by y-coordinate        // O(n log n)
    return CLOSEST_REC(Px, Py)

CLOSEST_REC(Px, Py):
    n ← |Px|
    if n ≤ 3:
        return BRUTE_FORCE(Px)

    mid ← n / 2
    midX ← Px[mid].x

    // Split preserving y-order
    Pyl, Pyr ← split Py at midX        // O(n)

    (p1, q1, δL) ← CLOSEST_REC(Px[0..mid-1], Pyl)
    (p2, q2, δR) ← CLOSEST_REC(Px[mid..n-1], Pyr)

    δ ← min(δL, δR)
    best ← pair achieving δ

    // Build strip from Py (already y-sorted!)
    strip ← [p ∈ Py : |p.x - midX| < δ]     // O(n)

    // check strip: O(n) with the 7-point bound
    for i ← 0 to |strip| - 1:
        for j ← i+1 while strip[j].y - strip[i].y < δ:
            d ← dist(strip[i], strip[j])
            if d < δ:
                δ ← d
                best ← (strip[i], strip[j])

    return best, δ
```

---

## Why Pre-sorting by Y Matters

```
WITHOUT pre-sorting by y:
  Must sort strip by y each recursive call
  Strip has up to n points → O(n log n) per level
  T(n) = 2T(n/2) + O(n log n) = O(n log²n)

WITH pre-sorting by y (Py maintained throughout):
  Strip already in y-order (subset of Py)
  Building strip: O(n) scan
  Checking strip: O(n)
  T(n) = 2T(n/2) + O(n) = O(n log n)    ← Better!

┌────────────────────┬─────────────────┐
│ Approach           │ Time            │
├────────────────────┼─────────────────┤
│ No y pre-sort      │ O(n log²n)      │
│ With y pre-sort    │ O(n log n)      │
│ Difference at n=10⁶│ 20× speed-up   │
└────────────────────┴─────────────────┘
```

---

## Edge Cases

```
1. Duplicate points: distance = 0 → handle early
2. All points on a line: strip degenerates, but still O(n)
3. All points in strip: unusual but inner loop still O(1) per point
4. Ties at midpoint: careful with ≤ vs < when splitting
5. n = 1: no pair exists → return ∞
6. n = 2: trivially return that pair
```

---

## Summary Table

| Concept | Details |
|---------|---------|
| Strip width | 2δ centered on dividing line |
| Key insight | At most 8 points in any δ × 2δ box |
| Comparisons per point | ≤ 7 (constant) |
| Strip check time | O(n) |
| Why y-sort helps | Strip already sorted, no re-sorting |
| Without y pre-sort | O(n log²n) total |
| With y pre-sort | O(n log n) total |

---

## Quick Revision Questions

1. **Why does naively checking the strip give O(n²)?**
2. **How many points can fit in a δ × 2δ rectangle?**
3. **Why can each δ/2 × δ/2 cell contain at most 1 point?**
4. **What stops the inner loop in the optimized strip check?**
5. **How does pre-sorting by y improve from O(n log²n) to O(n log n)?**
6. **Does the 7-point bound hold for all inputs?**

---

[← Previous: D&C Approach](02-dc-approach.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)
