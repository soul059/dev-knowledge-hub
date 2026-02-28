# Unit 6: Queue Problems

[← Previous: Priority Queue](../05-Priority-Queue/01-priority-queue.md) | [Back to README](../README.md) | [Next: Monotonic Queue →](../07-Monotonic-Queue/01-monotonic-queue.md)

---

## Chapter Overview

This unit covers **classic queue-based problems** commonly asked in interviews and competitive programming. Each problem includes the intuition, approach, pseudocode, step-by-step trace, and complexity analysis.

---

## 1. Queue Using Two Stacks

### Problem

Implement a queue (FIFO) using only two stacks (LIFO).

### Intuition

```
  Stack = LIFO     Queue = FIFO
  
  If you pour items from one stack into another,
  the order REVERSES — giving you FIFO from LIFO!

  Stack1 (input):    Stack2 (output):
    ┌───┐              ┌───┐
    │ C │  ──pour──→   │ A │   ← Now A is on top!
    ├───┤              ├───┤
    │ B │              │ B │
    ├───┤              ├───┤
    │ A │              │ C │
    └───┘              └───┘
   Push order: A,B,C   Pop order: A,B,C  = FIFO!
```

### Approach: Lazy Transfer

- **Enqueue:** Always push to `stack1`.
- **Dequeue:** Pop from `stack2`. If `stack2` is empty, transfer all elements from `stack1` to `stack2` first.

### Pseudocode

```
class QueueUsingStacks:
    stack1 = []    // For enqueue
    stack2 = []    // For dequeue

    ENQUEUE(item):
        stack1.push(item)

    DEQUEUE():
        if stack2 is empty:
            if stack1 is empty:
                ERROR "Queue Underflow"
            // Transfer all from stack1 to stack2
            while stack1 is not empty:
                stack2.push(stack1.pop())
        return stack2.pop()

    PEEK():
        if stack2 is empty:
            while stack1 is not empty:
                stack2.push(stack1.pop())
        return stack2.top()

    IS_EMPTY():
        return stack1.isEmpty() AND stack2.isEmpty()
```

### Step-by-Step Trace

```
═══ Enqueue(1), Enqueue(2), Enqueue(3) ═══

  stack1:    stack2:
  ┌───┐
  │ 3 │     (empty)
  ├───┤
  │ 2 │
  ├───┤
  │ 1 │
  └───┘

═══ Dequeue() ═══
  stack2 empty → Transfer all from stack1 to stack2:
    pop 3, push to s2
    pop 2, push to s2
    pop 1, push to s2

  stack1:    stack2:
             ┌───┐
  (empty)    │ 1 │  ← pop this → returns 1 ✓
             ├───┤
             │ 2 │
             ├───┤
             │ 3 │
             └───┘

═══ Enqueue(4), Enqueue(5) ═══

  stack1:    stack2:
  ┌───┐     ┌───┐
  │ 5 │     │ 2 │
  ├───┤     ├───┤
  │ 4 │     │ 3 │
  └───┘     └───┘

═══ Dequeue() → 2 ═══
  stack2 not empty → just pop → returns 2 ✓

═══ Dequeue() → 3 ═══
  stack2 not empty → just pop → returns 3 ✓

═══ Dequeue() → 4 ═══
  stack2 empty → Transfer from stack1:
    pop 5, push to s2
    pop 4, push to s2

  stack1:    stack2:
             ┌───┐
  (empty)    │ 4 │  ← pop → returns 4 ✓
             ├───┤
             │ 5 │
             └───┘

  Overall dequeue order: 1, 2, 3, 4 = FIFO ✓
```

### Complexity

| Operation | Worst Case | Amortized |
|-----------|:----------:|:---------:|
| Enqueue | O(1) | O(1) |
| Dequeue | O(n) | **O(1)** |
| Space | O(n) | O(n) |

> **Amortized O(1):** Each element is moved from stack1 to stack2 at most once. Over n operations, the total transfers are O(n), so each dequeue is O(1) amortized.

---

## 2. Stack Using Two Queues

### Problem

Implement a stack (LIFO) using only two queues (FIFO).

### Approach: Make Push Expensive

