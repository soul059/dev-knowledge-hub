# Unit 6: Quick Sort — Partition Schemes

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Implementation →](03-implementation.md)

---

## Overview

The partition function is the **heart of Quick Sort**. There are two main partition schemes — **Lomuto** and **Hoare** — each with different trade-offs. Understanding both deeply is essential for interviews and competitive programming.

---

## Lomuto Partition Scheme

```
  Uses the LAST element as pivot.
  Maintains a boundary between ≤ pivot and > pivot regions.
  
  Convention:
  - i tracks the boundary of the "≤ pivot" region
  - j scans through the array left to right
  
  ┌──────────────────────────────────────────────────────────┐
  │  Array layout during Lomuto partition:                   │
  │                                                          │
  │  [  ≤ pivot  |  > pivot  |  unexamined  | pivot ]       │
  │   low     i    i+1    j-1   j       high-1  high        │
  └──────────────────────────────────────────────────────────┘
```

### Lomuto Step-by-Step Trace

```
  Array: [7, 2, 1, 6, 8, 5, 3, 4]    pivot = 4, i = -1
  
  j=0: arr[0]=7 > 4   → skip
       [7, 2, 1, 6, 8, 5, 3, 4]    i=-1
  
  j=1: arr[1]=2 ≤ 4   → i=0, swap arr[0]↔arr[1]
       [2, 7, 1, 6, 8, 5, 3, 4]    i=0
  
  j=2: arr[2]=1 ≤ 4   → i=1, swap arr[1]↔arr[2]
       [2, 1, 7, 6, 8, 5, 3, 4]    i=1
  
  j=3: arr[3]=6 > 4   → skip
       [2, 1, 7, 6, 8, 5, 3, 4]    i=1
  
  j=4: arr[4]=8 > 4   → skip
       [2, 1, 7, 6, 8, 5, 3, 4]    i=1
  
  j=5: arr[5]=5 > 4   → skip
       [2, 1, 7, 6, 8, 5, 3, 4]    i=1
  
  j=6: arr[6]=3 ≤ 4   → i=2, swap arr[2]↔arr[6]
       [2, 1, 3, 6, 8, 5, 7, 4]    i=2
  
  Final: swap arr[i+1]↔arr[high] → swap arr[3]↔arr[7]
       [2, 1, 3, 4, 8, 5, 7, 6]
                 ↑
            pivot at index 3 (returned)
```

### Lomuto Code

```java
int lomutoPartition(int[] arr, int low, int high) {
    int pivot = arr[high];   // Last element as pivot
    int i = low - 1;         // Boundary of ≤ region
    
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr, i, j);
        }
    }
    
    swap(arr, i + 1, high);  // Place pivot
    return i + 1;             // Return pivot index
}
```

---

## Hoare Partition Scheme

```
  Uses the FIRST element as pivot.
  Two pointers move TOWARD each other from both ends.
  
  ┌──────────────────────────────────────────────────────────┐
  │  Convention:                                             │
  │  - i starts from left, moves right to find element > p  │
  │  - j starts from right, moves left to find element < p  │
  │  - When both found, swap them                           │
  │  - When pointers cross, partition is complete           │
  │                                                          │
  │  [  ≤ pivot  |  unknown  |  ≥ pivot  ]                 │
  │   low     i→               ←j     high                 │
  └──────────────────────────────────────────────────────────┘
  
  NOTE: In Hoare scheme, the pivot does NOT necessarily
  end up at the returned index! The return value is
  the split point, not the pivot position.
```

### Hoare Step-by-Step Trace

