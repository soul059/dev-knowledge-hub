# Unit 5: Merge Sort — Iterative (Bottom-Up) Merge Sort

[← Previous: Space Optimization](05-space-optimization.md) | [Next: Unit 6 — Quick Sort →](../06-Quick-Sort/01-algorithm-description.md)

---

## Overview

Standard Merge Sort is top-down (recursive). **Bottom-Up Merge Sort** achieves the same result iteratively — no recursion needed. It merges small chunks first, then progressively larger ones, eliminating the O(log n) recursion stack.

---

## Top-Down vs Bottom-Up

```
  TOP-DOWN (Recursive):
  Start with full array, split to base case, merge up.
  
  [38, 27, 43, 3, 9, 82, 10]
           ↓ SPLIT
  [38, 27, 43, 3] [9, 82, 10]
        ↓ SPLIT        ↓ SPLIT
  [38,27] [43,3] [9,82] [10]
    ↓ ↓     ↓ ↓    ↓ ↓    │
  [38][27] [43][3] [9][82] [10]
    ↓ ↓     ↓ ↓    ↓ ↓     │
  [27,38] [3,43] [9,82]  [10]    ← Merge pairs
       ↓       ↓       ↓
  [3,27,38,43] [9,10,82]         ← Merge fours
          ↓          ↓
  [3,9,10,27,38,43,82]           ← Merge all
  
  
  BOTTOM-UP (Iterative):
  Start with size-1 chunks, merge up progressively.
  
  [38] [27] [43] [3] [9] [82] [10]    ← Size 1 chunks
    ↓    ↓    ↓   ↓   ↓    ↓    ↓
  [27,38] [3,43] [9,82] [10]          ← Merge size-1 pairs
       ↓       ↓       ↓
  [3,27,38,43] [9,10,82]              ← Merge size-2 pairs
          ↓          ↓
  [3,9,10,27,38,43,82]                ← Merge size-4 pairs
  
  SAME RESULT, NO RECURSION!
```

---

## How Bottom-Up Works

```
  Size = 1: Merge pairs of single elements
  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
  │ 5 │ │ 2 │ │ 8 │ │ 1 │ │ 4 │ │ 7 │ │ 3 │ │ 6 │
  └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘
    └──┬──┘     └──┬──┘     └──┬──┘     └──┬──┘
  
  Size = 2: After merging pairs
  ┌───┬───┐ ┌───┬───┐ ┌───┬───┐ ┌───┬───┐
  │ 2 │ 5 │ │ 1 │ 8 │ │ 4 │ 7 │ │ 3 │ 6 │
  └───┴───┘ └───┴───┘ └───┴───┘ └───┴───┘
    └────┬────┘           └────┬────┘
  
  Size = 4: After merging quads
  ┌───┬───┬───┬───┐ ┌───┬───┬───┬───┐
  │ 1 │ 2 │ 5 │ 8 │ │ 3 │ 4 │ 6 │ 7 │
  └───┴───┴───┴───┘ └───┴───┴───┴───┘
        └──────────┬──────────┘
  
  Size = 8: Final merge
  ┌───┬───┬───┬───┬───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │  ✓
  └───┴───┴───┴───┴───┴───┴───┴───┘
  
  Sizes: 1 → 2 → 4 → 8 → 16 → ... (doubles each pass)
  Number of passes: log₂(n)
```

---

## Implementation

### Java

```java
public static void mergeSortBottomUp(int[] arr) {
    int n = arr.length;
    int[] aux = new int[n];
    
    // Size of subarrays to merge: 1, 2, 4, 8, ...
    for (int size = 1; size < n; size *= 2) {
        
        // Merge adjacent pairs of subarrays
        for (int lo = 0; lo < n - size; lo += 2 * size) {
            int mid = lo + size - 1;
            int hi = Math.min(lo + 2 * size - 1, n - 1);
            merge(arr, aux, lo, mid, hi);
        }
    }
}

private static void merge(int[] arr, int[] aux, int lo, int mid, int hi) {
    // Copy to auxiliary
    for (int k = lo; k <= hi; k++) {
        aux[k] = arr[k];
    }
    
    int i = lo, j = mid + 1;
    for (int k = lo; k <= hi; k++) {
        if      (i > mid)          arr[k] = aux[j++];
        else if (j > hi)           arr[k] = aux[i++];
        else if (aux[i] <= aux[j]) arr[k] = aux[i++];
        else                       arr[k] = aux[j++];
    }
}
```

### Python

```python
def merge_sort_bottom_up(arr):
    n = len(arr)
    aux = [0] * n
    
    size = 1
    while size < n:
        lo = 0
        while lo < n - size:
            mid = lo + size - 1
            hi = min(lo + 2 * size - 1, n - 1)
            merge(arr, aux, lo, mid, hi)
            lo += 2 * size
        size *= 2

def merge(arr, aux, lo, mid, hi):
    aux[lo:hi+1] = arr[lo:hi+1]
    
    i, j = lo, mid + 1
    for k in range(lo, hi + 1):
        if   i > mid:          arr[k] = aux[j]; j += 1
        elif j > hi:           arr[k] = aux[i]; i += 1
        elif aux[i] <= aux[j]: arr[k] = aux[i]; i += 1
        else:                  arr[k] = aux[j]; j += 1
```

### C++

