# Chapter 3: Ternary vs Binary Search — When to Use Which

[← Previous: Unimodal Functions](02-unimodal-functions.md) | [Next Unit: Exponential Search →](../09-Exponential-Search/01-concept.md)

---

## Overview

This chapter provides a definitive comparison and shows that binary search can often **replace** ternary search, even for unimodal problems.

---

## Head-to-Head Comparison

| Aspect | Binary Search | Ternary Search |
|--------|---------------|----------------|
| Divisions per step | 2 | 3 |
| Comparisons per step | 1-2 | 2 |
| Reduction per step | 1/2 eliminated | 1/3 eliminated |
| Works on sorted arrays | ✓ (optimal) | ✓ (slower) |
| Works on unimodal functions | With derivative trick | ✓ (native) |
| Total comparisons | ~log₂ n | ~2 log₃ n ≈ 1.26 log₂ n |
| Implementation | Simpler | More complex |

---

## Binary Search on Unimodal Functions (The Derivative Trick)

```
   Instead of ternary search, use binary search on the DERIVATIVE:
   
   For discrete arrays:
   - If arr[mid] < arr[mid + 1] → ascending → peak is right → lo = mid + 1
   - If arr[mid] > arr[mid + 1] → descending → peak is left → hi = mid
   
   This IS binary search! (Template 3)
```

```python
# Find peak element — Binary search version
def findPeak(arr):
    lo, hi = 0, len(arr) - 1
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] < arr[mid + 1]:
            lo = mid + 1       # go right
        else:
            hi = mid           # stay or go left
    return lo
```

```
   This requires only 1 comparison per step vs 2 for ternary!
   → Binary search on derivative is FASTER.
```

---

## When Ternary Search Is Actually Needed

```
   ┌────────────────────────────────────────────────────────────┐
   │                                                            │
   │  Use ternary search when:                                 │
   │                                                            │
   │  1. Continuous function (no array, only f(x))             │
   │     AND derivative is hard or expensive to compute        │
   │                                                            │
   │  2. Problem asks for exact peak of a CONVEX function      │
   │     where you cannot compute the derivative               │
   │                                                            │
   │  3. Competitive programming convention (some platforms    │
   │     have standard ternary search problems)                │
   │                                                            │
   │  In most other cases, binary search is preferred!         │
   │                                                            │
   └────────────────────────────────────────────────────────────┘
```

---

## Mathematical Proof: Binary > Ternary for Sorted Data

```
   Binary search: log₂ n comparisons
   Ternary search on sorted: 2 × log₃ n comparisons
   
   2 × log₃ n = 2 × (log₂ n / log₂ 3) = 2 × log₂ n / 1.585
              ≈ 1.262 × log₂ n
   
   Ratio: 1.262 × log₂ n / log₂ n = 1.262
   
   Ternary is 26.2% more comparisons than binary!
   
   For n = 10⁹:
   Binary:  log₂(10⁹) ≈ 30 comparisons
   Ternary: 2 × log₃(10⁹) ≈ 38 comparisons
```

---

## What About K-ary Search?

```
   As K increases:
   
   K=2:  1 comparison,  eliminate 50%  → efficiency = 0.50/1 = 0.500
   K=3:  2 comparisons, eliminate 67%  → efficiency = 0.67/2 = 0.333
   K=4:  3 comparisons, eliminate 75%  → efficiency = 0.75/3 = 0.250
   K=10: 9 comparisons, eliminate 90%  → efficiency = 0.90/9 = 0.100
   
   Binary search (K=2) has the best comparison efficiency!
   
   Mathematically:  minimize  (K-1) / log₂(K)
   The minimum is at K = 2.
```

---

## Practical Example: Minimum Distance

> Given a function `f(x) = |x - 3| + |x - 7|`, find the x that minimizes it on [0, 10].

**Ternary search (direct):**
```python
def f(x):
    return abs(x - 3) + abs(x - 7)

lo, hi = 0.0, 10.0
for _ in range(200):
    m1 = lo + (hi - lo) / 3
    m2 = hi - (hi - lo) / 3
    if f(m1) > f(m2):
        lo = m1
    else:
        hi = m2
print((lo + hi) / 2)   # ≈ 5.0 (any x in [3,7] is optimal)
```

**Binary search (on derivative):**
```python
# f'(x) = sign(x-3) + sign(x-7) (derivative of sum of absolute values)
# f'(x) < 0 when x < 3, f'(x) = 0 when 3 ≤ x ≤ 7, f'(x) > 0 when x > 7
# Binary search for transition point

lo, hi = 0.0, 10.0
for _ in range(200):
    mid = (lo + hi) / 2
    if f(mid) > f(mid + 1e-9):   # is slope negative? (decreasing)
        lo = mid
    else:
        hi = mid
print((lo + hi) / 2)   # ≈ 3.0 (left edge of flat region)
```

---

## Decision Guide

```
   DO I NEED TERNARY SEARCH?
   
   Q1: Is the data in a sorted array?
       YES → Use Binary Search (always better)
   
   Q2: Am I finding peak/valley of a discrete array?
       YES → Use Binary Search with derivative trick
   
   Q3: Am I optimizing a continuous unimodal function?
       Q3a: Can I compute or approximate the derivative?
            YES → Binary search on derivative
            NO  → Ternary Search ✓
   
   Q4: Competitive programming with standard ternary problem?
       YES → Ternary Search ✓ (it's expected)
```

---

## Summary

```
   ╔══════════════════════════════════════════════════════════════╗
   ║                                                              ║
   ║  Binary search:  Almost always preferred                    ║
   ║  Ternary search: Niche use for continuous unimodal          ║
   ║                  functions without accessible derivatives   ║
   ║                                                              ║
   ║  Key takeaway: Ternary search EXISTS and is VALID,         ║
   ║  but binary search can usually do the same job faster.      ║
   ║                                                              ║
   ╚══════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Questions

1. **Why is binary search 26% faster than ternary on sorted arrays?**
2. **How can binary search find the peak of a unimodal function?**
3. **When is ternary search genuinely necessary?**
4. **What happens as we go from binary (K=2) to higher K-ary search?**
5. **For the function f(x) = (x-5)², what would ternary search find?**

---

[← Previous: Unimodal Functions](02-unimodal-functions.md) | [Next Unit: Exponential Search →](../09-Exponential-Search/01-concept.md)
