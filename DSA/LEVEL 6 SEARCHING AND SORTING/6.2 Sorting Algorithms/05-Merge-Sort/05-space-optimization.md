# Unit 5: Merge Sort — Space Optimization

[← Previous: Complexity Analysis](04-complexity-analysis.md) | [Next: Iterative Merge Sort →](06-iterative-merge-sort.md)

---

## Overview

Merge Sort's biggest weakness is its **O(n) auxiliary space**. This chapter explores strategies to reduce memory usage, including iterative (bottom-up) merge sort, in-place merge techniques, and the natural merge sort variant.

---

## The Space Problem

```
  Standard Merge Sort:
  
  Array:     [█████████████████████████] n elements
  Aux space: [█████████████████████████] n elements
  Stack:     [████]                      log(n) frames
  
  Total memory = 2n + O(log n)
  
  For sorting 1 million integers (4 bytes each):
  Array:     4 MB
  Aux array: 4 MB  ← This is the problem!
  Stack:     ~80 bytes (20 frames × 4 bytes)
  
  Compare with in-place sorts (Insertion, Selection):
  Total memory = n + O(1)
```

---

## Optimization 1: Single Auxiliary Array

```
  PROBLEM: Standard implementation allocates new arrays in every merge call
  
  Standard (BAD for memory):
  ┌────────────────────────────────────────────┐
  │  merge(arr, 0, 0, 1)  → allocates 2 ints │
  │  merge(arr, 2, 2, 3)  → allocates 2 ints │
  │  merge(arr, 0, 1, 3)  → allocates 4 ints │
  │  merge(arr, 4, 4, 5)  → allocates 2 ints │
  │  merge(arr, 4, 5, 7)  → allocates 4 ints │
  │  merge(arr, 0, 3, 7)  → allocates 8 ints │
  │  Total allocations: 6 (garbage collection)│
  └────────────────────────────────────────────┘
  
  Optimized (GOOD):
  ┌────────────────────────────────────────────┐
  │  Allocate ONE aux array of size n upfront  │
  │  All merges reuse the same aux array       │
  │  Total allocations: 1                      │
  └────────────────────────────────────────────┘
```

```java
// Allocate aux once, pass it through
public static void mergeSort(int[] arr) {
    int[] aux = new int[arr.length];
    mergeSort(arr, aux, 0, arr.length - 1);
}

private static void mergeSort(int[] arr, int[] aux, int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    mergeSort(arr, aux, lo, mid);
    mergeSort(arr, aux, mid + 1, hi);
    merge(arr, aux, lo, mid, hi);
}
```

---

## Optimization 2: Skip Merge When Already Sorted

```
  If the last element of left half ≤ first element of right half,
  the halves are already in order — no merge needed!
  
  [1, 3, 5, | 7, 8, 9]
         5  ≤  7  → Already merged! Skip!
  
  This makes Merge Sort O(n) for already sorted input!
```

```java
private static void mergeSort(int[] arr, int[] aux, int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    mergeSort(arr, aux, lo, mid);
    mergeSort(arr, aux, mid + 1, hi);
    
    // OPTIMIZATION: Skip merge if already in order
    if (arr[mid] <= arr[mid + 1]) return;  // ← This one line!
    
    merge(arr, aux, lo, mid, hi);
}
```

```
  Impact on sorted data [1, 2, 3, ..., n]:
  
  Without optimization: O(n log n) — merges at every level
  With optimization:    O(n) — skips ALL merges!
  
  Every split results in arr[mid] ≤ arr[mid+1], so
  all merge calls are skipped. Only comparisons happen.
  Total: n-1 comparisons = O(n)
```

---

## Optimization 3: Use Insertion Sort for Small Subarrays

```
  When subarrays become small (n ≤ threshold), switch to Insertion Sort.
  
  ┌──────────────────────────────────────────────────────────┐
  │  Why?                                                    │
  │  1. Insertion Sort has lower overhead (no recursion)     │
  │  2. Insertion Sort is faster for small n                 │
  │  3. Reduces number of recursive calls significantly      │
  │                                                          │
  │  Typical threshold: 7-15 elements                        │
  │  Java Arrays.sort uses threshold of 7                    │
  └──────────────────────────────────────────────────────────┘
```

```java
private static final int THRESHOLD = 10;

private static void mergeSort(int[] arr, int[] aux, int lo, int hi) {
    // Switch to insertion sort for small subarrays
    if (hi - lo < THRESHOLD) {
        insertionSort(arr, lo, hi);
        return;
    }
    
    int mid = lo + (hi - lo) / 2;
    mergeSort(arr, aux, lo, mid);
    mergeSort(arr, aux, mid + 1, hi);
    
    if (arr[mid] <= arr[mid + 1]) return;  // Already sorted
    merge(arr, aux, lo, mid, hi);
}

private static void insertionSort(int[] arr, int lo, int hi) {
    for (int i = lo + 1; i <= hi; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= lo && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

```
  Effect on recursion tree (n = 64, threshold = 8):
  
  Without threshold:            With threshold:
  Level 0: 1 problem (n=64)    Level 0: 1 problem (n=64)
  Level 1: 2 (n=32)            Level 1: 2 (n=32)
  Level 2: 4 (n=16)            Level 2: 4 (n=16)
  Level 3: 8 (n=8)             Level 3: 8 (n=8) ← Insertion Sort!
  Level 4: 16 (n=4)            (stops here)
  Level 5: 32 (n=2)
  Level 6: 64 (n=1)
  
  Recursive calls: 127          Recursive calls: 15
  Merge calls: 63               Merge calls: 7
  
  ~85% fewer function calls!
