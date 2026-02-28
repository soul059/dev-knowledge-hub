# Unit 11: Sorting Algorithm Comparison — Comparison Table

[← Previous: Unit 10 — Bucket Sort](../10-Bucket-Sort/06-comparison-with-other-sorts.md) | [Next: Stability Analysis →](02-stability-analysis.md)

---

## Overview

This chapter provides the **definitive comparison** of all sorting algorithms covered in this course. Use these tables as a quick-reference guide for choosing the right algorithm, understanding trade-offs, and preparing for interviews/exams.

---

## Master Comparison Table

```
  ┌──────────────┬──────────┬───────────┬──────────┬───────┬────────┬────────┬──────────┐
  │  Algorithm   │ Best     │ Average   │ Worst    │ Space │ Stable │In-place│ Type     │
  ├──────────────┼──────────┼───────────┼──────────┼───────┼────────┼────────┼──────────┤
  │ Bubble Sort  │ O(n)     │ O(n²)     │ O(n²)    │ O(1)  │  Yes   │  Yes   │ Compare  │
  │ Selection    │ O(n²)    │ O(n²)     │ O(n²)    │ O(1)  │  No*   │  Yes   │ Compare  │
  │ Insertion    │ O(n)     │ O(n²)     │ O(n²)    │ O(1)  │  Yes   │  Yes   │ Compare  │
  │ Merge Sort   │ O(n lg n)│ O(n lg n) │ O(n lg n)│ O(n)  │  Yes   │  No    │ Compare  │
  │ Quick Sort   │ O(n lg n)│ O(n lg n) │ O(n²)    │O(lg n)│  No    │  Yes   │ Compare  │
  │ Heap Sort    │ O(n lg n)│ O(n lg n) │ O(n lg n)│ O(1)  │  No    │  Yes   │ Compare  │
  │ Counting     │ O(n+k)   │ O(n+k)    │ O(n+k)   │O(n+k) │  Yes   │  No    │ Non-comp │
  │ Radix Sort   │ O(dn)    │ O(dn)     │ O(dn)    │O(n+k) │  Yes   │  No    │ Non-comp │
  │ Bucket Sort  │ O(n+k)   │ O(n)      │ O(n²)    │O(n+k) │ Dep.   │  No    │ Non-comp │
  └──────────────┴──────────┴───────────┴──────────┴───────┴────────┴────────┴──────────┘
  
  * Selection sort can be made stable but standard implementation is not
  lg n = log₂(n)    k = range of values    d = number of digits
```

---

## Visualizing Growth Rates

```
  Time
  │
  │                                          ╱ O(n²)
  │                                        ╱
  │                                      ╱
  │                                   ╱
  │                                ╱
  │                            ╱         ╱  O(n log n)
  │                        ╱          ╱
  │                    ╱           ╱
  │               ╱            ╱
  │          ╱             ╱                ╱ O(n)
  │     ╱              ╱               ╱
  │ ╱              ╱              ╱
  │─────────────────────────────────────────── n
  
  At n = 1,000,000:
  O(n²)      = 1,000,000,000,000  (1 trillion)
  O(n log n) =    20,000,000      (20 million)
  O(n)       =     1,000,000      (1 million)
  
  That's 50,000x difference between O(n²) and O(n log n)!
```

---

## Comparison by Number of Comparisons

```
  n = 1,000 elements:
  
  ┌──────────────┬───────────────┬───────────────┬───────────────┐
  │  Algorithm   │ Best Case     │ Average       │ Worst Case    │
  ├──────────────┼───────────────┼───────────────┼───────────────┤
  │ Bubble Sort  │ 999           │ ~250,000      │ 499,500       │
  │ Selection    │ 499,500       │ 499,500       │ 499,500       │
  │ Insertion    │ 999           │ ~250,000      │ 499,500       │
  │ Merge Sort   │ ~5,000        │ ~10,000       │ ~10,000       │
  │ Quick Sort   │ ~10,000       │ ~10,000       │ 499,500       │
  │ Heap Sort    │ ~10,000       │ ~20,000       │ ~20,000       │
  │ Counting     │ 0             │ 0             │ 0             │
  │ Radix        │ 0             │ 0             │ 0             │
  │ Bucket       │ varies        │ varies        │ varies        │
  └──────────────┴───────────────┴───────────────┴───────────────┘
```

---

## Comparison by Number of Swaps / Writes

```
  ┌──────────────┬─────────────────┬─────────────────────────────┐
  │  Algorithm   │ Swaps/Writes    │  Notes                      │
  ├──────────────┼─────────────────┼─────────────────────────────┤
  │ Bubble Sort  │ O(n²) swaps     │ Many swaps, each O(1)      │
  │ Selection    │ O(n) swaps      │ Best for minimizing writes │
  │ Insertion    │ O(n²) shifts    │ Shifts, not full swaps     │
  │ Merge Sort   │ O(n lg n) copies│ Copies to auxiliary array  │
  │ Quick Sort   │ O(n lg n) swaps │ Efficient partitioning     │
  │ Heap Sort    │ O(n lg n) swaps │ Heapify + extract          │
  │ Counting     │ O(n+k) writes   │ No swaps, direct placement │
  │ Radix        │ d × O(n) writes │ Per digit pass             │
  │ Bucket       │ O(n+inner) var. │ Distribution + inner sort  │
  └──────────────┴─────────────────┴─────────────────────────────┘
  
  Winner for minimum writes: Selection Sort (only n-1 swaps)
  → Best choice when writing to memory is expensive (e.g., flash/EEPROM)
```

