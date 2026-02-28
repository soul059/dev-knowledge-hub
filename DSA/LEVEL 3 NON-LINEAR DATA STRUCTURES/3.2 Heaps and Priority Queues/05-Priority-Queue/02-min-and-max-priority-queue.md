# 5.2 Min & Max Priority Queue

[← Previous: Priority Queue ADT](01-priority-queue-adt.md) | [Next: Update Priority →](03-update-priority.md)

---

## Overview

Priority queues come in two flavors: **min-PQ** (smallest first) and **max-PQ** (largest first). Both use the same heap mechanics with opposite comparison directions.

---

## Min-Priority Queue

The element with the **smallest** key/priority is extracted first.

```
Min-Priority Queue State:

  Insert(5), Insert(1), Insert(3), Insert(2)

  Internal Min-Heap:
       1
      / \
     2   3
    /
   5

  peek() → 1
  extract_min() → 1, heap becomes:
       2
      / \
     5   3
```

### Pseudocode

```
class MinPriorityQueue:
    heap = []
    
    INSERT(value):
        heap.append(value)
        SIFT_UP(heap, heap.size - 1)
    
    EXTRACT_MIN():
        IF heap is empty: ERROR
        min_val = heap[0]
        heap[0] = heap[last]
        heap.remove_last()
        SIFT_DOWN(heap, 0)
        RETURN min_val
    
    PEEK():
        RETURN heap[0]
```

### Common Use Cases

| Use Case | Why Min-PQ? |
|----------|-------------|
| Dijkstra's algorithm | Always process nearest unvisited node |
| Prim's MST | Always pick cheapest edge |
| Event simulation | Process earliest event first |
| Huffman coding | Combine two lowest frequencies |
| Task scheduling (shortest first) | Process shortest task next |

---

## Max-Priority Queue

The element with the **largest** key/priority is extracted first.

```
Max-Priority Queue State:

  Insert(5), Insert(1), Insert(3), Insert(2)

  Internal Max-Heap:
       5
      / \
     2   3
    /
   1

  peek() → 5
  extract_max() → 5, heap becomes:
       3
      / \
     2   1
```

### Common Use Cases

| Use Case | Why Max-PQ? |
|----------|-------------|
| Job scheduling by priority | Highest priority job first |
| Bandwidth management | Highest priority packet first |
| Kth largest element | Maintain heap of candidates |
| Merge K sorted (descending) | Always extend the largest |

---

## Quick Check

1. **In Dijkstra's algorithm, why must we use a min-priority queue?**
   <details><summary>Answer</summary>
   Dijkstra processes vertices in order of increasing distance from the source. Extracting the minimum-distance vertex ensures that when we process a vertex, we've already found the shortest path to it.
   </details>

---

[← Previous: Priority Queue ADT](01-priority-queue-adt.md) | [Next: Update Priority →](03-update-priority.md)
