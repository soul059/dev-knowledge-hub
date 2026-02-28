# Chapter 6: Container with Most Water

[← Previous: Three Sum Problem](05-three-sum-problem.md) | [Back to README](../README.md) | [Next Unit: Sliding Window →](../04-Sliding-Window/01-fixed-size-window.md)

---

## Overview

The "Container With Most Water" problem (LeetCode #11) is a classic two-pointer problem that beautifully demonstrates why the greedy pointer-movement strategy works. Given vertical lines, find two that together hold the most water.

---

## Problem Statement

```
Given: Array height[] where height[i] is the height of line at position i
Find:  Two lines that, with the x-axis, form a container holding the most water
Return: Maximum water area

  height = [1, 8, 6, 2, 5, 4, 8, 3, 7]

      8 │    █              █
      7 │    █              █        █
      6 │    █  █           █        █
      5 │    █  █     █     █        █
      4 │    █  █     █  █  █        █
      3 │    █  █     █  █  █  █     █
      2 │    █  █  █  █  █  █  █     █
      1 │ █  █  █  █  █  █  █  █     █
      0 └─────────────────────────────
        0  1  2  3  4  5  6  7  8

  Water between lines 1 and 8 (indices 1 and 8):
  width = 8 - 1 = 7
  height = min(8, 7) = 7
  area = 7 × 7 = 49
```

---

## Key Formula

$$\text{Area} = \min(\text{height}[L], \text{height}[R]) \times (R - L)$$

The water level is limited by the **shorter** line (water would spill over the shorter one).

---

## Approach 1: Brute Force — O(n²)

```
FUNCTION maxArea_brute(height, n):
    maxWater = 0
    FOR i = 0 TO n-2:
        FOR j = i+1 TO n-1:
            area = MIN(height[i], height[j]) × (j - i)
            maxWater = MAX(maxWater, area)
    RETURN maxWater
```

**Time: O(n²)** | **Space: O(1)**

---

## Approach 2: Two Pointers — O(n)

```
FUNCTION maxArea(height, n):
    L = 0, R = n - 1
    maxWater = 0

    WHILE L < R:
        width = R - L
        h = MIN(height[L], height[R])
        area = h × width
        maxWater = MAX(maxWater, area)

        // Move the pointer at the shorter line
        IF height[L] < height[R]:
            L = L + 1
        ELSE:
            R = R - 1

    RETURN maxWater
```

---

## Step-by-Step Trace

### height = [1, 8, 6, 2, 5, 4, 8, 3, 7]

```
  L=0, R=8: h=min(1,7)=1, w=8, area=8,  max=8
            height[L]=1 < height[R]=7 → L++

  L=1, R=8: h=min(8,7)=7, w=7, area=49, max=49  ← OPTIMAL
            height[L]=8 > height[R]=7 → R--

  L=1, R=7: h=min(8,3)=3, w=6, area=18, max=49
            height[L]=8 > height[R]=3 → R--

  L=1, R=6: h=min(8,8)=8, w=5, area=40, max=49
            height[L]=8 == height[R]=8 → R-- (either works)

  L=1, R=5: h=min(8,4)=4, w=4, area=16, max=49
            height[R]=4 < height[L]=8 → R--

  L=1, R=4: h=min(8,5)=5, w=3, area=15, max=49
            height[R]=5 < height[L]=8 → R--

  L=1, R=3: h=min(8,2)=2, w=2, area=4,  max=49
            height[R]=2 < height[L]=8 → R--

  L=1, R=2: h=min(8,6)=6, w=1, area=6,  max=49
            height[R]=6 < height[L]=8 → R--

  L=1, R=1: L not < R → STOP

  ANSWER: 49 ✓
```

---

## Why Move the Shorter Line? — Proof

```
  ┌──────────────────────────────────────────────────┐
  │  INTUITION:                                       │
  │                                                   │
  │  Area = min(h[L], h[R]) × width                  │
  │                                                   │
  │  If h[L] < h[R]:                                  │
  │    Current area is limited by h[L].               │
  │    Moving R left: width decreases, and            │
  │    min(h[L], h[R']) ≤ h[L] (still limited by L).  │
  │    → Area can ONLY decrease or stay same.         │
  │    → So all pairs (L, R'), R' < R are WORSE.     │
  │    → Safe to discard L entirely, move to L+1.    │
  │                                                   │
  │  Moving the SHORTER side is the only way we       │
  │  might find a taller line → potentially bigger    │
  │  area despite narrower width.                     │
  └──────────────────────────────────────────────────┘
```

### Formal Argument

```
  Suppose h[L] ≤ h[R].

  For any R' < R:
    width' = R' - L < R - L = width
    h' = min(h[L], h[R']) ≤ h[L]    (since h[L] is the bottleneck)
    area' = h' × width' ≤ h[L] × width = current area

  Therefore, NO pair (L, R') with R' < R can beat the current area.
  We can safely move L to L+1 without missing the optimal.
```

---

## Visual Proof

```
  Case: h[L] = 3, h[R] = 8, width = 6

  Current:
  3 │ █ ~~~~~~~~~~~~~ █████████
  2 │ █ ~~~~~~~~~~~~~ █████████
  1 │ █ ~~~~~~~~~~~~~ █████████
    └─────────────────────────
     L                       R
     area = 3 × 6 = 18

  If we keep L and try smaller R:

  3 │ █ ~~~~~~~~~~ █ (any height ≤ 8)
  2 │ █ ~~~~~~~~~~ █
  1 │ █ ~~~~~~~~~~ █
    └──────────────────
     L              R'
     area ≤ 3 × 5 = 15  (can only get smaller!)

  → Moving R is pointless. Move L instead.
```

---

## Complexity Analysis

```
  Each step: L++ or R--
  Start: L=0, R=n-1 → gap = n-1
  Each step reduces gap by 1
  Total steps: exactly n-1

  Time: O(n)
  Space: O(1)
```

---

## Related Problems

| Problem | Key Difference |
|---------|----------------|
| Container With Most Water | Lines have no width, area = min(h) × width |
| Trapping Rain Water | Fill between ALL bars, not just two |
| Largest Rectangle in Histogram | Bars have width, use stack-based approach |

```
  Container:                    Trapping Rain Water:
  █     █                       █
  █ ~~~ █                       █ █ ~ █
  █ ~~~ █ ~ █                   █ █ ~ █ █
  ──────────                    ──────────
  (two lines only)              (water between all bars)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Find two lines forming max area container |
| Formula | area = min(h[L], h[R]) × (R - L) |
| Optimal approach | Two pointers from both ends |
| Key rule | Move the pointer at the shorter line |
| Time | O(n) |
| Space | O(1) |
| Why it works | Shorter line limits area; moving taller side can only decrease area |

---

## Quick Revision Questions

1. **Why is the area limited by the shorter line?** Explain with a physical analogy.

2. **Prove that moving the taller pointer can never improve the area.**

3. **Is the algorithm greedy?** What greedy choice is being made at each step?

4. **What happens when both heights are equal?** Does it matter which pointer you move?

5. **How is "Container With Most Water" different from "Trapping Rain Water"?**

6. **If height = [1, 1, 1, 1, 1], what is the maximum area?** Trace through the algorithm.

---

[← Previous: Three Sum Problem](05-three-sum-problem.md) | [Back to README](../README.md) | [Next Unit: Sliding Window →](../04-Sliding-Window/01-fixed-size-window.md)
