# Unit 9: Radix Sort — Complexity Analysis

[← Previous: Implementation](03-implementation.md) | [Next: String Sorting →](05-string-sorting.md)

---

## Overview

Radix Sort's complexity depends on three parameters: **n** (number of elements), **d** (number of digits/passes), and **k** (base/radix). Understanding the interplay between these is key to knowing when Radix Sort outperforms comparison sorts.

---

## Time Complexity

```
  RADIX-SORT does d passes of Counting Sort.
  Each Counting Sort pass: O(n + k)
  
  Total: O(d × (n + k))
  
  ┌──────────────────────────────────────────────────────────┐
  │  Variables:                                              │
  │  n = number of elements                                 │
  │  d = number of digits (passes)                          │
  │  k = base (radix)                                       │
  │                                                          │
  │  Base 10:  k = 10                                       │
  │  Base 2:   k = 2                                        │
  │  Base 256: k = 256                                      │
  │  Base n:   k = n                                        │
  └──────────────────────────────────────────────────────────┘
```

---

## When Is Radix Sort O(n)?

```
  d × (n + k) = O(n) when:
  
  1. d is constant (fixed number of digits)
  2. k = O(n) or k is constant
  
  ┌──────────────────────────────────────────────────────────┐
  │  Example 1: Sort n phone numbers (10 digits, base 10)  │
  │  d = 10, k = 10                                        │
  │  Time = 10 × (n + 10) = O(n) ✓                        │
  │                                                          │
  │  Example 2: Sort n values in range [0, n²]             │
  │  Using base n: d = 2 digits, k = n                     │
  │  Time = 2 × (n + n) = O(n) ✓                          │
  │                                                          │
  │  Example 3: Sort n values in range [0, n^c] for const c│
  │  Using base n: d = c digits, k = n                     │
  │  Time = c × (n + n) = O(cn) = O(n) ✓                  │
  │                                                          │
  │  When d and k are bounded by functions of n, radix sort │
  │  achieves LINEAR time!                                   │
  └──────────────────────────────────────────────────────────┘
```

---

## Radix Sort vs Quick Sort: Detailed Analysis

```
  Quick Sort: O(n log n) average
  Radix Sort: O(d(n + k))
  
  For b-bit integers using base r = 2^p:
  d = ⌈b/p⌉ passes, k = 2^p = r
  
  Radix time: ⌈b/p⌉ × (n + 2^p)
  
  Choose p = log₂ n (base = n):
  d = ⌈b/log n⌉
  k = n
  Time = ⌈b/log n⌉ × (n + n) = O(bn/log n)
  
  Compare with Quick Sort ≈ c₁ × n log n:
  
  Radix wins when: bn/log n < c₁ × n log n
                   b < c₁ × (log n)²
  
  For 32-bit ints (b=32), n=10⁶:
  32 < c₁ × 400   → YES (for reasonable c₁)
  Radix Sort wins!
  
  For 64-bit ints (b=64), n=1000:
  64 < c₁ × 100   → Maybe, depends on constant
  Could go either way.
  
  ┌──────────────────────────────────────────────────────┐
  │  Rule of thumb:                                     │
  │  Radix beats Quick Sort when n is large AND         │
  │  the number of bits b is not too large relative     │
  │  to log²(n).                                        │
  └──────────────────────────────────────────────────────┘
```

---

## Space Complexity

```
  ┌─────────────────────────────────────────────────────────┐
  │  Count array:  O(k)   (size = base/radix)              │
  │  Output array: O(n)   (for stable counting sort)       │
  │  ────────────────────                                   │
  │  Total: O(n + k)                                       │
  │                                                         │
  │  For base 10:  O(n + 10)  = O(n)                       │
  │  For base 256: O(n + 256) = O(n)                       │
  │                                                         │
  │  NOT in-place! Needs the output array.                  │
  └─────────────────────────────────────────────────────────┘
  
  MSD Radix Sort additionally uses:
  O(d) recursion stack = O(max_digits) call stack
```

---

## Base (Radix) Selection Trade-off

```
  ┌───────────┬──────────┬──────────────┬───────────────────┐
  │  Base     │ Passes   │ Count array  │ Total time        │
  ├───────────┼──────────┼──────────────┼───────────────────┤
  │  2        │ 32       │ 2            │ 32 × (n+2) ≈ 32n │
  │  10       │ 10       │ 10           │ 10 × (n+10) ≈ 10n│
  │  16       │ 8        │ 16           │ 8 × (n+16) ≈ 8n  │
  │  256      │ 4        │ 256          │ 4 × (n+256) ≈ 4n │
  │  65536    │ 2        │ 65536        │ 2 × (n+65536)    │
  │  n        │ ⌈32/lgn⌉│ n            │ (32/lgn)(n+n)    │
  └───────────┴──────────┴──────────────┴───────────────────┘
  
  Increasing base:
  ✓ Fewer passes → faster
  ✗ Larger count array → more memory, cache misses
  
  Optimal base ≈ max(256, n) for most practical cases.
  Base 256 is the common practical choice.
```

---

## All Cases Are the Same

```
  ┌─────────────┬────────────────┐
  │  Case       │ Time           │
  ├─────────────┼────────────────┤
  │  Best       │ O(d(n + k))   │
  │  Average    │ O(d(n + k))   │
  │  Worst      │ O(d(n + k))   │
  └─────────────┴────────────────┘
  
  Radix Sort is NOT adaptive:
  - Sorted input takes same time as random input
  - Always processes all d digits
  - No early termination (LSD version)
  
  MSD CAN terminate early if groups become singletons,
  but worst case is still O(d(n + k)).
```

---

## Comparison Table: Non-Comparison Sorts

```
  ┌──────────────────┬───────────────┬──────────┬─────────┐
  │  Algorithm       │ Time          │ Space    │ Stable  │
  ├──────────────────┼───────────────┼──────────┼─────────┤
  │  Counting Sort   │ O(n + k)      │ O(n + k) │ Yes     │
  │  Radix Sort      │ O(d(n + k))   │ O(n + k) │ Yes     │
  │  Bucket Sort     │ O(n + k) avg  │ O(n + k) │ Yes     │
  └──────────────────┴───────────────┴──────────┴─────────┘
  
  k means different things:
  - Counting: k = range of values
  - Radix: k = base (radix), d = digits
  - Bucket: k = number of buckets
```

---

## Summary Table

| Metric | Value |
|--------|-------|
| Time | O(d × (n + k)) |
| Space | O(n + k) |
| Best/Average/Worst | All same |
| Stable | Yes |
| In-place | No |
| Adaptive | No |
| Linear when | d constant, k = O(n) |
| Beats Quick Sort when | b < c × (log n)² |

---

## Quick Revision Questions

1. **What are d, n, and k in Radix Sort's time complexity?**
2. **Under what conditions is Radix Sort O(n)?**
3. **Why does increasing the base reduce passes but increase space?**
4. **When does Quick Sort beat Radix Sort despite O(n log n) vs O(n)?**
5. **How much extra space does Radix Sort need?**

---

[← Previous: Implementation](03-implementation.md) | [Next: String Sorting →](05-string-sorting.md)