```cpp
void mergeSortBottomUp(vector<int>& arr) {
    int n = arr.size();
    vector<int> aux(n);
    
    for (int size = 1; size < n; size *= 2) {
        for (int lo = 0; lo < n - size; lo += 2 * size) {
            int mid = lo + size - 1;
            int hi = min(lo + 2 * size - 1, n - 1);
            merge(arr, aux, lo, mid, hi);
        }
    }
}
```

---

## Detailed Trace

```
  Array: [5, 2, 8, 1, 4, 7, 3]    n = 7

  === Pass 1 (size = 1) ===
  
  lo=0: merge [5] and [2]   → [2, 5, ...]
  lo=2: merge [8] and [1]   → [..., 1, 8, ...]
  lo=4: merge [4] and [7]   → [..., 4, 7, ...]
  lo=6: 6 < 7-1=6? NO → skip (3 is alone, no pair)
  
  Array: [2, 5, 1, 8, 4, 7, 3]
  
  === Pass 2 (size = 2) ===
  
  lo=0: merge [2,5] and [1,8]    → [1, 2, 5, 8, ...]
  lo=4: merge [4,7] and [3]      → [..., 3, 4, 7]
  
  Array: [1, 2, 5, 8, 3, 4, 7]
  
  === Pass 3 (size = 4) ===
  
  lo=0: merge [1,2,5,8] and [3,4,7]  → [1, 2, 3, 4, 5, 7, 8]
  
  Array: [1, 2, 3, 4, 5, 7, 8]  ✓ SORTED!
  
  3 passes = ⌈log₂(7)⌉ = 3  ✓
```

---

## Handling Odd-Sized Arrays

```
  When n is not a power of 2, the last subarray may be smaller:
  
  n = 7, size = 2:
  ┌─────┐ ┌─────┐ ┌─────┐ ┌───┐
  │ 0,1 │ │ 2,3 │ │ 4,5 │ │ 6 │  ← Last chunk has 1 element
  └─────┘ └─────┘ └─────┘ └───┘
    pair     pair     pair   leftover
  
  The code handles this with:
    hi = Math.min(lo + 2*size - 1, n - 1)
  
  This caps the right boundary at n-1, so the last merge
  handles a smaller right subarray correctly.
  
  The condition lo < n - size ensures we only merge when
  there's at least something to merge with (right part exists).
```

---

## Bottom-Up for Linked Lists

```
  Bottom-up merge sort is IDEAL for linked lists because:
  
  1. No random access needed (unlike top-down with midpoint)
  2. In-place merge (just re-link pointers, O(1) space!)
  3. No auxiliary array needed
  
  ┌──────────────────────────────────────────────────────┐
  │  For linked lists:                                   │
  │  Top-down:  O(n log n) time, O(log n) stack space  │
  │  Bottom-up: O(n log n) time, O(1) space  ← BETTER │
  └──────────────────────────────────────────────────────┘
```

```java
// Bottom-up merge sort for linked list (sketch)
ListNode sortList(ListNode head) {
    if (head == null || head.next == null) return head;
    
    int length = getLength(head);
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    
    for (int size = 1; size < length; size *= 2) {
        ListNode prev = dummy;
        ListNode curr = dummy.next;
        
        while (curr != null) {
            ListNode left = curr;
            ListNode right = split(left, size);
            curr = split(right, size);
            prev = merge(prev, left, right);
        }
    }
    return dummy.next;
}
```

---

## Top-Down vs Bottom-Up Comparison

```
  ┌──────────────────┬───────────────────┬───────────────────┐
  │  Aspect          │  Top-Down         │  Bottom-Up        │
  ├──────────────────┼───────────────────┼───────────────────┤
  │  Approach        │  Recursive        │  Iterative        │
  │  Direction       │  Split → merge up │  Merge small → big│
  │  Stack space     │  O(log n)         │  O(1)             │
  │  Time            │  O(n log n)       │  O(n log n)       │
  │  Aux space       │  O(n)             │  O(n)             │
  │  Total space     │  O(n + log n)     │  O(n)             │
  │  Cache behavior  │  Good             │  Excellent        │
  │  For arrays      │  Good             │  Good             │
  │  For linked lists│  Need midpoint    │  IDEAL (O(1) aux) │
  │  Ease of code    │  Easier           │  Slightly harder  │
  │  Skip-sort opt   │  Easy (check mid) │  Not applicable   │
  │  Natural runs    │  Hard to detect   │  Can be added     │
  └──────────────────┴───────────────────┴───────────────────┘
```

---

## Summary Table

| Feature | Bottom-Up Merge Sort |
|---------|---------------------|
| Time | O(n log n) all cases |
| Space (array) | O(n) aux + O(1) stack |
| Space (linked list) | O(1) |
| Passes | log₂(n) |
| Work per pass | O(n) |
| Recursive? | No |
| Stable? | Yes |
| Best for | Linked lists, stack-limited environments |

---

## Quick Revision Questions

1. **How does bottom-up merge sort differ from top-down?**
2. **What are the merge sizes in each pass?**
3. **How many passes does bottom-up merge sort make for n=64?**
4. **Why is bottom-up preferred for linked lists?**
5. **How does the code handle arrays whose size isn't a power of 2?**
6. **What is the stack space for bottom-up vs top-down?**

---

[← Previous: Space Optimization](05-space-optimization.md) | [Next: Unit 6 — Quick Sort →](../06-Quick-Sort/01-algorithm-description.md)
