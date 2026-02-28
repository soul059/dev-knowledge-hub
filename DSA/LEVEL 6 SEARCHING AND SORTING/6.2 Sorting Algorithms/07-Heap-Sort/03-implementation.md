# Unit 7: Heap Sort — Implementation

[← Previous: Heapify Process](02-heapify.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)

---

## Overview

Heap Sort has two phases: **Build the max-heap**, then **repeatedly extract the maximum**. This chapter provides complete implementations in Java, Python, and C++ with step-by-step traces.

---

## Complete Algorithm Trace

```
  Array: [5, 3, 8, 1, 2]
  
  ══════════════════════════════════════
  PHASE 1: BUILD MAX-HEAP
  ══════════════════════════════════════
  
  Last non-leaf = (5/2) - 1 = 1
  
  Heapify(1): node=3, children=1,2
    3 > 1 and 3 > 2 → no swap
  
  Heapify(0): node=5, children=3,8
    8 > 5 → swap arr[0] ↔ arr[2]
    Array: [8, 3, 5, 1, 2]
    Continue: node=5, no children beyond heap → done
  
  Max-heap: [8, 3, 5, 1, 2]
  
       8
      / \
     3   5
    / \
   1   2
  
  ══════════════════════════════════════
  PHASE 2: EXTRACT & SORT
  ══════════════════════════════════════
  
  Iteration 1: heapSize=5
    Swap arr[0] ↔ arr[4]: [2, 3, 5, 1, | 8]
    Heapify(0), heapSize=4:
      2 < 5 → swap: [5, 3, 2, 1, | 8]
    Array: [5, 3, 2, 1, | 8]
  
       5           Sorted: 8
      / \
     3   2
    /
   1
  
  Iteration 2: heapSize=4
    Swap arr[0] ↔ arr[3]: [1, 3, 2, | 5, 8]
    Heapify(0), heapSize=3:
      1 < 3 → swap: [3, 1, 2, | 5, 8]
    Array: [3, 1, 2, | 5, 8]
  
       3           Sorted: 5, 8
      / \
     1   2
  
  Iteration 3: heapSize=3
    Swap arr[0] ↔ arr[2]: [2, 1, | 3, 5, 8]
    Heapify(0), heapSize=2:
      2 > 1 → no swap
    Array: [2, 1, | 3, 5, 8]
  
       2           Sorted: 3, 5, 8
      /
     1
  
  Iteration 4: heapSize=2
    Swap arr[0] ↔ arr[1]: [1, | 2, 3, 5, 8]
    Heapify(0), heapSize=1: no children → done
    Array: [1, | 2, 3, 5, 8]
  
  RESULT: [1, 2, 3, 5, 8]  ✓ Sorted!
```

---

## Java Implementation

```java
public class HeapSort {
    
    public static void heapSort(int[] arr) {
        int n = arr.length;
        
        // Phase 1: Build max-heap
        // Start from last non-leaf node and heapify each
        for (int i = n / 2 - 1; i >= 0; i--) {
            heapify(arr, n, i);
        }
        
        // Phase 2: Extract elements one by one
        for (int i = n - 1; i > 0; i--) {
            // Move current root (max) to end
            int temp = arr[0];
            arr[0] = arr[i];
            arr[i] = temp;
            
            // Heapify reduced heap
            heapify(arr, i, 0);
        }
    }
    
    private static void heapify(int[] arr, int heapSize, int i) {
        int largest = i;          // Initialize largest as root
        int left = 2 * i + 1;    // Left child
        int right = 2 * i + 2;   // Right child
        
        // If left child is larger than root
        if (left < heapSize && arr[left] > arr[largest]) {
            largest = left;
        }
        
        // If right child is larger than largest so far
        if (right < heapSize && arr[right] > arr[largest]) {
            largest = right;
        }
        
        // If largest is not root
        if (largest != i) {
            int temp = arr[i];
            arr[i] = arr[largest];
            arr[largest] = temp;
            
            // Recursively heapify the affected subtree
            heapify(arr, heapSize, largest);
        }
    }
    
    // Driver
    public static void main(String[] args) {
        int[] arr = {12, 11, 13, 5, 6, 7};
        heapSort(arr);
        System.out.println(java.util.Arrays.toString(arr));
        // Output: [5, 6, 7, 11, 12, 13]
    }
}
```

---

## Python Implementation

```python
def heap_sort(arr):
    n = len(arr)
    
    # Phase 1: Build max-heap
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)
    
    # Phase 2: Extract elements
    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]  # Swap root to end
        heapify(arr, i, 0)               # Heapify reduced heap

def heapify(arr, heap_size, i):
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2
    
    if left < heap_size and arr[left] > arr[largest]:
        largest = left
    if right < heap_size and arr[right] > arr[largest]:
        largest = right
    
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, heap_size, largest)

# Test
arr = [12, 11, 13, 5, 6, 7]
heap_sort(arr)
print(arr)  # [5, 6, 7, 11, 12, 13]
```

---

## C++ Implementation

