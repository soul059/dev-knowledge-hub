# 3.6 Edge Cases

[← Previous: Build Max-Heap Trace](05-build-max-heap-trace.md) | [Next Unit: Heap Sort →](../04-Heap-Sort/01-heap-sort-algorithm.md)

---

## Overview

Understanding edge cases ensures your heap implementation handles all inputs correctly.

---

## 1. Empty Heap

```
BUILD_HEAP([]) → nothing to do
```

---

## 2. Single Element

```
BUILD_HEAP([42]) → already a valid heap (leaf node)
```

---

## 3. All Same Elements

```
BUILD_HEAP([5, 5, 5, 5]) → already a valid heap (all equal)
Any permutation is valid since parent = child everywhere.
```

---

## 4. Already a Valid Heap

```
BUILD_HEAP([9, 8, 7, 6, 5]) → No swaps needed
Each sift-down compares but finds no violations.
Still O(n) — we must check all internal nodes.
```

---

## 5. Reverse Sorted (Worst Case for Max-Heap Build)

```
BUILD_HEAP([1, 2, 3, 4, 5, 6, 7])
Every internal node must sift all the way down → maximum work.
But still O(n) total!
```

---

## 6. Node with Only One Child

```
When n is even, the last internal node has only a left child:

       10
      / \
    20   30
   /
  40

Node 20 (index 1) has:
  left  = 2*1+1 = 3 (exists: 40)
  right = 2*1+2 = 4 (does NOT exist!)

Must check bounds before comparing right child.
```

---

## Unit 3 Summary Table

| Concept | Key Point |
|---------|-----------|
| Sift Up | Move element toward root; used in insert; O(log n) |
| Sift Down | Move element toward leaves; used in extract & build; O(log n) |
| Build Heap (bottom-up) | Sift down all internal nodes from bottom; O(n) |
| Build Heap (top-down) | Sift up each element as inserted; O(n log n) |
| O(n) proof insight | Leaves do 0 work; most work is at top where few nodes exist |
| When to sift up | Element might be too large for its position (max-heap) |
| When to sift down | Element might be too small for its position (max-heap) |
| Arbitrary replacement | May need sift up OR sift down — check both directions |
| One-child edge case | Last internal node may have only a left child |

---

## Quick Check

1. **Why does build-heap use sift-DOWN and not sift-UP?**
   <details><summary>Answer</summary>
   Sift-down build heap is O(n) because many nodes near the leaves do little work. Sift-up build heap is O(n log n) because many nodes near the leaves do maximum work (sifting all the way up).
   </details>

2. **What happens when a node has only one child during sift-down?**
   <details><summary>Answer</summary>
   You must check bounds before accessing the right child. Only compare with the left child if the right child index is out of bounds.
   </details>

---

[← Previous: Build Max-Heap Trace](05-build-max-heap-trace.md) | [Next Unit: Heap Sort →](../04-Heap-Sort/01-heap-sort-algorithm.md)
