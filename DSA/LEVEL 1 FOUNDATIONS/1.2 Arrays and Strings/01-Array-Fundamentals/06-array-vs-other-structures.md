# Chapter 6: Array vs Other Data Structures

[← Previous: Time Complexity](05-time-complexity.md) | [Back to README](../README.md) | [Next Unit: Traversal Patterns →](../02-Basic-Array-Techniques/01-traversal-patterns.md)

---

## Overview

Choosing the right data structure is often more important than choosing the right algorithm. This chapter compares arrays with linked lists, hash maps, stacks, queues, and trees so you know when to pick which.

---

## Array vs Linked List

```
  ARRAY (contiguous):
  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
   0x100  0x104  0x108  0x10C  0x110

  LINKED LIST (scattered):
  ┌──────┬────┐   ┌──────┬────┐   ┌──────┬────┐
  │  10  │ ──────►│  20  │ ──────►│  30  │null│
  └──────┴────┘   └──────┴────┘   └──────┴────┘
   0x200           0x500           0x380
   (random addresses — not contiguous)
```

| Operation | Array | Linked List | Winner |
|-----------|-------|-------------|--------|
| Access by index | O(1) | O(n) | Array |
| Search | O(n) | O(n) | Tie |
| Insert at beginning | O(n) | O(1) | Linked List |
| Insert at end | O(1)* | O(1)** | Tie |
| Insert at middle | O(n) | O(1)*** | Linked List |
| Delete at beginning | O(n) | O(1) | Linked List |
| Delete at end | O(1) | O(n)**** | Array |
| Memory per element | Low | High (extra pointer) | Array |
| Cache performance | Excellent | Poor | Array |

*\* if space available  
\*\* if tail pointer maintained  
\*\*\* O(1) insert after a given node, but O(n) to find the node  
\*\*\*\* O(1) with doubly linked list*

### When to Choose What

```
  ┌──────────────────────────────────────────────┐
  │  USE ARRAY WHEN:                              │
  │  • Need fast random access by index           │
  │  • Data size is known or mostly fixed          │
  │  • Sequential traversal is frequent            │
  │  • Memory efficiency matters                   │
  │  • Cache performance is critical               │
  ├──────────────────────────────────────────────┤
  │  USE LINKED LIST WHEN:                        │
  │  • Frequent insertions/deletions at front      │
  │  • Size is very unpredictable                  │
  │  • Don't need random access                    │
  │  • Memory is not contiguous                    │
  └──────────────────────────────────────────────┘
```

---

## Array vs Hash Map (Dictionary)

```
  ARRAY:                        HASH MAP:
  Index → Value                 Key → Value
  
  ┌────┬───────┐                ┌────────────┬───────┐
  │ 0  │ "Tom" │                │ "name"     │ "Tom" │
  │ 1  │ "Amy" │                │ "age"      │  25   │
  │ 2  │ "Bob" │                │ "city"     │ "NYC" │
  └────┴───────┘                └────────────┴───────┘

  Array: integer indices only    HashMap: any hashable key
```

| Operation | Array | Hash Map | Notes |
|-----------|-------|----------|-------|
| Access | O(1) by index | O(1) avg by key | Hash map can use string keys |
| Search | O(n) | O(1) avg | Hash map is better for lookups |
| Insert | O(n) worst | O(1) avg | Hash map doesn't shift |
| Delete | O(n) worst | O(1) avg | Hash map doesn't shift |
| Ordered? | Yes (by index) | No (usually) | Use TreeMap for order |
| Memory | Minimal | Higher (hash table overhead) | Array is more compact |
| Iteration | O(n), cache-friendly | O(n), less cache-friendly | Array is usually faster |

### Use Case Decision

```
  "I need to look up values by integer position"  →  ARRAY
  "I need to look up values by name/string/key"   →  HASH MAP
  "I need to count frequency of elements"          →  HASH MAP
  "I need fast duplicate detection"                →  HASH MAP (or Set)
  "I need sorted data with fast access"            →  ARRAY (sorted)
```

---

## Array vs Stack / Queue

Stacks and Queues are **abstract data types** often **implemented using arrays**.

```
  STACK (LIFO — Last In, First Out):
  
  Push 10, 20, 30:
  ┌──────┐
  │  30  │ ← top (last in, first out)
  ├──────┤
  │  20  │
  ├──────┤
  │  10  │
  └──────┘

  QUEUE (FIFO — First In, First Out):
  
  Enqueue 10, 20, 30:
  ┌──────┬──────┬──────┐
  │  10  │  20  │  30  │
  └──────┴──────┴──────┘
    ▲                ▲
  front            rear
  (dequeue)       (enqueue)
```

