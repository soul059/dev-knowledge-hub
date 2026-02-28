# Unit 10: Bucket Sort — Comparison with Other Sorts

[← Previous: Hybrid Approaches](05-hybrid-approaches.md) | [Next: Unit 11 — Sorting Algorithm Comparison →](../11-Sorting-Algorithm-Comparison/01-comparison-table.md)

---

## Overview

This chapter places Bucket Sort in the context of all sorting algorithms studied so far, highlighting when it excels and when other algorithms are better choices.

---

## The Three Linear-Time Sorts

```
  ┌─────────────────────────────────────────────────────────────────────┐
  │                   LINEAR-TIME SORTING ALGORITHMS                   │
  ├──────────────────┬──────────────────┬───────────────────────────────┤
  │  COUNTING SORT   │  RADIX SORT      │  BUCKET SORT                 │
  │                  │                  │                               │
  │  Count each      │  Sort digit by   │  Distribute into             │
  │  value, build    │  digit using a   │  range-based buckets,        │
  │  prefix sums     │  stable sub-sort │  sort each bucket            │
  │                  │                  │                               │
  │  When: small     │  When: fixed-    │  When: data is               │
  │  integer range   │  length keys     │  uniformly distributed       │
  │                  │                  │                               │
  │  O(n+k)          │  O(d(n+k))       │  O(n) avg, O(n²) worst      │
  │  guaranteed      │  guaranteed      │  NOT guaranteed              │
  └──────────────────┴──────────────────┴───────────────────────────────┘
```

---

## Counting Sort vs Bucket Sort

```
  ┌──────────────────┬──────────────────────┬────────────────────────┐
  │  Aspect          │  Counting Sort       │  Bucket Sort           │
  ├──────────────────┼──────────────────────┼────────────────────────┤
  │  Input           │  Integers only       │  Any comparable type   │
  │  Range needed?   │  Yes (must know k)   │  Yes (min/max)         │
  │  Time            │  O(n+k) always       │  O(n) avg, O(n²) worst│
  │  Space           │  O(n+k)              │  O(n+k)               │
  │  Stable?         │  Yes                 │  If inner sort stable  │
  │  In-place?       │  No                  │  No                   │
  │  Works on floats?│  No                  │  Yes ✓                │
  │  Works on objects?│ No (needs int keys) │  Yes ✓                │
  │  Distribution    │  Doesn't matter      │  Must be ~uniform     │
  │  Worst-case safe?│  Yes ✓              │  No ✗                 │
  └──────────────────┴──────────────────────┴────────────────────────┘
  
  Key difference:
  Counting Sort counts EXACT values → guaranteed O(n+k)
  Bucket Sort distributes into RANGES → depends on distribution
```

### When to prefer Counting Sort

- Integer data with small range (k ≤ 10n)
- Need guaranteed worst-case performance
- Repeated values are common

### When to prefer Bucket Sort

- Floating-point numbers
- Continuous data
- Data is known to be uniformly distributed

---

## Radix Sort vs Bucket Sort

```
  ┌──────────────────┬──────────────────────┬────────────────────────┐
  │  Aspect          │  Radix Sort          │  Bucket Sort           │
  ├──────────────────┼──────────────────────┼────────────────────────┤
  │  Approach        │  Digit-by-digit      │  Single distribution   │
  │  Passes          │  d passes            │  1 pass + inner sorts  │
  │  Time            │  O(d(n+k))           │  O(n) avg              │
  │  Data type       │  Fixed-length keys   │  Any comparable        │
  │  Distribution    │  Doesn't matter      │  Needs uniformity      │
  │  Worst case      │  O(d(n+k))           │  O(n²)                │
  │  Stability       │  Required (uses it)  │  Optional              │
  │  Sub-sort        │  Counting sort       │  Any sort              │
  └──────────────────┴──────────────────────┴────────────────────────┘
```

### Conceptual relationship

```
  Radix Sort ≈ Multi-level Bucket Sort
  
  Radix sorts by EACH DIGIT → like having buckets within buckets
  
  MSD Radix Sort:
    Level 1: bucket by 100s digit    [0xx] [1xx] [2xx] ...
    Level 2: bucket by 10s digit     [00x] [01x] [02x] ...
    Level 3: bucket by 1s digit      [000] [001] [002] ...
  
  Bucket Sort: single-level distribution into ranges
    [0-99] [100-199] [200-299] ...
    Then sort within each range
  
  MSD Radix = recursive bucket sort with base-k buckets!
```

---

## Comparison-Based Sorts vs Bucket Sort

