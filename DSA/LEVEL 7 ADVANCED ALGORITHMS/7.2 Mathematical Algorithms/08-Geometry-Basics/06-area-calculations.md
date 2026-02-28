# Chapter 6: Area Calculations

[← Previous: Intersection Points](05-intersection-points.md) | [Back to README](../README.md) | [Next: Unit 9 - Convex Hull →](../09-Computational-Geometry/01-convex-hull.md)

---

## Overview

Area computation appears everywhere — from triangles to arbitrary polygons. The **Shoelace formula** is the key tool. This chapter covers signed area, polygon area, triangle area, and applications.

---

## 6.1 Triangle Area (Three Points)

```
  Given P₁=(x₁,y₁), P₂=(x₂,y₂), P₃=(x₃,y₃):
  
  Area = ½ |x₁(y₂-y₃) + x₂(y₃-y₁) + x₃(y₁-y₂)|
  
  Equivalently (using cross product):
  Area = ½ |cross(P₂-P₁, P₃-P₁)|
       = ½ |(x₂-x₁)(y₃-y₁) - (y₂-y₁)(x₃-x₁)|
  
  The SIGNED area (without absolute value):
  Positive → CCW orientation
  Negative → CW orientation
  Zero     → collinear

  Example: P₁=(0,0), P₂=(4,0), P₃=(2,3)
  Area = ½ |0(0-3) + 4(3-0) + 2(0-0)|
       = ½ |0 + 12 + 0| = 6

  ★ Keep 2×Area as integer to avoid fractions!
  ★ If 2×Area is odd, the triangle has no integer-coordinate area.
```

---

## 6.2 Shoelace Formula (Polygon Area)

```
  Given polygon with vertices (x₁,y₁), (x₂,y₂), ..., (xₙ,yₙ)
  listed in order (CW or CCW):
  
  2A = |Σᵢ (xᵢ × yᵢ₊₁ - xᵢ₊₁ × yᵢ)|
  
  where indices wrap: (xₙ₊₁, yₙ₊₁) = (x₁, y₁)

FUNCTION polygonArea(vertices[], n):
    area = 0
    FOR i = 0 TO n - 1:
        j = (i + 1) % n
        area += vertices[i].x × vertices[j].y
        area -= vertices[j].x × vertices[i].y
    RETURN abs(area) / 2

  Time:  O(n)
  Space: O(1)
```

### Visual: How Shoelace Works

```
  Vertices listed CCW: (1,1), (4,1), (4,4), (1,4) — a 3×3 square
  
  ┌──────────────────────────────────────────────────┐
  │   "Shoelace" cross-multiplication:               │
  │                                                   │
  │   x₁×y₂  x₂×y₃  x₃×y₄  x₄×y₁                  │
  │   1×1  + 4×4  + 4×4  + 1×1  = 1+16+16+1 = 34   │
  │                                                   │
  │   x₂×y₁  x₃×y₂  x₄×y₃  x₁×y₄                  │
  │   4×1  + 4×1  + 1×4  + 1×4  = 4+4+4+4   = 16   │
  │                                                   │
  │   2A = |34 - 16| = 18                            │
  │   A = 9 ✓ (3×3 square)                          │
  └──────────────────────────────────────────────────┘
```

### Trace: Pentagon

```
  Vertices: (0,0), (4,0), (5,3), (2,5), (0,3)
  
  i=0: 0×0 - 4×0 = 0
  i=1: 4×3 - 5×0 = 12
  i=2: 5×5 - 2×3 = 19
  i=3: 2×3 - 0×5 = 6
  i=4: 0×0 - 0×3 = 0
  
  Sum = 0 + 12 + 19 + 6 + 0 = 37
  A = |37| / 2 = 18.5
```

---

## 6.3 Pick's Theorem (Lattice Polygons)

