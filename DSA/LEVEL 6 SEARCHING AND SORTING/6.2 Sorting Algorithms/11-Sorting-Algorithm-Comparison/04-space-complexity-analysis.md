# Unit 11: Sorting Algorithm Comparison — Space Complexity Analysis

[← Previous: Time Complexity Deep Dive](03-time-complexity-deep-dive.md) | [Next: Choosing the Right Algorithm →](05-choosing-the-right-algorithm.md)

---

## Overview

Memory usage is often the deciding factor when choosing a sorting algorithm. This chapter analyzes **auxiliary space** (extra memory beyond the input array) for every algorithm, including often-overlooked costs like recursion stack space.

---

## Space Complexity Summary

```
  ┌──────────────┬──────────────┬───────────────────────────────────┐
  │  Algorithm   │ Extra Space  │ What Uses the Space?              │
  ├──────────────┼──────────────┼───────────────────────────────────┤
  │ Bubble Sort  │ O(1)         │ Temp variable for swap            │
  │ Selection    │ O(1)         │ Temp variable, min index          │
  │ Insertion    │ O(1)         │ Key variable                      │
  │ Quick Sort   │ O(lg n)*     │ Recursion stack (balanced)        │
  │ Heap Sort    │ O(1)         │ Temp for swap                     │
  │ Merge Sort   │ O(n)         │ Auxiliary array for merging       │
  │ Counting     │ O(n + k)     │ Count array + output array        │
  │ Radix Sort   │ O(n + k)     │ Counting sort arrays per digit   │
  │ Bucket Sort  │ O(n + k)     │ k bucket lists + n elements      │
  └──────────────┴──────────────┴───────────────────────────────────┘
  
  * Quick Sort worst case: O(n) stack depth (unbalanced partitions)
```

---

## In-Place vs Not In-Place

```
  ┌──────────────────────────────────────────────────────────────┐
  │  IN-PLACE: Uses O(1) or O(lg n) extra space               │
  │  (modifies the array directly, no large auxiliary storage)  │
  ├──────────────────────────────────────────────────────────────┤
  │  Bubble Sort        O(1)                                    │
  │  Selection Sort     O(1)                                    │
  │  Insertion Sort     O(1)                                    │
  │  Heap Sort          O(1)       ← best: O(1) + O(n lg n)   │
  │  Quick Sort         O(lg n)    ← recursion stack           │
  ├──────────────────────────────────────────────────────────────┤
  │  NOT IN-PLACE: Uses O(n) or more extra space              │
  ├──────────────────────────────────────────────────────────────┤
  │  Merge Sort         O(n)                                    │
  │  Counting Sort      O(n + k)                               │
  │  Radix Sort         O(n + k)                               │
  │  Bucket Sort        O(n + k)                               │
  └──────────────────────────────────────────────────────────────┘
```

---

## Deep Dive: Recursion Stack Space

### Quick Sort

```
  BALANCED PARTITIONS (best case):
  
  Depth = lg n  →  O(lg n) stack frames
  
  quickSort(0, n-1)
    quickSort(0, n/2-1)          ← frame 1
      quickSort(0, n/4-1)        ← frame 2
        ...                       ← lg n frames max
  
  UNBALANCED PARTITIONS (worst case):
  
  Depth = n  →  O(n) stack frames!
  
  quickSort(0, n-1)
    quickSort(0, n-2)            ← frame 1
      quickSort(0, n-3)          ← frame 2
        ...                       ← n-1 frames!
  
  FIX: Tail-call optimization
  
  void quickSort(int[] arr, int lo, int hi) {
      while (lo < hi) {
          int pivot = partition(arr, lo, hi);
          if (pivot - lo < hi - pivot) {
              quickSort(arr, lo, pivot - 1);   // recurse smaller
              lo = pivot + 1;                  // iterate larger
          } else {
              quickSort(arr, pivot + 1, hi);   // recurse smaller
              hi = pivot - 1;                  // iterate larger
          }
      }
  }
  
  This GUARANTEES O(lg n) stack depth even in worst case!
```

### Merge Sort

```
  Recursion depth is always lg n → O(lg n) stack space.
  
  But the AUXILIARY ARRAY is O(n) — this dominates.
  
  Total space: O(n) + O(lg n) = O(n)
  
  Optimization: allocate ONE auxiliary array of size n,
  pass it to all recursive calls.
  
  void mergeSort(int[] arr, int[] aux, int lo, int hi) {
      if (lo >= hi) return;
      int mid = lo + (hi - lo) / 2;
      mergeSort(arr, aux, lo, mid);
      mergeSort(arr, aux, mid + 1, hi);
      merge(arr, aux, lo, mid, hi);  // uses shared aux
  }
```

---

## Memory Layout Visualization

```
  Merge Sort Memory:
  ┌────────────────────────────────┐
  │  Original Array (n elements)  │  ← always present
  ├────────────────────────────────┤
  │  Auxiliary Array (n elements) │  ← O(n) extra
  ├──────────┐                    │
  │  Stack   │                    │
  │  lg n    │                    │
  │  frames  │                    │
  └──────────┘────────────────────┘
  Total: O(n)
  
  
  Heap Sort Memory:
  ┌────────────────────────────────┐
  │  Original Array (n elements)  │  ← always present
  ├──────────┐                    │
  │  O(1)    │  ← temp swap var  │
  └──────────┘────────────────────┘
  Total: O(1) extra     ← most space-efficient O(n lg n) sort!
  
  
  Quick Sort Memory:
  ┌────────────────────────────────┐
  │  Original Array (n elements)  │  ← always present
  ├──────────┐                    │
  │  Stack   │                    │
  │  lg n    │                    │
  │  frames  │                    │
  └──────────┘────────────────────┘
  Total: O(lg n) extra (with tail-call optimization)
  
  
  Counting Sort Memory:
  ┌────────────────────────────────┐
  │  Original Array (n elements)  │
  ├────────────────────────────────┤
  │  Count Array (k elements)     │
  ├────────────────────────────────┤
  │  Output Array (n elements)    │
  └────────────────────────────────┘
  Total: O(n + k) extra
```

