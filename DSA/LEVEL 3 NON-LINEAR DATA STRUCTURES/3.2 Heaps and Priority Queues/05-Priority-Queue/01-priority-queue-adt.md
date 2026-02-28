# 5.1 Priority Queue — The Abstract Data Type

[← Previous Unit: Heap Sort](../04-Heap-Sort/06-variations-and-implementation.md) | [Next: Min & Max Priority Queue →](02-min-and-max-priority-queue.md)

---

## Overview

A **Priority Queue** is an abstract data type where each element has a "priority" and the element with the highest (or lowest) priority is served first. The **heap** is the most common and efficient implementation.

---

## What Is It?

A priority queue supports these operations:

| Operation | Description |
|-----------|-------------|
| `insert(element, priority)` | Add an element with given priority |
| `extract_top()` | Remove and return the highest-priority element |
| `peek()` | View the highest-priority element without removing |
| `is_empty()` | Check if the queue is empty |

---

## Priority Queue vs Regular Queue vs Stack

```
Regular Queue (FIFO):     Stack (LIFO):        Priority Queue:
                          
 IN → [A][B][C] → OUT    IN/OUT → [C]          IN → [B:3][A:1][C:5] → OUT(C:5)
                                   [B]                 ↑
 First in, first out.              [A]          Highest priority out.
                          Last in, first out.
```

---

## Possible Implementations

| Implementation | Insert | Extract | Peek |
|----------------|--------|---------|------|
| Unsorted array | O(1) | O(n) | O(n) |
| Sorted array | O(n) | O(1) | O(1) |
| Unsorted linked list | O(1) | O(n) | O(n) |
| Sorted linked list | O(n) | O(1) | O(1) |
| **Binary heap** | **O(log n)** | **O(log n)** | **O(1)** |
| BST (balanced) | O(log n) | O(log n) | O(log n) |

> **Binary heap** gives the best balance: O(log n) for both insert and extract, O(1) for peek.

---

## Quick Check

1. **Why is a binary heap preferred over a sorted array for implementing a priority queue?**
   <details><summary>Answer</summary>
   A sorted array gives O(1) extract but O(n) insert (shifting elements). A binary heap gives O(log n) for both operations — a better trade-off when both are frequent.
   </details>

---

[← Previous Unit: Heap Sort](../04-Heap-Sort/06-variations-and-implementation.md) | [Next: Min & Max Priority Queue →](02-min-and-max-priority-queue.md)
