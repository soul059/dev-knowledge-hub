# Chapter 1: What is Monotonic Stack?

[← Previous: Expression Tree](../04-Expression-Evaluation/06-expression-tree.md) | [Next: Monotonically Increasing Stack →](02-increasing-stack.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)

---

## Overview

A **monotonic stack** is a stack that maintains its elements in either **non-decreasing** or **non-increasing** order from bottom to top. It is a powerful technique for solving problems that require finding the "next greater," "next smaller," or nearest elements satisfying a condition — all in **O(n)** time.

---

## What Makes a Stack "Monotonic"?

```
┌──────────────────────────────────────────────────────────┐
│                  MONOTONIC STACK                         │
│                                                          │
│  A stack where elements are maintained in a              │
│  consistent ORDER from bottom to top.                    │
│                                                          │
│  Monotonically INCREASING (bottom → top):                │
│  ┌───┐                                                   │
│  │ 8 │ ← top (largest)                                   │
│  │ 5 │                                                   │
│  │ 3 │                                                   │
│  │ 1 │ ← bottom (smallest)                               │
│  └───┘  Each element ≥ element below it                  │
│                                                          │
│  Monotonically DECREASING (bottom → top):                │
│  ┌───┐                                                   │
│  │ 1 │ ← top (smallest)                                  │
│  │ 3 │                                                   │
│  │ 5 │                                                   │
│  │ 8 │ ← bottom (largest)                                │
│  └───┘  Each element ≤ element below it                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## How It Works

When pushing a new element, we **pop elements** that violate the monotonic property:

```
Maintaining DECREASING stack (big at bottom, small at top):

Input: [3, 1, 4, 1, 5]

Push 3:  Stack: [3]          ✓ monotonic
Push 1:  Stack: [3, 1]      ✓ 1 ≤ 3
Push 4:  4 > 1? Pop 1       Stack: [3]
         4 > 3? Pop 3       Stack: []
         Push 4              Stack: [4]          ✓
Push 1:  Stack: [4, 1]      ✓ 1 ≤ 4
Push 5:  5 > 1? Pop 1       Stack: [4]
         5 > 4? Pop 4       Stack: []
         Push 5              Stack: [5]          ✓

Key insight: When an element is POPPED, the incoming 
element is its "next greater element"!
```

---

## The Two Types

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  INCREASING Stack         │  DECREASING Stack               │
│  (ascending bottom→top)   │  (descending bottom→top)        │
│                           │                                 │
│  ┌───┐                    │  ┌───┐                          │
│  │ 7 │ top                │  │ 2 │ top                      │
│  │ 5 │                    │  │ 5 │                          │
│  │ 3 │                    │  │ 7 │                          │
│  │ 1 │ bottom             │  │ 9 │ bottom                   │
│  └───┘                    │  └───┘                          │
│                           │                                 │
│  Pop when: incoming <     │  Pop when: incoming >           │
│            stack top      │            stack top             │
│                           │                                 │
│  Finds: Next SMALLER      │  Finds: Next GREATER            │
│         element           │         element                 │
│                           │                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Generic Template

```
FUNCTION monotonicStack(arr):
    stack ← empty stack
    result ← array of size n, initialized to -1
    
    FOR i = 0 TO n-1:
        // Pop elements that violate monotonic property
        WHILE stack is NOT empty AND condition(arr[i], arr[stack.top()]):
            idx ← stack.pop()
            result[idx] ← arr[i]    // arr[i] is the answer for arr[idx]
        
        stack.push(i)    // Push INDEX, not value
    
    RETURN result
```

### Important: Push Indices, Not Values

```
We push INDICES onto the stack (not values) because:
  1. We need to know WHICH element the answer belongs to
  2. We can always look up the value using arr[index]
  3. Some problems need the index for distance calculation
