# 6.1 The Golden Rule: Which Heap to Use?

[← Previous: Unit 5 — Design Patterns](../05-Priority-Queue/06-design-patterns.md) | [Next: Kth Largest Element →](02-kth-largest-element.md)

---

## Overview

"K-element" problems are among the most common heap interview questions. The core idea: **use a heap to efficiently track the top K elements** without sorting everything.

---

## The Golden Rule

This is the **most important** insight for K-element problems:

```
┌──────────────────────────────────────────────────────────┐
│  Finding K LARGEST  → Use a MIN-heap of size K           │
│  Finding K SMALLEST → Use a MAX-heap of size K           │
│                                                          │
│  Why? The heap root acts as a "gatekeeper":              │
│  - Min-heap root = smallest of the K largest             │
│  - If new element > root, it replaces root               │
│  - Elements smaller than root are rejected               │
└──────────────────────────────────────────────────────────┘
```

---

## Why Not the "Obvious" Choice?

You might think: "K largest → max-heap → extract K times."

That works but is **less efficient** for streaming data:

| Approach | Build | Extract K | Total | Extra Space |
|----------|-------|-----------|-------|-------------|
| Max-heap of ALL n elements | O(n) | O(K log n) | O(n + K log n) | O(n) |
| **Min-heap of K elements** | O(K) | — | **O(n log K)** | **O(K)** |

When K << n, O(n log K) << O(n + K log n), and space O(K) << O(n).

---

## Intuition: The Gatekeeper

```
Streaming 10 numbers, keeping top 3 (K=3):

  Min-heap (size 3) acts as a "VIP bouncer":
  
  Root = smallest in VIP section = entry threshold
  
  New arrival > threshold? → Come in, weakest VIP leaves
  New arrival ≤ threshold? → Rejected
  
  After processing all elements:
  VIP section = the K largest elements ✓
```

---

## Choosing the Right Approach

| Array Size | K Size | Best Approach | Complexity |
|-----------|--------|---------------|------------|
| Small n | Any K | Sort + slice | O(n log n) |
| Large n | Small K | Heap of size K | O(n log K) |
| Large n | K ≈ n | Build heap + extract K | O(n + K log n) |
| Large n | K = 1 | Simple max/min scan | O(n) |
| Any n | K = median | QuickSelect | O(n) avg |

---

## Quick Check

1. **To find the 5 smallest elements from a stream of 1 million numbers, what type of heap and what size?**
   <details><summary>Answer</summary>
   **Max-heap of size 5**. The root is the largest among the 5 smallest. Any new element smaller than the root replaces it. Space: O(5) = O(1). Time: O(n log 5) ≈ O(n).
   </details>

---

[← Previous: Unit 5 — Design Patterns](../05-Priority-Queue/06-design-patterns.md) | [Next: Kth Largest Element →](02-kth-largest-element.md)