```cpp
#include <iostream>
#include <vector>
using namespace std;

void heapify(vector<int>& arr, int heapSize, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    
    if (left < heapSize && arr[left] > arr[largest])
        largest = left;
    if (right < heapSize && arr[right] > arr[largest])
        largest = right;
    
    if (largest != i) {
        swap(arr[i], arr[largest]);
        heapify(arr, heapSize, largest);
    }
}

void heapSort(vector<int>& arr) {
    int n = arr.size();
    
    // Build max-heap
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);
    
    // Extract elements
    for (int i = n - 1; i > 0; i--) {
        swap(arr[0], arr[i]);
        heapify(arr, i, 0);
    }
}

int main() {
    vector<int> arr = {12, 11, 13, 5, 6, 7};
    heapSort(arr);
    for (int x : arr) cout << x << " ";
    // Output: 5 6 7 11 12 13
    return 0;
}
```

---

## Iterative Heapify (Avoiding Recursion Stack)

```java
// Replace recursive heapify with iterative version
// Avoids stack overflow for very large arrays
private static void heapifyIterative(int[] arr, int heapSize, int i) {
    while (true) {
        int largest = i;
        int left = 2 * i + 1;
        int right = 2 * i + 2;
        
        if (left < heapSize && arr[left] > arr[largest])
            largest = left;
        if (right < heapSize && arr[right] > arr[largest])
            largest = right;
        
        if (largest == i) break;  // Heap property restored
        
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;
        
        i = largest;  // Move to the swapped position
    }
}
```

---

## Min-Heap Sort (Descending Order)

```java
// To sort in DESCENDING order, use a min-heap
private static void heapifyMin(int[] arr, int heapSize, int i) {
    int smallest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    
    if (left < heapSize && arr[left] < arr[smallest])
        smallest = left;
    if (right < heapSize && arr[right] < arr[smallest])
        smallest = right;
    
    if (smallest != i) {
        int temp = arr[i];
        arr[i] = arr[smallest];
        arr[smallest] = temp;
        heapifyMin(arr, heapSize, smallest);
    }
}

// Result: array sorted in descending order
```

---

## Generic Heap Sort (Custom Comparator)

```java
import java.util.Comparator;

public class GenericHeapSort {
    
    public static <T> void heapSort(T[] arr, Comparator<T> comp) {
        int n = arr.length;
        
        for (int i = n / 2 - 1; i >= 0; i--)
            heapify(arr, n, i, comp);
        
        for (int i = n - 1; i > 0; i--) {
            T temp = arr[0];
            arr[0] = arr[i];
            arr[i] = temp;
            heapify(arr, i, 0, comp);
        }
    }
    
    private static <T> void heapify(T[] arr, int heapSize, int i, 
                                     Comparator<T> comp) {
        int largest = i;
        int left = 2 * i + 1;
        int right = 2 * i + 2;
        
        if (left < heapSize && comp.compare(arr[left], arr[largest]) > 0)
            largest = left;
        if (right < heapSize && comp.compare(arr[right], arr[largest]) > 0)
            largest = right;
        
        if (largest != i) {
            T temp = arr[i];
            arr[i] = arr[largest];
            arr[largest] = temp;
            heapify(arr, heapSize, largest, comp);
        }
    }
}
```

---

## Common Implementation Pitfalls

```
  ┌─────────────────────────────────────────────────────────────┐
  │  PITFALL 1: Off-by-one in build-heap                       │
  │  ✗ for (int i = n/2; i >= 0; i--)    // starts too high   │
  │  ✓ for (int i = n/2 - 1; i >= 0; i--)                     │
  │                                                             │
  │  PITFALL 2: Using n instead of heapSize in extract phase   │
  │  ✗ heapify(arr, n, 0)       // includes sorted section    │
  │  ✓ heapify(arr, i, 0)       // only unsorted portion      │
  │                                                             │
  │  PITFALL 3: Forgetting to swap before heapify              │
  │  Must swap arr[0] ↔ arr[i] THEN heapify                   │
  │                                                             │
  │  PITFALL 4: Modifying heapSize incorrectly                 │
  │  The sorted section grows from the END of the array        │
  │  heapSize shrinks by 1 each extraction                     │
  └─────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Implementation Aspect | Details |
|---|---|
| Build heap loop | `i = n/2 - 1` down to `0` |
| Extract loop | `i = n-1` down to `1` |
| Heap size during extract | Decreases: `n-1, n-2, ..., 1` |
| Recursive vs iterative heapify | Both O(log n); iterative avoids stack |
| Min-heap variant | Change `>` to `<` in comparisons → descending sort |
| Generic version | Use `Comparator<T>` for custom ordering |

---

## Quick Revision Questions

1. **What are the two phases of heap sort?**
2. **Why do we swap arr[0] with arr[i] and not just remove the root?**
3. **In the extract phase, why is heapify called with size `i` instead of `n`?**
4. **How would you modify heap sort to sort in descending order?**
5. **What's the advantage of iterative heapify over recursive?**

---

[← Previous: Heapify Process](02-heapify.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)
