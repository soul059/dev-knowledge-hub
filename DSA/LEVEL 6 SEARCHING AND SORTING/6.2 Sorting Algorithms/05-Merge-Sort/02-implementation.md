# Unit 5: Merge Sort — Implementation

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Merge Process Deep Dive →](03-merge-process.md)

---

## Overview

This chapter provides complete implementations of Merge Sort in Java, Python, and C++ with detailed explanations of every part. We cover the standard recursive approach, memory optimization, and common pitfalls.

---

## Java Implementation

```java
public class MergeSort {
    
    public static void mergeSort(int[] arr) {
        if (arr.length <= 1) return;
        mergeSort(arr, 0, arr.length - 1);
    }
    
    private static void mergeSort(int[] arr, int left, int right) {
        if (left >= right) return;  // Base case: 0 or 1 element
        
        int mid = left + (right - left) / 2;  // Avoid overflow
        
        mergeSort(arr, left, mid);       // Sort left half
        mergeSort(arr, mid + 1, right);  // Sort right half
        merge(arr, left, mid, right);     // Merge sorted halves
    }
    
    private static void merge(int[] arr, int left, int mid, int right) {
        // Create temporary arrays
        int n1 = mid - left + 1;  // Size of left subarray
        int n2 = right - mid;      // Size of right subarray
        
        int[] L = new int[n1];
        int[] R = new int[n2];
        
        // Copy data to temp arrays
        for (int i = 0; i < n1; i++)
            L[i] = arr[left + i];
        for (int j = 0; j < n2; j++)
            R[j] = arr[mid + 1 + j];
        
        // Merge back into arr[left..right]
        int i = 0, j = 0, k = left;
        
        while (i < n1 && j < n2) {
            if (L[i] <= R[j]) {  // <= for stability
                arr[k] = L[i];
                i++;
            } else {
                arr[k] = R[j];
                j++;
            }
            k++;
        }
        
        // Copy remaining elements
        while (i < n1) {
            arr[k] = L[i];
            i++; k++;
        }
        while (j < n2) {
            arr[k] = R[j];
            j++; k++;
        }
    }
    
    // Test
    public static void main(String[] args) {
        int[] arr = {38, 27, 43, 3, 9, 82, 10};
        mergeSort(arr);
        System.out.println(Arrays.toString(arr));
        // Output: [3, 9, 10, 27, 38, 43, 82]
    }
}
```

---

## Python Implementation

```python
def merge_sort(arr):
    """Sort array in-place using merge sort."""
    if len(arr) <= 1:
        return
    
    mid = len(arr) // 2
    left = arr[:mid]    # Create left subarray
    right = arr[mid:]   # Create right subarray
    
    merge_sort(left)    # Recursively sort left
    merge_sort(right)   # Recursively sort right
    
    merge(arr, left, right)  # Merge back

def merge(arr, left, right):
    """Merge two sorted arrays into arr."""
    i = j = k = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:  # <= for stability
            arr[k] = left[i]
            i += 1
        else:
            arr[k] = right[j]
            j += 1
        k += 1
    
    # Copy remaining elements
    while i < len(left):
        arr[k] = left[i]
        i += 1; k += 1
    
    while j < len(right):
        arr[k] = right[j]
        j += 1; k += 1

# Cleaner Pythonic version (returns new sorted list)
def merge_sort_functional(arr):
    """Functional merge sort - returns new sorted list."""
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort_functional(arr[:mid])
    right = merge_sort_functional(arr[mid:])
    
    return merge_functional(left, right)

def merge_functional(left, right):
    """Merge two sorted lists into a new sorted list."""
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

---

## C++ Implementation

```cpp
#include <vector>
using namespace std;