- **Push:** Enqueue to `q2`. Then transfer all elements from `q1` to `q2`. Swap `q1` and `q2`.
- **Pop:** Simply dequeue from `q1`.

```
PUSH(item):
    q2.enqueue(item)
    while q1 is not empty:
        q2.enqueue(q1.dequeue())
    SWAP(q1, q2)

POP():
    if q1 is empty:
        ERROR "Stack Underflow"
    return q1.dequeue()

TOP():
    return q1.peek()
```

### Step-by-Step Trace

```
═══ Push(1) ═══
  q2: [1]
  Transfer q1→q2: nothing to transfer
  Swap: q1=[1], q2=[]

  q1: [1]

═══ Push(2) ═══
  q2: [2]
  Transfer q1→q2: move 1 → q2=[2, 1]
  Swap: q1=[2, 1], q2=[]

  q1: front→ [2, 1] ←rear
       ↑
      top

═══ Push(3) ═══
  q2: [3]
  Transfer q1→q2: move 2, 1 → q2=[3, 2, 1]
  Swap: q1=[3, 2, 1], q2=[]

  q1: front→ [3, 2, 1] ←rear
       ↑
      top

═══ Pop() → 3 ═══
  Dequeue from q1 → 3 (the front = most recent = LIFO!) ✓

  q1: front→ [2, 1] ←rear

═══ Pop() → 2  ✓
═══ Pop() → 1  ✓
```

### Alternative: Make Pop Expensive

```
PUSH(item):
    q1.enqueue(item)

POP():
    // Move all except last to q2
    while q1.size() > 1:
        q2.enqueue(q1.dequeue())
    item = q1.dequeue()    // Last element = most recently added
    SWAP(q1, q2)
    return item
```

### Complexity Comparison

| Approach | Push | Pop | Space |
|----------|:----:|:---:|:-----:|
| Push expensive | O(n) | O(1) | O(n) |
| Pop expensive | O(1) | O(n) | O(n) |

---

## 3. First Non-Repeating Character in a Stream

### Problem

Given a stream of characters, find the first non-repeating character at each step. If no non-repeating character exists, output `-1`.

```
Stream:  a, a, b, c, b, d
Output:  a, -1, b, b, c, c
```

### Approach

Use a **queue** to maintain the order of characters and a **frequency map** to track counts.

```
FIRST_NON_REPEATING(stream):
    queue = empty queue
    freq = empty hashmap

    for each char c in stream:
        freq[c] = freq[c] + 1
        queue.enqueue(c)

        // Remove characters from front that are now repeating
        while queue is not empty AND freq[queue.peek()] > 1:
            queue.dequeue()

        if queue is empty:
            output -1
        else:
            output queue.peek()
```

### Step-by-Step Trace

```
Stream: a, a, b, c, b, d

═══ Process 'a' ═══
  freq: {a:1}
  queue: [a]
  Front 'a' has freq 1 → output: 'a'

═══ Process 'a' ═══
  freq: {a:2}
  queue: [a, a]
  Front 'a' has freq 2 → dequeue → [a]
  Front 'a' has freq 2 → dequeue → []
  Queue empty → output: -1

═══ Process 'b' ═══
  freq: {a:2, b:1}
  queue: [b]
  Front 'b' has freq 1 → output: 'b'

═══ Process 'c' ═══
  freq: {a:2, b:1, c:1}
  queue: [b, c]
  Front 'b' has freq 1 → output: 'b'

═══ Process 'b' ═══
  freq: {a:2, b:2, c:1}
  queue: [b, c]
  Front 'b' has freq 2 → dequeue → [c]
  Front 'c' has freq 1 → output: 'c'

═══ Process 'd' ═══
  freq: {a:2, b:2, c:1, d:1}
  queue: [c, d]
  Front 'c' has freq 1 → output: 'c'

Final output: a, -1, b, b, c, c  ✓
```

### Complexity

| Metric | Value |
|--------|:-----:|
| Time | O(n) — each character enqueued and dequeued at most once |
| Space | O(n) — queue + frequency map |

---

## 4. Circular Tour Problem (Gas Station)

### Problem

There are `n` petrol pumps in a circle. Each pump `i` has `petrol[i]` fuel and requires `cost[i]` fuel to reach the next pump. Find the starting pump from which a truck can complete the full circle.

