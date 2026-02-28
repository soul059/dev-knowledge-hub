# Chapter 3: Monotonically Decreasing Stack

[← Previous: Increasing Stack](02-increasing-stack.md) | [Next: Next Greater Element →](04-next-greater-element.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)

---

## Overview

A **monotonically decreasing stack** maintains elements in **descending order** from bottom to top. When a new element is **larger** than the stack top, we pop elements until the monotonic property is restored. This type is used to find the **Next Greater Element (NGE)** and **Previous Greater Element (PGE)**.

---

## Definition

```
┌──────────────────────────────────────────────────────┐
│         MONOTONICALLY DECREASING STACK               │
│                                                      │
│  Property: bottom ≥ ... ≥ top                        │
│                                                      │
│  ┌───┐                                               │
│  │ 2 │ ← top (smallest)                              │
│  │ 4 │                                               │
│  │ 7 │                                               │
│  │ 9 │ ← bottom (largest)                            │
│  └───┘                                               │
│                                                      │
│  Pop WHEN: incoming element > stack.top()             │
│  Popped element's answer: the incoming element       │
│  Remaining stack.top() after pushing: prev greater   │
└──────────────────────────────────────────────────────┘
```

---

## Building the Stack

### Input: [5, 3, 8, 4, 2, 7, 1]

```
Step 1: Push 5
  Stack: [5]    ✓ trivially decreasing

Step 2: Push 3 (3 < 5 ✓ decreasing)
  Stack: [5, 3]

Step 3: Push 8 (8 > 3 ✗ violates!)
  Pop 3 → Next Greater of 3 is 8
  Pop 5 → Next Greater of 5 is 8
  Push 8
  Stack: [8]

Step 4: Push 4 (4 < 8 ✓)
  Stack: [8, 4]

Step 5: Push 2 (2 < 4 ✓)
  Stack: [8, 4, 2]

Step 6: Push 7 (7 > 2 ✗ violates!)
  Pop 2 → Next Greater of 2 is 7
  Pop 4 → Next Greater of 4 is 7
  7 < 8 ✓ → Push 7
  Stack: [8, 7]

Step 7: Push 1 (1 < 7 ✓)
  Stack: [8, 7, 1]

Summary of next greater elements found:
  3 → 8,  5 → 8,  2 → 7,  4 → 7
  8, 7, 1 → no next greater (remained in stack)
```

---

## Application: Next Greater Element

```
FUNCTION nextGreaterElement(arr):
    n ← length(arr)
    result ← array of size n, fill with -1
    stack ← empty stack    // stores indices
    
    FOR i = 0 TO n-1:
        WHILE stack is NOT empty AND arr[i] > arr[stack.top()]:
            idx ← stack.pop()
            result[idx] ← arr[i]
        stack.push(i)
    
    RETURN result
```

### Trace: arr = [2, 1, 2, 4, 3]

```
┌──────┬───────┬───────────────────────────┬──────────┬────────────────┐
│ i    │ arr[i]│ Action                    │ Stack(i) │ result         │
├──────┼───────┼───────────────────────────┼──────────┼────────────────┤
│  0   │  2    │ Push 0                    │ [0]      │ [-1,-1,-1,-1,-1]│
│  1   │  1    │ 1<2 → Push 1              │ [0,1]    │ [-1,-1,-1,-1,-1]│
│  2   │  2    │ 2>1 → Pop 1, res[1]=2     │ [0,2]    │ [-1, 2,-1,-1,-1]│
│      │       │ 2≤2 → Push 2              │          │                │
│  3   │  4    │ 4>2 → Pop 2, res[2]=4     │ [3]      │ [-1, 2, 4,-1,-1]│
│      │       │ 4>2 → Pop 0, res[0]=4     │          │                │
│      │       │ Push 3                    │          │                │
│  4   │  3    │ 3<4 → Push 4              │ [3,4]    │ [-1, 2, 4,-1,-1]│
└──────┴───────┴───────────────────────────┴──────────┴────────────────┘

Remaining: indices 3,4 → -1

Result: [4, 2, 4, -1, -1]

Verify:
  2 → next greater: 4 ✓    (skips 1 and 2)
  1 → next greater: 2 ✓
  2 → next greater: 4 ✓
  4 → none:        -1 ✓
  3 → none:        -1 ✓
```

