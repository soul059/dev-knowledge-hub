# Chapter 2: Selection Sort — Implementation

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Complexity Analysis →](03-complexity-analysis.md)

---

## Overview

Complete implementations of Selection Sort with detailed walkthroughs and execution traces.

---

## Java Implementation

```java
public class SelectionSort {
    
    public static void selectionSort(int[] arr) {
        int n = arr.length;
        
        for (int i = 0; i < n - 1; i++) {
            // Find minimum element in unsorted portion
            int minIndex = i;
            
            for (int j = i + 1; j < n; j++) {
                if (arr[j] < arr[minIndex]) {
                    minIndex = j;
                }
            }
            
            // Swap minimum with first unsorted element
            if (minIndex != i) {
                int temp = arr[i];
                arr[i] = arr[minIndex];
                arr[minIndex] = temp;
            }
        }
    }
    
    public static void main(String[] args) {
        int[] arr = {64, 25, 12, 22, 11};
        selectionSort(arr);
        // Output: [11, 12, 22, 25, 64]
    }
}
```

## Python Implementation

```python
def selection_sort(arr):
    n = len(arr)
    
    for i in range(n - 1):
        min_idx = i                         # assume i is minimum
        
        for j in range(i + 1, n):           # search unsorted portion
            if arr[j] < arr[min_idx]:
                min_idx = j
        
        if min_idx != i:                    # swap if needed
            arr[i], arr[min_idx] = arr[min_idx], arr[i]

# Example
arr = [64, 25, 12, 22, 11]
selection_sort(arr)
print(arr)  # [11, 12, 22, 25, 64]
```

## C++ Implementation

```cpp
void selectionSort(vector<int>& arr) {
    int n = arr.size();
    
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) {
                minIdx = j;
            }
        }
        
        if (minIdx != i) {
            swap(arr[i], arr[minIdx]);
        }
    }
}
```

---

## Line-by-Line Walkthrough

```
  for i = 0 to n-2:              ← Position to fill (0, 1, 2, ..., n-2)
  │
  │   minIndex = i                ← Start by assuming arr[i] is minimum
  │
  └── for j = i+1 to n-1:        ← Scan rest of array
      │
      └── if A[j] < A[minIndex]:  ← Found something smaller?
              minIndex = j        ← Track its index
      
      After inner loop: minIndex = index of smallest in unsorted portion
      
      swap(A[i], A[minIndex])     ← Place minimum at position i
```

---

## Complete Execution Trace

```
  Input: [64, 25, 12, 22, 11]     n = 5
  
  ═══ Pass i=0 ═══  (Fill position 0)
  minIndex = 0 (value 64)
  j=1: arr[1]=25 < 64? YES → minIndex=1
  j=2: arr[2]=12 < 25? YES → minIndex=2
  j=3: arr[3]=22 < 12? NO
  j=4: arr[4]=11 < 12? YES → minIndex=4
  Swap arr[0] and arr[4]:
  [64, 25, 12, 22, 11] → [11, 25, 12, 22, 64]
   ✓
  
  ═══ Pass i=1 ═══  (Fill position 1)
  minIndex = 1 (value 25)
  j=2: arr[2]=12 < 25? YES → minIndex=2
  j=3: arr[3]=22 < 12? NO
  j=4: arr[4]=64 < 12? NO
  Swap arr[1] and arr[2]:
  [11, 25, 12, 22, 64] → [11, 12, 25, 22, 64]
   ✓   ✓
  
  ═══ Pass i=2 ═══  (Fill position 2)
  minIndex = 2 (value 25)
  j=3: arr[3]=22 < 25? YES → minIndex=3
  j=4: arr[4]=64 < 22? NO
  Swap arr[2] and arr[3]:
  [11, 12, 25, 22, 64] → [11, 12, 22, 25, 64]
   ✓   ✓   ✓
  
  ═══ Pass i=3 ═══  (Fill position 3)
  minIndex = 3 (value 25)
  j=4: arr[4]=64 < 25? NO
  No swap needed (minIndex == 3 == i)
  [11, 12, 22, 25, 64]
   ✓   ✓   ✓   ✓   ✓

  Result: [11, 12, 22, 25, 64] ✓
```

---

## Maximum Selection Sort (Descending)

To sort in **descending order**, find the **maximum** instead:

```java
public static void selectionSortDesc(int[] arr) {
    int n = arr.length;
    
    for (int i = 0; i < n - 1; i++) {
        int maxIndex = i;
        
        for (int j = i + 1; j < n; j++) {
            if (arr[j] > arr[maxIndex]) {   // find MAX instead
                maxIndex = j;
            }
        }
        
        if (maxIndex != i) {
            int temp = arr[i];
            arr[i] = arr[maxIndex];
            arr[maxIndex] = temp;
        }
    }
}
```

---

## Double Selection Sort (Both ends)

Find both min AND max in each pass, placing them at both ends:

```python
def double_selection_sort(arr):
    n = len(arr)
    left = 0
    right = n - 1
    
    while left < right:
        min_idx = left
        max_idx = left
        
        for i in range(left, right + 1):
            if arr[i] < arr[min_idx]:
                min_idx = i
            if arr[i] > arr[max_idx]:
                max_idx = i
        
        # Place minimum at left
        arr[left], arr[min_idx] = arr[min_idx], arr[left]
        
        # If max was at left, it's been moved to min_idx
        if max_idx == left:
            max_idx = min_idx
        
        # Place maximum at right
        arr[right], arr[max_idx] = arr[max_idx], arr[right]
        
        left += 1
        right -= 1
```

```
  Trace: [5, 3, 1, 4, 2]
  
  Pass 1: min=1(idx 2), max=5(idx 0)
          Swap 5↔1: [1, 3, 5, 4, 2]
          Swap 5↔2: [1, 3, 2, 4, 5]
          → [1, 3, 2, 4, 5]    left=1, right=3
  
  Pass 2: min=2(idx 2), max=4(idx 3)
          Swap 3↔2: [1, 2, 3, 4, 5]
          Already done!  ✓
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Outer loop | i from 0 to n-2 (positions to fill) |
| Inner loop | j from i+1 to n-1 (search for minimum) |
| Key operation | Track minIndex, swap once per pass |
| Total comparisons | n(n-1)/2 |
| Total swaps | At most n-1 |
| Descending variant | Find max instead of min |
| Double selection | Fill both ends simultaneously |

---

## Quick Revision Questions

1. **Write Selection Sort from memory.**
2. **How many swaps occur in the worst case?**
3. **Trace Selection Sort on [3, 5, 1, 2, 4].**
4. **Why do we check `if minIndex != i` before swapping?**
5. **How would you modify it for descending order?**
6. **What is the advantage of double selection sort?**

---

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Complexity Analysis →](03-complexity-analysis.md)
