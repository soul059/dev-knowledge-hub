# Chapter 5: Selection Sort vs Bubble Sort

[← Previous: Stability Consideration](04-stability-consideration.md) | [Next Unit: Insertion Sort →](../04-Insertion-Sort/01-algorithm-description.md)

---

## Overview

Bubble Sort and Selection Sort are both O(n²) elementary sorts, but they have different strengths. This chapter provides a detailed head-to-head comparison.

---

## Side-by-Side Mechanism

```
  BUBBLE SORT                           SELECTION SORT
  ═══════════                           ══════════════
  
  Compare ADJACENT pairs,               Find MINIMUM in unsorted,
  swap if out of order.                  swap to front.
  
  [5, 3, 8, 1, 2]                      [5, 3, 8, 1, 2]
   ↕                                    scan all → min=1
  [3, 5, 8, 1, 2]                      swap 5↔1
      ↕                                [1, 3, 8, 5, 2]
  [3, 5, 8, 1, 2]                       ✓
         ↕
  [3, 5, 1, 8, 2]                      Next pass: scan [3,8,5,2]
            ↕                           min=2, swap 3↔2
  [3, 5, 1, 2, 8]  ← 8 placed         [1, 2, 8, 5, 3]
                ✓                        ✓  ✓
  
  Largest BUBBLES UP                    Smallest SELECTED and placed
  Multiple swaps per pass               ONE swap per pass
```

---

## Detailed Comparison Table

```
  ┌──────────────────────┬─────────────────────┬─────────────────────┐
  │     Property         │   Bubble Sort       │   Selection Sort    │
  ├──────────────────────┼─────────────────────┼─────────────────────┤
  │ Best case time       │ O(n) (optimized)    │ O(n²) always        │
  │ Average case time    │ O(n²)               │ O(n²)               │
  │ Worst case time      │ O(n²)               │ O(n²)               │
  │ Space                │ O(1)                │ O(1)                │
  │ Stable               │ YES ✓               │ NO ✗                │
  │ Adaptive             │ YES (with flag)     │ NO                  │
  │ Comparisons (worst)  │ n(n-1)/2            │ n(n-1)/2            │
  │ Swaps (worst)        │ n(n-1)/2            │ n-1                 │
  │ Swaps (best)         │ 0                   │ 0                   │
  │ Online               │ NO                  │ NO                  │
  │ Method               │ Adjacent swapping   │ Find min + swap     │
  └──────────────────────┴─────────────────────┴─────────────────────┘
```

---

## Swap Count Comparison

```
  Array: [5, 4, 3, 2, 1]  (Reverse sorted — worst case)
  
  BUBBLE SORT SWAPS:
  Pass 0: 4 swaps  (5↔4, 5↔3, 5↔2, 5↔1)
  Pass 1: 3 swaps  (4↔3, 4↔2, 4↔1)
  Pass 2: 2 swaps  (3↔2, 3↔1)
  Pass 3: 1 swap   (2↔1)
  Total: 10 swaps
  
  SELECTION SORT SWAPS:
  Pass 0: 1 swap (5↔1)
  Pass 1: 1 swap (4↔2)
  Pass 2: 0 swaps (3 already in place)
  Pass 3: 0 swaps (already sorted)
  Total: 2 swaps
  
  ┌──────────────────────────────────────────────┐
  │  Selection Sort: 2 swaps vs 10 swaps!       │
  │  5× fewer data movements in this example    │
  └──────────────────────────────────────────────┘
```

---

## Adaptivity Comparison

```
  Nearly sorted: [1, 2, 3, 5, 4]
  
  BUBBLE SORT (optimized):
  Pass 0: 3 no-swaps, 1 swap → [1,2,3,4,5] → swapped=true
  Pass 1: all no-swaps → swapped=false → BREAK!
  Total: 7 comparisons → O(n)  ← ADAPTIVE!
  
  SELECTION SORT:
  Pass 0: scan 4 elements → min=1 → no swap
  Pass 1: scan 3 elements → min=2 → no swap
  Pass 2: scan 2 elements → min=3 → no swap
  Pass 3: scan 1 element  → min=4 → swap 5↔4
  Total: 10 comparisons → O(n²) ← NOT ADAPTIVE!
  
  Bubble Sort is 30% faster on this input!
```

---

## When Each Wins

```
  ┌──────────────────────────────────────────────────────────┐
  │  BUBBLE SORT WINS when:                                  │
  │  • Data is nearly sorted (adaptive O(n))                │
  │  • Stability is required                                 │
  │  • Need to detect if already sorted                      │
  ├──────────────────────────────────────────────────────────┤
  │  SELECTION SORT WINS when:                               │
  │  • Swaps/writes are expensive                           │
  │  • Memory writes cost more than reads                    │
  │  • Working with flash memory or EEPROM                   │
  │  • You want predictable O(n) swap count                  │
  └──────────────────────────────────────────────────────────┘
```

---

## Performance on Different Inputs

| Input Type | Bubble Sort | Selection Sort | Winner |
|-----------|-------------|----------------|--------|
| Already sorted | **O(n)** | O(n²) | Bubble |
| Nearly sorted | **O(n)** | O(n²) | Bubble |
| Random | O(n²) | **O(n²) fewer swaps** | Selection |
| Reverse sorted | O(n²) many swaps | **O(n²) few swaps** | Selection |
| Few unique values | O(n²) | O(n²) | Tie |

---

## The Bottom Line

```
  ┌──────────────────────────────────────────────────────────┐
  │  For most practical purposes, NEITHER is good enough.   │
  │                                                          │
  │  But if forced to choose between the two:               │
  │                                                          │
  │  • Use BUBBLE SORT for nearly sorted data               │
  │  • Use SELECTION SORT for minimizing writes             │
  │                                                          │
  │  For everything else, use INSERTION SORT, which is      │
  │  better than both in almost every scenario.             │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Factor | Bubble Sort | Selection Sort |
|--------|-------------|----------------|
| Swaps (worst) | O(n²) | O(n) ← winner |
| Best case | O(n) ← winner | O(n²) |
| Stability | Stable ← winner | Unstable |
| Adaptivity | Adaptive ← winner | Not adaptive |
| Predictability | Varies by input | Consistent behavior |
| Write efficiency | Poor | Excellent ← winner |

---

## Quick Revision Questions

1. **Which algorithm makes fewer swaps: Bubble or Selection?**
2. **Which algorithm is adaptive? What does that mean practically?**
3. **For reverse-sorted input, compare the swap counts of both.**
4. **Why is Bubble Sort stable but Selection Sort unstable?**
5. **In what scenario would you choose Selection Sort over Bubble Sort?**
6. **Which O(n²) sort is generally better than both? Why?**

---

[← Previous: Stability Consideration](04-stability-consideration.md) | [Next Unit: Insertion Sort →](../04-Insertion-Sort/01-algorithm-description.md)
