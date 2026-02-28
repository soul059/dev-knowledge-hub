# Chapter 4: Trapping Rain Water

[← Previous: Largest Rectangle in Matrix](03-largest-rectangle-matrix.md) | [Next: Pattern Recognition →](05-pattern-recognition.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)

---

## Overview

The **Trapping Rain Water** problem (LeetCode #42) asks: given bar heights representing an elevation map, compute how much water can be trapped after raining. While solvable with multiple approaches, the **stack-based approach** elegantly finds water trapped layer by layer.

---

## Problem Statement

```
heights = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]

Elevation map with trapped water (~ = water):

3         │                    │
2    │~~~~│         │          │
1    │~~~~│    │~~~~│ │~~~~│
0 ___│____│____|____|_│____|_│    
  0  1  0  2  1  0  1  3  2  1  2  1

Trapped water = 6 units
```

---

## Approach 1: Stack-Based (Layer by Layer)

```
┌──────────────────────────────────────────────────────────┐
│  Idea: Use a decreasing stack.                           │
│                                                          │
│  When we find a bar taller than the stack top:           │
│  Water is trapped between the current bar, the stack     │
│  top (bottom of water), and the new stack top (left      │
│  wall) after popping.                                    │
│                                                          │
│  Water depth = min(left_wall, right_wall) - bottom       │
│  Water width = right_index - left_index - 1              │
│  Water volume = depth × width                            │
└──────────────────────────────────────────────────────────┘
```

### Algorithm

```
FUNCTION trap_stack(height):
    stack ← empty stack    // stores indices
    water ← 0
    
    FOR i = 0 TO n-1:
        WHILE stack NOT empty AND height[i] > height[stack.top()]:
            bottom ← height[stack.pop()]     // The pit/valley
            
            IF stack is empty: BREAK         // No left wall
            
            left ← stack.top()               // Left wall index
            w ← i - left - 1                 // Width
            h ← MIN(height[left], height[i]) - bottom   // Depth
            water ← water + w × h
        
        stack.push(i)
    
    RETURN water
```

### Trace: height = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]

```
i=0, h=0: Push 0.  Stack: [0]
i=1, h=1: 1>0 → Pop 0 (bottom=0), stack empty → break. Push 1.
           Stack: [1]
i=2, h=0: 0<1 → Push 2.  Stack: [1,2]
i=3, h=2: 2>0 → Pop 2 (bottom=0), left=1
           w=3-1-1=1, depth=min(1,2)-0=1, water+=1  [total=1]
           2>1 → Pop 1 (bottom=1), stack empty → break
           Push 3.  Stack: [3]

i=4, h=1: Push 4.  Stack: [3,4]
i=5, h=0: Push 5.  Stack: [3,4,5]
i=6, h=1: 1>0 → Pop 5 (bottom=0), left=4
           w=6-4-1=1, depth=min(1,1)-0=1, water+=1  [total=2]
           1≤1 → stop. Push 6.  Stack: [3,4,6]

i=7, h=3: 3>1 → Pop 6 (bottom=1), left=4
           w=7-4-1=2, depth=min(1,3)-1=0, water+=0
           3>1 → Pop 4 (bottom=1), left=3
           w=7-3-1=3, depth=min(2,3)-1=1, water+=3  [total=5]
           3>2 → Pop 3 (bottom=2), stack empty → break
           Push 7.  Stack: [7]

i=8, h=2: Push 8.  Stack: [7,8]
i=9, h=1: Push 9.  Stack: [7,8,9]
i=10, h=2: 2>1 → Pop 9 (bottom=1), left=8
            w=10-8-1=1, depth=min(2,2)-1=1, water+=1  [total=6]
            2≤2 → stop. Push 10.  Stack: [7,8,10]

i=11, h=1: Push 11.  Stack: [7,8,10,11]

Total water = 6 ✓
```

---

## Approach 2: Two-Pointer (Optimal Space)

```
FUNCTION trap_twoPointer(height):
    left ← 0, right ← n-1
    leftMax ← 0, rightMax ← 0
    water ← 0
    
    WHILE left < right:
        IF height[left] < height[right]:
            IF height[left] >= leftMax:
                leftMax ← height[left]
            ELSE:
                water ← water + (leftMax - height[left])
            left ← left + 1
        ELSE:
            IF height[right] >= rightMax:
                rightMax ← height[right]
            ELSE:
                water ← water + (rightMax - height[right])
            right ← right - 1
    
    RETURN water
```

---

## Approach 3: Prefix Max Arrays

```
FUNCTION trap_prefixMax(height):
    leftMax ← array: leftMax[i] = max(height[0..i])
    rightMax ← array: rightMax[i] = max(height[i..n-1])
    
    water ← 0
    FOR i = 0 TO n-1:
        water += MIN(leftMax[i], rightMax[i]) - height[i]
    
    RETURN water
```

### Intuition:

```
Water at position i = min(maxLeft, maxRight) - height[i]

  maxLeft      maxRight
  ────┐        ┌────
      │  water │
      │~~~~~~~~│
      │  h[i]  │
  ────┴────────┴────

Water level = min of two walls
Subtract ground level = height[i]
```

---

## All Three Approaches Compared

```
┌──────────────────┬──────┬───────┬────────────────────────┐
│ Approach         │ Time │ Space │ How it works           │
├──────────────────┼──────┼───────┼────────────────────────┤
│ Prefix Max       │ O(n) │ O(n)  │ Pre-compute max arrays │
│ Stack            │ O(n) │ O(n)  │ Layer-by-layer horizontal│
│ Two Pointer      │ O(n) │ O(1)  │ Converge from both ends│
└──────────────────┴──────┴───────┴────────────────────────┘
```

---

## Stack Approach: Water Calculation Visualization

```
height = [2, 1, 0, 1, 3]

Step 1: Pop index 2 (h=0), left wall=1(h=1), right wall=3(h=1)
        depth = min(1,1) - 0 = 1, width = 3-1-1 = 1
        Water: 1
        
            ┌───┐
  ┌───┐ ┌~┐ │   │
  │   │ │ │ │   │
  └───┘ └─┘ └───┘
   (2)  (1) (0) (1) (3)
              ^
              This cell: 1 unit

Step 2: Pop index 1 (h=1), left wall=0(h=2), right wall=3(h=1)
        depth = min(2,1) - 1 = 0, width = ... 
        Actually 0 depth → no water (already at level 1)

Step 3: Pop index 0 (h=2), right wall=4(h=3)
        etc.

Total water = 1 + 0 + 1 + 2 = 4
(Verify: position 1 gets 1, position 2 gets 2, position 3 gets 1 = 4 ✓)
```

---

## Complexity Analysis

| Approach | Time | Space |
|----------|------|-------|
| **Brute Force** | O(n²) | O(1) |
| **Prefix Max** | O(n) | O(n) |
| **Stack** | O(n) | O(n) |
| **Two Pointer** | O(n) | O(1) |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Water trapped between bars |
| **Stack Type** | Monotonically decreasing |
| **Pop gives** | Bottom of water pocket |
| **Left wall** | Stack top after pop |
| **Right wall** | Current index i |
| **Water** | min(leftH, rightH) - bottomH × width |
| **Time** | O(n) |
| **LeetCode** | #42 |

---

## Quick Revision Questions

1. **What does the stack store in the rain water problem?**
   > Indices of bars, maintained in decreasing order of height.

2. **How is water volume calculated when an element is popped?**
   > `width = i - stack.top() - 1`, `depth = min(height[left], height[i]) - bottom`, `volume = width × depth`.

3. **What happens when the stack is empty after a pop?**
   > There's no left wall, so no water can be trapped. We break out of the while loop.

4. **Why is the two-pointer approach more space-efficient?**
   > It only uses two variables (leftMax, rightMax) instead of arrays or a stack.

5. **For height = [3, 0, 0, 0, 3], how much water is trapped?**
   > 9. Each of the 3 middle positions holds 3 units (min(3,3) - 0 = 3), total = 3 × 3 = 9.

---

[← Previous: Largest Rectangle in Matrix](03-largest-rectangle-matrix.md) | [Next: Pattern Recognition →](05-pattern-recognition.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)
