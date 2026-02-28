# 10.3 Binomial Heap (Conceptual)

[← Previous: Fibonacci Heap](02-fibonacci-heap.md) | [Next: Indexed Priority Queue →](04-indexed-priority-queue.md)

---

## What Is It?

A **binomial heap** is a collection of **binomial trees** that supports efficient merge.

---

## Binomial Trees

```
B₀:   ○         (1 node, degree 0)

B₁:   ○         (2 nodes, degree 1)
      |
      ○

B₂:   ○         (4 nodes, degree 2)
      /|
     ○  ○
     |
     ○

B₃:   ○         (8 nodes, degree 3)
      /|\
     ○  ○ ○
     /| |
    ○  ○ ○
    |
    ○
```

**Pattern**: Bₖ has 2ᵏ nodes. Bₖ is formed by attaching Bₖ₋₁ as a child of another Bₖ₋₁.

---

## Binomial Heap Structure

A binomial heap with n elements uses binomial trees corresponding to the **binary representation** of n.

```
Example: n = 13 = 1101₂

Uses trees: B₃ (8 nodes) + B₂ (4 nodes) + B₀ (1 node)

   B₃          B₂      B₀
    ○           ○        ○
   /|\         /|
  ○  ○  ○    ○  ○
  /| |        |
 ○  ○ ○      ○
 |
 ○

Total: 8 + 4 + 1 = 13 nodes ✅
```

---

## Key Operations

| Operation | Complexity |
|-----------|-----------|
| Insert | O(log n) worst, **O(1) amortized** |
| Find Min | O(log n) or O(1) with pointer |
| Extract Min | O(log n) |
| **Merge** | **O(log n)** → much better than binary heap's O(n) |
| Decrease Key | O(log n) |

---

## Merge Process (Key Insight)

Merging two binomial heaps is like **binary addition**:
- If both have a Bₖ → combine into Bₖ₊₁ (like carry in addition)
- If only one has a Bₖ → keep it

```
Heap A: B₀ + B₂ (5 nodes = 101₂)
Heap B: B₁ + B₂ (6 nodes = 110₂)

Merge: 101 + 110 = 1011₂
Result: B₀ + B₁ + B₃ (11 nodes) ✅
```

---

## Binary Heap vs Binomial Heap

| Operation | Binary Heap | Binomial Heap |
|-----------|-------------|---------------|
| Insert | O(log n) | O(1) amortized |
| Extract Min | O(log n) | O(log n) |
| **Merge** | **O(n)** | **O(log n)** |
| Implementation | Simple array | Pointer-based, complex |

---

[← Previous: Fibonacci Heap](02-fibonacci-heap.md) | [Next: Indexed Priority Queue →](04-indexed-priority-queue.md)
