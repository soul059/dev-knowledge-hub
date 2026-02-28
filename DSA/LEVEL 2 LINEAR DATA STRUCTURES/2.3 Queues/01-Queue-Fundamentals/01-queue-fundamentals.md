# Unit 1: Queue Fundamentals

[← Back to README](../README.md) | [Next: Queue Implementation →](../02-Queue-Implementation/01-queue-implementation.md)

---

## Chapter Overview

This unit introduces the **Queue** data structure — its definition, the FIFO principle, the abstract data type (ADT), real-world analogies, applications, and guidance on when to use queues over other data structures.

---

## 1. What is a Queue?

A **queue** is a linear data structure that stores elements in a **sequential order**. Elements are added at one end (the **rear/back**) and removed from the other end (the **front**).

Think of it like a line of people waiting — the first person to join the line is the first person to be served.

```
  ┌─────────────────────────────────────────────────┐
  │                    QUEUE                         │
  │                                                  │
  │   ENQUEUE                          DEQUEUE       │
  │   (Insert)                         (Remove)      │
  │      │                                │          │
  │      ▼                                ▼          │
  │   ┌──────┬──────┬──────┬──────┬──────┐          │
  │   │  E   │  D   │  C   │  B   │  A   │          │
  │   └──────┴──────┴──────┴──────┴──────┘          │
  │   ↑                                ↑            │
  │   REAR                           FRONT           │
  │   (Back)                         (Front)         │
  └─────────────────────────────────────────────────┘
```

### Key Properties

| Property | Description |
|----------|-------------|
| **Order** | Elements maintain insertion order |
| **Access** | Only the front element is accessible |
| **Insertion** | Always at the rear |
| **Deletion** | Always from the front |
| **Type** | Linear, ordered collection |

---

## 2. FIFO Principle

**FIFO = First In, First Out**

The element that enters the queue **first** is the element that leaves the queue **first**. This is the defining characteristic of a queue.

### FIFO vs LIFO Comparison

```
  QUEUE (FIFO)                     STACK (LIFO)
  ────────────                     ────────────
  
  Insert: A, B, C                  Insert: A, B, C
  
  ┌───┬───┬───┐                    ┌───┐
  │ C │ B │ A │                    │ C │ ← Top
  └───┴───┴───┘                    ├───┤
  ↑           ↑                    │ B │
  Rear      Front                  ├───┤
                                   │ A │
  Remove order: A, B, C           └───┘
  (Same as insert order)
                                   Remove order: C, B, A
                                   (Reverse of insert order)
```

### Step-by-Step FIFO Trace

Let's trace operations on an empty queue:

```
Operation          Queue State (Front → Rear)     Front    Rear
─────────          ──────────────────────────      ─────    ────
Initial            [ ]                             -        -
Enqueue(10)        [ 10 ]                          10       10
Enqueue(20)        [ 10, 20 ]                      10       20
Enqueue(30)        [ 10, 20, 30 ]                  10       30
Dequeue() → 10     [ 20, 30 ]                      20       30
Dequeue() → 20     [ 30 ]                          30       30
Enqueue(40)        [ 30, 40 ]                      30       40
Dequeue() → 30     [ 40 ]                          40       40
Dequeue() → 40     [ ]                             -        -
```

> **Key Insight:** The dequeue order (10, 20, 30, 40) matches the enqueue order. That's FIFO.

---

## 3. Queue ADT (Abstract Data Type)

An ADT defines **what** operations are available, not **how** they are implemented. The Queue ADT specifies a contract that any queue implementation must satisfy.

### Core Operations

| Operation | Description | Returns |
|-----------|-------------|---------|
| `enqueue(item)` | Add item to the rear of the queue | void |
| `dequeue()` | Remove and return item from the front | item |
| `peek()` / `front()` | Return front item without removing it | item |
| `isEmpty()` | Check if queue has no elements | boolean |
| `size()` | Return number of elements in queue | integer |

### Optional Operations

| Operation | Description |
|-----------|-------------|
| `rear()` | View the rear element without removing |
| `isFull()` | Check if queue is at capacity (bounded queues) |
| `clear()` | Remove all elements |
| `contains(item)` | Check if item exists in the queue |

