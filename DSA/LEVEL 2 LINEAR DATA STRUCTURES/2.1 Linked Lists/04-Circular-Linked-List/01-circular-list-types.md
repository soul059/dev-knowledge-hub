# Chapter 1: Singly and Doubly Circular Lists

[← Previous: DLL Traversal and Trade-offs](../03-Doubly-Linked-List/03-dll-traversal-and-tradeoffs.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Operations on Circular Lists →](02-circular-list-operations.md)

---

## Chapter Overview

A **Circular Linked List** is a variation where the **last node connects back to the first node** instead of pointing to NULL. This chapter covers both singly and doubly circular variants, their structure, and when they are useful.

---

## 1.1 What is a Circular Linked List?

In a standard linked list, the last node's `next` is NULL. In a circular linked list, the last node's `next` points **back to the first node**, forming a loop.

### Standard vs Circular

```
  Standard Singly Linked List:
  ────────────────────────────
  HEAD ──► [10] ──► [20] ──► [30] ──► NULL
                                       ↑
                                  Terminates here

  Circular Singly Linked List:
  ────────────────────────────
  HEAD ──► [10] ──► [20] ──► [30] ──┐
            ↑                        │
            └────────────────────────┘
                  Last node → First node
```

**Key Property:** There is NO NULL pointer — the list forms a continuous ring.

---

## 1.2 Singly Circular Linked List (SCLL)

### Structure

Each node has `data` and `next`, where the last node's `next` = HEAD.

```
          ┌─────────────────────────────────┐
          │                                 │
          ▼                                 │
        ┌──────┬───┐  ┌──────┬───┐  ┌──────┬───┐
        │  10  │ ──┼─►│  20  │ ──┼─►│  30  │ ──┼──┘
        └──────┴───┘  └──────┴───┘  └──────┴───┘
          ↑
         HEAD

  Or visualized as a ring:

          ┌──► [10] ──► [20] ──► [30] ──┐
          │                              │
          └──────────────────────────────┘
```

### Using TAIL Instead of HEAD

It's often more efficient to maintain a **TAIL pointer** (pointing to the last node) rather than HEAD:

```
  With TAIL pointer:
  
         TAIL
          │
          ▼
  ┌──► [10] ──► [20] ──► [30] ──┐
  │                              │
  └──────────────────────────────┘

  HEAD = TAIL.next    (can derive HEAD from TAIL!)
  
  Benefits:
  • Insert at beginning: O(1)  — insert after TAIL, update TAIL.next
  • Insert at end:       O(1)  — insert after TAIL, update TAIL
  • Both without traversal!
```

---

## 1.3 Doubly Circular Linked List (DCLL)

### Structure

Each node has `prev`, `data`, and `next`. The last node's `next` = HEAD, and HEAD's `prev` = TAIL.

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │   ┌────────────────────────────────────────────────┐     │
  │   │                                                │     │
  │   ▼                                                │     │
  │ ┌────┬──────┬────┐  ┌────┬──────┬────┐  ┌────┬──────┬────┐
  └─┤prev│  10  │next├─►│prev│  20  │next├─►│prev│  30  │next├─┘
    └────┴──────┴────┘  └────┴──────┴────┘  └────┴──────┴────┘
      ↑        │          ↑        │          ↑        │
      │        └──────────┘        └──────────┘        │
      │                                                │
      └────────────────────────────────────────────────┘
      
  Simplified ring view:
  
        ┌──► [10] ⇄ [20] ⇄ [30] ──┐
        │                           │
        └───◄───────────────────◄───┘
              (both directions!)
```

### Key Properties of DCLL

```
  HEAD.prev = TAIL (last node)
  TAIL.next = HEAD (first node)

  For ANY node X:
    X.next.prev = X
    X.prev.next = X
```

---

## 1.4 How to Identify the "End" of a Circular List

Since there's no NULL, how do you know when you've traversed the entire list?

### Method 1: Start Point Check

```
function traverse(HEAD):
    if HEAD == NULL: return         ← Empty list
    
    current = HEAD
    do:
        print(current.data)
        current = current.next
    while current ≠ HEAD            ← Stop when we return to start!
```

**Notice:** We use a **do-while** loop (process first, then check). A regular while loop would never enter because `current == HEAD` is immediately true.

### Method 2: Using TAIL

```
function traverse(TAIL):
    if TAIL == NULL: return
    
    current = TAIL.next             ← Start at HEAD (= TAIL.next)
    while true:
        print(current.data)
        if current == TAIL:
            break                   ← Reached the last node
        current = current.next
```

---

## 1.5 Empty and Single-Node Circular Lists

### Empty Circular List

```
  HEAD ──► NULL   (or TAIL ──► NULL)
  
  Same as regular — no nodes exist
```

### Single-Node Singly Circular

```
           ┌───┐
           │   │
           ▼   │
         [10]──┘
           ↑
          HEAD

  node.next = node (points to itself!)
