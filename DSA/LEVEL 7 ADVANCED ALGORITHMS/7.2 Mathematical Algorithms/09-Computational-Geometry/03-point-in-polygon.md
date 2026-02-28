# Chapter 3: Point in Polygon

[← Previous: Line Intersection](02-line-intersection.md) | [Back to README](../README.md) | [Next: Closest Pair of Points →](04-closest-pair-of-points.md)

---

## Overview

Determining whether a point lies inside, outside, or on the boundary of a polygon is a fundamental query. The **ray casting** algorithm is the simplest and most general approach.

---

## 3.1 Ray Casting Algorithm

```
  Idea: Cast a ray from point P to infinity (usually rightward).
  Count how many polygon edges the ray crosses.
  
  Odd crossings  → INSIDE
  Even crossings → OUTSIDE
  
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │      ·─────────·                                        │
  │     /           \                                       │
  │    /    P ──────────→  crosses 1 edge → INSIDE          │
  │   /               \                                     │
  │  ·                  ·                                    │
  │   \                /                                     │
  │    ·──────────────·                                      │
  │                                                          │
  │      Q ──────────────→ crosses 0 edges → OUTSIDE        │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

```
FUNCTION pointInPolygon(P, polygon[n]):
    crossings = 0
    FOR i = 0 TO n - 1:
        A = polygon[i]
        B = polygon[(i + 1) % n]
        
        // Check if ray from P rightward crosses edge AB
        IF (A.y ≤ P.y < B.y) OR (B.y ≤ P.y < A.y):
            // x-coordinate of intersection
            t = (P.y - A.y) / (B.y - A.y)
            x_intersect = A.x + t × (B.x - A.x)
            IF P.x < x_intersect:
                crossings += 1
    
    IF crossings is odd: RETURN "INSIDE"
    ELSE: RETURN "OUTSIDE"

Time:  O(n) per query
Space: O(1)
```

### Trace

```
  Polygon: (0,0), (6,0), (6,4), (0,4) — a rectangle
  Query point: P = (3, 2)
  
  Edge (0,0)-(6,0): y range [0,0], P.y=2 → doesn't cross
  Edge (6,0)-(6,4): y range [0,4], P.y=2 ∈ [0,4)
    t = (2-0)/(4-0) = 0.5, x = 6+0.5×0 = 6
    P.x=3 < 6 → crossing! count=1
  Edge (6,4)-(0,4): y range [4,4], P.y=2 → doesn't cross
  Edge (0,4)-(0,0): y range [0,4], P.y=2 ∈ [0,4)
    t = (2-4)/(0-4) = 0.5, x = 0+0.5×0 = 0
    P.x=3 < 0? NO → no crossing
  
  crossings = 1 (odd) → INSIDE ✓
```

---

## 3.2 Point in Convex Polygon — O(log n)

```
  For CONVEX polygon with vertices in CCW order:
  
  1. Pick vertex V₀ as reference
  2. Binary search on angle: find sector where P lies
  3. Check if P is on the correct side of the bounding edge
  
FUNCTION pointInConvexPolygon(P, polygon[n]):
    // All cross products relative to polygon[0]
    IF cross(polygon[1]-polygon[0], P-polygon[0]) < 0: RETURN OUTSIDE
    IF cross(polygon[n-1]-polygon[0], P-polygon[0]) > 0: RETURN OUTSIDE
    
    // Binary search for the sector
    lo = 1, hi = n - 1
    WHILE hi - lo > 1:
        mid = (lo + hi) / 2
        IF cross(polygon[mid]-polygon[0], P-polygon[0]) ≥ 0:
            lo = mid
        ELSE:
            hi = mid
    
    // Check if P is inside triangle [0, lo, lo+1]
    RETURN cross(polygon[lo+1]-polygon[lo], P-polygon[lo]) ≥ 0

  Time: O(log n) per query after O(n) preprocessing
  
  ★ Much faster than O(n) ray casting for many queries!
```

---

## 3.3 Winding Number Algorithm

```
  Alternative to ray casting, handles all edge cases cleanly.
  
  Winding number = how many times the polygon winds around P.
  
  winding = 0 → OUTSIDE
  winding ≠ 0 → INSIDE
  
  FOR each edge (A, B) of polygon:
      IF A.y ≤ P.y:
          IF B.y > P.y:  // upward crossing
              IF cross(A, B, P) > 0:  // P left of edge
                  winding += 1
      ELSE:
          IF B.y ≤ P.y:  // downward crossing
              IF cross(A, B, P) < 0:  // P right of edge
                  winding -= 1
  
  Time: O(n)
  
  ★ More robust than ray casting for complex polygons
    (self-intersecting, with holes).
```

---

## 3.4 Point on Polygon Boundary

```
  Check if P lies on any edge of the polygon:
  
  FOR each edge (A, B):
      IF cross(A-P, B-P) == 0 AND     // collinear
         dot(A-P, B-P) ≤ 0:            // P between A and B
          RETURN "ON BOUNDARY"
  
  Or equivalently:
  IF collinear(A, B, P) AND
     min(A.x,B.x) ≤ P.x ≤ max(A.x,B.x) AND
     min(A.y,B.y) ≤ P.y ≤ max(A.y,B.y):
      RETURN "ON BOUNDARY"
```

---

## 3.5 Edge Cases in Ray Casting

```
  ┌──────────────────────────────────────────────────────────┐
  │  Problem              │  Solution                       │
  ├───────────────────────┼─────────────────────────────────┤
  │  Ray passes through   │  Use half-open interval:        │
  │  a vertex             │  (A.y ≤ P.y < B.y) to count    │
  │                       │  vertex exactly once             │
  │                                                          │
  │  Ray along an edge    │  Perturb ray direction slightly │
  │                       │  or use the y-interval rule      │
  │                                                          │
  │  Point ON boundary    │  Check separately before ray    │
  │                       │  casting                         │
  └───────────────────────┴─────────────────────────────────┘
  
  ★ The half-open interval trick [ ≤ ... < ] ensures each
    vertex is counted for exactly one of its two incident edges.
```

---

## 3.6 Decision Guide

```
  ┌──────────────────────────────────────────────────────────┐
  │  Polygon type      │  Algorithm       │  Time per query  │
  ├────────────────────┼──────────────────┼──────────────────┤
  │  Simple (general)  │  Ray casting     │  O(n)            │
  │  Simple (complex)  │  Winding number  │  O(n)            │
  │  Convex            │  Binary search   │  O(log n) ★      │
  │  With holes        │  Winding number  │  O(n)            │
  └────────────────────┴──────────────────┴──────────────────┘
```

---

## Summary Table

| Method | Time | Polygon Type | Handles Boundary |
|--------|------|-------------|-----------------|
| Ray casting | O(n) | Simple | With care |
| Winding number | O(n) | Any | Yes |
| Binary search | O(log n) | Convex only | Yes |
| Boundary check | O(n) | Any | — |

---

## Quick Revision Questions

1. **Explain** why odd crossings means inside in ray casting.
2. **Trace** ray casting for point (2,2) in triangle (0,0),(5,0),(0,5).
3. **Why is** the binary search method limited to convex polygons?
4. **How does** the half-open interval trick handle vertices?
5. **What is** the winding number and when is it nonzero?
6. **Compare** ray casting and winding number for self-intersecting polygons.

---

[← Previous: Line Intersection](02-line-intersection.md) | [Back to README](../README.md) | [Next: Closest Pair of Points →](04-closest-pair-of-points.md)
