# 4.2 Complete Step-by-Step Trace

[← Previous: Heap Sort Algorithm](01-heap-sort-algorithm.md) | [Next: In-Place Sorting →](03-in-place-sorting.md)

---

## Overview

A full visual walkthrough of Heap Sort on a 5-element array, showing both phases in detail.

---

## Trace

```
Input Array: [4, 10, 3, 5, 1]

═══ PHASE 1: Build Max-Heap ═══

Array: [4, 10, 3, 5, 1]
       4                    10
      / \     build →      / \
    10   3    heap        5    3
   / \                   / \
  5   1                 4   1

Max-Heap: [10, 5, 3, 4, 1]

═══ PHASE 2: Extract and Sort ═══

── Iteration 1 (i=4) ──
Swap array[0] ↔ array[4]: swap 10 ↔ 1
Array: [1, 5, 3, 4, | 10]
                       ↑ sorted

Sift down index 0 (heap size = 4):
  1 < 5 → swap with 5
  1 < 4 → swap with 4
  
Array: [5, 4, 3, 1, | 10]

       5
      / \
     4   3            10 (sorted)
    /
   1

── Iteration 2 (i=3) ──
Swap array[0] ↔ array[3]: swap 5 ↔ 1
Array: [1, 4, 3, | 5, 10]
                    ↑ sorted

Sift down index 0 (heap size = 3):
  1 < 4 → swap with 4
  
Array: [4, 1, 3, | 5, 10]

       4
      / \             5, 10 (sorted)
     1   3

── Iteration 3 (i=2) ──
Swap array[0] ↔ array[2]: swap 4 ↔ 3
Array: [3, 1, | 4, 5, 10]
                ↑ sorted

Sift down index 0 (heap size = 2):
  3 > 1 → no swap needed

Array: [3, 1, | 4, 5, 10]

── Iteration 4 (i=1) ──
Swap array[0] ↔ array[1]: swap 3 ↔ 1
Array: [1, | 3, 4, 5, 10]
            ↑ sorted

Heap size = 1 → no sift needed

═══ FINAL SORTED ARRAY ═══
[1, 3, 4, 5, 10] ✅
```

---

## Key Observations

| Step | Heap Size | Sorted Elements |
|------|----------|-----------------|
| Start | 5 | none |
| After extract 1 | 4 | 10 |
| After extract 2 | 3 | 5, 10 |
| After extract 3 | 2 | 4, 5, 10 |
| After extract 4 | 1 | 3, 4, 5, 10 |
| Done | 0 | 1, 3, 4, 5, 10 |

---

## Quick Check

1. **Heap Sort is in-place. How does it manage to sort without extra space?**
   <details><summary>Answer</summary>
   The array is divided into two regions: heap[0..i-1] and sorted[i..n-1]. Each extraction swaps the max to position i and shrinks the heap by 1. The sorted portion grows from right to left.
   </details>

---

[← Previous: Heap Sort Algorithm](01-heap-sort-algorithm.md) | [Next: In-Place Sorting →](03-in-place-sorting.md)
