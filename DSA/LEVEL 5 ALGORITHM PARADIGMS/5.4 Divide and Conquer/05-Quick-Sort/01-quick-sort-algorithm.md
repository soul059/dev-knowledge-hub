# Chapter 1: Quick Sort Algorithm

## Overview

**Quick Sort** is the most widely used sorting algorithm in practice. Unlike Merge Sort where the work is in the combine step, Quick Sort does its work in the **divide** step through **partitioning**. It sorts **in-place** with O(log n) stack space and achieves **O(n log n) average case** — often faster than Merge Sort due to better cache performance.

---

## The Algorithm at a Glance

```
┌───────────────────────────────────────────────────────────┐
│  QUICK SORT                                               │
│                                                           │
│  1. DIVIDE:  Choose a pivot; partition array so that      │
│              elements ≤ pivot are left, > pivot are right │
│  2. CONQUER: Recursively sort left and right partitions   │
│  3. COMBINE: Nothing! (array is sorted in-place)          │
│                                                           │
│  Key insight: Work is done in DIVIDE (partitioning)       │
│  Contrast with Merge Sort: work is in COMBINE (merging)  │
└───────────────────────────────────────────────────────────┘
```

---

## Visual: How Quick Sort Works

```
  [7, 2, 1, 6, 8, 5, 3, 4]     pivot = 4
                                 
  Partition around pivot 4:
  [2, 1, 3] [4] [7, 6, 8, 5]    ← 4 is in final position!
      ↓              ↓
  [1] [2] [3]   [5] [6] [7, 8]  ← recursive sorts
                           ↓
                      [7] [8]
  
  Result: [1, 2, 3, 4, 5, 6, 7, 8]
  
  No merge needed — everything is sorted in-place!
```

---

## Pseudocode

```
function quickSort(arr, lo, hi):
    if lo < hi:
        pivotIndex = partition(arr, lo, hi)
        quickSort(arr, lo, pivotIndex - 1)    // sort left
        quickSort(arr, pivotIndex + 1, hi)    // sort right

// Lomuto Partition Scheme
function partition(arr, lo, hi):
    pivot = arr[hi]                // last element as pivot
    i = lo - 1                    // boundary of "small" elements
    
    for j = lo to hi - 1:
        if arr[j] <= pivot:
            i++
            swap(arr[i], arr[j])
    
    swap(arr[i + 1], arr[hi])     // place pivot in correct position
    return i + 1                   // return pivot's final index
```

---

## Step-by-Step Trace (Lomuto)

```
Array: [7, 2, 1, 6, 8, 5, 3, 4]    pivot = 4 (last element)
        ↑i                   ↑
       lo=-1                 hi

i starts at -1 (before lo)

j=0: arr[0]=7 > 4  → skip
j=1: arr[1]=2 ≤ 4  → i=0, swap(arr[0],arr[1])
     [2, 7, 1, 6, 8, 5, 3, 4]
      ↑i

j=2: arr[2]=1 ≤ 4  → i=1, swap(arr[1],arr[2])
     [2, 1, 7, 6, 8, 5, 3, 4]
         ↑i

j=3: arr[3]=6 > 4  → skip
j=4: arr[4]=8 > 4  → skip
j=5: arr[5]=5 > 4  → skip

j=6: arr[6]=3 ≤ 4  → i=2, swap(arr[2],arr[6])
     [2, 1, 3, 6, 8, 5, 7, 4]
            ↑i

End: swap(arr[i+1], arr[hi]) = swap(arr[3], arr[7])
     [2, 1, 3, 4, 8, 5, 7, 6]
               ↑ pivot at index 3

Return 3

Left:  [2, 1, 3]  → quickSort(arr, 0, 2)
Right: [8, 5, 7, 6] → quickSort(arr, 4, 7)
```

---

## Partition Invariant

```
During partition, the array maintains this structure:

┌────────────┬──────────────┬─────────────┬───────┐
│  ≤ pivot   │   > pivot    │  unexplored │ pivot │
└────────────┴──────────────┴─────────────┴───────┘
 lo         i  i+1          j            hi-1  hi

After partition completes:
┌────────────┬───────┬──────────────┐
│  ≤ pivot   │ pivot │   > pivot    │
└────────────┴───────┴──────────────┘
 lo        i+1      i+2           hi
```

---

## Merge Sort vs Quick Sort

```
┌──────────────────────────────────────────────────────┐
│          MERGE SORT         │       QUICK SORT       │
├─────────────────────────────┼────────────────────────┤
│  Divide: trivial (split)    │  Divide: partition ← ! │
│  Combine: merge ← !         │  Combine: nothing      │
│  Stable: yes                │  Stable: no            │
│  Space: O(n) extra          │  Space: O(log n) stack │
│  Worst: O(n log n)         │  Worst: O(n²)          │
│  Cache: poor (random access)│  Cache: good (linear)  │
└─────────────────────────────┴────────────────────────┘

In practice, Quick Sort is ~2-3× faster than Merge Sort
due to better cache performance and lower constant factors.
```

---

## The D&C Structure

```
DIVIDE:   Choose pivot, partition into three parts:
          [elements ≤ pivot] [pivot] [elements > pivot]
          This is where ALL the work happens.
          Cost: O(n) for one partition.

CONQUER:  Recursively sort the two partitions.
          Two subproblems of (ideally) size n/2 each.

COMBINE:  Nothing to do!
          Since partitioning is in-place and pivot is
          in its final position, the array is already sorted.
```

---

## Why the Pivot Matters

```
Good pivot (near median):
  [n/2 elements] [pivot] [n/2 elements]
  T(n) = 2T(n/2) + O(n) → O(n log n)

Bad pivot (extreme values):
  [] [pivot] [n-1 elements]          (or vice versa)
  T(n) = T(n-1) + O(n) → O(n²)

                 UNBALANCED TREE
                /
               n
              / \
            n-1   0
            / \
          n-2   0
          / \
        n-3   0
        ...
        Total work: n + (n-1) + ... + 1 = O(n²)
```

---

## Summary Table

| Property | Value |
|----------|-------|
| Time (Best) | O(n log n) |
| Time (Average) | O(n log n) |
| Time (Worst) | O(n²) |
| Space | O(log n) average stack |
| Stable | No |
| In-place | Yes |
| Adaptive | No |
| Where work happens | Divide step (partitioning) |

---

## Quick Revision Questions

1. **In which D&C step does Quick Sort do its main work?**
2. **What is the role of the pivot in Quick Sort?**
3. **Why does Quick Sort not need a combine step?**
4. **What causes Quick Sort's worst case of O(n²)?**
5. **Why is Quick Sort often faster than Merge Sort in practice despite having a worse worst case?**

---

[Next: Partition Schemes →](02-partition-schemes.md)
