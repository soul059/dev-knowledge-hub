# Unit 12: Special Sorting Problems — K-Way Merge

[← Previous: Sorting Linked Lists](04-sorting-linked-lists.md) | [Next: External Sorting →](06-external-sorting.md)

---

## Overview

The **K-Way Merge** problem combines K sorted arrays (or lists) into a single sorted output. It's a fundamental building block for external sorting, database operations, and distributed systems.

---

## Problem Statement

```
  Given K sorted arrays, merge them into one sorted array.
  
  Input:
    [1, 5, 9]
    [2, 6, 8]
    [3, 7, 10]
  
  Output: [1, 2, 3, 5, 6, 7, 8, 9, 10]
```

---

## Approach 1: Brute Force

```
  Concatenate all K arrays → Sort the result
  
  Total elements: N = n₁ + n₂ + ... + nₖ
  Time:  O(N log N)
  Space: O(N)
  
  This ignores the fact that individual arrays are sorted!
```

---

## Approach 2: Sequential 2-Way Merge

```
  Merge array 1 with array 2 → result₁
  Merge result₁ with array 3 → result₂
  ...
  Merge resultₖ₋₂ with array K → final
  
  Time: O(N × K)   ← each element may be copied K times
  Space: O(N)
  
  ┌──────────────────────────────────────────┐
  │  [1,5,9] merge [2,6,8] → [1,2,5,6,8,9] │
  │  [1,2,5,6,8,9] merge [3,7,10]           │
  │  → [1,2,3,5,6,7,8,9,10]                 │
  └──────────────────────────────────────────┘
```

---

## Approach 3: Min-Heap (Optimal)

```
  Use a min-heap of size K to always pick the smallest element.
  
  ┌──────────────────────────────────────────────────────────────┐
  │  Step 1: Push first element of each array into min-heap    │
  │  Step 2: Extract min from heap → add to result             │
  │  Step 3: Push next element from that array into heap       │
  │  Step 4: Repeat until heap is empty                        │
  │                                                             │
  │  Time:  O(N log K)   ← each of N elements: O(log K) heap  │
  │  Space: O(K)          ← heap holds K elements              │
  └──────────────────────────────────────────────────────────────┘
```

### Visual Trace

```
  Arrays:  A = [1, 5, 9]
           B = [2, 6, 8]
           C = [3, 7, 10]
  
  Init heap: {(1,A,0), (2,B,0), (3,C,0)}
  
  Step 1: Extract 1 (from A[0])
          Push A[1]=5 → heap: {(2,B,0), (3,C,0), (5,A,1)}
          Result: [1]
  
  Step 2: Extract 2 (from B[0])
          Push B[1]=6 → heap: {(3,C,0), (5,A,1), (6,B,1)}
          Result: [1, 2]
  
  Step 3: Extract 3 (from C[0])
          Push C[1]=7 → heap: {(5,A,1), (6,B,1), (7,C,1)}
          Result: [1, 2, 3]
  
  Step 4: Extract 5 (from A[1])
          Push A[2]=9 → heap: {(6,B,1), (7,C,1), (9,A,2)}
          Result: [1, 2, 3, 5]
  
  ... continue until heap empty ...
  
  Final: [1, 2, 3, 5, 6, 7, 8, 9, 10]
```

### Java Implementation

```java
import java.util.*;

public class KWayMerge {
    
    public static int[] mergeKArrays(int[][] arrays) {
        // Min-heap: {value, arrayIndex, elementIndex}
        PriorityQueue<int[]> heap = new PriorityQueue<>(
            (a, b) -> a[0] - b[0]
        );
        
        int totalSize = 0;
        
        // Push first element of each array
        for (int i = 0; i < arrays.length; i++) {
            if (arrays[i].length > 0) {
                heap.offer(new int[]{arrays[i][0], i, 0});
                totalSize += arrays[i].length;
            }
        }
        
        int[] result = new int[totalSize];
        int idx = 0;
        
        while (!heap.isEmpty()) {
            int[] min = heap.poll();
            result[idx++] = min[0];
            
            int arrIdx = min[1];
            int elemIdx = min[2] + 1;
            
            if (elemIdx < arrays[arrIdx].length) {
                heap.offer(new int[]{arrays[arrIdx][elemIdx], arrIdx, elemIdx});
            }
        }
        
        return result;
    }
    
    public static void main(String[] args) {
        int[][] arrays = {
            {1, 5, 9},
            {2, 6, 8},
            {3, 7, 10}
        };
        System.out.println(Arrays.toString(mergeKArrays(arrays)));
        // [1, 2, 3, 5, 6, 7, 8, 9, 10]
    }
}
```

