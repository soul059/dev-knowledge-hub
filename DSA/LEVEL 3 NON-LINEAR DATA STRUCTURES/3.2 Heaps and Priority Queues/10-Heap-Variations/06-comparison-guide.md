# 10.6 Comparison of All Heap Variants

[← Previous: Double-Ended Priority Queue](05-double-ended-priority-queue.md) | [Back to README](../README.md)

---

## Full Comparison Table

```
┌─────────────────┬─────────┬───────┬──────────┬─────────┬───────┐
│                 │ Insert  │ ExtMin│ Dec.Key  │ Merge   │ Space │
├─────────────────┼─────────┼───────┼──────────┼─────────┼───────┤
│ Binary Heap     │O(log n) │O(logn)│ O(log n) │ O(n)    │ O(n)  │
│ D-ary Heap      │O(log_D n)│O(D·log_D n)│O(log_D n)│O(n)│ O(n)  │
│ Binomial Heap   │O(1)*    │O(logn)│ O(log n) │O(log n) │ O(n)  │
│ Fibonacci Heap  │O(1)*    │O(logn)*│ O(1)*   │ O(1)    │ O(n)  │
│ Indexed PQ      │O(log n) │O(logn)│O(log n)  │ O(n)    │ O(n)  │
│ Interval Heap   │O(log n) │O(logn)│ —        │ O(n)    │ O(n)  │
└─────────────────┴─────────┴───────┴──────────┴─────────┴───────┘
  * = amortized
```

---

## Decision Guide

```
                    Need efficient merge?
                    /                   \
                  YES                   NO
                 /                       \
      Binomial/Fibonacci              Need decrease-key?
                                      /            \
                                    YES            NO
                                   /                \
                           Indexed PQ            Need both 
                           (practical)           min & max?
                                                /        \
                                              YES        NO
                                             /            \
                                      Interval Heap    Binary or
                                      or DEPQ         D-ary Heap
```

---

## Real-World Applications

| Variation | Application |
|-----------|-------------|
| **D-ary heap** | Dijkstra's algorithm (D=4 for cache performance) |
| **Fibonacci heap** | Theoretical graph algorithms, academic papers |
| **Binomial heap** | Mergeable priority queues |
| **Indexed PQ** | Graph algorithms (Dijkstra, Prim, A*) |
| **DEPQ** | Bandwidth management, cache eviction (evict min priority or max age) |
| **Interval heap** | Real-time systems needing both min and max deadlines |

---

## Unit 10 Summary Table

| Variation | Key Feature | Best For |
|-----------|-------------|----------|
| D-ary heap | D children per node | Insert-heavy workloads, cache optimization |
| Fibonacci heap | O(1) decrease-key (amortized) | Theoretical Dijkstra improvement |
| Binomial heap | O(log n) merge | Mergeable priority queues |
| Indexed PQ | O(1) element lookup + O(log n) update | Graph algorithms with update priority |
| DEPQ | O(1) both min and max access | Dual-extremum queries |
| Interval heap | Each node stores [min, max] interval | Efficient DEPQ implementation |

---

## Quick Revision Questions

1. **In a 4-ary heap, what are the children of node at index 3?**
   <details><summary>Answer</summary>
   Children indices: 4*3+1=13, 4*3+2=14, 4*3+3=15, 4*3+4=16. The parent formula is (i-1)/4.
   </details>

2. **Why does increasing D in a D-ary heap help insert but hurt extract-min?**
   <details><summary>Answer</summary>
   The tree height is log_D(n), which decreases with larger D → insert (sift-up) traverses fewer levels. But sift-down must compare with D children at each level, so extract-min does D comparisons per level → O(D · log_D(n)) comparisons total.
   </details>

3. **What makes Fibonacci heaps impractical despite superior theoretical complexity?**
   <details><summary>Answer</summary>
   (1) Complex implementation with cascading cuts and marking. (2) Large constant factors in operations. (3) Poor cache performance due to pointer-based structure. (4) Binary/D-ary heaps are faster in practice for typical input sizes.
   </details>

4. **How does an indexed priority queue achieve O(log n) decrease-key while a standard binary heap needs O(n)?**
   <details><summary>Answer</summary>
   The indexed PQ maintains a hash map from element key to heap index. This gives O(1) lookup of where an element is in the heap. Then sift-up from that index takes O(log n). Without the index map, finding the element requires O(n) linear scan.
   </details>

5. **What is the relationship between the binary representation of n and the structure of a binomial heap with n elements?**
   <details><summary>Answer</summary>
   A binomial heap with n elements contains binomial tree Bₖ if and only if the k-th bit of n's binary representation is 1. For example, n=11=1011₂ contains B₀, B₁, and B₃ (trees with 1, 2, and 8 nodes).
   </details>

6. **In an interval heap, why is the minimum at the root's left endpoint and the maximum at the root's right endpoint?**
   <details><summary>Answer</summary>
   The left endpoints form a min-heap (parent's left ≤ children's left), so the global minimum bubbles to the root's left. The right endpoints form a max-heap (parent's right ≥ children's right), so the global maximum bubbles to the root's right.
   </details>

---

## Course Conclusion

Congratulations on completing the Heaps and Priority Queues guide! Here's what you've mastered:

| Unit | Core Takeaway |
|------|--------------|
| 1 | Heap = complete binary tree + heap property, stored as array |
| 2 | Insert (sift up) and Extract (sift down) — both O(log n) |
| 3 | Build heap = O(n) using bottom-up sift down |
| 4 | Heap Sort = O(n log n) in-place, not stable |
| 5 | Priority Queue = abstract type, heap = best implementation |
| 6 | K-element problems: min-heap for K largest, max-heap for K smallest |
| 7 | Two Heaps: split into lower (max) + upper (min) halves for median |
| 8 | Merge K sorted: min-heap of K candidates → O(N log K) |
| 9 | Advanced: greedy + heap for scheduling, cooldown, sequence generation |
| 10 | Variations: D-ary, Fibonacci, binomial, indexed PQ, interval heap |

> **Next step**: Practice! Implement each data structure from scratch, then solve problems on LeetCode/HackerRank using these patterns.

---

[← Previous: Double-Ended Priority Queue](05-double-ended-priority-queue.md) | [Back to README](../README.md)
