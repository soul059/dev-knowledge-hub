# Chapter 1: DLL Structure and Advantages

[← Previous: Edge Cases and Complexity](../02-Singly-Linked-List-Operations/03-edge-cases-and-complexity.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: DLL Insertion and Deletion →](02-dll-insertion-and-deletion.md)

---

## Chapter Overview

The **Doubly Linked List (DLL)** extends the singly linked list by adding a **previous pointer** to each node. This enables **bidirectional traversal** and simplifies several operations. This chapter covers the structure, advantages, and conceptual foundation.

---

## 1.1 Node Structure

A DLL node has **three** components:

```
     ┌──────────┬──────────┬──────────┐
     │   PREV   │   DATA   │   NEXT   │
     │(address) │  (value) │(address) │
     └──────────┴──────────┴──────────┘
```

### Pseudocode

```
class DLLNode:
    prev       ← pointer to previous node
    data       ← value stored
    next       ← pointer to next node

    constructor(value):
        prev = NULL
        data = value
        next = NULL
```

### Code (C-style)

```c
struct DLLNode {
    struct DLLNode* prev;
    int data;
    struct DLLNode* next;
};
```

### Code (Python)

```python
class DLLNode:
    def __init__(self, data):
        self.prev = None
        self.data = data
        self.next = None
```

---

## 1.2 Visualizing a Doubly Linked List

```
   HEAD                                                    TAIL
    │                                                       │
    ▼                                                       ▼
  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
  │NULL│ 10 │  ──┼────►│◄── │ 20 │  ──┼────►│◄── │ 30 │NULL│
  └──────────────┘     └──────────────┘     └──────────────┘

  Simplified notation:
  NULL ◄── [10] ⇄ [20] ⇄ [30] ──► NULL
            ↑                        ↑
           HEAD                    TAIL
```

### Each Link is Bidirectional

```
      Node A               Node B
  ┌────┬────┬────┐     ┌────┬────┬────┐
  │prev│ 10 │next│     │prev│ 20 │next│
  └────┴────┴────┘     └────┴────┴────┘
         │    │              │    │
         │    └──────────────┘    │
         │         B.prev = A     │
         └────────────────────────┘
               A.next = B

  A.next  = address of B
  B.prev  = address of A
```

---

## 1.3 Empty and Single-Node DLL

### Empty DLL

```
  HEAD ──► NULL
  TAIL ──► NULL
```

### Single-Node DLL

```
  HEAD    TAIL
   │       │
   ▼       ▼
 ┌────┬────┬────┐
 │NULL│ 10 │NULL│
 └────┴────┴────┘

 • HEAD = TAIL = the only node
 • node.prev = NULL (no predecessor)
 • node.next = NULL (no successor)
```

---

## 1.4 SLL vs DLL: Structural Comparison

```
  Singly Linked List (SLL):

  HEAD
   │
   ▼
 ┌────┬────┐   ┌────┬────┐   ┌────┬────┐
 │ 10 │ ───┼──►│ 20 │ ───┼──►│ 30 │NULL│
 └────┴────┘   └────┴────┘   └────┴────┘
  8 bytes/node  8 bytes/node  8 bytes/node
  (4 data + 4 ptr)


  Doubly Linked List (DLL):

  HEAD                                    TAIL
   │                                       │
   ▼                                       ▼
 ┌────┬────┬────┐ ┌────┬────┬────┐ ┌────┬────┬────┐
 │NULL│ 10 │ ───┼►│◄───│ 20 │ ───┼►│◄───│ 30 │NULL│
 └────┴────┴────┘ └────┴────┴────┘ └────┴────┴────┘
  12 bytes/node    12 bytes/node    12 bytes/node
  (4 data + 4+4 ptrs)               (on 32-bit)
```

---

## 1.5 Advantages of DLL over SLL

| Advantage | Explanation |
|-----------|-------------|
| **Bidirectional traversal** | Can traverse forward AND backward |
| **O(1) delete with node reference** | Given a node pointer, delete in O(1) — no need for `prev` tracking |
| **O(1) delete from end** | With TAIL pointer, go to last and use `prev` |
| **Easier reverse traversal** | Start from TAIL, follow `prev` pointers |
| **Simpler insertion before a node** | Have direct access to the predecessor |
| **Better for LRU Cache** | Quick removal and reinsertion at head |

### Key Advantage: O(1) Deletion

