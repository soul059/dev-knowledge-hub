# Unit 6: Quick Sort — Implementation

[← Previous: Partition Schemes](02-partition-schemes.md) | [Next: Pivot Selection →](04-pivot-selection.md)

---

## Overview

This chapter provides complete, production-quality implementations of Quick Sort in Java, Python, and C++. We cover the standard recursive version, randomized version, and the three-way partition variant.

---

## Standard Quick Sort (Lomuto)

### Java

```java
public class QuickSort {
    
    public static void quickSort(int[] arr) {
        quickSort(arr, 0, arr.length - 1);
    }
    
    private static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pivotIndex = partition(arr, low, high);
            quickSort(arr, low, pivotIndex - 1);   // Left of pivot
            quickSort(arr, pivotIndex + 1, high);   // Right of pivot
        }
    }
    
    private static int partition(int[] arr, int low, int high) {
        int pivot = arr[high];
        int i = low - 1;
        
        for (int j = low; j < high; j++) {
            if (arr[j] <= pivot) {
                i++;
                swap(arr, i, j);
            }
        }
        
        swap(arr, i + 1, high);
        return i + 1;
    }
    
    private static void swap(int[] arr, int a, int b) {
        int temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }
    
    public static void main(String[] args) {
        int[] arr = {10, 7, 8, 9, 1, 5};
        quickSort(arr);
        System.out.println(Arrays.toString(arr));
        // Output: [1, 5, 7, 8, 9, 10]
    }
}
```

### Python

```python
def quick_sort(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1
    
    if low < high:
        pivot_index = partition(arr, low, high)
        quick_sort(arr, low, pivot_index - 1)
        quick_sort(arr, pivot_index + 1, high)

def partition(arr, low, high):
    pivot = arr[high]
    i = low - 1
    
    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    
    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1
```

### C++

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    
    swap(arr[i + 1], arr[high]);
    return i + 1;
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}
```

---

## Randomized Quick Sort

```
  ┌──────────────────────────────────────────────────────────┐
  │  Problem: Fixed pivot (first/last) causes O(n²) on     │
  │  sorted input — a common case!                          │
  │                                                          │
  │  Solution: Pick a RANDOM element as pivot.              │
  │  Expected time: O(n log n) for ANY input.               │
  │                                                          │
  │  Implementation: Swap random element with last element, │
  │  then use standard Lomuto partition.                    │
  └──────────────────────────────────────────────────────────┘
```

### Java (Randomized)

```java
import java.util.Random;

public class RandomizedQuickSort {
    private static Random rand = new Random();
    
    public static void quickSort(int[] arr) {
        quickSort(arr, 0, arr.length - 1);
    }
    
    private static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pivotIndex = randomizedPartition(arr, low, high);
            quickSort(arr, low, pivotIndex - 1);
            quickSort(arr, pivotIndex + 1, high);
        }
    }
    
    private static int randomizedPartition(int[] arr, int low, int high) {
        // Pick random pivot and move to end
        int randomIndex = low + rand.nextInt(high - low + 1);
        swap(arr, randomIndex, high);
        return partition(arr, low, high);  // Standard Lomuto
    }
    
    private static int partition(int[] arr, int low, int high) {
        int pivot = arr[high];
        int i = low - 1;
        
        for (int j = low; j < high; j++) {
            if (arr[j] <= pivot) {
                i++;
                swap(arr, i, j);
            }
        }
        swap(arr, i + 1, high);
        return i + 1;
    }
    
    private static void swap(int[] arr, int a, int b) {
        int temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }
}
```

### Python (Randomized)

```python
import random

def quick_sort_random(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1
    
    if low < high:
        pivot_index = randomized_partition(arr, low, high)
        quick_sort_random(arr, low, pivot_index - 1)
        quick_sort_random(arr, pivot_index + 1, high)

def randomized_partition(arr, low, high):
    rand_idx = random.randint(low, high)
    arr[rand_idx], arr[high] = arr[high], arr[rand_idx]
    return partition(arr, low, high)
```

---

## Quick Sort with Hoare Partition

```java
public class QuickSortHoare {
    
    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int p = hoarePartition(arr, low, high);
            quickSort(arr, low, p);       // NOTE: p, not p-1
            quickSort(arr, p + 1, high);
        }
    }
    
    private static int hoarePartition(int[] arr, int low, int high) {
        int pivot = arr[low];
        int i = low - 1, j = high + 1;
        
        while (true) {
            do { i++; } while (arr[i] < pivot);
            do { j--; } while (arr[j] > pivot);
            
            if (i >= j) return j;
            swap(arr, i, j);
        }
    }
    
    private static void swap(int[] arr, int a, int b) {
        int temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }
}
```

---

## Three-Way Quick Sort

```java
public class QuickSort3Way {
    
