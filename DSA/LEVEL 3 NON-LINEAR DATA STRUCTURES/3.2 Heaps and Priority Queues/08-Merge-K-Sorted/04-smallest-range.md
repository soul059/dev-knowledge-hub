# 8.4 Smallest Range Covering Elements from K Lists

[← Previous: Merge K Sorted Arrays](03-merge-k-sorted-arrays.md) | [Next: Heap vs Divide & Conquer →](05-heap-vs-divide-conquer.md)

---

## Problem

Given K sorted lists, find the **smallest range [a, b]** such that the range includes **at least one element from each list**.

```
lists = [[4,10,15,24,26], [0,9,12,20], [5,18,22,30]]
Answer: [20, 24] — contains 24 from list 1, 20 from list 2, 22 from list 3
```

---

## Key Insight

At any point, the heap holds one element from each list. The range is:
- **Lower bound**: min-heap root (smallest current element)
- **Upper bound**: the maximum among all current elements

We want to **minimize** (upper - lower).

---

## Algorithm

```
SMALLEST_RANGE(lists):
    min_heap = []
    current_max = -∞
    best_range = [-∞, ∞]
    
    // Initialize with first element of each list
    FOR i = 0 TO K-1:
        min_heap.insert((lists[i][0], i, 0))
        current_max = max(current_max, lists[i][0])
    
    WHILE TRUE:
        (min_val, list_idx, elem_idx) = min_heap.extract_min()
        
        // Current range: [min_val, current_max]
        IF current_max - min_val < best_range[1] - best_range[0]:
            best_range = [min_val, current_max]
        
        // Advance the list that contributed the minimum
        IF elem_idx + 1 >= lists[list_idx].length:
            BREAK   // Can't advance — one list exhausted
        
        next_val = lists[list_idx][elem_idx + 1]
        min_heap.insert((next_val, list_idx, elem_idx + 1))
        current_max = max(current_max, next_val)
    
    RETURN best_range
```

---

## Trace

```
lists = [[4,10,15,24,26], [0,9,12,20], [5,18,22,30]]

Init: heap = [(0,1,0), (4,0,0), (5,2,0)], max = 5
Range: [0, 5], width = 5, best = [0, 5]

Extract (0,1,0), advance list 1 → insert (9,1,1), max = max(5,9) = 9
  Range: [4, 9], width = 5, best = [0, 5] (tie, keep first)

Extract (4,0,0), advance list 0 → insert (10,0,1), max = max(9,10) = 10
  Range: [5, 10], width = 5

Extract (5,2,0), advance list 2 → insert (18,2,1), max = max(10,18) = 18
  Range: [9, 18], width = 9

Extract (9,1,1), advance → insert (12,1,2), max = 18
  Range: [10, 18], width = 8

Extract (10,0,1), advance → insert (15,0,2), max = 18
  Range: [12, 18], width = 6

Extract (12,1,2), advance → insert (20,1,3), max = max(18,20) = 20
  Range: [15, 20], width = 5

Extract (15,0,2), advance → insert (24,0,3), max = max(20,24) = 24
  Range: [18, 24], width = 6

Extract (18,2,1), advance → insert (22,2,2), max = 24
  Range: [20, 24], width = 4 → NEW BEST! ✅

Extract (20,1,3), list 1 has no more → STOP

Answer: [20, 24] ✅
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(N log K) |
| Space | O(K) |

---

## Quick Check

1. **Why do we always advance the list that contributed the minimum?**
   <details><summary>Answer</summary>
   The minimum defines the lower bound of the range. To potentially shrink the range, we need to increase the lower bound. The only way to do that is to advance past the current minimum by getting the next element from its list.
   </details>

---

[← Previous: Merge K Sorted Arrays](03-merge-k-sorted-arrays.md) | [Next: Heap vs Divide & Conquer →](05-heap-vs-divide-conquer.md)
