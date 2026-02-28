# Chapter 3: Largest Rectangle in Binary Matrix

[← Previous: Max Area Histogram](02-max-area-histogram.md) | [Next: Trapping Rain Water →](04-trapping-rain-water.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)

---

## Overview

The **Maximal Rectangle** problem (LeetCode #85) extends the histogram problem to 2D: find the largest rectangle containing only 1s in a binary matrix. The key insight is to **reduce each row to a histogram** and apply the Largest Rectangle in Histogram algorithm.

---

## Problem Statement

```
Matrix:
  ┌───┬───┬───┬───┬───┐
  │ 1 │ 0 │ 1 │ 0 │ 0 │   Row 0
  ├───┼───┼───┼───┼───┤
  │ 1 │ 0 │ 1 │ 1 │ 1 │   Row 1
  ├───┼───┼───┼───┼───┤
  │ 1 │ 1 │ 1 │ 1 │ 1 │   Row 2
  ├───┼───┼───┼───┼───┤
  │ 1 │ 0 │ 0 │ 1 │ 0 │   Row 3
  └───┴───┴───┴───┴───┘

Largest rectangle of all 1s: area = 6
  (rows 1-2, cols 2-4: 2 rows × 3 cols)
```

---

## Key Insight: Row-by-Row Histograms

```
┌──────────────────────────────────────────────────────────┐
│  Treat each row as the BASE of a histogram.             │
│  Heights = consecutive 1s going upward from that row.   │
│                                                          │
│  If matrix[row][col] == 0: height = 0                   │
│  If matrix[row][col] == 1: height = height_above + 1    │
│                                                          │
│  Then solve "Largest Rectangle in Histogram" for each   │
│  row's heights. Maximum across all rows = answer.       │
└──────────────────────────────────────────────────────────┘
```

### Building Heights Row by Row

```
Matrix:       Heights after each row:

Row 0: [1,0,1,0,0]    heights = [1, 0, 1, 0, 0]
                          │        │        
                       ┌──┐     ┌──┐        
                       │1 │     │1 │        
                       └──┘     └──┘        

Row 1: [1,0,1,1,1]    heights = [2, 0, 2, 1, 1]
                       ┌──┐     ┌──┐
                       │2 │     │2 │
                       │  │     │  │┌──┬──┐
                       └──┘     └──┘│1 │1 │
                                    └──┴──┘

Row 2: [1,1,1,1,1]    heights = [3, 1, 3, 2, 2]
                       ┌──┐     ┌──┐
                       │3 │     │3 │
                       │  │     │  │┌──┬──┐
                       │  │┌──┐ │  ││2 │2 │
                       │  ││1 │ │  ││  │  │
                       └──┘└──┘ └──┘└──┴──┘

Row 3: [1,0,0,1,0]    heights = [4, 0, 0, 3, 0]
                       ┌──┐          ┌──┐
                       │4 │          │3 │
                       │  │          │  │
                       │  │          │  │
                       │  │          │  │
                       └──┘          └──┘
```

---

## Algorithm

```
FUNCTION maximalRectangle(matrix):
    IF matrix is empty: RETURN 0
    rows ← number of rows
    cols ← number of columns
    heights ← array of size cols, fill with 0
    maxArea ← 0
    
    FOR row = 0 TO rows-1:
        // Update heights for this row
        FOR col = 0 TO cols-1:
            IF matrix[row][col] == 1:
                heights[col] ← heights[col] + 1
            ELSE:
                heights[col] ← 0    // Reset: broken column
        
        // Apply histogram algorithm on current heights
        area ← largestRectangleInHistogram(heights)
        maxArea ← MAX(maxArea, area)
    
    RETURN maxArea
```

---

## Full Trace

```
Matrix:
  [1, 0, 1, 0, 0]
  [1, 0, 1, 1, 1]
  [1, 1, 1, 1, 1]
  [1, 0, 0, 1, 0]

═══ Row 0 ═══
heights = [1, 0, 1, 0, 0]
Histogram: max area = 1

═══ Row 1 ═══
heights = [2, 0, 2, 1, 1]
Histogram analysis:
  Bar 0 (h=2): extends just itself → 2×1 = 2
  Bar 2 (h=2): extends just itself → 2×1 = 2
  Bar 3 (h=1): extends to bars 2,3,4 → 1×3 = 3
  Bar 4 (h=1): same span as bar 3 when considered
  Max area this row = 3

═══ Row 2 ═══
heights = [3, 1, 3, 2, 2]
Histogram analysis:
  Bar 0 (h=3): width=1 → area=3
  Bar 1 (h=1): width=5 → area=5
  Bar 2 (h=3): width=1 → area=3
  Bar 3 (h=2): extends to bars 2,3,4 → 2×3 = 6  ← MAX!
  Bar 4 (h=2): same group
  Max area this row = 6

═══ Row 3 ═══
heights = [4, 0, 0, 3, 0]
Histogram: max area = 4 (bar 0 alone)

═══ ANSWER = 6 ═══
```

---

## Why Height Resets to 0

```
When matrix[row][col] = 0:

  Column col is BROKEN at this row.
  No rectangle can span across this 0.
  
  ┌───┐
  │ 1 │ ← heights build up
  │ 1 │
  │ 0 │ ← BREAK! height resets to 0
  │ 1 │ ← starts fresh from 1
  └───┘
  
  If we didn't reset: we'd count non-contiguous 1s,
  which don't form a valid rectangle.
```

---

## Complexity Analysis

| Aspect | Complexity |
|--------|-----------|
| **Time** | O(rows × cols) — each row histogram is O(cols) |
| **Space** | O(cols) — heights array + stack |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Largest rectangle of 1s in binary matrix |
| **Reduction** | Each row → histogram → max rectangle |
| **Height Update** | `h[col] = (cell==1) ? h[col]+1 : 0` |
| **Sub-problem** | Largest Rectangle in Histogram (per row) |
| **Time** | O(rows × cols) |
| **Space** | O(cols) |
| **LeetCode** | #85 |

---

## Quick Revision Questions

1. **How does a 2D matrix problem reduce to a 1D histogram problem?**
   > Each row acts as a histogram base. Heights are the count of consecutive 1s going upward from that row.

2. **What happens to the height when a 0 is encountered in the matrix?**
   > It resets to 0 because the column of 1s is broken at that point.

3. **How many times do we run the histogram algorithm?**
   > Once per row in the matrix (total = number of rows).

4. **What is the total time complexity?**
   > O(rows × cols). Each histogram computation is O(cols), and we do it for each of the rows.

5. **For a 3×3 matrix of all 1s, what is the answer?**
   > 9. The entire matrix is the rectangle: 3 rows × 3 cols.

---

[← Previous: Max Area Histogram](02-max-area-histogram.md) | [Next: Trapping Rain Water →](04-trapping-rain-water.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)
