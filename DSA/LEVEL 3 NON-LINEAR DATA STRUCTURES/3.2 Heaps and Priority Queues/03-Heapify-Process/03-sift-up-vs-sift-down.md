# 3.3 Sift Up vs Sift Down Comparison

[← Previous: Heapify Down](02-heapify-down.md) | [Next: Build Heap O(n) Proof →](04-build-heap-on-proof.md)

---

## Overview

Both sift up and sift down restore the heap property along a single path, but they work in opposite directions and are used in different contexts.

---

## Visual Comparison

```
      SIFT UP                    SIFT DOWN
      
        ↑↑↑                       ↓↓↓
       /   \                     /   \
      ↑     ·                   ↓     ·
     / \   / \                 / \   / \
    ↑   · ·   ·              ·   · ↓   ·
   Start here               Start here
   (bottom)                  (top)
```

---

## Comparison Table

| Aspect | Sift Up | Sift Down |
|--------|---------|-----------|
| **Direction** | Bottom → Top | Top → Bottom |
| **Used in** | Insert, Increase Key | Extract, Decrease Key, Build Heap |
| **Comparisons/level** | 1 (parent only) | 2 (both children) |
| **Starts at** | Leaf or modified node | Root or modified node |
| **Ends at** | Root (worst case) | Leaf (worst case) |
| **Time** | O(log n) | O(log n) |

---

## When to Use Which

```
┌─────────────────────────┬─────────────────┐
│  What Changed?          │  What to Do?    │
├─────────────────────────┼─────────────────┤
│ Added element at bottom │ Sift UP         │
│ Replaced root           │ Sift DOWN       │
│ Increased key (max-heap)│ Sift UP         │
│ Decreased key (max-heap)│ Sift DOWN       │
│ Increased key (min-heap)│ Sift DOWN       │
│ Decreased key (min-heap)│ Sift UP         │
│ Replaced arbitrary node │ Sift UP or DOWN │
└─────────────────────────┴─────────────────┘
```

---

## Key Invariant

> After sift up/down completes, the **entire** heap satisfies the heap property. This is because:
> 1. The subtrees that weren't on the sift path were already valid
> 2. The sift fixes every violated parent-child pair on its path

---

## Quick Check

1. **After replacing an arbitrary node in a max-heap, how do you decide whether to sift up or sift down?**
   <details><summary>Answer</summary>
   Compare the new value with its parent: if larger than parent, sift up. If smaller than either child, sift down. At most one direction is needed since the rest of the heap is already valid.
   </details>

---

[← Previous: Heapify Down](02-heapify-down.md) | [Next: Build Heap O(n) Proof →](04-build-heap-on-proof.md)
