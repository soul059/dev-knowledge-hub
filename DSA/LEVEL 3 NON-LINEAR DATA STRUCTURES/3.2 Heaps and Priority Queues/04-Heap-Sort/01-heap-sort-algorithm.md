# 4.1 Heap Sort Algorithm

[← Previous Unit: Heapify Process](../03-Heapify-Process/06-edge-cases.md) | [Next: Step-by-Step Trace →](02-step-by-step-trace.md)

---

## Overview

Heap Sort is a comparison-based sorting algorithm that leverages the heap data structure to sort an array **in-place** with **O(n log n)** worst-case guarantee. Unlike Quick Sort, it never degrades to O(n²). Unlike Merge Sort, it requires no extra space.

---

## Core Idea

1. **Build a Max-Heap** from the unsorted array (O(n))
2. **Repeatedly extract the maximum** and place it at the end (O(n log n))

After step 2, the array is sorted in **ascending order**.

---

## Why Max-Heap for Ascending Order?

```
Max-Heap → Extract max → Place at END → Repeat

After all extractions:
[smallest ... ... ... largest]
 ← ascending order →
```

For **descending order**, use a **min-heap** instead.

---

## High-Level Pseudocode

```
HEAP_SORT(array):
    n = array.length
    
    // Phase 1: Build max-heap — O(n)
    BUILD_MAX_HEAP(array)
    
    // Phase 2: Extract elements — O(n log n)
    FOR i = n-1 DOWN TO 1:
        SWAP(array[0], array[i])    // Move max to end
        SIFT_DOWN(array, 0, i)      // Fix heap of size i
```

---

## Phase 1: Building the Max-Heap

```
BUILD_MAX_HEAP(array):
    n = array.length
    FOR i = (n/2 - 1) DOWN TO 0:
        SIFT_DOWN(array, i, n)
```

This is the same O(n) algorithm from Unit 3. After this phase, `array[0]` is the maximum element.

### Example

```
Input: [4, 10, 3, 5, 1, 7, 6]

Step-by-step build (see Unit 3 for details):
Result Max-Heap: [10, 5, 7, 4, 1, 3, 6]

         10
        /  \
       5    7
      / \  / \
     4  1 3   6
```

---

## Phase 2: The Extraction Process

### The Key Trick

Instead of actually removing the root (which would need extra space), we:
1. **Swap** root with the **last element** of the unsorted portion
2. **Shrink** the heap size by 1 (the swapped element is now in its final position)
3. **Sift down** the new root to restore the heap property

```
┌──────────────────────────────────────────────┐
│  [  HEAP PORTION  |  SORTED PORTION  ]       │
│   ← shrinks →       ← grows →               │
│                                              │
│  heap[0..i-1]        array[i..n-1]           │
│  (valid max-heap)    (final sorted position) │
└──────────────────────────────────────────────┘
```

---

## Quick Check

1. **Why do we use a max-heap (not min-heap) to sort in ascending order?**
   <details><summary>Answer</summary>
   We extract the max and place it at the END of the array. After all extractions, the largest elements are at the end and smallest at the front → ascending order.
   </details>

---

[← Previous Unit: Heapify Process](../03-Heapify-Process/06-edge-cases.md) | [Next: Step-by-Step Trace →](02-step-by-step-trace.md)
