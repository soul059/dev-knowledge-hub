# 5.3 Update Priority (Decrease / Increase Key)

[← Previous: Min & Max Priority Queue](02-min-and-max-priority-queue.md) | [Next: Custom Comparators →](04-custom-comparators.md)

---

## Why Update Priority?

Many algorithms need to **change the priority** of an element already in the queue:
- **Dijkstra's Algorithm**: When a shorter path to a vertex is found, decrease its distance key
- **Prim's MST**: When a cheaper edge to a vertex is found, decrease its weight key
- **Process Scheduling**: A process's priority may change based on system conditions

---

## Decrease-Key (Min-Heap)

Lowering an element's value may violate the min-heap property with its parent → **sift up**.

```
Decrease-Key Example (Min-Heap):
Priority queue: [2, 5, 3, 8, 7, 6]

      2
     / \
    5   3
   / \ /
  8  7 6

Decrease 8 → 1 (index 3):

      2              2              1
     / \            / \            / \
    5   3   →      1   3   →     2   3
   / \ /          / \ /          / \ /
  1  7 6         5  7 6         5  7 6

Step 1: Set heap[3] = 1
Step 2: Sift up: 1 < 5 → swap → 1 < 2 → swap → root reached
```

### Pseudocode

```
DECREASE_KEY(heap, index, new_value):
    IF new_value > heap[index]:
        ERROR "New value is larger"
    heap[index] = new_value
    SIFT_UP(heap, index)       // Restore heap property upward
```

---

## Increase-Key (Max-Heap)

Raising an element's value may violate the max-heap property with its parent → **sift up**.

```
INCREASE_KEY(heap, index, new_value):
    IF new_value < heap[index]:
        ERROR "New value is smaller"
    heap[index] = new_value
    SIFT_UP(heap, index)       // Restore heap property upward
```

For a **min-heap** increase-key, the element may need to **sift down** instead:

```
INCREASE_KEY_MIN_HEAP(heap, index, new_value):
    IF new_value < heap[index]:
        ERROR "New value is smaller"
    heap[index] = new_value
    SIFT_DOWN(heap, index)     // Restore heap property downward
```

---

## The Index Lookup Problem

The heap stores elements by position, not by name. How do we find an element's index?

### Naive Approach: Linear Search — O(n)

```
UPDATE_PRIORITY_NAIVE(heap, element, new_priority):
    FOR i = 0 TO heap.size - 1:
        IF heap[i].id == element:
            old = heap[i].priority
            heap[i].priority = new_priority
            IF new_priority < old: SIFT_UP(heap, i)
            ELSE: SIFT_DOWN(heap, i)
            RETURN
```

### Optimized Approach: Index Map — O(1) lookup

Maintain a **hash map** from element ID → heap index:

```
class IndexedPriorityQueue:
    heap = []               // Array of (priority, element_id)
    index_map = {}          // element_id → position in heap

    SWAP(i, j):
        index_map[heap[i].id] = j
        index_map[heap[j].id] = i
        heap[i], heap[j] = heap[j], heap[i]

    INSERT(id, priority):
        heap.append( (priority, id) )
        index_map[id] = heap.size - 1
        SIFT_UP(heap, heap.size - 1)

    UPDATE(id, new_priority):
        i = index_map[id]       // O(1) lookup!
        old = heap[i].priority
        heap[i].priority = new_priority
        IF new_priority < old: SIFT_UP(heap, i)
        ELSE: SIFT_DOWN(heap, i)
```

### Complexity Comparison

| Operation | Without Index Map | With Index Map |
|-----------|-------------------|----------------|
| Find element | O(n) | O(1) |
| Update priority | O(n + log n) = O(n) | O(log n) |
| Insert | O(log n) | O(log n) |
| Extra space | — | O(n) |

---

## Quick Check

1. **After decrease-key on a min-heap, do we sift up or down?**
   <details><summary>Answer</summary>
   Sift **up**. The element became smaller, so it may now be less than its parent.
   </details>

2. **Why is the index map crucial for Dijkstra's algorithm?**
   <details><summary>Answer</summary>
   Dijkstra calls decrease-key frequently (up to O(E) times). Without the index map, each call is O(V) to find the element, making the total O(VE). With the map, each call is O(log V), yielding O((V + E) log V).
   </details>

---

[← Previous: Min & Max Priority Queue](02-min-and-max-priority-queue.md) | [Next: Custom Comparators →](04-custom-comparators.md)
