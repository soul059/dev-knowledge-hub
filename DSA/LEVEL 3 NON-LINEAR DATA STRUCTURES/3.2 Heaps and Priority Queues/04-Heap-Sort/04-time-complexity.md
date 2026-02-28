# 4.4 Time Complexity Analysis

[← Previous: In-Place Sorting](03-in-place-sorting.md) | [Next: Comparison & Stability →](05-comparison-and-stability.md)

---

## Overview

Heap Sort achieves O(n log n) in **all cases** — best, average, and worst. No degenerate inputs exist.

---

## Phase 1: Build Heap

$$T_{\text{build}} = O(n)$$

(Proven in Unit 3 — bottom-up Floyd's algorithm)

---

## Phase 2: n-1 Extractions

Each extraction:
- Swap: O(1)
- Sift down through heap of size `i`: O(log i)

$$T_{\text{extract}} = \sum_{i=1}^{n-1} O(\log i) = O(n \log n)$$

---

## Total

$$T(n) = O(n) + O(n \log n) = O(n \log n)$$

---

## All Cases

| Case | Time | When |
|------|------|------|
| **Best** | O(n log n) | Always (no shortcut for already-sorted) |
| **Average** | O(n log n) | |
| **Worst** | O(n log n) | Always (guaranteed!) |

> **Key advantage**: Heap Sort is O(n log n) in **all** cases. No degenerate inputs.

---

## Comparison of Exact Constants

| Algorithm | Approximate Comparisons |
|-----------|------------------------|
| Heap Sort | ~2n log₂ n |
| Quick Sort | ~1.4n log₂ n |
| Merge Sort | ~n log₂ n |

Heap Sort does roughly twice the comparisons of Merge Sort due to the 2-child comparison at each sift-down level.

---

## Quick Check

1. **What is the exact number of swap operations in the worst case of Heap Sort for n elements?**
   <details><summary>Answer</summary>
   Build heap: O(n) swaps. Extraction: each of n-1 extractions does at most ⌊log₂ i⌋ swaps. Total ≈ n + Σ log₂ i ≈ n log n swaps.
   </details>

---

[← Previous: In-Place Sorting](03-in-place-sorting.md) | [Next: Comparison & Stability →](05-comparison-and-stability.md)
