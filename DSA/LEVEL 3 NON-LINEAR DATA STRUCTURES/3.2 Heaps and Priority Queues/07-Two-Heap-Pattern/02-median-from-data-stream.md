# 7.2 Find Median from Data Stream

[← Previous: Core Idea](01-core-idea.md) | [Next: Sliding Window Median →](03-sliding-window-median.md)

---

## Problem

Design a data structure that supports:
- `addNum(num)`: Add a number from the data stream
- `findMedian()`: Return the median of all numbers so far

---

## Algorithm

```
class MedianFinder:
    max_heap = []    // Lower half (stores negated values for max behavior)
    min_heap = []    // Upper half
    
    ADD_NUM(num):
        // Step 1: Add to appropriate heap
        IF max_heap is empty OR num ≤ max_heap.peek():
            max_heap.insert(num)
        ELSE:
            min_heap.insert(num)
        
        // Step 2: Rebalance — sizes should differ by at most 1
        IF max_heap.size > min_heap.size + 1:
            min_heap.insert(max_heap.extract())
        ELSE IF min_heap.size > max_heap.size:
            max_heap.insert(min_heap.extract())
    
    FIND_MEDIAN():
        IF max_heap.size > min_heap.size:
            RETURN max_heap.peek()    // Odd count
        ELSE:
            RETURN (max_heap.peek() + min_heap.peek()) / 2.0  // Even count
```

---

## Step-by-Step Trace

```
Stream: 5, 15, 1, 3, 8, 7, 9

═══ Add 5 ═══
max_heap empty → add to max_heap
max_heap: [5]    min_heap: []
Median: 5

═══ Add 15 ═══
15 > peek(5) → add to min_heap
max_heap: [5]    min_heap: [15]
Sizes balanced (1, 1)
Median: (5 + 15) / 2 = 10.0

═══ Add 1 ═══
1 ≤ peek(5) → add to max_heap
max_heap: [5, 1]    min_heap: [15]
max_heap.size(2) > min_heap.size(1) + 1? No (2 = 1+1) → balanced

     MAX-HEAP         MIN-HEAP
        5     ≤         15
       /
      1

Median: 5 (odd count, max_heap root)

═══ Add 3 ═══
3 ≤ peek(5) → add to max_heap
max_heap: [5, 3, 1]    min_heap: [15]
max_heap.size(3) > min_heap.size(1) + 1? Yes!
→ Move 5 from max_heap to min_heap

max_heap: [3, 1]    min_heap: [5, 15]

     MAX-HEAP         MIN-HEAP
        3     ≤          5
       /                /
      1               15

Median: (3 + 5) / 2 = 4.0

═══ Add 8 ═══
8 > peek(3) → add to min_heap
max_heap: [3, 1]    min_heap: [5, 15, 8]
min_heap.size(3) > max_heap.size(2)? Yes!
→ Move 5 from min_heap to max_heap

max_heap: [5, 3, 1]    min_heap: [8, 15]

     MAX-HEAP         MIN-HEAP
        5     ≤          8
       / \              /
      3   1           15

Median: 5 (odd count)

═══ Add 7 ═══
7 > peek(5) → add to min_heap
max_heap: [5, 3, 1]    min_heap: [7, 15, 8]
Sizes balanced (3, 3)
Median: (5 + 7) / 2 = 6.0

Sorted so far: [1, 3, 5, 7, 8, 15] → median = (5+7)/2 = 6.0 ✅

═══ Add 9 ═══
9 > peek(5) → add to min_heap
max_heap: [5, 3, 1]    min_heap: [7, 9, 8, 15]
min_heap.size(4) > max_heap.size(3)? Yes!
→ Move 7 from min_heap to max_heap

max_heap: [7, 5, 3, 1]    min_heap: [8, 9, 15]

     MAX-HEAP         MIN-HEAP
        7     ≤          8
       / \              / \
      5   3            9  15
     /
    1

Median: 7 (odd count)
Sorted: [1, 3, 5, 7, 8, 9, 15] → median = 7 ✅
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| addNum | O(log n) | O(n) total |
| findMedian | O(1) | |

---

## Python Implementation

```python
import heapq

class MedianFinder:
    def __init__(self):
        self.lo = []  # max-heap (negated) — lower half
        self.hi = []  # min-heap — upper half
    
    def addNum(self, num):
        heapq.heappush(self.lo, -num)
        
        # Ensure ordering: max of lo ≤ min of hi
        if self.hi and (-self.lo[0] > self.hi[0]):
            val = -heapq.heappop(self.lo)
            heapq.heappush(self.hi, val)
        
        # Balance sizes: lo can have at most 1 more than hi
        if len(self.lo) > len(self.hi) + 1:
            val = -heapq.heappop(self.lo)
            heapq.heappush(self.hi, val)
        elif len(self.hi) > len(self.lo):
            val = heapq.heappop(self.hi)
            heapq.heappush(self.lo, -val)
    
    def findMedian(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2.0
```

---

[← Previous: Core Idea](01-core-idea.md) | [Next: Sliding Window Median →](03-sliding-window-median.md)