void merge(vector<int>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    
    vector<int> L(arr.begin() + left, arr.begin() + mid + 1);
    vector<int> R(arr.begin() + mid + 1, arr.begin() + right + 1);
    
    int i = 0, j = 0, k = left;
    
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }
    
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void mergeSort(vector<int>& arr, int left, int right) {
    if (left >= right) return;
    
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

void mergeSort(vector<int>& arr) {
    if (arr.size() <= 1) return;
    mergeSort(arr, 0, arr.size() - 1);
}
```

---

## Execution Trace

```
  mergeSort([38, 27, 43, 3, 9, 82, 10], 0, 6)
  │
  ├── mergeSort([38, 27, 43, 3], 0, 3)
  │   ├── mergeSort([38, 27], 0, 1)
  │   │   ├── mergeSort([38], 0, 0) → return (base case)
  │   │   ├── mergeSort([27], 1, 1) → return (base case)
  │   │   └── merge(arr, 0, 0, 1) → [27, 38, ...]
  │   │
  │   ├── mergeSort([43, 3], 2, 3)
  │   │   ├── mergeSort([43], 2, 2) → return (base case)
  │   │   ├── mergeSort([3], 3, 3) → return (base case)
  │   │   └── merge(arr, 2, 2, 3) → [..., 3, 43, ...]
  │   │
  │   └── merge(arr, 0, 1, 3) → [3, 27, 38, 43, ...]
  │
  ├── mergeSort([9, 82, 10], 4, 6)
  │   ├── mergeSort([9, 82], 4, 5)
  │   │   ├── mergeSort([9], 4, 4) → return
  │   │   ├── mergeSort([82], 5, 5) → return
  │   │   └── merge(arr, 4, 4, 5) → [..., 9, 82, ...]
  │   │
  │   ├── mergeSort([10], 6, 6) → return
  │   │
  │   └── merge(arr, 4, 5, 6) → [..., 9, 10, 82]
  │
  └── merge(arr, 0, 3, 6) → [3, 9, 10, 27, 38, 43, 82]  ✓
```

---

## Memory-Optimized Version (Single Auxiliary Array)

```java
public class MergeSortOptimized {
    
    // Allocate auxiliary array ONCE instead of per merge call
    public static void mergeSort(int[] arr) {
        int[] aux = new int[arr.length];  // Single allocation
        mergeSort(arr, aux, 0, arr.length - 1);
    }
    
    private static void mergeSort(int[] arr, int[] aux, int left, int right) {
        if (left >= right) return;
        
        int mid = left + (right - left) / 2;
        mergeSort(arr, aux, left, mid);
        mergeSort(arr, aux, mid + 1, right);
        merge(arr, aux, left, mid, right);
    }
    
    private static void merge(int[] arr, int[] aux, int left, int mid, int right) {
        // Copy to auxiliary array
        for (int k = left; k <= right; k++) {
            aux[k] = arr[k];
        }
        
        int i = left, j = mid + 1;
        
        for (int k = left; k <= right; k++) {
            if (i > mid)              arr[k] = aux[j++];  // Left exhausted
            else if (j > right)       arr[k] = aux[i++];  // Right exhausted
            else if (aux[i] <= aux[j]) arr[k] = aux[i++]; // Left smaller
            else                      arr[k] = aux[j++];  // Right smaller
        }
    }
}
```

```
  Memory comparison:
  
  Standard:   O(n) total but many small allocations
              merge calls: n/2 + n/4 + n/8 + ... = n-1 allocations
  
  Optimized:  O(n) total with 1 allocation
              Reuses same auxiliary array for all merges
              Much better for garbage collection / cache performance
```

---

## Common Pitfalls

```
  ┌──────────────────────────────────────────────────────────┐
  │  PITFALL 1: Integer Overflow in Midpoint                 │
  │                                                          │
  │  WRONG:  mid = (left + right) / 2                       │
  │  RIGHT:  mid = left + (right - left) / 2                │
  │                                                          │
  │  When left + right > Integer.MAX_VALUE, overflow occurs! │
  ├──────────────────────────────────────────────────────────┤
  │  PITFALL 2: Using < instead of <= in merge              │
  │                                                          │
  │  WRONG:  if (L[i] < R[j])     ← UNSTABLE!             │
  │  RIGHT:  if (L[i] <= R[j])    ← stable                │
  │                                                          │
  │  < causes right element to come first when equal        │
  ├──────────────────────────────────────────────────────────┤
  │  PITFALL 3: Forgetting to copy remaining elements       │
  │                                                          │
  │  After the main while loop, one array might still       │
  │  have unprocessed elements. Must copy them!             │
  ├──────────────────────────────────────────────────────────┤
  │  PITFALL 4: Off-by-one in subarray sizes               │
  │                                                          │
  │  n1 = mid - left + 1   (includes mid)                   │
  │  n2 = right - mid       (excludes mid)                  │
  └──────────────────────────────────────────────────────────┘
```

---

## Generic Merge Sort (Java)

```java
public static <T extends Comparable<T>> void mergeSort(T[] arr) {
    @SuppressWarnings("unchecked")
    T[] aux = (T[]) new Comparable[arr.length];
    mergeSort(arr, aux, 0, arr.length - 1);
}

private static <T extends Comparable<T>> void mergeSort(T[] arr, T[] aux, int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    mergeSort(arr, aux, lo, mid);
    mergeSort(arr, aux, mid + 1, hi);
    merge(arr, aux, lo, mid, hi);
}

private static <T extends Comparable<T>> void merge(T[] arr, T[] aux, int lo, int mid, int hi) {
    for (int k = lo; k <= hi; k++) aux[k] = arr[k];
    
    int i = lo, j = mid + 1;
    for (int k = lo; k <= hi; k++) {
        if (i > mid)                        arr[k] = aux[j++];
        else if (j > hi)                    arr[k] = aux[i++];
        else if (aux[i].compareTo(aux[j]) <= 0) arr[k] = aux[i++];
        else                                arr[k] = aux[j++];
    }
}
```

---

## Summary Table

| Implementation | Language | Space Strategy | Best For |
|---------------|----------|----------------|----------|
| Standard | Java/C++ | New arrays per merge | Clarity, learning |
| Optimized | Java | Single auxiliary array | Production use |
| Functional | Python | Returns new lists | Functional style |
| Generic | Java | Single auxiliary array | Any Comparable type |

---

## Quick Revision Questions

1. **Why do we use `left + (right - left) / 2` instead of `(left + right) / 2`?**
2. **How does the optimized version reduce memory allocations?**
3. **What happens if you use `<` instead of `<=` in the merge comparison?**
4. **Write the merge function from memory — what are the 4 cases in the merge loop?**
5. **How many times is the merge function called for an array of size n?**
6. **What is the total space used by the standard vs. optimized implementation?**

---

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Merge Process Deep Dive →](03-merge-process.md)