```
  For a polygon with INTEGER vertex coordinates:
  
  A = I + B/2 - 1
  
  where:
  A = area of polygon
  I = number of INTERIOR lattice points
  B = number of BOUNDARY lattice points
  
  ┌────────────────────────────────────────────────────────┐
  │  How to compute B (boundary points):                   │
  │                                                         │
  │  For edge from (x₁,y₁) to (x₂,y₂):                   │
  │  Points on edge = gcd(|x₂-x₁|, |y₂-y₁|)              │
  │                                                         │
  │  B = Σ gcd(|Δxᵢ|, |Δyᵢ|) over all edges              │
  └────────────────────────────────────────────────────────┘
  
  Rearranged to find I:
  I = A - B/2 + 1   (where A from Shoelace, B from GCD sum)
  
  Example: Triangle (0,0), (4,0), (0,3)
  A = ½ × 4 × 3 = 6
  B = gcd(4,0) + gcd(4,3) + gcd(0,3) = 4 + 1 + 3 = 8
  I = 6 - 8/2 + 1 = 6 - 4 + 1 = 3
  
  Interior points: (1,1), (1,2), (2,1) → 3 ✓
```

---

## 6.4 Area of a Triangle (Multiple Methods)

```
  ┌───────────────────────────────────────────────────────────┐
  │  Method           │  Formula                              │
  ├────────────────────┼───────────────────────────────────────┤
  │  Cross product     │  ½|cross(B-A, C-A)|                  │
  │  Shoelace          │  Plug 3 vertices into formula        │
  │  Heron's formula   │  √(s(s-a)(s-b)(s-c)), s=(a+b+c)/2  │
  │  Base × Height     │  ½ × base × height                   │
  │  Determinant       │  ½|det([[x₁,y₁,1],[x₂,y₂,1],       │
  │                    │         [x₃,y₃,1]])|                 │
  └────────────────────┴───────────────────────────────────────┘
  
  ★ Cross product method is best for integer coordinates
  ★ Heron's formula needs side lengths (float), less accurate
```

---

## 6.5 Signed Area Applications

```
  1. Polygon orientation:
     Shoelace sum > 0 → CCW
     Shoelace sum < 0 → CW
     
  2. Point in triangle:
     Compute signed areas of triangles (P,A,B), (P,B,C), (P,C,A)
     If all have same sign → P is inside triangle
     
  3. Convex hull area:
     Apply Shoelace to hull vertices → O(n)

  4. Polygon winding:
     Sum of signed angles (or cross products) around a point
     tells you the winding number.
```

---

## 6.6 Area of Union / Intersection

```
  For simple cases:
  
  Two triangles: clip one against the other (Sutherland-Hodgman)
  Two convex polygons: O(n+m) intersection algorithm
  
  General polygon union: sweep line, O(n log n)
  
  ★ In CP, polygon area is almost always a direct Shoelace application.
```

---

## Summary Table

| Area Type | Method | Complexity |
|-----------|--------|-----------|
| Triangle (coordinates) | Cross product / Shoelace | O(1) |
| Triangle (side lengths) | Heron's formula | O(1) |
| Simple polygon | Shoelace formula | O(n) |
| Lattice polygon interior pts | Pick's theorem | O(n) |
| Convex hull area | Shoelace on hull | O(n) |

---

## Quick Revision Questions

1. **Compute** the area of triangle (1,1), (5,1), (3,6) using cross product.
2. **Apply** the Shoelace formula to square (0,0), (3,0), (3,3), (0,3).
3. **Use** Pick's theorem: A = 10, B = 6. How many interior points?
4. **Why is** 2×Area preferred over A in integer-coordinate problems?
5. **How do** you determine if polygon vertices are CCW or CW?
6. **Compute** boundary lattice points of triangle (0,0), (6,0), (0,4).

---

[← Previous: Intersection Points](05-intersection-points.md) | [Back to README](../README.md) | [Next: Unit 9 - Convex Hull →](../09-Computational-Geometry/01-convex-hull.md)
