# Chapter 5: Sweep Line Algorithm

[← Previous: Closest Pair of Points](04-closest-pair-of-points.md) | [Back to README](../README.md) | [Next: Rectangle Overlap →](06-rectangle-overlap.md)

---

## Overview

The **sweep line** paradigm processes geometric events in sorted order as an imaginary line sweeps across the plane. It reduces 2D problems to 1D data structure operations, commonly yielding $O(n \log n)$ solutions.

---

## 5.1 The Paradigm

```
  ┌──────────────────────────────────────────────────────────┐
  │  Key idea:                                               │
  │                                                          │
  │  1. Create EVENTS from geometric objects:                │
  │     • Start/end points, intersections, etc.              │
  │  2. Sort events by sweep coordinate (usually x)          │
  │  3. Process events left → right                         │
  │  4. Maintain a STATUS STRUCTURE (BST, BIT, etc.)        │
  │     representing the current slice                       │
  │                                                          │
  │     ║                                                    │
  │     ║  ──▶ sweep direction                              │
  │     ║                                                    │
  │     ║  status: ordered set of active objects             │
  │     ║  at this x-coordinate                              │
  │     ║                                                    │
  └──────────────────────────────────────────────────────────┘
```

---

## 5.2 Problem 1: Closest Pair (Sweep Line Approach)

```
  Alternative to divide-and-conquer:
  
  1. Sort points by x
  2. Maintain a balanced BST of active points (sorted by y)
  3. For each point P:
     a. Remove points with x < P.x - δ from BST
     b. Query BST for points within [P.y - δ, P.y + δ]
     c. Update δ if closer pair found
     d. Insert P into BST
  
  Time: O(n log n)
  Space: O(n)
  
  ★ Simpler to implement than divide-and-conquer for some!
```

---

## 5.3 Problem 2: Area of Union of Rectangles

```
  Given n axis-aligned rectangles, find total area of their union.
  
  ┌──────────────────────────────────────────────────────────┐
  │     ┌──────┐                                            │
  │     │  ┌───┼──┐     Overlapping rectangles              │
  │     │  │   │  │     Area ≠ sum of individual areas!     │
  │     └──┼───┘  │                                         │
  │        └──────┘                                         │
  └──────────────────────────────────────────────────────────┘
  
  Sweep line approach:
  1. Events: left edges (add) and right edges (remove)
  2. Sort by x-coordinate
  3. Status: track covered y-intervals using segment tree
  4. Between consecutive events, area += coveredLength × Δx
  
  Time: O(n log n) with segment tree for y-intervals
```

### Pseudocode

```
FUNCTION rectangleUnionArea(rects):
    events = []
    FOR each rect (x1, y1, x2, y2):
        events.add((x1, y1, y2, +1))    // left edge: open
        events.add((x2, y1, y2, -1))    // right edge: close
    
    Sort events by x
    Coordinate-compress y values
    Initialize segment tree on y-values
    
    area = 0
    FOR i = 0 TO events.size - 2:
        (x, y1, y2, type) = events[i]
        segTree.update(y1, y2, type)     // add or remove interval
        dx = events[i+1].x - x
        coveredY = segTree.queryCoveredLength()
        area += coveredY × dx
    
    RETURN area
```

---

## 5.4 Problem 3: Perimeter of Union of Rectangles

```
  Similar to area, but track the number of covered intervals
  and their boundaries.
  
  Perimeter contribution from horizontal sweeps AND vertical sweeps.
  Often solved with TWO sweep lines (one horizontal, one vertical)
  or a single sweep with careful bookkeeping.
  
  Time: O(n log n)
```

---

## 5.5 Problem 4: Skyline Problem

```
  Given n buildings (left, right, height), find the skyline contour.
  
  ┌──────────────────────────────────────────────────────────┐
  │        ┌────┐                                            │
  │     ┌──┤    │   ┌──┐                                    │
  │     │  │    │   │  │                                    │
  │  ┌──┤  │    ├───┤  ├──┐                                 │
  │  │  │  │    │   │  │  │                                 │
  │──┘  └──┘    └───┘  └──┘──                               │
  │                                                          │
  │  Skyline = upper envelope of all rectangles             │
  └──────────────────────────────────────────────────────────┘
  
  Events: building starts and ends (sorted by x)
  Status: multiset (or max-heap) of active heights
  
  At each event:
  - Add/remove height
  - If max height changes → output key point
  
  Time: O(n log n)
```

---

## 5.6 Problem 5: Maximum Overlap of Intervals

```
  1D version of sweep line (very common!):
  
  Given intervals [start, end], find maximum number of overlapping intervals.
  
  Events: (start, +1) and (end, -1)
  Sort by coordinate.
  Sweep and track running count.
  
  maxOverlap = max of running count
  
  Time: O(n log n)   Space: O(n)
```

---

## 5.7 General Sweep Line Template

```
  1. Create events from geometry
  2. Sort events by primary coordinate
  3. Initialize status data structure
  4. FOR each event:
       Update status (insert/delete/modify)
       Query status for needed information
       Accumulate result
  5. Return result
  
  ┌────────────────────────────────────────────────────────┐
  │  Problem            │ Status Structure │ Event Types   │
  ├─────────────────────┼──────────────────┼───────────────┤
  │  Segment intersect  │ Balanced BST     │ Endpoints     │
  │  Rectangle area     │ Segment tree     │ Left/right    │
  │  Closest pair       │ Balanced BST     │ Points        │
  │  Skyline            │ Multiset/heap    │ Start/end     │
  │  Interval overlap   │ Counter          │ Start/end     │
  └─────────────────────┴──────────────────┴───────────────┘
```

---

## Summary Table

| Problem | Status Structure | Time | Key Technique |
|---------|-----------------|------|---------------|
| Segment intersection | BST | O((n+k) log n) | Neighbor checking |
| Rectangle union area | Segment tree | O(n log n) | Covered length |
| Skyline | Max-heap/multiset | O(n log n) | Max height change |
| Closest pair | BST | O(n log n) | Window query |
| Interval overlap | Counter | O(n log n) | Running sum |

---

## Quick Revision Questions

1. **What are** the three components of any sweep line algorithm?
2. **How does** coordinate compression help in the rectangle union problem?
3. **What data structure** maintains active heights in the skyline problem?
4. **Why is** sweep line O(n log n) instead of O(n²)?
5. **Solve** maximum overlap for intervals [1,4], [2,6], [3,5], [5,8].
6. **When would** you use sweep line vs divide and conquer?

---

[← Previous: Closest Pair of Points](04-closest-pair-of-points.md) | [Back to README](../README.md) | [Next: Rectangle Overlap →](06-rectangle-overlap.md)
