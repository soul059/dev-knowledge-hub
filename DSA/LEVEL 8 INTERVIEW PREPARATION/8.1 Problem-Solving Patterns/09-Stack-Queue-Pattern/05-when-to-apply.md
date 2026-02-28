# Chapter 5: When to Apply Stack/Queue

## 📋 Chapter Overview
Decision guide for recognizing when stack or queue is the right data structure.

---

## 🔍 Stack Signal Checklist

```
  ✓ Matching / nesting — parentheses, tags, scopes
  ✓ "Next greater" or "next smaller" — monotonic stack
  ✓ Undo / backtrack — reverse chronological processing
  ✓ Expression evaluation — operators and operands
  ✓ DFS simulation — iterative traversal
  ✓ Histogram-type area — expanding/contracting regions
```

---

## 🔍 Queue Signal Checklist

```
  ✓ BFS / level-order — process nodes by distance
  ✓ Sliding window max/min — monotonic deque
  ✓ Scheduling / buffering — FIFO processing
  ✓ "First K" or "by arrival order"
  ✓ Circular buffer — ring structure
```

---

## 🧭 Decision Flowchart

```
  What order matters?
  │
  ├─ LIFO (last in, first out) ──▶ STACK
  │  │
  │  ├─ Matching? → parentheses, brackets
  │  ├─ Next greater/smaller? → monotonic stack
  │  ├─ Evaluate expression? → operand stack
  │  └─ DFS / backtrack? → recursion simulation
  │
  ├─ FIFO (first in, first out) ──▶ QUEUE
  │  │
  │  ├─ BFS / shortest path? → standard queue
  │  ├─ Sliding window? → monotonic deque
  │  └─ Process by arrival? → task queue
  │
  └─ Both ends needed ──▶ DEQUE
     │
     ├─ Sliding window max/min
     └─ Palindrome checking
```

---

## 🆚 Stack vs Queue in Common Patterns

| Pattern | Stack | Queue |
|---------|-------|-------|
| Graph traversal | DFS (iterative) | BFS (level order) |
| Shortest path (unweighted) | ✗ | ✓ (BFS guarantees shortest) |
| Matching brackets | ✓ | ✗ |
| Next greater element | ✓ (monotonic) | ✗ |
| Sliding window extrema | ✗ | ✓ (monotonic deque) |
| Expression parsing | ✓ | ✗ |
| Task scheduling | ✗ | ✓ (FIFO) |

---

## 📊 Complexity Reference

| Operation | Stack | Queue | Deque |
|-----------|-------|-------|-------|
| Push/Enqueue | O(1) | O(1) | O(1) |
| Pop/Dequeue | O(1) | O(1) | O(1) |
| Peek | O(1) | O(1) | O(1) both ends |
| Search | O(n) | O(n) | O(n) |
| Space | O(n) | O(n) | O(n) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Stack | LIFO — matching, evaluation, DFS, undo |
| Queue | FIFO — BFS, scheduling, buffering |
| Monotonic stack | O(n) next greater/smaller, histogram area |
| Monotonic deque | O(n) sliding window max/min |
| Decision | Determine required processing order first |

---

## ❓ Revision Questions

1. **Stack vs Queue: when each?** → Stack for LIFO (matching, DFS, undo); Queue for FIFO (BFS, scheduling).
2. **Monotonic stack: what problems?** → Next greater/smaller, largest rectangle, trapping rain water, stock span.
3. **Why deque for sliding window?** → Need to add at back (new elements) and remove from front (expired elements) — both O(1).
4. **Can you do BFS with a stack?** → No — stack gives DFS order. BFS requires FIFO processing for level-by-level traversal.
5. **When to use two stacks?** → Implement a queue (amortized O(1)), or track min/max alongside values (Min Stack).
6. **Monotonic deque invariant direction?** → Decreasing for max (front = max); increasing for min (front = min).

---

[← Previous: Classic Problems](04-classic-problems.md) | [Next: Heap Fundamentals →](../10-Heap-Pattern/01-heap-fundamentals.md)
