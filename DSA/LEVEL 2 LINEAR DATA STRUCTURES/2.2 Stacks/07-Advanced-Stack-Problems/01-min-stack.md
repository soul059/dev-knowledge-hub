# Chapter 1: Min Stack

[← Previous: Optimizing Brute Force](../06-Stock-Span-Problems/06-optimizing-brute-force.md) | [Next: Max Stack →](02-max-stack.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)

---

## Overview

The **Min Stack** (LeetCode #155) supports standard stack operations **plus** retrieving the minimum element — all in **O(1)** time. This is a fundamental design problem that teaches how to augment data structures with additional capabilities.

---

## Problem Statement

```
Design a stack that supports:
  push(x)  — Push element x onto stack
  pop()    — Remove top element
  top()    — Get top element
  getMin() — Retrieve minimum element

ALL operations must be O(1) time.
```

---

## Key Challenge

```
┌──────────────────────────────────────────────────────────┐
│  A normal stack tracks the TOP element.                  │
│  But the MINIMUM can be anywhere in the stack!           │
│                                                          │
│  ┌───┐                                                   │
│  │ 5 │ ← top                                             │
│  │ 2 │ ← minimum (buried!)                               │
│  │ 7 │                                                   │
│  │ 3 │                                                   │
│  └───┘                                                   │
│                                                          │
│  If we pop 5, min is still 2.                            │
│  If we pop 2, min becomes 3.                             │
│  How to track this without scanning?                     │
└──────────────────────────────────────────────────────────┘
```

---

## Approach 1: Two Stacks

Use a second stack to track minimums.

```
CLASS MinStack:
    mainStack ← empty stack
    minStack  ← empty stack    // Tracks min at each level
    
    FUNCTION push(x):
        mainStack.push(x)
        IF minStack is empty OR x <= minStack.top():
            minStack.push(x)
        ELSE:
            minStack.push(minStack.top())    // Repeat current min
    
    FUNCTION pop():
        mainStack.pop()
        minStack.pop()
    
    FUNCTION top():
        RETURN mainStack.top()
    
    FUNCTION getMin():
        RETURN minStack.top()
```

### Trace

```
push(5):  main=[5]      min=[5]       getMin→5
push(2):  main=[5,2]    min=[5,2]     getMin→2
push(7):  main=[5,2,7]  min=[5,2,2]   getMin→2
push(1):  main=[5,2,7,1] min=[5,2,2,1] getMin→1

     Main     Min
    ┌───┐   ┌───┐
    │ 1 │   │ 1 │  ← min at this level
    │ 7 │   │ 2 │  ← min at this level
    │ 2 │   │ 2 │  ← min at this level
    │ 5 │   │ 5 │  ← min at this level
    └───┘   └───┘

pop():    main=[5,2,7]  min=[5,2,2]   getMin→2  ✓
pop():    main=[5,2]    min=[5,2]     getMin→2  ✓
pop():    main=[5]      min=[5]       getMin→5  ✓
```

---

## Approach 2: Space-Optimized Two Stacks

Only push to minStack when the new value is ≤ current minimum.

```
CLASS MinStack_Optimized:
    mainStack ← empty stack
    minStack  ← empty stack
    
    FUNCTION push(x):
        mainStack.push(x)
        IF minStack is empty OR x <= minStack.top():
            minStack.push(x)    // Only push if new min or equal
    
    FUNCTION pop():
        val ← mainStack.pop()
        IF val == minStack.top():
            minStack.pop()      // Only pop if it was a min value
    
    FUNCTION top():
        RETURN mainStack.top()
    
    FUNCTION getMin():
        RETURN minStack.top()
```

### Trace: push 5, 2, 7, 2, 1

```
push(5): main=[5]         min=[5]          ← 5≤∅ → push
push(2): main=[5,2]       min=[5,2]        ← 2≤5 → push
push(7): main=[5,2,7]     min=[5,2]        ← 7>2 → don't push
push(2): main=[5,2,7,2]   min=[5,2,2]      ← 2≤2 → push (equal!)
push(1): main=[5,2,7,2,1] min=[5,2,2,1]    ← 1≤2 → push

pop():   main=[5,2,7,2]  1==1 → min=[5,2,2]  getMin→2 ✓
pop():   main=[5,2,7]    2==2 → min=[5,2]     getMin→2 ✓
pop():   main=[5,2]      7≠2 → min=[5,2]      getMin→2 ✓
pop():   main=[5]        2==2 → min=[5]        getMin→5 ✓
```

---

## Approach 3: Single Stack (Store Difference)

Use encoding to store both value and min information in one stack.

```
CLASS MinStack_SingleStack:
    stack ← empty stack
    minVal ← undefined
    
    FUNCTION push(x):
        IF stack is empty:
            stack.push(0)
            minVal ← x
        ELSE:
            diff ← x - minVal
            stack.push(diff)
            IF diff < 0:
                minVal ← x    // New minimum found
    
    FUNCTION pop():
        diff ← stack.pop()
        IF diff < 0:
            // Current min is being popped, restore previous min
            prevMin ← minVal - diff
            minVal ← prevMin
        IF stack is empty:
            minVal ← undefined
    
    FUNCTION top():
        diff ← stack.top()
        IF diff < 0:
            RETURN minVal     // Actual value IS the min
        ELSE:
            RETURN minVal + diff
    
    FUNCTION getMin():
        RETURN minVal
```

### Key Insight for Single Stack:

```
We store: diff = actual_value - current_min

If diff ≥ 0: value ≥ min → min unchanged
If diff < 0: value < min → NEW minimum found!

When diff < 0 is popped:
  previous_min = current_min - diff
  (because diff = new_val - old_min, so old_min = new_val - diff)
```

---

## Comparison of Approaches

```
┌──────────────────────┬────────────┬────────────┬──────────────┐
│ Approach             │ Time (all) │ Space      │ Caveat       │
├──────────────────────┼────────────┼────────────┼──────────────┤
│ Two stacks (basic)   │ O(1)       │ O(2n)      │ None         │
│ Two stacks (optimized│ O(1)       │ O(n) best  │ Duplicates!  │
│ Single stack (diff)  │ O(1)       │ O(n)       │ Overflow risk│
└──────────────────────┴────────────┴────────────┴──────────────┘
```

---

## Complexity Analysis

| Operation | Time | Space (Two Stacks) |
|-----------|------|--------------------|
| `push` | O(1) | O(1) per push |
| `pop` | O(1) | O(1) per pop |
| `top` | O(1) | — |
| `getMin` | O(1) | — |
| **Overall** | O(1) each | O(n) total |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Stack with O(1) minimum retrieval |
| **Best Approach** | Two stacks (simple) or single stack (space-efficient) |
| **Key Idea** | Track min at each stack level |
| **All operations** | O(1) |
| **LeetCode** | #155 |

---

## Quick Revision Questions

1. **Why can't we just use a single variable to track the minimum?**
   > When the minimum is popped, we need to know the *previous* minimum, which a single variable doesn't track.

2. **In the two-stack approach, why push to minStack even when value > current min?**
   > (Basic version) To keep minStack aligned with mainStack so every pop removes from both. The optimized version avoids this.

3. **In the optimized two-stack approach, why push for equal values (<=, not just <)?**
   > If we have duplicate minimums and only push for strictly less, popping one copy would incorrectly remove the min for remaining copies.

4. **What is the risk of the single-stack (difference) approach?**
   > Integer overflow — if values are very large or very small, the difference `x - minVal` may exceed integer bounds.

5. **Can Min Stack be combined with other operations like getMax?**
   > Yes — use a third stack (or pair of auxiliary stacks) to track both min and max. This leads to the "Min-Max Stack" data structure.

---

[← Previous: Optimizing Brute Force](../06-Stock-Span-Problems/06-optimizing-brute-force.md) | [Next: Max Stack →](02-max-stack.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)
