# Chapter 2: Bubble Sort — Implementation

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Optimization →](03-optimization.md)

---

## Overview

This chapter provides complete implementations of Bubble Sort in multiple languages, with detailed line-by-line explanation.

---

## Java Implementation

```java
public class BubbleSort {
    
    public static void bubbleSort(int[] arr) {
        int n = arr.length;
        
        // Outer loop: n-1 passes
        for (int i = 0; i < n - 1; i++) {
            
            // Inner loop: compare adjacent elements
            // Range shrinks because last i elements are sorted
            for (int j = 0; j < n - 1 - i; j++) {
                
                if (arr[j] > arr[j + 1]) {
                    // Swap arr[j] and arr[j+1]
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
    }
    
    public static void main(String[] args) {
        int[] arr = {64, 34, 25, 12, 22, 11, 90};
        bubbleSort(arr);
        // Output: [11, 12, 22, 25, 34, 64, 90]
    }
}
```

## Python Implementation

```python
def bubble_sort(arr):
    n = len(arr)
    
    for i in range(n - 1):                 # n-1 passes
        for j in range(n - 1 - i):         # shrinking window
            if arr[j] > arr[j + 1]:        # compare adjacent
                arr[j], arr[j + 1] = arr[j + 1], arr[j]  # swap

# Example
arr = [64, 34, 25, 12, 22, 11, 90]
bubble_sort(arr)
print(arr)  # [11, 12, 22, 25, 34, 64, 90]
```

## C++ Implementation

```cpp
#include <iostream>
#include <vector>
using namespace std;

void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
            }
        }
    }
}
```

---

## Line-by-Line Walkthrough

```
  for i = 0 to n-2:                    ← PASS number (0 to n-2)
  │
  │   After pass i, the (i+1) largest 
  │   elements are at the end
  │
  └── for j = 0 to n-2-i:              ← COMPARISON within pass
      │
      │   j is the LEFT element index
      │   j+1 is the RIGHT element index
      │
      └── if A[j] > A[j+1]:            ← Compare adjacent pair
              swap(A[j], A[j+1])        ← Swap if out of order
  
  
  EXAMPLE: n = 5, Array = [5, 3, 8, 1, 2]
  
  i=0: j goes 0→3  (4 comparisons)  → largest bubbles to index 4
  i=1: j goes 0→2  (3 comparisons)  → 2nd largest to index 3
  i=2: j goes 0→1  (2 comparisons)  → 3rd largest to index 2
  i=3: j goes 0→0  (1 comparison)   → 4th largest to index 1
                                       5th (smallest) at index 0
  
  Total: 4 + 3 + 2 + 1 = 10 comparisons
```

---

## Swap Mechanism Detailed

```
  Swapping A[j] and A[j+1] using a temp variable:
  
  Before:  A[j] = 5    A[j+1] = 3
  
  Step 1:  temp = A[j]           // temp = 5
  Step 2:  A[j] = A[j+1]        // A[j] = 3
  Step 3:  A[j+1] = temp        // A[j+1] = 5
  
  After:   A[j] = 3    A[j+1] = 5
  
  ┌─────────┐     ┌─────────┐     ┌─────────┐
  │  temp=?  │     │  temp=5  │     │  temp=5  │
  ├─────────┤     ├─────────┤     ├─────────┤
  │ j │ j+1 │     │ j │ j+1 │     │ j │ j+1 │
  │ 5 │  3  │ ──► │ 3 │  3  │ ──► │ 3 │  5  │
  └───┴─────┘     └───┴─────┘     └───┴─────┘
    Step 1           Step 2           Step 3
```

---

## Complete Execution Trace

```
  Input: [64, 34, 25, 12, 22, 11, 90]     n = 7
  
  ═══ Pass i=0 (j: 0→5) ═══
  j=0: [64,34] → swap → [34,64,25,12,22,11,90]
  j=1: [64,25] → swap → [34,25,64,12,22,11,90]
  j=2: [64,12] → swap → [34,25,12,64,22,11,90]
  j=3: [64,22] → swap → [34,25,12,22,64,11,90]
  j=4: [64,11] → swap → [34,25,12,22,11,64,90]
  j=5: [64,90] → no    → [34,25,12,22,11,64,90]   ← 90 ✓
  
  ═══ Pass i=1 (j: 0→4) ═══
  j=0: [34,25] → swap → [25,34,12,22,11,64,90]
  j=1: [34,12] → swap → [25,12,34,22,11,64,90]
  j=2: [34,22] → swap → [25,12,22,34,11,64,90]
  j=3: [34,11] → swap → [25,12,22,11,34,64,90]
  j=4: [34,64] → no    → [25,12,22,11,34,64,90]   ← 64 ✓
  
  ═══ Pass i=2 (j: 0→3) ═══
  j=0: [25,12] → swap → [12,25,22,11,34,64,90]
  j=1: [25,22] → swap → [12,22,25,11,34,64,90]
  j=2: [25,11] → swap → [12,22,11,25,34,64,90]
  j=3: [25,34] → no    → [12,22,11,25,34,64,90]   ← 34 ✓
  
  ═══ Pass i=3 (j: 0→2) ═══
  j=0: [12,22] → no    → [12,22,11,25,34,64,90]
  j=1: [22,11] → swap → [12,11,22,25,34,64,90]
  j=2: [22,25] → no    → [12,11,22,25,34,64,90]   ← 25 ✓
  
  ═══ Pass i=4 (j: 0→1) ═══
  j=0: [12,11] → swap → [11,12,22,25,34,64,90]
  j=1: [12,22] → no    → [11,12,22,25,34,64,90]   ← 22 ✓
  
  ═══ Pass i=5 (j: 0→0) ═══
  j=0: [11,12] → no    → [11,12,22,25,34,64,90]   ← 12 ✓
  
  Result: [11, 12, 22, 25, 34, 64, 90]  ← 11 ✓ (only element left)
```

---

## Descending Order Variant

To sort in **descending order**, simply change the comparison:

```java
// Ascending: swap when left > right
if (arr[j] > arr[j + 1])      // smallest bubbles DOWN

// Descending: swap when left < right  
if (arr[j] < arr[j + 1])      // largest bubbles DOWN
```

---

## Generic Implementation (Java)

```java
public static <T extends Comparable<T>> void bubbleSort(T[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j].compareTo(arr[j + 1]) > 0) {
                T temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
// Works with String[], Integer[], or any Comparable type
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Outer loop | i from 0 to n-2 (controls passes) |
| Inner loop | j from 0 to n-2-i (comparisons per pass) |
| Swap condition | A[j] > A[j+1] (ascending) |
| Swap method | Three-step using temp variable |
| Total comparisons | n(n-1)/2 |
| Total swaps (worst) | n(n-1)/2 (reverse sorted input) |

---

## Quick Revision Questions

1. **Write Bubble Sort pseudocode from memory.**
2. **Why does the inner loop go from 0 to n-2-i?**
3. **How many total comparisons for an array of size 10?**
4. **How would you modify Bubble Sort for descending order?**
5. **Trace Bubble Sort on [4, 1, 3, 2] showing each swap.**
6. **What is the maximum number of swaps for n elements?**

---

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Optimization →](03-optimization.md)