```

---

## Why O(n)?

```
┌──────────────────────────────────────────────────────────┐
│  Despite the nested loop (for + while), it's O(n):      │
│                                                          │
│  Each element is:                                        │
│    - Pushed onto the stack EXACTLY ONCE                  │
│    - Popped from the stack AT MOST ONCE                  │
│                                                          │
│  Total operations = n pushes + at most n pops = 2n       │
│  Therefore: O(n)                                         │
│                                                          │
│  This is called AMORTIZED O(1) per element.              │
│                                                          │
│  ┌───────────────────────────────────────┐               │
│  │ Element │  Pushed │  Popped │ Total   │               │
│  ├─────────┼─────────┼─────────┼─────────┤               │
│  │ arr[0]  │    1    │  0 or 1 │         │               │
│  │ arr[1]  │    1    │  0 or 1 │         │               │
│  │  ...    │   ...   │   ...   │         │               │
│  │ arr[n-1]│    1    │  0 or 1 │         │               │
│  ├─────────┼─────────┼─────────┼─────────┤               │
│  │ Total   │    n    │   ≤ n   │  ≤ 2n   │               │
│  └───────────────────────────────────────┘               │
└──────────────────────────────────────────────────────────┘
```

---

## Problems Solved by Monotonic Stack

```
┌──────────────────────────────────────────────────────────┐
│  1. Next Greater Element (NGE)                          │
│  2. Next Smaller Element (NSE)                          │
│  3. Previous Greater Element (PGE)                      │
│  4. Previous Smaller Element (PSE)                      │
│  5. Stock Span Problem                                  │
│  6. Maximum Area Histogram                              │
│  7. Largest Rectangle in Binary Matrix                  │
│  8. Trapping Rain Water                                 │
│  9. Daily Temperatures                                  │
│  10. Remove K Digits                                    │
│  11. Sum of Subarray Minimums                           │
│  12. Sliding Window Maximum (with deque variant)        │
└──────────────────────────────────────────────────────────┘
```

---

## Monotonic Stack vs Brute Force

```
Problem: Next Greater Element for each element in array

Brute Force:
  FOR each element:
    Scan all elements to the right
  Time: O(n²)

Monotonic Stack:
  Single pass through array
  Time: O(n)

Example: arr = [4, 5, 2, 10, 8]

Brute Force:                    Monotonic Stack:
  4 → scan [5,2,10,8] → 5       Process in one pass
  5 → scan [2,10,8]   → 10      Each element pushed once
  2 → scan [10,8]     → 10      Each element popped once
  10 → scan [8]       → -1      Total: O(n) ✓
  8 → scan []         → -1
  Comparisons: 4+3+2+1+0 = 10

For large n, the difference is massive:
  n = 10,000:   Brute = ~50M ops vs Stack = ~20K ops
  n = 1,000,000: Brute = ~500B ops vs Stack = ~2M ops
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Definition** | Stack maintaining monotonic order |
| **Types** | Increasing (ascending) and Decreasing (descending) |
| **Time Complexity** | O(n) amortized |
| **Space Complexity** | O(n) worst case |
| **Key Insight** | When element is popped, the incoming element answers its query |
| **Store** | Indices (not values) on the stack |
| **Applications** | NGE, NSE, histograms, rain water, stock span |

---

## Quick Revision Questions

1. **What does "monotonic" mean in the context of a stack?**
   > Elements are maintained in a consistent order (either non-decreasing or non-increasing) from bottom to top.

2. **Why is the time complexity O(n) despite the nested while loop?**
   > Each element is pushed exactly once and popped at most once, giving a total of at most 2n operations.

3. **Why do we push indices instead of values?**
   > To track which element in the original array the answer belongs to, and to calculate distances when needed.

4. **Which type of monotonic stack finds the next greater element?**
   > A monotonically decreasing stack (pops when incoming element is greater than top).

5. **How does a monotonic stack improve over brute force?**
   > From O(n²) (checking all pairs) to O(n) (each element processed at most twice: once pushed, once popped).

---

[← Previous: Expression Tree](../04-Expression-Evaluation/06-expression-tree.md) | [Next: Monotonically Increasing Stack →](02-increasing-stack.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)
