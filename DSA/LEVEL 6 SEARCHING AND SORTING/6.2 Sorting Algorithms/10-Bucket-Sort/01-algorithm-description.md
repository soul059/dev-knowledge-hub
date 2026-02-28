# Unit 10: Bucket Sort — Algorithm Description

[← Previous: Unit 9 - Radix Sort Applications](../09-Radix-Sort/06-applications.md) | [Next: Implementation →](02-implementation.md)

---

## Overview

Bucket Sort distributes elements into "buckets" based on their value range, sorts each bucket individually, then concatenates. When input is **uniformly distributed**, it achieves **O(n)** average-case time. It's a **distribution sort** — not comparison-based at the top level.

---

## The Key Idea

```
  Divide the value range into equal intervals (buckets).
  Elements fall into buckets based on their value.
  Sort each bucket. Concatenate.
  
  Array: [0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12, 0.23, 0.68]
  
  10 buckets for range [0, 1):
  
  Bucket 0 [0.0-0.1): 
  Bucket 1 [0.1-0.2): 0.17, 0.12
  Bucket 2 [0.2-0.3): 0.26, 0.21, 0.23
  Bucket 3 [0.3-0.4): 0.39
  Bucket 4 [0.4-0.5): 
  Bucket 5 [0.5-0.6): 
  Bucket 6 [0.6-0.7): 0.68
  Bucket 7 [0.7-0.8): 0.78, 0.72
  Bucket 8 [0.8-0.9): 
  Bucket 9 [0.9-1.0): 0.94
  
  Sort each bucket (e.g., insertion sort):
  Bucket 1: [0.12, 0.17]
  Bucket 2: [0.21, 0.23, 0.26]
  Bucket 7: [0.72, 0.78]
  
  Concatenate all buckets:
  [0.12, 0.17, 0.21, 0.23, 0.26, 0.39, 0.68, 0.72, 0.78, 0.94]
  
  SORTED! ✓
```

---

## How It Works Visually

```
  Input: [29, 25, 3, 49, 9, 37, 21, 43]
  Range: 0-49, 5 buckets (each covers 10 values)
  
  ┌─────────┬────────────────────┐
  │ Bucket  │ Elements           │
  ├─────────┼────────────────────┤
  │  0-9    │ 3, 9               │
  │ 10-19   │                    │
  │ 20-29   │ 29, 25, 21         │
  │ 30-39   │ 37                 │
  │ 40-49   │ 49, 43             │
  └─────────┴────────────────────┘
         │
         ▼ Sort each bucket
  ┌─────────┬────────────────────┐
  │  0-9    │ 3, 9               │
  │ 20-29   │ 21, 25, 29         │
  │ 30-39   │ 37                 │
  │ 40-49   │ 43, 49             │
  └─────────┴────────────────────┘
         │
         ▼ Concatenate
  [3, 9, 21, 25, 29, 37, 43, 49]  ✓
```

---

## Algorithm Steps

```
  BUCKET-SORT(A, n):
  
  1. Find min and max of A                    O(n)
  2. Create k empty buckets                   O(k)
  3. For each element:                         O(n)
       bucket_index = (A[i] - min) × k / (max - min + 1)
       Place A[i] in bucket[bucket_index]
  4. Sort each bucket (insertion sort)         O(n) expected
  5. Concatenate all buckets into A            O(n + k)
  
  Total: O(n + k) expected (uniform distribution)
```

---

## Why Uniform Distribution Matters

```
  UNIFORM distribution: elements spread evenly across buckets
  
  ┌─────────────────────────────────────────────────────┐
  │  n elements in k=n buckets:                        │
  │  Expected elements per bucket = n/n = 1            │
  │  Sorting 1 element = O(1)                          │
  │  Total sort: n × O(1) = O(n)  ✓                   │
  └─────────────────────────────────────────────────────┘
  
  SKEWED distribution: most elements in one bucket
  
  ┌─────────────────────────────────────────────────────┐
  │  Worst case: all elements in ONE bucket            │
  │  Sorting that bucket: O(n²) (insertion sort)       │
  │  or O(n log n) if using merge sort per bucket      │
  │                                                     │
  │  Bucket Sort degrades to its inner sort!           │
  └─────────────────────────────────────────────────────┘
  
  Example of skewed data:
  [0.01, 0.02, 0.03, 0.04, 0.05, 0.06, 0.07, 0.08]
  All elements in bucket 0! → defeats the purpose.
```

---

## Bucket Index Calculation

```
  For values in range [min, max]:
  
  bucket_index = ⌊(value - min) × k / (max - min + 1)⌋
  
  Example: values in [0, 100), k = 10 buckets
  
  value = 37:  ⌊37 × 10 / 100⌋ = ⌊3.7⌋ = 3
  value = 72:  ⌊72 × 10 / 100⌋ = ⌊7.2⌋ = 7
  value = 99:  ⌊99 × 10 / 100⌋ = ⌊9.9⌋ = 9
  value = 0:   ⌊0 × 10 / 100⌋  = ⌊0⌋   = 0
  
  For floating-point in [0, 1):
  bucket_index = ⌊value × k⌋
  
  value = 0.78: ⌊0.78 × 10⌋ = ⌊7.8⌋ = 7
  value = 0.12: ⌊0.12 × 10⌋ = ⌊1.2⌋ = 1
```

---

## Pseudocode

```
  BUCKET-SORT(A, n, k):
      // A = input, n = size, k = number of buckets
      
      min = MIN(A)
      max = MAX(A)
      range = max - min + 1
      
      B = array of k empty lists
      
      // Distribute into buckets
      for i = 0 to n-1:
          idx = (A[i] - min) * k / range
          B[idx].append(A[i])
      
      // Sort each bucket
      for j = 0 to k-1:
          INSERTION-SORT(B[j])
      
      // Concatenate
      idx = 0
      for j = 0 to k-1:
          for each element in B[j]:
              A[idx++] = element
```

---

## Properties at a Glance

| Property | Value |
|----------|-------|
| Type | Distribution sort (non-comparison at top level) |
| Time (average, uniform) | O(n + k) |
| Time (worst) | O(n²) or O(n log n) depending on inner sort |
| Space | O(n + k) |
| Stable | Yes (if inner sort is stable) |
| In-place | No |
| Best for | Uniformly distributed data |

---

## Quick Revision Questions

1. **What assumption does Bucket Sort make about input data?**
2. **How is the bucket index calculated?**
3. **What happens when all elements fall into one bucket?**
4. **Why is insertion sort commonly used within buckets?**
5. **How many buckets should you typically create?**

---

[← Previous: Unit 9 - Radix Sort Applications](../09-Radix-Sort/06-applications.md) | [Next: Implementation →](02-implementation.md)
