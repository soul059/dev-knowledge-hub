# Chapter 2: Monotonic Stack

## 📋 Chapter Overview
Monotonic stack pattern: efficiently find the next/previous greater/smaller element. Core technique for O(n) solutions to problems that seem O(n²).

---

## 🧠 Core Concept

```
  Monotonic Stack: stack where elements are always in
  sorted order (increasing or decreasing).
  
  Monotonic DECREASING stack (for Next Greater Element):
  
  Before push 4:        After pop 2,3 and push 4:
  ┌───┐                 ┌───┐
  │ 3 │                 │   │
  │ 2 │                 │   │
  │ 5 │                 │ 5 │
  │ 7 │                 │ 7 │
  └───┘                 └───┘
                         push 4:
                        ┌───┐
                        │ 4 │
                        │ 5 │
                        │ 7 │
                        └───┘
  
  Pop 2 → next greater of 2 is 4
  Pop 3 → next greater of 3 is 4
```

---

## 📝 Problem 1: Next Greater Element (Template)

**Problem:** For each element, find the next element to its right that is greater.

```
function nextGreaterElement(nums):
    n = len(nums)
    result = array of -1s, size n
    stack = []  // stores INDICES, decreasing by value
    
    for i = 0 to n-1:
        while stack not empty and nums[stack.top()] < nums[i]:
            idx = stack.pop()
            result[idx] = nums[i]
        stack.push(i)
    
    return result
```

### Trace

```
  nums = [2, 1, 2, 4, 3]
  
  i=0, val=2: stack empty → push 0.          stack=[0]
  i=1, val=1: 1 < nums[0]=2 → push 1.       stack=[0,1]
  i=2, val=2: 2 > nums[1]=1 → pop 1, result[1]=2.
              2 = nums[0]=2, not > → stop.   push 2. stack=[0,2]
  i=3, val=4: 4 > nums[2]=2 → pop 2, result[2]=4
              4 > nums[0]=2 → pop 0, result[0]=4
              push 3.                        stack=[3]
  i=4, val=3: 3 < nums[3]=4 → push 4.       stack=[3,4]
  
  Remaining in stack: result[3]=-1, result[4]=-1
  
  Result: [4, 2, 4, -1, -1]
```

**Complexity:** O(n) — each element pushed and popped at most once.

---

## 📝 Problem 2: Daily Temperatures (LeetCode 739)

**Problem:** For each day, how many days until a warmer temperature?

```
function dailyTemperatures(temps):
    n = len(temps)
    result = array of 0s, size n
    stack = []  // indices, decreasing by temperature
    
    for i = 0 to n-1:
        while stack not empty and temps[stack.top()] < temps[i]:
            idx = stack.pop()
            result[idx] = i - idx
        stack.push(i)
    
    return result
```

### Trace

```
  temps = [73, 74, 75, 71, 69, 72, 76, 73]
  
  i=0 (73): push 0.                             stack=[0]
  i=1 (74): 74>73 → pop 0, res[0]=1-0=1.        stack=[1]
  i=2 (75): 75>74 → pop 1, res[1]=2-1=1.        stack=[2]
  i=3 (71): push 3.                             stack=[2,3]
  i=4 (69): push 4.                             stack=[2,3,4]
  i=5 (72): 72>69 → pop 4, res[4]=5-4=1.
            72>71 → pop 3, res[3]=5-3=2.        stack=[2,5]
  i=6 (76): 76>72 → pop 5, res[5]=6-5=1.
            76>75 → pop 2, res[2]=6-2=4.        stack=[6]
  i=7 (73): push 7.                             stack=[6,7]
  
  Result: [1, 1, 4, 2, 1, 1, 0, 0]
```

---

## 📝 Problem 3: Largest Rectangle in Histogram (LeetCode 84)

**Problem:** Find the largest rectangular area in a histogram.

