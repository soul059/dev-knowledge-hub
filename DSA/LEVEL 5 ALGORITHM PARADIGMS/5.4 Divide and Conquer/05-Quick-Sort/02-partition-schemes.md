# Chapter 2: Partition Schemes

## Overview

The **partition** step is the heart of Quick Sort. Different partition schemes offer different tradeoffs in performance, stability, and handling of duplicates. This chapter covers **Lomuto**, **Hoare**, and **Three-Way (Dutch National Flag)** partitioning.

---

## 1. Lomuto Partition Scheme

**Strategy**: Use the last element as pivot. Maintain boundary between "≤ pivot" and "> pivot" regions.

```
function lomutoPartition(arr, lo, hi):
    pivot = arr[hi]
    i = lo - 1
    
    for j = lo to hi - 1:
        if arr[j] <= pivot:
            i++
            swap(arr[i], arr[j])
    
    swap(arr[i + 1], arr[hi])
    return i + 1
```

### Trace

```
[3, 7, 8, 5, 2, 1, 9, 4]   pivot = 4
 j                    hi
 i=-1

j=0: 3 ≤ 4 → i=0, swap(0,0) → [3, 7, 8, 5, 2, 1, 9, 4]
j=1: 7 > 4 → skip
j=2: 8 > 4 → skip
j=3: 5 > 4 → skip
j=4: 2 ≤ 4 → i=1, swap(1,4) → [3, 2, 8, 5, 7, 1, 9, 4]
j=5: 1 ≤ 4 → i=2, swap(2,5) → [3, 2, 1, 5, 7, 8, 9, 4]
j=6: 9 > 4 → skip

Final: swap(i+1, hi) = swap(3, 7)
[3, 2, 1, 4, 7, 8, 9, 5]
          ↑ pivot at index 3
```

### Properties

```
✓ Simple to understand and implement
✓ Always uses last element as pivot
✗ More swaps than Hoare (about 3× more on average)
✗ O(n²) on already-sorted arrays (pivot always extreme)
✗ Not stable
```

---

## 2. Hoare Partition Scheme

**Strategy**: Use two pointers moving toward each other. More efficient — fewer swaps on average.

```
function hoarePartition(arr, lo, hi):
    pivot = arr[lo]              // first element as pivot
    i = lo - 1
    j = hi + 1
    
    while true:
        do: i++ while arr[i] < pivot   // find element ≥ pivot from left
        do: j-- while arr[j] > pivot   // find element ≤ pivot from right
        
        if i >= j:
            return j                    // return partition boundary
        
        swap(arr[i], arr[j])
```

### Trace

```
[7, 1, 3, 9, 2, 5, 8, 4]   pivot = 7
 ↑i                     ↑j  (conceptually start outside array)

Round 1:
  i moves right: arr[0]=7 ≥ 7 → stop at i=0
  j moves left:  arr[7]=4 ≤ 7 → stop at j=7
  i(0) < j(7) → swap(arr[0], arr[7])
  [4, 1, 3, 9, 2, 5, 8, 7]

Round 2:
  i moves right: arr[1]=1 < 7, arr[2]=3 < 7, arr[3]=9 ≥ 7 → i=3
  j moves left:  arr[7]=7 ≤ 7 → j=7... wait, arr[6]=8 > 7, arr[5]=5 ≤ 7 → j=5
  i(3) < j(5) → swap(arr[3], arr[5])
  [4, 1, 3, 5, 2, 9, 8, 7]

Round 3:
  i moves right: arr[4]=2 < 7, arr[5]=9 ≥ 7 → i=5
  j moves left:  arr[5]=9 > 7, arr[4]=2 ≤ 7 → j=4
  i(5) ≥ j(4) → return j=4

Result: arr[0..4] ≤ 7, arr[5..7] ≥ 7
[4, 1, 3, 5, 2 | 9, 8, 7]
```

### Important: Hoare vs Lomuto in quickSort

```
// With Lomuto: pivot is at returned index
quickSort(arr, lo, pivotIndex - 1)
quickSort(arr, pivotIndex + 1, hi)

// With Hoare: pivot may NOT be at returned position
// The returned j is a BOUNDARY, not pivot position
quickSort(arr, lo, j)          // includes boundary
quickSort(arr, j + 1, hi)
```

### Properties

```
✓ ~3× fewer swaps than Lomuto on average
✓ Better practical performance
✓ Works well with many duplicates (if properly implemented)
✗ Slightly harder to understand
✗ Pivot not placed in final position
✗ More subtle edge cases
✗ Not stable
```

---

## 3. Three-Way Partition (Dutch National Flag)

**Strategy**: Partition into THREE regions: < pivot, = pivot, > pivot. Handles duplicates optimally.

