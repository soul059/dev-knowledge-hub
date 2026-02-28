# Chapter 2: Distance Formulas

[← Previous: Point Representation](01-point-representation.md) | [Back to README](../README.md) | [Next: Line Equations →](03-line-equations.md)

---

## Overview

Distance computation is fundamental to geometry problems. This chapter covers Euclidean, Manhattan, Chebyshev distances, conversions between them, and the critical point-to-line distance.

---

## 2.1 Euclidean Distance

```
  d(P₁, P₂) = √((x₂-x₁)² + (y₂-y₁)²)
  
  3D: d = √((x₂-x₁)² + (y₂-y₁)² + (z₂-z₁)²)
  
  ★ Prefer SQUARED distance to avoid √ and floating point:
  d² = (x₂-x₁)² + (y₂-y₁)²
  
  Use d² for comparisons. Only take √ when outputting distance.
  
  Example: A=(1,2), B=(4,6)
  d² = 9 + 16 = 25
  d = 5
```

---

## 2.2 Manhattan Distance

```
  d_M(P₁, P₂) = |x₂-x₁| + |y₂-y₁|
  
  "Taxicab distance" — travel along grid lines only.
  
  ┌──────────────────────────────────────────┐
  │     B(4,4)                               │
  │     │                                     │
  │     │← 2 →│                              │
  │     ·─────C(6,4)                         │
  │     ↑                                     │
  │   3 ↑  Euclidean: 3.6                    │
  │     ↑  Manhattan: 5  (|6-4| + |4-1| = 5)│
  │     A(4,1)                               │
  └──────────────────────────────────────────┘
  
  Manhattan distance "diamond" (locus of equal distance):
  
       ·
      · ·
     ·   ·     All points at Manhattan distance r
      · ·      from center form a diamond (rotated square)
       ·
```

---

## 2.3 Chebyshev Distance

```
  d_C(P₁, P₂) = max(|x₂-x₁|, |y₂-y₁|)
  
  "King's move distance" — a chess king can move diagonally.
  
  Locus of equal Chebyshev distance: axis-aligned square.
  
       ·····
       ·   ·
       · C ·     All points at Chebyshev distance r
       ·   ·     from center form a square
       ·····
```

---

## 2.4 Manhattan ↔ Chebyshev Conversion

```
  ★ KEY TRICK: Rotate coordinates by 45°!
  
  Transform: (x, y) → (x+y, x-y)
  
  After this transform:
  Manhattan distance = Chebyshev distance in new coordinates!
  
  Why useful?
  "Find min Manhattan distance" → transform → "Find min Chebyshev distance"
  Chebyshev is just max of two 1D problems, which is easier!
  
  Example: A=(1,3), B=(4,1)
  Manhattan: |4-1| + |1-3| = 5
  
  Transform: A'=(1+3, 1-3)=(4,-2), B'=(4+1, 4-1)=(5,3)
  Chebyshev: max(|5-4|, |3-(-2)|) = max(1, 5) = 5 ✓
```

---

## 2.5 Point-to-Line Distance

```
  Line: ax + by + c = 0
  Point: P = (x₀, y₀)
  
  Distance = |a×x₀ + b×y₀ + c| / √(a² + b²)
  
  Example: Line: 3x + 4y - 5 = 0, Point: (1, 2)
  d = |3(1) + 4(2) - 5| / √(9 + 16) = |3+8-5| / 5 = 6/5 = 1.2
```

---

## 2.6 Point-to-Segment Distance

```
  Segment AB, point P.
  
  Cases:
  1. Foot of perpendicular falls ON segment → perpendicular distance
  2. Otherwise → min(dist(P,A), dist(P,B))
  
FUNCTION pointToSegment(P, A, B):
    AB = B - A
    AP = P - A
    t = dot(AP, AB) / dot(AB, AB)    // projection parameter
    t = clamp(t, 0, 1)               // restrict to segment
    closest = A + t × AB
    RETURN dist(P, closest)
    
  ┌──────────────────────────────────────────────┐
  │                P                             │
  │               /|                             │
  │              / |  ← perpendicular            │
  │   A─────────F──┼───B                         │
  │             ↑                                │
  │           foot of perpendicular              │
  │                                              │
  │  t < 0 → closest is A                       │
  │  t > 1 → closest is B                       │
  │  0 ≤ t ≤ 1 → closest is on segment          │
  └──────────────────────────────────────────────┘
```

---

## 2.7 Distance Between Two Parallel Lines

```
  Line 1: ax + by + c₁ = 0
  Line 2: ax + by + c₂ = 0
  
  Distance = |c₁ - c₂| / √(a² + b²)
  
  ★ Lines must be in same form (same a, b coefficients)!
```

---

## 2.8 Squared Distance for Comparisons

```
  ⚠️ NEVER use sqrt for comparisons!
  
  // BAD:
  if sqrt(dx*dx + dy*dy) < r: ...
  
  // GOOD:
  if dx*dx + dy*dy < r*r: ...
  
  Benefits:
  1. Faster (no sqrt)
  2. Exact (no floating point error)
  3. Works with integer coordinates
```

---

## Summary Table

| Distance | Formula | Locus Shape |
|----------|---------|-------------|
| Euclidean | √(Δx²+Δy²) | Circle |
| Manhattan | \|Δx\|+\|Δy\| | Diamond (45° square) |
| Chebyshev | max(\|Δx\|,\|Δy\|) | Axis-aligned square |
| Point to line | \|ax₀+by₀+c\|/√(a²+b²) | — |
| Point to segment | Uses projection parameter t | — |

---

## Quick Revision Questions

1. **Why** should you compare squared distances instead of actual distances?
2. **Compute** Manhattan and Euclidean distance between (3,7) and (6,3).
3. **Transform** points (2,5) and (7,1) to convert Manhattan to Chebyshev.
4. **Find** the distance from point (0,0) to line 5x + 12y - 26 = 0.
5. **When does** point-to-segment distance differ from point-to-line distance?
6. **What shape** is the locus of constant Chebyshev distance?

---

[← Previous: Point Representation](01-point-representation.md) | [Back to README](../README.md) | [Next: Line Equations →](03-line-equations.md)
