# 5.6 Design Patterns with Priority Queues

[← Previous: Standard Library Usage](05-standard-library-usage.md) | [Next: Unit 6 — K-Element Golden Rule →](../06-K-Element-Problems/01-golden-rule.md)

---

## Lazy Deletion

When decrease-key or removal isn't supported (most standard libraries), use **lazy deletion**: mark entries as invalid and skip them on extraction.

```
Lazy Deletion Pattern:

  Push (priority=5, task="A")
  Push (priority=3, task="B")

  Need to update A's priority to 1:
    → Push (priority=1, task="A")   // new entry
    → Mark old entry as stale

  Heap now has:
    (1, "A")  ← valid
    (3, "B")  ← valid
    (5, "A")  ← STALE (skip when popped)

  Pop → (1, "A") → valid ✓ → process
  Pop → (3, "B") → valid ✓ → process
  Pop → (5, "A") → STALE ✗ → discard, pop again
```

### Pseudocode

```python
import heapq

class LazyPQ:
    def __init__(self):
        self.heap = []
        self.entry_finder = {}      # task → current entry
        self.counter = 0            # unique tie-breaker
        self.REMOVED = "<removed>"
    
    def push(self, task, priority):
        if task in self.entry_finder:
            self.remove(task)       # invalidate old entry
        entry = [priority, self.counter, task]
        self.counter += 1
        self.entry_finder[task] = entry
        heapq.heappush(self.heap, entry)
    
    def remove(self, task):
        entry = self.entry_finder.pop(task)
        entry[2] = self.REMOVED     # mark as stale
    
    def pop(self):
        while self.heap:
            priority, count, task = heapq.heappop(self.heap)
            if task is not self.REMOVED:
                del self.entry_finder[task]
                return task, priority
        raise KeyError("empty")
```

> **Trade-off**: Lazy deletion uses extra memory (stale entries remain in heap) but avoids the complexity of an indexed priority queue.

---

## Event-Driven Simulation

A priority queue of events ordered by timestamp drives simulations:

```
Event Queue (min-heap by time):

  (t=0.0,  "Customer 1 arrives")
  (t=1.5,  "Customer 2 arrives")
  (t=3.2,  "Customer 1 finishes service")

Algorithm:
  WHILE event_queue is not empty:
      event = EXTRACT_MIN(event_queue)
      process(event)
      // Processing may generate new events:
      new_events = simulate(event)
      FOR each new_event in new_events:
          INSERT(event_queue, new_event)
```

Applications: network packet simulation, CPU scheduling, Monte Carlo methods.

---

## Best-First Search

Many graph/search algorithms use a PQ to always expand the most promising node:

| Algorithm | Priority Key | Purpose |
|-----------|-------------|---------|
| Dijkstra | Distance from source | Shortest path |
| A* | f(n) = g(n) + h(n) | Shortest path with heuristic |
| Prim's MST | Edge weight to tree | Minimum spanning tree |
| Best-first search | Heuristic value | Explore most promising first |

```
BEST_FIRST_SEARCH(start, goal):
    pq = MinPriorityQueue()
    pq.INSERT( (heuristic(start), start) )
    visited = {}
    
    WHILE pq is not empty:
        (cost, node) = pq.EXTRACT_MIN()
        IF node == goal: RETURN cost
        IF node in visited: CONTINUE
        visited.add(node)
        FOR each neighbor of node:
            IF neighbor not in visited:
                pq.INSERT( (cost + weight(node, neighbor), neighbor) )
```

---

## Streaming Top-K

Maintain a **min-heap of size K** to track the K largest elements in a data stream:

```
Stream: 5, 12, 3, 8, 20, 1, 15    K = 3

Step 1: Push 5  → heap = [5]
Step 2: Push 12 → heap = [5, 12]
Step 3: Push 3  → heap = [3, 12, 5]     (size = K)
Step 4: 8 > min(3) → pop 3, push 8  → [5, 12, 8]
Step 5: 20 > min(5) → pop 5, push 20 → [8, 20, 12]
Step 6: 1 < min(8) → skip
Step 7: 15 > min(8) → pop 8, push 15 → [12, 20, 15]

Final top-3: {12, 15, 20} ✓
```

Each element takes O(log K) time → total O(n log K), O(K) space.

---

## Real-World Applications

| Domain | Pattern | Structure |
|--------|---------|-----------|
| OS kernel | Process scheduling | Max-PQ by priority |
| Network router | Packet scheduling | Multi-level PQ |
| Database | Query optimization | Cost-based PQ |
| Game engine | Event loop | Time-ordered PQ |
| Load balancer | Server selection | Min-PQ by load |

---

## Unit 5 Summary

| Topic | Key Insight |
|-------|-------------|
| PQ ADT | Abstract interface: insert + extract_min/max + peek |
| Min/Max PQ | Same mechanics, opposite comparisons |
| Update priority | Decrease/increase key + index map for O(log n) |
| Custom comparators | Tuples, `__lt__`, `Comparator` for complex ordering |
| Standard libraries | Know defaults (Python/Java = min, C++ = max) |
| Lazy deletion | Skip stale entries — simple alternative to indexed PQ |
| Design patterns | Event sim, best-first search, streaming top-K |

---

## Revision Questions

1. Implement a `MedianFinder` that supports `addNum(int)` and `findMedian()` using two heaps.
2. Design a task scheduler with priority and deadline using a custom comparator.
3. When is lazy deletion preferable to an indexed priority queue?
4. Write Dijkstra's algorithm using Python's `heapq` with lazy deletion.
5. How would you modify a PQ to support both min and max extraction efficiently?

---

[← Previous: Standard Library Usage](05-standard-library-usage.md) | [Next: Unit 6 — K-Element Golden Rule →](../06-K-Element-Problems/01-golden-rule.md)
