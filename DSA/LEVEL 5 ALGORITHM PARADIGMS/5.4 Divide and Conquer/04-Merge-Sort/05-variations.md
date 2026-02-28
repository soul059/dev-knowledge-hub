# Chapter 5: Merge Sort Variations

## Overview

The standard merge sort can be adapted in several ways to improve performance, reduce space, or solve specialized problems. This chapter covers **Bottom-Up Merge Sort**, **Natural Merge Sort**, **K-Way Merge**, **In-Place Merge Sort**, and **Linked List Merge Sort**.

---

## 1. Bottom-Up (Iterative) Merge Sort

Instead of recursion (top-down), we start with small subarrays and merge upward.

```
Array: [38, 27, 43, 3, 9, 82, 10]

Pass 1 (width=1): merge pairs of 1
  [38,27] → [27,38]   [43,3] → [3,43]   [9,82] → [9,82]   [10]
  
Pass 2 (width=2): merge pairs of 2
  [27,38,3,43] → [3,27,38,43]   [9,82,10] → [9,10,82]
  
Pass 3 (width=4): merge pairs of 4
  [3,27,38,43,9,10,82] → [3,9,10,27,38,43,82]

Done! ✓
```

### Pseudocode

```
function bottomUpMergeSort(arr, n):
    width = 1
    while width < n:
        for i = 0; i < n; i += 2 * width:
            lo = i
            mid = min(i + width - 1, n - 1)
            hi = min(i + 2 * width - 1, n - 1)
            if mid < hi:
                merge(arr, lo, mid, hi)
        width *= 2
```

### Advantages

```
┌──────────────────────────────────────────────────────┐
│  ✓ No recursion → no stack overflow risk             │
│  ✓ O(1) stack space (only O(n) for merge buffer)     │
│  ✓ Cache-friendly: processes elements sequentially   │
│  ✓ Easier to parallelize                             │
│                                                      │
│  ✗ Cannot exploit partially sorted input as easily   │
│  ✗ Always does exactly ⌈log₂(n)⌉ passes             │
└──────────────────────────────────────────────────────┘
```

---

## 2. Natural Merge Sort

Exploits existing sorted **runs** in the input.

```
Array: [1, 3, 5, 2, 4, 8, 6, 7, 9]

Find natural runs (ascending sequences):
  Run 1: [1, 3, 5]
  Run 2: [2, 4, 8]
  Run 3: [6, 7, 9]

Merge runs pairwise:
  [1,3,5] + [2,4,8] → [1,2,3,4,5,8]
  [6,7,9] stays

Final merge:
  [1,2,3,4,5,8] + [6,7,9] → [1,2,3,4,5,6,7,8,9]
```

### Pseudocode

```
function naturalMergeSort(arr, n):
    repeat:
        runs = findRuns(arr, n)     // find all natural runs
        if len(runs) == 1: break    // sorted!
        
        // Merge adjacent runs
        for i = 0; i < len(runs) - 1; i += 2:
            merge(arr, runs[i].start, runs[i].end, runs[i+1].end)
```

```
Best case:  Already sorted → 1 run → O(n) ✓
Worst case: Reversed → n runs of size 1 → O(n log n)
Average:    Adaptive to existing order
```

> **Note**: Python's `sorted()` and `list.sort()` use **Timsort**, which is essentially an optimized natural merge sort.

---

## 3. K-Way Merge

Merge K sorted arrays simultaneously using a min-heap.

```
K = 3 sorted arrays:
  A: [1, 5, 9]
  B: [2, 6, 10]
  C: [3, 7, 11]

Min-Heap approach:
  heap = [(1,A), (2,B), (3,C)]
  
  Extract min → 1 (from A), push A's next → 5
  heap = [(2,B), (3,C), (5,A)]
  
  Extract min → 2 (from B), push B's next → 6
  heap = [(3,C), (5,A), (6,B)]
  ...continue...
  
Result: [1, 2, 3, 5, 6, 7, 9, 10, 11]

Time: O(N log K)  where N = total elements
Space: O(K) for the heap
```

### K-Way Merge Sort

```
Instead of splitting into 2 parts, split into K parts:

T(n) = K × T(n/K) + O(n log K)

Still O(n log n) overall, but fewer merge passes:
  2-way: log₂(n) passes
  K-way: log_K(n) passes

Useful for external sorting where K = number of disk blocks.
```

