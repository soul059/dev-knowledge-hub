# Chapter 2: Quick Select Algorithm

## Overview

**Quick Select** (also called Hoare's selection algorithm) adapts Quick Sort's partitioning to find the k-th smallest element in **O(n) expected time**. Instead of recursing on both halves, it recurses on **only one** — the half containing the target.

---

## Algorithm

```
function quickSelect(arr, lo, hi, k):
    if lo == hi:
        return arr[lo]
    
    // Partition and get pivot position
    pivotIndex = partition(arr, lo, hi)
    
    if k == pivotIndex:
        return arr[k]                           // found it!
    else if k < pivotIndex:
        return quickSelect(arr, lo, pivotIndex - 1, k)   // search left
    else:
        return quickSelect(arr, pivotIndex + 1, hi, k)   // search right

// Usage: find k-th smallest (0-indexed)
// quickSelect(arr, 0, n-1, k-1)
```

---

## How It Works Visually

```
Find 4th smallest (k=3, 0-indexed) in [3, 2, 1, 5, 6, 4]

Step 1: Partition around 4 (last element)
  [3, 2, 1] [4] [6, 5]
   0  1  2   3   4  5
  
  Pivot at index 3, k=3 → k == pivotIndex → FOUND!
  Answer: 4 ✓ (done in ONE partition!)
```

```
Find 2nd smallest (k=1) in [3, 2, 1, 5, 6, 4]

Step 1: Partition around 4
  [3, 2, 1] [4] [6, 5]
   0  1  2   3   4  5
  
  k=1 < pivotIndex=3 → search LEFT [3, 2, 1]

Step 2: Partition [3, 2, 1] around 1
  [] [1] [3, 2]
      0   1  2
  
  k=1 > pivotIndex=0 → search RIGHT [3, 2]

Step 3: Partition [3, 2] around 2
  [] [2] [3]
      1   2
  
  k=1 == pivotIndex=1 → FOUND!
  Answer: 2 ✓
```

---

## D&C Structure

```
┌────────────────────────────────────────────────────────────┐
│  QUICK SORT                  QUICK SELECT                  │
│                                                            │
│  Divide: partition           Divide: partition             │
│  Conquer: sort BOTH halves   Conquer: recurse ONE half     │
│  Combine: nothing            Combine: nothing              │
│                                                            │
│  T(n) = 2T(n/2) + O(n)     T(n) = T(n/2) + O(n)         │
│  = O(n log n)               = O(n)                       │
│                                                            │
│  Two recursive calls         ONE recursive call            │
│  → eliminates half           → eliminates half             │
│  → but still processes both  → ONLY processes one          │
└────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Trace (Complete)

```
Find 5th smallest (k=4, 0-indexed) in:
arr = [12, 3, 5, 7, 4, 19, 26]

STEP 1: quickSelect(arr, 0, 6, 4)
  Partition around 26 (last element):
  [12, 3, 5, 7, 4, 19] [26]
   0   1  2  3  4  5     6
  
  pivotIndex = 6
  k=4 < 6 → search LEFT quickSelect(arr, 0, 5, 4)

STEP 2: quickSelect(arr, 0, 5, 4)
  Partition [12, 3, 5, 7, 4, 19] around 19:
  [12, 3, 5, 7, 4] [19]
   0   1  2  3  4    5
  
  pivotIndex = 5
  k=4 < 5 → search LEFT quickSelect(arr, 0, 4, 4)

STEP 3: quickSelect(arr, 0, 4, 4)
  Partition [12, 3, 5, 7, 4] around 4:
  [3] [4] [12, 5, 7]                     wait, let's trace carefully
  
  Lomuto partition with pivot = arr[4] = 4:
    i=-1
    j=0: arr[0]=12 > 4 → skip
    j=1: arr[1]=3 ≤ 4 → i=0, swap(0,1) → [3, 12, 5, 7, 4]
    j=2: arr[2]=5 > 4 → skip
    j=3: arr[3]=7 > 4 → skip
    swap(i+1=1, hi=4) → [3, 4, 5, 7, 12]
    
  pivotIndex = 1
  k=4 > 1 → search RIGHT quickSelect(arr, 2, 4, 4)

STEP 4: quickSelect(arr, 2, 4, 4)
  Partition [5, 7, 12] around 12:
  [5, 7] [12]
   2  3    4
  
  pivotIndex = 4
  k=4 == 4 → FOUND!
  Answer: arr[4] = 12 ✓
  
  Verify: sorted = [3, 4, 5, 7, 12, 19, 26]
  5th smallest (0-indexed: 4) = 12 ✓
```

---

## Implementation (Python)

```python
import random

def quick_select(arr, k):
    """Find k-th smallest element (1-indexed)."""
    return _quick_select(arr, 0, len(arr) - 1, k - 1)

def _quick_select(arr, lo, hi, k):
    if lo == hi:
        return arr[lo]
    
    # Random pivot for expected O(n)
    rand_idx = random.randint(lo, hi)
    arr[rand_idx], arr[hi] = arr[hi], arr[rand_idx]
    
    # Lomuto partition
    pivot = arr[hi]
    i = lo - 1
    for j in range(lo, hi):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i + 1], arr[hi] = arr[hi], arr[i + 1]
    pivot_idx = i + 1
    
    if k == pivot_idx:
        return arr[k]
    elif k < pivot_idx:
        return _quick_select(arr, lo, pivot_idx - 1, k)
    else:
        return _quick_select(arr, pivot_idx + 1, hi, k)

# Usage
arr = [12, 3, 5, 7, 4, 19, 26]
print(quick_select(arr, 5))  # 12
```

---

## Iterative Version

```
function quickSelectIterative(arr, k):
    lo = 0, hi = n - 1
    
    while lo < hi:
        pivotIndex = partition(arr, lo, hi)
        
        if pivotIndex == k:
            return arr[k]
        else if k < pivotIndex:
            hi = pivotIndex - 1
        else:
            lo = pivotIndex + 1
    
    return arr[lo]      // lo == hi == k
```

The iterative version is often preferred — no recursion overhead.

---

## Complexity Analysis

```
EXPECTED TIME: O(n)

With a random pivot, expected partition is roughly balanced.
T(n) = T(n/2) + O(n)     (on average, recurse on half)

Unrolling:
  T(n) = cn + cn/2 + cn/4 + cn/8 + ...
       = cn × (1 + 1/2 + 1/4 + 1/8 + ...)
       = cn × 2
       = O(n)

The geometric series converges to 2n!
Unlike Quick Sort, we DON'T add both sides.

WORST CASE: O(n²)
  If pivot is always extreme:
  T(n) = T(n-1) + cn
  = c(n + n-1 + ... + 1) = O(n²)

SPACE: O(1) iterative, O(log n) expected recursive
```

---

## Summary Table

| Property | Value |
|----------|-------|
| Expected time | O(n) |
| Worst case | O(n²) |
| Space (iterative) | O(1) |
| In-place | Yes (modifies input) |
| Stable | No |
| Key idea | Partition + recurse on ONE side |
| Recurrence | T(n) = T(n/2) + O(n) → O(n) |

---

## Quick Revision Questions

1. **How does Quick Select differ from Quick Sort in its recursion?**
2. **Why is T(n) = T(n/2) + O(n) = O(n), not O(n log n)?**
3. **What happens when the pivot index equals k?**
4. **What is the worst case and when does it occur?**
5. **How does randomization help Quick Select?**

---

[← Previous: Kth Element Problem](01-kth-element-problem.md) | [Next: Average Case Analysis →](03-average-case-analysis.md)
