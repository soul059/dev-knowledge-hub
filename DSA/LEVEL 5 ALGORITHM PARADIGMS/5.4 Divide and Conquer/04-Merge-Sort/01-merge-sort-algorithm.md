# Chapter 1: Merge Sort Algorithm

## Overview

**Merge Sort** is the quintessential Divide and Conquer sorting algorithm. It guarantees O(n log n) time in **all cases** (best, average, worst), making it one of the most reliable sorting algorithms. It is also **stable**, preserving the relative order of equal elements.

---

## The Algorithm at a Glance

```
┌───────────────────────────────────────────────────────────┐
│  MERGE SORT                                               │
│                                                           │
│  1. DIVIDE:  Split array into two halves                  │
│  2. CONQUER: Recursively sort each half                   │
│  3. COMBINE: Merge the two sorted halves                  │
│                                                           │
│  T(n) = 2T(n/2) + O(n)  →  O(n log n)                   │
└───────────────────────────────────────────────────────────┘
```

---

## Visual: Top-Down Merge Sort

```
                    [38, 27, 43, 3, 9, 82, 10]
                   /                           \
          [38, 27, 43, 3]              [9, 82, 10]
          /             \              /          \
     [38, 27]       [43, 3]       [9, 82]       [10]
      /    \         /    \       /    \           |
   [38]   [27]   [43]   [3]   [9]   [82]       [10]
      \    /         \    /       \    /           |
     [27, 38]       [3, 43]      [9, 82]        [10]
          \             /              \          /
       [3, 27, 38, 43]              [9, 10, 82]
                   \                       /
            [3, 9, 10, 27, 38, 43, 82]
```

---

## Pseudocode

```
function mergeSort(arr, lo, hi):
    if lo >= hi:
        return                       // base case: 0 or 1 element
    
    mid = lo + (hi - lo) / 2
    mergeSort(arr, lo, mid)          // sort left half
    mergeSort(arr, mid + 1, hi)      // sort right half
    merge(arr, lo, mid, hi)          // merge sorted halves

function merge(arr, lo, mid, hi):
    temp = new array of size (hi - lo + 1)
    i = lo                           // pointer for left half
    j = mid + 1                      // pointer for right half
    k = 0                            // pointer for temp array
    
    while i <= mid AND j <= hi:
        if arr[i] <= arr[j]:         // <= ensures stability
            temp[k++] = arr[i++]
        else:
            temp[k++] = arr[j++]
    
    while i <= mid:                  // copy remaining left
        temp[k++] = arr[i++]
    
    while j <= hi:                   // copy remaining right
        temp[k++] = arr[j++]
    
    copy temp back to arr[lo..hi]
```

---

## Step-by-Step Trace

```
Input: [38, 27, 43, 3]

DIVIDE PHASE (splitting):
  mergeSort([38,27,43,3], 0, 3)
    split at mid=1
    mergeSort([38,27], 0, 1)
      split at mid=0
      mergeSort([38], 0, 0) → base case
      mergeSort([27], 1, 1) → base case
      merge([38,27], 0, 0, 1)
    mergeSort([43,3], 2, 3)
      split at mid=2
      mergeSort([43], 2, 2) → base case
      mergeSort([3], 3, 3)  → base case
      merge([43,3], 2, 2, 3)
    merge(result, 0, 1, 3)

MERGE PHASE (combining):
  merge([38,27], 0, 0, 1):
    i=0(38), j=1(27): 27 < 38 → temp=[27], j=2
    i=0(38): copy → temp=[27,38]
    Result: [27, 38]

  merge([43,3], 2, 2, 3):
    i=2(43), j=3(3): 3 < 43 → temp=[3], j=4
    i=2(43): copy → temp=[3,43]
    Result: [3, 43]

  merge([27,38,3,43], 0, 1, 3):
    i=0(27), j=2(3):  3 < 27  → temp=[3],     j=3
    i=0(27), j=3(43): 27 < 43 → temp=[3,27],  i=1
    i=1(38), j=3(43): 38 < 43 → temp=[3,27,38], i=2
    j=3(43): copy → temp=[3,27,38,43]
    Result: [3, 27, 38, 43] ✓
```

---

## The D&C Structure

```
┌───────────────────────────────────────────────────────┐
│                                                       │
│  DIVIDE:   Split array at midpoint                    │
│            → Trivial: O(1) time                       │
│            → Always splits evenly (balanced)          │
│                                                       │
│  CONQUER:  Recursively sort both halves               │
│            → Two subproblems, each size n/2           │
│            → a=2, b=2                                 │
│                                                       │
│  COMBINE:  Merge two sorted halves                    │
│            → O(n) time using two pointers             │
│            → This is where the real work happens      │
│                                                       │
│  Recurrence: T(n) = 2T(n/2) + O(n)                  │
│  By Master Theorem: Case 2 → O(n log n)             │
│                                                       │
└───────────────────────────────────────────────────────┘
```

---

## Why Merge Sort Is Stable

```
Stability: Equal elements maintain their original order.

arr = [(3,A), (1,B), (3,C), (2,D)]

After sorting by key:
  Stable:     [(1,B), (2,D), (3,A), (3,C)]  ← (3,A) before (3,C) ✓
  Unstable:   [(1,B), (2,D), (3,C), (3,A)]  ← order changed ✗

In merge(): using <= (not <) ensures stability:
  if arr[i] <= arr[j]:  // when equal, take from LEFT first
      temp[k++] = arr[i++]
```

---

## Implementation Variants

### Python Implementation

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

### In-Place Index-Based (C++/Java style)

```
function mergeSort(arr, lo, hi):
    if lo >= hi: return
    mid = lo + (hi - lo) / 2
    mergeSort(arr, lo, mid)
    mergeSort(arr, mid + 1, hi)
    merge(arr, lo, mid, hi)

// Call: mergeSort(arr, 0, n-1)
```

---

## Recursion Tree

```
Level 0:          mergeSort(n)                    merge cost: n
                 /            \
Level 1:    sort(n/2)      sort(n/2)              merge cost: n/2 + n/2 = n
            /     \        /     \
Level 2: sort(n/4) sort(n/4) sort(n/4) sort(n/4) merge cost: 4×(n/4) = n
            ...        ...       ...       ...
Level k:  n leaf nodes of size 1                  merge cost: n×1 = n

Total levels: log₂(n)
Cost per level: O(n)
Total: O(n log n)
```

---

## Summary Table

| Property | Value |
|----------|-------|
| Time (Best) | O(n log n) |
| Time (Average) | O(n log n) |
| Time (Worst) | O(n log n) |
| Space | O(n) auxiliary |
| Stable | Yes |
| In-place | No (requires O(n) extra space) |
| Adaptive | No (same time regardless of input) |
| Divide | Split at midpoint — O(1) |
| Combine | Merge — O(n) |

---

## Quick Revision Questions

1. **What are the three steps of Merge Sort in D&C terms?**
2. **Why does Merge Sort always guarantee O(n log n)?**
3. **How does the merge step ensure stability?**
4. **What is the space complexity, and why can't standard Merge Sort be in-place?**
5. **Write the recurrence relation and identify which Master Theorem case applies.**

---

[Next: The Merge Process →](02-merge-process.md)
