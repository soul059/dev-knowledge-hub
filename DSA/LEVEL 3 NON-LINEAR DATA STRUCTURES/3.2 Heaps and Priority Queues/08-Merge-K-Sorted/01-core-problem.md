# 8.1 The Core Problem: Merge K Sorted

[← Previous: Unit 7 — Pattern Recognition](../07-Two-Heap-Pattern/06-pattern-recognition.md) | [Next: Merge K Sorted Lists →](02-merge-k-sorted-lists.md)

---

## Overview

Merging K sorted sequences is a classic heap application. The heap acts as a **K-way selector**, always choosing the smallest (or largest) element among K candidates. This pattern appears in database query processing, external sorting, and distributed systems.

---

## The Problem

### Given

K sorted lists (or arrays, or streams), each already sorted individually.

### Goal

Merge them into a **single sorted** output.

---

## Why Not Just Concatenate and Sort?

| Approach | Time | Space |
|----------|------|-------|
| Concatenate + sort | O(N log N) where N = total elements | O(N) |
| **K-way merge with heap** | **O(N log K)** | **O(K)** |

When K << N, the heap approach is significantly faster.

---

## Why O(N log K)?

```
Total elements: N = n₁ + n₂ + ... + nₖ

Each of the N elements:
  - Inserted into heap once:  O(log K)
  - Extracted from heap once: O(log K)

Total: N × O(log K) = O(N log K)
```

---

## Why is O(N log K) Better Than O(N log N)?

When K << N:
```
K = 10, N = 1,000,000

O(N log N) = 1,000,000 × 20 = 20,000,000
O(N log K) = 1,000,000 × 3.3 = 3,300,000

~6x faster!
```

---

## The Core Mechanism

```
Maintain a min-heap that always holds ONE element from EACH list.

Step 1: Insert the HEAD of each list into the min-heap
Step 2: Extract-min → this is the next element in the merged output
Step 3: If the extracted element has a NEXT element, insert it into the heap
Step 4: Repeat until heap is empty
```

---

## Quick Check

1. **Why does the heap hold exactly K elements (one from each list)?**
   <details><summary>Answer</summary>
   We need to compare the current minimum candidates from ALL K lists. Each list contributes its smallest unprocessed element to the heap. After extracting the global minimum, we replace it with the next element from that same list, maintaining exactly one representative per active list.
   </details>

---

[← Previous: Unit 7 — Pattern Recognition](../07-Two-Heap-Pattern/06-pattern-recognition.md) | [Next: Merge K Sorted Lists →](02-merge-k-sorted-lists.md)
