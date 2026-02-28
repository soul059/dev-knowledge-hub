# Unit 11: Sorting Algorithm Comparison — Choosing the Right Algorithm

[← Previous: Space Complexity Analysis](04-space-complexity-analysis.md) | [Next: Language Built-in Sorts →](06-language-builtin-sorts.md)

---

## Overview

This chapter provides a **practical decision framework** for choosing the right sorting algorithm based on data characteristics, constraints, and requirements.

---

## The Ultimate Decision Flowchart

```
  START: Need to sort data
         │
         ▼
  ┌──────────────────────────┐
  │  How large is n?         │
  └──────┬───────────────────┘
         │
    ┌────┴────────────┐
    ▼                 ▼
  n ≤ 50          n > 50
    │                 │
    ▼                 ▼
  Insertion     ┌──────────────────────┐
  Sort          │ Is data integer with │
  (fastest for  │ small range k?       │
   tiny arrays) └──────┬───────────────┘
                  ┌────┴────────┐
                  ▼             ▼
                 Yes            No
                  │             │
                  ▼             ▼
              ┌────────┐  ┌───────────────────────┐
              │k ≤ n?  │  │ Is data float/string? │
              └───┬────┘  └──────┬────────────────┘
             Yes  │  No         │
              ▼   ▼        ┌────┴────────┐
          Counting │       ▼             ▼
           Sort    │   Uniformly      Not uniform
                   │   distributed?      │
                   ▼       │             ▼
              Fixed-width  ▼         ┌───────────────┐
              keys?    Bucket        │Need stability?│
               │        Sort         └──────┬────────┘
          Yes  │  No                   Yes  │  No
           ▼   ▼                        ▼   ▼
         Radix  ▼                    Merge  Quick Sort
         Sort  ┌──────────────────┐  Sort  (or Introsort)
               │Need guaranteed   │
               │O(n lg n)?        │
               └──────┬───────────┘
                  Yes  │  No
                   ▼   ▼
                 Merge  Quick Sort
                 Sort   (Introsort)
```

---

## Decision Matrix

```
  ┌─────────────────────────┬──────────────────────────────────┐
  │  CONSTRAINT             │  BEST ALGORITHM                  │
  ├─────────────────────────┼──────────────────────────────────┤
  │  General purpose        │  Quick Sort / Introsort          │
  │  Stability required     │  Merge Sort / Timsort            │
  │  O(1) extra space       │  Heap Sort                       │
  │  Nearly sorted data     │  Insertion Sort / Timsort        │
  │  Small n (≤ 50)         │  Insertion Sort                  │
  │  Integers, small range  │  Counting Sort                   │
  │  Fixed-width integers   │  Radix Sort                      │
  │  Uniform distribution   │  Bucket Sort                     │
  │  Linked list            │  Merge Sort                      │
  │  External (disk)        │  External Merge Sort             │
  │  Minimize writes        │  Selection Sort / Cycle Sort     │
  │  Worst-case guarantee   │  Merge Sort or Heap Sort         │
  │  Parallel sorting       │  Merge Sort / Bitonic Sort       │
  │  Online (streaming)     │  Insertion Sort                  │
  └─────────────────────────┴──────────────────────────────────┘
```

---

## Scenario-Based Guide

### Scenario 1: Sorting Student Records by GPA

```
  Data: 500 students, GPA range 0.0 to 4.0
  Requirements: Stable (preserve alphabetical order within same GPA)
  
  Analysis:
  • n = 500 (moderate)
  • Floating-point keys → can't use counting sort
  • Stability needed → eliminates Quick Sort, Heap Sort
  • Known range [0, 4.0] → bucket sort possible
  
  Best choices:
  1. Merge Sort   — stable, guaranteed O(n lg n)
  2. Bucket Sort   — if GPAs roughly uniform (they might be!)
  3. Timsort       — built-in, stable, adaptive
  
  Recommendation: Use language's built-in sort (Timsort/stable)
```

### Scenario 2: Sorting 10 Million Log Timestamps

```
  Data: 10⁷ Unix timestamps (32-bit integers)
  Requirements: Fast, memory available
  
  Analysis:
  • n = 10⁷ (large)
  • Integer keys, fixed width (32 bits = 4 bytes)
  • Range: ~10⁹ (too large for counting sort)
  • 4 bytes = 4 digits in base 256
  
  Best choices:
  1. Radix Sort (base 256, 4 passes) — O(4n) ≈ O(n)
  2. Quick Sort — O(n lg n) ≈ 23n operations
  3. Counting Sort — range too large (10⁹)
  
  Recommendation: Radix Sort (base 256) — ~4x faster than Quick Sort
```

### Scenario 3: Embedded Sensor Sorting

```
  Data: 200 temperature readings (integers 0-100)
  Requirements: O(1) extra space, fast
  
  Analysis:
  • n = 200 (small)
  • Integer range k = 101
  • Space constrained
  
  Best choices:
  1. Insertion Sort — O(1) space, n is small enough
  2. Heap Sort — O(1) space, O(n lg n)
  3. Counting Sort — O(k) = O(101) extra, but very fast
  
  Recommendation: Counting Sort if 101 ints fit; else Insertion Sort
```

