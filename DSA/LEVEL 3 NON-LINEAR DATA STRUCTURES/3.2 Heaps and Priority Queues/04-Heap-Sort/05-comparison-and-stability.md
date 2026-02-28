# 4.5 Comparison with Other Sorts & Stability

[← Previous: Time Complexity](04-time-complexity.md) | [Next: Variations & Implementation →](06-variations-and-implementation.md)

---

## Overview

How Heap Sort stacks up against other sorting algorithms, and why it is not stable.

---

## Comparison Table

```
┌───────────────┬──────────┬──────────┬──────────┬────────┬──────────┐
│  Algorithm    │  Best    │ Average  │  Worst   │ Space  │ Stable?  │
├───────────────┼──────────┼──────────┼──────────┼────────┼──────────┤
│ Heap Sort     │ O(nlogn) │ O(nlogn) │ O(nlogn) │ O(1)   │   No     │
│ Merge Sort    │ O(nlogn) │ O(nlogn) │ O(nlogn) │ O(n)   │   Yes    │
│ Quick Sort    │ O(nlogn) │ O(nlogn) │ O(n²)    │ O(logn)│   No     │
│ Insertion Sort│ O(n)     │ O(n²)    │ O(n²)    │ O(1)   │   Yes    │
│ Tim Sort      │ O(n)     │ O(nlogn) │ O(nlogn) │ O(n)   │   Yes    │
└───────────────┴──────────┴──────────┴──────────┴────────┴──────────┘
```

---

## When to Choose Heap Sort

| Scenario | Best Choice | Why |
|----------|-------------|-----|
| Guaranteed O(n log n) | **Heap Sort** | Never degrades |
| Minimal memory | **Heap Sort** | O(1) auxiliary |
| Fast in practice | Quick Sort | Better cache performance |
| Stability needed | Merge Sort | Preserves equal-element order |
| Nearly sorted data | Insertion Sort / Tim Sort | Adaptive algorithms win |
| Need Kth element without full sort | **Partial Heap Sort** | Stop after K extractions |

---

## Why Heap Sort Loses in Practice

Despite O(n log n) worst case:

1. **Cache unfriendly**: Array jumps (parent ↔ child) cause cache misses
2. **More comparisons**: ~2n log n comparisons vs ~1.4n log n for Quick Sort
3. **Not adaptive**: Doesn't benefit from partially sorted input
4. **Not stable**: Equal elements may change relative order

---

## Stability

Heap Sort is **NOT stable**. Equal elements may swap relative order.

```
Example: Sort [(A,5), (B,3), (C,5)] by value

A and C have equal keys (5). After heap sort, C might appear before A.

Why? During extraction, elements are moved to distant positions
in the array, disrupting original ordering of equal elements.
```

---

## Quick Check

1. **Why is Heap Sort not stable?**
   <details><summary>Answer</summary>
   During extract, the root is swapped with the last heap element, which may jump over equal-valued elements. This long-range swap can change the relative order of equal keys.
   </details>

2. **Why does Quick Sort typically outperform Heap Sort in practice despite having a worse worst case?**
   <details><summary>Answer</summary>
   Quick Sort has better cache locality (sequential access patterns), fewer comparisons on average (~1.4n log n vs ~2n log n), and a smaller constant factor. Heap Sort's parent-child array jumps cause frequent cache misses.
   </details>

---

[← Previous: Time Complexity](04-time-complexity.md) | [Next: Variations & Implementation →](06-variations-and-implementation.md)
