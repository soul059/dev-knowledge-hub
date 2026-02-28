# Range Sum Applications

## Chapter Overview

BIT excels at dynamic prefix/range sum problems. This chapter covers several practical applications where a BIT provides clean, efficient solutions to problems that would otherwise require complex approaches.

---

## Application 1: Dynamic Running Sum

```
Problem: Stream of values arriving one at a time.
  - Add a new value
  - Query sum of last k values
  - Query sum of values in position range [L, R]

Solution: BIT with a position counter.

  pos = 0
  function add(val):
      pos += 1
      update(pos, val)

  function sum_last_k():
      return range_sum(pos - k + 1, pos)

  function range_sum(L, R):
      return prefix_sum(R) - prefix_sum(L - 1)

Time: O(log n) per operation
Common use: Sliding window sums with arbitrary access
```

---

## Application 2: Prefix Sum with Updates

```
Problem: Array supports both updates and prefix queries.
  This is the NATIVE BIT problem.

Example: Score tracking
  - Player i scores points → update(i, points)
  - Total score of players 1..k → prefix_sum(k)
  - Score of players L..R → prefix_sum(R) - prefix_sum(L-1)

BIT is perfect: O(log n) update, O(log n) query
vs. naive array: O(1) update, O(n) prefix query
vs. prefix array: O(n) update, O(1) prefix query
BIT balances both!
```

---

## Application 3: Counting Elements Less Than X

```
Problem: Given a dynamic set of elements, count how many are < X.

Approach: Use BIT as a frequency array.

  BIT[v] = count of elements with value v
  prefix_sum(X-1) = count of elements < X
  prefix_sum(X) = count of elements ≤ X

  insert(v):  update(v, +1)
  delete(v):  update(v, -1)
  count_less(X): prefix_sum(X - 1)
  count_between(L, R): prefix_sum(R) - prefix_sum(L - 1)
  rank(v): prefix_sum(v)         // how many elements ≤ v

With coordinate compression: handles values up to 10^9.

Application: Online median, order statistics
```

---

## Application 4: K-th Smallest Element

```
Problem: Maintain a dynamic multiset, find k-th smallest element.

Approach: Binary search on BIT prefix sums.

Standard binary search: O(log² n)
  function kth_smallest(k):
      lo = 1, hi = max_val
      while lo < hi:
          mid = (lo + hi) / 2
          if prefix_sum(mid) >= k:
              hi = mid
          else:
              lo = mid + 1
      return lo

Optimized binary search on BIT: O(log n)
  function kth_smallest(k):
      pos = 0
      // Walk through BIT levels from highest bit to lowest
      bitMask = highest_power_of_2 ≤ n
      while bitMask > 0:
          next = pos + bitMask
          if next <= n and B[next] < k:
              k -= B[next]
              pos = next
          bitMask >>= 1
      return pos + 1

This works by descending through the BIT tree,
choosing left or right based on the count.
```

---

## Application 5: Cumulative Frequency Table

```
Problem: Maintain cumulative frequencies that can be updated.

Use case: Histogram with dynamic buckets.

  Buckets: [0-9], [10-19], [20-29], ...
  
  BIT index i → frequency of bucket i
  prefix_sum(k) → number of elements in buckets 0 to k
  
  Example:
    add_to_bucket(3, 5)  → 5 elements in bucket [30-39]
    cumulative(5)         → elements in buckets 0 through 5

  Percentile query:
    value at p-th percentile = kth_smallest(⌈total × p/100⌉)
```

---

## Application 6: Range Add + Point Sum (Difference BIT)

```
Problem: Add value to range [L,R], query single element.

  Use BIT on difference array.
  
  range_add(L, R, val):
      update(L, +val)
      if R+1 <= n: update(R+1, -val)
  
  point_query(i):
      return prefix_sum(i)

Example: Temperature adjustments
  "Increase temperature readings 3-7 by 5 degrees"
  "What is the adjusted reading at sensor 5?"
  
  range_add(3, 7, +5)
  point_query(5) → gives adjusted value

Much simpler than segment tree with lazy propagation!
```

---

## Pattern Recognition

```
When to use BIT for a problem:

✓ Problem involves SUMS (or XOR)
✓ Operations are point update + range query
✓ Or range update + point query
✓ Values can be mapped to indices [1, n]
✓ Online processing (queries interleaved with updates)

Red flags (need segment tree instead):
✗ Min/max queries
✗ Non-invertible operations
✗ Complex merge functions
✗ Need range update + range query (possible with 2 BITs but complex)
```

---

## Summary Table

| Application | BIT Size | Update | Query | Key Insight |
|------------|---------|--------|-------|-------------|
| Running sum | n positions | O(log n) | O(log n) | Standard BIT |
| Element counting | max_val | O(log n) | O(log n) | Frequency array |
| K-th smallest | max_val | O(log n) | O(log n) | Binary search on BIT |
| Cumulative frequency | #buckets | O(log n) | O(log n) | Prefix = cumulative |
| Range add + point query | n | O(log n) | O(log n) | Difference BIT |

---

## Quick Revision Questions

1. How do you count elements less than X using BIT? *(BIT as frequency array, prefix_sum(X-1))*
2. How do you find the k-th smallest with BIT? *(Binary search on prefix sums, or O(log n) BIT walk)*
3. What's the difference BIT used for? *(Range add + point query: update(L, +val), update(R+1, -val), point = prefix_sum(i))*
4. When should you prefer BIT over segment tree? *(When the operation is invertible (sum, XOR) and the query pattern fits prefix decomposition)*
5. What is coordinate compression used for? *(Mapping large values to [1, n] so BIT indices are manageable)*

---

[← Previous: Counting Inversions](01-counting-inversions.md) | [Next: 2D BIT →](03-2d-bit.md)
