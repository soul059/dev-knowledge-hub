# Chapter 5: 3D Extension & Generalizations

## Overview

The Closest Pair algorithm generalizes naturally from 2D to 3D and higher dimensions. This chapter covers the extension to 3D, the challenges of higher dimensions, and related geometric problems that build on the Closest Pair framework.

---

## 3D Closest Pair: Concept

```
In 3D, each point has three coordinates: p = (x, y, z)

Distance: dist(p, q) = √((p.x-q.x)² + (p.y-q.y)² + (p.z-q.z)²)

The D&C approach extends:
  1. DIVIDE by a plane (e.g., x = median)  instead of a line
  2. CONQUER each half recursively
  3. COMBINE by checking a SLAB of width 2δ  instead of a strip
```

---

## 2D vs 3D: Visual Comparison

```
2D: Divide by a LINE          3D: Divide by a PLANE
    ┌──────┬──────┐               ┌──────────┐
    │  L   │  R   │              /  L    /  R /
    │  •   │  •   │             /  •    /  • /
    │ •    │   •  │            / •     /   •/
    │   •  │  •   │           /   •   /  • /
    └──────┴──────┘          └──────────┘

    Strip: 2D band             Slab: 3D band
    Width: 2δ                  Thickness: 2δ
```

---

## 3D Algorithm

```
CLOSEST_PAIR_3D(P):
    Sort P by x → Px
    Sort P by y → Py
    Sort P by z → Pz
    return CLOSEST_3D_REC(Px, Py, Pz)

CLOSEST_3D_REC(Px, Py, Pz):
    if |Px| ≤ some constant c:
        return BRUTE_FORCE(Px)
    
    // DIVIDE by median x-plane
    mid ← |Px| / 2
    midX ← Px[mid].x
    
    // Split Py and Pz by midX (maintain sort orders)
    Split into left/right maintaining y and z orders
    
    // CONQUER
    δL ← CLOSEST_3D_REC(left Px, left Py, left Pz)
    δR ← CLOSEST_3D_REC(right Px, right Py, right Pz)
    δ ← min(δL, δR)
    
    // COMBINE — check the slab
    slab ← {p ∈ Py : |p.x - midX| < δ}
    
    // In 3D: check slab column by column
    CHECK_SLAB_3D(slab, δ)
```

---

## Slab Analysis in 3D

```
In 2D:
  δ × 2δ rectangle → at most 8 points → 7 comparisons
  
In 3D:
  δ × 2δ × δ box → at most O(1) points per box
  
  Divide slab into cells of size δ/2 × δ/2 × δ/2:
  
  ┌───────────────────────┐
  │  3D Box: 2δ × δ × δ   │
  │                        │
  │  Cells per dimension:  │
  │    x: 2δ/(δ/2) = 4    │
  │    y: δ/(δ/2)  = 2    │
  │    z: δ/(δ/2)  = 2    │
  │                        │
  │  Total cells: 4×2×2 = 16│
  │                        │
  │  Max 1 point per cell  │
  │  → at most 16 points   │
  │  → 15 comparisons      │
  └───────────────────────┘

Still O(1) comparisons per point!
```

---

## Generalizing to d Dimensions

```
In d-dimensional space:

  Point: p = (x₁, x₂, ..., x_d)
  Distance: Euclidean L₂ norm

  D&C approach:
    1. Divide by hyperplane through median of first coordinate
    2. Conquer recursively
    3. Combine: check (d-1)-dimensional slab of width 2δ

  Box size: 2δ × δ^(d-1)
  Cell size: (δ/2)^d
  Number of cells: 4 × 2^(d-1) = 2^(d+1)
  Max points per box: 2^(d+1) = O(2^d)

  Comparisons per point: O(2^d)

  ┌──────────┬───────────────┬──────────────────────┐
  │ Dimension│ Cells in box  │ Comparisons per point│
  ├──────────┼───────────────┼──────────────────────┤
  │ 1D       │ 4             │ ≤ 3                  │
  │ 2D       │ 8             │ ≤ 7                  │
  │ 3D       │ 16            │ ≤ 15                 │
  │ dD       │ 2^(d+1)       │ ≤ 2^(d+1) - 1       │
  └──────────┴───────────────┴──────────────────────┘
```

---

## Complexity in d Dimensions

```
Time: T(n) = 2T(n/2) + O(2^d · n)
            = O(2^d · n log n)

For fixed d: O(n log n) (constant depends on d)

┌────────────────────────────────────────────┐
│ The "curse of dimensionality":             │
│                                            │
│ As d grows, the constant factor 2^d grows  │
│ exponentially! For d = 20:                 │
│   2^d ≈ 1,000,000                          │
│                                            │
│ Alternatives for high-d:                   │
│   - Approximate nearest neighbor (ANN)     │
│   - Locality-Sensitive Hashing (LSH)       │
│   - k-d trees (practical for d ≤ ~20)      │
│   - Ball trees                             │
└────────────────────────────────────────────┘
```

