# 1.1 What is a Heap?

[← Back to README](../README.md) | [Next: Heap Property →](02-heap-property.md)

---

## Overview

Before diving into heap operations and algorithms, you need a rock-solid understanding of **what a heap is**, **why it exists**, and **how it is stored in memory**. This topic introduces the heap as a data structure and its core purpose.

---

## Definition

A **heap** is a specialized **tree-based** data structure that satisfies the **heap property**. It is one of the most efficient ways to implement a **priority queue** — a data structure where the highest (or lowest) priority element is always accessible in O(1) time.

---

## Core Idea

> A heap lets you always know what the **most important** element is, without sorting everything.

Think of a hospital emergency room:
- Patients don't wait in a simple queue (FIFO)
- The most critical patient is treated first
- A heap models this: the "most urgent" item is always at the top

---

## Key Properties

1. It is a **complete binary tree** (covered in detail in [Topic 3](03-complete-binary-tree.md))
2. It satisfies the **heap property** — min or max (covered in [Topic 2](02-heap-property.md))
3. It is typically stored as an **array**, not as linked nodes (covered in [Topic 4](04-array-representation.md))

---

## Real-World Applications of Heaps

| Application | How Heaps Help |
|-------------|---------------|
| **OS Process Scheduling** | Highest-priority process runs first |
| **Dijkstra's Algorithm** | Extract nearest unvisited node |
| **Huffman Coding** | Build optimal prefix codes |
| **Median Maintenance** | Two heaps track running median |
| **Event-Driven Simulation** | Process earliest event first |
| **Load Balancing** | Assign task to least-loaded server |
| **Merge K Sorted Lists** | Always extend the smallest element |

---

## Quick Check

1. **What type of tree is a heap based on?**
   <details><summary>Answer</summary>
   A complete binary tree.
   </details>

2. **What abstract data type does a heap most efficiently implement?**
   <details><summary>Answer</summary>
   A priority queue — where the highest (or lowest) priority element is always accessible in O(1).
   </details>

---

[← Back to README](../README.md) | [Next: Heap Property →](02-heap-property.md)
