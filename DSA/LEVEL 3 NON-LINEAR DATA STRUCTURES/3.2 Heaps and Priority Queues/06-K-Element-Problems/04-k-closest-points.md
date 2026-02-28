# 6.4 K Closest Points to Origin

[← Previous: Kth Smallest & K Largest](03-kth-smallest-element.md) | [Next: Top K Frequent Elements →](05-top-k-frequent.md)

---

## Problem

Given n points in a 2D plane, find the K closest to the origin (0, 0).

**Distance**: √(x² + y²), but we can compare x² + y² (skip the square root — monotonic function preserves ordering).

---

## Approach: Max-Heap of Size K

We want K **smallest** distances → use **max-heap** of size K.
- The root = farthest among our K candidates
- When a closer point arrives, it evicts the farthest

```
K_CLOSEST_POINTS(points, K):
    max_heap = new MaxHeap()   // by distance
    
    FOR each point (x, y) in points:
        dist = x*x + y*y
        IF max_heap.size < K:
            max_heap.insert((dist, x, y))
        ELSE IF dist < max_heap.peek().dist:
            max_heap.extract_max()
            max_heap.insert((dist, x, y))
    
    RETURN max_heap.to_array()
```

---

## Trace

```
Points: [(1,3), (3,4), (2,-1)], K = 2

Distances: (1,3)→10, (3,4)→25, (2,-1)→5

Process (1,3), dist=10: size < 2 → insert
  Max-heap: [(10, 1, 3)]

Process (3,4), dist=25: size < 2 → insert
  Max-heap: [(25, 3, 4), (10, 1, 3)]
  
       25
      /
    10

Process (2,-1), dist=5: 5 < peek(25) → replace
  Extract (25, 3, 4), insert (5, 2, -1)
  Max-heap: [(10, 1, 3), (5, 2, -1)]

Result: [(1,3), (2,-1)] ✅
```

---

## Python Implementation

```python
import heapq

def k_closest(points, K):
    # Use max-heap via negation
    max_heap = []
    for x, y in points:
        dist = x*x + y*y
        if len(max_heap) < K:
            heapq.heappush(max_heap, (-dist, x, y))
        elif -dist > max_heap[0][0]:  # dist < farthest in heap
            heapq.heapreplace(max_heap, (-dist, x, y))
    
    return [(x, y) for _, x, y in max_heap]
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n log K) |
| Space | O(K) |

---

## Quick Check

1. **Why can we compare x²+y² instead of √(x²+y²)?**
   <details><summary>Answer</summary>
   The square root is a monotonically increasing function. If a² < b², then √a² < √b². So comparing squared distances gives the same ordering as comparing actual distances, while avoiding expensive floating-point sqrt operations.
   </details>

---

[← Previous: Kth Smallest & K Largest](03-kth-smallest-element.md) | [Next: Top K Frequent Elements →](05-top-k-frequent.md)
