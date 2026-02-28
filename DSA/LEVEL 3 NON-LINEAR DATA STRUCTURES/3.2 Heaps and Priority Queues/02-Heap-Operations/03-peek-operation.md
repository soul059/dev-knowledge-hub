# 2.3 Peek Operation

[← Previous: Extract Min/Max](02-extract-min-max.md) | [Next: Build Heap from Array →](04-build-heap-from-array.md)

---

## Overview

The simplest heap operation — and the **primary advantage** of using a heap. Constant-time access to the min or max element.

---

## The Operation

Just return the root without removing it.

```
PEEK(heap):
    IF heap is empty:
        ERROR "Heap is empty"
    RETURN heap[0]
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(1) |
| Space | O(1) |

---

## Why This Matters

This is the **entire reason heaps exist**. No matter how many elements are in the heap, accessing the highest-priority element is always O(1).

Compare with alternatives:

| Data Structure | Access Min/Max |
|---------------|---------------|
| **Heap** | **O(1)** |
| Unsorted array | O(n) — must scan everything |
| Sorted array | O(1) — but insert is O(n) |
| BST (balanced) | O(log n) |

---

## Quick Check

1. **Does peek modify the heap in any way?**
   <details><summary>Answer</summary>
   No. Peek only reads the root element (heap[0]) and returns it. The heap remains unchanged.
   </details>

---

[← Previous: Extract Min/Max](02-extract-min-max.md) | [Next: Build Heap from Array →](04-build-heap-from-array.md)
