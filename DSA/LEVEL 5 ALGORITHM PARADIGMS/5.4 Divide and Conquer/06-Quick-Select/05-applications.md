# Chapter 5: Applications of Selection Algorithms

## Overview

The ability to find the k-th smallest element in linear time unlocks efficient solutions to many problems. This chapter covers practical applications of Quick Select and selection techniques.

---

## 1. Finding the Median

```
The median is the most important order statistic.

Odd n:  median = (n+1)/2-th smallest
Even n: median = average of n/2-th and (n/2+1)-th smallest

arr = [7, 3, 1, 5, 9]
Median = 3rd smallest = 5

Using Quick Select: O(n) instead of O(n log n) with sorting
```

### Weighted Median

```
Each element has a weight. Find element where total weight
on each side is ≤ 1/2 of total weight.

Used in: Facility location, approximation algorithms
Can be found in O(n) using modified Quick Select.
```

---

## 2. Top-K Elements

```
Problem: Find the K largest elements (not necessarily sorted)

Method:
  1. Use Quick Select to find the K-th largest element → O(n)
  2. Partition: all elements ≥ K-th largest are the answer → O(n)
  Total: O(n)

Example: Find top 3 from [7, 3, 1, 5, 9, 4, 8]
  3rd largest = 7 (using Quick Select)
  Elements ≥ 7: [7, 9, 8]
  
  If you also need them SORTED: O(n + K log K)
```

---

## 3. K-th Largest Element (LeetCode 215)

```
The K-th largest = the (n - K + 1)-th smallest.

arr = [3, 2, 1, 5, 6, 4], K = 2
  2nd largest = (6 - 2 + 1) = 5th smallest
  Sorted: [1, 2, 3, 4, 5, 6]
  5th smallest = 5 → 2nd largest = 5 ✓

Solution: quickSelect(arr, n - K)  // 0-indexed
```

---

## 4. Median of Two Sorted Arrays

```
Problem: Find the median of two sorted arrays in O(log(min(m,n)))

This is a binary search problem, not Quick Select, but related:

arr1 = [1, 3, 8, 9, 15]
arr2 = [7, 11, 18, 19, 21, 25]

Combined median = 11 (6th smallest of 11 elements)

Binary search on partition point of shorter array.
```

---

## 5. Closest to Median

```
Problem: Find K elements closest to the median.

1. Find median using Quick Select → O(n)
2. Compute |arr[i] - median| for all elements → O(n)
3. Use Quick Select on absolute differences to find K-th → O(n)
4. All elements with |diff| ≤ K-th difference → O(n)

Total: O(n) — no sorting needed!
```

---

## 6. Optimal Partition for Load Balancing

```
Problem: Divide n tasks into two groups to minimize 
the difference in total workload.

This relates to the median:
  If you partition at the median, each group
  has similar total weight (approximately).

Quick Select is used in parallel computing to
efficiently balance load across processors.
```

---

## 7. Database Percentile Queries

```
SQL: SELECT PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY salary)
     FROM employees;

Finding the 90th percentile = finding the element at position
0.9 × n in sorted order.

With Quick Select: O(n) instead of O(n log n) full sort.

Used internally by database query optimizers.
```

---

## 8. Quicksort Pivot via Median

```
Using the true median as pivot guarantees balanced Quick Sort:

function optimalQuickSort(arr, lo, hi):
    if lo >= hi: return
    
    // Find true median in O(n)
    medianValue = quickSelect(arr, lo, hi, (lo+hi)/2)
    
    // Partition around median
    p = partition(arr, lo, hi, medianValue)
    
    optimalQuickSort(arr, lo, p - 1)
    optimalQuickSort(arr, p + 1, hi)

// Guaranteed O(n log n) — but the constant factor is large.
// This is why practical implementations prefer simpler pivots.
```

---

## 9. Streaming and Online Selection

```
Problem: Find median in a data stream.

Two heaps approach:
  Max-heap: stores smaller half
  Min-heap: stores larger half
  
  Insert: O(log n)
  Query median: O(1)

This is different from Quick Select (which needs all data),
but shows why the selection problem matters in streaming.
```

---

## Summary of Applications

| Application | Approach | Time |
|-------------|----------|------|
| Median | Quick Select at n/2 | O(n) |
| Top-K elements | Quick Select + partition | O(n) |
| K-th largest | Quick Select at n-K | O(n) |
| Closest to median | Two Quick Selects | O(n) |
| Load balancing | Partition at median | O(n) |
| Database percentiles | Quick Select at p×n | O(n) |
| Optimal QS pivot | Median as pivot | O(n log n) total |

---

## Quick Revision Questions

1. **How do you find the K-th largest using Quick Select designed for K-th smallest?**
2. **How can you find the K elements closest to the median in O(n)?**
3. **Why don't we always use the true median as pivot in Quick Sort?**
4. **What is the difference between selection in a static array vs a stream?**
5. **How does Top-K differ from finding just the K-th element?**

---

[← Previous: Median of Medians](04-median-of-medians.md) | [Next: Quick Select Applications Summary →](06-applications-summary.md)
