# Chapter 6: Binary Insertion Sort

[← Previous: When Insertion Sort Shines](05-when-insertion-sort-shines.md) | [Next: Unit 5 — Merge Sort →](../05-Merge-Sort/01-algorithm-description.md)

---

## Overview

Binary Insertion Sort is an optimization of standard Insertion Sort that uses **binary search** to find the correct insertion position, reducing the number of comparisons from O(n) to O(log n) per element. However, the number of shifts remains O(n), so the overall time complexity stays O(n²).

---

## The Key Insight

```
  Standard Insertion Sort:                Binary Insertion Sort:
  
  Find position: LINEAR SCAN             Find position: BINARY SEARCH
  
  sorted part: [2, 5, 8, 12, 15, 20]    sorted part: [2, 5, 8, 12, 15, 20]
  key = 10                               key = 10
  
  Compare 20 > 10? Yes                   Binary search in [2,5,8,12,15,20]:
  Compare 15 > 10? Yes                     mid=8, 10>8 → search right
  Compare 12 > 10? Yes                     mid=15, 10<15 → search left
  Compare  8 > 10? No → found spot!        mid=12, 10<12 → search left
                                           → insert at index 3
  4 comparisons (linear)                 3 comparisons (logarithmic)
  
  ┌──────────────────────────────────────────────────────────┐
  │  In BOTH cases, we still shift 3 elements to the right  │
  │  The shift cost is the same: O(n) per insertion         │
  └──────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Trace

```
  Array: [37, 23, 0, 17, 12, 72, 31]
  
  Pass 1 (key=23):
    Sorted: [37]
    Binary search for 23 in [37]: pos=0
    Shift 37 right, insert 23
    → [23, 37, 0, 17, 12, 72, 31]
  
  Pass 2 (key=0):
    Sorted: [23, 37]
    Binary search for 0 in [23,37]:
      mid=23, 0<23 → left
      → pos=0
    Shift 23,37 right, insert 0
    → [0, 23, 37, 17, 12, 72, 31]
  
  Pass 3 (key=17):
    Sorted: [0, 23, 37]
    Binary search for 17 in [0,23,37]:
      mid=23, 17<23 → left
      mid=0, 17>0 → right
      → pos=1
    Shift 23,37 right, insert 17
    → [0, 17, 23, 37, 12, 72, 31]
  
  Pass 4 (key=12):
    Sorted: [0, 17, 23, 37]
    Binary search for 12 in [0,17,23,37]:
      mid=17, 12<17 → left
      mid=0, 12>0 → right
      → pos=1
    Shift 17,23,37 right, insert 12
    → [0, 12, 17, 23, 37, 72, 31]
  
  Pass 5 (key=72):
    Sorted: [0, 12, 17, 23, 37]
    Binary search for 72 in [0,12,17,23,37]:
      mid=17, 72>17 → right
      mid=37, 72>37 → right
      → pos=5 (end)
    No shifts needed!
    → [0, 12, 17, 23, 37, 72, 31]
  
  Pass 6 (key=31):
    Sorted: [0, 12, 17, 23, 37, 72]
    Binary search for 31 in [0,12,17,23,37,72]:
      mid=23, 31>23 → right
      mid=72, 31<72 → left
      mid=37, 31<37 → left
      → pos=4
    Shift 37,72 right, insert 31
    → [0, 12, 17, 23, 31, 37, 72]   ✓ SORTED!
```

---

## Implementation

### Java

```java
public static void binaryInsertionSort(int[] arr) {
    int n = arr.length;
    
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        
        // Binary search for insertion position
        int pos = binarySearch(arr, key, 0, i - 1);
        
        // Shift elements to make room
        for (int j = i - 1; j >= pos; j--) {
            arr[j + 1] = arr[j];
        }
        
        arr[pos] = key;
    }
}

// Find leftmost position where key should be inserted
private static int binarySearch(int[] arr, int key, int low, int high) {
    while (low <= high) {
        int mid = low + (high - low) / 2;
        
        if (key < arr[mid]) {
            high = mid - 1;
        } else {
            low = mid + 1;  // key >= arr[mid], go right (stability!)
        }
    }
    return low;  // insertion position
}
```

### Python

```python
import bisect

def binary_insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        
        # Find insertion position using binary search
        pos = bisect.bisect_left(arr, key, 0, i)
        
        # Shift and insert (Python slice trick)
        # Equivalent to: shift elements arr[pos:i] right by 1, then arr[pos] = key
        arr[pos+1:i+1] = arr[pos:i]
        arr[pos] = key

# Manual binary search version:
def binary_insertion_sort_manual(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        low, high = 0, i - 1
        
        while low <= high:
            mid = (low + high) // 2
            if key < arr[mid]:
                high = mid - 1
            else:
                low = mid + 1
        
        # Shift and insert
        for j in range(i - 1, low - 1, -1):
            arr[j + 1] = arr[j]
        arr[low] = key
```

### C++

```cpp
void binaryInsertionSort(vector<int>& arr) {
    int n = arr.size();
    
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        
        // Use std::upper_bound for stable binary search
        // Or implement manually:
        int low = 0, high = i - 1;
        while (low <= high) {
            int mid = low + (high - low) / 2;
            if (key < arr[mid])
                high = mid - 1;
            else
                low = mid + 1;
        }
        
        // Shift elements
        for (int j = i - 1; j >= low; j--) {
            arr[j + 1] = arr[j];
        }
        arr[low] = key;
    }
}
```

---

## Stability of Binary Insertion Sort

```
  ┌──────────────────────────────────────────────────────────────┐
  │  KEY DETAIL: The binary search must find the RIGHTMOST      │
  │  position among equal elements to maintain stability.       │
  │                                                              │
  │  Array: [3a, 5, 7, 3b_key]                                 │
  │                                                              │
  │  CORRECT (stable):     Search returns pos=1 (after 3a)     │
  │    → [3a, 3b, 5, 7]   Order preserved ✓                   │
  │                                                              │
  │  WRONG (unstable):     Search returns pos=0 (before 3a)    │
  │    → [3b, 3a, 5, 7]   Order violated ✗                    │
  │                                                              │
  │  Solution: When key == arr[mid], go RIGHT (low = mid + 1)  │
  │  This pushes equal elements to the left of key's position  │
  └──────────────────────────────────────────────────────────────┘
