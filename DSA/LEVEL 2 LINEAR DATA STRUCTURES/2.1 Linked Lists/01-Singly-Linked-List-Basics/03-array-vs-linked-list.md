# Chapter 3: Array vs Linked List / When to Use Linked Lists

[← Previous: Head, Tail, and Traversal](02-head-tail-and-traversal.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 2 — Insertion Operations →](../02-Singly-Linked-List-Operations/01-insertion-operations.md)

---

## Chapter Overview

This chapter provides a **detailed comparison** between arrays and linked lists — the two foundational linear data structures. Understanding when to choose one over the other is a critical skill for problem-solving and system design.

---

## 3.1 Memory Layout Comparison

### Array: Contiguous Memory

```
  Index:    0      1      2      3      4
         ┌──────┬──────┬──────┬──────┬──────┐
         │  10  │  20  │  30  │  40  │  50  │
         └──────┴──────┴──────┴──────┴──────┘
 Address: 0x100  0x104  0x108  0x10C  0x110
          ◄────►
          4 bytes each (contiguous, predictable)
```

### Linked List: Scattered Memory

```
  HEAD
   │
   ▼
 [10|0x250]    [20|0x180]    [30|0x300]    [40|0x140]    [50|NULL]
  (0x100)  ───► (0x250)  ───► (0x180)  ───► (0x300)  ───► (0x140)
  
  Nodes scattered across memory — connected by pointers
```

---

## 3.2 Operation-by-Operation Comparison

### Access (Random Access)

```
Array:   arr[3] = base + 3 × size    → O(1) direct access
         ┌──┬──┬──┬──┬──┐
         │  │  │  │▓▓│  │   ← Jump directly to index 3
         └──┴──┴──┴──┴──┘

Linked:  HEAD → [1] → [2] → [3]     → O(n) sequential access
         Must visit nodes 0, 1, 2 before reaching node 3
```

### Insert at Beginning

```
Array:   Shift ALL elements right     → O(n)
         ┌──┬──┬──┬──┬──┐            ┌──┬──┬──┬──┬──┬──┐
         │10│20│30│40│50│    →→→     │ 5│10│20│30│40│50│
         └──┴──┴──┴──┴──┘            └──┴──┴──┴──┴──┴──┘
                                      ↑  ←  ←  ←  ←  ←
                                     new   (all shifted right)

Linked:  Just change HEAD pointer     → O(1)
         
         Before:  HEAD → [10] → [20] → [30] → NULL
         
         After:   HEAD → [ 5] → [10] → [20] → [30] → NULL
                          ↑
                         new node (new.next = old HEAD)
```

### Insert at End

```
Array (dynamic):  Append at end       → O(1) amortized
         ┌──┬──┬──┬──┬──┬──┐
         │10│20│30│40│50│60│         ← Just place at next slot
         └──┴──┴──┴──┴──┴──┘

Linked (no tail):  Traverse to end   → O(n)
         HEAD → [10] → [20] → ... → [50] → [60] → NULL
                                              ↑
                                        Must traverse here first

Linked (with tail):                   → O(1)
         TAIL.next = newNode; TAIL = newNode
```

### Insert in Middle (at position k)

```
Array:   Shift elements from k onward → O(n)

Linked:  Traverse to position k       → O(n)   (traverse)
         Then relink pointers          → O(1)   (actual insert)
         Total:                        → O(n)
```

### Delete from Beginning

```
Array:   Shift ALL elements left      → O(n)
         ┌──┬──┬──┬──┬──┐            ┌──┬──┬──┬──┐
         │10│20│30│40│50│    →→→     │20│30│40│50│
         └──┴──┴──┴──┴──┘            └──┴──┴──┴──┘
          ×  →  →  →  →

Linked:  Just move HEAD               → O(1)
         Before:  HEAD → [10] → [20] → [30] → NULL
         After:          (freed)
                  HEAD ─────────► [20] → [30] → NULL
```

### Delete from End

```
Array:   Simply reduce size           → O(1)

Linked (singly):  Need previous node → O(n)
         Must traverse to find second-to-last node
```

---

## 3.3 Complete Comparison Table

