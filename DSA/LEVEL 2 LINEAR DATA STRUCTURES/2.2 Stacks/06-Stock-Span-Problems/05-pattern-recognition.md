# Chapter 5: Pattern Recognition — When to Use Monotonic Stack

[← Previous: Trapping Rain Water](04-trapping-rain-water.md) | [Next: Optimizing Brute Force with Stack →](06-optimizing-brute-force.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)

---

## Overview

Recognizing **when** to apply a monotonic stack is often harder than implementing one. This chapter provides a systematic framework to identify monotonic stack problems through **key phrases**, **problem patterns**, and **decision trees**.

---

## Pattern Recognition Framework

```
┌──────────────────────────────────────────────────────────┐
│               MONOTONIC STACK SIGNALS                    │
│                                                          │
│  Ask these questions about the problem:                  │
│                                                          │
│  1. Does it involve NEAREST/NEXT/PREVIOUS element        │
│     satisfying a comparison (>, <, ≥, ≤)?               │
│                                                          │
│  2. Does it involve SPANS or RANGES extending            │
│     left/right until a condition breaks?                 │
│                                                          │
│  3. Is there a BRUTE FORCE O(n²) where for each         │
│     element, you scan left or right?                     │
│                                                          │
│  4. Does it involve AREA calculation with heights?       │
│                                                          │
│  If YES to any → consider monotonic stack!               │
└──────────────────────────────────────────────────────────┘
```

---

## Key Phrases That Signal Monotonic Stack

```
┌────────────────────────────────┬──────────────────────────┐
│ Phrase in Problem              │ Stack Pattern            │
├────────────────────────────────┼──────────────────────────┤
│ "next greater element"         │ Decreasing stack, L→R    │
│ "next smaller element"         │ Increasing stack, L→R    │
│ "previous greater element"     │ Decreasing stack, L→R    │
│ "previous smaller element"     │ Increasing stack, L→R    │
│ "days until warmer"            │ Decreasing (NGE variant) │
│ "span" or "consecutive"       │ Decreasing (PGE variant) │
│ "largest rectangle"            │ Increasing (NSE+PSE)     │
│ "water trapped"                │ Decreasing               │
│ "remove digits" / "smallest"  │ Increasing               │
│ "asteroid collision"           │ Custom conditions        │
│ "first bar taller/shorter"    │ Monotonic stack           │
│ "sliding window maximum"      │ Decreasing deque          │
└────────────────────────────────┴──────────────────────────┘
```

---

## Decision Tree

```
Problem involves array/sequence
         │
         ▼
   Looking for nearest/first
   element in one direction?
     │            │
    YES          NO
     │            │
     ▼            ▼
  Greater or    Involves area
  smaller?      or spans?
   │     │       │        │
  >/>≥  </<≤   YES       NO
   │     │       │        │
   ▼     ▼       ▼        ▼
  DEC.  INC.  Histogram  Probably
  STACK STACK  Related    not stack
               │
               ▼
          Need both PSE
          and NSE (or PGE/NGE)
```

---

## Common Problem ↔ Stack Mapping

### Category 1: Direct NGE/NSE Variants

```
┌────────────────────────────────────────────────────────┐
│ Problem                │ Type    │ Stack    │ Detail   │
├────────────────────────┼─────────┼──────────┼──────────┤
│ Next Greater Element   │ NGE     │ Decr.    │ L→R      │
│ Next Greater (Circular)│ NGE     │ Decr.    │ 2n iter  │
│ Daily Temperatures     │ NGE     │ Decr.    │ Distance │
│ Next Smaller Element   │ NSE     │ Incr.    │ L→R      │
│ Stock Span             │ PGE     │ Decr.    │ Distance │
│ Sum of Subarray Mins   │ NSE+PSE │ Incr.    │ Both     │
│ Sum of Subarray Maxs   │ NGE+PGE │ Decr.    │ Both     │
└────────────────────────┴─────────┴──────────┴──────────┘
```

### Category 2: Area/Volume Problems

