# Chapter 6: Complete Selection Reference & Practice

## Overview

This chapter consolidates the key selection algorithms, their complexities, implementation tips, and common interview problems involving selection and partition-based techniques.

---

## Algorithm Comparison Summary

```
┌──────────────────────┬──────────────┬──────────────┬──────────┬────────┐
│ Algorithm            │ Avg Time     │ Worst Time   │ Space    │ Notes  │
├──────────────────────┼──────────────┼──────────────┼──────────┼────────┤
│ Sort + index         │ O(n log n)   │ O(n log n)   │ O(n)     │ Simple │
│ Min-heap + extract   │ O(n+k log n) │ O(n+k log n) │ O(n)     │ k≪n   │
│ Max-heap of size k   │ O(n log k)   │ O(n log k)   │ O(k)     │ k≪n   │
│ Quick Select         │ O(n)         │ O(n²)        │ O(1)*    │ Fast   │
│ Rand Quick Select    │ O(n)         │ O(n²)†       │ O(1)*    │ Fast   │
│ Median of Medians    │ O(n)         │ O(n)         │ O(n)     │ Slow c │
│ IntroSelect          │ O(n)         │ O(n)         │ O(n)     │ Best   │
└──────────────────────┴──────────────┴──────────────┴──────────┴────────┘
* iterative version  † probability ≈ 2ⁿ/n!
```

---

## Quick Select: Template Code

```python
import random

def quick_select(arr, k):
    """Find k-th smallest (0-indexed). Modifies arr."""
    lo, hi = 0, len(arr) - 1
    while lo < hi:
        # Random pivot
        ri = random.randint(lo, hi)
        arr[ri], arr[hi] = arr[hi], arr[ri]
        
        # Lomuto partition
        pivot = arr[hi]
        i = lo - 1
        for j in range(lo, hi):
            if arr[j] <= pivot:
                i += 1
                arr[i], arr[j] = arr[j], arr[i]
        arr[i+1], arr[hi] = arr[hi], arr[i+1]
        p = i + 1
        
        if p == k:
            return arr[p]
        elif k < p:
            hi = p - 1
        else:
            lo = p + 1
    return arr[lo]

# k-th smallest (1-indexed): quick_select(arr, k-1)
# k-th largest:              quick_select(arr, n-k)
# median:                    quick_select(arr, n//2)
```

---

## Common Interview Problems

### Problem 1: Kth Largest Element (LC 215)

```python
def findKthLargest(nums, k):
    return quick_select(nums, len(nums) - k)
```

### Problem 2: Top K Frequent Elements (LC 347)

```
1. Count frequencies → {val: count} hashmap     O(n)
2. Quick Select on unique values by frequency    O(m), m = unique
3. Return all values with freq ≥ k-th freq       O(m)
```

### Problem 3: Wiggle Sort II (LC 324)

```
1. Find median using Quick Select                O(n)
2. Three-way partition around median              O(n)
3. Place elements in wiggle pattern               O(n)
```

### Problem 4: K Closest Points to Origin (LC 973)

```python
def kClosest(points, k):
    # Quick select based on distance
    distances = [(x*x + y*y, [x,y]) for x,y in points]
    # Partition so k smallest distances are in left
    quick_select_on_distances(distances, k)
    return [d[1] for d in distances[:k]]
```

---

## Partition-Based Techniques

```
The partition step from Quick Select/Sort is useful beyond sorting:

┌──────────────────────────────────────────────────────────┐
│  1. Dutch National Flag: 3-way partition                  │
│     → Sort array of 0s, 1s, 2s in O(n)                  │
│                                                          │
│  2. Two-way partition:                                    │
│     → Separate evens and odds                            │
│     → Separate negatives and positives                   │
│     → Move zeros to end                                  │
│                                                          │
│  3. Quick Select: find k-th element via partition         │
│                                                          │
│  4. Nuts and Bolts: match nuts to bolts using partition   │
└──────────────────────────────────────────────────────────┘
```

---

## Key Insights to Remember

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  1. Selection (k-th element) is EASIER than sorting        │
│     → Sorting: Ω(n log n)   Selection: Θ(n)              │
│                                                            │
│  2. Quick Select = Quick Sort with ONE recursive call      │
│     → Same partition, but only process one side            │
│     → Work decreases geometrically → O(n)                 │
│                                                            │
│  3. Randomization makes expected case = O(n) for ANY input │
│     → Can't be adversarially attacked                     │
│                                                            │
│  4. Median of Medians: theoretical guarantee               │
│     → O(n) worst case, but large constant                 │
│     → Groups of 5: smallest size that works               │
│                                                            │
│  5. In practice, use randomized Quick Select               │
│     → Simple, fast, expected O(n)                         │
│     → IntroSelect for worst-case guarantee                │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Recurrence Relations Recap

```
Quick Sort:    T(n) = 2T(n/2) + O(n) = O(n log n)
Quick Select:  T(n) = T(n/2) + O(n)  = O(n)
Binary Search: T(n) = T(n/2) + O(1)  = O(log n)

Pattern:
  T(n) = T(n/2) + O(nᵏ)
  
  If k = 0: T(n) = O(log n)     (binary search)
  If k = 1: T(n) = O(n)         (quick select)
  
  General: T(n) = T(n/2) + O(nᵏ) = O(nᵏ) for k ≥ 1
  (The O(nᵏ) dominates when only one recursive call)
```

---

## Summary Table

| Topic | Key Takeaway |
|-------|-------------|
| Selection problem | Find k-th element in O(n), not O(n log n) |
| Quick Select | Partition + recurse on one side, O(n) avg |
| Median of Medians | Groups of 5 → guaranteed 70/30 split → O(n) |
| IntroSelect | Quick Select + MoM fallback (practical O(n)) |
| Key recurrence | T(n) = T(n/2) + O(n) = O(n) |
| Common problems | K-th largest, Top-K, Median, Closest points |

---

## Quick Revision Questions

1. **What recurrence gives O(n) for one recursive call with O(n) partition?**
2. **How does IntroSelect combine the best of Quick Select and Median of Medians?**
3. **Convert "find 3rd largest" to a Quick Select call on an array of size n.**
4. **How would you find the top K elements in O(n) time?**
5. **Why is the selection problem theoretically easier than sorting?**

---

[← Previous: Applications](05-applications.md) | [Back to Unit 6](../README.md)