```
  Pumps arranged in circle:

        Pump 0
       petrol=4
       cost=6
      ↗        ↘
  Pump 3        Pump 1
  petrol=5      petrol=6
  cost=2        cost=5
      ↖        ↙
        Pump 2
       petrol=7
       cost=3
```

### Key Insight

```
  net[i] = petrol[i] - cost[i]

  If total(net) < 0 → No solution exists.
  If total(net) ≥ 0 → Start after the point where cumulative sum is most negative.
```

### Approach: Single Pass with Queue-like Logic

```
CIRCULAR_TOUR(petrol[], cost[], n):
    start = 0
    tank = 0
    total_surplus = 0

    for i = 0 to n - 1:
        net = petrol[i] - cost[i]
        tank = tank + net
        total_surplus = total_surplus + net

        if tank < 0:
            // Can't reach pump i+1 from current start
            // Reset: try starting from i+1
            start = i + 1
            tank = 0

    if total_surplus >= 0:
        return start
    else:
        return -1    // No valid start
```

### Step-by-Step Trace

```
  Pump:    0     1     2     3
  Petrol:  4     6     7     5
  Cost:    6     5     3     2
  Net:    -2     1     4     3

  Total surplus = -2 + 1 + 4 + 3 = 6 ≥ 0 → Solution exists!

  i=0: net=-2, tank=-2 < 0 → start=1, tank=0
  i=1: net=1, tank=1 ≥ 0 → continue
  i=2: net=4, tank=5 ≥ 0 → continue
  i=3: net=3, tank=8 ≥ 0 → continue

  Answer: start = 1 ✓

  Verify starting from pump 1:
    Pump 1: tank = 0 + 6 - 5 = 1  ✓
    Pump 2: tank = 1 + 7 - 3 = 5  ✓
    Pump 3: tank = 5 + 5 - 2 = 8  ✓
    Pump 0: tank = 8 + 4 - 6 = 6  ✓ Complete circle!
```

### Complexity

| Metric | Value |
|--------|:-----:|
| Time | O(n) |
| Space | O(1) |

---

## 5. LRU Cache Concept

### Problem

Design a data structure that supports `get(key)` and `put(key, value)` operations with O(1) time, and evicts the **Least Recently Used** item when the cache reaches capacity.

### Concept: Queue + HashMap

```
  ┌─────────────────────────────────────────────────┐
  │                  LRU CACHE                       │
  │                                                  │
  │  HashMap (key → node reference) for O(1) lookup  │
  │                                                  │
  │  Doubly Linked List for O(1) reorder/eviction    │
  │                                                  │
  │  MRU ←──────────────────────────── LRU           │
  │  (front)                          (rear)         │
  │                                                  │
  │  ┌───┐    ┌───┐    ┌───┐    ┌───┐               │
  │  │ D │ ←→ │ B │ ←→ │ C │ ←→ │ A │               │
  │  └───┘    └───┘    └───┘    └───┘               │
  │  Most                        Least              │
  │  Recent                      Recent             │
  │  Used                        Used               │
  │                              ↑                   │
  │                         EVICT THIS               │
  │                         when capacity             │
  │                         is exceeded               │
  └─────────────────────────────────────────────────┘
```

### Operations

```
GET(key):
    if key not in hashmap:
        return -1
    node = hashmap[key]
    moveToFront(node)      // Mark as most recently used
    return node.value

PUT(key, value):
    if key in hashmap:
        node = hashmap[key]
        node.value = value
        moveToFront(node)  // Mark as most recently used
    else:
        if size == capacity:
            // Evict LRU (tail of doubly linked list)
            lruNode = removeTail()
            hashmap.remove(lruNode.key)
        newNode = new Node(key, value)
        addToFront(newNode)
        hashmap[key] = newNode
```

### Trace Example (Capacity = 3)