| Feature | Array | Stack | Queue |
|---------|-------|-------|-------|
| Access pattern | Random | Top only | Front only |
| Insert | Anywhere | Top only (push) | Rear only (enqueue) |
| Remove | Anywhere | Top only (pop) | Front only (dequeue) |
| Use case | General storage | Undo, DFS, expression eval | BFS, scheduling, buffering |
| Implementation | Primitive | Built on array/linked list | Built on array/linked list |

---

## Array vs Tree (BST / Balanced BST)

```
  SORTED ARRAY:
  ┌──┬──┬──┬──┬──┬──┬──┐
  │ 1│ 3│ 5│ 7│ 9│11│13│     Binary search: O(log n)
  └──┴──┴──┴──┴──┴──┴──┘     Insert: O(n) — must shift!

  BINARY SEARCH TREE:
           7
         ╱   ╲
        3      11
       ╱ ╲    ╱  ╲
      1   5  9    13              Search: O(log n)
                                  Insert: O(log n)
```

| Operation | Sorted Array | Balanced BST |
|-----------|-------------|--------------|
| Search | O(log n) | O(log n) |
| Insert | O(n) | O(log n) |
| Delete | O(n) | O(log n) |
| Min/Max | O(1) | O(log n) |
| Ordered traversal | O(n) | O(n) |
| Memory | Low | Higher (pointers) |
| Random access | O(1) | O(n) |

**Key insight:** If you need frequent insertions AND searches on sorted data, use a balanced BST. If data is static (rarely changes), sorted array + binary search is ideal.

---

## Master Comparison Table

```
  ┌──────────────┬────────┬────────┬────────┬────────┬────────┐
  │  Operation   │ Array  │ Linked │HashMap │ BST    │ Heap   │
  │              │        │  List  │        │(balanced)       │
  ├──────────────┼────────┼────────┼────────┼────────┼────────┤
  │ Access [i]   │  O(1)  │  O(n)  │  O(1)  │  O(n)  │  O(n)  │
  │ Search       │  O(n)  │  O(n)  │  O(1)  │O(logn) │  O(n)  │
  │ Insert       │  O(n)  │  O(1)  │  O(1)  │O(logn) │O(logn) │
  │ Delete       │  O(n)  │  O(1)  │  O(1)  │O(logn) │O(logn) │
  │ Find Min     │  O(n)  │  O(n)  │  O(n)  │O(logn) │  O(1)  │
  │ Sorted Order │O(nlogn)│O(nlogn)│O(nlogn)│  O(n)  │O(nlogn)│
  │ Memory       │  Low   │ Medium │  High  │ Medium │  Low   │
  │ Cache        │ Great  │  Poor  │  OK    │  Poor  │ Great  │
  └──────────────┴────────┴────────┴────────┴────────┴────────┘
```

---

## Decision Flowchart

```
  What do I need?
       │
       ├── Random access by index? ──────────────► ARRAY
       │
       ├── Lookup by key/name? ──────────────────► HASH MAP
       │
       ├── Frequent insert/delete at ends? ──────► LINKED LIST
       │
       ├── Sorted data + insert/delete? ─────────► BALANCED BST
       │
       ├── Get min/max quickly? ─────────────────► HEAP
       │
       ├── LIFO (last in, first out)? ───────────► STACK
       │
       └── FIFO (first in, first out)? ──────────► QUEUE
```

---

## Real-World Examples

| Scenario | Best Structure | Why |
|----------|----------------|-----|
| Store student grades by roll number | Array | Index = roll number |
| Phone book lookup by name | Hash Map | Key = name |
| Browser back/forward buttons | Stack | LIFO behavior |
| Print queue | Queue | FIFO behavior |
| Autocomplete suggestions | Trie / Sorted Array | Prefix matching |
| Task scheduler | Heap / Priority Queue | Get highest priority |
| File system directory | Tree | Hierarchical structure |

---

## Summary Table

| When You Need... | Use |
|-------------------|-----|
| Fast access by position | Array |
| Fast access by key | Hash Map |
| Fast insert/delete anywhere | Linked List |
| Sorted dynamic data | Balanced BST |
| Priority-based retrieval | Heap |
| LIFO operations | Stack (array-based) |
| FIFO operations | Queue (array-based) |

---

## Quick Revision Questions

1. **When would you choose a linked list over an array?** Give two concrete scenarios.

2. **Why are hash maps not always better than arrays** despite O(1) operations? (Think cache, memory, ordering.)

3. **A sorted array has O(log n) search. A BST also has O(log n) search.** What's the advantage of a BST?

4. **Can you implement a stack using an array?** What is the time complexity of push and pop?

5. **Why are arrays more cache-friendly than linked lists?** Draw the memory layout difference.

6. **For a phonebook app, would you use an array or a hash map?** Justify your choice.

---

[← Previous: Time Complexity](05-time-complexity.md) | [Back to README](../README.md) | [Next Unit: Traversal Patterns →](../02-Basic-Array-Techniques/01-traversal-patterns.md)
