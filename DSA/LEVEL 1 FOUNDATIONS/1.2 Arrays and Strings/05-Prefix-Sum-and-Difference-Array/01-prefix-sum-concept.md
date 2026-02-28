# Chapter 1: Prefix Sum Concept

[← Previous Unit: Window Expansion and Contraction](../04-Sliding-Window/06-window-expansion-contraction.md) | [Back to README](../README.md) | [Next: Building Prefix Sum Array →](02-building-prefix-sum.md)

---

## Overview

A **prefix sum** (also called **cumulative sum**) is a derived array where each element stores the sum of all elements from the start up to that index. It's one of the most powerful preprocessing techniques in competitive programming and interviews.

---

## Core Idea

```
  Original:   arr = [3, 1, 4, 1, 5, 9]
  
  Prefix Sum: pre = [3, 4, 8, 9, 14, 23]
                     │  │  │  │   │   │
                     │  │  │  │   │   └─ 3+1+4+1+5+9 = 23
                     │  │  │  │   └── 3+1+4+1+5 = 14
                     │  │  │  └─── 3+1+4+1 = 9
                     │  │  └──── 3+1+4 = 8
                     │  └───── 3+1 = 4
                     └────── 3

  pre[i] = arr[0] + arr[1] + ... + arr[i]
```

---

## Why Prefix Sums?

```
  PROBLEM: Find the sum of arr[2..4]   (elements at index 2, 3, 4)

  WITHOUT prefix sum:
    sum = arr[2] + arr[3] + arr[4]
    sum = 4 + 1 + 5 = 10
    Time: O(length of range)

  WITH prefix sum:
    sum = pre[4] - pre[1]     ← CONSTANT TIME!
    sum = 14 - 4 = 10
    Time: O(1)

  ┌──────────────────────────────────────────────────┐
  │  Prefix sum converts range sum queries from      │
  │  O(n) per query to O(1) per query!               │
  │                                                  │
  │  Trade-off: O(n) preprocessing + O(1) per query  │
  └──────────────────────────────────────────────────┘
```

---

## The Range Sum Formula

$$\text{sum}(l, r) = \text{prefix}[r] - \text{prefix}[l-1]$$

Where:
- $l$ = left index of range
- $r$ = right index of range
- $\text{prefix}[-1] = 0$ (by convention)

```
  arr:    [3,  1,  4,  1,  5,  9]
  index:   0   1   2   3   4   5

  prefix: [3,  4,  8,  9, 14, 23]

  sum(2, 4) = prefix[4] - prefix[1] = 14 - 4 = 10
              ├─────────┤              └───────────┘
              sum(0..4)    minus sum(0..1)

  Visually:
  arr:  [3, 1, | 4, 1, 5, | 9]
               ├──────────┤
               sum(2,4)=10

  prefix[4] = 3+1+4+1+5 = 14   (everything from 0 to 4)
  prefix[1] = 3+1 = 4           (everything from 0 to 1)
  difference = 4+1+5 = 10       (just the range 2 to 4)  ✓
```

---

## Handling Edge Case: l = 0

```
  sum(0, r) = prefix[r] - prefix[-1]
  
  Since prefix[-1] doesn't exist, we define it as 0:
  sum(0, r) = prefix[r] - 0 = prefix[r]

  Common implementation:
  
  Option A: Handle l=0 separately
    IF l == 0: RETURN prefix[r]
    ELSE: RETURN prefix[r] - prefix[l-1]

  Option B: Use 1-indexed prefix array with prefix[0] = 0
    prefix = [0, 3, 4, 8, 9, 14, 23]
              ^
              dummy zero at index 0
    
    sum(l, r) = prefix[r+1] - prefix[l]    ← always works!
```

---

## Intuitive Analogy

