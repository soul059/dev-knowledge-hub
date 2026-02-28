# Counting Inversions

## Chapter Overview

Counting inversions (pairs where i < j but A[i] > A[j]) is a classic problem elegantly solved by BIT. The approach processes elements from right to left (or by value), using BIT as a dynamic frequency counter.

---

## Problem Definition

```
Inversion: A pair (i, j) where i < j and A[i] > A[j]

Example: A = [3, 1, 2, 5, 4]

Inversions:
  (0,1): A[0]=3 > A[1]=1  ✓
  (0,2): A[0]=3 > A[2]=2  ✓
  (3,4): A[3]=5 > A[4]=4  ✓

Total: 3 inversions

Brute force: Check all pairs → O(n²)
BIT approach: O(n log n)
```

---

## The BIT Approach

```
Idea: Process array from LEFT to RIGHT.
For each element A[i]:
  1. Query: How many elements already seen are GREATER than A[i]?
     → These form inversions with A[i] (they came before, but are larger)
     → inversions += (number of elements seen so far) - prefix_sum(A[i])
     
     Or equivalently:
     → inversions += count of elements in range [A[i]+1, max_val]
  
  2. Update: Mark A[i] as "seen" → update(A[i], +1)

BIT acts as a frequency array where B[v] counts occurrences of value v.

Algorithm:
  inversions = 0
  for i = 0 to n-1:
      inversions += i - prefix_sum(A[i])   // elements seen but > A[i]
      update(A[i], +1)                      // mark A[i] as seen
  return inversions
```

---

## Worked Example

```
A = [3, 1, 2, 5, 4]    values in range [1, 5]
BIT of size 5+1, initialized to 0.

i=0, A[0]=3:
  seen = 0, prefix_sum(3) = 0
  inversions += 0 - 0 = 0
  update(3, +1)    BIT: freq[3]=1
  inversions = 0

i=1, A[1]=1:
  seen = 1, prefix_sum(1) = 0   (no value ≤ 1 seen yet)
  inversions += 1 - 0 = 1       (one element > 1 before it: 3)
  update(1, +1)    BIT: freq[1]=1, freq[3]=1
  inversions = 1

i=2, A[2]=2:
  seen = 2, prefix_sum(2) = 1   (one value ≤ 2 seen: 1)
  inversions += 2 - 1 = 1       (one element > 2 before it: 3)
  update(2, +1)    BIT: freq[1]=1, freq[2]=1, freq[3]=1
  inversions = 2

i=3, A[3]=5:
  seen = 3, prefix_sum(5) = 3   (three values ≤ 5 seen: 1,2,3)
  inversions += 3 - 3 = 0       (no element > 5 before it)
  update(5, +1)
  inversions = 2

i=4, A[4]=4:
  seen = 4, prefix_sum(4) = 3   (values ≤ 4 seen: 1,2,3)
  inversions += 4 - 3 = 1       (one element > 4 before it: 5)
  update(4, +1)
  inversions = 3

Total inversions: 3  ✓
```

---

## Coordinate Compression

```
When values are large (up to 10^9), BIT can't index them directly.
Solution: Compress coordinates to [1, n].

Steps:
  1. Sort unique values
  2. Map each value to its rank

Example:
  A = [100, 3, 50, 1000, 7]
  
  Sorted unique: [3, 7, 50, 100, 1000]
  Rank mapping: 3→1, 7→2, 50→3, 100→4, 1000→5
  
  Compressed: [4, 1, 3, 5, 2]
  
  Now use BIT of size 5 instead of size 1000.
  Inversions are preserved because ranks maintain relative order.

Time: O(n log n) for sorting + O(n log n) for BIT = O(n log n)
```

---

## Alternative: Right-to-Left Processing

```
Process from RIGHT to LEFT. For each A[i]:
  inversions += prefix_sum(A[i] - 1)   // count elements smaller than A[i] AFTER it

Wait — that counts elements AFTER i that are SMALLER — not inversions.
Actually: elements after i that are smaller → NOT inversions (i < j, A[i] > A[j]).
For right-to-left: we want elements after i that we've already added (these are to the RIGHT).
  inversions += prefix_sum(A[i] - 1)   // elements to the right and smaller?

Let me reconsider:
  Right to left, element to the right of i are already in BIT.
  prefix_sum(A[i]-1) = count of elements to the RIGHT that are < A[i]
  These form inversions! (i < j but A[i] > A[j])

Algorithm:
  inversions = 0
  for i = n-1 down to 0:
      inversions += prefix_sum(A[i] - 1)
      update(A[i], +1)

Same result, just a different way to think about it.
```

---

## Complexity

```
Time:  O(n log n)
  - Coordinate compression: O(n log n) for sorting
  - BIT operations: n × O(log n) = O(n log n)

Space: O(n)
  - BIT array: O(n)
  - Compressed values: O(n)

Comparison:
  Brute force: O(n²) — too slow for n ≥ 10^5
  Merge sort:  O(n log n) — classic divide-and-conquer approach
  BIT:         O(n log n) — often simpler to code
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Count pairs (i,j) with i < j and A[i] > A[j] |
| BIT role | Frequency counter for values seen so far |
| Left-to-right | inversions += (seen count) - prefix_sum(A[i]) |
| Right-to-left | inversions += prefix_sum(A[i] - 1) |
| Coordinate compression | Map values to [1,n] preserving relative order |
| Time | O(n log n) |
| Space | O(n) |

---

## Quick Revision Questions

1. What does the BIT represent when counting inversions? *(A frequency array — BIT[v] = count of value v seen so far)*
2. When processing left-to-right, how do you count inversions for element A[i]? *(inversions += i - prefix_sum(A[i]), which counts elements seen that are greater)*
3. Why is coordinate compression needed? *(Values may be up to 10^9, but BIT index must be at most n)*
4. Does coordinate compression change the inversion count? *(No — it preserves relative order)*
5. What's the time complexity? *(O(n log n) for sorting + O(n log n) for BIT = O(n log n))*

---

[← Previous Unit: Building a BIT](../06-Binary-Indexed-Tree/06-building-a-bit.md) | [Next: Range Sum Applications →](02-range-sum-applications.md)