```
┌────────────────────────────────────────────────────────┐
│ Problem                │ Uses         │ Stack          │
├────────────────────────┼──────────────┼────────────────┤
│ Largest Rect Histogram │ PSE + NSE    │ Increasing     │
│ Maximal Rectangle      │ Histogram/row│ Increasing     │
│ Trapping Rain Water    │ Boundaries   │ Decreasing     │
│ Container Most Water   │ Two pointers │ NOT stack      │
└────────────────────────┴──────────────┴────────────────┘
```

### Category 3: Optimization/Greedy with Stack

```
┌────────────────────────────────────────────────────────┐
│ Problem                │ Property     │ Stack          │
├────────────────────────┼──────────────┼────────────────┤
│ Remove K Digits        │ Smallest num │ Increasing     │
│ Remove Duplicate Letters│ Lexicographic│ Increasing     │
│ 132 Pattern            │ Special      │ Decreasing     │
│ Asteroid Collision     │ Custom logic │ Mixed          │
└────────────────────────┴──────────────┴────────────────┘
```

---

## Anti-Patterns: When NOT to Use Stack

```
┌──────────────────────────────────────────────────────────┐
│  ✗ Finding ALL greater/smaller elements (not just next) │
│  ✗ Counting inversions (use merge sort)                 │
│  ✗ Range minimum/maximum queries (use segment tree/ST)  │
│  ✗ Sliding window with fixed size k (use deque, not     │
│    plain stack, though deque IS a double-ended stack)    │
│  ✗ Two-sum / pair problems (use hash map)               │
│  ✗ Sorted order maintenance (use BST/heap)              │
└──────────────────────────────────────────────────────────┘
```

---

## Choosing the Stack Type

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Want to find GREATER elements?                          │
│    → Use DECREASING stack                                │
│    → Pop smaller elements (they found their answer)      │
│                                                          │
│  Want to find SMALLER elements?                          │
│    → Use INCREASING stack                                │
│    → Pop larger elements (they found their answer)       │
│                                                          │
│  Memory aid:                                             │
│    The stack KEEPS elements of the type you're looking   │
│    for, and POPS elements that found their answer.       │
│                                                          │
│    Decreasing stack KEEPS large elements                 │
│    → because we're looking for the next GREATER          │
│                                                          │
│    Increasing stack KEEPS small elements                 │
│    → because we're looking for the next SMALLER          │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Practice Problem Classification Exercise

```
Classify each problem: which stack type?

1. "Find for each building, the nearest taller building to the right"
   → NGE → Decreasing stack ✓

2. "For each temperature, days until a colder day"
   → NSE with distance → Increasing stack ✓

3. "Maximum width ramp: max j-i such that arr[i] ≤ arr[j]"
   → Modified monotonic approach → Decreasing stack ✓

4. "Online stock span"
   → PGE with distance → Decreasing stack ✓

5. "Sum of all min(subarray) for every subarray"
   → NSE + PSE → Increasing stack ✓
```

---

## Summary Table

| Signal | Stack Type | Direction |
|--------|-----------|-----------|
| Next Greater | Decreasing | L→R |
| Next Smaller | Increasing | L→R |
| Previous Greater | Decreasing | L→R (top after pop) |
| Previous Smaller | Increasing | L→R (top after pop) |
| Histogram area | Increasing | L→R with sentinel |
| Rain water | Decreasing | L→R |
| Remove digits (smallest) | Increasing | L→R |

---

## Quick Revision Questions

1. **What is the primary signal that a problem uses monotonic stack?**
   > When you need to find the nearest/next/previous element that is greater or smaller — especially when brute force would scan linearly for each element.

2. **How do you decide between increasing and decreasing stack?**
   > Looking for greater → decreasing stack. Looking for smaller → increasing stack. The stack keeps the type you're searching for.

3. **When should you NOT use a monotonic stack?**
   > When you need ALL greater/smaller elements (not just nearest), range queries, or fixed-size sliding windows.

4. **What brute force complexity suggests a monotonic stack optimization?**
   > O(n²) where for each element you scan left or right to find the first element satisfying a comparison.

5. **How do area problems (histogram, rain water) relate to monotonic stack?**
   > They need boundaries (PSE/NSE) to determine how far a height extends, which is exactly what monotonic stacks compute.

---

[← Previous: Trapping Rain Water](04-trapping-rain-water.md) | [Next: Optimizing Brute Force with Stack →](06-optimizing-brute-force.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)
