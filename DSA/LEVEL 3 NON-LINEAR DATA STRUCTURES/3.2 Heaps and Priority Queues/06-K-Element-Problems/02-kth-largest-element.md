# 6.2 Kth Largest Element

[← Previous: The Golden Rule](01-golden-rule.md) | [Next: Kth Smallest Element →](03-kth-smallest-element.md)

---

## Problem

Given an unsorted array, find the Kth largest element.

**Example**: arr = [3, 2, 1, 5, 6, 4], K = 2 → Answer: **5**

---

## Approach: Min-Heap of Size K

```
FIND_KTH_LARGEST(arr, K):
    min_heap = new MinHeap()
    
    FOR each element in arr:
        IF min_heap.size < K:
            min_heap.insert(element)
        ELSE IF element > min_heap.peek():
            min_heap.extract_min()    // Remove smallest of top-K
            min_heap.insert(element)  // Add new candidate
    
    RETURN min_heap.peek()   // Root = Kth largest
```

---

## Step-by-Step Trace

```
arr = [3, 2, 1, 5, 6, 4], K = 2

Process 3: heap.size < 2 → insert → heap = [3]
Process 2: heap.size < 2 → insert → heap = [2, 3]

  Min-heap:    2
              /
             3

Process 1: 1 < peek(2) → skip
Process 5: 5 > peek(2) → extract 2, insert 5 → heap = [3, 5]

  Min-heap:    3
              /
             5

Process 6: 6 > peek(3) → extract 3, insert 6 → heap = [5, 6]

  Min-heap:    5
              /
             6

Process 4: 4 < peek(5) → skip

Answer: peek() = 5 ✅ (2nd largest)
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n log K) |
| Space | O(K) |

---

## Alternative: QuickSelect

QuickSelect gives O(n) average but O(n²) worst case. Heap approach gives **O(n log K) guaranteed**.

| Approach | Average | Worst | Space |
|----------|---------|-------|-------|
| Heap (size K) | O(n log K) | O(n log K) | O(K) |
| QuickSelect | O(n) | O(n²) | O(1) |
| Sort + index | O(n log n) | O(n log n) | O(1) |

---

## Quick Check

1. **Why does the min-heap root represent the Kth largest element?**
   <details><summary>Answer</summary>
   The min-heap of size K holds the K largest elements seen so far. The root is the **smallest** of those K elements — which by definition is the Kth largest overall.
   </details>

---

[← Previous: The Golden Rule](01-golden-rule.md) | [Next: Kth Smallest Element →](03-kth-smallest-element.md)
