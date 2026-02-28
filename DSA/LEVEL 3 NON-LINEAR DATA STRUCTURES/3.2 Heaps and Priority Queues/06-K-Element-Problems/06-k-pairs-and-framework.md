# 6.6 K Pairs with Smallest Sums & Problem-Solving Framework

[← Previous: Top K Frequent Elements](05-top-k-frequent.md) | [Next: Unit 7 — Two Heap Core Idea →](../07-Two-Heap-Pattern/01-core-idea.md)

---

## K Pairs with Smallest Sums

### Problem

Given two sorted arrays `nums1` and `nums2`, find K pairs (u, v) with the smallest sums, where u ∈ nums1 and v ∈ nums2.

**Example**: nums1 = [1,7,11], nums2 = [2,4,6], K = 3 → [(1,2), (1,4), (1,6)]

### Key Insight

Since both arrays are sorted, the smallest possible pair is `(nums1[0], nums2[0])`. The next candidates are `(nums1[1], nums2[0])` and `(nums1[0], nums2[1])`.

```
        nums2 →
        2    4    6
  n  1 [1+2, 1+4, 1+6]     = [3, 5, 7]
  u  7 [7+2, 7+4, 7+6]     = [9, 11, 13]
  m 11 [11+2,11+4,11+6]    = [13, 15, 17]
  s
  1
  ↓
  
  Start at top-left, expand like BFS
```

### Approach: Min-Heap with Expansion

```
K_SMALLEST_PAIRS(nums1, nums2, K):
    min_heap = []
    result = []
    
    // Initialize: pair each nums1[i] with nums2[0]
    FOR i = 0 TO min(K, len(nums1)) - 1:
        min_heap.insert((nums1[i] + nums2[0], i, 0))
    
    WHILE K > 0 AND min_heap not empty:
        (sum, i, j) = min_heap.extract_min()
        result.append((nums1[i], nums2[j]))
        K -= 1
        
        // Expand: try next element in nums2
        IF j + 1 < len(nums2):
            min_heap.insert((nums1[i] + nums2[j+1], i, j+1))
    
    RETURN result
```

### Trace

```
nums1 = [1, 7, 11], nums2 = [2, 4, 6], K = 3

Initialize heap with (nums1[i] + nums2[0]):
  (1+2, 0, 0) = (3, 0, 0)
  (7+2, 1, 0) = (9, 1, 0)
  (11+2, 2, 0) = (13, 2, 0)

Heap: [(3,0,0), (9,1,0), (13,2,0)]

── Extract (3, 0, 0) → pair (1, 2) ──
  Expand: j+1=1 < 3 → insert (1+4, 0, 1) = (5, 0, 1)
  Heap: [(5,0,1), (9,1,0), (13,2,0)]
  Result: [(1,2)]

── Extract (5, 0, 1) → pair (1, 4) ──
  Expand: j+1=2 < 3 → insert (1+6, 0, 2) = (7, 0, 2)
  Heap: [(7,0,2), (9,1,0), (13,2,0)]
  Result: [(1,2), (1,4)]

── Extract (7, 0, 2) → pair (1, 6) ──
  Expand: j+1=3, not < 3 → no insert
  Heap: [(9,1,0), (13,2,0)]
  Result: [(1,2), (1,4), (1,6)] ✅
```

### Complexity

| Metric | Value |
|--------|-------|
| Time | O(K log K) (heap never exceeds size min(K, n)) |
| Space | O(K) |

---

## Problem-Solving Framework for K-Element Problems

### Decision Tree

```
          K-Element Problem
                |
    ┌───────────┴───────────┐
    Need K              Need Kth
    elements?            element?
    │                    │
    ├─ K largest         ├─ Kth largest
    │  → Min-heap(K)     │  → Min-heap(K), return peek
    │                    │
    ├─ K smallest        ├─ Kth smallest
    │  → Max-heap(K)     │  → Max-heap(K), return peek
    │                    │
    ├─ K closest/nearest │
    │  → Max-heap(K)     │
    │    by distance      │
    │                    │
    └─ K most frequent   │
       → Count + Min-heap(K) by freq
```

### Pattern Template

```python
def k_element_pattern(elements, K, comparator):
    heap = []
    
    for elem in elements:
        key = extract_key(elem)  # distance, frequency, value, etc.
        
        if len(heap) < K:
            heapq.heappush(heap, (key, elem))
        elif should_replace(key, heap[0][0]):
            heapq.heapreplace(heap, (key, elem))
    
    return [elem for _, elem in heap]
```

---

## Real-World Applications

| Application | K-Element Pattern Used |
|-------------|----------------------|
| **Search engine** | Top K relevant results by score |
| **Recommendation engine** | K nearest neighbors (KNN) |
| **Network monitoring** | Top K bandwidth consumers |
| **Database** | ORDER BY ... LIMIT K |
| **Social media** | Top K trending topics by frequency |
| **Gaming** | Top K leaderboard scores |

---

## Unit 6 Summary Table

| Problem | Heap Type | Size | Complexity |
|---------|-----------|------|------------|
| Kth largest | Min-heap | K | O(n log K) |
| Kth smallest | Max-heap | K | O(n log K) |
| K largest elements | Min-heap | K | O(n log K) |
| K closest points | Max-heap | K | O(n log K) |
| Top K frequent | Min-heap (by freq) | K | O(n + m log K) |
| K smallest pairs | Min-heap + expansion | ≤ K | O(K log K) |
| **Key Insight** | For "K largest" → min-heap; for "K smallest" → max-heap |

---

## Revision Questions

1. **You have 1 billion numbers streaming in. Find the top 100 largest. What's the time and space complexity?**
   <details><summary>Answer</summary>
   Use a min-heap of size 100. Time: O(n log 100) = O(n) effectively constant log factor. Space: O(100) = O(1). You process each of the 1 billion numbers once, each taking at most O(log 100) ≈ O(7) operations.
   </details>

2. **For "Top K frequent elements", why is the total time O(n + m log K) and not O(n log K)?**
   <details><summary>Answer</summary>
   O(n) is for building the frequency map (scanning all elements). The heap operations are O(m log K) where m is the number of unique elements. Since m ≤ n, the total is O(n + m log K).
   </details>

3. **In "K pairs with smallest sums", why do we only initialize the heap with pairs involving nums2[0]?**
   <details><summary>Answer</summary>
   Since both arrays are sorted, the K smallest sums must start from the beginning. We initialize with (nums1[i], nums2[0]) for each i, then expand by incrementing the j index. This limits the heap to at most min(K, len(nums1)) entries.
   </details>

4. **What's the advantage of `heapq.heapreplace()` over separate extract + insert?**
   <details><summary>Answer</summary>
   `heapreplace()` combines extract-min and insert into a single operation with one sift-down, saving roughly one O(log K) operation.
   </details>

---

[← Previous: Top K Frequent Elements](05-top-k-frequent.md) | [Next: Unit 7 — Two Heap Core Idea →](../07-Two-Heap-Pattern/01-core-idea.md)