### Scenario 4: Sorting Strings by Length

```
  Data: 100,000 variable-length strings
  Requirements: Stable, general purpose
  
  Analysis:
  • n = 100,000 (large)
  • String comparison is O(length) per comparison
  • Variable length → MSD Radix possible but complex
  • Need stability
  
  Best choices:
  1. Timsort (built-in) — stable, O(n lg n × L) where L = avg length
  2. MSD Radix Sort — O(n × W) where W = max string length
  3. Three-way String Quick Sort — excellent for strings
  
  Recommendation: Built-in sort for simplicity; MSD Radix for max speed
```

### Scenario 5: Real-Time Streaming Data

```
  Data: Continuous stream, need sorted order at all times
  Requirements: Insert each element into sorted position
  
  Analysis:
  • Online algorithm needed
  • Each insertion should be fast
  
  Best choices:
  1. Insertion Sort — O(n) per insert into sorted array
  2. Binary search + shift — O(lg n) search + O(n) shift
  3. Balanced BST / Skip List — O(lg n) per insert
  4. Priority Queue (Heap) — O(lg n) per insert
  
  Recommendation: Balanced BST or Heap for true streaming
```

---

## Common Interview Questions and Answers

```
  Q: "Which sorting algorithm is best?"
  A: There is no single best algorithm. It depends on:
     - Data size, type, and distribution
     - Space constraints
     - Stability requirements
     - Whether data is partially sorted
  
  Q: "Why not always use Quick Sort?"
  A: Quick Sort fails when:
     - Stability is required
     - Worst-case O(n²) is unacceptable
     - Data is in a linked list (poor random access)
  
  Q: "When is O(n²) acceptable?"
  A: When n is very small (≤ 50). The constant factor matters
     more than the growth rate for small inputs.
  
  Q: "Why do programming languages use hybrid sorts?"
  A: To combine the best properties:
     - Timsort = Merge (stability, O(n lg n)) + Insertion (small n, adaptive)
     - Introsort = Quick (speed) + Heap (worst-case) + Insertion (small n)
```

---

## Anti-Patterns: When NOT to Use

```
  ✗ DON'T use Bubble Sort for anything other than teaching
  ✗ DON'T use Selection Sort when stability matters
  ✗ DON'T use Counting Sort when range k >> n
  ✗ DON'T use Bucket Sort on unknown distributions
  ✗ DON'T use Merge Sort when memory is extremely tight
  ✗ DON'T use Quick Sort when worst-case O(n²) is unacceptable
  ✗ DON'T use Heap Sort when stability is required
  ✗ DON'T implement your own sort unless you have a specific reason
  ✗ DON'T use O(n²) sorts when n > 10,000
```

---

## Algorithm Selection Cheat Sheet

```
  ┌─────────────────────────────────────────────────────────────┐
  │               QUICK SELECTION GUIDE                        │
  ├───────────────────┬─────────────────────────────────────────┤
  │  "Just sort it"   │ Use your language's built-in sort      │
  │  "It's mostly     │ Timsort or Insertion Sort              │
  │   sorted"         │                                        │
  │  "Small array"    │ Insertion Sort (n ≤ 50)                │
  │  "Integer keys"   │ Counting (small k) or Radix (large k) │
  │  "Must be stable" │ Merge Sort or Timsort                  │
  │  "No extra RAM"   │ Heap Sort                              │
  │  "Linked list"    │ Merge Sort                             │
  │  "Huge dataset    │ External Merge Sort                    │
  │   on disk"        │                                        │
  │  "Fastest average"│ Quick Sort with median-of-3            │
  │  "Never O(n²)"    │ Merge Sort or Heap Sort                │
  │  "Uniform floats" │ Bucket Sort                            │
  └───────────────────┴─────────────────────────────────────────┘
```

---

## Summary Table

| Scenario | Recommended Algorithm | Reason |
|----------|----------------------|--------|
| Default / general | Language built-in | Optimized hybrid (Timsort/Introsort) |
| Small n (≤ 50) | Insertion Sort | Low overhead |
| Stability required | Merge Sort | Stable + O(n lg n) |
| O(1) space | Heap Sort | Only O(1)-space O(n lg n) sort |
| Nearly sorted | Insertion Sort | O(n) on sorted data |
| Integers, small range | Counting Sort | O(n+k) guaranteed |
| Large integers | Radix Sort | O(dn), faster than comparison |
| Uniform floats | Bucket Sort | O(n) average |
| Linked list | Merge Sort | O(1) space, natural fit |

---

## Quick Revision Questions

1. **You need to sort 1 million 64-bit integers with O(1) extra space. What do you choose?**
2. **Your data is 90% sorted with a few elements out of place. Best algorithm?**
3. **You have 10,000 strings. Stability is required. What sort?**
4. **You're sorting on a system with only 1KB free memory. Available options?**
5. **Design a decision tree for a library's `sort()` function that handles all cases.**
6. **Why do all major languages use hybrid sorts instead of pure algorithms?**

---

[← Previous: Space Complexity Analysis](04-space-complexity-analysis.md) | [Next: Language Built-in Sorts →](06-language-builtin-sorts.md)
