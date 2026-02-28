# Chapter 5: Classic Problems

## 📋 Chapter Overview
In-depth walkthroughs of the most frequently asked two-pointer problems in interviews.

---

## 🧪 Problem 1: Trapping Rain Water

**Problem:** Given elevation map, compute how much water it can trap.

**Input:** `height = [0,1,0,2,1,0,1,3,2,1,2,1]`  **Output:** `6`

```
  Visualization:
                        █
            █           █ █   █
    █       █ █   █     █ █ █ █ █
  0 1 0 2 1 0 1 3 2 1 2 1
  
  Water fills the gaps:
                        █
            █ ~ ~ ~ ~ ~ █ █ ~ █
    █ ~ ~ ~ █ █ ~ █ ~ ~ █ █ █ █ █
  0 1 0 2 1 0 1 3 2 1 2 1
  
  Water at each position = min(leftMax, rightMax) - height[i]
```

### Two Pointer Approach:

```
  left = 0, right = 11
  leftMax = 0, rightMax = 0
  
  Process the side with smaller max (that side is the bottleneck):
  
  ┌──────┬──────┬────────┬─────────┬───────┬───────┐
  │  L   │  R   │ lMax   │  rMax   │ water │ total │
  ├──────┼──────┼────────┼─────────┼───────┼───────┤
  │  0   │ 11   │ 0      │  1      │  0    │  0    │
  │  1   │ 11   │ 1      │  1      │  0    │  0    │
  │  1   │ 10   │ 1      │  2      │  0    │  0    │
  │  2   │ 10   │ 1      │  2      │  1    │  1    │
  │  3   │ 10   │ 2      │  2      │  0    │  1    │
  │  4   │ 10   │ 2      │  2      │  1    │  2    │
  │  5   │ 10   │ 2      │  2      │  2    │  4    │
  │  6   │ 10   │ 2      │  2      │  1    │  5    │
  │  7   │ 10   │ 3      │  2      │  0    │  5    │
  │  7   │  9   │ 3      │  2      │  1    │  6    │
  │  7   │  8   │ 3      │  2      │  0    │  6    │
  └──────┴──────┴────────┴─────────┴───────┴───────┘
  
  Answer: 6
```

### Pseudocode:

```
function trap(height):
    left = 0, right = n - 1
    leftMax = 0, rightMax = 0
    water = 0
    
    while left < right:
        if height[left] < height[right]:
            if height[left] >= leftMax:
                leftMax = height[left]
            else:
                water += leftMax - height[left]
            left++
        else:
            if height[right] >= rightMax:
                rightMax = height[right]
            else:
                water += rightMax - height[right]
            right--
    
    return water
```

**Time:** O(n)  **Space:** O(1)

---

## 🧪 Problem 2: Sort Colors (Dutch National Flag)

**Problem:** Sort array containing only 0, 1, 2 in-place.

**Input:** `[2, 0, 2, 1, 1, 0]`  **Output:** `[0, 0, 1, 1, 2, 2]`

```
  Three pointers: low, mid, high
  
  ┌─────┬─────┬──────┬────────────────────────────┐
  │ low │ mid │ high │ Array                       │
  ├─────┼─────┼──────┼────────────────────────────┤
  │  0  │  0  │  5   │ [2, 0, 2, 1, 1, 0]        │
  │     │     │      │ arr[mid]=2, swap(mid,high) │
  │  0  │  0  │  4   │ [0, 0, 2, 1, 1, 2]        │
  │     │     │      │ arr[mid]=0, swap(low,mid)  │
  │  1  │  1  │  4   │ [0, 0, 2, 1, 1, 2]        │
  │     │     │      │ arr[mid]=0, swap(low,mid)  │
  │  2  │  2  │  4   │ [0, 0, 2, 1, 1, 2]        │
  │     │     │      │ arr[mid]=2, swap(mid,high) │
  │  2  │  2  │  3   │ [0, 0, 1, 1, 2, 2]        │
  │     │     │      │ arr[mid]=1, mid++          │
  │  2  │  3  │  3   │ [0, 0, 1, 1, 2, 2]        │
  │     │     │      │ arr[mid]=1, mid++          │
  │  2  │  4  │  3   │ mid > high, DONE!          │
  └─────┴─────┴──────┴────────────────────────────┘
  
  Result: [0, 0, 1, 1, 2, 2] ✓
```