```
  Think of prefix sums like ODOMETER READINGS:

  ┌──────────────────────────────────────────────┐
  │                                              │
  │  City A ──5km──> City B ──3km──> City C      │
  │                                              │
  │  Odometer at A: 100 km                       │
  │  Odometer at B: 105 km                       │
  │  Odometer at C: 108 km                       │
  │                                              │
  │  Distance A→C = 108 - 100 = 8 km            │
  │  Distance B→C = 108 - 105 = 3 km            │
  │                                              │
  │  The odometer is your PREFIX SUM!            │
  │  Any segment distance = end reading          │
  │                        − start reading       │
  └──────────────────────────────────────────────┘
```

---

## When to Use Prefix Sums

```
  ✓ Multiple range sum queries on a STATIC array
  ✓ Finding subarray with given sum
  ✓ Counting subarrays with sum = k
  ✓ Equilibrium index problems
  ✓ 2D range queries (with 2D prefix sum)

  ✗ Array is frequently UPDATED (use Segment Tree / BIT)
  ✗ Need range MIN/MAX (use Sparse Table / Segment Tree)
  ✗ Single query only (just iterate — no preprocessing needed)
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Build prefix array | O(n) | O(n) |
| Answer range sum query | O(1) | - |
| Q queries on static array | O(n + Q) | O(n) |
| Brute force Q queries | O(n × Q) | O(1) |

```
  For n = 10⁶ and Q = 10⁵:
  
  Brute force: 10⁶ × 10⁵ = 10¹¹ → TLE
  Prefix sum:  10⁶ + 10⁵ = ~10⁶  → fast!
```

---

## Prefix Sum Variants

```
  ┌──────────────────┬──────────────────────────────────────┐
  │ Variant          │ Definition                           │
  ├──────────────────┼──────────────────────────────────────┤
  │ Prefix Sum       │ pre[i] = arr[0] + ... + arr[i]      │
  │ Prefix Product   │ pre[i] = arr[0] × ... × arr[i]      │
  │ Prefix XOR       │ pre[i] = arr[0] ⊕ ... ⊕ arr[i]      │
  │ Prefix Max       │ pre[i] = max(arr[0], ..., arr[i])   │
  │ Prefix Min       │ pre[i] = min(arr[0], ..., arr[i])   │
  │ Suffix Sum       │ suf[i] = arr[i] + ... + arr[n-1]    │
  └──────────────────┴──────────────────────────────────────┘
  
  Note: Range queries with O(1) only work for SUM and XOR
  (because they have inverse operations: subtraction and XOR).
  Max and Min prefix arrays do NOT support range queries.
```

---

## Real-World Applications

| Application | Description |
|------------|-------------|
| Financial analysis | Running balance, cumulative revenue |
| Image processing | Summed area table (2D prefix sum) |
| Database queries | Aggregate over date ranges |
| Game development | Cumulative score tracking |
| Statistics | Cumulative distribution function (CDF) |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| What is prefix sum | Running cumulative sum of array |
| Formula | pre[i] = pre[i-1] + arr[i] |
| Range sum formula | sum(l,r) = pre[r] - pre[l-1] |
| Build time | O(n) |
| Query time | O(1) |
| Space | O(n) |
| Key insight | Preprocessing enables instant range queries |

---

## Quick Revision Questions

1. **What is the mathematical formula** for prefix[i] in terms of prefix[i-1] and arr[i]?

2. **How do you compute sum(l, r)** using the prefix array? What's the edge case when l = 0?

3. **Why can't you do range MAX queries** with a prefix max array?

4. **What is the trade-off** of using prefix sums vs brute force?

5. **Given arr = [2, 4, 6, 8, 10], compute the prefix sum array.** Then find sum(1, 3).

6. **When should you NOT use prefix sums?** Name two scenarios.

---

[← Previous Unit: Window Expansion and Contraction](../04-Sliding-Window/06-window-expansion-contraction.md) | [Back to README](../README.md) | [Next: Building Prefix Sum Array →](02-building-prefix-sum.md)
