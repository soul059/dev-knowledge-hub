# Chapter 4: Slope Calculations

[← Previous: Line Equations](03-line-equations.md) | [Back to README](../README.md) | [Next: Intersection Points →](05-intersection-points.md)

---

## Overview

Slope measures a line's steepness. Handling slope correctly — especially vertical lines, precision, and canonical representation — is critical for geometric algorithms.

---

## 4.1 Basic Slope Definition

```
         y₂ - y₁       Δy       rise
  m = ─────────── = ────── = ──────
         x₂ - x₁       Δx       run
  
  ┌──────────────────────────────────────────────────────────┐
  │  m > 0: line goes UP to the right        /              │
  │  m < 0: line goes DOWN to the right      \              │
  │  m = 0: horizontal line                  ──             │
  │  m = ±∞: vertical line (undefined!)      |              │
  └──────────────────────────────────────────────────────────┘
  
  ⚠️ NEVER divide when dx could be 0!
  Instead, store slope as (dy, dx) pair.
```

---

## 4.2 Integer Slope Representation

```
  ★ Golden rule: Store slope as reduced fraction (dy, dx)
  
FUNCTION slope(P1, P2):
    dy = P2.y - P1.y
    dx = P2.x - P1.x
    
    IF dx == 0: RETURN (1, 0)       // vertical — use sentinel
    IF dy == 0: RETURN (0, 1)       // horizontal
    
    g = gcd(abs(dy), abs(dx))
    dy /= g
    dx /= g
    
    IF dx < 0:                      // normalize: dx always positive
        dy = -dy
        dx = -dx
    
    RETURN (dy, dx)
  
  This gives a UNIQUE representation for each distinct slope.
  Can be used as hash map key!
  
  Examples:
  (1,2)→(3,6): (dy,dx) = (4,4) → gcd=4 → (1,1)
  (1,2)→(5,4): (dy,dx) = (2,2) → gcd=2 → (1,1)  ← same slope!
  (0,0)→(0,5): (dy,dx) = (5,0) → vertical → (1,0)
```

---

## 4.3 Max Points on a Line (Classic Problem)

```
  Given n points, find max number on any single line.
  
  For each point P:
    Create map: slope → count
    For each other point Q:
      Compute canonical slope of PQ
      Increment map[slope]
    Track duplicates of P separately
    Answer = max(counts) + duplicates + 1
  
  Time: O(n²)   Space: O(n)
  
  Example: Points = [(1,1), (2,2), (3,3), (4,1), (5,2)]
  
  From (1,1):
    to (2,2): slope (1,1) → count=1
    to (3,3): slope (1,1) → count=2
    to (4,1): slope (0,1) → count=1
    to (5,2): slope (1,4) → count=1
  
  Max from (1,1): 2 + 1(itself) = 3 points on slope (1,1)
```

---

## 4.4 Angle from Slope

```
  θ = atan2(dy, dx)
  
  Returns angle in radians ∈ (-π, π]
  
  ┌──────────────────────────────────────────────┐
  │   Quadrant II  │  Quadrant I                 │
  │   (π/2, π)     │  (0, π/2)                  │
  │                 │                             │
  │   ─────────────O──────────── → x             │
  │                 │                             │
  │   Quadrant III │  Quadrant IV                │
  │   (-π, -π/2)  │  (-π/2, 0)                  │
  └──────────────────────────────────────────────┘
  
  ★ atan2 handles all quadrants correctly, including vertical!
  ★ Use atan2 for angular sorting in sweep line algorithms.
```

---

## 4.5 Angle Between Two Lines

```
  Lines with slopes m₁ and m₂:
  
  tan(θ) = |m₁ - m₂| / (1 + m₁×m₂)
  
  (assuming 1 + m₁×m₂ ≠ 0, otherwise lines are perpendicular)
  
  Using direction vectors (dx₁,dy₁) and (dx₂,dy₂):
  cos(θ) = |dot| / (|v₁| × |v₂|)
  
  Where dot = dx₁×dx₂ + dy₁×dy₂
  
  ★ For integer-only problems, you can compare angles without
    computing atan: use cross product sign and dot product sign.
```

---

## 4.6 Sorting by Slope (Polar Angle Sort)

```
  Sort points by angle from a reference point O.
  
  Comparator using cross product (no floating point!):
  
FUNCTION compareAngle(A, B, O):
    // Vectors from O
    va = A - O
    vb = B - O
    cross = va.x × vb.y - va.y × vb.x
    IF cross > 0: RETURN A before B     // A is CCW from B
    IF cross < 0: RETURN B before A
    // Collinear: closer point first
    RETURN dist²(O, A) < dist²(O, B)

  ⚠️ Must handle quadrants separately for full 360° sort,
  or use atan2 (with floating point).
  
  ★ Used in: Convex hull (Graham scan), sweep line
```

---

## 4.7 Common Pitfalls

```
  ┌──────────────────────────────────────────────────────────┐
  │  Pitfall                  │  Solution                    │
  ├───────────────────────────┼──────────────────────────────┤
  │  Division by zero (dx=0) │  Store as (dy,dx) pair       │
  │  Float comparison        │  Use reduced fraction         │
  │  Non-canonical slope     │  Normalize sign and GCD      │
  │  Duplicate points        │  Handle separately            │
  │  Overflow in cross prod  │  Use long long                │
  │  atan vs atan2           │  Always use atan2             │
  └───────────────────────────┴──────────────────────────────┘
```

---

## Summary Table

| Concept | Method |
|---------|--------|
| Slope | (dy, dx) as reduced fraction |
| Vertical | Sentinel (1, 0) |
| Horizontal | Sentinel (0, 1) |
| Canonical form | gcd + normalize dx > 0 |
| Angle | atan2(dy, dx) |
| Compare angles | Cross product (no float) |
| Max collinear | Hash slope per point, O(n²) |

---

## Quick Revision Questions

1. **Why** store slopes as (dy, dx) pairs instead of floating-point numbers?
2. **Canonicalize** the slope of the line through (3, 7) and (9, 1).
3. **How many** distinct slopes can n points generate?
4. **What is** the angle between lines with slopes 2 and -1/2?
5. **Explain** how to sort points by polar angle without atan2.
6. **What happens** if you use atan instead of atan2?

---

[← Previous: Line Equations](03-line-equations.md) | [Back to README](../README.md) | [Next: Intersection Points →](05-intersection-points.md)
