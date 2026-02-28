# Unit 2: Queue Implementation

[← Previous: Queue Fundamentals](../01-Queue-Fundamentals/01-queue-fundamentals.md) | [Back to README](../README.md) | [Next: Circular Queue →](../03-Circular-Queue/01-circular-queue.md)

---

## Chapter Overview

This unit covers **three** core ways to implement a queue: array-based, circular array-based, and linked list-based. You'll learn the mechanics of each approach, analyze their complexities, handle edge cases, and understand the trade-offs for choosing one over another.

---

## 1. Array-Based Queue (Naive/Linear)

The simplest implementation uses a fixed-size array with two pointers: `front` and `rear`.

### Structure

```
  Index:     0      1      2      3      4      5      6
           ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┐
  Array:   │  10  │  20  │  30  │  40  │      │      │      │
           └──────┴──────┴──────┴──────┴──────┴──────┴──────┘
              ↑                    ↑
            front                rear
           (index 0)           (index 3)

  Capacity = 7,  Size = 4
```

### Approach 1: Shift Elements on Dequeue (Simple but Slow)

```
ENQUEUE(item):
    if rear == capacity - 1:
        ERROR "Queue Overflow"
    rear = rear + 1
    array[rear] = item

DEQUEUE():
    if front > rear:
        ERROR "Queue Underflow"
    item = array[front]
    // Shift all elements left by 1
    for i = 0 to rear - 1:
        array[i] = array[i + 1]
    rear = rear - 1
    return item
```

**Problem:** Every `dequeue()` shifts all elements → **O(n)** per dequeue.

### Approach 2: Move Front Pointer (Fast but Wastes Space)

```
ENQUEUE(item):
    if rear == capacity - 1:
        ERROR "Queue Overflow"
    rear = rear + 1
    array[rear] = item

DEQUEUE():
    if front > rear:
        ERROR "Queue Underflow"
    item = array[front]
    front = front + 1
    return item
```

### Step-by-Step Trace (Approach 2)

```
Initial:  front = 0, rear = -1, capacity = 5

  Index:    0     1     2     3     4
          ┌─────┬─────┬─────┬─────┬─────┐
          │     │     │     │     │     │    Empty
          └─────┴─────┴─────┴─────┴─────┘
            ↑
           f,r=-1

─── Enqueue(10): rear=0 ───
          ┌─────┬─────┬─────┬─────┬─────┐
          │ 10  │     │     │     │     │
          └─────┴─────┴─────┴─────┴─────┘
            ↑
           f=0, r=0

─── Enqueue(20): rear=1 ───
          ┌─────┬─────┬─────┬─────┬─────┐
          │ 10  │ 20  │     │     │     │
          └─────┴─────┴─────┴─────┴─────┘
            ↑     ↑
           f=0   r=1

─── Enqueue(30): rear=2 ───
          ┌─────┬─────┬─────┬─────┬─────┐
          │ 10  │ 20  │ 30  │     │     │
          └─────┴─────┴─────┴─────┴─────┘
            ↑           ↑
           f=0         r=2

─── Dequeue() → 10: front=1 ───
          ┌─────┬─────┬─────┬─────┬─────┐
          │ xx  │ 20  │ 30  │     │     │
          └─────┴─────┴─────┴─────┴─────┘
                  ↑     ↑
                 f=1   r=2

─── Dequeue() → 20: front=2 ───
          ┌─────┬─────┬─────┬─────┬─────┐
          │ xx  │ xx  │ 30  │     │     │
          └─────┴─────┴─────┴─────┴─────┘
                        ↑
                      f=2, r=2

─── Enqueue(40): rear=3 ───
          ┌─────┬─────┬─────┬─────┬─────┐
          │ xx  │ xx  │ 30  │ 40  │     │
          └─────┴─────┴─────┴─────┴─────┘
                        ↑     ↑
                       f=2   r=3

─── Enqueue(50): rear=4 ───
          ┌─────┬─────┬─────┬─────┬─────┐
          │ xx  │ xx  │ 30  │ 40  │ 50  │
          └─────┴─────┴─────┴─────┴─────┘
                        ↑           ↑
                       f=2         r=4

─── Enqueue(60): rear would be 5 → OVERFLOW! ───
   But indices 0 and 1 are wasted (xx)!
```

