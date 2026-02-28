# 7.6 Pattern Recognition & Summary

[← Previous: Balancing Strategy](05-balancing-strategy.md) | [Next: Unit 8 — Merge K Core Problem →](../08-Merge-K-Sorted/01-core-problem.md)

---

## Recognizing Two-Heap Problems

| Signal | Example Problem |
|--------|----------------|
| "Find the median" | Streaming median, sliding window median |
| "Balance two groups" | IPO/maximize capital |
| "Partition into two optimized halves" | Minimize difference between two groups |
| "Track boundary between small and large" | Running percentile |

---

## Real-World Applications

| Application | How Two Heaps Help |
|-------------|-------------------|
| **Real-time analytics** | Running median of response times |
| **Financial trading** | Median stock price over streaming data |
| **Network monitoring** | Median latency tracking |
| **Resource allocation** | Balanced partitioning of tasks |
| **Streaming statistics** | Running percentiles (p50, p90, p99) |

---

## Unit 7 Summary Table

| Concept | Key Point |
|---------|-----------|
| Two Heap Pattern | Max-heap (lower half) + Min-heap (upper half) |
| Ordering Invariant | max_heap root ≤ min_heap root |
| Balance Invariant | Size difference ≤ 1 |
| Median (odd n) | Root of the larger heap |
| Median (even n) | Average of both roots |
| Insert | Add to correct heap + rebalance — O(log n) |
| Find Median | Return root(s) — O(1) |
| Sliding Window | Use lazy deletion for removals |
| IPO / Capital | Min-heap (waiting) + Max-heap (ready) |
| When to use | Need fast median/percentile with streaming inserts |

---

## Revision Questions

1. **After adding 6 numbers, the max-heap has [5, 3, 1] and the min-heap has [7, 9, 12]. What is the median?**
   <details><summary>Answer</summary>
   Even count → average of the two roots: (5 + 7) / 2 = 6.0. Sorted: [1, 3, 5, 7, 9, 12] → median = (5+7)/2 = 6.0 ✅.
   </details>

2. **What happens if we add 4 to the heaps in Q1?**
   <details><summary>Answer</summary>
   4 ≤ max_heap root (5) → add to max-heap → [5, 4, 3, 1]. Sizes: 4 vs 3 → max_heap.size > min_heap.size + 1 → move 5 to min-heap. Result: max = [4, 3, 1], min = [5, 7, 9, 12]. Median = 5 (odd count, min_heap is larger).
   </details>

3. **Why is lazy deletion useful for sliding window median?**
   <details><summary>Answer</summary>
   Standard heaps don't support O(log n) deletion of arbitrary elements. Lazy deletion marks elements for removal but only actually removes them when they reach the heap top. This avoids O(n) searches while maintaining O(log n) amortized per operation.
   </details>

4. **In the IPO problem, why do we need TWO heaps instead of just sorting by profit?**
   <details><summary>Answer</summary>
   Not all projects are immediately affordable. The min-heap (by capital) tracks which projects become affordable as our capital grows. The max-heap (by profit) lets us pick the most profitable among affordable projects. Simply sorting by profit ignores capital requirements.
   </details>

---

[← Previous: Balancing Strategy](05-balancing-strategy.md) | [Next: Unit 8 — Merge K Core Problem →](../08-Merge-K-Sorted/01-core-problem.md)