---

## Application: Previous Greater Element

```
FUNCTION prevGreaterElement(arr):
    n ← length(arr)
    result ← array of size n, fill with -1
    stack ← empty stack
    
    FOR i = 0 TO n-1:
        WHILE stack is NOT empty AND arr[stack.top()] <= arr[i]:
            stack.pop()
        
        IF stack is NOT empty:
            result[i] ← arr[stack.top()]
        
        stack.push(i)
    
    RETURN result
```

### Trace: arr = [10, 4, 2, 20, 40, 12, 30]

```
i=0: 10, empty → result[0]=-1, push 0        Stack: [0]
i=1: 4, top=10>4 → result[1]=10, push 1       Stack: [0,1]
i=2: 2, top=4>2 → result[2]=4, push 2         Stack: [0,1,2]
i=3: 20, pop 2(2≤20), pop 1(4≤20), pop 0(10≤20)
     empty → result[3]=-1, push 3              Stack: [3]
i=4: 40, pop 3(20≤40)
     empty → result[4]=-1, push 4              Stack: [4]
i=5: 12, top=40>12 → result[5]=40, push 5     Stack: [4,5]
i=6: 30, pop 5(12≤30), top=40>30
     → result[6]=40, push 6                    Stack: [4,6]

Result: [-1, 10, 4, -1, -1, 40, 40]
```

---

## Increasing vs Decreasing: Side-by-Side

```
┌────────────────────────┬────────────────────┬────────────────────┐
│ Property               │ Increasing Stack   │ Decreasing Stack   │
├────────────────────────┼────────────────────┼────────────────────┤
│ Order (bottom→top)     │ Small → Large      │ Large → Small      │
│ Pop when incoming is   │ Smaller than top   │ Larger than top    │
│ Finds (on pop)         │ Next Smaller       │ Next Greater       │
│ Remaining top after    │ Previous Smaller   │ Previous Greater   │
│ push                   │ of current         │ of current         │
│ Elements stay if       │ Smaller or equal   │ Larger or equal    │
│                        │ to incoming        │ to incoming        │
└────────────────────────┴────────────────────┴────────────────────┘
```

---

## When to Use Decreasing Stack

```
Use decreasing stack when the problem involves:

  ✓ Finding Next Greater Element
  ✓ Finding Previous Greater Element
  ✓ Stock span (consecutive days price was ≤ today)
  ✓ Daily temperatures (days until warmer)
  ✓ Maximum area histogram (uses both types!)
  ✓ Problems asking about "first larger" element
```

---

## Complexity Analysis

| Aspect | Complexity |
|--------|-----------|
| **Time** | O(n) — each element pushed/popped at most once |
| **Space** | O(n) — stack may hold all elements |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Stack Order** | Descending from bottom to top |
| **Pop Condition** | Incoming > stack top |
| **Finds (on pop)** | Next Greater Element |
| **Also Finds** | Previous Greater Element |
| **Time** | O(n) |
| **Space** | O(n) |

---

## Quick Revision Questions

1. **What order does a decreasing stack maintain?**
   > Elements decrease from bottom to top (largest at bottom, smallest at top).

2. **When do we pop from a decreasing stack?**
   > When the incoming element is greater than the stack top.

3. **What does the incoming element represent for popped elements?**
   > It is the "next greater element" for each popped element.

4. **How does the decreasing stack find the Previous Greater Element?**
   > After popping all smaller/equal elements, the remaining stack top is the previous greater element.

5. **What is the key difference between increasing and decreasing monotonic stacks?**
   > Increasing stack finds next/prev *smaller*; decreasing stack finds next/prev *greater*.

---

[← Previous: Increasing Stack](02-increasing-stack.md) | [Next: Next Greater Element →](04-next-greater-element.md) | [↑ Back to Unit 5](../README.md#unit-5-monotonic-stack) | [🏠 Home](../README.md)
