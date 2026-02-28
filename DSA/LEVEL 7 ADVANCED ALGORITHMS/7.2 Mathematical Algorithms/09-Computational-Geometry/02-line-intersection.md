# Chapter 2: Line Intersection (Sweep Line)

[← Previous: Convex Hull](01-convex-hull.md) | [Back to README](../README.md) | [Next: Point in Polygon →](03-point-in-polygon.md)

---

## Overview

Detecting intersections among $n$ line segments is a classic computational geometry problem. The naive $O(n^2)$ approach checks all pairs; the **Bentley-Ottmann sweep line** algorithm does it in $O((n+k) \log n)$ where $k$ is the number of intersections.

---

## 2.1 Naive Approach

```
  Check every pair of segments for intersection.
  
  FOR i = 0 TO n - 1:
      FOR j = i + 1 TO n - 1:
          IF segments[i] intersects segments[j]:
              report (i, j)
  
  Time: O(n²)
  Works fine for n ≤ 1000, but too slow for n ≥ 10⁵.
```

---

## 2.2 Sweep Line Concept

```
  A vertical line sweeps from LEFT to RIGHT across the plane.
  
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │    \    /──────         Events (sorted by x):           │
  │     \  /               • Left endpoint  → INSERT        │
  │      \/                • Right endpoint → DELETE         │
  │      /\                • Intersection   → SWAP          │
  │     /  \                                                │
  │    /    \─────         At any sweep position, maintain   │
  │                        active segments sorted by y.      │
  │    ←─── sweep line                                      │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
  
  Key insight: Two segments can only intersect if they are
  ADJACENT in the vertical ordering at some point during the sweep.
```

---

## 2.3 Bentley-Ottmann Algorithm

```
  Data structures:
  • Event queue: priority queue sorted by x (then y)
  • Status structure: balanced BST of active segments, ordered by y

  Events:
  1. Left endpoint of segment s:
     - Insert s into status
     - Check s against its neighbors above and below for intersection
     
  2. Right endpoint of segment s:
     - Check if s's neighbors (above and below) intersect
     - Remove s from status
     
  3. Intersection of s₁ and s₂:
     - Report intersection
     - Swap s₁ and s₂ in status
     - Check new neighbors for intersections

Time:  O((n + k) log n)    where k = number of intersections
Space: O(n + k)
```

---

## 2.4 Simplified Problem: Any Intersection?

```
  Just detect IF any two segments intersect (yes/no).
  
  Simplified sweep (Shamos-Hoey):
  • Only left/right endpoint events (no intersection events)
  • At each insert: check neighbors
  • At each delete: check if gap neighbors now intersect
  • Stop at first intersection found
  
  Time: O(n log n)
  Space: O(n)
```

---

## 2.5 Segment Intersection Check (Review)

```
FUNCTION segmentsIntersect(A, B, C, D):
    d1 = cross(C, D, A)
    d2 = cross(C, D, B)
    d3 = cross(A, B, C)
    d4 = cross(A, B, D)
    
    IF d1 * d2 < 0 AND d3 * d4 < 0:
        RETURN TRUE
    
    // Collinear cases
    IF d1 == 0 AND onSegment(C, D, A): RETURN TRUE
    IF d2 == 0 AND onSegment(C, D, B): RETURN TRUE
    IF d3 == 0 AND onSegment(A, B, C): RETURN TRUE
    IF d4 == 0 AND onSegment(A, B, D): RETURN TRUE
    
    RETURN FALSE

  O(1) per pair check
```

---

## 2.6 Applications

```
  ┌──────────────────────────────────────────────────────────┐
  │  Problem                        │  Approach              │
  ├─────────────────────────────────┼────────────────────────┤
  │  Count intersections            │  Sweep line + BIT      │
  │  Map overlay                    │  Sweep line            │
  │  Visibility from a point        │  Angular sweep         │
  │  Simple polygon check           │  No self-intersections │
  │  Red-blue intersection          │  Modified sweep        │
  └─────────────────────────────────┴────────────────────────┘
```

---

## 2.7 Counting Inversions as Intersections

```
  Interesting connection: 
  Given n horizontal segments at different y-levels,
  connected to points on the right in a different order,
  the number of crossings = number of inversions!
  
  ┌──────────────────────────────────────────────┐
  │  Left    Right                               │
  │  1 ───── 3     1-3 and 2-1 cross!           │
  │  2 ───── 1     Count = merge sort inversions │
  │  3 ───── 2     = 3 crossings                 │
  └──────────────────────────────────────────────┘
  
  Count inversions: O(n log n) via merge sort.
```

---

## Summary Table

| Method | Time | When |
|--------|------|------|
| Brute force | O(n²) | n ≤ 1000 |
| Shamos-Hoey | O(n log n) | Any intersection? |
| Bentley-Ottmann | O((n+k) log n) | Report all intersections |
| Merge sort inversions | O(n log n) | Count crossings |

---

## Quick Revision Questions

1. **What events** does the sweep line process?
2. **Why** must we check neighbors when inserting or deleting a segment?
3. **What is** the key difference between Shamos-Hoey and Bentley-Ottmann?
4. **When would** brute-force O(n²) be acceptable?
5. **Explain** the connection between segment crossings and inversions.
6. **What data structure** is used for the sweep status?

---

[← Previous: Convex Hull](01-convex-hull.md) | [Back to README](../README.md) | [Next: Point in Polygon →](03-point-in-polygon.md)
