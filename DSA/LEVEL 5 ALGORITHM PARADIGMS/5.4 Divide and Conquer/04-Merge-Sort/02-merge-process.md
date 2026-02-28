# Chapter 2: The Merge Process

## Overview

The **merge** step is the heart of Merge Sort. It takes two sorted subarrays and combines them into one sorted array in O(n) time using the **two-pointer technique**. Understanding the merge process deeply is essential — it appears in many other algorithms beyond sorting.

---

## The Two-Pointer Merge

```
Left:  [3, 8, 12, 27]     Right: [1, 9, 15, 20]
        i                          j

Compare arr[i] and arr[j], pick the smaller one.

Step 1: 3 vs 1 → pick 1     Result: [1]
                                      j→
Step 2: 3 vs 9 → pick 3     Result: [1, 3]
                              i→
Step 3: 8 vs 9 → pick 8     Result: [1, 3, 8]
                              i→
Step 4: 12 vs 9 → pick 9    Result: [1, 3, 8, 9]
                                      j→
Step 5: 12 vs 15 → pick 12  Result: [1, 3, 8, 9, 12]
                              i→
Step 6: 27 vs 15 → pick 15  Result: [1, 3, 8, 9, 12, 15]
                                      j→
Step 7: 27 vs 20 → pick 20  Result: [1, 3, 8, 9, 12, 15, 20]
                                      j→ (exhausted)
Step 8: copy 27             Result: [1, 3, 8, 9, 12, 15, 20, 27]
```

---

## Visual: Merge as a Zipper

```
Left Array:     3 ─── 8 ─── 12 ─── 27
                 \     \      \      \
                  ╲     ╲      ╲      ╲
Result:    1 ── 3 ── 8 ── 9 ── 12 ── 15 ── 20 ── 27
                /          /          /      /
               ╱          ╱          ╱      ╱
Right Array:  1 ─── 9 ─── 15 ─── 20

Like a zipper: elements interleave in sorted order
```

---

## Detailed Pseudocode

```
function merge(arr, lo, mid, hi):
    // Create temporary arrays
    leftSize = mid - lo + 1
    rightSize = hi - mid
    
    L = new array[leftSize]     // copy left half
    R = new array[rightSize]    // copy right half
    
    for i = 0 to leftSize - 1:
        L[i] = arr[lo + i]
    for j = 0 to rightSize - 1:
        R[j] = arr[mid + 1 + j]
    
    // Merge back
    i = 0, j = 0, k = lo
    
    while i < leftSize AND j < rightSize:
        if L[i] <= R[j]:        // <= for stability
            arr[k] = L[i]
            i++
        else:
            arr[k] = R[j]
            j++
        k++
    
    // Copy remaining elements
    while i < leftSize:
        arr[k++] = L[i++]
    while j < rightSize:
        arr[k++] = R[j++]
```

---

## Merge with Sentinel Values

Using ∞ as a sentinel removes the need for boundary checks:

```
function mergeWithSentinel(arr, lo, mid, hi):
    leftSize = mid - lo + 1
    rightSize = hi - mid
    
    L = new array[leftSize + 1]
    R = new array[rightSize + 1]
    
    // Copy + add sentinel
    for i = 0 to leftSize - 1: L[i] = arr[lo + i]
    for j = 0 to rightSize - 1: R[j] = arr[mid + 1 + j]
    L[leftSize] = ∞            // sentinel
    R[rightSize] = ∞           // sentinel
    
    i = 0, j = 0
    for k = lo to hi:
        if L[i] <= R[j]:
            arr[k] = L[i++]
        else:
            arr[k] = R[j++]
    // No need for "copy remaining" — sentinels handle it!
```

```
Why sentinels work:

L: [3, 8, 12, ∞]
R: [1, 9, ∞]

When R is exhausted, R[j] = ∞
Any remaining L[i] < ∞, so they get copied automatically.
```

---

## Complexity of Merge

```
┌────────────────────────────────────────────────────────┐
│  Time Complexity:  O(n)                                │
│    - Each element is compared and moved exactly once   │
│    - Total comparisons ≤ n - 1                         │
│    - Total moves = n (to temp) + n (back) = 2n        │
│                                                        │
│  Space Complexity: O(n)                                │
│    - Need temporary array of size n                    │
│    - This is why merge sort is NOT in-place            │
│                                                        │
│  Comparisons Analysis:                                 │
│    Best case:  n/2 comparisons                         │
│      (all of one array < all of other)                 │
│    Worst case: n - 1 comparisons                       │
│      (elements interleave perfectly)                   │
└────────────────────────────────────────────────────────┘
```

