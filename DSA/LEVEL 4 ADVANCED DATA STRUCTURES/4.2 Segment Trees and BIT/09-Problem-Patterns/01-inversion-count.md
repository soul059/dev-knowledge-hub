# Inversion Count

## Chapter Overview

Counting inversions is one of the most classic applications of both BIT and segment trees. An inversion is a pair (i, j) where i < j but A[i] > A[j]. This chapter covers multiple approaches and deep analysis of the problem.

---

## Problem Definition

```
Given array A[1..n], count pairs (i, j) where:
  i < j  AND  A[i] > A[j]

Example: A = [3, 1, 2]
  (1,2): A[1]=3 > A[2]=1  ✓ inversion
  (1,3): A[1]=3 > A[3]=2  ✓ inversion
  (2,3): A[2]=1 > A[3]=2  ✗ not inversion
  
  Answer: 2 inversions

Inversions measure "how unsorted" an array is:
  Sorted array: 0 inversions
  Reverse sorted: n(n-1)/2 inversions (maximum)
```

---

## Approach 1: BIT (Optimal)

```
Idea: Process elements LEFT to RIGHT.
For each A[i], count how many previously seen elements are GREATER.

Algorithm:
  inversions = 0
  for i from 1 to n:
      // How many elements already inserted are > A[i]?
      // = (total inserted so far) - (count of elements ≤ A[i])
      inversions += (i - 1) - prefix_query(A[i])
      
      // Mark A[i] as seen
      update(A[i], +1)
  
  return inversions

The BIT acts as a frequency array on VALUES.
prefix_query(X) = count of elements ≤ X seen so far.
```

---

## Step-by-Step Trace

```
A = [3, 1, 4, 2]     (values in range [1, 4])
BIT on values [1..4], initially all zeros.

i=1: A[1] = 3
  Elements seen before: 0
  Elements ≤ 3 seen: prefix(3) = 0
  Inversions from this element: 0 - 0 = 0
  update(3, +1) → BIT: [0, 0, 1, 0]  (value 3 seen once)
  Total inversions: 0

i=2: A[2] = 1
  Elements seen before: 1
  Elements ≤ 1 seen: prefix(1) = 0
  Inversions from this: 1 - 0 = 1    (3 > 1)
  update(1, +1) → BIT: [1, 0, 1, 0]
  Total inversions: 1

i=3: A[3] = 4
  Elements seen before: 2
  Elements ≤ 4 seen: prefix(4) = 2   (values 1 and 3)
  Inversions from this: 2 - 2 = 0
  update(4, +1) → BIT: [1, 0, 1, 1]
  Total inversions: 1

i=4: A[4] = 2
  Elements seen before: 3
  Elements ≤ 2 seen: prefix(2) = 1   (value 1)
  Inversions from this: 3 - 1 = 2    (3 > 2 and 4 > 2)
  update(2, +1) → BIT: [1, 1, 1, 1]
  Total inversions: 1 + 2 = 3

Answer: 3 inversions
Verify: (3,1), (3,2), (4,2) ✓
```

---

## Coordinate Compression

```
If values can be up to 10^9, we can't create BIT of size 10^9.
Solution: compress values to [1..n].

A = [100, 5, 999, 50]

Step 1: Sort unique values with indices
  sorted: [(5,2), (50,4), (100,1), (999,3)]

Step 2: Map to ranks
  5 → 1, 50 → 2, 100 → 3, 999 → 4

Step 3: Replace values
  A' = [3, 1, 4, 2]

Step 4: Count inversions on A' using BIT of size n

Important: compression preserves relative order,
so inversion count is identical.
```

---

## Approach 2: Segment Tree

```
Same logic as BIT, but using a segment tree on values:

inversions = 0
for i from 1 to n:
    // Query: count of values in range [A[i]+1, MAX] already seen
    inversions += query(1, 1, MAX, A[i]+1, MAX)
    
    // Update: mark A[i] as seen
    update(1, 1, MAX, A[i], +1)

return inversions

Advantage over BIT: directly query count of elements > A[i]
without the subtraction trick.

Same complexity: O(n log n)
```

