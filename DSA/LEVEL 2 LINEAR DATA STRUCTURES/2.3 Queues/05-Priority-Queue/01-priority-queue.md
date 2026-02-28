# Unit 5: Priority Queue

[← Previous: Deque](../04-Deque/01-deque.md) | [Back to README](../README.md) | [Next: Queue Problems →](../06-Queue-Problems/01-queue-problems.md)

---

## Chapter Overview

A **priority queue** breaks the FIFO rule — elements are not served in arrival order but in **priority order**. This unit covers the concept, implementation approaches, min vs max variants, the heap-based implementation, and applications in classical algorithms.

---

## 1. Priority Queue Concept

### What is a Priority Queue?

A **priority queue** is an abstract data type where each element has an associated **priority**. The element with the highest (or lowest) priority is dequeued first, regardless of when it was enqueued.

```
  REGULAR QUEUE (FIFO):
  ┌────┬────┬────┬────┐
  │ D  │ C  │ B  │ A  │    → Dequeue order: A, B, C, D
  └────┴────┴────┴────┘      (arrival order)

  PRIORITY QUEUE:
  ┌──────────┬──────────┬──────────┬──────────┐
  │ D (p=1)  │ C (p=5)  │ B (p=2)  │ A (p=3)  │
  └──────────┴──────────┴──────────┴──────────┘
      ↓
  Dequeue order: C(5), A(3), B(2), D(1)
  (priority order, NOT arrival order)
```

### Key Properties

| Property | Regular Queue | Priority Queue |
|----------|:------------:|:--------------:|
| Order | Arrival (FIFO) | Priority value |
| Dequeue rule | Oldest first | Highest priority first |
| Equal priority | — | FIFO among equal (usually) |
| Underlying structure | Array/LL | Heap (typically) |

### ADT Operations

| Operation | Description | Typical Time |
|-----------|-------------|:------------:|
| `insert(item, priority)` | Add element with priority | O(log n) |
| `extractMax()` / `extractMin()` | Remove highest/lowest priority | O(log n) |
| `peek()` | View highest/lowest priority element | O(1) |
| `isEmpty()` | Check if PQ is empty | O(1) |
| `changePriority(item, newP)` | Update priority of existing item | O(log n) |
| `size()` | Return number of elements | O(1) |

---

## 2. Implementation Approaches

### Approach 1: Unsorted Array

```
  Insert: Just add to end → O(1)
  Extract: Scan entire array for max/min → O(n)

  ┌────┬────┬────┬────┬────┐
  │ 3  │ 7  │ 1  │ 5  │ 2  │     Insert(4): just append → O(1)
  └────┴────┴────┴────┴────┘     ExtractMax: scan for 7 → O(n)
```

### Approach 2: Sorted Array

```
  Insert: Find correct position + shift → O(n)
  Extract: Remove from end (max) or front (min) → O(1)

  ┌────┬────┬────┬────┬────┐
  │ 1  │ 2  │ 3  │ 5  │ 7  │     ExtractMax: remove 7 → O(1)
  └────┴────┴────┴────┴────┘     Insert(4): find spot + shift → O(n)
```

### Approach 3: Unsorted Linked List

```
  Insert: Add at head → O(1)
  Extract: Scan for max/min → O(n)

  HEAD → [3] → [7] → [1] → [5] → NULL
```

### Approach 4: Sorted Linked List

```
  Insert: Walk to correct position → O(n)
  Extract: Remove head (min-PQ) or tail (max-PQ) → O(1)

  HEAD → [1] → [3] → [5] → [7] → NULL
  ExtractMin: remove head → O(1)
```

### Approach 5: Binary Heap (Optimal)

```
  Insert: Add at end + bubble up → O(log n)
  Extract: Remove root + heapify down → O(log n)
  Peek: Return root → O(1)

  Best of both worlds: O(log n) for both operations!
```

### Complexity Comparison

| Implementation | Insert | Extract (Max/Min) | Peek | Space |
|---------------|:------:|:------------------:|:----:|:-----:|
| Unsorted Array | **O(1)** | O(n) | O(n) | O(n) |
| Sorted Array | O(n) | **O(1)** | **O(1)** | O(n) |
| Unsorted Linked List | **O(1)** | O(n) | O(n) | O(n) |
| Sorted Linked List | O(n) | **O(1)** | **O(1)** | O(n) |
| **Binary Heap** | **O(log n)** | **O(log n)** | **O(1)** | O(n) |

> **Verdict:** Binary heap gives the best balanced performance and is the standard implementation.

---

## 3. Min-Priority vs Max-Priority

### Max-Priority Queue