---

## Best vs Worst Case for Merge

```
BEST CASE (n/2 comparisons):
L: [1, 2, 3]    R: [4, 5, 6]
All of L < all of R → just compare L elements against R[0]
Comparisons: 3 (= n/2)

WORST CASE (n-1 comparisons):
L: [1, 3, 5]    R: [2, 4, 6]
Elements interleave → compare at every step
Comparisons: 5 (= n-1)

Result: [1, 2, 3, 4, 5, 6] in both cases
```

---

## Optimization: Skip Merge if Already Sorted

```
function mergeSort(arr, lo, hi):
    if lo >= hi: return
    mid = lo + (hi - lo) / 2
    mergeSort(arr, lo, mid)
    mergeSort(arr, mid + 1, hi)
    
    // OPTIMIZATION: skip merge if already in order
    if arr[mid] <= arr[mid + 1]:
        return                       // already sorted!
    
    merge(arr, lo, mid, hi)
```

This makes the best case O(n) for already-sorted arrays!

---

## Optimization: Reuse Auxiliary Array

Instead of creating a new array in every merge call:

```
function mergeSort(arr):
    aux = new array[n]              // allocate once
    mergeSortHelper(arr, aux, 0, n-1)

function mergeSortHelper(arr, aux, lo, hi):
    if lo >= hi: return
    mid = lo + (hi - lo) / 2
    mergeSortHelper(arr, aux, lo, mid)
    mergeSortHelper(arr, aux, mid + 1, hi)
    merge(arr, aux, lo, mid, hi)

function merge(arr, aux, lo, mid, hi):
    // Copy to aux
    for k = lo to hi: aux[k] = arr[k]
    
    i = lo, j = mid + 1
    for k = lo to hi:
        if i > mid:           arr[k] = aux[j++]
        else if j > hi:       arr[k] = aux[i++]
        else if aux[i] <= aux[j]: arr[k] = aux[i++]
        else:                 arr[k] = aux[j++]
```

---

## Merge Beyond Sorting

The merge operation appears in many algorithms:

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  1. Merge K Sorted Lists (with min-heap)                   │
│  2. External Sort (merge sorted chunks from disk)          │
│  3. Count Inversions (count during merge)                  │
│  4. Count Smaller After Self                               │
│  5. Merge Intervals (after sorting by start)               │
│  6. Merge Two Sorted Linked Lists                          │
│  7. Union of two sorted arrays                             │
│  8. Intersection of two sorted arrays                      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Merge Two Sorted Linked Lists

```
function mergeLists(l1, l2):
    dummy = new Node(0)
    current = dummy
    
    while l1 != null AND l2 != null:
        if l1.val <= l2.val:
            current.next = l1
            l1 = l1.next
        else:
            current.next = l2
            l2 = l2.next
        current = current.next
    
    current.next = (l1 != null) ? l1 : l2
    return dummy.next

// O(n+m) time, O(1) space — no extra array needed!
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Core idea | Two pointers traversing sorted halves |
| Time | O(n) per merge |
| Space | O(n) auxiliary array |
| Comparisons | Between n/2 (best) and n-1 (worst) |
| Stability | Ensured by using <= (not <) |
| Sentinel trick | Use ∞ to avoid boundary checks |
| Key optimization | Skip merge when arr[mid] ≤ arr[mid+1] |
| Reuse buffer | Allocate auxiliary array once, not per call |

---

## Quick Revision Questions

1. **How does the two-pointer merge ensure O(n) time?**
2. **What is the minimum and maximum number of comparisons in merge?**
3. **How does using <= instead of < ensure stability?**
4. **What is the sentinel value technique and what does it eliminate?**
5. **What optimization allows merge sort to run in O(n) on sorted input?**
6. **Why is the merge step the reason merge sort cannot be in-place?**

---

[← Previous: Merge Sort Algorithm](01-merge-sort-algorithm.md) | [Next: Complexity Analysis →](03-complexity-analysis.md)
