# Chapter 2: Top-K Problems

## 📋 Chapter Overview
Using heaps to efficiently solve "K largest", "K smallest", "K most frequent", and similar top-K problems.

---

## 🧠 Core Pattern

```
  Top-K Largest → use MIN HEAP of size K
    • If new element > heap.peek(), replace
    • Heap root = Kth largest
  
  Top-K Smallest → use MAX HEAP of size K
    • If new element < heap.peek(), replace
    • Heap root = Kth smallest
  
  Why opposite heap? We want to EVICT the worst candidate.
  Min heap of K: root is smallest among K largest → evict smallest.
```

---

## 📝 Problem 1: Kth Largest Element (LeetCode 215)

### Approach 1: Min Heap of Size K

```
function findKthLargest(nums, k):
    minHeap of size k
    
    for each num in nums:
        if heap.size < k:
            heap.push(num)
        elif num > heap.peek():
            heap.poll()
            heap.push(num)
    
    return heap.peek()   // Kth largest
```

### Trace

```
  nums = [3,2,1,5,6,4], k = 2
  
  3: heap = [3]             (size < 2)
  2: heap = [2, 3]          (size == 2)
  1: 1 ≤ peek(2) → skip
  5: 5 > 2 → poll 2, push 5. heap = [3, 5]
  6: 6 > 3 → poll 3, push 6. heap = [5, 6]
  4: 4 ≤ 5 → skip
  
  Answer: peek() = 5 (2nd largest)
```

**Complexity:** O(n log k) time, O(k) space

### Approach 2: QuickSelect (Average O(n))

```
function quickSelect(nums, k):
    target = n - k   // index of kth largest in sorted array
    left, right = 0, n-1
    
    while left < right:
        pivot = partition(nums, left, right)
        if pivot == target: return nums[pivot]
        elif pivot < target: left = pivot + 1
        else: right = pivot - 1
    
    return nums[left]
```

Quickselect: O(n) average, O(n²) worst. Heap approach: O(n log k) guaranteed.

---

## 📝 Problem 2: Top K Frequent Elements (LeetCode 347)

```
function topKFrequent(nums, k):
    freqMap = count frequency of each number
    minHeap = []   // (frequency, number)
    
    for each (num, freq) in freqMap:
        heap.push((freq, num))
        if heap.size > k:
            heap.poll()   // remove least frequent among top-k
    
    return [num for (freq, num) in heap]
```

### Trace

```
  nums = [1,1,1,2,2,3], k = 2
  freqMap: {1:3, 2:2, 3:1}
  
  (3,1): heap = [(3,1)]
  (2,2): heap = [(2,2),(3,1)]
  (1,3): heap push → [(1,3),(3,1),(2,2)]. size=3 > k=2
         poll min → remove (1,3). heap = [(2,2),(3,1)]
  
  Answer: [2, 1]
```

### Alternative: Bucket Sort O(n)

```
function topKFrequentBucket(nums, k):
    freqMap = count frequencies
    buckets = array of n+1 empty lists
    
    for each (num, freq):
        buckets[freq].add(num)
    
    result = []
    for freq = n down to 1:
        for num in buckets[freq]:
            result.add(num)
            if result.size == k: return result
```

---

## 📝 Problem 3: K Closest Points to Origin (LeetCode 973)

```
function kClosest(points, k):
    maxHeap = []  // (-distance², point) — negate for max heap
    
    for each point [x, y]:
        dist = x*x + y*y
        heap.push((-dist, point))
        if heap.size > k:
            heap.poll()   // remove farthest among k closest
    
    return [point for (_, point) in heap]
```

**Optimization:** Use squared distance — no need for sqrt.

**Complexity:** O(n log k) time, O(k) space

---

## 📝 Problem 4: Kth Smallest in Sorted Matrix (LeetCode 378)

```
  Matrix (sorted rows and columns):
  [ 1,  5,  9]
  [10, 11, 13]
  [12, 13, 15]
  
function kthSmallest(matrix, k):
    minHeap = []
    n = len(matrix)
    
    // Add first element of each row
    for i = 0 to min(k, n) - 1:
        heap.push((matrix[i][0], i, 0))  // (value, row, col)
    
    // Extract k-1 times and expand
    for _ = 0 to k-2:
        (val, row, col) = heap.poll()
        if col + 1 < n:
            heap.push((matrix[row][col+1], row, col+1))
    
    return heap.peek().val
```

### Trace

```
  k = 3
  
  Init: heap = [(1,0,0), (10,1,0), (12,2,0)]
  
  Poll (1,0,0). Push (5,0,1). heap = [(5,0,1),(10,1,0),(12,2,0)]
  Poll (5,0,1). Push (9,0,2). heap = [(9,0,2),(10,1,0),(12,2,0)]
  
  Peek: 9. Answer: 9 (3rd smallest)
```

---

## 📊 Top-K Strategy Guide

| Want | Heap Type | Size | Why |
|------|----------|------|-----|
| K largest | Min heap | K | Root = Kth largest; evict smallest |
| K smallest | Max heap | K | Root = Kth smallest; evict largest |
| K most frequent | Min heap (by freq) | K | Same as K largest on frequencies |
| K closest | Max heap (by dist) | K | Evict farthest |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Opposite heap | Use min heap for K largest, max heap for K smallest |
| Size-K heap | Maintain only K elements — O(n log k) |
| QuickSelect | O(n) average alternative for single Kth element |
| Bucket sort | O(n) for K most frequent when range is bounded |
| Sorted matrix | BFS-like expansion with min heap |

---

## ❓ Revision Questions

1. **Why min heap for K largest?** → Min heap root is the smallest among K largest elements; new elements replace it only if they're bigger.
2. **Time complexity of heap approach vs sorting?** → Heap: O(n log k). Sort: O(n log n). When k << n, heap is significantly faster.
3. **QuickSelect advantage/disadvantage?** → O(n) average but O(n²) worst case; modifies input array. Heap: O(n log k) guaranteed.
4. **K closest points: why squared distance?** → Avoids expensive sqrt; comparison order is the same since sqrt is monotonic.
5. **Sorted matrix: why BFS-like?** → Start from smallest elements (first column); expand to the right, always extracting the global minimum via heap.

---

[← Previous: Heap Fundamentals](01-heap-fundamentals.md) | [Next: Two-Heap Pattern →](03-two-heap-pattern.md)