---

## 4. In-Place Merge Sort (Concept)

Standard merge sort uses O(n) extra space. Can we do it in-place?

```
┌────────────────────────────────────────────────────────────┐
│  In-place merge is POSSIBLE but complex and slow:          │
│                                                            │
│  Method 1: Block merge (Kronrod's algorithm)               │
│    → O(n log n) time, O(1) space, but NOT stable           │
│                                                            │
│  Method 2: Rotate-based merge                              │
│    → O(n log²n) time, O(1) space, stable                   │
│    → Merge by finding where mid element goes, then rotate  │
│                                                            │
│  In practice: The O(n) space version is faster             │
│  → The constant factors of in-place merge are too large    │
└────────────────────────────────────────────────────────────┘
```

### Simple In-Place Merge (O(n²) per merge)

```
function inPlaceMerge(arr, lo, mid, hi):
    i = lo, j = mid + 1
    while i <= mid AND j <= hi:
        if arr[i] <= arr[j]:
            i++
        else:
            // Shift arr[i..mid] right by 1
            temp = arr[j]
            for k = j downto i+1:
                arr[k] = arr[k-1]
            arr[i] = temp
            i++; mid++; j++

// Total: O(n²) per merge → O(n² log n) overall — WORSE!
```

---

## 5. Merge Sort on Linked Lists

Merge sort is **ideal** for linked lists:

```
┌────────────────────────────────────────────────────────────┐
│  Why linked lists + merge sort = perfect match:            │
│                                                            │
│  ✓ Merge is O(1) space (just re-link pointers)             │
│  ✓ Split at midpoint using slow/fast pointers              │
│  ✓ No random access needed (unlike quick sort)             │
│  ✓ Stable sort preserved naturally                         │
│                                                            │
│  Result: O(n log n) time, O(log n) stack space             │
│          (O(1) auxiliary space — no temp array!)            │
└────────────────────────────────────────────────────────────┘
```

### Pseudocode

```
function mergeSortList(head):
    if head == null OR head.next == null:
        return head
    
    // Find middle using slow/fast pointers
    slow = head, fast = head.next
    while fast != null AND fast.next != null:
        slow = slow.next
        fast = fast.next.next
    
    mid = slow.next
    slow.next = null          // split the list
    
    left = mergeSortList(head)
    right = mergeSortList(mid)
    
    return mergeTwoLists(left, right)

function mergeTwoLists(l1, l2):
    dummy = new Node(0)
    tail = dummy
    
    while l1 != null AND l2 != null:
        if l1.val <= l2.val:
            tail.next = l1
            l1 = l1.next
        else:
            tail.next = l2
            l2 = l2.next
        tail = tail.next
    
    tail.next = (l1 != null) ? l1 : l2
    return dummy.next
```

---

## Comparison of Variations

| Variation | Time | Space | Stable | Best Use |
|-----------|------|-------|--------|----------|
| Top-Down (Standard) | O(n log n) | O(n) | Yes | General purpose |
| Bottom-Up | O(n log n) | O(n) | Yes | Avoid recursion |
| Natural | O(n) to O(n log n) | O(n) | Yes | Nearly sorted data |
| K-Way | O(N log K) | O(K) | Yes | External sorting |
| In-Place (rotate) | O(n log²n) | O(1) | Yes | Memory-constrained |
| Linked List | O(n log n) | O(log n) | Yes | Linked list data |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Bottom-Up | Iterative; merge width 1→2→4→...→n |
| Natural | Finds existing sorted runs; adaptive |
| K-Way | Uses min-heap; essential for external sort |
| In-Place | Possible but impractical; high constant |
| Linked List | Perfect fit; O(1) merge space |
| Timsort | Hybrid of natural merge sort + insertion sort |

---

## Quick Revision Questions

1. **How does Bottom-Up merge sort avoid recursion?**
2. **What makes Natural Merge Sort adaptive to nearly-sorted input?**
3. **What data structure is used in K-Way merge and what's its complexity?**
4. **Why is merge sort the preferred sort for linked lists?**
5. **Why is in-place merge sort rarely used in practice?**
6. **What is Timsort and how does it relate to merge sort?**

---

[← Previous: Counting Inversions](04-counting-inversions.md) | [Next: External Sorting →](06-external-sorting.md)
