# Chapter 6: Quick Sort vs Other Sorting Algorithms

## Overview

Quick Sort is one of many O(n log n) sorting algorithms. This chapter provides a comprehensive comparison to help you understand when to use Quick Sort versus alternatives, and why Quick Sort remains the most popular choice in practice.

---

## The Big Comparison Table

```
┌──────────────┬────────────┬────────────┬──────────┬───────┬────────┬──────────┐
│  Algorithm   │   Best     │   Average  │   Worst  │ Space │ Stable │ In-Place │
├──────────────┼────────────┼────────────┼──────────┼───────┼────────┼──────────┤
│ Quick Sort   │ O(n log n) │ O(n log n) │  O(n²)   │O(lg n)│  No    │   Yes    │
│ Merge Sort   │ O(n log n) │ O(n log n) │O(n log n)│ O(n)  │  Yes   │   No     │
│ Heap Sort    │ O(n log n) │ O(n log n) │O(n log n)│ O(1)  │  No    │   Yes    │
│ Tim Sort     │   O(n)     │ O(n log n) │O(n log n)│ O(n)  │  Yes   │   No     │
│ Insertion    │   O(n)     │   O(n²)    │  O(n²)   │ O(1)  │  Yes   │   Yes    │
│ Shell Sort   │ O(n log n) │  varies    │  varies  │ O(1)  │  No    │   Yes    │
│ Counting     │  O(n+k)   │  O(n+k)    │ O(n+k)   │ O(k)  │  Yes   │   No     │
│ Radix        │  O(nk)    │  O(nk)     │ O(nk)    │O(n+k) │  Yes   │   No     │
└──────────────┴────────────┴────────────┴──────────┴───────┴────────┴──────────┘
```

---

## Quick Sort vs Merge Sort

```
┌───────────────────────┬─────────────────────────────────┐
│     QUICK SORT        │        MERGE SORT               │
├───────────────────────┼─────────────────────────────────┤
│ Work in DIVIDE step   │ Work in COMBINE step            │
│ In-place (O(lg n))    │ Not in-place (O(n) extra)       │
│ Not stable            │ Stable                          │
│ O(n²) worst case      │ O(n log n) always               │
│ Better cache perf.    │ Worse cache (extra array)       │
│ ~2× faster constants  │ More data movement              │
│ Partition: ~n/6 swaps │ Merge: 2n moves per level       │
│ Bad for linked lists  │ Perfect for linked lists        │
│ Bad for external sort │ Perfect for external sort       │
└───────────────────────┴─────────────────────────────────┘

WHEN TO USE:
  Quick Sort → Arrays, in-memory, general purpose
  Merge Sort → Linked lists, external sorting, need stability
```

### Cache Performance Explained

```
Quick Sort:                    Merge Sort:
  Partition scans array         Copies to temp array
  linearly (sequential          then copies back
  access)                       (two arrays in cache)
  
  ┌─────────────┐               ┌─────────────┐
  │ arr[0..n]   │ ← hot         │ arr[0..n]   │ ← cold
  └─────────────┘               └─────────────┘
  One array in cache             ┌─────────────┐
                                │ temp[0..n]  │ ← needs space
                                └─────────────┘
                                
  Quick Sort wins on cache!
```

---

## Quick Sort vs Heap Sort

```
┌───────────────────────┬──────────────────────────────────┐
│     QUICK SORT        │         HEAP SORT                │
├───────────────────────┼──────────────────────────────────┤
│ O(n²) worst case      │ O(n log n) always                │
│ O(log n) space        │ O(1) space                       │
│ Cache-friendly        │ Cache-UNFRIENDLY                 │
│ Faster in practice    │ ~2-5× slower (scattered access)  │
│ Not stable            │ Not stable                       │
│ Good locality         │ Poor locality (heap jumps)       │
└───────────────────────┴──────────────────────────────────┘

Heap Sort problems:
  Parent at i, children at 2i+1 and 2i+2
  → Random-looking memory access pattern
  → Many cache misses
  → Despite O(n log n) guarantee, slower in practice

IntroSort solves this: use Quick Sort normally,
fall back to Heap Sort only when depth limit exceeded.
```

---

## Quick Sort vs Tim Sort

