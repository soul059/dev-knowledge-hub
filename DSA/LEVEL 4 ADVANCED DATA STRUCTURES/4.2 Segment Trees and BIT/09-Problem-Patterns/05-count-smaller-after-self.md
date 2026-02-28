# Count Smaller After Self

## Chapter Overview

"For each element A[i], count how many elements to its RIGHT are strictly smaller." This is one of the most popular interview and competitive programming problems. It's closely related to inversion counting and showcases the power of BIT and segment trees.

---

## Problem Statement

```
Given array A[1..n], for each i compute:
  result[i] = |{ j : j > i AND A[j] < A[i] }|

Example: A = [5, 2, 6, 1]
  result[1] = count of elements after 5 that are < 5 = {2, 1} = 2
  result[2] = count of elements after 2 that are < 2 = {1}   = 1
  result[3] = count of elements after 6 that are < 6 = {1}   = 1
  result[4] = count of elements after 1 that are < 1 = {}    = 0
  
  Output: [2, 1, 1, 0]
```

---

## Approach: Process RIGHT to LEFT with BIT

```
Key insight: Process elements from RIGHT to LEFT.
When processing A[i], all elements to its right are already in the BIT.

Algorithm:
  result[1..n] = all zeros
  
  for i from n down to 1:
      // Count elements already in BIT that are < A[i]
      result[i] = BIT.prefix_query(A[i] - 1)
      
      // Insert A[i] into BIT
      BIT.update(A[i], +1)
  
  return result

The BIT is a frequency array on VALUES.
prefix_query(X) = count of values ≤ X inserted so far (= elements to the right).

Time: O(n log n)
Space: O(n)
```

---

## Step-by-Step Trace

```
A = [5, 2, 6, 1]
Coordinate compress: 5→3, 2→2, 6→4, 1→1
Compressed: [3, 2, 4, 1]

BIT on values [1..4], initially all zeros.

i=4: A[i]=1 (compressed)
  result[4] = prefix(1 - 1) = prefix(0) = 0
  update(1, +1) → BIT freq: [1, 0, 0, 0]
  result = [_, _, _, 0]

i=3: A[i]=4
  result[3] = prefix(4 - 1) = prefix(3) = 1  (value 1 is ≤ 3)
  update(4, +1) → BIT freq: [1, 0, 0, 1]
  result = [_, _, 1, 0]

i=2: A[i]=2
  result[2] = prefix(2 - 1) = prefix(1) = 1  (value 1 is ≤ 1)
  update(2, +1) → BIT freq: [1, 1, 0, 1]
  result = [_, 1, 1, 0]

i=1: A[i]=3
  result[1] = prefix(3 - 1) = prefix(2) = 2  (values 1 and 2 are ≤ 2)
  update(3, +1) → BIT freq: [1, 1, 1, 1]
  result = [2, 1, 1, 0]

Answer: [2, 1, 1, 0] ✓
```

---

## With Segment Tree

```
Same logic, using segment tree for frequency counting:

for i from n down to 1:
    // Count elements with value in [1, A[i]-1]
    result[i] = seg.query_sum(1, A[i] - 1)
    
    // Mark A[i] as seen
    seg.update(A[i], +1)

Time: O(n log n)

Segment tree is more flexible if you also need:
  - Count of elements GREATER than A[i] to the right
  - K-th smallest among right elements
  - Min/max of right elements less than A[i]
```

---

## Variations

### Count Smaller Before Self

```
"For each A[i], count elements to the LEFT that are < A[i]"

Process LEFT to RIGHT:
  for i from 1 to n:
      result[i] = BIT.prefix_query(A[i] - 1)
      BIT.update(A[i], +1)
```

### Count Greater After Self

```
"For each A[i], count elements to the RIGHT that are > A[i]"

Process RIGHT to LEFT:
  for i from n down to 1:
      total_right = (n - i) - ... // elements already inserted
      // Elements > A[i] = total inserted - elements ≤ A[i]
      result[i] = total_inserted - BIT.prefix_query(A[i])
      BIT.update(A[i], +1)
      total_inserted += 1

Or equivalently:
  result[i] = BIT.range_query(A[i]+1, MAX_VAL)
```

