# Chapter 5: Intersection Points

[← Previous: Slope Calculations](04-slope-calculations.md) | [Back to README](../README.md) | [Next: Area Calculations →](06-area-calculations.md)

---

## Overview

Finding where lines, segments, and circles intersect is a core geometric operation. This chapter covers line-line intersection, segment-segment intersection, and related techniques.

---

## 5.1 Line-Line Intersection

```
  Lines in general form:
  L₁: a₁x + b₁y + c₁ = 0
  L₂: a₂x + b₂y + c₂ = 0
  
  Using Cramer's rule:
  D  = a₁b₂ - a₂b₁
  Dx = b₁c₂ - b₂c₁
  Dy = a₂c₁ - a₁c₂
  
  IF D == 0: lines are parallel (no intersection or coincident)
  ELSE: x = Dx/D,  y = Dy/D

FUNCTION lineIntersection(a1,b1,c1, a2,b2,c2):
    D = a1*b2 - a2*b1
    IF D == 0:
        // Check if same line: a1*c2 == a2*c1 and b1*c2 == b2*c1
        RETURN "parallel" or "coincident"
    x = (b1*c2 - b2*c1) / D
    y = (a2*c1 - a1*c2) / D
    RETURN (x, y)
```

### Trace

```
  L₁: 2x + 3y - 12 = 0    (a₁=2, b₁=3, c₁=-12)
  L₂: x - y - 1 = 0       (a₂=1, b₂=-1, c₂=-1)
  
  D  = 2×(-1) - 1×3 = -5
  Dx = 3×(-1) - (-1)×(-12) = -3 - 12 = -15
  Dy = 1×(-12) - 2×(-1) = -12 + 2 = -10
  
  x = -15 / -5 = 3
  y = -10 / -5 = 2
  
  Intersection: (3, 2)
  Check L₁: 2(3)+3(2)-12 = 0 ✓
  Check L₂: 3-2-1 = 0 ✓
```

---

## 5.2 Segment-Segment Intersection

```
  Segments P₁P₂ and P₃P₄. Check if they intersect.
  
  Step 1: Compute orientations
  d₁ = orientation(P₃, P₄, P₁)
  d₂ = orientation(P₃, P₄, P₂)
  d₃ = orientation(P₁, P₂, P₃)
  d₄ = orientation(P₁, P₂, P₄)
  
  Step 2: General case
  IF d₁ × d₂ < 0 AND d₃ × d₄ < 0:
      RETURN TRUE   // segments straddle each other
  
  Step 3: Collinear / touching cases
  IF d₁ == 0 AND onSegment(P₃,P₄, P₁): RETURN TRUE
  IF d₂ == 0 AND onSegment(P₃,P₄, P₂): RETURN TRUE
  IF d₃ == 0 AND onSegment(P₁,P₂, P₃): RETURN TRUE
  IF d₄ == 0 AND onSegment(P₁,P₂, P₄): RETURN TRUE
  
  RETURN FALSE

  ┌──────────────────────────────────────────────────────────┐
  │  Visualization:                                          │
  │                                                          │
  │  P₁          P₃                                          │
  │    \        /           d₁ and d₂ have opposite signs   │
  │     \      /            AND d₃ and d₄ have opposite     │
  │      \    /             signs → segments CROSS!          │
  │       \  /                                               │
  │        \/                                                │
  │        /\                                                │
  │       /  \                                               │
  │      P₄   P₂                                            │
  └──────────────────────────────────────────────────────────┘
```

### Helper: On Segment

```
FUNCTION onSegment(A, B, P):
    // Check if P lies on segment AB (given P is collinear with A, B)
    RETURN min(A.x,B.x) ≤ P.x ≤ max(A.x,B.x) AND
           min(A.y,B.y) ≤ P.y ≤ max(A.y,B.y)
```

---

## 5.3 Finding Intersection Point of Two Segments

```
  Once you know segments intersect, find the exact point:
  
  Parametric approach:
  P = P₁ + t(P₂ - P₁)
  P = P₃ + u(P₄ - P₃)
  
  Solve:
  t = cross(P₃-P₁, P₄-P₃) / cross(P₂-P₁, P₄-P₃)
  u = cross(P₃-P₁, P₂-P₁) / cross(P₂-P₁, P₄-P₃)
  
  Intersection at: P₁ + t(P₂ - P₁)
  Valid segment intersection: 0 ≤ t ≤ 1 AND 0 ≤ u ≤ 1
  
  Time: O(1)
```

---

## 5.4 Line-Circle Intersection

```
  Line: ax + by + c = 0
  Circle: center (cx, cy), radius r
  
  Distance from center to line:
  d = |a×cx + b×cy + c| / √(a² + b²)
  
  Cases:
  d > r  → no intersection
  d = r  → tangent (1 point)
  d < r  → secant (2 points)
  
  ┌──────────────────────────────────────────────────┐
  │                                                   │
  │  ──────────────    d > r: no intersection        │
  │        ○                                          │
  │                                                   │
  │  ──────────────    d = r: tangent (1 point)      │
  │        ○──●                                       │
  │                                                   │
  │    ────●──○──●──  d < r: secant (2 points)       │
  │                                                   │
  └──────────────────────────────────────────────────┘
  
  To find intersection points:
  foot = projection of center onto line
  offset = √(r² - d²) along line direction
  points = foot ± offset × (unit direction)
```

---

## 5.5 Circle-Circle Intersection

```
  C₁: center O₁, radius r₁
  C₂: center O₂, radius r₂
  d = dist(O₁, O₂)
  
  Cases:
  d > r₁+r₂        → no intersection (too far)
  d < |r₁-r₂|      → no intersection (one inside other)
  d = 0 and r₁=r₂  → infinite intersections (same circle)
  d = r₁+r₂        → external tangent (1 point)
  d = |r₁-r₂|      → internal tangent (1 point)
  otherwise         → 2 intersection points
  
  To find points: use the radical axis or trigonometric approach.
```

---

## 5.6 Complexity Summary

```
  ┌─────────────────────────────────────────┐
  │  Operation                │  Time       │
  ├───────────────────────────┼─────────────┤
  │  Line-line intersection   │  O(1)       │
  │  Segment-segment test     │  O(1)       │
  │  Line-circle intersection │  O(1)       │
  │  Any-pair among n objects │  O(n²) naive│
  │  Sweep line (segments)    │  O(n log n) │
  └───────────────────────────┴─────────────┘
```

---

## Summary Table

| Problem | Method | Key Condition |
|---------|--------|---------------|
| Line-line | Cramer's rule | D ≠ 0 |
| Segment-segment | Orientation test | Opposite orientations |
| Find intersection point | Parametric form | t, u ∈ [0,1] |
| Line-circle | Distance comparison | d vs r |
| Circle-circle | Distance comparison | d vs r₁±r₂ |

---

## Quick Revision Questions

1. **Find** the intersection of 3x - 2y = 1 and x + y = 5.
2. **Why do** segment tests need both orientation AND collinear checks?
3. **What does** D = 0 in Cramer's rule mean geometrically?
4. **How many** intersection points can a line and circle have?
5. **Compute** the parametric intersection of segments (0,0)-(4,4) and (0,4)-(4,0).
6. **When** would you use sweep line instead of brute-force pairwise checking?

---

[← Previous: Slope Calculations](04-slope-calculations.md) | [Back to README](../README.md) | [Next: Area Calculations →](06-area-calculations.md)