> **The Wasted Space Problem:** Approach 2 is O(1) for dequeue, but once `rear` reaches the end, space at the beginning is wasted. **This is exactly why circular queues exist** (covered in Unit 3).

---

## 2. Circular Queue (Preview)

The circular queue wraps around using modular arithmetic, eliminating wasted space.

```
  Conceptual View (ring):

          ┌───┐
       ┌──│ 0 │──┐
       │  └───┘  │
     ┌───┐     ┌───┐
     │ 4 │     │ 1 │
     └───┘     └───┘
       │  ┌───┐  │
       └──│ 3 │──┘
          └───┘
           │
         ┌───┐
         │ 2 │
         └───┘

  next index = (current + 1) % capacity
```

### Key Formula

```
next_position = (current_position + 1) % capacity
```

> Full details in [Unit 3: Circular Queue](../03-Circular-Queue/01-circular-queue.md).

---

## 3. Linked List-Based Queue

Uses a singly linked list with `front` and `rear` pointers. No fixed capacity — grows dynamically.

### Structure

```
  front                                    rear
    │                                        │
    ▼                                        ▼
  ┌──────┬───┐    ┌──────┬───┐    ┌──────┬──────┐
  │  10  │ ──┼──→ │  20  │ ──┼──→ │  30  │ NULL │
  └──────┴───┘    └──────┴───┘    └──────┴──────┘
    Node 1          Node 2          Node 3
```

### Node Definition

```
class Node:
    data    // the stored value
    next    // pointer to next node
```

### Pseudocode

```
class LinkedListQueue:
    front = NULL
    rear  = NULL
    size  = 0

    ENQUEUE(item):
        newNode = new Node(item)
        newNode.next = NULL
        
        if rear == NULL:           // Queue is empty
            front = newNode
            rear  = newNode
        else:
            rear.next = newNode    // Link new node after rear
            rear = newNode         // Update rear pointer
        
        size = size + 1

    DEQUEUE():
        if front == NULL:
            ERROR "Queue Underflow"
        
        item = front.data
        front = front.next        // Move front to next node
        
        if front == NULL:          // Queue became empty
            rear = NULL
        
        size = size - 1
        return item

    PEEK():
        if front == NULL:
            ERROR "Queue Underflow"
        return front.data

    IS_EMPTY():
        return front == NULL
```

### Step-by-Step Trace

```
─── Initial: front=NULL, rear=NULL ───

   (empty)

─── Enqueue(10) ───

   front, rear
       │
       ▼
   ┌──────┬──────┐
   │  10  │ NULL │
   └──────┴──────┘

─── Enqueue(20) ───

   front              rear
     │                  │
     ▼                  ▼
   ┌──────┬───┐    ┌──────┬──────┐
   │  10  │ ──┼──→ │  20  │ NULL │
   └──────┴───┘    └──────┴──────┘

─── Enqueue(30) ───

   front                             rear
     │                                 │
     ▼                                 ▼
   ┌──────┬───┐    ┌──────┬───┐    ┌──────┬──────┐
   │  10  │ ──┼──→ │  20  │ ──┼──→ │  30  │ NULL │
   └──────┴───┘    └──────┴───┘    └──────┴──────┘

─── Dequeue() → 10 ───

                front              rear
                  │                  │
                  ▼                  ▼
               ┌──────┬───┐    ┌──────┬──────┐
               │  20  │ ──┼──→ │  30  │ NULL │
               └──────┴───┘    └──────┴──────┘

   Node with 10 is freed/garbage collected.

─── Dequeue() → 20 ───

              front, rear
                  │
                  ▼
             ┌──────┬──────┐
             │  30  │ NULL │
             └──────┴──────┘

─── Dequeue() → 30 ───

   front = NULL, rear = NULL  →  Queue is empty again.
```

---

## 4. Queue Operations Complexity

### Time Complexity Comparison