```
Tim Sort (Python, Java default):
  Hybrid of merge sort + insertion sort
  
  1. Find natural "runs" (ascending sequences)
  2. Extend short runs with insertion sort (to min ~32)
  3. Merge runs using a merge stack with invariants

┌───────────────────────┬──────────────────────────────────┐
│     QUICK SORT        │          TIM SORT                │
├───────────────────────┼──────────────────────────────────┤
│ O(n²) worst case      │ O(n log n) always                │
│ O(log n) space        │ O(n) space                       │
│ Not stable            │ Stable                           │
│ Not adaptive          │ Adaptive (O(n) on sorted input)  │
│ Better for random data│ Better for real-world data       │
│ C/C++ preferred       │ Python/Java default              │
└───────────────────────┴──────────────────────────────────┘

Tim Sort wins on:
  ✓ Partially sorted data
  ✓ Data with existing patterns
  ✓ Stability requirement
  ✓ Guaranteed O(n log n)

Quick Sort wins on:
  ✓ Random data (lower constant)
  ✓ Memory-constrained environments
  ✓ When in-place is required
```

---

## When to Use Each Algorithm

```
┌────────────────────────────────────────────────────────────┐
│  DECISION GUIDE                                            │
│                                                            │
│  Need stable sort?                                         │
│    YES → Merge Sort or Tim Sort                            │
│    NO  → Continue                                          │
│                                                            │
│  Memory constrained (no O(n) extra)?                       │
│    YES → Quick Sort or Heap Sort                           │
│    NO  → Continue                                          │
│                                                            │
│  Sorting linked list?                                      │
│    YES → Merge Sort                                        │
│    NO  → Continue                                          │
│                                                            │
│  External sort (data on disk)?                             │
│    YES → External Merge Sort                               │
│    NO  → Continue                                          │
│                                                            │
│  Data mostly sorted?                                       │
│    YES → Tim Sort or Insertion Sort (if small)             │
│    NO  → Continue                                          │
│                                                            │
│  Need guaranteed O(n log n)?                               │
│    YES → Merge Sort, Heap Sort, or IntroSort               │
│    NO  → Quick Sort (usually fastest)                      │
│                                                            │
│  Very small array (n < 20)?                                │
│    YES → Insertion Sort                                    │
│    NO  → Quick Sort with small-array cutoff                │
│                                                            │
│  Integer keys with small range?                            │
│    YES → Counting Sort or Radix Sort                       │
│    NO  → Comparison-based sort                             │
└────────────────────────────────────────────────────────────┘
```

---

## What Standard Libraries Use

| Language | Default Sort | Algorithm |
|----------|-------------|-----------|
| C (qsort) | Quick Sort variant | Most implementations |
| C++ (std::sort) | IntroSort | Quick Sort + Heap Sort + Insertion |
| Java (Arrays.sort) | Dual-Pivot Quick Sort | For primitives |
| Java (Collections.sort) | Tim Sort | For objects |
| Python (sorted) | Tim Sort | Always |
| C# (Array.Sort) | IntroSort | Quick Sort + Heap + Insertion |
| Rust (sort) | Tim Sort variant | Stable, adaptive |
| Go (sort.Sort) | IntroSort variant | Pattern-defeating |

---

## Practical Benchmark Comparison

```
Relative performance (normalized, random data):

n = 10,000 elements:
  Quick Sort:     1.0×   (baseline)
  Merge Sort:     1.5×
  Heap Sort:      2.0×
  Tim Sort:       1.2×
  Insertion Sort: 50×

n = 1,000,000 elements:
  Quick Sort:     1.0×
  Merge Sort:     1.8×
  Heap Sort:      3.5×
  Tim Sort:       1.3×

Note: These are rough estimates. Actual performance
depends on hardware, cache sizes, and implementation.
```

---

## Summary Table

| Criterion | Winner |
|-----------|--------|
| Overall fastest (random data) | Quick Sort |
| Guaranteed O(n log n) | Merge Sort / Heap Sort |
| Least memory | Heap Sort (O(1) extra) |
| Stable sort | Merge Sort / Tim Sort |
| Nearly sorted data | Tim Sort |
| Linked lists | Merge Sort |
| External sorting | Merge Sort |
| Small arrays (n<20) | Insertion Sort |
| Integer keys | Counting/Radix Sort |
| Production C++ | IntroSort |
| Production Python/Java | Tim Sort |

---

## Quick Revision Questions

1. **Why is Quick Sort faster than Merge Sort in practice despite potentially worse worst case?**
2. **Why is Heap Sort slower in practice despite O(n log n) guarantee?**
3. **When should you prefer Merge Sort over Quick Sort?**
4. **What is Tim Sort and why do Python and Java use it?**
5. **What algorithm does C++ std::sort use, and why?**

---

[← Previous: Randomized Quick Sort](05-randomized-quicksort.md) | [Back to Unit 5](../README.md)
