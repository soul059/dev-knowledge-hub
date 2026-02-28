# 3.4 Building a Heap in O(n) — The Proof

[← Previous: Sift Up vs Sift Down](03-sift-up-vs-sift-down.md) | [Next: Build Max-Heap Trace →](05-build-max-heap-trace.md)

---

## Overview

The naive approach (insert n elements one by one) takes O(n log n). Floyd's bottom-up approach takes O(n). This is a **significant improvement** and the proof is a classic in algorithm analysis.

---

## The Algorithm Recap

```
BUILD_HEAP(array):
    n = array.length
    FOR i = (n/2 - 1) DOWN TO 0:    // Start from last internal node
        SIFT_DOWN(array, i, n)
```

---

## Visual: Which Nodes Do Work?

```
For a heap with 15 elements (height 3):

Level 0 (root):      1 node   → sifts down up to 3 levels
Level 1:             2 nodes  → sifts down up to 2 levels  
Level 2:             4 nodes  → sifts down up to 1 level
Level 3 (leaves):    8 nodes  → sifts down 0 levels (NO WORK!)

        ┌──────┐
        │  ×3  │  ← 1 node, max 3 swaps
        └──┬───┘
      ┌────┴────┐
    ┌─┴──┐  ┌──┴─┐
    │ ×2 │  │ ×2 │  ← 2 nodes, max 2 swaps each
    └─┬──┘  └──┬─┘
   ┌──┴──┐  ┌──┴──┐
  ┌┴┐  ┌┴┐ ┌┴┐ ┌┴┐
  │×1│ │×1│ │×1│ │×1│  ← 4 nodes, max 1 swap each
  └──┘ └──┘ └──┘ └──┘
  ○○ ○○ ○○ ○○           ← 8 leaf nodes, 0 swaps!
```

---

## Mathematical Proof

For a complete binary tree of height `h`:
- Level `k` has at most `2^k` nodes
- A node at level `k` can sift down at most `h - k` levels

**Total work:**

$$T(n) = \sum_{k=0}^{h} 2^k \cdot (h - k)$$

Let `j = h - k` (substitute):

$$T(n) = \sum_{j=0}^{h} 2^{h-j} \cdot j = 2^h \sum_{j=0}^{h} \frac{j}{2^j}$$

The key identity:

$$\sum_{j=0}^{\infty} \frac{j}{2^j} = 2$$

This is a convergent series! So:

$$T(n) = 2^h \cdot 2 = 2^{h+1}$$

Since $h = \lfloor \log_2 n \rfloor$, we get $2^{h+1} \leq 2n$.

$$\boxed{T(n) = O(n)}$$

---

## Intuitive Understanding

> **Half the nodes are leaves** (do nothing). A quarter of nodes do 1 swap. An eighth do 2 swaps. And so on. This geometric decrease means the total work converges to O(n).

---

## Why NOT O(n) with Sift Up?

If we build a heap using **sift up** (top-down):

$$T(n) = \sum_{k=0}^{h} 2^k \cdot k$$

Here, the **many nodes at the bottom** do the **most work** (sifting up `k` levels from level `k`).

$$\sum_{k=0}^{h} 2^k \cdot k = 2^{h+1}(h-1) + 2 \approx n \log n$$

**Conclusion**: Sift-down build heap = O(n). Sift-up build heap = O(n log n).

---

## Summary

| Approach | Time | Why |
|----------|------|-----|
| Bottom-up (sift down) | **O(n)** | Many nodes do little work (leaves = 0) |
| Top-down (sift up) | O(n log n) | Many nodes do lots of work (leaves sift up log n) |

---

## Quick Check

1. **In build-heap, why do we start from `n/2 - 1` instead of `n - 1`?**
   <details><summary>Answer</summary>
   Nodes from index n/2 to n-1 are leaf nodes. Leaves are trivially valid heaps (single element). Starting from the last internal node avoids wasting time on leaves.
   </details>

2. **Can you build a heap in O(n) using sift-up? Why or why not?**
   <details><summary>Answer</summary>
   No. With sift-up, ≈n/2 leaf-level insertions each do O(log n) work, giving Ω(n log n). The sum diverges unlike the sift-down case.
   </details>

---

[← Previous: Sift Up vs Sift Down](03-sift-up-vs-sift-down.md) | [Next: Build Max-Heap Trace →](05-build-max-heap-trace.md)
