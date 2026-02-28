# Unit 7: Heap Sort — Applications

[← Previous: Complexity Analysis](04-complexity-analysis.md) | [Next: Priority Queue →](06-priority-queue.md)

---

## Overview

While Heap Sort itself is rarely the fastest general-purpose sort, heap-based techniques are fundamental to many important algorithms. The heap data structure shines in problems involving **partial sorting, streaming data, and selection**.

---

## 1. Finding K Largest (or Smallest) Elements

```
  Problem: Find the K largest elements from an array of size N.
  
  ╔══════════════════════════════════════════════════════════════╗
  ║  Approach 1: Sort entire array → O(n log n)               ║
  ║  Approach 2: Build max-heap + extract K times → O(n + K log n) ║
  ║  Approach 3: Use a MIN-heap of size K → O(n log K)        ║
  ╚══════════════════════════════════════════════════════════════╝
  
  Approach 3 is best when K << N!
```

### Min-Heap of Size K

```
  Array: [7, 10, 4, 3, 20, 15], K = 3
  
  Process each element:
  
  Step 1: 7  → heap: [7]           size < K, insert
  Step 2: 10 → heap: [7, 10]      size < K, insert
  Step 3: 4  → heap: [4, 10, 7]   size = K (min-heap)
  
       4          ← min at top
      / \
    10    7
  
  Step 4: 3 → 3 < 4 (heap min)? YES → skip (too small)
  Step 5: 20 → 20 > 4? YES → remove 4, insert 20
     heap: [7, 10, 20]
  Step 6: 15 → 15 > 7? YES → remove 7, insert 15
     heap: [10, 15, 20]
  
  Result: [10, 15, 20] ← 3 largest elements ✓
```

### Java Implementation

```java
public static int[] kLargest(int[] arr, int k) {
    // Min-heap of size k
    PriorityQueue<Integer> minHeap = new PriorityQueue<>(k);
    
    for (int num : arr) {
        if (minHeap.size() < k) {
            minHeap.offer(num);
        } else if (num > minHeap.peek()) {
            minHeap.poll();
            minHeap.offer(num);
        }
    }
    
    return minHeap.stream().mapToInt(Integer::intValue).toArray();
}
// Time: O(n log k), Space: O(k)
```

---

## 2. Sorting a Nearly Sorted (K-Sorted) Array

```
  Problem: Each element is at most K positions away from its
           sorted position.
  
  Array: [2, 6, 3, 12, 56, 8]    K = 3
  
  Key insight: The minimum element must be among the first K+1 elements!
  
  Use a min-heap of size K+1:
  
  ┌──────────────────────────────────────────────────────────┐
  │  1. Insert first K+1 elements: {2, 6, 3, 12}           │
  │     Heap: [2, 6, 3, 12]                                │
  │                                                          │
  │  2. Extract-min → 2, insert 56                          │
  │     Output: [2]    Heap: [3, 6, 56, 12]                 │
  │                                                          │
  │  3. Extract-min → 3, insert 8                           │
  │     Output: [2, 3]    Heap: [6, 12, 56, 8]             │
  │                                                          │
  │  4. No more elements; extract all remaining:            │
  │     → 6, 8, 12, 56                                      │
  │                                                          │
  │  Result: [2, 3, 6, 8, 12, 56] ✓                        │
  └──────────────────────────────────────────────────────────┘
  
  Time: O(n log k)  — MUCH better than O(n log n) when k << n
```

### Java Implementation

```java
public static void sortNearlySorted(int[] arr, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    
    int index = 0;
    for (int i = 0; i < arr.length; i++) {
        minHeap.offer(arr[i]);
        
        if (minHeap.size() > k) {
            arr[index++] = minHeap.poll();
        }
    }
    
    // Empty remaining elements
    while (!minHeap.isEmpty()) {
        arr[index++] = minHeap.poll();
    }
}
```

---

## 3. Merge K Sorted Lists/Arrays

```
  Problem: Merge K sorted arrays into one sorted array.
  
  Lists:  [1, 5, 9]
          [2, 6, 10]
          [3, 7, 11]
  
  Use a min-heap of size K:
  
  ┌─────────────────────────────────────────────────────────┐
  │  Heap stores: (value, list_index, element_index)       │
  │                                                         │
  │  Init: heap = [(1,0,0), (2,1,0), (3,2,0)]             │
  │                                                         │
  │  Extract 1 → add next from list 0: (5,0,1)            │
  │  Extract 2 → add next from list 1: (6,1,1)            │
  │  Extract 3 → add next from list 2: (7,2,1)            │
  │  Extract 5 → add next from list 0: (9,0,2)            │
  │  ...                                                    │
  │  Output: [1, 2, 3, 5, 6, 7, 9, 10, 11]                │
  └─────────────────────────────────────────────────────────┘
  
  Time: O(N log K) where N = total elements, K = number of lists
```