```
  Array: [7, 2, 1, 6, 8, 5, 3, 4]    pivot = 7 (first element)
  
  i=0, j=7:
    Move i right while arr[i] < 7 → i=0 (arr[0]=7, stop)
    Move j left while arr[j] > 7  → j=7 (arr[7]=4 < 7, stop)
    i < j? Yes → swap arr[0]↔arr[7]
    [4, 2, 1, 6, 8, 5, 3, 7]    i=0, j=7
  
  i=1, j=6:
    Move i right while arr[i] < 7 → i stays at 1,2,3 → i=4 (arr[4]=8 ≥ 7)
    Move j left while arr[j] > 7  → j=7 (arr[7]=7, ≥7) → j=6 (arr[6]=3 < 7)
    i < j? 4 < 6? Yes → swap arr[4]↔arr[6]
    [4, 2, 1, 6, 3, 5, 8, 7]    i=4, j=6
  
  i=5, j=5:
    Move i right while arr[i] < 7 → i=5 (arr[5]=5 < 7) → i=6 (arr[6]=8 ≥ 7)
    Move j left while arr[j] > 7  → j=6 (arr[6]=8 > 7) → j=5 (arr[5]=5 < 7)
    i < j? 6 < 5? No → return j = 5
  
  Result: [4, 2, 1, 6, 3, 5, | 8, 7]
           ← all ≤ 7 →        ← ≥ 7 →
  
  Return index 5 (the split point)
```

### Hoare Code

```java
int hoarePartition(int[] arr, int low, int high) {
    int pivot = arr[low];    // First element as pivot
    int i = low - 1;
    int j = high + 1;
    
    while (true) {
        // Move i right until we find element >= pivot
        do { i++; } while (arr[i] < pivot);
        
        // Move j left until we find element <= pivot
        do { j--; } while (arr[j] > pivot);
        
        // If pointers crossed, partition is done
        if (i >= j) return j;
        
        // Swap elements on wrong sides
        swap(arr, i, j);
    }
}

// NOTE: QuickSort with Hoare partition has different recursive calls:
void quickSortHoare(int[] arr, int low, int high) {
    if (low < high) {
        int p = hoarePartition(arr, low, high);
        quickSortHoare(arr, low, p);        // NOT p-1!
        quickSortHoare(arr, p + 1, high);
    }
}
```

---

## Lomuto vs Hoare Comparison

```
  ┌────────────────────┬─────────────────────┬────────────────────┐
  │  Aspect            │  Lomuto             │  Hoare             │
  ├────────────────────┼─────────────────────┼────────────────────┤
  │  Pivot choice      │  Last element       │  First element     │
  │  Pointer count     │  One (j scans)      │  Two (i→, ←j)     │
  │  Swaps (avg)       │  ~n/2               │  ~n/6              │
  │  Swaps (when sorted)│ n-1 (every element)│  ~n/6              │
  │  Comparisons       │  n-1                │  n+1 (slightly more)│
  │  Pivot position    │  EXACT final pos    │  NOT necessarily   │
  │  Code simplicity   │  Simpler            │  Trickier          │
  │  Performance       │  Slower             │  ~3x fewer swaps   │
  │  Infinite loop risk│  No                 │  Yes (if not careful)│
  │  Stability         │  Unstable           │  Unstable          │
  │  Common use        │  Teaching           │  Production        │
  └────────────────────┴─────────────────────┴────────────────────┘
  
  In practice, Hoare is FASTER due to 3x fewer swaps!
```

---

## Why Hoare Does Fewer Swaps

```
  Consider: [1, 2, 3, 4, 5]  pivot = 1
  
  Lomuto: Every element ≤ 1 triggers a swap
    j=0: 1 ≤ 1 → swap(0,0) [self-swap]
    All others > 1 → skip
    Final swap: put pivot → 1 unnecessary swap
    Total: 1 swap (not bad for this case)
  
  But for: [2, 1, 3, 4, 5]  pivot = 5 (Lomuto)
    j=0: 2 ≤ 5 → i=0, swap(0,0) [self-swap]
    j=1: 1 ≤ 5 → i=1, swap(1,1) [self-swap]  
    j=2: 3 ≤ 5 → i=2, swap(2,2) [self-swap]
    j=3: 4 ≤ 5 → i=3, swap(3,3) [self-swap]
    Total: 4 self-swaps + 1 final = WASTEFUL
  
  Hoare for same array: pivot = 2
    i→ finds 2 (≥2), j← finds 5 (wait, >2)→ 4→ 3→ 1 (≤2)
    Swap 2↔1: [1, 2, 3, 4, 5]
    Pointers cross → done. Only 1 actual swap!
  
  Hoare avoids self-swaps and scans from both ends efficiently.
```

---

## Three-Way Partition (Dutch National Flag)