```
  SLL — Delete Node B (given pointer to B):
  ───────────────────────────────────────────
  Problem: Need A to update A.next, but can't reach A from B!
  
  [A] ──► [B] ──► [C]
           ↑
     given this, but need A
  
  Must traverse from HEAD to find A → O(n)


  DLL — Delete Node B (given pointer to B):
  ───────────────────────────────────────────
  [A] ⇄ [B] ⇄ [C]
          ↑
    B.prev = A (direct access!)
    B.next = C (direct access!)
  
  A.next = C
  C.prev = A
  free(B)
  → O(1)!
```

---

## 1.6 Disadvantages of DLL

| Disadvantage | Explanation |
|-------------|-------------|
| **Extra memory** | Each node needs an additional pointer (~4-8 bytes) |
| **More complex code** | Every insert/delete must update 2 pointers per link |
| **More bugs** | Easy to forget updating `prev` → corrupted list |
| **Slightly slower** | Extra pointer assignment per operation |

### Memory Comparison (64-bit system, int data)

```
  SLL Node:   4 (data) + 8 (next)              = 12 bytes
  DLL Node:   4 (data) + 8 (next) + 8 (prev)   = 20 bytes

  For 1000 nodes:
    SLL: ~12,000 bytes
    DLL: ~20,000 bytes    ← 67% more memory!
```

---

## 1.7 When to Use DLL

### Use DLL When:

```
✓ Need bidirectional traversal (e.g., browser back/forward)
✓ Frequent deletion by node reference (e.g., LRU cache)
✓ Need to delete from the end in O(1) (with TAIL)
✓ Implementing deque (double-ended queue)
✓ Need to insert before a given node in O(1)
✓ Undo/redo functionality
```

### Stick with SLL When:

```
✓ Memory is constrained
✓ Only need forward traversal
✓ Simpler code is preferred
✓ Implementing stack or simple queue
```

---

## 1.8 DLL Invariants (Rules That Must Always Hold)

```
For any two adjacent nodes A and B:

  A.next == B  implies  B.prev == A

  ┌────┬────┬────┐     ┌────┬────┬────┐
  │    │  A │  ──┼────►│◄── │  B │    │
  └────┴────┴────┘     └────┴────┴────┘
       A.next = B       B.prev = A

For HEAD:
  HEAD.prev == NULL

For TAIL:
  TAIL.next == NULL

These must ALWAYS be true after every operation.
Breaking these invariants = corrupted list!
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| DLL Node | Has `prev`, `data`, and `next` |
| Bidirectional | Can traverse forward and backward |
| O(1) deletion | Given node pointer, can delete without traversal |
| Memory cost | ~50-67% more memory than SLL per node |
| Code complexity | Every link update requires 2 pointer changes |
| Invariant | If `A.next = B` then `B.prev = A` (always!) |
| Best use cases | LRU cache, deque, browser history, undo/redo |

---

## Quick Revision Questions

1. **How many pointers does a DLL node contain? Name them.**
   <details><summary>Answer</summary>Two pointers: `prev` (points to previous node) and `next` (points to next node).</details>

2. **Why can a DLL delete a node in O(1) given just a pointer to that node, while SLL cannot?**
   <details><summary>Answer</summary>In a DLL, the node's `prev` pointer gives direct access to the predecessor. You can update `prev.next` and `next.prev` without traversal. In SLL, you have no way to reach the predecessor without traversing from HEAD.</details>

3. **What is the DLL invariant that must hold for adjacent nodes?**
   <details><summary>Answer</summary>If `A.next == B`, then `B.prev == A`. Both directions must be consistent after every operation.</details>

4. **How much extra memory does a DLL use compared to SLL on a 64-bit system?**
   <details><summary>Answer</summary>8 extra bytes per node for the `prev` pointer. For 1000 nodes with int data: SLL ≈ 12KB, DLL ≈ 20KB (67% more).</details>

5. **Name three real-world applications where DLL is preferred over SLL.**
   <details><summary>Answer</summary>1) Browser back/forward navigation, 2) LRU cache implementation, 3) Undo/redo functionality in text editors.</details>

6. **What are `HEAD.prev` and `TAIL.next` in a standard DLL?**
   <details><summary>Answer</summary>Both are NULL. HEAD has no predecessor, TAIL has no successor.</details>

---

[← Previous: Edge Cases and Complexity](../02-Singly-Linked-List-Operations/03-edge-cases-and-complexity.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: DLL Insertion and Deletion →](02-dll-insertion-and-deletion.md)
