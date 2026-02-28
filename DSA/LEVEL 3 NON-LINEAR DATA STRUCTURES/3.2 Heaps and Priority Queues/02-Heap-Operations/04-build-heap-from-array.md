# 2.4 Build Heap from Array

[← Previous: Peek Operation](03-peek-operation.md) | [Next: Delete & Replace →](05-delete-and-replace.md)

---

## Overview

Given an unsorted array, rearrange it into a valid heap **in-place**. Floyd's bottom-up algorithm achieves this in O(n) — better than the O(n log n) naive approach.

---

## The Problem

Transform an arbitrary array into one that satisfies the heap property, using no extra space.

---

## Naive Approach — Repeated Insert

```
For each element in array:
    Insert into empty heap    // O(log n) each

Total: O(n log n)
```

---

## Optimal Approach — Bottom-Up Heapify (Floyd's Algorithm)

```
BUILD_HEAP(array):
    n = array.length
    // Start from last non-leaf node, go backwards to root
    FOR i = (n/2 - 1) DOWN TO 0:
        SIFT_DOWN(array, i, n)
```

**Key Insight**: Leaf nodes (indices n/2 to n-1) are **already valid heaps** of size 1. We only need to fix internal nodes.

---

## Step-by-Step Trace — Build Max-Heap

```
Input Array: [4, 10, 3, 5, 1]

Tree form (NOT a heap yet):
          4
         / \
       10    3
      / \
     5   1

n = 5, last non-leaf = 5/2 - 1 = 1

─── i = 1 (value = 10) ───
Children: 5 (index 3), 1 (index 4)
largest = 10 (itself)
10 ≥ 5 and 10 ≥ 1 → No swap needed

          4
         / \
       10    3
      / \
     5   1

─── i = 0 (value = 4) ───
Children: 10 (index 1), 3 (index 2)
largest = 10 (index 1)
4 < 10 → SWAP(4, 10)

          10
         / \
        4    3
      / \
     5   1

Continue sifting 4 down at index 1:
Children: 5 (index 3), 1 (index 4)
largest = 5 (index 3)
4 < 5 → SWAP(4, 5)

          10
         / \
        5    3
      / \
     4   1

4 at index 3 has no children → STOP

Final Max-Heap: [10, 5, 3, 4, 1] ✅

          10
         / \
        5    3
      / \
     4   1
```

---

## Time Complexity

| Approach | Time |
|----------|------|
| Naive (repeated insert) | O(n log n) |
| **Bottom-up (Floyd's)** | **O(n)** |

**Intuition**: Most nodes are near the bottom of the tree. Leaves (about n/2 nodes) do zero work. Nodes one level above do at most 1 swap. Only the root does ⌊log n⌋ work. The total sums to O(n).

> The detailed mathematical proof is in [Unit 3: Heapify Process](../03-Heapify-Process/04-build-heap-on-proof.md).

---

## Quick Check

1. **What is the time complexity of building a heap using repeated insertions vs Floyd's algorithm?**
   <details><summary>Answer</summary>
   Repeated insertions: O(n log n). Floyd's bottom-up algorithm: O(n).
   </details>

2. **Why does Floyd's algorithm start from the last non-leaf instead of the root?**
   <details><summary>Answer</summary>
   Starting from the bottom ensures that when we sift down a node, both its subtrees are already valid heaps. This is the invariant that keeps the algorithm correct.
   </details>

---

[← Previous: Peek Operation](03-peek-operation.md) | [Next: Delete & Replace →](05-delete-and-replace.md)
