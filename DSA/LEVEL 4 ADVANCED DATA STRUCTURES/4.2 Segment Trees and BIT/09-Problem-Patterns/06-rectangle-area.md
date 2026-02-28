# Rectangle Area (Coordinate Compression + Sweep Line)

## Chapter Overview

Computing the total area covered by a set of axis-aligned rectangles is a classic geometry problem that uses coordinate compression with a segment tree or BIT. This chapter covers the sweep line algorithm and its connection to range data structures.

---

## Problem Statement

```
Given n axis-aligned rectangles, each defined by (x1, y1, x2, y2),
find the total area covered by their union (overlapping areas counted once).

Example:
  Rectangle 1: (1, 1, 4, 4)
  Rectangle 2: (2, 2, 5, 5)

  ──────────────
  5 │    ┌───┐
  4 │┌──┐│   │
  3 ││  ├┤   │     Overlap region: (2,2)-(4,4)
  2 ││  ├┤   │     Total area = 9 + 9 - 4 = 14
  1 │└──┘└───┘       (if using inclusion-exclusion)
  0 └──────────
    0 1 2 3 4 5

  But with many rectangles, inclusion-exclusion is exponential.
  We need a sweep line approach.
```

---

## Sweep Line Concept

```
Sweep a vertical line from left to right.

At each x-coordinate where a rectangle starts or ends:
  - Rectangle starts: ADD its y-range to the "active set"
  - Rectangle ends: REMOVE its y-range from the "active set"

Between consecutive x-events:
  Area contribution = (active y-length) × (x-distance)

                 Active y-ranges
  x=1: add [1,4] → length = 3
  x=2: add [2,5] → length = 4 (union of [1,4] and [2,5] = [1,5])
  x=4: rem [1,4] → length = 3 (only [2,5] remains)
  x=5: rem [2,5] → length = 0

  Area = 3×(2-1) + 4×(4-2) + 3×(5-4)
       = 3 + 8 + 3 = 14 ✓
```

---

## Algorithm Steps

```
1. Create events for each rectangle:
   (x1, "open", y1, y2)   — left edge
   (x2, "close", y1, y2)  — right edge

2. Sort events by x-coordinate.

3. Coordinate compress y-values.

4. Maintain a segment tree on y-axis that tracks:
   - How many rectangles cover each y-segment
   - Total length of y-segments covered by ≥ 1 rectangle

5. Sweep left to right:
   total_area = 0
   prev_x = events[0].x
   for each event (x, type, y1, y2):
       // Add area contribution from prev_x to current x
       active_length = seg_tree.query_total_covered()
       total_area += active_length × (x - prev_x)
       
       // Process event
       if type == "open":
           seg_tree.range_add(y1, y2, +1)
       else:
           seg_tree.range_add(y1, y2, -1)
       
       prev_x = x
   
   return total_area
```

---

## Segment Tree for Active Length

```
Each segment tree node covers a y-range [l, r]:
  count[node] = number of rectangles fully covering [l, r]
  covered[node] = total length within [l, r] that is covered by ≥ 1 rectangle

Update covered:
  if count[node] > 0:
      // This entire segment is covered
      covered[node] = y[r+1] - y[l]   // actual y-length (using compressed coords)
  else if l == r:
      covered[node] = 0
  else:
      covered[node] = covered[left] + covered[right]

Key insight: We DON'T need lazy propagation!
  - count[] only changes when a rectangle fully covers a segment
  - We update count[] for the fully-covering level
  - covered[] is recomputed bottom-up
  
  This works because rectangle edges always correspond to
  coordinate-compressed segment boundaries.
```

---

## Coordinate Compression for Y-axis

```
Rectangles: (1,1,4,4), (2,2,5,5), (0,3,6,6)

Y-values: {1, 2, 3, 4, 5, 6}
Compressed: y[0]=1, y[1]=2, y[2]=3, y[3]=4, y[4]=5, y[5]=6

Segments (between consecutive y-values):
  seg 0: [1, 2)  length = 1
  seg 1: [2, 3)  length = 1
  seg 2: [3, 4)  length = 1
  seg 3: [4, 5)  length = 1
  seg 4: [5, 6)  length = 1

Segment tree has 5 leaves (one per y-segment).
Node covers segment range → actual y-length = y[r+1] - y[l].
```

