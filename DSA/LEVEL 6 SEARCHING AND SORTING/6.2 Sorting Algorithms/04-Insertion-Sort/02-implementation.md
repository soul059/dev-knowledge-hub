# Chapter 2: Insertion Sort — Implementation

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Complexity Analysis →](03-complexity-analysis.md)

---

## Java Implementation

```java
public class InsertionSort {
    
    public static void insertionSort(int[] arr) {
        int n = arr.length;
        
        for (int i = 1; i < n; i++) {
            int key = arr[i];           // element to insert
            int j = i - 1;
            
            // Shift elements greater than key to the right
            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];    // shift right
                j--;
            }
            
            arr[j + 1] = key;           // insert key
        }
    }
    
    public static void main(String[] args) {
        int[] arr = {12, 11, 13, 5, 6};
        insertionSort(arr);
        // Output: [5, 6, 11, 12, 13]
    }
}
```

## Python Implementation

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1
        
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1
        
        arr[j + 1] = key

# Example
arr = [12, 11, 13, 5, 6]
insertion_sort(arr)
print(arr)  # [5, 6, 11, 12, 13]
```

## C++ Implementation

```cpp
void insertionSort(vector<int>& arr) {
    int n = arr.size();
    
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        
        arr[j + 1] = key;
    }
}
```

---

## Complete Execution Trace

```
  Input: [12, 11, 13, 5, 6]    n = 5
  
  ═══ i=1: key=11 ═══
  j=0: arr[0]=12 > 11? YES → shift: [12,12,13,5,6], j=-1
  Insert key at j+1=0: [11,12,13,5,6]
                         ✓  ✓
  
  ═══ i=2: key=13 ═══
  j=1: arr[1]=12 > 13? NO → STOP
  Insert key at j+1=2: [11,12,13,5,6]   (no change, already in place)
                         ✓  ✓  ✓
  
  ═══ i=3: key=5 ═══
  j=2: arr[2]=13 > 5? YES → shift: [11,12,13,13,6], j=1
  j=1: arr[1]=12 > 5? YES → shift: [11,12,12,13,6], j=0
  j=0: arr[0]=11 > 5? YES → shift: [11,11,12,13,6], j=-1
  Insert key at j+1=0: [5,11,12,13,6]
                         ✓  ✓  ✓  ✓
  
  ═══ i=4: key=6 ═══
  j=3: arr[3]=13 > 6? YES → shift: [5,11,12,13,13], j=2
  j=2: arr[2]=12 > 6? YES → shift: [5,11,12,12,13], j=1
  j=1: arr[1]=11 > 6? YES → shift: [5,11,11,12,13], j=0
  j=0: arr[0]=5 > 6?  NO  → STOP
  Insert key at j+1=1: [5,6,11,12,13]
                         ✓  ✓  ✓  ✓  ✓    SORTED!
```

---

## The Shift Mechanism — Detailed View

```
  Inserting key=5 into sorted portion [11, 12, 13]:
  
  Step 0: Save key
  key = 5
  [11, 12, 13, 5, ...]
                ↑ saved
  
  Step 1: 13 > 5 → shift right
  [11, 12, 13, 13, ...]
              →→→
  
  Step 2: 12 > 5 → shift right
  [11, 12, 12, 13, ...]
          →→→
  
  Step 3: 11 > 5 → shift right
  [11, 11, 12, 13, ...]
      →→→
  
  Step 4: j = -1 → STOP, insert key at position 0
  [5, 11, 12, 13, ...]
   ↑ inserted
  
  Each shift is ONE assignment: A[j+1] = A[j]
  Final insert is ONE assignment: A[j+1] = key
```

---

## Descending Order

```java
// Change comparison direction: arr[j] < key instead of arr[j] > key
while (j >= 0 && arr[j] < key) {
    arr[j + 1] = arr[j];
    j--;
}
```

---

## Recursive Insertion Sort

```python
def insertion_sort_recursive(arr, n=None):
    if n is None:
        n = len(arr)
    
    if n <= 1:
        return
    
    # Sort first n-1 elements
    insertion_sort_recursive(arr, n - 1)
    
    # Insert last element into sorted portion
    key = arr[n - 1]
    j = n - 2
    
    while j >= 0 and arr[j] > key:
        arr[j + 1] = arr[j]
        j -= 1
    
    arr[j + 1] = key
```

```
  Recursion trace for [4, 2, 1]:
  
  sort([4,2,1], n=3)
  └── sort([4,2,1], n=2)
      └── sort([4,2,1], n=1) → base case, return
      Insert arr[1]=2 into [4]: → [2,4,1]
  Insert arr[2]=1 into [2,4]: → [1,2,4]
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Outer loop | i from 1 to n-1 |
| Key | arr[i] — element to insert |
| Inner loop | j from i-1 down to 0 (while A[j] > key) |
| Shift | A[j+1] = A[j] (move element right) |
| Insert | A[j+1] = key (place at correct position) |
| Assignments per element | shifts + 2 (save + insert) |

---

## Quick Revision Questions

1. **Write Insertion Sort from memory in your preferred language.**
2. **What happens when key is larger than all sorted elements?**
3. **Trace Insertion Sort on [5, 3, 1, 4, 2].**
4. **Why does j stop at -1 in some cases?**
5. **How many assignments occur when inserting into position 0 of a sorted array of size k?**
6. **How does the recursive version differ structurally?**

---

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Complexity Analysis →](03-complexity-analysis.md)