---

## Comparison by Adaptivity

An algorithm is **adaptive** if it benefits from existing order in the input.

```
  ┌──────────────┬──────────┬────────────────────────────────────┐
  │  Algorithm   │ Adaptive?│ On nearly sorted data              │
  ├──────────────┼──────────┼────────────────────────────────────┤
  │ Bubble Sort  │ Yes      │ O(n) with early-exit optimization │
  │ Selection    │ No       │ Always O(n²)                      │
  │ Insertion    │ Yes ✓    │ O(n) — best adaptive sort         │
  │ Merge Sort   │ No*      │ Always O(n lg n)                  │
  │ Quick Sort   │ No       │ Could be O(n²) with bad pivot!    │
  │ Heap Sort    │ No       │ Always O(n lg n)                  │
  │ Counting     │ No       │ Always O(n+k)                     │
  │ Radix        │ No       │ Always O(dn)                      │
  │ Bucket       │ No       │ Depends on inner sort             │
  │ Timsort**    │ Yes ✓    │ O(n) — designed for real data     │
  └──────────────┴──────────┴────────────────────────────────────┘
  
  * Natural merge sort IS adaptive
  ** Timsort = Merge + Insertion hybrid (Python/Java default)
```

---

## Comparison by Practical Performance

```
  Real-world speed depends on more than Big-O.
  
  CACHE PERFORMANCE:
  ┌──────────────┬──────────────────────────────────────────────┐
  │ Cache-friendly│ Insertion, Quick Sort, Merge (bottom-up)   │
  │ Cache-poor    │ Heap Sort (jumps around), Merge (top-down) │
  └──────────────┴──────────────────────────────────────────────┘
  
  CONSTANT FACTOR:
  ┌──────────────┬──────────────────────────────────────────────┐
  │ Low overhead  │ Insertion Sort, Quick Sort                  │
  │ High overhead │ Heap Sort (sift-down), Merge (copies)      │
  └──────────────┴──────────────────────────────────────────────┘
  
  BRANCH PREDICTION:
  ┌──────────────┬──────────────────────────────────────────────┐
  │ Predictable   │ Insertion Sort, Merge Sort                  │
  │ Unpredictable │ Quick Sort (partition), Heap Sort           │
  └──────────────┴──────────────────────────────────────────────┘
```

---

## The Big Picture: Which Sort to Use?

```
  ┌───────────────────────────────────────────────────────────────────┐
  │  In practice, you almost always use your language's built-in     │
  │  sort (Timsort, Introsort, etc.). But knowing WHEN each         │
  │  algorithm shines helps you understand WHY built-in sorts       │
  │  are designed the way they are.                                  │
  └───────────────────────────────────────────────────────────────────┘
  
  Small n (≤ 16):       Insertion Sort
  General purpose:      Quick Sort / Introsort / Timsort
  Guaranteed O(n lg n): Merge Sort or Heap Sort
  Integers, small k:    Counting Sort
  Fixed-width keys:     Radix Sort
  Uniform floats:       Bucket Sort
  Nearly sorted:        Insertion Sort / Timsort
  Linked lists:         Merge Sort
  External (disk):      Merge Sort (external)
  Minimize writes:      Selection Sort or Cycle Sort
  Educational:          Bubble Sort (never in production!)
```

---

## Summary Table

| Criteria | Best Algorithm | Why |
|----------|----------------|-----|
| Simplicity | Bubble / Insertion | Easy to code & understand |
| General purpose | Quick Sort / Timsort | Best avg performance |
| Worst-case guarantee | Merge / Heap | O(n lg n) always |
| Space efficiency | Heap Sort | O(1) extra, O(n lg n) time |
| Stability | Merge Sort | Stable + O(n lg n) |
| Small arrays | Insertion Sort | Low overhead |
| Integer range | Counting Sort | O(n+k) guaranteed |
| Large integers | Radix Sort | O(dn) |
| Uniform floats | Bucket Sort | O(n) average |

---

## Quick Revision Questions

1. **Which O(n log n) sort uses O(1) extra space?**
2. **Which sort performs the fewest number of writes/swaps?**
3. **Name all stable sorting algorithms from this course.**
4. **Why is Quick Sort faster than Merge Sort in practice despite both being O(n log n)?**
5. **Which sort would you use for 10 million uniformly distributed doubles?**
6. **Create a decision tree for choosing a sort algorithm given data characteristics.**

---

[← Previous: Unit 10 — Bucket Sort](../10-Bucket-Sort/06-comparison-with-other-sorts.md) | [Next: Stability Analysis →](02-stability-analysis.md)
