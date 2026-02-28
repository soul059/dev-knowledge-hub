# Chapter 6: Rectangle Overlap

[← Previous: Sweep Line Algorithm](05-sweep-line-algorithm.md) | [Back to README](../README.md) | [Next: Unit 10 - Fermat's Little Theorem →](../10-Advanced-Number-Theory/01-fermats-little-theorem.md)

---

## Overview

Rectangle overlap problems appear frequently in competitive programming and system design (collision detection, window management, map queries). This chapter covers detection, area computation, and multi-rectangle union/intersection using sweep lines and coordinate compression.

---

## 6.1 Two Rectangles — Overlap Detection

```
  Axis-aligned rectangles:
  R1 = (x1, y1, x2, y2)   // bottom-left to top-right
  R2 = (x3, y3, x4, y4)
  
  ┌─────────────────────────────────────────────────┐
  │  NO overlap when any of these is true:          │
  │                                                 │
  │   x1 ≥ x4   (R1 is to the right of R2)        │
  │   x3 ≥ x2   (R2 is to the right of R1)        │
  │   y1 ≥ y4   (R1 is above R2)                  │
  │   y3 ≥ y2   (R2 is above R1)                  │
  │                                                 │
  │  Overlap exists ⟺ NONE of the above holds     │
  └─────────────────────────────────────────────────┘
```

### Pseudocode

```
FUNCTION isOverlapping(R1, R2):
    IF R1.x1 >= R2.x2 OR R2.x1 >= R1.x2:
        RETURN false
    IF R1.y1 >= R2.y2 OR R2.y1 >= R1.y2:
        RETURN false
    RETURN true
```

### Trace Example

```
  R1 = (1, 1, 5, 5)    R2 = (3, 3, 7, 7)
  
      y
    7 ┌─────┐
    6 │     │
    5 ┌─────┼──┐│
    4 │  ▓▓▓│▓▓││     ▓ = overlap region
    3 │  ▓▓▓└──┼┘
    2 │     │
    1 └─────┘
      1  2  3  4  5  6  7  x
  
  Check: 1 < 7 ✓  3 < 5 ✓  1 < 7 ✓  3 < 5 ✓
  → Overlapping!
```

**Time:** $O(1)$ | **Space:** $O(1)$

---

## 6.2 Two Rectangles — Intersection Area

```
  Overlap rectangle (if it exists):
  
    overlapX1 = max(R1.x1, R2.x1)
    overlapY1 = max(R1.y1, R2.y1)
    overlapX2 = min(R1.x2, R2.x2)
    overlapY2 = min(R1.y2, R2.y2)
    
    width  = max(0, overlapX2 - overlapX1)
    height = max(0, overlapY2 - overlapY1)
    
    intersectionArea = width × height
```

### Pseudocode

```
FUNCTION intersectionArea(R1, R2):
    w = max(0, min(R1.x2, R2.x2) - max(R1.x1, R2.x1))
    h = max(0, min(R1.y2, R2.y2) - max(R1.y1, R2.y1))
    RETURN w × h
```

### Trace

```
  R1 = (1,1,5,5)   R2 = (3,3,7,7)
  
  w = min(5,7) - max(1,3) = 5 - 3 = 2
  h = min(5,7) - max(1,3) = 5 - 3 = 2
  Area = 2 × 2 = 4 ✓
```

---

## 6.3 Two Rectangles — Union Area

```
  Union = Area(R1) + Area(R2) - Intersection(R1, R2)
  
  From the trace above:
  Area(R1) = 4 × 4 = 16
  Area(R2) = 4 × 4 = 16
  Intersection = 4
  
  Union = 16 + 16 - 4 = 28
```

---

## 6.4 Multiple Rectangles — Coordinate Compression

```
  When dealing with n rectangles with large coordinates:
  
  1. Collect all distinct x-values → compress to indices 0..m
  2. Collect all distinct y-values → compress to indices 0..k
  3. Create a (m-1) × (k-1) grid of small rectangles
  4. Mark which small rectangles are covered
  5. Sum areas of covered cells
  
  Grid after compression:
  
    y4 ─ ┌───┬───┬───┐
         │   │ ▓ │ ▓ │    ▓ = covered cells
    y3 ─ ├───┼───┼───┤
         │ ▓ │ ▓ │   │
    y2 ─ ├───┼───┼───┤
         │ ▓ │   │   │
    y1 ─ └───┴───┴───┘
         x1  x2  x3  x4
  
  Time: O(n³) naive, O(n² log n) with sweep line + segment tree
```

---

## 6.5 Multiple Rectangles — Sweep Line + Segment Tree

```
  Best approach for n rectangles:
  
  1. Create events: left edges (+1) and right edges (-1)
  2. Sort events by x-coordinate
  3. Coordinate-compress y-intervals
  4. Use segment tree to track covered y-length
  
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │  For each pair of consecutive x-events:          │
  │                                                  │
  │    area += coveredYLength × (x[i+1] - x[i])     │
  │                                                  │
  │  The segment tree tracks for each y-interval:    │
  │    count[node] = # of rectangles covering it     │
  │    coveredLen  = total length where count > 0    │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

### Segment Tree Node

```
  struct Node {
      int count;         // number of full covers
      double coveredLen; // total covered length
  };
  
  Update: when adding interval [a,b]:
    For each leaf node fully inside [a,b]: count++
    Rebuild coveredLen bottom-up:
      if count > 0: coveredLen = full length of this range
      else:         coveredLen = sum of children's coveredLen
  
  Query: root.coveredLen gives total covered length
```

### Full Pseudocode

```
FUNCTION areaOfUnion(rectangles):
    // Step 1: Create events
    events = []
    yCoords = set()
    FOR each rect (x1, y1, x2, y2):
        events.add((x1, y1, y2, OPEN))
        events.add((x2, y1, y2, CLOSE))
        yCoords.add(y1)
        yCoords.add(y2)
    
    Sort events by x (break ties: OPEN before CLOSE)
    sortedY = sorted(yCoords)
    
    // Step 2: Build segment tree on y-intervals
    tree = SegmentTree(sortedY)
    
    // Step 3: Sweep
    area = 0
    FOR i = 0 TO events.size - 2:
        (x, y1, y2, type) = events[i]
        IF type == OPEN:
            tree.add(y1, y2)
        ELSE:
            tree.remove(y1, y2)
        
        dx = events[i+1].x - x
        coveredY = tree.queryCoveredLength()
        area += coveredY × dx
    
    RETURN area
```

### Step-by-Step Trace

```
  Rectangles: R1=(0,0,4,4)  R2=(2,1,6,3)
  
  Events sorted by x:
    x=0: OPEN  (0,4)   from R1
    x=2: OPEN  (1,3)   from R2
    x=4: CLOSE (0,4)   from R1
    x=6: CLOSE (1,3)   from R2
  
  Compressed y: {0, 1, 3, 4}
  
  Process x=0: add [0,4]
    covered = 4
    dx = 2-0 = 2
    area += 4×2 = 8
  
  Process x=2: add [1,3]
    covered still 4 (R1 covers [0,4] which includes [1,3])
    dx = 4-2 = 2
    area += 4×2 = 8
  
  Process x=4: remove [0,4]
    covered = 2 (only [1,3] from R2)
    dx = 6-4 = 2
    area += 2×2 = 4
  
  Total area = 8 + 8 + 4 = 20 ✓
  
  Verify: R1 area = 16, R2 area = 8
          Overlap = (min(4,6)-max(0,2)) × (min(4,3)-max(0,1)) = 2×2 = 4
          Union = 16 + 8 - 4 = 20 ✓
```

**Time:** $O(n \log n)$ | **Space:** $O(n)$

---

## 6.6 Inclusion-Exclusion for k Rectangles

```
  For small k (≤ 20):
  
  |R1 ∪ R2 ∪ ... ∪ Rk| = Σ|Ri| - Σ|Ri ∩ Rj| + Σ|Ri ∩ Rj ∩ Rk| - ...
  
  Intersection of axis-aligned rectangles is also a rectangle (or empty):
    Intersection.x1 = max of all x1's
    Intersection.y1 = max of all y1's
    Intersection.x2 = min of all x2's
    Intersection.y2 = min of all y2's
  
  If x1 < x2 and y1 < y2, it's valid; otherwise empty.
  
  Time: O(2^k × k)  — only practical for small k
```

---

## 6.7 Applications

```
  ┌──────────────────────────────────────────────────┐
  │  1. Collision detection in games                 │
  │  2. Window/UI layout overlap detection           │
  │  3. Map region overlap queries                   │
  │  4. Image composition & clipping                 │
  │  5. Geographic Information Systems (GIS)         │
  │  6. VLSI chip design (region coverage)           │
  └──────────────────────────────────────────────────┘
```

---

## 6.8 Common Patterns & Pitfalls

```
  ✓ DO: Use max(0, ...) to clamp widths/heights (avoid negative area)
  ✓ DO: Use long long for area when coordinates are large
  ✓ DO: Handle the case where rectangles just touch (area = 0)
  ✓ DO: Sort events carefully (OPEN before CLOSE at same x)
  
  ✗ DON'T: Forget coordinate compression for large coordinates
  ✗ DON'T: Use inclusion-exclusion for n > 20 (exponential blowup)
  ✗ DON'T: Confuse "open" vs "closed" intervals at boundaries
```

---

## Summary Table

| Problem | Approach | Time | Space |
|---------|----------|------|-------|
| 2-rect overlap check | Separation test | O(1) | O(1) |
| 2-rect intersection area | min/max formula | O(1) | O(1) |
| 2-rect union area | A+B-intersection | O(1) | O(1) |
| n-rect union area | Sweep + seg tree | O(n log n) | O(n) |
| n-rect union area (naive) | Coord compression | O(n³) | O(n²) |
| k-rect (small k) | Inclusion-exclusion | O(2^k · k) | O(k) |

---

## Quick Revision Questions

1. **State** the four non-overlap conditions for two axis-aligned rectangles.
2. **Compute** the intersection area of (0,0,3,3) and (1,1,5,5).
3. **Why** is `max(0, ...)` crucial in the intersection area formula?
4. **Explain** how a segment tree tracks covered y-length in the sweep line approach.
5. **What is** coordinate compression and when is it needed?
6. **For 3** rectangles, write the inclusion-exclusion formula for union area.

---

[← Previous: Sweep Line Algorithm](05-sweep-line-algorithm.md) | [Back to README](../README.md) | [Next: Unit 10 - Fermat's Little Theorem →](../10-Advanced-Number-Theory/01-fermats-little-theorem.md)
