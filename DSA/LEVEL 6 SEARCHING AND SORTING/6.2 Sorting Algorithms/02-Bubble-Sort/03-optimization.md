# Chapter 3: Bubble Sort — Optimization (Early Termination)

[← Previous: Implementation](02-implementation.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)

---

## Overview

The basic Bubble Sort always runs all n-1 passes, even if the array becomes sorted early. The **optimized version** detects when no swaps occur in a pass and terminates early, making it **adaptive** to nearly-sorted input.

---

## The Problem with Basic Bubble Sort

```
  Array: [1, 2, 3, 5, 4]    ← Already nearly sorted!
  
  Pass 1: Compare (1,2)✓ (2,3)✓ (3,5)✓ (5,4)swap → [1,2,3,4,5]
  Pass 2: Compare (1,2)✓ (2,3)✓ (3,4)✓            → No swaps!
  Pass 3: Compare (1,2)✓ (2,3)✓                     → No swaps!
  Pass 4: Compare (1,2)✓                             → No swaps!
  
  Passes 2-4 are WASTED — array was already sorted after Pass 1!
  
  ┌──────────────────────────────────────────────┐
  │  INSIGHT: If a pass makes ZERO swaps,        │
  │  the array is already sorted.                │
  │  We can STOP early!                          │
  └──────────────────────────────────────────────┘
```

---

## Optimized Implementation

### Pseudocode

```
OPTIMIZED-BUBBLE-SORT(A, n):
    for i = 0 to n-2:
        swapped = false                   // ← flag for this pass
        
        for j = 0 to n-2-i:
            if A[j] > A[j+1]:
                swap(A[j], A[j+1])
                swapped = true            // ← a swap happened
        
        if swapped == false:              // ← no swaps in this pass
            break                         // ← array is sorted, stop!
```

### Java

```java
public static void bubbleSortOptimized(int[] arr) {
    int n = arr.length;
    
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;         // track swaps
        
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }
        
        if (!swapped) break;             // early termination
    }
}
```

### Python

```python
def bubble_sort_optimized(arr):
    n = len(arr)
    
    for i in range(n - 1):
        swapped = False
        
        for j in range(n - 1 - i):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        
        if not swapped:
            break                        # array is sorted
```

---

## Trace: Optimized vs Basic

```
  Array: [2, 1, 3, 4, 5]
  
  ═══ BASIC BUBBLE SORT ═══
  Pass 0: (2,1)swap (2,3)no (3,4)no (4,5)no  → [1,2,3,4,5]   4 comparisons
  Pass 1: (1,2)no   (2,3)no (3,4)no          → [1,2,3,4,5]   3 comparisons
  Pass 2: (1,2)no   (2,3)no                  → [1,2,3,4,5]   2 comparisons
  Pass 3: (1,2)no                             → [1,2,3,4,5]   1 comparison
  Total: 10 comparisons
  
  ═══ OPTIMIZED BUBBLE SORT ═══
  Pass 0: (2,1)swap (2,3)no (3,4)no (4,5)no  → [1,2,3,4,5]   4 comparisons
           swapped = true
  Pass 1: (1,2)no   (2,3)no (3,4)no          → [1,2,3,4,5]   3 comparisons
           swapped = false → BREAK!
  Total: 7 comparisons  (saved 3 comparisons = 30% fewer)
```

---

## Best Case: Already Sorted

```
  Array: [1, 2, 3, 4, 5]    ← Already sorted!
  
  ═══ BASIC ═══
  Pass 0: 4 comparisons, 0 swaps
  Pass 1: 3 comparisons, 0 swaps
  Pass 2: 2 comparisons, 0 swaps
  Pass 3: 1 comparison,  0 swaps
  Total: 10 comparisons                   → O(n²) always!
  
  ═══ OPTIMIZED ═══
  Pass 0: 4 comparisons, 0 swaps
           swapped = false → BREAK!
  Total: 4 comparisons = n-1              → O(n) best case!
  
  ┌──────────────────────────────────────────────┐
  │  Optimization changes best case from         │
  │  O(n²) to O(n) — a dramatic improvement     │
  │  for sorted or nearly-sorted data!           │
  └──────────────────────────────────────────────┘
```

