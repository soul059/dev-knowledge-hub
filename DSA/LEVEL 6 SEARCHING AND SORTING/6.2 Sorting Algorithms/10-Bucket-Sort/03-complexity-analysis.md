# Unit 10: Bucket Sort — Complexity Analysis

[← Previous: Implementation](02-implementation.md) | [Next: Uniform Distribution →](04-uniform-distribution.md)

---

## Overview

Bucket Sort achieves **average-case O(n)** time — but only under the right conditions. This chapter breaks down exactly when, why, and how.

---

## Time Complexity Summary

```
  ┌────────────────────────────────────────────────────┐
  │              Bucket Sort Complexity                │
  ├────────────────┬───────────────────────────────────┤
  │  Best Case     │  O(n + k)                        │
  │  Average Case  │  O(n + n²/k + k)  ≈ O(n)        │
  │  Worst Case    │  O(n²)                           │
  ├────────────────┼───────────────────────────────────┤
  │  Space         │  O(n + k)                        │
  │  Stable?       │  Yes (if inner sort is stable)   │
  │  In-place?     │  No                              │
  └────────────────┴───────────────────────────────────┘
    where k = number of buckets
```

---

## Average Case: O(n) Derivation

### Setup

- n elements uniformly distributed in [0, 1)
- k = n buckets
- Each element maps to bucket `⌊value × n⌋`

### Expected Bucket Size

```
  With n elements and n buckets, uniform distribution:
  
  Expected elements per bucket = n / n = 1
  
  P(element goes to bucket i) = 1/n   (for each element)
  
  Let nᵢ = number of elements in bucket i
  E[nᵢ] = 1
```

### Total Work

```
  Total time = O(n)          distribute elements
             + Σᵢ O(nᵢ²)    sort each bucket (insertion sort)
             + O(k)          concatenate
  
  Expected total sorting work:
  E[Σ nᵢ²] = ?
  
  For uniform distribution with n elements, n buckets:
  E[nᵢ²] = E[nᵢ]² + Var(nᵢ)
          = 1 + (1 - 1/n)
          = 2 - 1/n
  
  E[Σ nᵢ²] = n × (2 - 1/n) = 2n - 1 = O(n)
  
  Total expected time = O(n) + O(n) + O(n) = O(n) ✓
```

### Formal Proof Using Indicator Variables

```
  Let Xᵢⱼ = 1 if element j falls in bucket i, else 0
  
  nᵢ = Σⱼ Xᵢⱼ
  
  E[nᵢ²] = E[(Σⱼ Xᵢⱼ)²]
          = E[Σⱼ Xᵢⱼ² + Σⱼ Σₖ≠ⱼ Xᵢⱼ Xᵢₖ]
          = Σⱼ E[Xᵢⱼ²] + Σⱼ Σₖ≠ⱼ E[Xᵢⱼ] E[Xᵢₖ]
          = n × (1/n) + n(n-1) × (1/n²)
          = 1 + (n-1)/n
          = 2 - 1/n
  
  Sum over all n buckets:
  E[Σ nᵢ²] = n(2 - 1/n) = 2n - 1 = Θ(n)
```

---

## Best Case: O(n + k)

```
  When every bucket gets exactly 0 or 1 element:
  
  Sorting 0-1 elements = O(1) per bucket
  Total = O(n) distribute + O(k) × O(1) sort + O(k) concat
        = O(n + k)
  
  With k = n:  O(n)
```

---

## Worst Case: O(n²)

```
  When ALL elements fall into ONE bucket:
  
  ┌─────────────────────────────────────────────┐
  │  Bucket 0: [all n elements]                │
  │  Bucket 1: empty                           │
  │  Bucket 2: empty                           │
  │  ...                                        │
  │  Bucket k-1: empty                         │
  └─────────────────────────────────────────────┘
  
  Sorting one bucket of n elements with insertion sort:
  O(n²)
  
  This happens when:
  • Data is heavily clustered (not uniform)
  • All values fall in the same small range
  • Hash function maps poorly
```

---

## Space Complexity: O(n + k)

```
  Space Breakdown:
  ┌───────────────────────────────────────┐
  │  k buckets (list structures)  → O(k) │
  │  n elements distributed      → O(n)  │
  │  Total                        → O(n+k)│
  └───────────────────────────────────────┘
  
  With k = n:  O(n) additional space
  
  Note: This is NOT in-place.
  The extra space is unavoidable in standard bucket sort.
```

---

## Comparison of k (# Buckets) Choice

```
  k = number of buckets
  
  ┌──────────┬─────────────┬─────────────────────────────────┐
  │  k       │ Time        │ Notes                           │
  ├──────────┼─────────────┼─────────────────────────────────┤
  │  k = 1   │ O(n²)       │ Degenerates to insertion sort  │
  │  k = √n  │ O(n)        │ Balanced buckets, fewer lists  │
  │  k = n   │ O(n)        │ Classic choice, more memory    │
  │  k = n²  │ O(n + n²)   │ Waste memory, no benefit      │
  └──────────┴─────────────┴─────────────────────────────────┘
  
  Practical sweet spot: k ≈ n  or  k ≈ √n
```

---

## Why It's Not Always O(n)

```
  ┌─────────────────────────────────────────────────────────────┐
  │                 CONDITIONS FOR O(n)                        │
  ├─────────────────────────────────────────────────────────────┤
  │  1. Input is uniformly distributed                         │
  │  2. Number of buckets ≈ n                                 │
  │  3. Bucket index function distributes elements evenly     │
  │                                                            │
  │  If ANY condition fails → performance degrades:           │
  │                                                            │
  │  • Skewed data + naive bucketing → few overloaded buckets │
  │  • Too few buckets → large inner sorts                    │
  │  • Poor hash function → clustering                        │
  │                                                            │
  │  Worst case is ALWAYS O(n²) regardless of bucket count    │
  └─────────────────────────────────────────────────────────────┘
```

---

## Comparison with Other Linear-Time Sorts

| Aspect | Counting Sort | Radix Sort | Bucket Sort |
|--------|--------------|------------|-------------|
| Best case | O(n + k) | O(d(n+k)) | O(n + k) |
| Worst case | O(n + k) | O(d(n+k)) | O(n²) |
| Worst-case guarantee | Yes | Yes | **No** |
| Depends on distribution | No | No | **Yes** |
| Space | O(n + k) | O(n + k) | O(n + k) |

> **Key insight**: Bucket Sort is the only "linear-time" sort that can degrade to O(n²). Its average-case guarantee depends on distribution assumptions.

---

## Summary Table

| Metric | Value | Condition |
|--------|-------|-----------|
| Average time | O(n) | Uniform distribution, k = n |
| Worst time | O(n²) | All elements in one bucket |
| Best time | O(n + k) | ≤ 1 element per bucket |
| Space | O(n + k) | Always |
| Stable | Yes | If inner sort is stable |
| Key requirement | Uniform distribution | For O(n) guarantee |

---

## Quick Revision Questions

1. **Derive the expected value of Σnᵢ² when n elements are uniformly distributed into n buckets.**
2. **Why does bucket sort have O(n²) worst case while counting sort doesn't?**
3. **What happens to performance if you use k = √n buckets instead of k = n?**
4. **Can bucket sort ever be faster than O(n)? Why or why not?**
5. **Why does the choice of inner sorting algorithm affect stability but not average-case time?**

---

[← Previous: Implementation](02-implementation.md) | [Next: Uniform Distribution →](04-uniform-distribution.md)