---

## Related Geometric Problems

### 1. All Nearest Neighbors

```
Problem: For EACH point, find its nearest neighbor.

Brute force: O(n²)
D&C: O(n log n) — similar to closest pair

Key idea: closest pair is a special case
(find the minimum over all nearest-neighbor distances)
```

### 2. Farthest Pair (Diameter)

```
Problem: Find the two points with MAXIMUM distance.

      •
         •  ← farthest from some •
    •
                       •

Solution: Compute convex hull first → O(n log n)
Then use rotating calipers: O(n)
Total: O(n log n)
```

### 3. Bichromatic Closest Pair

```
Problem: Given RED points and BLUE points,
         find the closest red-blue pair.

   R •      • B
      • R        • B
         • R   • B

D&C approach: O(n log n)
(Only consider cross-color pairs in combine step)
```

---

## Python Implementation (Complete)

```python
import math

def closest_pair(points):
    """Find closest pair of 2D points in O(n log n)."""
    n = len(points)
    if n < 2:
        return float('inf'), None, None
    
    # Pre-sort
    Px = sorted(points, key=lambda p: (p[0], p[1]))
    Py = sorted(points, key=lambda p: (p[1], p[0]))
    
    return _closest_rec(Px, Py)

def _closest_rec(Px, Py):
    n = len(Px)
    
    # Base case
    if n <= 3:
        return _brute_force(Px)
    
    # Divide
    mid = n // 2
    mid_x = Px[mid][0]
    
    # Split Py into left and right (maintaining y-order)
    left_set = set(map(tuple, Px[:mid]))
    Pyl = [p for p in Py if tuple(p) in left_set]
    Pyr = [p for p in Py if tuple(p) not in left_set]
    
    # Conquer
    d_left, p1l, p2l = _closest_rec(Px[:mid], Pyl)
    d_right, p1r, p2r = _closest_rec(Px[mid:], Pyr)
    
    if d_left <= d_right:
        delta, best_p, best_q = d_left, p1l, p2l
    else:
        delta, best_p, best_q = d_right, p1r, p2r
    
    # Combine — build strip from Py (y-sorted)
    strip = [p for p in Py if abs(p[0] - mid_x) < delta]
    
    # Check strip
    for i in range(len(strip)):
        j = i + 1
        while j < len(strip) and strip[j][1] - strip[i][1] < delta:
            d = _dist(strip[i], strip[j])
            if d < delta:
                delta = d
                best_p, best_q = strip[i], strip[j]
            j += 1
    
    return delta, best_p, best_q

def _brute_force(points):
    min_d = float('inf')
    best_p = best_q = None
    for i in range(len(points)):
        for j in range(i+1, len(points)):
            d = _dist(points[i], points[j])
            if d < min_d:
                min_d = d
                best_p, best_q = points[i], points[j]
    return min_d, best_p, best_q

def _dist(p, q):
    return math.sqrt((p[0]-q[0])**2 + (p[1]-q[1])**2)

# Example usage
points = [(2,3), (12,30), (40,50), (5,1), (12,10), (3,4)]
dist, p, q = closest_pair(points)
print(f"Closest pair: {p}, {q} with distance {dist:.4f}")
# Output: Closest pair: (2, 3), (3, 4) with distance 1.4142
```

---

## Practice Problems

```
┌────┬────────────────────────────────┬────────────┐
│ #  │ Problem                        │ Difficulty │
├────┼────────────────────────────────┼────────────┤
│ 1  │ Closest Pair (basic)           │ Medium     │
│ 2  │ Closest Pair with Manhattan    │ Medium     │
│    │ distance                       │            │
│ 3  │ K-nearest points to origin     │ Medium     │
│ 4  │ Min cost to connect points     │ Hard       │
│ 5  │ Farthest pair (diameter)       │ Hard       │
│ 6  │ Bichromatic closest pair       │ Hard       │
└────┴────────────────────────────────┴────────────┘
```

---

## Summary Table

| Topic | Key Point |
|-------|-----------|
| 3D extension | Divide by plane, slab instead of strip |
| 3D comparisons | ≤ 15 per point (16 cells) |
| d-dimensional | O(2^d · n log n), exponential in d |
| Curse of dim. | For high d, use ANN/LSH instead |
| All nearest neighbors | O(n log n) via similar D&C |
| Farthest pair | Convex hull + rotating calipers O(n log n) |
| Optimality | Θ(n log n) for fixed d |

---

## Quick Revision Questions

1. **How does the dividing structure change from 2D to 3D?**
2. **How many cells are in the 3D slab box?**
3. **Why does the algorithm become impractical for very high dimensions?**
4. **What is the bichromatic closest pair problem?**
5. **What data structures are better for high-dimensional nearest neighbor?**

---

[← Previous: Complexity Analysis](04-complexity-analysis.md) | [Back to README](../README.md)
