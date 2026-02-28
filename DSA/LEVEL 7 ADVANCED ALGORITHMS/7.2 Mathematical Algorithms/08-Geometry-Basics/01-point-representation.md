# Chapter 1: Point Representation

[← Previous Unit: Probabilistic Data Structures](../07-Probability-Basics/06-probabilistic-data-structures.md) | [Back to README](../README.md) | [Next: Distance Formulas →](02-distance-formulas.md)

---

## Overview

Points are the fundamental building blocks of computational geometry. This chapter covers representations, coordinate systems, and basic operations on points in 2D and 3D.

---

## 1.1 2D Point Representation

```
  A point P in 2D: P = (x, y)
  
  ┌─────────────────────────────────────────────┐
  │        y                                     │
  │        │                                     │
  │    4   │       ● P(3, 4)                    │
  │        │      /                              │
  │    3   │     /                               │
  │        │    /  distance from origin = 5     │
  │    2   │   /                                 │
  │        │  /                                  │
  │    1   │ /                                   │
  │        │/                                    │
  │    ────O────────────── x                     │
  │        │    1  2  3  4                       │
  └─────────────────────────────────────────────┘

  Data structure:
  
  struct Point:
      x: integer or double
      y: integer or double
  
  ★ In competitive programming, prefer INTEGER coordinates
    to avoid floating-point errors!
```

---

## 1.2 Vector Operations

```
  Vector AB = B - A = (Bx - Ax, By - Ay)
  
  ┌──────────────────────────────────────────────────────────┐
  │  Operation        │  Formula                             │
  ├───────────────────┼──────────────────────────────────────┤
  │  Addition         │  (x₁+x₂, y₁+y₂)                    │
  │  Subtraction      │  (x₁-x₂, y₁-y₂)                    │
  │  Scalar multiply  │  (k×x, k×y)                         │
  │  Dot product      │  x₁×x₂ + y₁×y₂                     │
  │  Cross product    │  x₁×y₂ - y₁×x₂                     │
  │  Magnitude        │  √(x² + y²)                         │
  │  Normalize        │  (x/|v|, y/|v|)                     │
  └───────────────────┴──────────────────────────────────────┘
```

---

## 1.3 Cross Product (2D)

```
  cross(A, B) = Ax × By - Ay × Bx

  Geometric meaning: signed area of parallelogram formed by A and B.
  
  ┌──────────────────────────────────────────────────────────┐
  │  cross > 0: B is COUNTER-CLOCKWISE from A               │
  │  cross = 0: A and B are COLLINEAR (parallel)            │
  │  cross < 0: B is CLOCKWISE from A                       │
  └──────────────────────────────────────────────────────────┘

  ★ THE MOST IMPORTANT operation in computational geometry!
  
  Application: Orientation of 3 points P, Q, R
  cross(Q-P, R-P) = (Qx-Px)(Ry-Py) - (Qy-Py)(Rx-Px)
  
  > 0 → left turn (CCW)
  = 0 → collinear
  < 0 → right turn (CW)
  
         R                R
         ↗               ↘
    P → Q            P → Q
    (CCW, +)          (CW, -)
```

---

## 1.4 Dot Product

```
  dot(A, B) = Ax×Bx + Ay×By = |A|×|B|×cos(θ)
  
  ┌──────────────────────────────────────────────────────────┐
  │  dot > 0: angle < 90° (acute)                          │
  │  dot = 0: angle = 90° (perpendicular)                  │
  │  dot < 0: angle > 90° (obtuse)                         │
  └──────────────────────────────────────────────────────────┘
  
  Applications:
  • Check perpendicularity: dot == 0
  • Projection of A onto B: (dot(A,B) / |B|²) × B
  • Angle between vectors: θ = arccos(dot / (|A|×|B|))
```

---

## 1.5 Orientation Test (Pseudocode)

```
FUNCTION orientation(P, Q, R):
    val = (Q.x - P.x) × (R.y - P.y) - (Q.y - P.y) × (R.x - P.x)
    IF val > 0: RETURN "CCW"         // left turn
    IF val < 0: RETURN "CW"          // right turn
    RETURN "COLLINEAR"

  ★ Use long long in C++ to avoid overflow with integer coordinates!
  ★ For double coordinates, compare with EPS instead of 0.
```

### Trace

```
  P=(0,0), Q=(4,0), R=(2,3)
  val = (4-0)×(3-0) - (0-0)×(2-0) = 12 - 0 = 12 > 0
  → CCW (R is to the left of line PQ) ✓
  
  P=(0,0), Q=(4,0), R=(2,-3)
  val = 4×(-3) - 0×2 = -12 < 0
  → CW (R is to the right of line PQ) ✓
  
  P=(0,0), Q=(2,2), R=(4,4)
  val = 2×4 - 2×4 = 0
  → COLLINEAR ✓
```

---

## 1.6 Polar Coordinates

```
  Cartesian (x, y) ↔ Polar (r, θ)
  
  To polar:
    r = √(x² + y²)
    θ = atan2(y, x)     ← use atan2, NOT atan!
  
  To Cartesian:
    x = r × cos(θ)
    y = r × sin(θ)
  
  ★ atan2(y, x) returns angle in (-π, π]
  ★ atan(y/x) loses quadrant information!
  
  Useful for: angular sorting, sweep line on angles
```

---

## 1.7 Coordinate Compression

```
  When coordinates can be up to 10⁹ but only n ≤ 10⁵ points:
  
  1. Collect all x-coordinates
  2. Sort and deduplicate
  3. Replace each x with its rank (index in sorted array)
  
  Before: x values = [10⁹, 5, 10⁸, 5]
  Sorted unique:     [5, 10⁸, 10⁹]
  After:             [2, 0, 1, 0]   ← rank indices
  
  Now array indices fit in O(n) range.
```

---

## 1.8 Floating Point Pitfalls

```
  ┌──────────────────────────────────────────────────────────┐
  │  Rule                        │  Why                      │
  ├──────────────────────────────┼───────────────────────────┤
  │  Use integers when possible  │  No precision loss         │
  │  Compare with EPS (10⁻⁹)    │  Float equality unreliable│
  │  Use atan2, not atan         │  Correct quadrant          │
  │  Avoid √ when possible       │  Compare squared distances│
  │  Use long long for cross     │  Products overflow int     │
  └──────────────────────────────┴───────────────────────────┘
  
  Instead of: dist(A,B) < dist(C,D)
  Use:        dist²(A,B) < dist²(C,D)   (no sqrt needed!)
```

---

## Summary Table

| Operation | Formula | Use |
|-----------|---------|-----|
| Cross product | x₁y₂ - y₁x₂ | Orientation, area |
| Dot product | x₁x₂ + y₁y₂ | Angle, projection |
| Orientation | sign of cross(Q-P, R-P) | CCW/CW/collinear |
| Magnitude | √(x²+y²) | Distance from origin |
| Polar angle | atan2(y, x) | Angular sorting |

---

## Quick Revision Questions

1. **What does** a positive cross product indicate geometrically?
2. **Compute** the orientation of P=(1,1), Q=(4,2), R=(3,5).
3. **Why is** atan2 preferred over atan for polar angle computation?
4. **When should** you use squared distances instead of actual distances?
5. **What is** coordinate compression and when is it useful?
6. **Can the** cross product of two 2D vectors be a vector? Explain.

---

[← Previous Unit: Probabilistic Data Structures](../07-Probability-Basics/06-probabilistic-data-structures.md) | [Back to README](../README.md) | [Next: Distance Formulas →](02-distance-formulas.md)