```

---

## Complexity Analysis

```
  ┌────────────────────────────────────────────────────────────┐
  │                    COMPARISONS                             │
  │                                                            │
  │  For each element i, binary search in sorted part of       │
  │  size i: O(log i) comparisons                             │
  │                                                            │
  │  Total comparisons = Σ(i=1 to n-1) log₂(i)               │
  │                    = log₂(1) + log₂(2) + ... + log₂(n-1) │
  │                    = log₂((n-1)!) ≈ n·log₂(n) - n        │
  │                    = O(n log n)                            │
  │                                                            │
  │  vs. Standard Insertion Sort: O(n²) comparisons (worst)   │
  │  Improvement: n² → n log n for comparisons!               │
  ├────────────────────────────────────────────────────────────┤
  │                    SHIFTS (MOVES)                          │
  │                                                            │
  │  Binary search doesn't reduce shifts!                     │
  │  Worst case: Σ(i=1 to n-1) i = n(n-1)/2 = O(n²)        │
  │                                                            │
  │  Total time = O(n log n) comparisons + O(n²) shifts      │
  │             = O(n²)  (shifts dominate)                    │
  ├────────────────────────────────────────────────────────────┤
  │  OVERALL:                                                  │
  │  Time:  O(n²) worst, O(n²) average, O(n log n) best      │
  │  Space: O(1) — in-place                                   │
  │  Stable: Yes (with correct binary search)                  │
  └────────────────────────────────────────────────────────────┘
```

### Standard vs Binary Insertion Sort

```
  Metric           │ Standard     │ Binary
  ─────────────────┼──────────────┼──────────────
  Comparisons      │ O(n²) worst  │ O(n log n) ✓
  (finding pos)    │ O(n) best    │ O(n log n) ✗
  ─────────────────┼──────────────┼──────────────
  Shifts/Moves     │ O(n²) worst  │ O(n²) worst
                   │ O(0) best    │ O(0) best
  ─────────────────┼──────────────┼──────────────
  Total Time       │ O(n²)        │ O(n²)
  ─────────────────┼──────────────┼──────────────
  Best Case Total  │ O(n) ✓       │ O(n log n) ✗
  ─────────────────┼──────────────┼──────────────
  Space            │ O(1)         │ O(1)
  ─────────────────┼──────────────┼──────────────
  Stable           │ Yes          │ Yes
  
  SURPRISE: Standard Insertion Sort has a BETTER best case!
  Binary search adds O(log n) comparisons even when data is sorted.
```

---

## When to Use Binary Insertion Sort

```
  ✓ USE when comparisons are EXPENSIVE
    - Sorting strings (comparison = strcmp, O(length))
    - Sorting objects with complex comparators
    - Sorting records with composite keys
    
    Example: Sorting 1000 strings of length 100
    Standard: ~250,000 comparisons × 100 chars = 25M char compares
    Binary:   ~10,000 comparisons × 100 chars = 1M char compares
                                                  25x fewer!
  
  ✗ AVOID when comparisons are CHEAP
    - Sorting integers (comparison = one CPU instruction)
    - Nearly sorted data (standard is O(n) best!)
    - Very small arrays (overhead not worth it)
```

---

## Pseudocode

```
  BINARY-INSERTION-SORT(A, n)
  ─────────────────────────────
  for i ← 1 to n-1
      key ← A[i]
      
      // Binary search for position
      low ← 0
      high ← i - 1
      while low ≤ high
          mid ← ⌊(low + high) / 2⌋
          if key < A[mid]
              high ← mid - 1
          else
              low ← mid + 1
      
      // Shift elements right
      for j ← i-1 downto low
          A[j+1] ← A[j]
      
      A[low] ← key
```

---

## Summary Table

| Aspect | Standard Insertion | Binary Insertion |
|--------|-------------------|-----------------|
| Comparisons (worst) | O(n²) | O(n log n) |
| Shifts (worst) | O(n²) | O(n²) |
| Total time (worst) | O(n²) | O(n²) |
| Best case | O(n) | O(n log n) |
| Space | O(1) | O(1) |
| Stability | Yes | Yes |
| Best for | Cheap comparisons, nearly sorted | Expensive comparisons |

---

## Quick Revision Questions

1. **What does Binary Insertion Sort optimize — comparisons or shifts?**
2. **Why is the overall complexity still O(n²)?**
3. **Which version has a BETTER best case — standard or binary? Why?**
4. **How do you ensure stability in Binary Insertion Sort?**
5. **When is Binary Insertion Sort preferable to the standard version?**
6. **What is the total comparison count: O(n log n). Derive it.**

---

[← Previous: When Insertion Sort Shines](05-when-insertion-sort-shines.md) | [Next: Unit 5 — Merge Sort →](../05-Merge-Sort/01-algorithm-description.md)