---

## 4. Kth Largest/Smallest Element (Selection Problem)

```
  Problem: Find the Kth largest element.
  
  ┌────────────────────────────────────────────────────────────┐
  │  Method 1: Sort → O(n log n)                             │
  │  Method 2: Max-heap, extract K times → O(n + K log n)    │
  │  Method 3: Min-heap of size K → O(n log K)               │
  │  Method 4: QuickSelect → O(n) average                    │
  └────────────────────────────────────────────────────────────┘
  
  Heap approach (Method 2):
  
  Array: [3, 2, 1, 5, 6, 4], K = 2  (2nd largest)
  
  1. Build max-heap: [6, 5, 4, 2, 3, 1]
  2. Extract max → 6 (1st largest)
  3. Heapify → [5, 3, 4, 2, 1]
  4. Extract max → 5 (2nd largest) ← ANSWER
  
  Time: O(n + K log n)
  Better than full sort when K is small!
```

---

## 5. Median Maintenance (Running Median)

```
  Problem: Maintain the median as elements arrive one by one.
  
  Strategy: Use TWO heaps!
  
  ┌──────────────────────────────────────────────────────┐
  │  MAX-HEAP (left half)    │   MIN-HEAP (right half)  │
  │  stores smaller half     │   stores larger half     │
  │                          │                           │
  │      ┌───┐              │         ┌───┐            │
  │      │ 3 │ ← max        │   min → │ 5 │            │
  │      ├───┤              │         ├───┤            │
  │      │ 2 │              │         │ 7 │            │
  │      ├───┤              │         ├───┤            │
  │      │ 1 │              │         │ 9 │            │
  │      └───┘              │         └───┘            │
  │                          │                           │
  │  Median = max of left heap = 3                      │
  │  (when sizes equal, median = avg of both tops)      │
  └──────────────────────────────────────────────────────┘
  
  Invariant: |size(maxHeap) - size(minHeap)| ≤ 1
  
  Insert rules:
  1. If num ≤ maxHeap.top → insert into maxHeap
  2. Else → insert into minHeap
  3. Rebalance if size difference > 1
  
  Median query: O(1)
  Insert: O(log n)
```

---

## 6. External Sorting (Large Files)

```
  When data doesn't fit in memory:
  
  ┌─────────────────────────────────────────────────────────┐
  │  Phase 1: Create sorted runs                           │
  │    Read chunks → sort each → write to disk             │
  │                                                         │
  │  Phase 2: K-way merge using min-heap                   │
  │    Heap holds 1 element from each sorted run           │
  │    Extract-min → write to output                       │
  │    Insert next element from same run                   │
  │                                                         │
  │  Run 1: [1, 5, 9, ...]  ─┐                            │
  │  Run 2: [2, 6, 10, ...] ─┼── min-heap → output file   │
  │  Run 3: [3, 7, 11, ...] ─┘                            │
  └─────────────────────────────────────────────────────────┘
  
  This is how databases and OS utilities sort huge files!
```

---

## 7. When to Use Heap Sort

```
  ✓ USE HEAP SORT WHEN:
  ┌──────────────────────────────────────────────────────────┐
  │  • Worst-case O(n log n) guarantee needed              │
  │  • Memory is severely constrained (O(1) space)         │
  │  • Security-sensitive (no O(n²) worst case)            │
  │  • Embedded systems with limited stack/memory          │
  │  • Part of Introsort (fallback from quicksort)         │
  └──────────────────────────────────────────────────────────┘
  
  ✗ DON'T USE HEAP SORT WHEN:
  ┌──────────────────────────────────────────────────────────┐
  │  • Stability is required → use Merge Sort              │
  │  • Average-case speed matters most → use Quick Sort    │
  │  • Data is nearly sorted → use Insertion Sort          │
  │  • Cache performance is critical → Quick Sort wins     │
  └──────────────────────────────────────────────────────────┘
```

---

## Applications Summary Table

| Application | Technique | Time Complexity |
|---|---|---|
| K largest elements | Min-heap of size K | O(n log K) |
| Sort nearly sorted | Min-heap of size K+1 | O(n log K) |
| Merge K sorted lists | Min-heap of size K | O(N log K) |
| Kth largest element | Max-heap + extract K | O(n + K log n) |
| Running median | Two heaps (max + min) | O(log n) per insert |
| External sorting | K-way merge with heap | O(N log K) |

---

## Quick Revision Questions

1. **How would you find the K smallest elements using a heap?**
2. **Why is a min-heap of size K better than sorting for K largest?**
3. **Explain how two heaps solve the running median problem.**
4. **What makes heap sort suitable for embedded systems?**
5. **How is a heap used in external merge sort?**

---

[← Previous: Complexity Analysis](04-complexity-analysis.md) | [Next: Priority Queue →](06-priority-queue.md)
