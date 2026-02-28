# Chapter 5: Randomized Quick Sort

## Overview

**Randomized Quick Sort** eliminates the dependency on input order by choosing a **random pivot**. This guarantees an **expected O(n log n)** running time for any input, making it one of the most practical sorting algorithms.

---

## The Algorithm

```
function randomizedQuickSort(arr, lo, hi):
    if lo < hi:
        // Random pivot selection
        randIdx = random(lo, hi)
        swap(arr[randIdx], arr[hi])
        
        p = partition(arr, lo, hi)      // standard Lomuto or Hoare
        randomizedQuickSort(arr, lo, p - 1)
        randomizedQuickSort(arr, p + 1, hi)
```

The **only change** from standard Quick Sort: one random swap before partitioning.

---

## Why Randomization Works

```
┌─────────────────────────────────────────────────────────┐
│  WITHOUT randomization:                                  │
│    Worst case INPUT exists → adversary can force O(n²)   │
│                                                          │
│  WITH randomization:                                     │
│    No worst case INPUT — only worst case RANDOM CHOICES  │
│    P(bad random choices) exponentially small             │
│                                                          │
│  The analysis shifts from:                               │
│    "What's the worst input?" (deterministic)             │
│  to:                                                     │
│    "What's the expected time?" (randomized)              │
│                                                          │
│  Expected time = O(n log n) for EVERY input              │
└─────────────────────────────────────────────────────────┘
```

---

## Expected Number of Comparisons

```
Theorem: Randomized Quick Sort makes at most 
         2n ln(n) ≈ 1.39 n log₂(n) comparisons in expectation.

Proof Sketch (indicator random variables):

Let X = total comparisons.
Let X_{ij} = indicator: 1 if element i and j are compared, 0 otherwise.

X = Σ X_{ij} for all pairs i < j

E[X] = Σ P(i and j are compared)

Key insight: i and j are compared ⟺ one of them is chosen 
as pivot before any element between them (in sorted order).

P(i compared with j) = 2/(j - i + 1)

E[X] = Σ_{i=1}^{n} Σ_{j=i+1}^{n} 2/(j-i+1)
     = 2n × H_n - 2n
     ≈ 2n ln(n)
     ≈ 1.39 n log₂(n)

where H_n is the nth harmonic number.
```

---

## Visualizing the Probability

```
Sorted elements: z₁ < z₂ < z₃ < z₄ < z₅ < z₆ < z₇

P(z₂ compared with z₅)?

Elements between z₂ and z₅ (inclusive): {z₂, z₃, z₄, z₅}

z₂ and z₅ are compared ⟺ z₂ or z₅ is the FIRST among
{z₂, z₃, z₄, z₅} to be chosen as a pivot.

P = 2/4 = 1/2

If z₃ or z₄ is chosen as pivot first, z₂ and z₅ end up
in different partitions and are NEVER compared.
```

---

## Expected vs Worst Case

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│  Expected time: Θ(n log n)                          │
│    → This holds for EVERY input                      │
│    → Expectation is over random coin flips           │
│                                                      │
│  Worst case (bad luck): O(n²)                       │
│    → All random pivots happen to be extreme          │
│    → P(worst case) ≈ 2ⁿ/n! → negligibly small       │
│                                                      │
│  Variance: quite low                                 │
│    → Var(comparisons) = O(n²)                       │
│    → Std dev = O(n)                                  │
│    → Time is concentrated near the expected value    │
│                                                      │
│  High probability bound:                             │
│    P(time > c × n log n) ≤ 1/n² for some constant c│
└──────────────────────────────────────────────────────┘
```

---

## Randomized vs Deterministic

```
DETERMINISTIC Quick Sort:
  Input: [1, 2, 3, ..., n]
  Pivot = first element
  ALWAYS takes O(n²) — no randomness, no luck involved

RANDOMIZED Quick Sort:
  Input: [1, 2, 3, ..., n]  (same adversarial input)
  EXPECTED O(n log n) — bad luck is possible but unlikely

Same input, vastly different guarantees!

Key difference:
  Deterministic: adversary chooses input AFTER seeing algorithm
  Randomized: adversary chooses input BEFORE seeing random choices
```

---

## Alternative: Shuffle First

Instead of random pivot, shuffle the array first:

```
function shuffleSort(arr):
    // Fisher-Yates shuffle: O(n)
    for i = n-1 downto 1:
        j = random(0, i)
        swap(arr[i], arr[j])
    
    // Now use standard (deterministic) Quick Sort
    quickSort(arr, 0, n-1)
```

```
✓ Same expected O(n log n) guarantee
✓ Uses standard quickSort code (no modification needed)
✗ O(n) extra time for shuffle
✗ Modifies input before sorting (same as quicksort though)

Equivalent in analysis to randomized pivot selection.
```

---

## Python Implementation

```python
import random

def randomized_quicksort(arr, lo, hi):
    if lo < hi:
        # Random pivot
        rand_idx = random.randint(lo, hi)
        arr[rand_idx], arr[hi] = arr[hi], arr[rand_idx]
        
        # Lomuto partition
        pivot = arr[hi]
        i = lo - 1
        for j in range(lo, hi):
            if arr[j] <= pivot:
                i += 1
                arr[i], arr[j] = arr[j], arr[i]
        arr[i + 1], arr[hi] = arr[hi], arr[i + 1]
        p = i + 1
        
        randomized_quicksort(arr, lo, p - 1)
        randomized_quicksort(arr, p + 1, hi)

# Usage
arr = [3, 6, 8, 10, 1, 2, 1]
randomized_quicksort(arr, 0, len(arr) - 1)
print(arr)  # [1, 1, 2, 3, 6, 8, 10]
```

---

## Las Vegas vs Monte Carlo

```
Randomized Quick Sort is a LAS VEGAS algorithm:

Las Vegas:
  → ALWAYS gives correct result
  → Running time is random (expected O(n log n))
  → Example: Randomized Quick Sort

Monte Carlo:
  → Gives correct result with high PROBABILITY
  → Running time is fixed
  → Example: Randomized primality test

Quick Sort: output is always correct (sorted array),
but time varies with random choices.
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Modification | Random pivot before partition |
| Expected time | O(n log n) for ANY input |
| Expected comparisons | ~1.39 n log₂(n) |
| Worst case | O(n²) but probability ≈ 2ⁿ/n! |
| Variance | Low — concentrated near expected value |
| Algorithm type | Las Vegas (always correct) |
| Alternative | Shuffle first, then deterministic sort |

---

## Quick Revision Questions

1. **What single change converts deterministic Quick Sort to randomized?**
2. **Why is expected O(n log n) a stronger guarantee than average-case O(n log n)?**
3. **What is the expected number of comparisons and how is it derived?**
4. **What type of randomized algorithm is Quick Sort — Las Vegas or Monte Carlo?**
5. **How does shuffling the array first achieve the same guarantee?**

---

[← Previous: Worst Case Analysis](04-worst-case-analysis.md) | [Next: Quick Sort vs Other Sorts →](06-quicksort-vs-others.md)
