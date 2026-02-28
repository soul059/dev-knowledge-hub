# Chapter 4: Closest Pair of Points

[← Previous: Point in Polygon](03-point-in-polygon.md) | [Back to README](../README.md) | [Next: Sweep Line Algorithm →](05-sweep-line-algorithm.md)

---

## Overview

Given $n$ points, find the pair with minimum Euclidean distance. The naive approach is $O(n^2)$; the **divide and conquer** approach achieves $O(n \log n)$.

---

## 4.1 Naive Approach

```
FUNCTION closestPairNaive(points, n):
    minDist = ∞
    FOR i = 0 TO n - 1:
        FOR j = i + 1 TO n - 1:
            d = dist(points[i], points[j])
            IF d < minDist:
                minDist = d
                bestPair = (i, j)
    RETURN minDist

Time:  O(n²)
Space: O(1)
```

---

## 4.2 Divide and Conquer — O(n log n)

```
  ┌──────────────────────────────────────────────────────────┐
  │  Algorithm:                                              │
  │  1. Sort points by x-coordinate                         │
  │  2. DIVIDE into left and right halves                   │
  │  3. Recursively find closest pair in each half          │
  │  4. δ = min(left_min, right_min)                        │
  │  5. COMBINE: check pairs spanning the dividing line,    │
  │     but only within a 2δ-wide strip!                    │
  └──────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────┐
  │          │                                               │
  │    L     │     R        The strip around the            │
  │   ·  ·   │   ·         dividing line has width 2δ.     │
  │     ·    │  ·  ·       Only O(n) points in the strip.  │
  │  ·       │    ·        Each point needs to check only   │
  │    ·   · │ ·           ~7 neighbors (sorted by y).     │
  │          │                                               │
  │  ←──δ──→│←──δ──→                                       │
  └─────────────────────────────────────────────────────────┘
```

### The Key Insight: Strip Processing

```
  Within the strip of width 2δ, sort points by y-coordinate.
  
  For each point P in the strip:
    Only check the next 7 points (by y-order).
    
  Why at most 7?
  In a δ × 2δ rectangle, at most 8 points can exist
  (each at distance ≥ δ from each other, by packing argument).
  So each point has ≤ 7 neighbors to check.
  
  ★ This makes the strip processing O(n), not O(n²)!
```

---

## 4.3 Pseudocode

```
FUNCTION closestPair(points):
    Sort points by x
    RETURN closestPairRec(points, 0, n - 1)

FUNCTION closestPairRec(P, lo, hi):
    IF hi - lo < 3:
        RETURN bruteForce(P, lo, hi)
    
    mid = (lo + hi) / 2
    midX = P[mid].x
    
    dL = closestPairRec(P, lo, mid)
    dR = closestPairRec(P, mid + 1, hi)
    delta = min(dL, dR)
    
    // Build strip
    strip = []
    FOR i = lo TO hi:
        IF |P[i].x - midX| < delta:
            strip.add(P[i])
    
    Sort strip by y
    
    // Check strip pairs
    FOR i = 0 TO strip.size - 1:
        FOR j = i + 1 TO strip.size - 1:
            IF strip[j].y - strip[i].y ≥ delta: BREAK
            delta = min(delta, dist(strip[i], strip[j]))
    
    RETURN delta

Time:  T(n) = 2T(n/2) + O(n log n) = O(n log² n)
       (O(n log n) if strip is maintained sorted via merge)
```

---

## 4.4 Optimized O(n log n) Version

```
  To avoid sorting the strip at each level:
  
  Maintain a second copy of points sorted by y.
  At each recursion level, split this y-sorted list in O(n).
  
  T(n) = 2T(n/2) + O(n) = O(n log n) ★
  
  Alternatively: use merge sort naturally —
  sort by y during the merge step of the recursion.
```

---

## 4.5 Trace Example

```
  Points: (2,3) (12,30) (40,50) (5,1) (12,10) (3,4)
  
  Sorted by x: (2,3) (3,4) (5,1) (12,10) (12,30) (40,50)
  
  Divide at mid = (5,1) | (12,10)
  
  Left: {(2,3),(3,4),(5,1)}
    Brute force: distances = {√2, √13, √10}
    dL = √2 ≈ 1.414
  
  Right: {(12,10),(12,30),(40,50)}
    Brute force: distances = {20, √1384, √1000}
    dR = 20
  
  δ = min(1.414, 20) = 1.414
  midX = 5
  
  Strip (|x - 5| < 1.414):
    (5,1), (3,4) — only these are within δ of midX
  
  Distance (5,1)-(3,4) = √(4+9) = √13 ≈ 3.6 > δ
  
  Final answer: √2 from (2,3)-(3,4) ✓
```

---

## 4.6 Randomized Approach — O(n) Expected

```
  Using a grid-based approach:
  
  1. Choose random permutation of points
  2. Maintain a grid with cell size δ/2
  3. Insert points one by one
  4. When inserting point P, check 5×5 neighborhood of cells
  5. If closer pair found, rebuild grid with smaller δ
  
  Expected time: O(n) — because rebuilds are rare
  
  ★ This is more complex to implement but theoretically optimal.
```

---

## Summary Table

| Algorithm | Time | Space | Notes |
|-----------|------|-------|-------|
| Brute force | O(n²) | O(1) | Simple, n ≤ 5000 |
| Divide & Conquer | O(n log n) | O(n) | Standard approach ★ |
| D&C (naive strip sort) | O(n log² n) | O(n) | Easier to code |
| Randomized grid | O(n) expected | O(n) | Complex implementation |

---

## Quick Revision Questions

1. **Why** can we limit strip checks to 7 neighbors per point?
2. **What is** the recurrence for the divide-and-conquer approach?
3. **How does** the optimized version achieve O(n log n) vs O(n log² n)?
4. **Trace** the algorithm on points (0,0), (1,1), (3,0), (4,1), (2,3).
5. **Can** this problem be solved with sweep line? How?
6. **What changes** for the 3D closest pair problem?

---

[← Previous: Point in Polygon](03-point-in-polygon.md) | [Back to README](../README.md) | [Next: Sweep Line Algorithm →](05-sweep-line-algorithm.md)