```
function threeWayPartition(arr, lo, hi):
    pivot = arr[lo]
    lt = lo        // arr[lo..lt-1]  < pivot
    gt = hi        // arr[gt+1..hi]  > pivot
    i = lo         // arr[lt..i-1]  == pivot
                   // arr[i..gt]     unexplored
    
    while i <= gt:
        if arr[i] < pivot:
            swap(arr[lt], arr[i])
            lt++; i++
        else if arr[i] > pivot:
            swap(arr[i], arr[gt])
            gt--              // don't increment i!
        else:  // arr[i] == pivot
            i++
    
    return (lt, gt)           // range of equal elements
```

### Trace

```
[4, 9, 4, 1, 7, 4, 8, 2, 4]   pivot = 4

lt=0, i=0, gt=8

i=0: arr[0]=4 == 4 → i=1
     [4, 9, 4, 1, 7, 4, 8, 2, 4]
      lt  i                    gt

i=1: arr[1]=9 > 4 → swap(arr[1],arr[8]), gt=7
     [4, 4, 4, 1, 7, 4, 8, 2, 9]
      lt  i                 gt

i=1: arr[1]=4 == 4 → i=2
i=2: arr[2]=4 == 4 → i=3
i=3: arr[3]=1 < 4 → swap(arr[0],arr[3]), lt=1, i=4
     [1, 4, 4, 4, 7, 4, 8, 2, 9]
         lt       i          gt

i=4: arr[4]=7 > 4 → swap(arr[4],arr[7]), gt=6
     [1, 4, 4, 4, 2, 4, 8, 7, 9]
         lt       i       gt

i=4: arr[4]=2 < 4 → swap(arr[1],arr[4]), lt=2, i=5
     [1, 2, 4, 4, 4, 4, 8, 7, 9]
            lt       i    gt

i=5: arr[5]=4 == 4 → i=6
i=6: arr[6]=8 > 4 → swap(arr[6],arr[6]), gt=5
     [1, 2, 4, 4, 4, 4, 8, 7, 9]
            lt          gt
                         i

i=6 > gt=5 → STOP

Result: lt=2, gt=5
[1, 2, | 4, 4, 4, 4 | 8, 7, 9]
 < 4      == 4          > 4
```

### Usage in Quick Sort

```
function quickSort3Way(arr, lo, hi):
    if lo >= hi: return
    (lt, gt) = threeWayPartition(arr, lo, hi)
    quickSort3Way(arr, lo, lt - 1)     // sort elements < pivot
    // skip arr[lt..gt] — all equal to pivot!
    quickSort3Way(arr, gt + 1, hi)     // sort elements > pivot
```

### Properties

```
✓ OPTIMAL for arrays with many duplicates
✓ O(n) for arrays where all elements are the same
✓ Avoids re-processing equal elements
✗ Slightly more complex than Lomuto
✗ No benefit if all elements are unique
```

---

## Comparison of Partition Schemes

```
                    Lomuto          Hoare        Three-Way
                    ──────          ─────        ─────────
Pivot position:     Last element    First elem   First elem
Pointer movement:   One direction   Both ends    Both ends
Average swaps:      ~n/2            ~n/6         ~n/3
Duplicates:         O(n²) worst     Better       O(n) optimal
Complexity:         Simple          Moderate     Moderate
Pivot in place:     Yes             No           Yes (range)
Stable:             No              No           No

Best for:
  Lomuto    → Teaching, simple implementation
  Hoare     → General purpose, efficiency
  Three-Way → Many duplicate values
```

---

## Invariants Summary

```
LOMUTO:
┌──────────┬──────────┬────────────┬───────┐
│  ≤ pivot │  > pivot │ unexplored │ pivot │
└──────────┴──────────┴────────────┴───────┘
lo        i  i+1      j          hi-1  hi

HOARE:
┌──────────┬────────────┬──────────┐
│ ≤ pivot  │ unexplored │ ≥ pivot  │
└──────────┴────────────┴──────────┘
lo        i              j        hi

THREE-WAY:
┌──────────┬──────────┬────────────┬──────────┐
│ < pivot  │ = pivot  │ unexplored │ > pivot  │
└──────────┴──────────┴────────────┴──────────┘
lo       lt-1  lt    i-1  i      gt  gt+1    hi
```

---

## Summary Table

| Scheme | Swaps (avg) | Duplicates | Simplicity | Best Use |
|--------|-------------|------------|------------|----------|
| Lomuto | ~n/2 | Poor | High | Teaching |
| Hoare | ~n/6 | Good | Medium | Performance |
| Three-Way | ~n/3 | Excellent | Medium | Many duplicates |

---

## Quick Revision Questions

1. **What is the key difference between Lomuto and Hoare partition?**
2. **Why does Lomuto's scheme degrade to O(n²) on sorted arrays?**
3. **Why doesn't Hoare's partition place the pivot in its final position?**
4. **What problem does three-way partitioning solve?**
5. **In three-way partition, why don't we increment i when swapping with gt?**
6. **Which partition scheme results in the fewest swaps on average?**

---

[← Previous: Quick Sort Algorithm](01-quick-sort-algorithm.md) | [Next: Pivot Selection →](03-pivot-selection.md)
