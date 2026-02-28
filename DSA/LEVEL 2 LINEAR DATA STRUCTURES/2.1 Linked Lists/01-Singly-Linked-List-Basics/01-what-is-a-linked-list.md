# Chapter 1: What is a Linked List? / Node Structure

[← Back to README](../README.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Head, Tail, and Traversal →](02-head-tail-and-traversal.md)

---

## Chapter Overview

This chapter introduces the **linked list** data structure from first principles. You will understand **why** linked lists exist, **how** they differ from contiguous memory structures, and the fundamental building block — the **Node**.

---

## 1.1 What is a Linked List?

A **linked list** is a linear data structure where elements are stored in **non-contiguous memory locations** and connected via **pointers** (references).

Unlike arrays, where elements sit side-by-side in memory, each element in a linked list can live *anywhere* in memory. The "link" between elements is maintained by each element storing the **address of the next element**.

### Analogy: Treasure Hunt

Imagine a treasure hunt where each clue tells you where to find the **next** clue:

```
Clue 1           Clue 2           Clue 3           Treasure!
"Go to           "Go to           "Go to           "You 
 the park"  -->   the library" -->  the cafe"  -->   found it!"
(Location A)     (Location B)     (Location C)     (Location D)
```

Each clue is at a **different location** (non-contiguous), and each one **points to the next**. This is exactly how a linked list works!

---

## 1.2 The Node — Building Block of a Linked List

Every linked list is made up of **nodes**. A node is a container that holds:

1. **Data** — the actual value/information
2. **Next pointer** — the address of the next node in the list

### ASCII Diagram: A Single Node

```
     ┌──────────┬──────────┐
     │   DATA   │   NEXT   │
     │  (value) │ (address)│
     └──────────┴──────────┘
```

### Node in Memory

```
     Address: 0x1A4

     ┌──────────┬──────────┐
     │    42    │  0x2B8   │──── points to next node at address 0x2B8
     └──────────┴──────────┘
```

### Pseudocode: Node Structure

```
class Node:
    data       ← value to store
    next       ← pointer to next Node (initially NULL)

    constructor(value):
        data = value
        next = NULL
```

### Code (C-style):
```c
struct Node {
    int data;
    struct Node* next;
};
```

### Code (Python-style):
```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None
```

---

## 1.3 Visualizing a Complete Linked List

A linked list is a **chain of nodes**, where each node points to the next:

```
  HEAD
   │
   ▼
 ┌──────┬───┐    ┌──────┬───┐    ┌──────┬───┐    ┌──────┬──────┐
 │  10  │ ──┼───►│  20  │ ──┼───►│  30  │ ──┼───►│  40  │ NULL │
 └──────┴───┘    └──────┴───┘    └──────┴───┘    └──────┴──────┘
  Node 0          Node 1          Node 2          Node 3
  (0x100)         (0x250)         (0x180)         (0x300)
```

**Key observations:**
- `HEAD` is a pointer that stores the address of the **first node**
- Each node's `next` stores the address of the **following node**
- The **last node** has `next = NULL` — this signals the end of the list
- Memory addresses are **not sequential** (0x100, 0x250, 0x180, 0x300)

### Empty Linked List

```
  HEAD ──► NULL

  (Head is NULL — the list has no nodes)
```

### Single-Element Linked List

```
  HEAD
   │
   ▼
 ┌──────┬──────┐
 │  10  │ NULL │
 └──────┴──────┘
```

---

## 1.4 How Nodes are Created in Memory

When you create a node, the system **dynamically allocates memory** on the heap:

### Step-by-Step: Building a 3-Node List

```
Step 1: Create Node with data = 10
        Memory allocates space at address 0x100

        HEAD ──► [10 | NULL]   (0x100)

Step 2: Create Node with data = 20
        Memory allocates space at address 0x250

        HEAD ──► [10 | 0x250] ──► [20 | NULL]   (0x250)

Step 3: Create Node with data = 30
        Memory allocates space at address 0x180

        HEAD ──► [10 | 0x250] ──► [20 | 0x180] ──► [30 | NULL]   (0x180)
```

**Notice:** The addresses (0x100, 0x250, 0x180) are NOT contiguous — they can be anywhere in memory!

---

## 1.5 Types of Linked Lists (Overview)