---

## Approach 3: Merge Sort (Divide & Conquer)

```
Modified merge sort counts inversions during the merge step.

merge_count(A, l, r):
    if l >= r: return 0
    mid = (l + r) / 2
    count = merge_count(A, l, mid)      // left half inversions
          + merge_count(A, mid+1, r)    // right half inversions
          + merge_and_count(A, l, mid, r) // cross inversions
    return count

merge_and_count(A, l, mid, r):
    // Merge two sorted halves and count inversions
    i = l, j = mid + 1, count = 0
    temp = []
    while i <= mid AND j <= r:
        if A[i] <= A[j]:
            temp.append(A[i]); i++
        else:
            // A[j] < A[i] → A[j] is inverted with ALL remaining
            // elements in left half: A[i], A[i+1], ..., A[mid]
            count += (mid - i + 1)
            temp.append(A[j]); j++
    // append remaining
    copy temp back to A[l..r]
    return count

Time: O(n log n), Space: O(n)
```

---

## Comparison of Approaches

```
                    BIT          Segment Tree    Merge Sort
Time                O(n log n)   O(n log n)      O(n log n)
Space               O(n)         O(4n)           O(n)
Code length         Short        Medium          Medium
Coord compress?     Yes          Yes             No (works on values)
Online (streaming)  Yes          Yes             No (needs all data)
Constant factor     Smallest     Medium          Medium
Extra functionality Easy to      Easy to         Hard to
                    extend       extend          extend
```

---

## Variations

### Count Inversions with Updates

```
"After changing A[i] to val, report new inversion count"

With BIT/SegTree: 
  Remove old value, add new value, adjust count.
  But recomputing total count from scratch each time is O(n log n).

Better: Maintain count incrementally
  When A[i] changes from old to new:
    Remove inversions involving old value at position i
    Add inversions involving new value at position i
  
  Inversions at position i:
    left_inversions = count of j < i with A[j] > A[i]
    right_inversions = count of j > i with A[j] < A[i]
  
  Need two BITs:
    BIT_prefix: for counting A[j] > X for j < i
    BIT_suffix: for counting A[j] < X for j > i
```

### Count of Significant Inversions

```
"Count pairs (i,j) where i < j and A[i] > 2 × A[j]"

BIT approach:
  for each A[i], query count of values ≤ A[i]/2 already seen
  (or equivalent: for each A[j], query count of values > 2×A[j])
  
  Same O(n log n) with coordinate compression.
```

---

## Practice Problems

```
1. Count inversions in array → standard BIT
2. Minimum adjacent swaps to sort → equals inversion count!
3. Count of range inversions → segment tree with merge sort tree
4. Number of significant inversions (A[i] > 2·A[j])
5. Count inversions after each swap
```

---

## Summary Table

| Approach | Time | Space | Online | Updates |
|----------|------|-------|--------|---------|
| BIT | O(n log n) | O(n) | Yes | O(log n) per |
| Segment Tree | O(n log n) | O(4n) | Yes | O(log n) per |
| Merge Sort | O(n log n) | O(n) | No | Rebuild needed |
| Brute Force | O(n²) | O(1) | Yes | O(n) per |

---

## Quick Revision Questions

1. What is an inversion? *(A pair (i, j) with i < j but A[i] > A[j])*
2. How does BIT count inversions? *(Process left to right; for each element, count previously seen elements greater than it using (i-1) - prefix_query(A[i]))*
3. Why is coordinate compression needed? *(BIT array size must match the value range; compression maps values to [1..n])*
4. What is the connection between inversions and sorting? *(Inversion count = minimum adjacent swaps needed to sort the array)*
5. Can inversions be counted in O(n log n) without BIT? *(Yes, using modified merge sort which counts cross-inversions during the merge step)*

---

[← Previous: Dynamic Segment Tree](../08-Advanced-Segment-Tree/06-dynamic-segment-tree.md) | [Next: Range Updates with Queries →](02-range-updates-with-queries.md)
