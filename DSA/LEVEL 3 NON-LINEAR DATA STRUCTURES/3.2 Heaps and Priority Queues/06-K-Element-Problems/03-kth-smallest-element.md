# 6.3 Kth Smallest & K Largest Elements

[← Previous: Kth Largest Element](02-kth-largest-element.md) | [Next: K Closest Points →](04-k-closest-points.md)

---

## Kth Smallest Element

### Problem

Find the Kth smallest element in an unsorted array.

**Mirror** of Kth largest — use a **max-heap of size K**.

### Pseudocode

```
FIND_KTH_SMALLEST(arr, K):
    max_heap = new MaxHeap()
    
    FOR each element in arr:
        IF max_heap.size < K:
            max_heap.insert(element)
        ELSE IF element < max_heap.peek():
            max_heap.extract_max()
            max_heap.insert(element)
    
    RETURN max_heap.peek()   // Root = Kth smallest
```

### Trace

```
arr = [7, 10, 4, 3, 20, 15], K = 3

Process 7:  size < 3 → insert → [7]
Process 10: size < 3 → insert → [10, 7]
Process 4:  size < 3 → insert → [10, 7, 4]

  Max-heap:     10
               /  \
              7    4

Process 3: 3 < peek(10) → extract 10, insert 3 → [7, 3, 4]

  Max-heap:      7
               /  \
              3    4

Process 20: 20 > peek(7) → skip
Process 15: 15 > peek(7) → skip

Answer: peek() = 7 ✅ (3rd smallest: 3, 4, 7)
```

---

## K Largest Elements

### Problem

Return the K largest elements (not just the Kth).

### Approach

Same as Kth largest, but return the **entire heap** at the end.

```
FIND_K_LARGEST(arr, K):
    min_heap = new MinHeap()
    
    FOR each element in arr:
        IF min_heap.size < K:
            min_heap.insert(element)
        ELSE IF element > min_heap.peek():
            min_heap.extract_min()
            min_heap.insert(element)
    
    RETURN min_heap.to_array()   // All K elements
```

### Visualization

```
arr = [1, 23, 12, 9, 30, 2, 50], K = 3

Step by step:
  After 1:     [1]
  After 23:    [1, 23]
  After 12:    [1, 23, 12]      ← heap full (size K=3)

  Min-heap:      1
                / \
              23   12

  After 9:   9 > 1 → replace 1 → [9, 23, 12]

  Min-heap:      9
                / \
              23   12

  After 30:  30 > 9 → replace 9 → [12, 23, 30]

  Min-heap:     12
               / \
             23   30

  After 2:   2 < 12 → skip
  After 50:  50 > 12 → replace 12 → [23, 50, 30]

  Min-heap:     23
               / \
             50   30

Result: {23, 30, 50} ← the 3 largest ✅
```

---

## Quick Check

1. **For K smallest elements, why do we use a max-heap instead of a min-heap?**
   <details><summary>Answer</summary>
   A max-heap of size K keeps the root as the largest among the K smallest elements (the gatekeeper). Any new element smaller than the root can replace it. This maintains exactly K smallest elements using only O(K) space.
   </details>

---

[← Previous: Kth Largest Element](02-kth-largest-element.md) | [Next: K Closest Points →](04-k-closest-points.md)
