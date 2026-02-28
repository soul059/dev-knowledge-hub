# 5.5 Standard Library Usage

[← Previous: Custom Comparators](04-custom-comparators.md) | [Next: Design Patterns →](06-design-patterns.md)

---

## Overview

All major languages provide built-in priority queue / heap implementations. Knowing their APIs, defaults, and pitfalls is essential for interviews and production code.

---

## Python — `heapq` Module

Python's `heapq` operates on a **regular list** (no wrapper class) and provides a **min-heap** only.

```python
import heapq

pq = []
heapq.heappush(pq, 5)
heapq.heappush(pq, 1)
heapq.heappush(pq, 3)

heapq.heappop(pq)         # → 1  (smallest)
pq[0]                     # → 3  (peek, no pop)
heapq.heapify([5, 1, 3])  # in-place, O(n)

# Useful utilities
heapq.nsmallest(2, pq)    # 2 smallest elements
heapq.nlargest(2, pq)     # 2 largest elements
heapq.heappushpop(pq, 4)  # push then pop (optimized)
heapq.heapreplace(pq, 4)  # pop then push (optimized)
```

### Max-Heap Workaround

```python
# Negate values
heapq.heappush(pq, -val)
max_val = -heapq.heappop(pq)
```

### Common Pitfalls — Python

| Pitfall | Fix |
|---------|-----|
| No built-in max-heap | Negate values |
| No decrease-key | Use lazy deletion or indexed wrapper |
| `heapq[0]` modifies nothing | Use `heappop()` to extract |
| Tuples with uncomparable items | Add counter as tie-breaker |

---

## Java — `PriorityQueue<E>`

Java provides a class-based min-heap with `Comparator` support.

```java
// Min-heap (default)
PriorityQueue<Integer> minPQ = new PriorityQueue<>();
minPQ.offer(5);
minPQ.offer(1);
minPQ.offer(3);
minPQ.poll();     // → 1
minPQ.peek();     // → 3

// Max-heap
PriorityQueue<Integer> maxPQ = new PriorityQueue<>(
    Collections.reverseOrder()
);

// Custom comparator
PriorityQueue<int[]> pq = new PriorityQueue<>(
    (a, b) -> a[0] - b[0]   // compare by first element
);
```

### Key Methods

| Method | Description |
|--------|-------------|
| `offer(e)` | Insert element |
| `poll()` | Remove & return min |
| `peek()` | View min without removing |
| `remove(e)` | Remove specific element — O(n) |
| `contains(e)` | Check membership — O(n) |
| `size()` | Number of elements |

### Common Pitfalls — Java

| Pitfall | Fix |
|---------|-----|
| `remove()` is O(n) | Avoid frequent arbitrary removals |
| Not thread-safe | Use `PriorityBlockingQueue` for concurrency |
| Iterator order ≠ sorted | Only `poll()` guarantees order |
| No decrease-key | Remove + re-insert, or use indexed PQ |

---

## C++ — `priority_queue`

C++ STL provides a **max-heap** by default.

```cpp
#include <queue>

// Max-heap (default)
priority_queue<int> maxPQ;
maxPQ.push(5);
maxPQ.push(1);
maxPQ.push(3);
maxPQ.top();       // → 5  (largest)
maxPQ.pop();       // removes 5

// Min-heap
priority_queue<int, vector<int>, greater<int>> minPQ;
minPQ.push(5);
minPQ.push(1);
minPQ.top();       // → 1  (smallest)
```

### Common Pitfalls — C++

| Pitfall | Fix |
|---------|-----|
| Default is max-heap (unlike Java/Python) | Use `greater<int>` for min |
| `top()` returns const reference | Copy before `pop()` |
| No `remove()` method | Use lazy deletion |
| No decrease-key | Lazy deletion + re-insert |

---

## Defaults Summary Table

| Language | Module/Class | Default Order | Max-Heap Method |
|----------|-------------|---------------|-----------------|
| Python | `heapq` | **Min** | Negate values |
| Java | `PriorityQueue` | **Min** | `Collections.reverseOrder()` |
| C++ | `priority_queue` | **Max** | Already max; use `greater<>` for min |
| C# | `PriorityQueue` (.NET 6+) | **Min** | Custom comparer |
| Go | `container/heap` | Custom (interface) | Implement `Less()` |

---

## Quick Check

1. **If you push [3, 1, 4, 1, 5] into a C++ `priority_queue<int>` and call `top()`, what do you get?**
   <details><summary>Answer</summary>
   **5** — C++ `priority_queue` is a max-heap by default.
   </details>

2. **In Python, how do you efficiently push a new element and pop the smallest in one operation?**
   <details><summary>Answer</summary>
   `heapq.heappushpop(heap, val)` — pushes val, then pops and returns the smallest. This is faster than separate push + pop because it avoids a redundant sift.
   </details>

---

[← Previous: Custom Comparators](04-custom-comparators.md) | [Next: Design Patterns →](06-design-patterns.md)
