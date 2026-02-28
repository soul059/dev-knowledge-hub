# 10.4 Indexed Priority Queue

[← Previous: Binomial Heap](03-binomial-heap.md) | [Next: Double-Ended Priority Queue →](05-double-ended-priority-queue.md)

---

## The Problem

Standard heaps can find the min/max in O(1) but **can't locate an arbitrary element** efficiently. Operations like "decrease the priority of element X" require O(n) search.

---

## Solution: Add an Index Map

```
Standard Heap:
  "Where is element X?" → Must scan entire array → O(n)

Indexed Priority Queue:
  "Where is element X?" → Look up in hash map → O(1)
  Then sift up/down → O(log n)
  Total: O(log n) for decrease-key!
```

---

## Data Structures

```
┌─────────────────────────────────────────────┐
│ Indexed Priority Queue                      │
│                                             │
│ 1. heap[]     — the heap array              │
│ 2. pos[key]   — maps key → index in heap    │
│ 3. keys present in the heap                 │
│                                             │
│ INVARIANT: pos[heap[i].key] = i for all i   │
│            heap[pos[key]].key = key          │
└─────────────────────────────────────────────┘
```

---

## Visual Example

```
Heap (min by priority):
Index:  [0]    [1]    [2]    [3]    [4]
Value: (A,3)  (C,5)  (B,7)  (D,8)  (E,10)

Position Map:
  A → 0
  B → 2
  C → 1
  D → 3
  E → 4

"Decrease key of B from 7 to 1":
  1. pos[B] = 2 → O(1)
  2. heap[2].priority = 1
  3. Sift up from index 2 → O(log n)
  
After: heap = [(B,1), (C,5), (A,3), (D,8), (E,10)]
Update pos: A→2, B→0, C→1
```

---

## Key Operations

```
INSERT(key, priority):
    heap.append((key, priority))
    pos[key] = heap.size - 1
    sift_up(heap.size - 1)          // Update pos during swaps!

DECREASE_KEY(key, new_priority):
    i = pos[key]                    // O(1) lookup
    heap[i].priority = new_priority
    sift_up(i)                      // O(log n)

EXTRACT_MIN():
    min_key = heap[0].key
    swap(0, heap.size - 1)          // Update pos for both!
    pos.remove(min_key)
    heap.remove_last()
    sift_down(0)
    RETURN min_key

CONTAINS(key):
    RETURN key in pos               // O(1)
```

---

## Critical: Update Position Map on Every Swap!

```
SWAP(i, j):
    pos[heap[i].key] = j
    pos[heap[j].key] = i
    heap[i], heap[j] = heap[j], heap[i]
```

---

## Complexity

| Operation | Standard Heap | **Indexed PQ** |
|-----------|--------------|----------------|
| Insert | O(log n) | O(log n) |
| Extract Min | O(log n) | O(log n) |
| Peek | O(1) | O(1) |
| Contains | O(n) | **O(1)** |
| **Decrease Key** | **O(n)** | **O(log n)** |
| **Delete Arbitrary** | **O(n)** | **O(log n)** |

---

## Primary Use Case: Dijkstra's Algorithm

```
DIJKSTRA_WITH_IPQ(graph, source):
    ipq = new IndexedPriorityQueue()
    ipq.insert(source, 0)
    
    WHILE ipq is not empty:
        (u, dist_u) = ipq.extract_min()
        
        FOR each neighbor v of u:
            new_dist = dist_u + weight(u, v)
            
            IF v not in ipq:
                ipq.insert(v, new_dist)
            ELSE IF new_dist < ipq.get_priority(v):
                ipq.decrease_key(v, new_dist)    // O(log n)!
```

---

[← Previous: Binomial Heap](03-binomial-heap.md) | [Next: Double-Ended Priority Queue →](05-double-ended-priority-queue.md)
