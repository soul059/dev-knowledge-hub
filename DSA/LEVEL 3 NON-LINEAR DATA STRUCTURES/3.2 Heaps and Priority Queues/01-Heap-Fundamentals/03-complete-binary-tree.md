# 1.3 Complete Binary Tree Requirement

[← Previous: Heap Property](02-heap-property.md) | [Next: Array Representation →](04-array-representation.md)

---

## Overview

A heap must be a **complete binary tree**. This structural constraint is what allows efficient array-based storage and guarantees O(log n) operations.

---

## Definition

A **complete binary tree** satisfies:

1. All levels are **fully filled**, except possibly the last level
2. The last level is filled from **left to right** (no gaps)

---

## Valid Complete Binary Trees

```
        ✅ Valid                    ✅ Valid
        
           1                          1
          / \                        / \
         2   3                      2   3
        / \                        / \ / \
       4   5                      4  5 6  7
                                 /
                                8
```

---

## Invalid (NOT Complete)

```
        ❌ Invalid                  ❌ Invalid
        
           1                          1
          / \                        / \
         2   3                      2   3
        / \   \                      \
       4   5   7  ← gap at 6!        5  ← gap at 4!
```

---

## Why Complete Binary Tree?

| Reason | Explanation |
|--------|-------------|
| **Height guarantee** | Complete binary tree of n nodes has height = ⌊log₂ n⌋ |
| **Array storage** | No wasted space — nodes fill array positions contiguously |
| **Cache friendly** | Array storage means sequential memory access |
| **Predictable structure** | Shape is fully determined by the number of elements |

---

## Key Formulas

| Property | Formula |
|----------|---------|
| Height | h = ⌊log₂ n⌋ |
| Max nodes at height h | 2^(h+1) - 1 |
| Min nodes at height h | 2^h |
| Nodes at level k | 2^k (for fully filled level) |

---

## Quick Check

1. **Why can't a non-complete binary tree be efficiently stored as an array?**
   <details><summary>Answer</summary>
   Gaps in the tree would require null/sentinel values in the array, wasting space and breaking the simple parent-child index formulas.
   </details>

2. **What is the maximum number of elements in a heap of height h?**
   <details><summary>Answer</summary>
   A complete binary tree of height h has at most 2^(h+1) - 1 nodes (when all levels are fully filled).
   </details>

---

[← Previous: Heap Property](02-heap-property.md) | [Next: Array Representation →](04-array-representation.md)