**Time:** O(n)  **Space:** O(1)

---

## 🧪 Problem 3: Squares of a Sorted Array

**Problem:** Given sorted array, return squares in sorted order.

**Input:** `[-4, -1, 0, 3, 10]`  **Output:** `[0, 1, 9, 16, 100]`

```
  Key insight: Largest squares are at the ENDS (negative or positive extremes)
  
  [-4, -1, 0, 3, 10]
    L              R      Compare |arr[L]|² vs |arr[R]|²
  
  ┌──────┬──────┬────────────┬────────────┬────────────────┐
  │  L   │  R   │ L²         │ R²         │ Result (back)  │
  ├──────┼──────┼────────────┼────────────┼────────────────┤
  │  0   │  4   │ 16         │ 100        │ [_,_,_,_,100]  │
  │  0   │  3   │ 16         │ 9          │ [_,_,_,16,100] │
  │  1   │  3   │ 1          │ 9          │ [_,_,9,16,100] │
  │  1   │  2   │ 1          │ 0          │ [_,1,9,16,100] │
  │  2   │  2   │ 0          │ 0          │ [0,1,9,16,100] │
  └──────┴──────┴────────────┴────────────┴────────────────┘
  
  Answer: [0, 1, 9, 16, 100] ✓
```

**Time:** O(n)  **Space:** O(n) for result

---

## 🧪 Problem 4: Partition Labels

**Problem:** Partition string so each letter appears in at most one part.

**Input:** `"ababcbacadefegdehijhklij"`

```
  Step 1: Find last occurrence of each character
  Step 2: Use two pointers to track partition boundaries
  
  Last occurrence: a→8, b→5, c→7, d→14, e→15, f→11, g→13, h→19, i→22, j→23, k→20, l→21
  
  Scan with two pointers (start and end of partition):
  i=0 'a': end = max(0, 8) = 8
  i=1 'b': end = max(8, 5) = 8
  ...
  i=8 'a': end = max(8, 8) = 8, i==end → partition! size=9
  i=9 'd': end = max(9, 14) = 14
  ...
  i=15 'e': end = max(15, 15) = 15, i==end → partition! size=7
  ...
  
  Answer: [9, 7, 8]
```

**Time:** O(n)  **Space:** O(1)

---

## 📊 Problems Summary

| Problem | Variant | Key Insight | Time |
|---------|---------|-------------|------|
| Trapping Rain Water | Opposite | Process shorter side first | O(n) |
| Sort Colors | 3 Pointers (DNF) | Three regions: 0s, 1s, 2s | O(n) |
| Squares Sorted Array | Opposite (fill back) | Largest at extremes | O(n) |
| Partition Labels | Same dir + tracking | Track last occurrence | O(n) |
| Merge Sorted Arrays | Same dir | Write from end to avoid shifting | O(m+n) |
| Backspace Compare | Same dir (backward) | Process '#' as backspace | O(n) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Trapping Water | Two-pointer from both ends, track left/right max |
| Dutch National Flag | Three pointers partition into three regions |
| Squares | Fill result from back, compare absolute values |
| Partition Labels | Track last occurrence, extend partition greedily |

---

## ❓ Quick Revision Questions

1. **In trapping rain water, why process the shorter side?** → The shorter max is the bottleneck; water level is determined by `min(leftMax, rightMax)`.
2. **What are the three regions in Dutch National Flag?** → `[0..low-1]` = 0s, `[low..mid-1]` = 1s, `[high+1..n-1]` = 2s.
3. **Why fill squares array from the back?** → Largest squares are at the extremes; filling from back avoids sorting.
4. **Time complexity of all these problems?** → All O(n), one or two passes.
5. **What makes trapping rain water a two-pointer problem?** → Each position's water depends on max heights to its left and right, which two pointers track simultaneously.
6. **In partition labels, how do you determine partition end?** → Track the farthest last occurrence of any character seen so far. When current index equals that farthest point, partition ends.

---

[← Previous: Template and Variations](04-template-and-variations.md) | [Next: When to Apply →](06-when-to-apply.md)

[← Back to README](../README.md)
