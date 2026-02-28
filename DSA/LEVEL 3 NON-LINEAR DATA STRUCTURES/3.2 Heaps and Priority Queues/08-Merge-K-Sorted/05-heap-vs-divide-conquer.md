# 8.5 Heap vs Divide & Conquer

[← Previous: Smallest Range](04-smallest-range.md) | [Next: Variations & Applications →](06-variations-and-applications.md)

---

## Alternative: Merge Pairs Recursively

Instead of a K-way heap merge, merge lists **two at a time**:

```
Divide and Conquer:

Round 1: Merge pairs
  [L1, L2, L3, L4, L5, L6] → [L12, L34, L56]

Round 2: Merge pairs again
  [L12, L34, L56] → [L1234, L56]

Round 3: Final merge
  [L1234, L56] → [L123456]
```

---

## Comparison

```
┌──────────────────┬────────────────────┬──────────────────────┐
│                  │  Heap (K-way)      │  Divide & Conquer    │
├──────────────────┼────────────────────┼──────────────────────┤
│ Time             │ O(N log K)         │ O(N log K)           │
│ Space (heap)     │ O(K)               │ O(1) extra           │
│ Space (recursion)│ —                  │ O(log K) stack       │
│ Implementation   │ Straightforward    │ Slightly tricky      │
│ Works for streams│ ✅ Yes              │ ❌ No (need all data) │
│ Parallelizable   │ Not easily         │ ✅ Yes               │
│ Constant factor  │ Higher (heap ops)  │ Lower (simple merge) │
└──────────────────┴────────────────────┴──────────────────────┘
```

---

## Divide and Conquer Pseudocode

```
MERGE_K_DC(lists, start, end):
    IF start == end:
        RETURN lists[start]
    IF start + 1 == end:
        RETURN MERGE_TWO(lists[start], lists[end])
    
    mid = (start + end) / 2
    left = MERGE_K_DC(lists, start, mid)
    right = MERGE_K_DC(lists, mid + 1, end)
    RETURN MERGE_TWO(left, right)
```

---

## When to Choose Which?

| Scenario | Best Approach |
|----------|--------------|
| K lists of similar length | Either (same asymptotic) |
| Streaming data | **Heap** (processes incrementally) |
| Highly parallel environment | **D&C** (pairs merge independently) |
| Memory constrained | **D&C** (O(1) vs O(K)) |
| Simple implementation | **Heap** (less recursive logic) |
| K = 2 | Direct two-way merge |

---

## All Approaches Comparison

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute force (concat + sort) | O(N log N) | O(N) | Ignores sorted property |
| Merge one by one | O(NK) | O(1) | K-1 two-way merges, increasing size |
| **Heap K-way merge** | O(N log K) | O(K) | Optimal for most cases |
| **Divide & conquer** | O(N log K) | O(log K) | Same time, less heap space |

---

## Quick Check

1. **For merging 2 sorted lists, is a heap necessary?**
   <details><summary>Answer</summary>
   No. Two-way merge uses two pointers and runs in O(n₁ + n₂) with O(1) extra space. The heap overhead (O(log 2) = O(1) per element) adds unnecessary constant factor. Use the simple two-pointer merge for K=2.
   </details>

---

[← Previous: Smallest Range](04-smallest-range.md) | [Next: Variations & Applications →](06-variations-and-applications.md)
