# Chapter 3: Line Equations

[← Previous: Distance Formulas](02-distance-formulas.md) | [Back to README](../README.md) | [Next: Slope Calculations →](04-slope-calculations.md)

---

## Overview

Lines are central to geometric algorithms. This chapter covers all standard representations, conversions between them, and how to handle edge cases like vertical lines and precision issues.

---

## 3.1 Forms of Line Equation

```
  ┌──────────────────────────────────────────────────────────┐
  │  Form              │  Equation            │  Notes       │
  ├────────────────────┼──────────────────────┼──────────────┤
  │  Slope-intercept   │  y = mx + b          │  No vertical │
  │  General form      │  ax + by + c = 0     │  Universal   │
  │  Two-point form    │  (y-y₁)/(x-x₁) =    │  With slope  │
  │                    │  (y₂-y₁)/(x₂-x₁)    │              │
  │  Parametric        │  P = A + t(B-A)      │  Segments!   │
  │  Normal form       │  x cos α + y sin α = d│ Distance    │
  └────────────────────┴──────────────────────┴──────────────┘
  
  ★ For competitive programming, use GENERAL FORM ax+by+c=0
    It handles vertical lines and avoids division!
```

---

## 3.2 General Form from Two Points

```
  Given P₁=(x₁,y₁), P₂=(x₂,y₂):
  
  a = y₂ - y₁
  b = x₁ - x₂
  c = x₂×y₁ - x₁×y₂
  
  Equation: a×x + b×y + c = 0
  
  Example: P₁=(1,2), P₂=(3,8)
  a = 8 - 2 = 6
  b = 1 - 3 = -2
  c = 3×2 - 1×8 = -2
  Line: 6x - 2y - 2 = 0  →  3x - y - 1 = 0
  
  Check: P₁: 3(1) - 2 - 1 = 0 ✓
         P₂: 3(3) - 8 - 1 = 0 ✓
```

---

## 3.3 Parametric Form

```
  Line through A and B:
  P(t) = A + t × (B - A) = (1-t)A + tB
  
  ┌──────────────────────────────────────────────────────────┐
  │  t = 0   →  P = A                                       │
  │  t = 1   →  P = B                                       │
  │  0 < t < 1 → Point between A and B (ON segment)        │
  │  t < 0   →  Beyond A                                    │
  │  t > 1   →  Beyond B                                    │
  └──────────────────────────────────────────────────────────┘
  
  ★ Very useful for:
  • Line segment operations (restrict t ∈ [0,1])
  • Finding intersection points
  • Interpolation
```

---

## 3.4 Collinearity Test

```
  Three points P, Q, R are collinear iff:
  cross(Q-P, R-P) = 0

FUNCTION collinear(P, Q, R):
    RETURN (Q.x-P.x)×(R.y-P.y) - (Q.y-P.y)×(R.x-P.x) == 0

  Example: P=(1,1), Q=(3,3), R=(5,5)
  cross = (3-1)×(5-1) - (3-1)×(5-1) = 8 - 8 = 0 → collinear ✓
  
  Example: P=(1,1), Q=(3,3), R=(5,6)
  cross = 2×5 - 2×4 = 10 - 8 = 2 ≠ 0 → NOT collinear ✓
```

---

## 3.5 Line Through Point with Given Slope

```
  Point (x₀, y₀), slope m:
  y - y₀ = m(x - x₀)
  
  General form: m×x - y + (y₀ - m×x₀) = 0
  Equivalently: a = m, b = -1, c = y₀ - m×x₀
  
  ★ For integer-only: store slope as fraction (dy, dx) in lowest terms.
  
FUNCTION canonicalSlope(dy, dx):
    IF dx < 0: dy = -dy; dx = -dx    // normalize sign
    IF dx == 0: dy = 1                 // vertical line
    IF dy == 0: dx = 1                 // horizontal line
    g = gcd(abs(dy), abs(dx))
    RETURN (dy/g, dx/g)
```

---

## 3.6 Parallel and Perpendicular Lines

```
  Line 1: a₁x + b₁y + c₁ = 0  (direction vector: (-b₁, a₁))
  Line 2: a₂x + b₂y + c₂ = 0  (direction vector: (-b₂, a₂))
  
  Parallel:       a₁×b₂ - a₂×b₁ == 0  (cross product of normals = 0)
  Perpendicular:  a₁×a₂ + b₁×b₂ == 0  (dot product of normals = 0)
  
  If slopes exist: m₁ = -a₁/b₁
  Parallel:       m₁ == m₂
  Perpendicular:  m₁ × m₂ == -1
  
  Example: y = 2x + 1  and  y = 2x - 3  → parallel (same slope 2)
           y = 2x + 1  and  y = -½x + 4  → perpendicular (2×-½ = -1)
```

---

## 3.7 Representing Lines in CP

```
  ★ Best practice for competitive programming:
  
  Store lines as (a, b, c) in CANONICAL form:
  1. gcd(|a|, |b|, |c|) = 1   (divide out common factors)
  2. a > 0, or a==0 and b > 0  (normalize sign)
  
  This ensures each line has a UNIQUE representation,
  useful for hashing and comparison.

FUNCTION canonicalizeLine(a, b, c):
    g = gcd(gcd(abs(a), abs(b)), abs(c))
    IF g > 0: a/=g; b/=g; c/=g
    IF a < 0 OR (a == 0 AND b < 0):
        a = -a; b = -b; c = -c
    RETURN (a, b, c)
```

---

## Summary Table

| Form | Equation | Handles Vertical |
|------|----------|-----------------|
| Slope-intercept | y = mx + b | No |
| General | ax + by + c = 0 | Yes ★ |
| Parametric | P = A + t(B-A) | Yes |
| Two-point | Cross product form | Yes |

| Test | Condition |
|------|-----------|
| Collinearity | cross(Q-P, R-P) = 0 |
| Parallel | a₁b₂ = a₂b₁ |
| Perpendicular | a₁a₂ + b₁b₂ = 0 |

---

## Quick Revision Questions

1. **Find** the general form of the line through (2,3) and (5,7).
2. **Why is** the general form preferred over slope-intercept in CP?
3. **Test** whether (1,1), (4,2), (7,3) are collinear.
4. **What is** the parametric equation of segment from (0,0) to (6,8)?
5. **Find** a line perpendicular to 3x + 4y = 12 passing through (1,1).
6. **How do** you canonicalize line representation for hashing?

---

[← Previous: Distance Formulas](02-distance-formulas.md) | [Back to README](../README.md) | [Next: Slope Calculations →](04-slope-calculations.md)