| Operation | Array (Shift) | Array (Front Ptr) | Circular Array | Linked List |
|-----------|:------------:|:------------------:|:--------------:|:-----------:|
| Enqueue   | O(1)         | O(1)               | O(1)           | O(1)        |
| Dequeue   | **O(n)**     | O(1)               | O(1)           | O(1)        |
| Peek      | O(1)         | O(1)               | O(1)           | O(1)        |
| isEmpty   | O(1)         | O(1)               | O(1)           | O(1)        |
| size      | O(1)         | O(1)               | O(1)           | O(1)        |
| isFull    | O(1)         | O(1)               | O(1)           | N/A*        |

> *Linked list queue has no fixed capacity (unless artificially limited).

### Space Complexity

| Implementation | Space | Notes |
|---------------|:-----:|-------|
| Array (Fixed) | O(n) | `n` = capacity (pre-allocated) |
| Circular Array | O(n) | `n` = capacity (pre-allocated) |
| Linked List | O(n) | `n` = current size + pointer overhead |

### Per-Element Memory Overhead

```
Array Queue:
  ┌──────────┐
  │   data   │     Only the data — no extra pointers
  └──────────┘
  Overhead per element: 0 pointers

Linked List Queue:
  ┌──────────┬──────────┐
  │   data   │   next   │     Extra pointer per element
  └──────────┴──────────┘
  Overhead per element: 1 pointer (4-8 bytes)
```

---

## 5. Handling Overflow and Underflow

### Underflow (Empty Queue)

Occurs when `dequeue()` or `peek()` is called on an empty queue.

```
Detection:
  Array Queue:      front > rear
  Circular Queue:   size == 0  (or front == -1)
  Linked List:      front == NULL

Handling Strategies:
  1. Throw an exception          → throw QueueEmptyException
  2. Return a sentinel value     → return -1 or NULL
  3. Return boolean + out param  → bool dequeue(out item)
  4. Assert (debug builds)       → assert(!isEmpty())
```

### Overflow (Full Queue)

Occurs when `enqueue()` is called on a full bounded queue.

```
Detection:
  Array Queue:      rear == capacity - 1
  Circular Queue:   size == capacity  (or (rear+1) % cap == front)
  Linked List:      Typically never (limited by system memory)

Handling Strategies:
  1. Throw an exception          → throw QueueFullException
  2. Resize the array            → double the capacity (dynamic array)
  3. Block the caller            → wait until space frees (blocking queue)
  4. Overwrite oldest            → circular buffer / ring buffer
  5. Reject and return false     → boolean enqueue(item)
```

### Dynamic Resizing Strategy

```
ENQUEUE_WITH_RESIZE(item):
    if size == capacity:
        new_capacity = capacity * 2
        new_array = allocate(new_capacity)
        
        // Copy elements in order from front to rear
        for i = 0 to size - 1:
            new_array[i] = old_array[(front + i) % capacity]
        
        front = 0
        rear = size - 1
        capacity = new_capacity
        array = new_array
    
    rear = (rear + 1) % capacity
    array[rear] = item
    size = size + 1
```

```
Before resize (capacity=4, full):
  ┌────┬────┬────┬────┐
  │ 30 │ 40 │ 10 │ 20 │     front=2, rear=1
  └────┴────┴────┴────┘
            ↑     ↑
           front  

After resize (capacity=8):
  ┌────┬────┬────┬────┬────┬────┬────┬────┐
  │ 10 │ 20 │ 30 │ 40 │    │    │    │    │
  └────┴────┴────┴────┴────┴────┴────┴────┘
    ↑              ↑
  front=0        rear=3
```

> **Amortized Cost:** Resizing is O(n), but since it doubles the capacity, the amortized cost per enqueue remains **O(1)**.

---

## 6. Implementation Trade-offs

### Comparison Matrix

| Criteria | Array (Linear) | Circular Array | Linked List |
|----------|:--------------:|:--------------:|:-----------:|
| Memory Usage | Fixed, may waste | Fixed, efficient | Dynamic, pointer overhead |
| Cache Performance | Excellent | Excellent | Poor (scattered nodes) |
| Resize Needed? | Yes | Yes | No |
| Max Capacity | Fixed | Fixed | System memory |
| Implementation | Simple | Moderate | Moderate |
| Dequeue Efficiency | O(n) shift or waste | O(1) | O(1) |
| Memory Allocation | One-time | One-time | Per-enqueue |

### When to Choose What