- The element with the **largest** priority value is dequeued first.
- Implemented with a **max-heap**.

```
  Example: Task scheduling by urgency

  Insert: (Task A, 3), (Task B, 7), (Task C, 1), (Task D, 5)

  Extract order: B(7), D(5), A(3), C(1)
                 Highest priority first

  Max-Heap visualization:
          7
        /   \
       5     1
      /
     3
```

### Min-Priority Queue

- The element with the **smallest** priority value is dequeued first.
- Implemented with a **min-heap**.

```
  Example: Dijkstra's shortest path (extract closest node)

  Insert: (Node A, 10), (Node B, 3), (Node C, 8), (Node D, 1)

  Extract order: D(1), B(3), C(8), A(10)
                 Smallest distance first

  Min-Heap visualization:
          1
        /   \
       3     8
      /
     10
```

### Converting Between Min and Max

```
  Trick: Negate all priorities!

  Max-PQ with min-heap:
    Insert (item, priority) → minHeap.insert(item, -priority)
    extractMax() → minHeap.extractMin()

  Example:
    Priorities: 3, 7, 1, 5
    Negated:   -3, -7, -1, -5

    Min-heap extractMin → -7 (which represents original 7 = MAX)
```

### Comparison

| Feature | Max-Priority Queue | Min-Priority Queue |
|---------|:------------------:|:------------------:|
| Dequeue order | Largest first | Smallest first |
| Heap type | Max-heap | Min-heap |
| `peek()` returns | Maximum element | Minimum element |
| Common use | Task scheduling | Dijkstra's, Huffman |

---

## 4. Use Cases

### Real-World Priority Queue Scenarios

```
  ┌───────────────────────────────────────────────┐
  │              HOSPITAL ER                       │
  │                                                │
  │  Patient Queue (Max-Priority by severity):     │
  │                                                │
  │  ┌────────────────┬──────────┐                │
  │  │ Patient        │ Priority │                │
  │  ├────────────────┼──────────┤                │
  │  │ Heart Attack   │    10    │ ← Served first │
  │  │ Broken Arm     │     5    │                │
  │  │ Headache       │     2    │                │
  │  │ Flu            │     1    │ ← Served last  │
  │  └────────────────┴──────────┘                │
  └───────────────────────────────────────────────┘
```

| Scenario | Priority Based On | Min or Max? |
|----------|-------------------|:-----------:|
| Hospital ER | Severity level | Max |
| Dijkstra's algorithm | Distance from source | Min |
| Huffman coding | Character frequency | Min |
| Job scheduler | Job priority/deadline | Max |
| Event simulation | Event timestamp | Min |
| A* pathfinding | f(n) = g(n) + h(n) | Min |
| Bandwidth allocation | Customer tier | Max |
| Prim's MST | Edge weight | Min |

---

## 5. Heap-Based Implementation (Intro)

A **binary heap** is a **complete binary tree** stored in an array with the **heap property**.

### Heap Properties

```
  MAX-HEAP: Parent ≥ Children (root is maximum)
  MIN-HEAP: Parent ≤ Children (root is minimum)

  Max-Heap Example:
              50
            /    \
          30      40
         /  \    /
        10   20  15

  Array representation (0-indexed):
  Index:  0    1    2    3    4    5
        [50] [30] [40] [10] [20] [15]
```

### Array Index Formulas (0-indexed)

| Relation | Formula |
|----------|---------|
| Parent of `i` | `(i - 1) / 2` (integer division) |
| Left child of `i` | `2 * i + 1` |
| Right child of `i` | `2 * i + 2` |

```
  Index mapping:

              [0]
            /     \
         [1]       [2]
        /   \     /
      [3]   [4] [5]

  Parent(4) = (4-1)/2 = 1  ✓ (index 1 is parent of index 4)
  Left(1) = 2*1+1 = 3      ✓
  Right(1) = 2*1+2 = 4     ✓
```

### Insert (Bubble Up / Sift Up)

```
INSERT(item):
    1. Add item at the END of the array (last position)
    2. BUBBLE UP: Compare with parent, swap if violating heap property
    3. Repeat until heap property is restored or root is reached

  Example: Insert 45 into max-heap

  Step 1: Add at end
              50
            /    \
          30      40
         /  \    /  \
        10   20 15  [45]    ← Added here

  Step 2: Compare 45 with parent 40 → 45 > 40 → SWAP
              50
            /    \
          30     [45]
         /  \    /  \
        10   20 15   40

  Step 3: Compare 45 with parent 50 → 45 < 50 → STOP
              50
            /    \
          30      45       ✓ Heap property restored
         /  \    /  \
        10   20 15   40
```