| Type | Description | Diagram |
|------|-------------|---------|
| **Singly Linked** | Each node points to the next | `[A]──►[B]──►[C]──►NULL` |
| **Doubly Linked** | Each node points to next AND previous | `NULL◄──[A]⇄[B]⇄[C]──►NULL` |
| **Circular Singly** | Last node points back to first | `[A]──►[B]──►[C]──┐` `↑←←←←←←←←←←←←←←←←┘` |
| **Circular Doubly** | Doubly linked + last connects to first | `┌►[A]⇄[B]⇄[C]◄┐` `└←←←←←←←←←←←←←←←┘` |

> We will cover each type in detail in later units. This unit focuses on **Singly Linked Lists**.

---

## 1.6 Why Do Linked Lists Exist?

### The Problem with Arrays

Arrays have a fundamental limitation: they require **contiguous memory allocation**.

```
Array of 5 elements (contiguous):

  ┌─────┬─────┬─────┬─────┬─────┐
  │  10 │  20 │  30 │  40 │  50 │
  └─────┴─────┴─────┴─────┴─────┘
  0x100  0x104  0x108  0x10C  0x110

  (Each element is exactly 4 bytes apart)
```

**Problems:**
1. **Fixed size** — Must decide size at creation (in static arrays)
2. **Expensive insertions/deletions** — Shifting elements costs O(n)
3. **Memory waste** — Unused allocated slots waste memory
4. **Contiguous requirement** — May not find a large enough continuous block

### How Linked Lists Solve These

| Problem | Array | Linked List |
|---------|-------|-------------|
| Fixed size | Pre-allocated, resize is expensive | Grows/shrinks dynamically |
| Insert at beginning | O(n) — shift all elements | O(1) — just change pointers |
| Delete from middle | O(n) — shift elements | O(1) — once node is found |
| Memory waste | Possible (unused slots) | No waste — allocate per node |
| Contiguous memory | Required | Not required |

---

## 1.7 The Cost of Flexibility

Linked lists trade **random access** for **dynamic flexibility**:

```
   Array: Want element at index 3?
   ───────────────────────────────
   base_address + (3 × element_size) = Direct access!    → O(1)

   Linked List: Want element at index 3?
   ───────────────────────────────
   Start at HEAD → go to next → go to next → go to next  → O(n)
   (Must traverse from the beginning!)
```

**Extra memory overhead per node:**
```
   Array element:     | 4 bytes (data only) |
   
   Linked list node:  | 4 bytes (data) | 4-8 bytes (pointer) |
                      
   ← Each node uses roughly 2x memory due to the pointer!
```

---

## 1.8 Real-World Applications

1. **Music playlists** — songs linked in sequence, easy to add/remove
2. **Browser history** — back/forward navigation (doubly linked)
3. **Undo/Redo** — chain of states
4. **Memory management** — OS free lists track available memory blocks
5. **Polynomial representation** — each term is a node
6. **Hash table chaining** — collision resolution using linked lists
7. **Graph adjacency lists** — representing graph edges

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Linked List | Linear structure with non-contiguous nodes connected by pointers |
| Node | Contains `data` + `next` pointer |
| HEAD | Pointer to the first node |
| NULL | Marks the end of the list |
| Dynamic sizing | Grows/shrinks at runtime — no pre-allocation needed |
| Memory layout | Nodes scattered in memory, connected via addresses |
| Trade-off | No random access (O(n)) but efficient insert/delete (O(1)) |
| Types | Singly, Doubly, Circular Singly, Circular Doubly |

---

## Quick Revision Questions

1. **What are the two components of a node in a singly linked list?**
   <details><summary>Answer</summary>Data (the stored value) and Next (pointer to the next node).</details>

2. **What does the HEAD pointer represent? What happens if HEAD is NULL?**
   <details><summary>Answer</summary>HEAD points to the first node of the list. If HEAD is NULL, the list is empty.</details>

3. **How does a linked list indicate the end of the list?**
   <details><summary>Answer</summary>The last node's `next` pointer is set to NULL.</details>

4. **Why can't you access the 5th element of a linked list directly?**
   <details><summary>Answer</summary>Linked list nodes are stored in non-contiguous memory. There's no formula to calculate the address of the 5th node — you must traverse from HEAD through each node sequentially.</details>

5. **What extra memory does a linked list node use compared to an array element?**
   <details><summary>Answer</summary>Each node requires additional memory for the `next` pointer (4 bytes on 32-bit, 8 bytes on 64-bit systems).</details>

6. **Name three real-world applications of linked lists.**
   <details><summary>Answer</summary>Music playlists, browser history (back/forward), undo/redo functionality, hash table chaining, OS memory management free lists.</details>

---

[← Back to README](../README.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Head, Tail, and Traversal →](02-head-tail-and-traversal.md)
