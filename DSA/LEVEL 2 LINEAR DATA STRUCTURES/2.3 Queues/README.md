# 📚 Queues — Comprehensive Study Notes

> **Level 2: Linear Data Structures | Module 2.3**
> A complete guide to Queues — from fundamentals to advanced patterns.

---

## Course Overview

Queues are one of the most fundamental data structures in computer science. They follow the **First-In, First-Out (FIFO)** principle and appear everywhere — from operating system scheduling to graph traversal algorithms. This course covers queue concepts from the ground up, building toward advanced patterns like monotonic queues and real-world applications.

```
  ENQUEUE                                              DEQUEUE
  -------->  +------+------+------+------+------+  -------->
             | A    | B    | C    | D    | E    |
  (Rear)     +------+------+------+------+------+     (Front)
             Newest                        Oldest
```

### What You'll Learn

- Core queue concepts and the FIFO principle
- Multiple implementation strategies (array, circular, linked list)
- Specialized queue variants (deque, priority queue, monotonic queue)
- Classic queue-based problems and problem-solving patterns
- Real-world applications of queues

### Prerequisites

- Basic understanding of arrays and linked lists
- Familiarity with time/space complexity (Big-O notation)
- Comfort with basic programming constructs (loops, conditionals)

---

## Complete Table of Contents

### [Unit 1: Queue Fundamentals](01-Queue-Fundamentals/01-queue-fundamentals.md)

| # | Topic | Description |
|---|-------|-------------|
| 1 | What is a Queue? | Definition, structure, and core idea |
| 2 | FIFO Principle | First-In, First-Out explained |
| 3 | Queue ADT | Abstract Data Type — operations and contract |
| 4 | Real-World Analogies | Intuitive examples from everyday life |
| 5 | Queue Applications Overview | Where queues are used |
| 6 | When to Use Queues | Decision framework for choosing queues |

---

### [Unit 2: Queue Implementation](02-Queue-Implementation/01-queue-implementation.md)

| # | Topic | Description |
|---|-------|-------------|
| 1 | Array-Based Queue | Simple linear array implementation |
| 2 | Circular Queue | Wrapping around to reuse space |
| 3 | Linked List-Based Queue | Dynamic memory implementation |
| 4 | Queue Operations Complexity | Time and space analysis |
| 5 | Handling Overflow/Underflow | Edge cases and error handling |
| 6 | Implementation Trade-offs | Comparing all approaches |

---

### [Unit 3: Circular Queue](03-Circular-Queue/01-circular-queue.md)

| # | Topic | Description |
|---|-------|-------------|
| 1 | Why Circular Queue? | The problem with linear queues |
| 2 | Implementation Details | Step-by-step circular queue code |
| 3 | Full vs Empty Detection | The classic circular queue challenge |
| 4 | Circular Array Technique | Modular arithmetic for wrap-around |
| 5 | Applications | Real-world circular queue uses |
| 6 | Ring Buffer Concept | Circular queue in systems programming |

---

### [Unit 4: Deque (Double-Ended Queue)](04-Deque/01-deque.md)

| # | Topic | Description |
|---|-------|-------------|
| 1 | What is a Deque? | Definition and structure |
| 2 | Operations (Front, Rear) | Insert/delete from both ends |
| 3 | Implementation Approaches | Array-based and linked list-based |
| 4 | Input-Restricted Deque | Insert at one end only |
| 5 | Output-Restricted Deque | Delete from one end only |
| 6 | Sliding Window Maximum | Classic deque application |

---

### [Unit 5: Priority Queue](05-Priority-Queue/01-priority-queue.md)

| # | Topic | Description |
|---|-------|-------------|
| 1 | Priority Queue Concept | Ordering by priority, not arrival |
| 2 | Implementation Approaches | Arrays, linked lists, heaps |
| 3 | Min-Priority vs Max-Priority | Ascending vs descending order |
| 4 | Use Cases | Real-world priority queue scenarios |
| 5 | Heap-Based Implementation (Intro) | Why heaps are ideal |
| 6 | Applications in Algorithms | Dijkstra's, Huffman, etc. |

---

### [Unit 6: Queue Problems](06-Queue-Problems/01-queue-problems.md)

| # | Topic | Description |
|---|-------|-------------|
| 1 | Queue Using Stacks | Simulating FIFO with two stacks |
| 2 | Stack Using Queues | Simulating LIFO with two queues |
| 3 | First Non-Repeating Character | Streaming character problem |
| 4 | Circular Tour Problem | Petrol pump / gas station |
| 5 | LRU Cache Concept | Least Recently Used eviction |
| 6 | Generate Binary Numbers | 1 to N binary generation |

---

### [Unit 7: Monotonic Queue](07-Monotonic-Queue/01-monotonic-queue.md)

| # | Topic | Description |
|---|-------|-------------|
| 1 | What is a Monotonic Queue? | Maintaining order invariants |
| 2 | Sliding Window Maximum | Classic monotonic deque problem |
| 3 | Sliding Window Minimum | Dual of maximum problem |
| 4 | Jump Game Optimization | DP optimization with queues |
| 5 | Constrained Subsequence Sum | Advanced DP + monotonic queue |
| 6 | Pattern Recognition | When to use monotonic queues |

---

### [Unit 8: Queue Applications](08-Queue-Applications/01-queue-applications.md)

| # | Topic | Description |
|---|-------|-------------|
| 1 | BFS Traversal (Preview) | Breadth-first search with queues |
| 2 | Level Order Traversal (Preview) | Tree level-by-level traversal |
| 3 | Process Scheduling | OS-level queue usage |
| 4 | Print Queue | Printer spooling simulation |
| 5 | Message Queues | Distributed systems messaging |
| 6 | Task Scheduling | Job and task scheduling |

---

## Quick Reference — Complexity Summary

| Operation | Array Queue | Circular Queue | Linked List Queue |
|-----------|:-----------:|:--------------:|:-----------------:|
| Enqueue   | O(1)        | O(1)           | O(1)              |
| Dequeue   | O(n)*       | O(1)           | O(1)              |
| Peek      | O(1)        | O(1)           | O(1)              |
| isEmpty   | O(1)        | O(1)           | O(1)              |
| Space     | O(n)        | O(n)           | O(n)              |

> *O(n) for naive array queue due to shifting; O(1) amortized with front pointer.

---

## How to Use These Notes

1. **Sequential Study** — Work through units 1-8 in order for a complete understanding.
2. **Quick Review** — Use summary tables and revision questions at the end of each chapter.
3. **Problem Practice** — Unit 6 and Unit 7 focus on problem-solving patterns.
4. **Reference** — Use the complexity table above and unit-specific tables for quick lookups.

---

## Navigation

| Unit | Topic | Link |
|------|-------|------|
| 1 | Queue Fundamentals | [Go →](01-Queue-Fundamentals/01-queue-fundamentals.md) |
| 2 | Queue Implementation | [Go →](02-Queue-Implementation/01-queue-implementation.md) |
| 3 | Circular Queue | [Go →](03-Circular-Queue/01-circular-queue.md) |
| 4 | Deque | [Go →](04-Deque/01-deque.md) |
| 5 | Priority Queue | [Go →](05-Priority-Queue/01-priority-queue.md) |
| 6 | Queue Problems | [Go →](06-Queue-Problems/01-queue-problems.md) |
| 7 | Monotonic Queue | [Go →](07-Monotonic-Queue/01-monotonic-queue.md) |
| 8 | Queue Applications | [Go →](08-Queue-Applications/01-queue-applications.md) |

---

*Happy Learning! 🚀*