```

---

## Optimization 4: Alternate Between Arrays

```
  Instead of copying to aux then merging back to arr,
  alternate roles: sometimes merge from arr to aux,
  sometimes from aux to arr.
  
  ┌──────────────────────────────────────────────────────┐
  │  Standard:  arr → aux (copy) → arr (merge back)     │
  │             Two copies per merge                     │
  │                                                      │
  │  Optimized: Even levels merge arr → aux              │
  │             Odd levels merge aux → arr               │
  │             Eliminates one copy per merge!           │
  └──────────────────────────────────────────────────────┘
```

---

## Natural Merge Sort

```
  Key idea: Detect existing sorted "runs" in the input
  instead of blindly splitting at the midpoint.
  
  Array: [3, 5, 8, | 2, 4, 9, | 1, 6, 7]
         ─────────   ─────────   ─────────
          Run 1        Run 2       Run 3
  
  Step 1: Identify natural runs
    Run 1: [3, 5, 8]    (indices 0-2)
    Run 2: [2, 4, 9]    (indices 3-5)
    Run 3: [1, 6, 7]    (indices 6-8)
  
  Step 2: Merge runs pairwise
    Merge Run 1 + Run 2: [2, 3, 4, 5, 8, 9]
    Run 3 stays:          [1, 6, 7]
  
  Step 3: Merge remaining
    [1, 2, 3, 4, 5, 6, 7, 8, 9]  ✓
  
  For nearly sorted data: very few runs → very fast!
  Already sorted: 1 run → O(n)!
```

```python
def natural_merge_sort(arr):
    n = len(arr)
    if n <= 1:
        return
    
    while True:
        # Find runs and merge pairs
        runs = find_runs(arr, n)
        if len(runs) <= 1:
            break  # One run = sorted!
        
        # Merge adjacent pairs of runs
        i = 0
        while i + 1 < len(runs):
            lo, mid = runs[i]
            _, hi = runs[i + 1]
            merge(arr, lo, mid, hi)
            i += 2

def find_runs(arr, n):
    """Find all ascending runs in the array."""
    runs = []
    start = 0
    for i in range(1, n):
        if arr[i] < arr[i - 1]:
            runs.append((start, i - 1))
            start = i
    runs.append((start, n - 1))
    return runs
```

---

## Space Comparison Summary

```
  Variant                  │ Time       │ Aux Space │ Notes
  ─────────────────────────┼────────────┼───────────┼──────────────
  Standard (new arrays)    │ O(n log n) │ O(n)      │ Many allocations
  Single aux array         │ O(n log n) │ O(n)      │ 1 allocation
  + Skip sorted check      │ O(n)-O(nlogn)│ O(n)    │ O(n) best case
  + Insertion cutoff       │ O(n log n) │ O(n)      │ ~15% faster
  + Alternating arrays     │ O(n log n) │ O(n)      │ Fewer copies
  Natural merge sort       │ O(n)-O(nlogn)│ O(n)    │ Adaptive!
  In-place merge sort      │ O(n log²n) │ O(1)      │ Complex, slower
  
  ┌──────────────────────────────────────────────────────────┐
  │  PRACTICAL RECOMMENDATION:                               │
  │  Use single aux array + skip sorted + insertion cutoff  │
  │  This is essentially what production sorts do.          │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Optimization | Benefit | Cost |
|-------------|---------|------|
| Single aux array | Fewer allocations, less GC | None |
| Skip if sorted | O(n) for sorted input | One extra comparison per merge |
| Insertion cutoff | ~15% faster, fewer calls | Slightly more complex code |
| Alternating arrays | Eliminates half the copies | More complex implementation |
| Natural merge sort | Adaptive to existing order | Run detection overhead |

---

## Quick Revision Questions

1. **Why is allocating one auxiliary array better than multiple small ones?**
2. **What single-line optimization makes Merge Sort O(n) for sorted input?**
3. **Why use Insertion Sort for small subarrays? What's a typical threshold?**
4. **How does Natural Merge Sort adapt to nearly sorted input?**
5. **Can Merge Sort ever be truly in-place? What's the trade-off?**
6. **Which optimizations does Java's built-in sort use?**

---

[← Previous: Complexity Analysis](04-complexity-analysis.md) | [Next: Iterative Merge Sort →](06-iterative-merge-sort.md)