    public static void quickSort(int[] arr) {
        quickSort(arr, 0, arr.length - 1);
    }
    
    private static void quickSort(int[] arr, int low, int high) {
        if (low >= high) return;
        
        int lt = low, gt = high, i = low + 1;
        int pivot = arr[low];
        
        while (i <= gt) {
            if (arr[i] < pivot) {
                swap(arr, lt++, i++);
            } else if (arr[i] > pivot) {
                swap(arr, i, gt--);
            } else {
                i++;
            }
        }
        
        // arr[low..lt-1] < pivot = arr[lt..gt] < arr[gt+1..high]
        quickSort(arr, low, lt - 1);
        quickSort(arr, gt + 1, high);
    }
    
    private static void swap(int[] arr, int a, int b) {
        int temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }
}
```

---

## Tail-Call Optimized Quick Sort

```
  Standard Quick Sort makes TWO recursive calls.
  We can eliminate one using tail-call optimization:
  
  Replace the second recursive call with a loop!
```

```java
// Tail-call optimized — reduces stack depth
private static void quickSort(int[] arr, int low, int high) {
    while (low < high) {
        int pi = partition(arr, low, high);
        
        // Recurse on smaller partition, loop on larger
        if (pi - low < high - pi) {
            quickSort(arr, low, pi - 1);  // Smaller part
            low = pi + 1;                  // Loop for larger part
        } else {
            quickSort(arr, pi + 1, high); // Smaller part
            high = pi - 1;                 // Loop for larger part
        }
    }
}
```

```
  This guarantees stack depth ≤ O(log n) even in worst case!
  
  Why? We always recurse on the SMALLER half (size ≤ n/2).
  The larger half is handled iteratively (while loop).
  
  Maximum recursion depth:
  Standard:        O(n) worst case (sorted input)
  Tail-optimized:  O(log n) always!
```

---

## Execution Trace

```
  quickSort([10, 7, 8, 9, 1, 5])
  
  partition with pivot=5:
  [1, 5, 8, 9, 10, 7]  → pivotIndex = 1
  
  ├── quickSort([1], 0, 0)  → base case
  └── quickSort([8, 9, 10, 7], 2, 5)
      
      partition with pivot=7:
      [7, 9, 10, 8]  → pivotIndex = 2
      → not quite, let's trace carefully:
      
      arr = [1, 5, 8, 9, 10, 7], low=2, high=5, pivot=arr[5]=7
      i=1, j scans 2..4:
        j=2: 8 > 7 → skip
        j=3: 9 > 7 → skip
        j=4: 10 > 7 → skip
      swap arr[2]↔arr[5]: [1, 5, 7, 9, 10, 8]  pivotIndex=2
      
      ├── quickSort([], 2, 1)  → base case
      └── quickSort([9, 10, 8], 3, 5)
          
          pivot=8: [8, 10, 9] → pivotIndex=3
          Wait: arr[3..5]=[9,10,8], pivot=arr[5]=8
          j=3: 9>8 skip, j=4: 10>8 skip
          swap arr[3]↔arr[5]: [8, 10, 9] pivotIndex=3
          After: [1,5,7,8,10,9]
          
          ├── quickSort([], 3, 2) → base
          └── quickSort([10, 9], 4, 5)
              pivot=9: swap 10↔9: [9, 10] pivotIndex=4
              After: [1,5,7,8,9,10]  ✓ SORTED!
```

---

## Summary Table

| Variant | Pivot | Worst Case | Duplicates | Stack |
|---------|-------|-----------|------------|-------|
| Standard Lomuto | Last | O(n²) | O(n²) | O(n) |
| Randomized | Random | O(n²) extremely rare | O(n²) | O(n) |
| Hoare | First | O(n²) | O(n²) | O(n) |
| Three-Way | First | O(n²) | O(n) | O(n) |
| Tail-optimized | Any | Same time | Same | O(log n) |

---

## Quick Revision Questions

1. **Write the Lomuto partition from memory.**
2. **What's the difference between Lomuto and Hoare recursive calls?**
3. **How does randomized partition prevent worst-case input?**
4. **Why does three-way partition handle duplicates efficiently?**
5. **How does tail-call optimization reduce stack depth to O(log n)?**
6. **Which variant would you use for an array with all identical elements?**

---

[← Previous: Partition Schemes](02-partition-schemes.md) | [Next: Pivot Selection →](04-pivot-selection.md)