```
  ┌────────────────────┬──────────┬───────────┬───────────────────┐
  │  Algorithm         │ Avg Time │ Worst     │ Key Requirement   │
  ├────────────────────┼──────────┼───────────┼───────────────────┤
  │ Merge Sort         │ O(n lg n)│ O(n lg n) │ None              │
  │ Quick Sort         │ O(n lg n)│ O(n²)     │ Good pivots       │
  │ Heap Sort          │ O(n lg n)│ O(n lg n) │ None              │
  │ Insertion Sort     │ O(n²)    │ O(n²)     │ None (good if ~sorted)│
  │ Bucket Sort        │ O(n)     │ O(n²)     │ Uniform distrib.  │
  └────────────────────┴──────────┴───────────┴───────────────────┘
  
  Bucket sort breaks the O(n log n) barrier because it is
  NOT a comparison-based sort — it uses VALUE INFORMATION
  to distribute elements, not just comparisons.
```

---

## Decision Flowchart

```
  ┌─────────────────────────────────┐
  │  What kind of data do you have? │
  └──────────┬──────────────────────┘
             │
       ┌─────┴─────────────┐
       ▼                   ▼
  ┌─────────┐        ┌──────────────┐
  │ Integers │        │ Float/Double  │
  └────┬────┘        └──────┬───────┘
       │                    │
       ▼                    ▼
  ┌──────────┐      ┌───────────────────┐
  │ Range    │      │ Uniform distrib.? │
  │ small?   │      └───────┬───────────┘
  └────┬─────┘         ┌────┴───┐
  Yes  │  No           ▼        ▼
  ▼    ▼          ┌────────┐ ┌──────┐
Count  ▼          │  Yes   │ │  No  │
Sort  Fixed      └────┬───┘ └──┬───┘
      width?          ▼        ▼
      ▼    ▼     Bucket     Merge Sort
     Radix  ▼     Sort      or Quick Sort
     Sort  Quick
           Sort
```

---

## When Bucket Sort Wins

```
  ✓ Sorting millions of uniformly distributed floats in [0, 1)
  ✓ Sorting sensor readings within a known, stable range
  ✓ Histogram generation (bucket phase is the answer itself)
  ✓ External sorting with range partitioning
  ✓ Data already has a known, uniform-ish distribution
  ✓ When combined with other sorts (hybrid approaches)
```

## When Bucket Sort Loses

```
  ✗ Unknown or skewed distribution → Quick/Merge Sort
  ✗ Small integer range → Counting Sort
  ✗ Fixed-width integer keys → Radix Sort
  ✗ Memory constrained → Heap Sort or Quick Sort (in-place)
  ✗ Need guaranteed worst-case → Merge Sort or Heap Sort
  ✗ Nearly sorted data → Insertion Sort or Timsort
  ✗ Small arrays (n < 50) → Insertion Sort
```

---

## Comprehensive Comparison Table

| Feature | Bubble | Selection | Insertion | Merge | Quick | Heap | Counting | Radix | **Bucket** |
|---------|--------|-----------|-----------|-------|-------|------|----------|-------|------------|
| Avg Time | O(n²) | O(n²) | O(n²) | O(n lg n) | O(n lg n) | O(n lg n) | O(n+k) | O(dn) | **O(n)** |
| Worst | O(n²) | O(n²) | O(n²) | O(n lg n) | O(n²) | O(n lg n) | O(n+k) | O(dn) | **O(n²)** |
| Space | O(1) | O(1) | O(1) | O(n) | O(lg n) | O(1) | O(n+k) | O(n+k) | **O(n+k)** |
| Stable | Yes | No | Yes | Yes | No | No | Yes | Yes | **Depends** |
| In-place | Yes | Yes | Yes | No | Yes | Yes | No | No | **No** |
| Comparison | Yes | Yes | Yes | Yes | Yes | Yes | No | No | **No** |
| Data type | Any | Any | Any | Any | Any | Any | Int | Int/Str | **Any** |

---

## Summary Table

| Comparison | Winner | Reason |
|-----------|--------|--------|
| Bucket vs Counting | Counting (integers) | Guaranteed O(n+k), no distribution assumption |
| Bucket vs Radix | Radix (fixed keys) | Guaranteed O(dn), no distribution assumption |
| Bucket vs Merge | Bucket (uniform data) | O(n) vs O(n log n) |
| Bucket vs Quick | Bucket (uniform data) | O(n) vs O(n log n) avg |
| Bucket vs Heap | Bucket (uniform data) | O(n) vs O(n log n), but heap uses O(1) space |
| Bucket on skewed data | Quick/Merge | Bucket degrades to O(n²) |

---

## Quick Revision Questions

1. **Why can bucket sort break the O(n log n) barrier while merge sort cannot?**
2. **Compare counting sort and bucket sort: when would you choose each?**
3. **How is MSD radix sort conceptually equivalent to recursive bucket sort?**
4. **Given 1 million uniformly distributed doubles in [0,1), which sort is fastest? Why?**
5. **If you don't know the distribution of your data, should you use bucket sort? What alternatives exist?**
6. **Create a decision table for choosing between counting, radix, and bucket sort.**

---

[← Previous: Hybrid Approaches](05-hybrid-approaches.md) | [Next: Unit 11 — Sorting Algorithm Comparison →](../11-Sorting-Algorithm-Comparison/01-comparison-table.md)