### Pseudocode — Queue ADT

```
ADT Queue:
    Data:
        A collection of elements with a front and rear reference
    
    Operations:
        enqueue(item):
            Pre:  Queue is not full (if bounded)
            Post: item is added at the rear
        
        dequeue() → item:
            Pre:  Queue is not empty
            Post: Front item is removed and returned
        
        peek() → item:
            Pre:  Queue is not empty
            Post: Front item is returned (queue unchanged)
        
        isEmpty() → boolean:
            Post: Returns true if queue has 0 elements
        
        size() → integer:
            Post: Returns count of elements in queue
```

### Precondition Violations

| Violation | Condition | Typical Response |
|-----------|-----------|------------------|
| **Underflow** | `dequeue()` or `peek()` on empty queue | Throw exception / return error |
| **Overflow** | `enqueue()` on full bounded queue | Throw exception / resize / block |

---

## 4. Real-World Analogies

Queues are everywhere in the real world. Recognizing these patterns helps you identify queue problems.

### Analogy 1: Ticket Counter Line

```
     ┌─────────────────────────────────┐
     │         TICKET COUNTER          │
     └─────────────────────────────────┘
                    ↑
                 (DEQUEUE)
                    │
     ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐
     │ E │──│ D │──│ C │──│ B │──│ A │──→  Served!
     └───┘  └───┘  └───┘  └───┘  └───┘
       ↑                           ↑
    New person                  Next to
     joins                     be served
    (ENQUEUE)                  (FRONT)
```

- People join at the back → `enqueue()`
- People are served from the front → `dequeue()`
- Fair: first come, first served → FIFO

### Analogy 2: Printer Queue

```
  Document D ──┐
  Document C ──┤    ┌──────────┐    ┌─────────┐
  Document B ──┼──→ │  PRINT   │──→ │ PRINTER │──→ Printed!
  Document A ──┘    │  QUEUE   │    └─────────┘
                    └──────────┘
```

- Documents are printed in the order they were submitted.

### Analogy 3: CPU Task Scheduling

```
  ┌─────────┐     ┌─────────────────┐     ┌───────┐
  │ Process  │     │   READY QUEUE   │     │       │
  │ Arrival  │────→│ P1 │ P2 │ P3   │────→│  CPU  │
  │          │     │                 │     │       │
  └─────────┘     └─────────────────┘     └───────┘
```

- Processes arrive and wait in the ready queue.
- The CPU picks the **next in line** (FIFO scheduling).

### More Real-World Queues

| Real World | Queue Element | Enqueue Event | Dequeue Event |
|------------|---------------|---------------|---------------|
| Bank line | Customer | Customer arrives | Customer served |
| Traffic | Car | Car arrives at signal | Signal turns green |
| Call center | Caller | Caller dials in | Agent picks up |
| Playlist | Song | Song added | Song plays |
| Download | File | Download requested | Download completes |

---

## 5. Queue Applications Overview

Queues are used extensively across computer science and software engineering:

### In Data Structures & Algorithms

| Application | How Queue is Used |
|-------------|-------------------|
| **BFS (Breadth-First Search)** | Explore graph level by level |
| **Level-order tree traversal** | Visit tree nodes level by level |
| **Topological sorting** | Kahn's algorithm uses a queue |
| **Sliding window problems** | Maintain a window of elements |

### In Operating Systems

| Application | How Queue is Used |
|-------------|-------------------|
| **CPU Scheduling** | Ready queue for processes |
| **Disk Scheduling** | Request queue for I/O |
| **Print Spooling** | Jobs wait in print queue |
| **Interrupt Handling** | Interrupts processed in order |

### In Networking

| Application | How Queue is Used |
|-------------|-------------------|
| **Packet Routing** | Packets buffered in queues |
| **Message Queuing** | Async communication between services |
| **Rate Limiting** | Request queues with throttling |

### In Application Development

