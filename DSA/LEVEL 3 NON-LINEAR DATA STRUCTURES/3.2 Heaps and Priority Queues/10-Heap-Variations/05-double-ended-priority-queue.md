# 10.5 Double-Ended Priority Queue & Interval Heap

[← Previous: Indexed Priority Queue](04-indexed-priority-queue.md) | [Next: Comparison Guide →](06-comparison-guide.md)

---

## What Is a DEPQ?

A **DEPQ** (Double-Ended Priority Queue) supports both **extract-min** and **extract-max** in O(log n).

```
Standard Min-Heap:  peek_min O(1), peek_max O(n)  ← bad!
Standard Max-Heap:  peek_max O(1), peek_min O(n)  ← bad!
DEPQ:              peek_min O(1), peek_max O(1)   ← both!
```

---

## Implementation Approaches

| Approach | Extract Min | Extract Max | Space |
|----------|------------|------------|-------|
| Two heaps + cross-links | O(log n) | O(log n) | O(n) |
| **Interval heap** | O(log n) | O(log n) | O(n) |
| Min-max heap | O(log n) | O(log n) | O(n) |

### Simple Approach: Two Heaps

Maintain a min-heap and a max-heap with **cross-references** between corresponding elements.

```
Min-Heap: [1, 3, 5, 7, 9]     Max-Heap: [9, 7, 5, 3, 1]
              ↕ ↕ ↕ ↕ ↕
         Cross-references

Extract-Min: Remove from min-heap, also remove from max-heap using cross-ref
Extract-Max: Remove from max-heap, also remove from min-heap using cross-ref
```

---

## Interval Heap

### What Is It?

An **interval heap** is a complete binary tree where each node stores an **interval [a, b]** with a ≤ b. It naturally supports both min and max access.

### Structure

```
            [3, 50]           ← root: global min=3, global max=50
           /       \
      [10, 40]    [15, 35]    ← each node stores an interval
       /    \
   [20,30] [25,28]

Properties:
  - Parent's left endpoint ≤ children's left endpoints  (min-heap on left)
  - Parent's right endpoint ≥ children's right endpoints (max-heap on right)
  - Within each node: left ≤ right
```

### Properties

```
For node with interval [a, b] and parent with interval [p, q]:
  1. p ≤ a     (min-heap property on left endpoints)
  2. q ≥ b     (max-heap property on right endpoints)
  3. a ≤ b     (valid interval within each node)
```

### Operations

| Operation | How | Time |
|-----------|-----|------|
| **Get Min** | Root's left endpoint | O(1) |
| **Get Max** | Root's right endpoint | O(1) |
| **Insert** | Add to last node, fix up | O(log n) |
| **Extract Min** | Remove root's left, fix | O(log n) |
| **Extract Max** | Remove root's right, fix | O(log n) |

### Insert Trace

```
Insert 5 into:
      [3, 50]
     /       \
  [10, 40]  [15, 35]

Step 1: Place in next available node position
      [3, 50]
     /       \
  [10, 40]  [15, 35]
  /
[5, ?]    ← incomplete node (odd number of total elements)

Step 2: Compare with parent interval [10, 40]
  5 < 10 (parent's left) → swap with parent's left
  
      [3, 50]
     /       \
  [5, 40]   [15, 35]
  /
[10, ?]

Step 3: Continue up — compare with grandparent [3, 50]
  5 ≥ 3 → stop (min-heap property OK)

Final: 5 is in correct position ✅
```

### Advantages

| Feature | Interval Heap | Two Separate Heaps |
|---------|--------------|-------------------|
| Space | n elements in ⌈n/2⌉ nodes | 2n elements in two arrays |
| Cache performance | Better (single array) | Worse (two arrays) |
| Implementation | Moderate | Simpler but more bookkeeping |
| Both min & max | O(1) each | O(1) each |

---

[← Previous: Indexed Priority Queue](04-indexed-priority-queue.md) | [Next: Comparison Guide →](06-comparison-guide.md)