```
  When array has many DUPLICATE elements, standard partition
  puts all duplicates on one side → unbalanced!
  
  [4, 4, 4, 4, 4, 4, 4, 4]  pivot = 4
  Lomuto: [4] [4, 4, 4, 4, 4, 4, 4]  → O(n²)!
  
  THREE-WAY PARTITION (Dijkstra):
  Separates into three regions: < pivot, = pivot, > pivot
  
  [  < pivot  |  = pivot  |  unknown  |  > pivot  ]
   low      lt             i         gt       high
  
  After partition:
  [1, 2, 3, | 4, 4, 4, 4, | 7, 8, 9]
   < pivot    = pivot (DONE!)  > pivot
  
  Only recurse on < and > regions — skip ALL duplicates!
```

```java
// Three-way partition (Dijkstra's Dutch National Flag)
void quickSort3Way(int[] arr, int low, int high) {
    if (low >= high) return;
    
    int lt = low, gt = high;
    int pivot = arr[low];
    int i = low + 1;
    
    while (i <= gt) {
        if (arr[i] < pivot) {
            swap(arr, lt++, i++);
        } else if (arr[i] > pivot) {
            swap(arr, i, gt--);
        } else {
            i++;  // Equal to pivot, leave in place
        }
    }
    
    // arr[low..lt-1] < pivot = arr[lt..gt] < arr[gt+1..high]
    quickSort3Way(arr, low, lt - 1);
    quickSort3Way(arr, gt + 1, high);
}
```

```
  Trace: [4, 7, 2, 4, 1, 4, 8, 4]  pivot = 4
  
  lt=0, i=1, gt=7
  i=1: 7 > 4 → swap(1,7), gt=6    [4, 4, 2, 4, 1, 4, 8, 7]
  i=1: 4 = 4 → i=2                [4, 4, 2, 4, 1, 4, 8, 7]
  i=2: 2 < 4 → swap(0,2), lt=1, i=3  [2, 4, 4, 4, 1, 4, 8, 7]
  i=3: 4 = 4 → i=4                [2, 4, 4, 4, 1, 4, 8, 7]
  i=4: 1 < 4 → swap(1,4), lt=2, i=5  [2, 1, 4, 4, 4, 4, 8, 7]
  i=5: 4 = 4 → i=6                [2, 1, 4, 4, 4, 4, 8, 7]
  i=6: 8 > 4 → swap(6,6), gt=5    [2, 1, 4, 4, 4, 4, 8, 7]
  i=6 > gt=5 → STOP
  
  Result: [2, 1, | 4, 4, 4, 4, | 8, 7]
          < 4     = 4 (skip!)    > 4
  
  Recurse on [2,1] and [8,7] only!
```

---

## Which Partition to Use?

```
                Start
                  │
                  ▼
          ┌───────────────────┐
          │  Many duplicates? │── YES ──→ THREE-WAY PARTITION
          └────────┬──────────┘
                   │ NO
                   ▼
          ┌───────────────────┐
          │  Learning/simple? │── YES ──→ LOMUTO
          └────────┬──────────┘
                   │ NO
                   ▼
          ┌───────────────────┐
          │  Performance      │── YES ──→ HOARE
          │  matters?         │
          └───────────────────┘
```

---

## Summary Table

| Scheme | Swaps (avg) | Pivot Position | Duplicates | Use Case |
|--------|------------|----------------|------------|----------|
| Lomuto | ~n/2 | Exact final | Poor (O(n²)) | Teaching, simple |
| Hoare | ~n/6 | Not exact | Poor (O(n²)) | Production |
| Three-Way | ~n/3 | Range [lt..gt] | Excellent (O(n)) | Data with duplicates |

---

## Quick Revision Questions

1. **What are the invariants maintained by Lomuto partition?**
2. **Why does Hoare partition do ~3x fewer swaps than Lomuto?**
3. **In Hoare partition, why is the recursive call `quickSort(low, p)` and not `quickSort(low, p-1)`?**
4. **When does three-way partitioning shine?**
5. **What happens if all elements are equal with Lomuto? With three-way?**
6. **Draw the array state after Lomuto partition of [5, 3, 8, 6, 2, 7, 1, 4].**

---

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Implementation →](03-implementation.md)