| Application | How Queue is Used |
|-------------|-------------------|
| **Event Loop** | Events processed in order (JS, UI) |
| **Task Queues** | Background job processing |
| **Caching (LRU)** | Track recently used items |
| **Streaming** | Buffer data between producer/consumer |

---

## 6. When to Use Queues

### Decision Framework

Ask yourself these questions:

```
  Do you need ordered processing?
         │
         ├── YES → Do you need FIFO order?
         │           │
         │           ├── YES → Do you need both-end access?
         │           │           │
         │           │           ├── YES → Use DEQUE
         │           │           │
         │           │           └── NO → Use QUEUE ✓
         │           │
         │           └── NO → Do you need priority-based order?
         │                       │
         │                       ├── YES → Use PRIORITY QUEUE
         │                       │
         │                       └── NO → Use STACK (LIFO)
         │
         └── NO → Do you need key-based access?
                     │
                     ├── YES → Use MAP / HASH TABLE
                     │
                     └── NO → Use ARRAY / LIST
```

### Queue vs Other Data Structures

| Criteria | Queue | Stack | Array | Linked List |
|----------|:-----:|:-----:|:-----:|:-----------:|
| FIFO order | ✅ | ❌ | ❌ | ❌ |
| LIFO order | ❌ | ✅ | ❌ | ❌ |
| Random access | ❌ | ❌ | ✅ | ❌ |
| Insert at both ends | ❌* | ❌ | ✅ | ✅ |
| O(1) insert/remove | ✅ | ✅ | ❌** | ✅ |

> *Deque supports both ends. **Array insert at front is O(n).

### Use Queue When:

1. **Processing order matters** — Tasks must be handled in arrival order.
2. **Producer-Consumer pattern** — One side produces, another consumes.
3. **Level-by-level exploration** — BFS, level-order traversal.
4. **Buffering** — Data arrives faster than it can be processed.
5. **Fairness is required** — First come, first served.

### Do NOT Use Queue When:

1. **You need random access** — Use an array or hash map.
2. **You need LIFO order** — Use a stack.
3. **You need sorted order** — Use a priority queue or sorted structure.
4. **You need to search** — Use a hash set or BST.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Queue | Linear data structure with FIFO ordering |
| FIFO | First In, First Out — insertion order = removal order |
| Enqueue | Add element at the rear |
| Dequeue | Remove element from the front |
| Peek | View front element without removing |
| ADT | Defines operations: enqueue, dequeue, peek, isEmpty, size |
| Underflow | Attempting dequeue/peek on an empty queue |
| Overflow | Attempting enqueue on a full bounded queue |
| Key Use Cases | BFS, scheduling, buffering, producer-consumer |

---

## Quick Revision Questions

1. **What is the fundamental ordering principle of a queue, and how does it differ from a stack?**

2. **List the five core operations of the Queue ADT and their time complexities.**

3. **What happens when you try to dequeue from an empty queue? What is this condition called?**

4. **Give three real-world examples where a queue model naturally applies.**

5. **In a BFS graph traversal, why is a queue used instead of a stack?**

6. **When would you choose a priority queue over a regular queue? Give an example scenario.**

---

## Answers (Hidden — Think First!)

<details>
<summary>Click to reveal answers</summary>

1. **FIFO (First In, First Out)** — the first element added is the first removed. A stack uses **LIFO (Last In, First Out)** — the last element added is the first removed.

2. `enqueue(item)` — O(1), `dequeue()` — O(1), `peek()` — O(1), `isEmpty()` — O(1), `size()` — O(1). All core operations are constant time.

3. It causes an **underflow** condition. Typically, an exception is thrown or an error value is returned.

4. Ticket counter line, printer queue, call center hold queue, traffic at a signal, CPU ready queue — any three of these.

5. BFS explores nodes **level by level**, visiting all neighbors before going deeper. A queue ensures nodes are processed in the order they were discovered (FIFO). A stack would cause DFS (depth-first), going deep before wide.

6. When elements have different urgency levels. Example: **hospital emergency room** — critical patients are treated before minor cases regardless of arrival order.

</details>

---

[← Back to README](../README.md) | [Next: Queue Implementation →](../02-Queue-Implementation/01-queue-implementation.md)
