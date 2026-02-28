# 2.5 Delete Arbitrary Element & Replace Root

[← Previous: Build Heap](04-build-heap-from-array.md) | [Next: Operations Summary →](06-operations-summary.md)

---

## Overview

Beyond the core insert/extract/peek, heaps support two additional operations: deleting an arbitrary element and replacing the root in one shot.

---

## Delete Arbitrary Element

### Strategy

1. Find the element (O(n) scan since heaps aren't searchable)
2. Replace it with the last element
3. Sift up OR sift down as needed

### Pseudocode

```
DELETE(heap, index):
    heap[index] = heap[heap.size - 1]   // Replace with last
    heap.remove_last()
    
    IF index > 0 AND heap[index] > heap[parent(index)]:   // Max-heap
        SIFT_UP(heap, index)
    ELSE:
        SIFT_DOWN(heap, index)
```

### Complexity

**Time**: O(n) to find + O(log n) to fix = **O(n)**

> With an **indexed priority queue** (Unit 10), deletion becomes O(log n) because we can locate elements in O(1).

---

## Replace Root (Replace Top)

A common optimization: instead of Extract + Insert (two O(log n) operations), do both in one:

```
REPLACE_TOP(heap, new_value):
    old_root = heap[0]
    heap[0] = new_value
    SIFT_DOWN(heap, 0)
    RETURN old_root
```

### Why Use This?

| Approach | Cost |
|----------|------|
| Extract + Insert | Sift down + Sift up = ~2 × O(log n) |
| **Replace Top** | **Single sift down = O(log n)** |

This is especially useful in **K-element problems** (Unit 6) where you repeatedly replace the root when a better candidate is found.

---

## Quick Check

1. **Can the sift-down procedure ever require sifting UP after replacing an arbitrary element?**
   <details><summary>Answer</summary>
   Yes! When deleting an arbitrary element, the replacement (last element) could be larger than the deleted element's parent (max-heap), requiring sift-up instead of sift-down. You must check both directions.
   </details>

2. **What advantage does "replace top" have over "extract + insert"?**
   <details><summary>Answer</summary>
   Replace top uses one sift-down (O(log n)). Extract + insert uses one sift-down + one sift-up (2 × O(log n)). Replace top saves roughly half the comparisons and swaps.
   </details>

---

[← Previous: Build Heap](04-build-heap-from-array.md) | [Next: Operations Summary →](06-operations-summary.md)
