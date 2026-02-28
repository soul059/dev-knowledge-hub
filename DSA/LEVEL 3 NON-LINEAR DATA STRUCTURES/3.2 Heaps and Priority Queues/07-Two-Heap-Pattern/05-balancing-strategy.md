# 7.5 Balancing Strategy & Variations

[← Previous: Maximize Capital](04-maximize-capital.md) | [Next: Pattern Recognition →](06-pattern-recognition.md)

---

## The Balancing Framework

Every two-heap problem requires maintaining these invariants:

```
INVARIANT 1: ORDERING
  max_heap.root ≤ min_heap.root
  (All elements in lower half ≤ all elements in upper half)

INVARIANT 2: SIZE BALANCE
  |max_heap.size - min_heap.size| ≤ 1
  (For median problems — other problems may have different balance rules)

REBALANCE PROCEDURE:
  IF max_heap too big:
      Move max_heap root → min_heap
  IF min_heap too big:
      Move min_heap root → max_heap
  IF max_heap.root > min_heap.root (ordering violated):
      Swap the roots between heaps
```

---

## Template Code

```python
def add_to_two_heaps(num, lo, hi):
    """lo = max-heap (negated), hi = min-heap"""
    
    # Add to appropriate heap
    if not lo or num <= -lo[0]:
        heapq.heappush(lo, -num)
    else:
        heapq.heappush(hi, num)
    
    # Rebalance
    if len(lo) > len(hi) + 1:
        val = -heapq.heappop(lo)
        heapq.heappush(hi, val)
    elif len(hi) > len(lo):
        val = heapq.heappop(hi)
        heapq.heappush(lo, -val)
```

---

## Variations and Extensions

### Running Percentile (e.g., 90th Percentile)

Instead of 50/50 split, maintain a different ratio:
- Max-heap stores bottom 90%
- Min-heap stores top 10%
- 90th percentile = max-heap root

```
Balance rule: max_heap.size ≈ 0.9 * total_size
```

### Weighted Median

Each element has a weight. Median is the value where cumulative weight crosses 50%.
- Track cumulative weight in each heap
- Rebalance when weight ratio drifts

---

## Two Heap vs Other Approaches

| Approach | Find Median | Insert | Remove |
|----------|-------------|--------|--------|
| **Two heaps** | **O(1)** | **O(log n)** | O(n) or O(log n) lazy |
| Sorted array | O(1) | O(n) | O(n) |
| BST (balanced) | O(log n) | O(log n) | O(log n) |
| Order-statistics tree | O(log n) | O(log n) | O(log n) |

Two heaps win when median queries are very frequent and removal is rare.

---

## Quick Check

1. **How would you modify the two-heap pattern to track the 75th percentile?**
   <details><summary>Answer</summary>
   Maintain the balance so max-heap has 75% of elements and min-heap has 25%. The max-heap root is the 75th percentile. Rebalance after each insert so that max_heap.size ≈ 0.75 * total and min_heap.size ≈ 0.25 * total.
   </details>

---

[← Previous: Maximize Capital](04-maximize-capital.md) | [Next: Pattern Recognition →](06-pattern-recognition.md)
