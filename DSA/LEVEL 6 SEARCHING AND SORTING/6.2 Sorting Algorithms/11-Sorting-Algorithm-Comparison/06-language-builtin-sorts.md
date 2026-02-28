# Unit 11: Sorting Algorithm Comparison — Language Built-in Sorts

[← Previous: Choosing the Right Algorithm](05-choosing-the-right-algorithm.md) | [Next: Unit 12 — Special Sorting Problems →](../12-Special-Sorting-Problems/01-dutch-national-flag.md)

---

## Overview

Every major programming language provides a built-in sort function. Understanding **what algorithm they use and why** gives you insight into real-world sorting engineering and helps you make informed decisions.

---

## Java: Dual-Pivot QuickSort + TimSort

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Java uses TWO different sorts depending on data type:      │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  Arrays.sort(int[]) → Dual-Pivot QuickSort                 │
  │  Arrays.sort(Object[]) → TimSort (stable)                  │
  │  Collections.sort() → TimSort (delegates to Arrays.sort)   │
  │                                                              │
  │  Why different?                                              │
  │  • Primitives: stability doesn't matter (no identity)      │
  │    → QuickSort is faster (cache-friendly, in-place)        │
  │  • Objects: stability matters (equal objects stay ordered)  │
  │    → TimSort provides stability guarantee                  │
  └──────────────────────────────────────────────────────────────┘
```

### Java's Dual-Pivot QuickSort

```
  Standard QuickSort: 1 pivot, 2 partitions
  Dual-Pivot QuickSort: 2 pivots, 3 partitions
  
  [elements < P1 | P1 ≤ elements ≤ P2 | elements > P2]
  
  Advantages:
  • ~20% fewer comparisons than single-pivot
  • Better cache utilization
  • Still O(n lg n) average
  
  Implementation details:
  • Switches to Insertion Sort for n < 47
  • Uses Counting Sort for bytes, shorts when range ≤ 3.4n
  • Randomized to avoid worst-case on adversarial inputs
```

### Java's TimSort

```
  TimSort (invented by Tim Peters, 2002):
  
  Step 1: Find natural "runs" (already sorted subsequences)
  Step 2: Extend short runs to minRun length using Insertion Sort
  Step 3: Merge runs using a merge stack
  
  Properties:
  • Stable ✓
  • O(n lg n) worst case ✓
  • O(n) on already sorted data ✓
  • Adaptive to existing order ✓
  • O(n) extra space
  • minRun typically 32-64
  
  Example on partially sorted data:
  [1, 3, 5, 7, | 2, 4, 6, 8, | 9, 10]
       run 1          run 2       run 3
  
  TimSort detects these runs and merges them efficiently.
  On already sorted data: only 1 run → O(n) time!
```

---

## Python: TimSort

```
  ┌──────────────────────────────────────────────────────────────┐
  │  sorted() and list.sort() both use TimSort                  │
  │                                                              │
  │  Python's TimSort:                                           │
  │  • Invented FOR Python by Tim Peters                        │
  │  • minRun = 32 (typically)                                  │
  │  • Galloping mode for merging (exponential search)          │
  │  • Stable ✓                                                 │
  │  • O(n lg n) worst case                                     │
  │  • O(n) best case (sorted data)                             │
  └──────────────────────────────────────────────────────────────┘
  
  # Python sort is always stable
  students = [("Alice", 85), ("Bob", 90), ("Carol", 85)]
  students.sort(key=lambda x: x[1])
  # Result: [("Alice", 85), ("Carol", 85), ("Bob", 90)]
  #          Alice still before Carol ✓ (stable)
  
  # Timsort's galloping mode:
  # When merging and one run consistently "wins",
  # switch to exponential search to skip ahead.
  # This makes merging [1,2,3,4,...,1000] + [1001,...,2000]
  # take O(n) instead of O(n log n).
```

---

## C++: Introsort (std::sort) + Stable Sort

```
  ┌──────────────────────────────────────────────────────────────┐
  │  std::sort() → Introsort (NOT stable)                      │
  │  std::stable_sort() → Merge Sort variant (stable)          │
  │  std::partial_sort() → Heapsort-based                      │
  │  std::nth_element() → Introselect                          │
  └──────────────────────────────────────────────────────────────┘
  
  Introsort (Introspective Sort):
  
  1. Start with QuickSort
  2. If recursion depth exceeds 2 × lg(n):
     → Switch to HeapSort (guarantees O(n lg n))
  3. For partitions < 16 elements:
     → Switch to Insertion Sort
  
  ┌──────────────────────────────────────────────────────┐
  │  Quick Sort:  Best average performance              │
  │  Heap Sort:   Worst-case safety net                 │
  │  Insertion:   Best for small partitions             │
  │                                                      │
  │  Result: O(n lg n) GUARANTEED, fast in practice     │
  └──────────────────────────────────────────────────────┘
```

```cpp
// Conceptual Introsort implementation
void introsort(int* arr, int n) {
    int maxDepth = 2 * (int)log2(n);
    introsortHelper(arr, 0, n - 1, maxDepth);
    insertionSort(arr, 0, n - 1);  // final pass
}

void introsortHelper(int* arr, int lo, int hi, int depth) {
    if (hi - lo < 16) return;  // leave for insertion sort
    
    if (depth == 0) {
        heapSort(arr, lo, hi);  // fallback
        return;
    }
    
    int pivot = partition(arr, lo, hi);
    introsortHelper(arr, lo, pivot - 1, depth - 1);
    introsortHelper(arr, pivot + 1, hi, depth - 1);
}
```

---

## JavaScript: TimSort (V8 Engine)

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Array.prototype.sort() → TimSort (since V8 v7.0, 2018)    │
  │                                                              │
  │  Before 2018: QuickSort (unstable!)                         │
  │  After 2018:  TimSort (stable, per ECMAScript spec)         │
  │                                                              │
  │  ECMAScript 2019 requires sort to be stable.                │
  └──────────────────────────────────────────────────────────────┘
  
  // JavaScript sort uses string comparison by default!
  [10, 9, 80].sort()           // [10, 80, 9] ← WRONG!
  [10, 9, 80].sort((a,b) => a - b)  // [9, 10, 80] ✓
```