```
  Histogram:
    6
  ┌───┐
  │   │ 5
  │   ├───┐
  │   │   │       4
  │   │   │   3 ┌───┐
  │   │   │ ┌───┤   │
  │   │   │ │   │   │
  │   │   │ │   │   │ 2
  │   │   │ │   │   ├───┐
  │   │   │ │   │   │   │
  └───┴───┴─┴───┴───┴───┘
   [6]  5   3   4   2

  Largest rectangle = 3 × 3 = 9 (using bars of height 3,4,2... 
  actually: bars at index 2,3,4 have min height 3, width 3 → area 9... 
  wait, let's trace properly)
```

### Pseudocode

```
function largestRectangleArea(heights):
    stack = []  // indices, increasing by height
    maxArea = 0
    
    for i = 0 to n:   // n is sentinel (height 0)
        h = (i == n) ? 0 : heights[i]
        
        while stack not empty and heights[stack.top()] > h:
            height = heights[stack.pop()]
            width = stack.empty() ? i : i - stack.top() - 1
            maxArea = max(maxArea, height * width)
        
        stack.push(i)
    
    return maxArea
```

### Trace

```
  heights = [2, 1, 5, 6, 2, 3]
  
  i=0 (2): push 0.                                      stack=[0]
  i=1 (1): 1 < 2 → pop 0. h=2, w=1. area=2. maxArea=2. stack=[1]
  i=2 (5): push 2.                                      stack=[1,2]
  i=3 (6): push 3.                                      stack=[1,2,3]
  i=4 (2): 2 < 6 → pop 3. h=6, w=4-2-1=1. area=6.      maxArea=6
           2 < 5 → pop 2. h=5, w=4-1-1=2. area=10.     maxArea=10
           2 ≥ 1 → stop.  push 4.                       stack=[1,4]
  i=5 (3): push 5.                                      stack=[1,4,5]
  i=6 (0): 0 < 3 → pop 5. h=3, w=6-4-1=1. area=3.
           0 < 2 → pop 4. h=2, w=6-1-1=4. area=8.
           0 < 1 → pop 1. h=1, w=6. area=6.             maxArea=10
  
  Answer: 10
```

---

## 📐 Monotonic Stack Variants

| Variant | Stack Order | Finds |
|---------|-------------|-------|
| Decreasing stack | Top is smallest | Next Greater Element |
| Increasing stack | Top is largest | Next Smaller Element |
| Previous greater | Iterate left to right, decreasing | Previous Greater Element |
| Circular variant | Process array twice (i % n) | Next Greater in circular array |

### Circular Next Greater (LeetCode 503)

```
function nextGreaterCircular(nums):
    n = len(nums)
    result = array of -1s, size n
    stack = []
    
    for i = 0 to 2*n - 1:
        while stack not empty and nums[stack.top()] < nums[i % n]:
            result[stack.pop()] = nums[i % n]
        if i < n: stack.push(i)
    
    return result
```

---

## 📊 Monotonic Stack Summary

| Problem | Stack Type | Result |
|---------|-----------|--------|
| Next greater element | Decreasing | Next larger value to right |
| Daily temperatures | Decreasing | Days until warmer |
| Next smaller element | Increasing | Next smaller value to right |
| Largest rectangle | Increasing | Max area using each bar as height |
| Trapping rain water | Decreasing (alt approach) | Water above each bar |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Monotonic stack | Maintain sorted invariant by popping violations |
| O(n) guarantee | Each element pushed/popped at most once |
| Next greater | Decreasing stack; pop when current > top |
| Largest rectangle | Increasing stack; calculate area on pop |
| Circular variant | Process array twice using modulo indexing |

---

## ❓ Revision Questions

1. **Why is monotonic stack O(n)?** → Each element is pushed once and popped once, total 2n operations.
2. **Next greater vs next smaller: difference?** → Greater uses decreasing stack; smaller uses increasing stack.
3. **Largest rectangle: why sentinel at end?** → Forces all remaining bars to be popped and their areas calculated.
4. **Width calculation: `i - stack.top() - 1`?** → The bar extends from the bar on top of stack (exclusive) to current index (exclusive).
5. **Circular next greater: why 2n iterations?** → Simulates wrapping around the array; second pass catches elements whose next greater is before them.

---

[← Previous: Stack Fundamentals](01-stack-fundamentals.md) | [Next: Queue Patterns →](03-queue-patterns.md)