---

## When Space Matters

```
  SCENARIO 1: Embedded System (2 KB RAM)
  ┌──────────────────────────────────────────────────┐
  │  Array of 256 sensor readings (1 KB)             │
  │  Available extra: 1 KB                           │
  │                                                   │
  │  ✓ Insertion Sort — O(1) extra                   │
  │  ✓ Heap Sort — O(1) extra                        │
  │  ✗ Merge Sort — needs 1 KB extra → too much!     │
  └──────────────────────────────────────────────────┘
  
  SCENARIO 2: Sorting 1 billion integers (4 GB)
  ┌──────────────────────────────────────────────────┐
  │  Array: 4 GB                                     │
  │  Available RAM: 8 GB                             │
  │                                                   │
  │  ✓ Quick Sort — O(lg n) = ~120 bytes stack      │
  │  ✓ Heap Sort — O(1) extra                        │
  │  ✗ Merge Sort — needs 4 GB extra → 8 GB total   │
  │  ✗ Counting Sort — k could be huge (4 billion)  │
  └──────────────────────────────────────────────────┘
  
  SCENARIO 3: Sorting linked list (no random access)
  ┌──────────────────────────────────────────────────┐
  │  Merge Sort → O(1) extra for linked lists!      │
  │  (rearrange pointers, no auxiliary array needed) │
  │  Quick Sort → O(lg n) stack but bad cache perf  │
  │  Heap Sort → needs array representation         │
  └──────────────────────────────────────────────────┘
```

---

## Space-Time Trade-offs

```
  ┌──────────────────┬──────────────┬──────────────┬────────────────┐
  │  Want             │ Algorithm   │ Time         │ Space          │
  ├──────────────────┼──────────────┼──────────────┼────────────────┤
  │  Min time         │ Quick Sort  │ O(n lg n)    │ O(lg n)        │
  │  Min space        │ Heap Sort   │ O(n lg n)    │ O(1)           │
  │  Balanced         │ Quick Sort  │ O(n lg n)    │ O(lg n)        │
  │  Min time, O(n)   │ Counting    │ O(n+k)       │ O(n+k)   more  │
  │  Guaranteed time  │ Merge Sort  │ O(n lg n)    │ O(n)     more  │
  │  Min writes       │ Selection   │ O(n²)        │ O(1)           │
  │  Adaptive         │ Insertion   │ O(n)~O(n²)   │ O(1)           │
  └──────────────────┴──────────────┴──────────────┴────────────────┘
  
  General pattern:
  • More space → faster algorithms possible
  • Less space → either slower or data-dependent
  • O(1) space + O(n lg n) time = Heap Sort (only option!)
```

---

## The Ideal Sort Doesn't Exist

```
  Can we have ALL of these at once?
  
  ✓ O(n lg n) worst case
  ✓ O(1) extra space  
  ✓ Stable
  ✓ Adaptive
  ✓ Good cache performance
  
  NO! This is the "sorting trilemma":
  
  ┌──────────────────────────────────────────────────┐
  │  Pick ANY TWO:                                   │
  │                                                   │
  │  1. O(n lg n) worst-case + Stable → Merge Sort  │
  │     but O(n) space                               │
  │                                                   │
  │  2. O(n lg n) worst-case + O(1) space → Heap    │
  │     but unstable, poor cache                     │
  │                                                   │
  │  3. Stable + O(1) space → Insertion Sort         │
  │     but O(n²) worst-case                         │
  │                                                   │
  │  No known algorithm simultaneously achieves:     │
  │  O(n lg n) + O(1) space + stable + practical     │
  └──────────────────────────────────────────────────┘
  
  Block Merge Sort (WikiSort) comes close but has
  high constant factors.
```

---

## Summary Table

| Algorithm | Auxiliary Space | In-Place? | Notes |
|-----------|----------------|-----------|-------|
| Bubble | O(1) | Yes | Just a temp variable |
| Selection | O(1) | Yes | Min index + temp |
| Insertion | O(1) | Yes | Key variable only |
| Quick Sort | O(lg n) avg | Yes | Stack depth; O(n) worst without optimization |
| Heap Sort | O(1) | Yes | Best space for O(n lg n) |
| Merge Sort | O(n) | No | Auxiliary array dominates |
| Counting | O(n+k) | No | Count + output arrays |
| Radix | O(n+k) | No | Per-digit counting arrays |
| Bucket | O(n+k) | No | Bucket lists |

---

## Quick Revision Questions

1. **Which O(n log n) sort uses the least extra space? How much?**
2. **What determines Quick Sort's stack depth? How can you guarantee O(lg n)?**
3. **Why does Merge Sort need O(n) space while Merge Sort on linked lists needs O(1)?**
4. **Why is there no sorting algorithm that is simultaneously O(n lg n), O(1) space, and stable?**
5. **For an embedded system with 2KB free memory, which sorts are feasible for 500 elements?**
6. **Compare the total memory footprint of Counting Sort vs Heap Sort for sorting 10⁶ integers in range [0, 10⁹].**

---

[← Previous: Time Complexity Deep Dive](03-time-complexity-deep-dive.md) | [Next: Choosing the Right Algorithm →](05-choosing-the-right-algorithm.md)