| Operation | Array | Linked List (Singly) | Winner |
|-----------|-------|---------------------|--------|
| **Access by index** | O(1) | O(n) | Array |
| **Search (unsorted)** | O(n) | O(n) | Tie |
| **Search (sorted)** | O(log n) binary search | O(n) | Array |
| **Insert at beginning** | O(n) | O(1) | Linked List |
| **Insert at end** | O(1) amortized | O(1) with tail, O(n) without | Tie |
| **Insert at position k** | O(n) | O(n) | Tie* |
| **Delete at beginning** | O(n) | O(1) | Linked List |
| **Delete at end** | O(1) | O(n) | Array |
| **Delete at position k** | O(n) | O(n) | Tie* |
| **Memory per element** | `sizeof(data)` | `sizeof(data) + sizeof(pointer)` | Array |
| **Memory allocation** | Contiguous block | Individual nodes | Linked List |
| **Cache performance** | Excellent | Poor | Array |
| **Dynamic resizing** | Expensive (copy all) | Free (per node) | Linked List |

\* For linked lists, once you have a reference to the node, insertion/deletion is O(1). The O(n) is for finding the position.

---

## 3.4 Memory Efficiency Analysis

### Per-Element Memory Usage

```
Array (int):
  ┌──────┐
  │ data │  = 4 bytes per element
  └──────┘

Linked List Node (int, 64-bit system):
  ┌──────┬──────────┐
  │ data │   next   │  = 4 + 8 = 12 bytes per node
  └──────┴──────────┘    (+ malloc overhead ≈ 16-24 bytes total!)
```

### Total Memory for n Elements

| Structure | Memory Usage | For 1000 ints |
|-----------|-------------|---------------|
| Array | `n × sizeof(data)` | 4,000 bytes |
| Linked List | `n × (sizeof(data) + sizeof(ptr) + overhead)` | ~24,000 bytes |

> **Linked lists use 4-6x more memory** per element than arrays!

### But... Array Resizing Overhead

Dynamic arrays (like `ArrayList`, `vector`) double in size when full:

```
  Capacity:  4  →  8  →  16  →  32  →  64 ...
  Elements:  4     5      9     17     33

  At worst, ~50% of allocated memory is UNUSED
  Example: 33 elements in a capacity-64 array wastes 31 slots
```

---

## 3.5 Cache Performance — Why Arrays are Faster in Practice

### How CPU Cache Works

```
  CPU reads memory in "cache lines" (64 bytes typically)
  
  Array — ONE cache line loads multiple elements:
  ┌──────────────────────────────────┐
  │ 10 │ 20 │ 30 │ 40 │ 50 │ 60 │ 70 │ 80 │  ← All in ONE cache line!
  └──────────────────────────────────┘
  Cache hit rate: VERY HIGH (spatial locality)
  
  Linked List — Each node may require a SEPARATE cache line fetch:
  
  Node at 0x100 ─── cache line 1
        │
  Node at 0x5A0 ─── cache line 2 (MISS!)
        │
  Node at 0x230 ─── cache line 3 (MISS!)
        │
  Node at 0x780 ─── cache line 4 (MISS!)
  
  Cache hit rate: VERY LOW (no spatial locality)
```

### Performance Impact

| Metric | Array | Linked List |
|--------|-------|-------------|
| Cache line utilization | ~100% | ~10-25% |
| Cache miss per access | Rare | Frequent |
| Prefetch friendly | Yes | No |
| Real-world traversal | 5-10x faster | Baseline |

> **In practice**, even O(n) array operations can be faster than O(1) linked list operations for small-to-medium data sizes due to cache efficiency.

---

## 3.6 When to Use Linked Lists

### Use Linked Lists When:

| Scenario | Why |
|----------|-----|
| Frequent insertions/deletions at the beginning | O(1) vs O(n) for arrays |
| Unknown or highly variable size | No need to pre-allocate or resize |
| No random access needed | Sequential access is sufficient |
| Implementing stacks/queues | Natural fit for LIFO/FIFO |
| Need to split/merge lists frequently | Just rearrange pointers |
| Memory fragmentation is a concern | Don't need contiguous memory |
| Building other structures | Hash tables (chaining), graphs (adjacency lists) |

### Use Arrays When:

| Scenario | Why |
|----------|-----|
| Frequent random access | O(1) indexing |
| Data size is known/fixed | No overhead of pointers |
| Cache performance matters | Spatial locality |
| Need binary search | Not possible on linked lists |
| Memory efficiency is critical | No pointer overhead |
| Simple iteration | Array traversal is CPU-cache friendly |