### Extract Max (Bubble Down / Sift Down)

```
EXTRACT_MAX():
    1. Save root (maximum element)
    2. Move LAST element to root position
    3. BUBBLE DOWN: Compare with children, swap with larger child
    4. Repeat until heap property is restored or leaf is reached
    5. Return saved maximum

  Example: Extract max from heap

  Step 1: Save root (50), move last (15) to root
             [15]
            /    \
          30      45
         /  \    /
        10   20 40

  Step 2: Compare 15 with children (30, 45) → swap with 45 (larger)
              45
            /    \
          30     [15]
         /  \    /
        10   20 40

  Step 3: Compare 15 with children (40) → swap with 40
              45
            /    \
          30      40
         /  \    /
        10   20 [15]

  Step 4: No more children → STOP
              45           ✓ Max-heap restored
            /    \
          30      40
         /  \    /
        10   20  15

  Return: 50
```

### Peek

```
PEEK():
    return array[0]    // Root is always max (or min)
    // O(1) — no modification needed
```

### Complexity Summary for Heap-Based PQ

| Operation | Time | Reason |
|-----------|:----:|--------|
| Insert | O(log n) | Bubble up at most `height` levels = log n |
| ExtractMax/Min | O(log n) | Bubble down at most `height` levels = log n |
| Peek | O(1) | Root is always the answer |
| Build Heap | O(n) | Bottom-up heapify (Floyd's algorithm) |
| ChangePriority | O(log n) | Bubble up or down after change |

---

## 6. Applications in Algorithms

### Application 1: Dijkstra's Shortest Path

```
  Find shortest paths from source to all vertices.

  DIJKSTRA(graph, source):
      dist[source] = 0, dist[all others] = ∞
      pq = MinPriorityQueue()
      pq.insert(source, 0)

      while pq is not empty:
          u = pq.extractMin()          // Get closest unvisited node
          for each neighbor v of u:
              newDist = dist[u] + weight(u, v)
              if newDist < dist[v]:
                  dist[v] = newDist
                  pq.insert(v, newDist)   // or decreaseKey

  Why PQ? Efficiently find the next closest node — O(log n)
  Without PQ: O(V²) scanning all vertices for minimum
  With PQ:    O((V + E) log V)
```

```
  Example Graph:
       A ──2── B
       │       │
       4       1
       │       │
       C ──3── D

  Dijkstra from A:
  PQ State         Extract    Update
  ──────────       ───────    ──────
  {(A,0)}          A(0)       B→2, C→4
  {(B,2),(C,4)}    B(2)       D→3
  {(D,3),(C,4)}    D(3)       C→min(4,6)=4
  {(C,4)}          C(4)       done

  Shortest distances: A=0, B=2, D=3, C=4
```

### Application 2: Huffman Coding

```
  Build optimal prefix-free code for data compression.

  Characters: a(5), b(9), c(12), d(13), e(16), f(45)

  HUFFMAN(frequencies):
      pq = MinPriorityQueue()
      for each (char, freq): pq.insert(node(char), freq)

      while pq.size() > 1:
          left  = pq.extractMin()
          right = pq.extractMin()
          merged = new node(left, right)
          merged.freq = left.freq + right.freq
          pq.insert(merged, merged.freq)

      return pq.extractMin()  // Root of Huffman tree

  Building the tree:
  Step 1: Extract a(5), b(9) → merge(14)
  Step 2: Extract c(12), d(13) → merge(25)
  Step 3: Extract merge(14), e(16) → merge(30)
  Step 4: Extract merge(25), merge(30) → merge(55)
  Step 5: Extract f(45), merge(55) → merge(100) — root!
```

### Application 3: Kth Largest Element

```
  Find the Kth largest element in a stream.

  Strategy: Use a MIN-PQ of size K
  - If PQ.size < K: insert element
  - Else if element > PQ.peek(): extractMin, insert element
  - PQ.peek() always gives the Kth largest

  Example: Stream = [3, 1, 5, 2, 7, 4],  K = 3

  Process 3: PQ = [3]               size=1
  Process 1: PQ = [1, 3]            size=2
  Process 5: PQ = [1, 3, 5]         size=3  → 3rd largest = 1
  Process 2: 2 > peek(1) → extract 1, insert 2
             PQ = [2, 3, 5]                  → 3rd largest = 2
  Process 7: 7 > peek(2) → extract 2, insert 7
             PQ = [3, 5, 7]                  → 3rd largest = 3
  Process 4: 4 > peek(3) → extract 3, insert 4
             PQ = [4, 5, 7]                  → 3rd largest = 4
```

### Application 4: Merge K Sorted Lists

```
  Merge K sorted linked lists into one sorted list.

  MERGE_K_LISTS(lists[]):
      pq = MinPriorityQueue()

      // Insert first element of each list
      for each list in lists:
          if list is not empty:
              pq.insert(list.head)

      result = new LinkedList()
      while pq is not empty:
          smallest = pq.extractMin()
          result.append(smallest)
          if smallest.next exists:
              pq.insert(smallest.next)

      return result

  Time: O(N log K) where N = total elements, K = number of lists
  Space: O(K) for the priority queue
```

### Application Summary

| Algorithm | PQ Type | Why PQ? | Complexity Gain |
|-----------|:-------:|---------|:---------------:|
| Dijkstra's | Min-PQ | Get closest unvisited node | O(V²) → O((V+E)log V) |
| Prim's MST | Min-PQ | Get minimum weight edge | O(V²) → O(E log V) |
| Huffman | Min-PQ | Merge two lowest frequencies | O(n²) → O(n log n) |
| Kth Largest | Min-PQ | Maintain top-K efficiently | O(nk) → O(n log k) |
| Merge K Lists | Min-PQ | Find smallest among K heads | O(nK) → O(n log K) |
| A* Search | Min-PQ | Get node with lowest f-score | Efficient pathfinding |
| Median Stream | Min+Max | Track upper/lower halves | O(n) → O(log n) per elem |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Priority Queue | Elements dequeued by priority, not arrival order |
| Max-PQ | Highest priority dequeued first (max-heap) |
| Min-PQ | Lowest priority dequeued first (min-heap) |
| Unsorted array PQ | O(1) insert, O(n) extract |
| Sorted array PQ | O(n) insert, O(1) extract |
| Heap PQ | O(log n) insert, O(log n) extract — best balance |
| Binary heap | Complete binary tree stored in array |
| Bubble up | Insert: move new element toward root |
| Bubble down | Extract: move replacement toward leaves |
| Negate trick | Convert min-PQ to max-PQ by negating priorities |
| Main algorithms | Dijkstra, Prim, Huffman, A*, Kth largest |

---

## Quick Revision Questions

1. **How does a priority queue differ from a regular queue? What determines the dequeue order?**

2. **Compare the time complexities of insert and extract for unsorted array, sorted array, and heap-based priority queues.**

3. **In a max-heap with array `[50, 30, 40, 10, 20, 15]`, what are the parent and children of the element at index 2? Use the formulas.**

4. **Describe the bubble-up process when inserting 45 into a max-heap `[50, 30, 40, 10, 20]`.**

5. **Why is a min-priority queue used in Dijkstra's algorithm instead of a max-priority queue?**

6. **How would you use a min-priority queue (the only type available) to implement a max-priority queue?**

---

## Answers (Hidden — Think First!)

<details>
<summary>Click to reveal answers</summary>

1. A regular queue dequeues in **arrival order** (FIFO). A priority queue dequeues by **priority value** — the element with the highest (max-PQ) or lowest (min-PQ) priority is served first, regardless of arrival time.

2. | Implementation | Insert | Extract |
   |:---:|:---:|:---:|
   | Unsorted Array | O(1) | O(n) |
   | Sorted Array | O(n) | O(1) |
   | Binary Heap | O(log n) | O(log n) |
   The heap provides the best balanced performance.

3. Element at index 2 is `40`.
   - Parent: `(2-1)/2 = 0` → element `50` ✓
   - Left child: `2*2+1 = 5` → element `15` ✓
   - Right child: `2*2+2 = 6` → doesn't exist (out of bounds)

4. Insert 45 at end → `[50, 30, 40, 10, 20, 45]`. Index 5, parent = `(5-1)/2 = 2` → value `40`. Since `45 > 40`, swap → `[50, 30, 45, 10, 20, 40]`. New index 2, parent = `(2-1)/2 = 0` → value `50`. Since `45 < 50`, stop. Final: `[50, 30, 45, 10, 20, 40]`.

5. Dijkstra's algorithm needs to process the node with the **smallest tentative distance** next. A min-PQ extracts the minimum efficiently, finding the closest unvisited node in O(log n) instead of O(V) linear scan.

6. **Negate all priorities.** To insert with priority `p`, insert with priority `-p`. When `extractMin` returns the most negative value, it corresponds to the original maximum priority. Example: priorities 3, 7, 1 → stored as -3, -7, -1 → `extractMin` returns -7, which represents original maximum 7.

</details>

---

[← Previous: Deque](../04-Deque/01-deque.md) | [Back to README](../README.md) | [Next: Queue Problems →](../06-Queue-Problems/01-queue-problems.md)