```

### Single-Node Doubly Circular

```
         ┌────┐
         │    │
         ▼    │
  ┌──── [10] ─┘
  │       ↑
  └───────┘

  node.next = node
  node.prev = node
  (Both point to itself!)
```

---

## 1.6 Comparison of All Linked List Types

### Structural Comparison

```
  SLL:    [A] ──► [B] ──► [C] ──► NULL

  DLL:    NULL ◄── [A] ⇄ [B] ⇄ [C] ──► NULL

  SCLL:   ┌──► [A] ──► [B] ──► [C] ──┐
          └───────────────────────────┘

  DCLL:   ┌──► [A] ⇄ [B] ⇄ [C] ──┐
          └◄──────────────────────◄┘
```

### Feature Comparison Table

| Feature | SLL | DLL | SCLL | DCLL |
|---------|-----|-----|------|------|
| Next pointer | ✓ | ✓ | ✓ | ✓ |
| Prev pointer | ✗ | ✓ | ✗ | ✓ |
| NULL termination | ✓ | ✓ | ✗ | ✗ |
| Circular | ✗ | ✗ | ✓ | ✓ |
| Forward traversal | ✓ | ✓ | ✓ | ✓ |
| Backward traversal | ✗ | ✓ | ✗ | ✓ |
| Memory per node (32-bit) | 8B | 12B | 8B | 12B |
| Insert at front | O(1) | O(1) | O(1) | O(1) |
| Insert at end | O(n)* | O(1)* | O(1)** | O(1)* |
| Delete from end | O(n) | O(1)* | O(n) | O(1)* |
| Continuous cycling | ✗ | ✗ | ✓ | ✓ |

\* With TAIL pointer &nbsp;&nbsp; \** With TAIL pointer, O(1) for both front and end

---

## 1.7 Conversion Between Types

### SLL to SCLL

```
function sllToCircular(HEAD):
    if HEAD == NULL: return HEAD
    
    current = HEAD
    while current.next ≠ NULL:
        current = current.next
    
    current.next = HEAD          ← Last node points to HEAD
    return HEAD
```

### SCLL to SLL

```
function circularToSll(HEAD):
    if HEAD == NULL: return HEAD
    
    current = HEAD
    while current.next ≠ HEAD:
        current = current.next
    
    current.next = NULL          ← Break the cycle
    return HEAD
```

### SLL to DLL

```
function sllToDll(HEAD):
    if HEAD == NULL: return HEAD
    
    current = HEAD
    current.prev = NULL          ← HEAD has no predecessor
    
    while current.next ≠ NULL:
        current.next.prev = current   ← Set prev for each node
        current = current.next
    
    return HEAD                  ← TAIL = current (last node)
```

---

## 1.8 When to Use Each Type

```
  Need simple forward-only list?
    → SLL

  Need bidirectional traversal?
    → DLL

  Need continuous cycling (round-robin)?
    → SCLL

  Need bidirectional + continuous cycling?
    → DCLL
  
  Need LRU cache / text editor?
    → DLL (or DCLL with sentinel)

  Need task scheduler?
    → SCLL (round-robin)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Circular list | Last node points to first — no NULL termination |
| SCLL | Singly circular — one `next` pointer, loops back |
| DCLL | Doubly circular — `prev` and `next`, both loop |
| TAIL pointer | Often preferred over HEAD for circular lists (access both ends in O(1)) |
| Traversal | Use do-while or start-point check to avoid infinite loop |
| Single node | Points to itself (node.next = node) |
| Continuous cycling | Natural for round-robin, games, playlists |

---

## Quick Revision Questions

1. **What is the key difference between a circular list and a standard list?**
   <details><summary>Answer</summary>In a circular list, the last node's `next` pointer points to the first node (HEAD) instead of NULL. The list forms a ring with no NULL termination.</details>

2. **Why is a do-while loop preferred for traversing a circular list?**
   <details><summary>Answer</summary>Because the starting condition `current == HEAD` is true from the start. A while loop would never execute. Do-while processes the node first, then checks if we've returned to the start.</details>

3. **Why is maintaining TAIL often better than HEAD for a circular list?**
   <details><summary>Answer</summary>With TAIL, you can access both ends: HEAD = TAIL.next (O(1)). This allows O(1) insertion at both the beginning and end without needing two separate pointers.</details>

4. **What does a single-node circular list look like?**
   <details><summary>Answer</summary>The node's `next` pointer points to itself. In DCLL, both `next` and `prev` point to the same node (itself).</details>

5. **How do you convert a singly linked list to a circular one?**
   <details><summary>Answer</summary>Traverse to the last node (where `next == NULL`), then set `last.next = HEAD`. Time: O(n).</details>

6. **When would you choose DCLL over DLL?**
   <details><summary>Answer</summary>When you need continuous cycling in both directions — e.g., a playlist that loops, or a circular buffer with bidirectional navigation. DCLL also simplifies some sentinel-based implementations.</details>

---

[← Previous: DLL Traversal and Trade-offs](../03-Doubly-Linked-List/03-dll-traversal-and-tradeoffs.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Operations on Circular Lists →](02-circular-list-operations.md)
