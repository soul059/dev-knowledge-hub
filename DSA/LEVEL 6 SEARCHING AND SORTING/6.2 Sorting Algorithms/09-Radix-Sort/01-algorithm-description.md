# Unit 9: Radix Sort — Algorithm Description

[← Previous: Unit 8 - Counting Sort Variations](../08-Counting-Sort/06-variations.md) | [Next: LSD vs MSD →](02-lsd-vs-msd.md)

---

## Overview

Radix Sort sorts numbers **digit by digit**, from least significant digit (LSD) to most significant digit (MSD) — or vice versa. It achieves **O(d × (n + k))** time, where d is the number of digits and k is the base (usually 10). When d is constant, this becomes **O(n)** — linear time!

---

## The Key Idea

```
  Instead of comparing whole numbers, sort them
  ONE DIGIT AT A TIME using a stable sort (Counting Sort).
  
  ┌──────────────────────────────────────────────────────┐
  │  Numbers: 329, 457, 657, 839, 436, 720, 355        │
  │                                                      │
  │  Step 1: Sort by ONES digit  (least significant)    │
  │  Step 2: Sort by TENS digit                         │
  │  Step 3: Sort by HUNDREDS digit (most significant)  │
  │                                                      │
  │  After 3 passes → fully sorted!                     │
  └──────────────────────────────────────────────────────┘
  
  WHY does this work?
  Because each digit-sort is STABLE, previous orderings
  are preserved for elements with equal current digits.
```

---

## Step-by-Step Trace (LSD Radix Sort)

```
  Original: [329, 457, 657, 839, 436, 720, 355]
  
  ╔══════════════════════════════════════════════════════╗
  ║  PASS 1: Sort by ONES digit (rightmost)            ║
  ╠══════════════════════════════════════════════════════╣
  ║  329 → 9    457 → 7    657 → 7    839 → 9         ║
  ║  436 → 6    720 → 0    355 → 5                     ║
  ║                                                      ║
  ║  Bucket 0: 720                                      ║
  ║  Bucket 5: 355                                      ║
  ║  Bucket 6: 436                                      ║
  ║  Bucket 7: 457, 657                                 ║
  ║  Bucket 9: 329, 839                                 ║
  ║                                                      ║
  ║  After: [720, 355, 436, 457, 657, 329, 839]        ║
  ╠══════════════════════════════════════════════════════╣
  ║  PASS 2: Sort by TENS digit                        ║
  ╠══════════════════════════════════════════════════════╣
  ║  720 → 2    355 → 5    436 → 3    457 → 5         ║
  ║  657 → 5    329 → 2    839 → 3                     ║
  ║                                                      ║
  ║  Bucket 2: 720, 329       (720 before 329: stable!)║
  ║  Bucket 3: 436, 839                                 ║
  ║  Bucket 5: 355, 457, 657                            ║
  ║                                                      ║
  ║  After: [720, 329, 436, 839, 355, 457, 657]        ║
  ╠══════════════════════════════════════════════════════╣
  ║  PASS 3: Sort by HUNDREDS digit                    ║
  ╠══════════════════════════════════════════════════════╣
  ║  720 → 7    329 → 3    436 → 4    839 → 8         ║
  ║  355 → 3    457 → 4    657 → 6                     ║
  ║                                                      ║
  ║  Bucket 3: 329, 355       (329 before 355: stable!)║
  ║  Bucket 4: 436, 457                                 ║
  ║  Bucket 6: 657                                      ║
  ║  Bucket 7: 720                                      ║
  ║  Bucket 8: 839                                      ║
  ║                                                      ║
  ║  After: [329, 355, 436, 457, 657, 720, 839]  ✓     ║
  ╚══════════════════════════════════════════════════════╝
  
  SORTED! Each pass maintains previous order via stability!
```

---

## Why Stability Is Essential

```
  After Pass 1 (sorted by ones):
  [720, 355, 436, 457, 657, 329, 839]
  
  The order 457 before 657 was established because
  they were in that order originally (4 before 6 in hundreds).
  Wait — no, it's because both have ones=7 and stability
  preserved their original order.
  
  After Pass 2 (sorted by tens):
  355, 457, 657 all have tens=5.
  Stability preserves their Pass-1 order (5 before 7 before 7).
  Their ones-digit ordering is maintained!
  
  After Pass 3 (sorted by hundreds):
  329, 355 both have hundreds=3.
  Stability preserves their Pass-2 order.
  329 (tens=2) comes before 355 (tens=5) ← CORRECT!
  
  ┌────────────────────────────────────────────────────────┐
  │  If ANY pass used an UNSTABLE sort, the final result  │
  │  would be WRONG. Stability is NON-NEGOTIABLE.         │
  └────────────────────────────────────────────────────────┘
```

---

## Radix Sort vs Comparison Sorts

```
  ┌──────────────────────────────────────────────────────────┐
  │  Comparison sorts are limited to Ω(n log n).           │
  │                                                          │
  │  Radix Sort: O(d × (n + k))                            │
  │                                                          │
  │  d = number of digits                                   │
  │  k = base (radix), usually 10                           │
  │                                                          │
  │  For 32-bit integers (d=10 decimal digits, k=10):       │
  │  O(10 × (n + 10)) = O(10n) = O(n)                      │
  │                                                          │
  │  This is FASTER than O(n log n) for large n!            │
  │                                                          │
  │  For n = 1,000,000:                                     │
  │  Quick Sort: ~20,000,000 operations (n × 20)            │
  │  Radix Sort: ~10,000,000 operations (n × 10 digits)    │
  │  Radix can be ~2x faster!                               │
  └──────────────────────────────────────────────────────────┘
```

---

## Pseudocode

```
  RADIX-SORT(A, d):
      // A = array, d = max number of digits
      for digit = 1 to d:
          // Use a STABLE sort on digit position
          COUNTING-SORT-BY-DIGIT(A, digit)
  
  COUNTING-SORT-BY-DIGIT(A, digitPlace):
      k = 10  // digits 0-9
      count = new array[k], initialized to 0
      output = new array[n]
      
      for i = 0 to n-1:
          d = GET-DIGIT(A[i], digitPlace)
          count[d]++
      
      for i = 1 to k-1:
          count[i] += count[i-1]
      
      for i = n-1 downto 0:
          d = GET-DIGIT(A[i], digitPlace)
          output[count[d] - 1] = A[i]
          count[d]--
      
      copy output to A
  
  GET-DIGIT(number, digitPlace):
      return (number / 10^(digitPlace-1)) % 10
```

---

## Properties at a Glance

| Property | Value |
|----------|-------|
| Type | Non-comparison, distribution sort |
| Time | O(d × (n + k)) |
| Space | O(n + k) |
| Stable | Yes (uses stable subroutine) |
| In-place | No |
| Best for | Fixed-length integers, strings |

---

## Quick Revision Questions

1. **What does "radix" mean in Radix Sort?**
2. **Why must the subroutine sort be stable?**
3. **What determines the number of passes (d)?**
4. **How does Radix Sort bypass the O(n log n) lower bound?**
5. **Why is Counting Sort the ideal subroutine for Radix Sort?**

---

[← Previous: Unit 8 - Counting Sort Variations](../08-Counting-Sort/06-variations.md) | [Next: LSD vs MSD →](02-lsd-vs-msd.md)