---

## Impact Analysis

```
  Comparisons saved for different inputs (n=5):
  
  Input Type          │ Basic │ Optimized │ Savings
  ────────────────────┼───────┼───────────┼────────
  Already sorted      │  10   │     4     │  60%
  Nearly sorted (1sw) │  10   │     7     │  30%
  Random              │  10   │    ~9     │  ~10%
  Reverse sorted      │  10   │    10     │   0%
  
  ┌──────────────────────────────────────────────┐
  │  The optimization helps MOST when the input  │
  │  is already sorted or nearly sorted.         │
  │  It does NOT help for worst case (reverse).  │
  └──────────────────────────────────────────────┘
```

---

## Further Optimization: Track Last Swap Position

An even more aggressive optimization tracks **where the last swap occurred**:

```
  IDEA: If last swap was at position k, then all elements 
  after k are already sorted. Next pass only needs to go up to k.
  
  Array: [1, 5, 3, 4, 2, 6, 7, 8]
                            ↑
                        last swap at j=3 (swapped 4,2)
  
  Elements after index 3 are sorted → next pass goes 0 to 3 only!
```

### Implementation

```python
def bubble_sort_advanced(arr):
    n = len(arr)
    end = n - 1  # unsorted boundary
    
    while end > 0:
        last_swap = 0  # track last swap position
        
        for j in range(end):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                last_swap = j  # record where swap happened
        
        end = last_swap  # next pass only goes up to here
        # If last_swap == 0, no swaps occurred → exit loop
```

### Trace

```
  Array: [1, 5, 3, 4, 2, 6, 7, 8]    end = 7
  
  Pass 1: 
    j=0: (1,5) no
    j=1: (5,3) swap → last_swap=1   [1,3,5,4,2,6,7,8]
    j=2: (5,4) swap → last_swap=2   [1,3,4,5,2,6,7,8]
    j=3: (5,2) swap → last_swap=3   [1,3,4,2,5,6,7,8]
    j=4: (5,6) no
    j=5: (6,7) no
    j=6: (7,8) no
    end = 3  (next pass: j goes 0 to 2 only!)
  
  Pass 2:
    j=0: (1,3) no
    j=1: (3,4) no
    j=2: (4,2) swap → last_swap=2   [1,3,2,4,5,6,7,8]
    end = 2
  
  Pass 3:
    j=0: (1,3) no
    j=1: (3,2) swap → last_swap=1   [1,2,3,4,5,6,7,8]
    end = 1
  
  Pass 4:
    j=0: (1,2) no
    last_swap = 0 → end = 0 → DONE!
  
  Only 4+3+2+1 = 10 comparisons instead of 28 for basic!
```

---

## Comparison of All Variants

| Variant | Best Case | Average | Worst Case | Key Feature |
|---------|-----------|---------|------------|-------------|
| Basic | O(n²) | O(n²) | O(n²) | No optimization |
| Early termination | **O(n)** | O(n²) | O(n²) | Stop when no swaps |
| Last swap tracking | **O(n)** | O(n²) | O(n²) | Reduces inner loop range |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Basic problem | Runs all passes even if sorted early |
| `swapped` flag | Detects if pass made zero swaps |
| Early termination | Break if no swaps → array is sorted |
| Best case improvement | O(n²) → O(n) |
| Last swap tracking | Reduces inner loop range further |
| Worst case | Still O(n²) — optimization doesn't help |

---

## Quick Revision Questions

1. **What condition allows early termination in Bubble Sort?**
2. **What is the best-case complexity of optimized Bubble Sort?**
3. **Does the optimization help for reverse-sorted input? Why?**
4. **How does tracking the last swap position improve performance?**
5. **Write the optimized Bubble Sort with the `swapped` flag.**
6. **For [1, 2, 3, 4, 5], how many comparisons does each variant make?**

---

[← Previous: Implementation](02-implementation.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)