### Count Greater Before Self

```
"For each A[i], count elements to the LEFT that are > A[i]"
(This counts inversions involving A[i] from the left side)

Process LEFT to RIGHT:
  for i from 1 to n:
      result[i] = (i - 1) - BIT.prefix_query(A[i])
      BIT.update(A[i], +1)
```

---

## Summary of All Four Variants

```
┌──────────────────────┬───────────────┬────────────────────────┐
│ Variant              │ Process order │ Formula                │
├──────────────────────┼───────────────┼────────────────────────┤
│ Smaller after (< →)  │ Right to left │ prefix(A[i] - 1)       │
│ Smaller before (← <) │ Left to right │ prefix(A[i] - 1)       │
│ Greater after (> →)  │ Right to left │ inserted - prefix(A[i])│
│ Greater before (← >) │ Left to right │ (i-1) - prefix(A[i])  │
└──────────────────────┴───────────────┴────────────────────────┘

All are O(n log n) with BIT.
```

---

## Handling Duplicates

```
"Count STRICTLY smaller" vs "Count smaller or equal"

Strictly smaller (A[j] < A[i]):
  result[i] = prefix(A[i] - 1)     ← excludes A[i] itself

Smaller or equal (A[j] ≤ A[i]):
  result[i] = prefix(A[i])         ← includes A[i]

For duplicates processed at same time:
  If elements with same value processed right-to-left,
  earlier same-value elements see later same-value elements in BIT.
  
  "Strictly smaller" correctly excludes equal elements
  because we query prefix(A[i] - 1).
```

---

## Merge Sort Approach

```
Modified merge sort can also solve this problem.

During merge of left half and right half (right half = "after"):
  When right element < left element:
    It contributes to count_smaller_after for ALL remaining left elements.

merge_count(A, indices, result, l, r):
    if l >= r: return
    mid = (l + r) / 2
    merge_count(A, indices, result, l, mid)
    merge_count(A, indices, result, mid+1, r)
    
    // Merge step
    i = l, j = mid + 1, right_count = 0
    temp = []
    while i <= mid AND j <= r:
        if A[indices[j]] < A[indices[i]]:
            right_count += 1
            temp.append(indices[j]); j++
        else:
            result[indices[i]] += right_count
            temp.append(indices[i]); i++
    // remaining left elements get right_count
    while i <= mid:
        result[indices[i]] += right_count
        temp.append(indices[i]); i++
    // copy temp back

Time: O(n log n), Space: O(n)
```

---

## Practice Problems

```
1. Count of Smaller Numbers After Self (LeetCode 315)
2. Count of Range Sum (LeetCode 327)
3. Reverse Pairs (LeetCode 493) — A[i] > 2×A[j]
4. Count Inversions
5. Create Sorted Array through Instructions (LeetCode 1649)
```

---

## Summary Table

| Approach | Time | Space | Updates | Code Length |
|----------|------|-------|---------|------------|
| BIT (right to left) | O(n log n) | O(n) | N/A | Short |
| Segment Tree | O(n log n) | O(4n) | Supported | Medium |
| Merge Sort | O(n log n) | O(n) | N/A | Medium |
| Brute Force | O(n²) | O(1) | N/A | Trivial |

---

## Quick Revision Questions

1. Why process right to left? *(So that when we process A[i], all elements to its right are already in the BIT)*
2. What does the BIT store? *(Frequency of each value — BIT acts as a frequency counter on the value domain)*
3. How do you handle large values (up to 10^9)? *(Coordinate compression to [1..n])*
4. How do you count greater-after instead of smaller-after? *(total_inserted - prefix(A[i]) = count of values > A[i] among right elements)*
5. What is the connection to inversions? *(Sum of count_smaller_after_self for all elements = total inversion count)*

---

[← Previous: Longest Increasing Subsequence](04-longest-increasing-subsequence.md) | [Next: Rectangle Area →](06-rectangle-area.md)
