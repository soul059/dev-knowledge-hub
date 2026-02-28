# Chapter 2: Maximum Area in Histogram

[← Previous: Stock Span Problem](01-stock-span-problem.md) | [Next: Largest Rectangle in Matrix →](03-largest-rectangle-matrix.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)

---

## Overview

The **Largest Rectangle in Histogram** (LeetCode #84) is one of the most important stack problems. Given bar heights representing a histogram, find the area of the **largest rectangle** that can be formed within it.

---

## Problem Statement

```
heights = [2, 1, 5, 6, 2, 3]

Histogram:
          ┌───┐
     ┌───┐│   │
     │   ││   │         ┌───┐
 ┌───┤   ││   │    ┌───┐│   │
 │   │   ││   │    │   ││   │
 │   │   ││   │    │   ││   │
 └───┴───┴┴───┘    └───┴┴───┘
  2    5    6        2    3
       1 (between)

Largest rectangle: 5 × 2 = 10

     ┌───┐
     │███││███│         
     │███││███│    
     │███││███│    
     │███││███│    
     └───┴┴───┘    
      5    6  → width=2, height=5, area=10
```

---

## Key Insight

```
┌──────────────────────────────────────────────────────────┐
│  For each bar of height h[i], the widest rectangle      │
│  using h[i] as height extends:                          │
│                                                          │
│  LEFT:  until we hit a bar shorter than h[i]            │
│         → Previous Smaller Element (PSE)                │
│                                                          │
│  RIGHT: until we hit a bar shorter than h[i]            │
│         → Next Smaller Element (NSE)                    │
│                                                          │
│  Width = right_boundary - left_boundary - 1             │
│  Area  = h[i] × width                                  │
│                                                          │
│  Maximum of all such areas = answer                     │
└──────────────────────────────────────────────────────────┘
```

---

## Approach 1: Two-Pass (Explicit PSE + NSE)

```
FUNCTION largestRectangle_TwoPass(heights):
    n ← length(heights)
    left ← array of size n     // PSE indices
    right ← array of size n    // NSE indices
    
    // Find Previous Smaller Element indices
    stack ← empty
    FOR i = 0 TO n-1:
        WHILE stack NOT empty AND heights[stack.top()] >= heights[i]:
            stack.pop()
        left[i] ← IF stack empty THEN -1 ELSE stack.top()
        stack.push(i)
    
    // Find Next Smaller Element indices
    stack ← empty
    FOR i = n-1 DOWNTO 0:
        WHILE stack NOT empty AND heights[stack.top()] >= heights[i]:
            stack.pop()
        right[i] ← IF stack empty THEN n ELSE stack.top()
        stack.push(i)
    
    // Calculate max area
    maxArea ← 0
    FOR i = 0 TO n-1:
        width ← right[i] - left[i] - 1
        area ← heights[i] × width
        maxArea ← MAX(maxArea, area)
    
    RETURN maxArea
```

### Trace: heights = [2, 1, 5, 6, 2, 3]

```
PSE (left boundaries):
  i=0: stack empty → left[0]=-1, push 0
  i=1: h[0]=2≥1 → pop, empty → left[1]=-1, push 1
  i=2: h[1]=1<5 → left[2]=1, push 2
  i=3: h[2]=5<6 → left[3]=2, push 3
  i=4: h[3]=6≥2→pop, h[2]=5≥2→pop, h[1]=1<2 → left[4]=1, push 4
  i=5: h[4]=2<3 → left[5]=4, push 5
  left = [-1, -1, 1, 2, 1, 4]

NSE (right boundaries):
  i=5: stack empty → right[5]=6, push 5
  i=4: h[5]=3≥2? NO → right[4]=5... wait.
  
  (Scan right to left)
  i=5: empty → right[5]=6, push 5
  i=4: h[5]=3 ≥ h[4]=2? YES → pop, empty → right[4]=6, push 4
  i=3: h[4]=2 < h[3]=6? YES → right[3]=4, push 3
  i=2: h[3]=6 ≥ h[2]=5? YES → pop, h[4]=2<5 → right[2]=4, push 2
  i=1: h[2]=5 ≥ h[1]=1? YES → pop, h[4]=2≥1→pop, empty → right[1]=6, push 1
  i=0: h[1]=1 < h[0]=2? YES → right[0]=1, push 0
  right = [1, 6, 4, 4, 6, 6]

Area calculation:
┌───────┬────────┬────────┬───────────────────┬────────┐
│ i     │ left   │ right  │ width             │ area   │
├───────┼────────┼────────┼───────────────────┼────────┤
│ 0(h=2)│  -1    │  1     │ 1-(-1)-1 = 1      │ 2×1=2  │
│ 1(h=1)│  -1    │  6     │ 6-(-1)-1 = 6      │ 1×6=6  │
│ 2(h=5)│   1    │  4     │ 4-1-1 = 2         │ 5×2=10 │
│ 3(h=6)│   2    │  4     │ 4-2-1 = 1         │ 6×1=6  │
│ 4(h=2)│   1    │  6     │ 6-1-1 = 4         │ 2×4=8  │
│ 5(h=3)│   4    │  6     │ 6-4-1 = 1         │ 3×1=3  │
└───────┴────────┴────────┴───────────────────┴────────┘

Maximum area = 10 (bar at index 2, height 5, spanning indices 2-3)
```

---

## Approach 2: Single-Pass (Optimized)

```
FUNCTION largestRectangle_SinglePass(heights):
    stack ← empty stack
    maxArea ← 0
    n ← length(heights)
    
    FOR i = 0 TO n:    // Note: goes to n (one extra)
        currHeight ← IF i == n THEN 0 ELSE heights[i]
        
        WHILE stack NOT empty AND currHeight < heights[stack.top()]:
            height ← heights[stack.pop()]
            width ← IF stack empty THEN i ELSE i - stack.top() - 1
            maxArea ← MAX(maxArea, height × width)
        
        stack.push(i)
    
    RETURN maxArea
```

### Single-Pass Trace: heights = [2, 1, 5, 6, 2, 3]

```
i=0, h=2: Push 0                    Stack: [0]
i=1, h=1: 1<2 → Pop 0
           height=2, width=1 (stack empty, so i=1)
           area = 2×1 = 2
           Push 1                    Stack: [1]
           
i=2, h=5: Push 2                    Stack: [1,2]
i=3, h=6: Push 3                    Stack: [1,2,3]
i=4, h=2: 2<6 → Pop 3
           height=6, width=4-2-1=1, area=6
           2<5 → Pop 2
           height=5, width=4-1-1=2, area=10 ← MAX!
           2≥1 → Push 4             Stack: [1,4]
           
i=5, h=3: Push 5                    Stack: [1,4,5]

i=6, h=0 (sentinel):
           0<3 → Pop 5
           height=3, width=6-4-1=1, area=3
           0<2 → Pop 4
           height=2, width=6-1-1=4, area=8
           0<1 → Pop 1
           height=1, width=6 (empty), area=6
           
maxArea = 10 ✓
```

---

## Why the Sentinel (h=0 at end)?

```
Without sentinel:        With sentinel (append 0):
  Some bars may remain    Guarantees ALL bars
  in the stack without    are popped and their
  being processed.        areas calculated.
  
  Need extra loop to      Clean single loop.
  handle remaining bars.
```

---

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| **Brute Force** | O(n²) | O(1) |
| **Two-Pass Stack** | O(n) | O(n) |
| **Single-Pass Stack** | O(n) | O(n) |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Largest rectangle in histogram |
| **Core Idea** | For each bar, find how far it can extend left and right |
| **Stack Type** | Increasing (pops when shorter bar arrives) |
| **Uses** | PSE (left boundary) + NSE (right boundary) |
| **Width Formula** | `right[i] - left[i] - 1` |
| **Time** | O(n) |
| **LeetCode** | #84 |

---

## Quick Revision Questions

1. **What determines the width of the rectangle for bar i?**
   > The distance between its Previous Smaller Element (left bound) and Next Smaller Element (right bound): `width = right[i] - left[i] - 1`.

2. **Why do we use a monotonically increasing stack?**
   > We need to find when a shorter bar appears (NSE), which limits how far a taller bar can extend.

3. **What is the purpose of the sentinel value (0) at the end?**
   > It ensures all remaining bars in the stack are popped and their areas calculated.

4. **In the single-pass approach, how is width calculated when the stack is empty after pop?**
   > `width = i` because the popped bar can extend all the way to the left edge (index 0).

5. **For heights = [3, 3, 3, 3], what is the largest rectangle?**
   > 12. Height 3, width 4, spanning all bars.

---

[← Previous: Stock Span Problem](01-stock-span-problem.md) | [Next: Largest Rectangle in Matrix →](03-largest-rectangle-matrix.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)