---

## Go: Pattern-Defeating QuickSort (pdqsort)

```
  ┌──────────────────────────────────────────────────────────────┐
  │  sort.Sort() → pdqsort (since Go 1.19)                     │
  │  sort.Stable() → stable sort variant                        │
  │                                                              │
  │  pdqsort: pattern-defeating quicksort                       │
  │  • Based on Introsort but smarter                           │
  │  • Detects patterns (sorted, reverse, few unique)           │
  │  • Falls back to heap sort if needed                        │
  │  • O(n) on already sorted data                              │
  │  • O(n lg n) worst case guaranteed                          │
  │  • NOT stable                                               │
  └──────────────────────────────────────────────────────────────┘
```

---

## Rust: pdqsort + TimSort

```
  ┌──────────────────────────────────────────────────────────────┐
  │  .sort_unstable() → pdqsort (faster, not stable)           │
  │  .sort()          → TimSort variant (stable)               │
  │                                                              │
  │  Rust explicitly separates stable and unstable sorts:       │
  │  • Use .sort() when stability matters                      │
  │  • Use .sort_unstable() for maximum speed                  │
  └──────────────────────────────────────────────────────────────┘
```

---

## Comparison of Language Built-in Sorts

```
  ┌──────────────┬──────────────────────┬────────┬──────────────┐
  │  Language    │ Algorithm            │ Stable │ Worst Case   │
  ├──────────────┼──────────────────────┼────────┼──────────────┤
  │  Java (prim) │ Dual-Pivot QuickSort│ No     │ O(n lg n)    │
  │  Java (obj)  │ TimSort             │ Yes    │ O(n lg n)    │
  │  Python      │ TimSort             │ Yes    │ O(n lg n)    │
  │  C++ sort    │ Introsort           │ No     │ O(n lg n)    │
  │  C++ stable  │ Merge Sort          │ Yes    │ O(n lg n)    │
  │  JavaScript  │ TimSort             │ Yes    │ O(n lg n)    │
  │  Go          │ pdqsort             │ No     │ O(n lg n)    │
  │  Rust sort   │ TimSort variant     │ Yes    │ O(n lg n)    │
  │  Rust unstable│ pdqsort            │ No     │ O(n lg n)    │
  │  C# (LINQ)   │ Introsort           │ No     │ O(n lg n)    │
  │  Swift       │ Introsort           │ No     │ O(n lg n)*   │
  └──────────────┴──────────────────────┴────────┴──────────────┘
  
  * Swift also provides .sorted() which may use TimSort
  
  Pattern: ALL modern languages guarantee O(n lg n) worst case
  using hybrid algorithms!
```

---

## The Hybrid Design Pattern

```
  All modern sorts follow this pattern:
  
  ┌─────────────────────────────────────────────────────────────┐
  │                                                             │
  │     Large partition  →  QuickSort (fastest average)        │
  │           │                                                 │
  │     Depth too deep?  →  HeapSort (safety net)              │
  │           │                                                 │
  │     Small partition  →  InsertionSort (low overhead)       │
  │           │                                                 │
  │     Detect patterns  →  Skip / optimize (adaptive)        │
  │                                                             │
  │  This is why you should almost NEVER write your own sort   │
  │  in production code. Use the built-in.                     │
  └─────────────────────────────────────────────────────────────┘
```

---

## When to Override Built-in Sort

```
  Use built-in sort UNLESS:
  
  1. Data has special structure you can exploit
     → Counting sort for small integers
     → Radix sort for fixed-width keys
     → Bucket sort for uniform floats
  
  2. Memory is extremely constrained
     → Heap sort (O(1) extra space)
  
  3. Data is streaming / online
     → Insertion sort or priority queue
  
  4. Data is on disk (too large for RAM)
     → External merge sort
  
  5. Sorting is the bottleneck AND you've profiled
     → Custom sort tuned to your data distribution
```

---

## Summary Table

| Language | Default Sort | Stable? | Approach |
|----------|-------------|---------|----------|
| Java (primitives) | Dual-Pivot QS | No | QuickSort + InsertionSort |
| Java (objects) | TimSort | Yes | MergeSort + InsertionSort |
| Python | TimSort | Yes | MergeSort + InsertionSort |
| C++ std::sort | Introsort | No | QuickSort + HeapSort + InsertionSort |
| JavaScript | TimSort | Yes | MergeSort + InsertionSort |
| Go | pdqsort | No | QuickSort + HeapSort + InsertionSort |
| Rust .sort() | TimSort | Yes | MergeSort + InsertionSort |

---

## Quick Revision Questions

1. **Why does Java use different sorting algorithms for primitives vs objects?**
2. **What is Introsort and what three algorithms does it combine?**
3. **What makes TimSort adaptive? How does it achieve O(n) on sorted data?**
4. **What is galloping mode in TimSort?**
5. **Why did JavaScript switch from QuickSort to TimSort?**
6. **Under what circumstances should you write a custom sort instead of using the built-in?**

---

[← Previous: Choosing the Right Algorithm](05-choosing-the-right-algorithm.md) | [Next: Unit 12 — Special Sorting Problems →](../12-Special-Sorting-Problems/01-dutch-national-flag.md)