---

## Worked Example

```
Rectangles: R1=(1,1,3,3), R2=(2,0,4,2)

Events (sorted by x):
  x=1: open  R1 y=[1,3)
  x=2: open  R2 y=[0,2)
  x=3: close R1 y=[1,3)
  x=4: close R2 y=[0,2)

Y-values: {0, 1, 2, 3}
Segments: [0,1), [1,2), [2,3)

Process:

x=1: open R1 [1,3) → add 1 to segments [1,2),[2,3)
  covered = 0 + 1 + 1 = 2  (segments [1,2) and [2,3))
  
x=2 (distance = 1):
  area += 2 × 1 = 2
  open R2 [0,2) → add 1 to segments [0,1),[1,2)
  covered = 1 + 1 + 1 = 3  (all three segments covered)

x=3 (distance = 1):
  area += 3 × 1 = 3
  close R1 [1,3) → sub 1 from segments [1,2),[2,3)
  Segment [0,1): count=1 → covered
  Segment [1,2): count=1 (R2 still covers) → covered
  Segment [2,3): count=0 → not covered
  covered = 1 + 1 + 0 = 2

x=4 (distance = 1):
  area += 2 × 1 = 2
  close R2 → sub 1 from segments [0,1),[1,2)
  covered = 0

Total area = 2 + 3 + 2 = 7

Verify: R1 area = 4, R2 area = 4, overlap = 1×1 = 1
        Union = 4 + 4 - 1 = 7 ✓
```

---

## Complexity

```
Let n = number of rectangles.

Events: 2n (one open + one close per rectangle)
Sorting events: O(n log n)
Y-coordinates: at most 2n unique values
Segment tree size: O(n)
Processing each event: O(log n) for segment tree update

Total time: O(n log n)
Total space: O(n)
```

---

## Related Problems

### Perimeter of Union of Rectangles

```
Same sweep line, but track:
  - Number of connected intervals on y-axis
  - Changes in covered length

Perimeter contribution:
  Vertical edges: |change in covered length| at each x-event
  Horizontal edges: 2 × number_of_intervals × x-distance
  
Requires segment tree that also counts:
  - Number of disjoint covered segments (not just total length)
```

### Rectangle Area with Point Queries

```
"How many rectangles cover point (x, y)?"

Offline: sweep line + BIT/segment tree on y-axis
  At x-events, add/remove y-ranges
  Query at specific (x, y): query BIT at y

Online: 2D segment tree or persistent segment tree
```

### Number of Distinct Rectangles Covering Each Point

```
Sweep line + segment tree with lazy propagation
  range_add(y1, y2, +1) for open events
  range_add(y1, y2, -1) for close events
  Query any point → count at that point
```

---

## Alternative: Discretization Without Segment Tree

```
For small n (≤ 1000):
  1. Coordinate compress both x and y
  2. Create a 2D grid of compressed cells
  3. Mark each cell as covered/not-covered
  4. Sum up areas of covered cells
  
  Time: O(n²) — acceptable for small n
  
  For large n: sweep line + segment tree is necessary.
```

---

## Summary Table

| Problem | Structure | Time | Key Technique |
|---------|----------|------|--------------|
| Union area | Segment tree | O(n log n) | Sweep line + range add |
| Union perimeter | Segment tree | O(n log n) | Track interval count |
| Rectangle cover count | BIT/SegTree | O(n log n) | Range add/query |
| 2D area with updates | 2D SegTree | O(n log² n) | 2D range operations |

---

## Quick Revision Questions

1. What is the sweep line approach for rectangle union area? *(Sweep vertically; at each x-event, add/remove y-ranges; area = active_y_length × x_distance between events)*
2. Why doesn't the segment tree need lazy propagation here? *(Because updates always correspond to full compressed segments, and we compute covered length bottom-up from count values)*
3. What does coordinate compression achieve? *(Reduces potentially infinite y-range to O(n) segments, making the segment tree size O(n))*
4. What is the time complexity? *(O(n log n) — sorting events + O(log n) per event for segment tree update)*
5. How is the covered length computed at each node? *(If count > 0: covered = full segment length. Otherwise: sum of children's covered lengths)*

---

[← Previous: Count Smaller After Self](05-count-smaller-after-self.md) | [Back to Course README →](../README.md)