```
┌──────────────────────────────────────────────────────────┐
│             CHOOSING A QUEUE IMPLEMENTATION               │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Known max size + need speed?                            │
│    → Circular Array Queue ✓                              │
│                                                          │
│  Unknown size + frequent resize?                         │
│    → Linked List Queue ✓                                 │
│                                                          │
│  Simple prototype / learning?                            │
│    → Linear Array Queue ✓                                │
│                                                          │
│  High performance + known size?                          │
│    → Circular Array (best cache locality) ✓              │
│                                                          │
│  Memory constrained + variable load?                     │
│    → Linked List (no wasted capacity) ✓                  │
│                                                          │
│  Need thread-safe bounded queue?                         │
│    → Circular Array with locks ✓                         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Language Standard Library Implementations

| Language | Queue Class | Underlying Structure |
|----------|-------------|---------------------|
| Java | `LinkedList` / `ArrayDeque` | Linked list / Circular array |
| Python | `collections.deque` | Doubly linked list of blocks |
| C++ | `std::queue` (adapter) | `std::deque` (block array) |
| JavaScript | No built-in | Use array with `push`/`shift` (O(n) shift!) |
| C# | `Queue<T>` | Circular array |

> **Tip for interviews:** In Java, prefer `ArrayDeque` over `LinkedList` for queue operations — it has better cache performance and lower overhead.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Array Queue (Shift) | Simple but O(n) dequeue due to shifting |
| Array Queue (Front Ptr) | O(1) dequeue but wastes space at front |
| Circular Queue | Wraps around using `% capacity` — no wasted space |
| Linked List Queue | Dynamic size, O(1) operations, pointer overhead |
| Overflow | Full queue — resize, block, or reject |
| Underflow | Empty queue — throw exception or return sentinel |
| Dynamic Resize | Double capacity when full — O(1) amortized |
| Cache Locality | Arrays > Linked lists for cache performance |
| Best General Choice | Circular array for bounded, linked list for unbounded |

---

## Quick Revision Questions

1. **Why is the naive array queue with element shifting inefficient? What is the time complexity of its dequeue operation?**

2. **In the front-pointer approach, what problem arises after multiple enqueue and dequeue operations? How does a circular queue solve this?**

3. **Draw the linked list queue state after: Enqueue(5), Enqueue(10), Dequeue(), Enqueue(15). What does `front` point to?**

4. **When resizing a circular array queue, why can't you simply allocate a bigger array and copy elements index-by-index?**

5. **Compare the per-element memory overhead of an array-based queue vs a linked list-based queue. When does this matter?**

6. **In Java, why is `ArrayDeque` generally preferred over `LinkedList` for implementing a queue?**

---

## Answers (Hidden — Think First!)

<details>
<summary>Click to reveal answers</summary>

1. After removing the front element, all remaining elements must shift left by one position. This is **O(n)** per dequeue operation, which is very costly for large queues.

2. Space at the beginning of the array is wasted (indices before `front` are unused). A circular queue wraps the `rear` pointer back to index 0 using `(rear + 1) % capacity`, reusing the freed space.

3. After all operations:
   ```
   front          rear
     │               │
     ▼               ▼
   ┌────┬───┐    ┌────┬──────┐
   │ 10 │ ──┼──→ │ 15 │ NULL │
   └────┴───┘    └────┴──────┘
   ```
   `front` points to the node containing **10**.

4. In a circular queue, logical order and physical order may differ. The front might be at index 3 and rear at index 1 (wrapped). A simple index-by-index copy would place elements in wrong order. You must copy from `front` to `rear` in logical order: `new[i] = old[(front + i) % capacity]`.

5. Array: **0 extra bytes** per element (just data). Linked list: **4-8 bytes** per element (one pointer). This matters when storing millions of small elements (e.g., chars or ints) — the pointer overhead can double the memory usage.

6. `ArrayDeque` uses a contiguous circular array, giving excellent **cache locality** (CPU prefetching). `LinkedList` has scattered nodes in memory, causing frequent **cache misses**. `ArrayDeque` also avoids per-node object allocation overhead.

</details>

---

[← Previous: Queue Fundamentals](../01-Queue-Fundamentals/01-queue-fundamentals.md) | [Back to README](../README.md) | [Next: Circular Queue →](../03-Circular-Queue/01-circular-queue.md)