---

## 3.7 Decision Framework

```
                    Need random access?
                    ┌──── YES ──── Use ARRAY
                    │
              ──────┤
                    │
                    └──── NO
                           │
                    Frequent insert/delete at front?
                    ┌──── YES ──── Use LINKED LIST
                    │
              ──────┤
                    │
                    └──── NO
                           │
                    Size highly variable?
                    ┌──── YES ──── Use LINKED LIST
                    │
              ──────┤
                    │
                    └──── NO
                           │
                    Need cache-friendly traversal?
                    ┌──── YES ──── Use ARRAY
                    │
              ──────┤
                    │
                    └──── NO ──── Either works; default to ARRAY
```

---

## 3.8 Hybrid Approach: Unrolled Linked List

An **unrolled linked list** combines the best of both:

```
  HEAD
   │
   ▼
 ┌─────────────────┬───┐    ┌─────────────────┬───┐
 │ [10][20][30][40] │ ──┼───►│ [50][60][70][80] │ ──┼──► NULL
 └─────────────────┴───┘    └─────────────────┴───┘
   4 elements per node        4 elements per node
```

- **Better cache performance** — multiple elements per node
- **Fewer pointers** — one per group instead of one per element
- Used in some real-world databases and file systems

---

## 3.9 Language-Specific Examples

| Language | Array Type | Linked List Type |
|----------|-----------|-----------------|
| C | `int arr[]` | Manual with `struct` |
| C++ | `std::vector` | `std::list` / `std::forward_list` |
| Java | `ArrayList` | `LinkedList` |
| Python | `list` (dynamic array) | No built-in (use `collections.deque`) |
| JavaScript | `Array` | No built-in |

> **Fun fact:** Python's `list` is actually a **dynamic array**, not a linked list, despite the name.

---

## Summary Table

| Aspect | Array | Linked List |
|--------|-------|-------------|
| Memory layout | Contiguous | Scattered |
| Access | O(1) random | O(n) sequential |
| Insert at front | O(n) shift | O(1) pointer change |
| Insert at end | O(1) amortized | O(1) with tail |
| Delete at front | O(n) shift | O(1) pointer change |
| Memory per element | Data only | Data + pointer + overhead |
| Cache friendliness | Excellent | Poor |
| Dynamic sizing | Costly resize | Free per node |
| Best for | Random access, fixed size | Frequent insert/delete |

---

## Quick Revision Questions

1. **Why is array access O(1) but linked list access O(n)?**
   <details><summary>Answer</summary>Arrays store elements contiguously, so the address of any element can be calculated with a formula: `base + index × size`. Linked lists store nodes at scattered locations — you must follow pointers sequentially from HEAD.</details>

2. **Why are arrays often faster than linked lists even for O(n) operations?**
   <details><summary>Answer</summary>Arrays benefit from CPU cache locality — a single cache line loads multiple contiguous elements. Linked list nodes are scattered in memory, causing frequent cache misses.</details>

3. **When would you choose a linked list over a dynamic array?**
   <details><summary>Answer</summary>When you need frequent insertions/deletions at the beginning, when the size varies dramatically, when you don't need random access, or when you can't guarantee contiguous memory availability.</details>

4. **How much extra memory does a linked list node use compared to an array element?**
   <details><summary>Answer</summary>Each node needs an additional pointer (4 bytes on 32-bit, 8 bytes on 64-bit) plus memory allocator overhead. Total can be 4-6x more than a plain array element.</details>

5. **Why can't you perform binary search on a linked list?**
   <details><summary>Answer</summary>Binary search requires O(1) access to the middle element. In a linked list, finding the middle takes O(n) traversal, making binary search O(n log n) — worse than linear search O(n).</details>

6. **What is an unrolled linked list and what problem does it solve?**
   <details><summary>Answer</summary>An unrolled linked list stores multiple elements per node (like a small array). This improves cache performance and reduces pointer overhead while maintaining the dynamic insertion/deletion benefits of linked lists.</details>

---

[← Previous: Head, Tail, and Traversal](02-head-tail-and-traversal.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 2 — Insertion Operations →](../02-Singly-Linked-List-Operations/01-insertion-operations.md)
