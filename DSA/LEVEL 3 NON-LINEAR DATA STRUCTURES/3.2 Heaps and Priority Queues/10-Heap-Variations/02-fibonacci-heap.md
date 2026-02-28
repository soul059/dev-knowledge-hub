# 10.2 Fibonacci Heap (Conceptual)

[← Previous: D-ary Heap](01-d-ary-heap.md) | [Next: Binomial Heap →](03-binomial-heap.md)

---

## What Is It?

A **Fibonacci heap** is a collection of heap-ordered trees with **lazy operations** that achieve amazing **amortized** complexities.

```
Fibonacci Heap (conceptual):

  Trees with different degrees, loosely organized:
  
    5       3         8
   /|      / \       |
  10 6    7   4     12
  |      |
  15     9

  min pointer → 3 (the global minimum)
```

---

## Key Operations

| Operation | Binary Heap | **Fibonacci Heap** |
|-----------|-------------|-------------------|
| Insert | O(log n) | **O(1)** amortized |
| Find Min | O(1) | **O(1)** |
| Extract Min | O(log n) | **O(log n)** amortized |
| **Decrease Key** | O(log n) | **O(1)** amortized |
| **Merge** | O(n) | **O(1)** |
| Delete | O(log n) | O(log n) amortized |

---

## How Does It Achieve O(1) Decrease Key?

1. **Cut** the node from its parent
2. Add it as a new root tree → O(1)
3. If parent lost too many children (**cascading cut**), cut parent too
4. Amortized analysis shows this averages to O(1)

```
Before decrease key (decrease 15 to 2):

   5             After:     5         2
  /|                       /         (new root tree)
 10 6                     6
 |
 15   ← cut this

min pointer now checks: min(old_min, 2) → 2
```

---

## Why Does It Matter?

**Dijkstra's algorithm** with a Fibonacci heap:

| Implementation | Dijkstra's Time |
|----------------|----------------|
| Binary heap | O((V + E) log V) |
| **Fibonacci heap** | **O(V log V + E)** |

For dense graphs (E ≈ V²), this is a significant improvement.

---

## Why Isn't It Used More?

| Pro | Con |
|-----|-----|
| Theoretically optimal | Complex implementation (~200+ lines) |
| Best asymptotic for Dijkstra | Large constant factors |
| O(1) merge | Poor cache performance (pointer-based) |
| | Binary heap wins for n < ~10,000 |

> **In practice**: Binary heaps or D-ary heaps are almost always preferred. Fibonacci heaps are mainly of theoretical interest.

---

[← Previous: D-ary Heap](01-d-ary-heap.md) | [Next: Binomial Heap →](03-binomial-heap.md)
