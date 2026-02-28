# 7.3 Sliding Window Median

[← Previous: Median from Data Stream](02-median-from-data-stream.md) | [Next: Maximize Capital →](04-maximize-capital.md)

---

## Problem

Given an array `nums` and a window size `k`, find the median of each sliding window.

```
nums = [1, 3, -1, -3, 5, 3, 6, 7], k = 3

Window          Sorted Window    Median
[1, 3, -1]     [-1, 1, 3]       1
[3, -1, -3]    [-3, -1, 3]     -1
[-1, -3, 5]    [-3, -1, 5]     -1
[-3, 5, 3]     [-3, 3, 5]       3
[5, 3, 6]      [3, 5, 6]        5
[3, 6, 7]      [3, 6, 7]        6

Answer: [1, -1, -1, 3, 5, 6]
```

---

## Challenge: Removing Elements

Unlike the streaming median, the sliding window requires **removing** old elements. Standard heaps don't support efficient removal.

---

## Approach: Two Heaps + Lazy Deletion

```
SLIDING_WINDOW_MEDIAN(nums, k):
    max_heap = []    // Lower half
    min_heap = []    // Upper half
    to_delete = {}   // Lazy deletion counter
    result = []
    balance = 0      // Track relative sizes
    
    // Initialize first window
    FOR i = 0 TO k-1:
        add_num(nums[i])
    result.append(get_median())
    
    // Slide window
    FOR i = k TO n-1:
        // Add new element
        add_num(nums[i])
        
        // Schedule removal of outgoing element
        outgoing = nums[i - k]
        to_delete[outgoing] += 1
        
        // Adjust balance based on which heap the outgoing element was in
        IF outgoing ≤ max_heap.peek():
            balance -= 1    // Lost one from lower half
        ELSE:
            balance += 1    // Lost one from upper half
        
        // Rebalance
        rebalance()
        
        // Clean heap tops (remove lazily deleted elements)
        prune_tops()
        
        result.append(get_median())
    
    RETURN result
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n log n) — each element inserted/deleted once |
| Space | O(n) for lazy deletion bookkeeping |

---

## Quick Check

1. **Why is lazy deletion useful for sliding window median?**
   <details><summary>Answer</summary>
   Standard heaps don't support O(log n) deletion of arbitrary elements. Lazy deletion marks elements for removal but only actually removes them when they reach the heap top. This avoids O(n) searches while maintaining O(log n) amortized per operation.
   </details>

---

[← Previous: Median from Data Stream](02-median-from-data-stream.md) | [Next: Maximize Capital →](04-maximize-capital.md)
