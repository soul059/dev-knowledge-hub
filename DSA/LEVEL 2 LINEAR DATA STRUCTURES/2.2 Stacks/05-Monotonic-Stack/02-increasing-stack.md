# Chapter 2: Monotonically Increasing Stack

[← Previous: Monotonic Stack Intro](01-monotonic-stack-intro.md) | [Next: Monotonically Decreasing Stack →](03-decreasing-stack.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)

---

## Overview

A **monotonically increasing stack** maintains elements in **ascending order** from bottom to top. When a new element is **smaller** than the stack top, we pop elements until the monotonic property is restored. This type is primarily used to find the **Next Smaller Element (NSE)** and **Previous Smaller Element (PSE)**.

---

## Definition

```
┌──────────────────────────────────────────────────────┐
│         MONOTONICALLY INCREASING STACK               │
│                                                      │
│  Property: bottom ≤ ... ≤ top                        │
│                                                      │
│  ┌───┐                                               │
│  │ 9 │ ← top (largest)                               │
│  │ 7 │                                               │
│  │ 4 │                                               │
│  │ 2 │ ← bottom (smallest)                           │
│  └───┘                                               │
│                                                      │
│  Pop WHEN: incoming element < stack.top()             │
│  Popped element's answer: the incoming element       │
│  Remaining stack.top() after pushing: prev smaller   │
└──────────────────────────────────────────────────────┘
```

---

## Building the Stack

### Input: [2, 5, 3, 7, 4, 1, 6]

```
Step 1: Push 2
  Stack: [2]
  ┌───┐
  │ 2 │
  └───┘

Step 2: Push 5 (5 > 2 ✓ increasing)
  Stack: [2, 5]
  ┌───┐
  │ 5 │
  │ 2 │
  └───┘

Step 3: Push 3 (3 < 5 ✗ violates!)
  Pop 5 → Next Smaller of 5 is 3
  3 > 2 ✓ → Push 3
  Stack: [2, 3]
  ┌───┐
  │ 3 │
  │ 2 │
  └───┘

Step 4: Push 7 (7 > 3 ✓)
  Stack: [2, 3, 7]
  ┌───┐
  │ 7 │
  │ 3 │
  │ 2 │
  └───┘

Step 5: Push 4 (4 < 7 ✗ violates!)
  Pop 7 → Next Smaller of 7 is 4
  4 > 3 ✓ → Push 4
  Stack: [2, 3, 4]
  ┌───┐
  │ 4 │
  │ 3 │
  │ 2 │
  └───┘

Step 6: Push 1 (1 < 4 ✗ violates!)
  Pop 4 → Next Smaller of 4 is 1
  Pop 3 → Next Smaller of 3 is 1
  Pop 2 → Next Smaller of 2 is 1
  Stack empty → Push 1
  Stack: [1]
  ┌───┐
  │ 1 │
  └───┘

Step 7: Push 6 (6 > 1 ✓)
  Stack: [1, 6]
  ┌───┐
  │ 6 │
  │ 1 │
  └───┘
```

---

## Application: Next Smaller Element

For each element, find the first smaller element to its **right**.

```
FUNCTION nextSmallerElement(arr):
    n ← length(arr)
    result ← array of size n, fill with -1
    stack ← empty stack    // stores indices
    
    FOR i = 0 TO n-1:
        WHILE stack is NOT empty AND arr[i] < arr[stack.top()]:
            idx ← stack.pop()
            result[idx] ← arr[i]
        stack.push(i)
    
    RETURN result
```

### Trace: arr = [4, 8, 5, 2, 25]

```
┌──────┬───────┬────────────────────────┬──────────┬──────────────────┐
│ i    │ arr[i]│ Action                 │ Stack(i) │ result           │
├──────┼───────┼────────────────────────┼──────────┼──────────────────┤
│  0   │  4    │ Push 0                 │ [0]      │ [-1,-1,-1,-1,-1] │
│  1   │  8    │ 8>4 → Push 1           │ [0,1]    │ [-1,-1,-1,-1,-1] │
│  2   │  5    │ 5<8 → Pop 1, res[1]=5  │ [0,2]    │ [-1, 5,-1,-1,-1] │
│      │       │ 5>4 → Push 2           │          │                  │
│  3   │  2    │ 2<5 → Pop 2, res[2]=2  │ [3]      │ [-1, 5, 2,-1,-1] │
│      │       │ 2<4 → Pop 0, res[0]=2  │          │                  │
│      │       │ Push 3                 │          │                  │
│  4   │  25   │ 25>2 → Push 4          │ [3,4]    │ [-1, 5, 2,-1,-1] │
└──────┴───────┴────────────────────────┴──────────┴──────────────────┘

Remaining in stack: indices 3,4 → no smaller element to right → -1

Result: [2, 5, 2, -1, -1]

Verification:
  4  → next smaller to right: 2  ✓
  8  → next smaller to right: 5  ✓
  5  → next smaller to right: 2  ✓
  2  → no smaller to right:  -1  ✓
  25 → no smaller to right:  -1  ✓
```

---

## Application: Previous Smaller Element

For each element, find the first smaller element to its **left**.

```
FUNCTION prevSmallerElement(arr):
    n ← length(arr)
    result ← array of size n, fill with -1
    stack ← empty stack
    
    FOR i = 0 TO n-1:
        WHILE stack is NOT empty AND arr[stack.top()] >= arr[i]:
            stack.pop()    // Remove elements ≥ current
        
        IF stack is NOT empty:
            result[i] ← arr[stack.top()]    // Top is prev smaller
        
        stack.push(i)
    
    RETURN result
```

### Trace: arr = [4, 8, 5, 2, 25]

```
i=0: arr[0]=4, stack empty → result[0]=-1, push 0
     Stack: [0]

i=1: arr[1]=8, top=4 < 8 → result[1]=4, push 1
     Stack: [0,1]

i=2: arr[2]=5, top=8 ≥ 5 → pop 1
     top=4 < 5 → result[2]=4, push 2
     Stack: [0,2]

i=3: arr[3]=2, top=5 ≥ 2 → pop 2
     top=4 ≥ 2 → pop 0
     empty → result[3]=-1, push 3
     Stack: [3]

i=4: arr[4]=25, top=2 < 25 → result[4]=2, push 4
     Stack: [3,4]

Result: [-1, 4, 4, -1, 2]
```

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n) | Each element pushed and popped at most once |
| **Space** | O(n) | Stack may hold all elements |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Stack Order** | Ascending from bottom to top |
| **Pop Condition** | Incoming element < stack top |
| **Finds** | Next Smaller Element (right) |
| **Also Finds** | Previous Smaller Element (left) |
| **Time** | O(n) |
| **Space** | O(n) |
| **When Popped** | Incoming element is the Next Smaller |
| **Stack Top After Push** | Previous Smaller of current |

---

## Quick Revision Questions

1. **What order does a monotonically increasing stack maintain?**
   > Elements increase from bottom to top (smallest at bottom, largest at top).

2. **When do we pop from an increasing stack?**
   > When the incoming element is smaller than the stack top (violates increasing order).

3. **What does the popped element's answer represent?**
   > The incoming element is the "next smaller element" for the popped element.

4. **How do we find the Previous Smaller Element using an increasing stack?**
   > After popping elements ≥ current, the stack top (if exists) is the previous smaller element.

5. **What happens to elements remaining on the stack after processing all input?**
   > They have no next smaller element to their right; their answer is -1.

---

[← Previous: Monotonic Stack Intro](01-monotonic-stack-intro.md) | [Next: Monotonically Decreasing Stack →](03-decreasing-stack.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)