### Python Implementation

```python
import heapq

def merge_k_arrays(arrays):
    heap = []
    
    # Push first element from each array
    for i, arr in enumerate(arrays):
        if arr:
            heapq.heappush(heap, (arr[0], i, 0))
    
    result = []
    while heap:
        val, arr_idx, elem_idx = heapq.heappop(heap)
        result.append(val)
        
        if elem_idx + 1 < len(arrays[arr_idx]):
            next_val = arrays[arr_idx][elem_idx + 1]
            heapq.heappush(heap, (next_val, arr_idx, elem_idx + 1))
    
    return result

# Example
arrays = [[1, 5, 9], [2, 6, 8], [3, 7, 10]]
print(merge_k_arrays(arrays))  # [1, 2, 3, 5, 6, 7, 8, 9, 10]
```

---

## Approach 4: Divide-and-Conquer (Tournament Style)

```
  Pair up arrays and merge them in rounds, like a tournament bracket.
  
  Round 1: Merge pairs
  [1,5,9]   [2,6,8]   → [1,2,5,6,8,9]
  [3,7,10]  [4,11]    → [3,4,7,10,11]
  
  Round 2: Merge results
  [1,2,5,6,8,9]  [3,4,7,10,11]  → [1,2,3,4,5,6,7,8,9,10,11]
  
  Rounds: lg K
  Work per round: O(N)  (each element merged once)
  Total: O(N log K)
```

```java
public static int[] mergeKDivideConquer(int[][] arrays) {
    if (arrays.length == 0) return new int[0];
    return mergeDC(arrays, 0, arrays.length - 1);
}

private static int[] mergeDC(int[][] arrays, int lo, int hi) {
    if (lo == hi) return arrays[lo];
    int mid = lo + (hi - lo) / 2;
    int[] left = mergeDC(arrays, lo, mid);
    int[] right = mergeDC(arrays, mid + 1, hi);
    return merge2(left, right);
}

private static int[] merge2(int[] a, int[] b) {
    int[] result = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length) {
        result[k++] = (a[i] <= b[j]) ? a[i++] : b[j++];
    }
    while (i < a.length) result[k++] = a[i++];
    while (j < b.length) result[k++] = b[j++];
    return result;
}
```

---

## Merge K Sorted Linked Lists (LeetCode 23)

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> heap = new PriorityQueue<>(
        (a, b) -> a.val - b.val
    );
    
    for (ListNode node : lists) {
        if (node != null) heap.offer(node);
    }
    
    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;
    
    while (!heap.isEmpty()) {
        ListNode min = heap.poll();
        curr.next = min;
        curr = curr.next;
        
        if (min.next != null) {
            heap.offer(min.next);
        }
    }
    
    return dummy.next;
}
```

---

## Complexity Comparison

```
  ┌───────────────────────┬──────────────┬─────────┐
  │  Approach             │ Time         │ Space   │
  ├───────────────────────┼──────────────┼─────────┤
  │  Concatenate + Sort   │ O(N log N)   │ O(N)    │
  │  Sequential Merge     │ O(N × K)     │ O(N)    │
  │  Min-Heap             │ O(N log K) ✓ │ O(K) ✓  │
  │  Divide & Conquer     │ O(N log K) ✓ │ O(N)    │
  └───────────────────────┴──────────────┴─────────┘
  
  N = total elements across all arrays
  K = number of arrays
  
  Min-Heap wins on both time and space!
  
  Key insight: log K << log N  when K << N
  For K=100, N=10⁶:  log K ≈ 7,  log N ≈ 20
```

---

## Applications

```
  1. External Sorting: Merge sorted runs from disk
  2. Database Operations: Merge sorted index segments
  3. Distributed Systems: Combine results from K servers
  4. Log Aggregation: Merge K sorted log files by timestamp
  5. Search Engines: Merge posting lists from multiple shards
  6. MapReduce: Merge phase combines sorted intermediate results
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Merge K sorted arrays/lists into one |
| Optimal approach | Min-heap of size K |
| Time | O(N log K) |
| Space | O(K) for heap |
| Key data structure | Priority Queue (Min-Heap) |
| Applications | External sort, databases, distributed systems |

---

## Quick Revision Questions

1. **Why is the min-heap approach O(N log K) instead of O(N log N)?**
2. **What information does each heap entry need to store?**
3. **Compare the sequential merge approach vs the divide-and-conquer approach.**
4. **Implement K-way merge for K sorted linked lists.**
5. **How is K-way merge used in external sorting?**

---

[← Previous: Sorting Linked Lists](04-sorting-linked-lists.md) | [Next: External Sorting →](06-external-sorting.md)
