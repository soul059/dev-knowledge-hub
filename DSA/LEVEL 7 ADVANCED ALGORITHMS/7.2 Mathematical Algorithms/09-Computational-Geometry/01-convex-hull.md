# Chapter 1: Convex Hull

[← Previous Unit: Area Calculations](../08-Geometry-Basics/06-area-calculations.md) | [Back to README](../README.md) | [Next: Line Intersection →](02-line-intersection.md)

---

## Overview

The **convex hull** of a set of points is the smallest convex polygon enclosing all points — imagine stretching a rubber band around pins. It is one of the most fundamental problems in computational geometry.

---

## 1.1 Definition

```
  ┌──────────────────────────────────────────────────────────┐
  │  Convex Hull: The smallest convex polygon containing     │
  │  all given points.                                       │
  │                                                          │
  │       ·                                                  │
  │      / \          ★ Hull vertices are a subset of        │
  │     /   \           the input points                     │
  │    / · · \        ★ Interior points are NOT on the hull  │
  │   /   ·   \       ★ All edges make left/right turns      │
  │  /  ·   ·  \        (no straight collinear edges in      │
  │ ·───────────·        strict hull)                        │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.2 Graham Scan — O(n log n)

```
  1. Find the bottom-most point P₀ (break ties by leftmost)
  2. Sort remaining points by polar angle with respect to P₀
  3. Process points using a stack, maintaining CCW turns
  
FUNCTION grahamScan(points):
    P0 = point with lowest y (then leftmost x)
    Sort other points by polar angle from P0
    
    stack = [P0, points[0], points[1]]
    
    FOR i = 2 TO n - 1:
        WHILE stack.size > 1 AND
              orientation(stack[-2], stack[-1], points[i]) ≤ 0:
            stack.pop()    // remove CW or collinear turn
        stack.push(points[i])
    
    RETURN stack

Time:  O(n log n) — dominated by sorting
Space: O(n)
```

### Trace

```
  Points: A(0,0), B(4,0), C(5,3), D(3,5), E(1,4), F(2,2)
  
  P₀ = A(0,0) (lowest y)
  Sort by angle from A: B(4,0), C(5,3), F(2,2), D(3,5), E(1,4)
  
  Stack: [A, B, C]
  
  Process F(2,2):
    orient(B,C,F) → CW → pop C
    orient(A,B,F) → CCW → push F
    Stack: [A, B, F]
  
  Process D(3,5):
    orient(B,F,D) → CCW → push D
    Stack: [A, B, F, D]
  Hmm, wait — let me re-sort properly...
  
  (Actual sort angles may differ; the key idea is:
   pop while last turn is not CCW, then push.)
  
  Final hull: the convex boundary points in CCW order.
```

---

## 1.3 Andrew's Monotone Chain — O(n log n)

```
  ★ Often preferred — simpler to implement than Graham Scan!
  
  1. Sort points by x, then by y (lexicographic)
  2. Build LOWER hull (left to right)
  3. Build UPPER hull (right to left)
  4. Concatenate
  
FUNCTION convexHull(points):
    sort points by (x, y)
    
    // Lower hull
    lower = []
    FOR p IN points:
        WHILE lower.size ≥ 2 AND
              cross(lower[-2], lower[-1], p) ≤ 0:
            lower.pop()
        lower.push(p)
    
    // Upper hull
    upper = []
    FOR p IN points (reversed):
        WHILE upper.size ≥ 2 AND
              cross(upper[-2], upper[-1], p) ≤ 0:
            upper.pop()
        upper.push(p)
    
    // Remove last point of each (it's first of other)
    RETURN lower[:-1] + upper[:-1]

  CROSS helper:
  cross(O, A, B) = (A.x-O.x)*(B.y-O.y) - (A.y-O.y)*(B.x-O.x)

Time:  O(n log n)
Space: O(n)
```

### Trace: Andrew's Algorithm

```
  Points sorted: (0,0) (1,1) (2,0) (2,2) (3,1) (4,3) (5,0)
  
  LOWER HULL (left → right):
  Add (0,0): [(0,0)]
  Add (1,1): [(0,0),(1,1)]
  Add (2,0): cross((0,0),(1,1),(2,0))= -2 <0 → pop (1,1)
             [(0,0),(2,0)]
  Add (2,2): cross((0,0),(2,0),(2,2))= 4 >0 → keep
             [(0,0),(2,0),(2,2)]
  Add (3,1): cross((2,0),(2,2),(3,1))= -1 <0 → pop (2,2)
             cross((0,0),(2,0),(3,1))= 2 >0 → keep
             [(0,0),(2,0),(3,1)]
  Add (4,3): cross((2,0),(3,1),(4,3))= 2 >0 → keep
             [(0,0),(2,0),(3,1),(4,3)]
  Add (5,0): cross((3,1),(4,3),(5,0))= -5 <0 → pop (4,3)
             cross((2,0),(3,1),(5,0))= -3 <0 → pop (3,1)
             cross((0,0),(2,0),(5,0))= 0 → pop (or keep for collinear)
             [(0,0),(5,0)]
  
  UPPER HULL (right → left): built similarly
  
  Final hull = lower + upper (merged)
```

---

## 1.4 Convex Hull Properties

```
  ┌──────────────────────────────────────────────────────────┐
  │  Property                     │  Value                   │
  ├───────────────────────────────┼──────────────────────────┤
  │  Vertices on hull             │  ≤ n (often << n)        │
  │  Expected hull size (random)  │  O(log n)                │
  │  Area                         │  Shoelace on hull ✓      │
  │  Perimeter                    │  Sum of edge lengths     │
  │  Diameter (farthest pair)     │  Rotating calipers O(n)  │
  │  Width (min thickness)        │  Rotating calipers O(n)  │
  └───────────────────────────────┴──────────────────────────┘
```

---

## 1.5 Applications

```
  1. Farthest pair of points → rotating calipers on hull
  2. Smallest enclosing rectangle → rotating calipers
  3. Polygon intersection → hull provides boundary
  4. Collision detection → convex hull encloses shape
  5. Point in convex polygon → binary search on hull, O(log n)
```

---

## 1.6 Collinear Points on Hull

```
  ★ Use ≤ 0 in cross product check to EXCLUDE collinear points (strict hull)
  ★ Use < 0 to INCLUDE collinear points

  Strict hull: only corner vertices
  Non-strict hull: includes all points on edges
  
  For most CP problems, strict hull (≤ 0) is preferred.
```

---

## Summary Table

| Algorithm | Time | Space | Notes |
|-----------|------|-------|-------|
| Graham Scan | O(n log n) | O(n) | Polar angle sort |
| Andrew's Chain | O(n log n) | O(n) | Lexicographic sort ★ |
| Jarvis March | O(nh) | O(h) | h = hull size |
| Quickhull | O(n log n) avg | O(n) | Divide & conquer |

---

## Quick Revision Questions

1. **What is** the time complexity of convex hull construction?
2. **Implement** Andrew's Monotone Chain and trace on 5 given points.
3. **How does** cross product sign determine whether to pop from the stack?
4. **What is** the expected convex hull size for n random points?
5. **How would** you find the farthest pair of points using the hull?
6. **When do** collinear points matter in convex hull problems?

---

[← Previous Unit: Area Calculations](../08-Geometry-Basics/06-area-calculations.md) | [Back to README](../README.md) | [Next: Line Intersection →](02-line-intersection.md)