```
═══ put(1, "A") ═══
  List: [1:A]
  Map: {1→node}

═══ put(2, "B") ═══
  List: [2:B] ←→ [1:A]
  Map: {1→node, 2→node}

═══ put(3, "C") ═══
  List: [3:C] ←→ [2:B] ←→ [1:A]
  Map: {1→node, 2→node, 3→node}

═══ get(1) → "A" ═══
  Move 1 to front:
  List: [1:A] ←→ [3:C] ←→ [2:B]
         ↑ MRU                 ↑ LRU

═══ put(4, "D") — capacity exceeded! ═══
  Evict LRU: remove [2:B] from tail
  Add [4:D] at front:
  List: [4:D] ←→ [1:A] ←→ [3:C]
  Map: {1→node, 3→node, 4→node}  (2 evicted!)

═══ get(2) → -1 ═══
  Key 2 was evicted. Cache miss!
```

### Complexity

| Operation | Time | Space |
|-----------|:----:|:-----:|
| get | O(1) | O(capacity) |
| put | O(1) | O(capacity) |

> **Why doubly linked list?** We need O(1) removal of an arbitrary node (when it's accessed, move it to front). With a doubly linked list, given a node reference, we can unlink it in O(1). A singly linked list would need O(n) to find the previous node.

---

## 6. Generate Binary Numbers 1 to N

### Problem

Generate binary representations of numbers 1 to N using a queue.

```
Input: N = 10
Output: 1, 10, 11, 100, 101, 110, 111, 1000, 1001, 1010
```

### Intuition

```
  Start with "1" in the queue.
  For each number dequeued, generate two new numbers:
    - Append "0" → next binary number (left child in binary tree)
    - Append "1" → next binary number (right child in binary tree)

  Binary Tree View:
                1
              /   \
            10     11
           / \    / \
         100 101 110 111
         ...
  
  BFS of this tree gives: 1, 10, 11, 100, 101, 110, 111, ...
  Which are binary representations of: 1, 2, 3, 4, 5, 6, 7, ...
```

### Pseudocode

```
GENERATE_BINARY(n):
    queue = empty queue
    result = []
    queue.enqueue("1")

    for i = 1 to n:
        current = queue.dequeue()
        result.append(current)
        queue.enqueue(current + "0")    // Append 0
        queue.enqueue(current + "1")    // Append 1

    return result
```

### Step-by-Step Trace (N = 6)

```
Queue: ["1"]

═══ i=1 ═══
  Dequeue: "1" → output
  Enqueue: "10", "11"
  Queue: ["10", "11"]
  Result: [1]

═══ i=2 ═══
  Dequeue: "10" → output
  Enqueue: "100", "101"
  Queue: ["11", "100", "101"]
  Result: [1, 10]

═══ i=3 ═══
  Dequeue: "11" → output
  Enqueue: "110", "111"
  Queue: ["100", "101", "110", "111"]
  Result: [1, 10, 11]

═══ i=4 ═══
  Dequeue: "100" → output
  Enqueue: "1000", "1001"
  Queue: ["101", "110", "111", "1000", "1001"]
  Result: [1, 10, 11, 100]

═══ i=5 ═══
  Dequeue: "101" → output
  Queue: ["110", "111", "1000", "1001", "1010", "1011"]
  Result: [1, 10, 11, 100, 101]

═══ i=6 ═══
  Dequeue: "110" → output
  Result: [1, 10, 11, 100, 101, 110]

Final: 1, 10, 11, 100, 101, 110  ✓
```

### Why Queue Works Here

```
  The queue naturally processes numbers in increasing order
  because binary numbers form a LEVEL-ORDER traversal of
  a binary tree.

  Level 0:  1                    (decimal 1)
  Level 1:  10, 11               (decimal 2, 3)
  Level 2:  100, 101, 110, 111   (decimal 4, 5, 6, 7)

  Queue = BFS = Level-order = Increasing order!
```

### Complexity

| Metric | Value |
|--------|:-----:|
| Time | O(n) |
| Space | O(n) — queue size grows but we only need n elements |

---

## Problem-Solving Patterns

### When to Think "Queue"

| Pattern | Problem Type | Queue Role |
|---------|-------------|------------|
| **FIFO processing** | Process in arrival order | Standard queue |
| **Level-by-level** | BFS, binary number gen | BFS queue |
| **Sliding window** | Window max/min, averages | Deque for O(1) access |
| **Streaming** | First non-repeating, median | Queue + auxiliary structure |
| **Simulation** | Round-robin, hot potato | Circular processing |
| **Two structures** | LRU cache, queue↔stack | Queue + hashmap / 2 stacks |

### Common Patterns Summary

```
┌────────────────────────────────────────────────────┐
│  PATTERN 1: BFS-like Generation                     │
│  Put seed → loop: dequeue, process, enqueue next    │
│  Examples: Binary numbers, tree nodes, graph BFS    │
├────────────────────────────────────────────────────┤
│  PATTERN 2: Stream Processing                       │
│  Queue maintains order, hashmap tracks counts       │
│  Examples: First non-repeating, recent average      │
├────────────────────────────────────────────────────┤
│  PATTERN 3: Simulate with Queue & Stack             │
│  Simulate one DS using another                      │
│  Examples: Queue ↔ Stack, interleaving problems     │
├────────────────────────────────────────────────────┤
│  PATTERN 4: Window Maintenance                      │
│  Deque maintains window invariant                   │
│  Examples: Sliding max/min, BFS with constraints    │
└────────────────────────────────────────────────────┘
```

---

## Summary Table

| Problem | Approach | Time | Space |
|---------|----------|:----:|:-----:|
| Queue using 2 stacks | Lazy transfer between stacks | O(1) amortized | O(n) |
| Stack using 2 queues | Rebuild order on push or pop | O(n) | O(n) |
| First non-repeating char | Queue + frequency map | O(n) | O(n) |
| Circular tour | Single-pass greedy | O(n) | O(1) |
| LRU Cache | Doubly linked list + hashmap | O(1) | O(capacity) |
| Generate binary 1-N | BFS-like queue generation | O(n) | O(n) |

---

## Quick Revision Questions

1. **In the "queue using two stacks" approach, why is the amortized dequeue cost O(1) even though a single transfer can be O(n)?**

2. **When implementing a stack using two queues, which approach (push-expensive vs pop-expensive) is better if pushes are more frequent than pops?**

3. **In the first non-repeating character problem, why do we need both a queue AND a frequency map? Could we use only one of them?**

4. **In the circular tour problem, why can we reset `start = i + 1` when `tank < 0`? Why not try every possible start?**

5. **Why does an LRU Cache use a doubly linked list instead of a singly linked list?**

6. **Explain why the generate binary numbers problem is essentially a BFS traversal.**

---

## Answers (Hidden — Think First!)

<details>
<summary>Click to reveal answers</summary>

1. Each element is transferred from stack1 to stack2 **at most once** in its lifetime. If we do `n` enqueues and `n` dequeues, there are at most `n` transfers total. Spread across `n` dequeue operations, the cost per dequeue is `n/n = O(1)` amortized.

2. **Pop-expensive** is better when pushes are more frequent. Push is O(1) (just enqueue), and the expensive O(n) pop happens less frequently. With push-expensive, every push would be O(n), making the frequent operation costly.

3. **Queue alone** maintains order but doesn't know if a character is repeating. **Frequency map alone** knows repetition but can't tell which non-repeating character came first. Together: frequency map answers "is it repeating?" and queue answers "which came first?"

4. If we can't reach pump `i+1` from `start`, then any intermediate pump `j` (start ≤ j ≤ i) also can't be a valid start — because we arrived at `j` from `start` with a non-negative tank, yet still ran out before `i+1`. Starting at `j` with 0 fuel would be even worse. So we safely skip to `i+1`.

5. When a node is accessed (cache hit), we need to **remove it from its current position** and move it to the front. With a doubly linked list, given a reference to the node, we can unlink it in O(1) — adjust `prev.next` and `next.prev`. A singly linked list needs O(n) to find the previous node for unlinking.

6. Binary numbers can be viewed as a binary tree: root=1, left child=append "0", right child=append "1". BFS of this tree visits: 1, 10, 11, 100, 101, 110, 111... which are binary representations of 1, 2, 3, 4, 5, 6, 7. The queue gives level-order (BFS) traversal = numerically increasing binary numbers.

</details>

---

[← Previous: Priority Queue](../05-Priority-Queue/01-priority-queue.md) | [Back to README](../README.md) | [Next: Monotonic Queue →](../07-Monotonic-Queue/01-monotonic-queue.md)
