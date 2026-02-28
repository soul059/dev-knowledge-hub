# Chapter 4: Array Sum (Recursive Approaches)

## Overview
We revisit array sum from Unit 2 with a focus on **multiple recursive strategies** — left-to-right, right-to-left, and divide-and-conquer — and compare their space complexities.

---

## Approach 1: Left to Right (Index-based)

```
function sum(arr, i=0):
    if i >= length(arr): return 0
    return arr[i] + sum(arr, i + 1)
```

```
sum([5, 3, 8, 1], 0)
= 5 + sum(arr, 1) = 5 + 12 = 17
      = 3 + sum(arr, 2) = 3 + 9 = 12
            = 8 + sum(arr, 3) = 8 + 1 = 9
                  = 1 + sum(arr, 4) = 1 + 0 = 1
                        = 0  (BASE)
```

## Approach 2: Subarray View

```
function sum(arr):
    if arr is empty: return 0
    return arr[0] + sum(arr[1:])     // Subarray excludes first

// Warning: arr[1:] creates a copy → O(n) per call → O(n²) total!
```

## Approach 3: Divide and Conquer

```
function sum(arr, low, high):
    if low > high: return 0
    if low == high: return arr[low]
    mid = (low + high) / 2
    return sum(arr, low, mid) + sum(arr, mid+1, high)
```

```
sum([5, 3, 8, 1], 0, 3)
          [5, 3, 8, 1]
          /            \
     [5, 3]          [8, 1]
     /    \          /    \
   [5]   [3]      [8]   [1]
    5  +  3   =8   8  +  1  =9
         8    +    9    = 17
```

**Space: O(log n)** stack depth — better than linear O(n)!

---

## Complexity Comparison

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Left-to-right | O(n) | O(n) | Simple, n frames |
| Subarray copy | O(n²) | O(n²) | Bad — copies at each level |
| Divide & conquer | O(n) | O(log n) | Best space efficiency |
| Tail recursive | O(n) | O(n) or O(1) | O(1) with TCO |
| Iterative | O(n) | O(1) | Best overall |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Best recursive approach | Divide and conquer — O(log n) space |
| Most common approach | Index-based left-to-right |
| Avoid | Subarray copy approach (O(n²)) |
| Pattern | Generalizes to any associative operation |

---

## Quick Revision Questions

1. **Why does the subarray copy approach** have O(n²) time?
2. **What is the space advantage** of divide and conquer?
3. **How would you adapt** this pattern for finding the product?
4. **Write the tail-recursive version** with an accumulator.
5. **What is the recurrence** for the divide-and-conquer approach?
6. **Trace sum([2, 7, 1, 4], 0, 3)** using divide and conquer.

---

[← Previous: Check Sorted Array](03-check-sorted-array.md) | [Next: Find Max/Min →](05-find-max-min.md)

[← Back to README](../README.md)
