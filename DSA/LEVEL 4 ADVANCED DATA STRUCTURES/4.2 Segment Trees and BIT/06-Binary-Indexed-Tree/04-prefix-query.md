# Prefix Query in BIT

## Chapter Overview

The prefix query is BIT's primary operation: compute the sum (or aggregate) of elements from index 1 to i. Range queries are derived from prefix queries using the inverse operation.

---

## The Algorithm

```
function prefix_sum(i):
    result = 0
    while i > 0:
        result += B[i]
        i -= i & (-i)      // i -= lowbit(i)
    return result

That's it. 3 lines for the complete prefix sum query.

What it does:
  1. Start at index i
  2. B[i] covers [i - lowbit(i) + 1, i] → add it
  3. Jump to i - lowbit(i), which covers the NEXT chunk
  4. Continue until i = 0 (entire prefix covered)
```

---

## Walkthrough: prefix_sum(7) with n=8

```
A = [_, 3, 1, 4, 2, 5, 3, 7]  (1-indexed)

BIT: B[1]=3, B[2]=4, B[3]=4, B[4]=10, B[5]=5, B[6]=8, B[7]=7, B[8]=25

Step 1: i = 7 = 111₂
  result += B[7] = 0 + 7 = 7     (B[7] covers [7,7])
  i -= lowbit(7) = 7 - 1 = 6

Step 2: i = 6 = 110₂
  result += B[6] = 7 + 8 = 15    (B[6] covers [5,6])
  i -= lowbit(6) = 6 - 2 = 4

Step 3: i = 4 = 100₂
  result += B[4] = 15 + 10 = 25  (B[4] covers [1,4])
  i -= lowbit(4) = 4 - 4 = 0

Step 4: i = 0 → STOP

prefix_sum(7) = 25
Check: 3+1+4+2+5+3+7 = 25  ✓

Ranges visited: [7,7] + [5,6] + [1,4] = [1,7]  ✓ (complete coverage)
```

---

## Why the Ranges Don't Overlap

```
Key insight: Each step's range ENDS where the previous step's range BEGINS.

prefix_sum(13):
  i=13, range [13,13], next: 13-1=12
  i=12, range [9,12],  next: 12-4=8
  i=8,  range [1,8],   next: 8-8=0

  [13,13] + [9,12] + [1,8] = [1,13]

  The ranges are CONTIGUOUS and NON-OVERLAPPING:
  [1, 8] [9, 12] [13, 13]
     └──────┴───────┘
       Complete coverage

This is guaranteed by the construction:
  B[i] covers [i - lowbit(i) + 1, i]
  Next index is i - lowbit(i)
  Its range ends at i - lowbit(i), which is exactly
  one less than the start of the current range.
```

---

## Range Query via Prefix Difference

```
range_sum(L, R) = prefix_sum(R) - prefix_sum(L - 1)

This works because sum is INVERTIBLE (subtraction undoes addition).

Example: sum(3, 6)
  = prefix_sum(6) - prefix_sum(2)
  = (3+1+4+2+5+3) - (3+1)
  = 18 - 4
  = 14
Check: 4+2+5+3 = 14  ✓

Time: O(log n) + O(log n) = O(log n)
```

---

## Trace: Range Sum

```
A = [_, 3, 1, 4, 2, 5, 3, 7, 6]   n=8
B = [_, 3, 4, 4, 10, 5, 8, 7, 32]

range_sum(3, 6):

  prefix_sum(6):
    i=6: result += B[6]=8,   i=6-2=4
    i=4: result += B[4]=10,  i=4-4=0
    result = 18

  prefix_sum(2):
    i=2: result += B[2]=4,   i=2-2=0
    result = 4

  Answer: 18 - 4 = 14

  Check: A[3]+A[4]+A[5]+A[6] = 4+2+5+3 = 14  ✓
```

---

## Prefix Sum of the Entire Array

```
prefix_sum(n) where n is a power of 2:

  i = n (a single set bit, like 1000...0)
  result += B[n]  (covers [1, n])
  i -= lowbit(n) = n - n = 0
  STOP.

  Just 1 iteration! B[n] = sum of entire array when n is power of 2.

prefix_sum(n) where n is NOT a power of 2, e.g., n=13:
  i=13: B[13], i=12
  i=12: B[12], i=8
  i=8:  B[8],  i=0
  3 iterations.

Worst case: n = 2^k - 1 (all bits set)
  e.g., n=15=1111₂ → 4 iterations → O(log n)
```

---

## XOR Prefix Query

```
BIT works for any invertible, associative operation.
For XOR:

function prefix_xor(i):
    result = 0         // identity for XOR
    while i > 0:
        result ^= B[i]
        i -= i & (-i)
    return result

function update_xor(i, delta):
    while i <= n:
        B[i] ^= delta
        i += i & (-i)

range_xor(L, R) = prefix_xor(R) ^ prefix_xor(L-1)

Works because:
  XOR is associative ✓
  XOR is its own inverse (a ^ a = 0) ✓
  Identity = 0 ✓
```

---

## Common Patterns

```
1. FREQUENCY COUNTING
   A[i] = number of occurrences of value i
   prefix_sum(v) = count of values ≤ v
   → Useful for order statistics, counting inversions

2. 0/1 ARRAY
   A[i] = 1 if element exists, 0 otherwise
   prefix_sum(i) = count of existing elements up to index i
   → Useful for online rank queries

3. COORDINATE COMPRESSED
   Map large values to [1, n]
   Use BIT on the compressed coordinates
   → Handles values up to 10^9 with only n BIT nodes
```

---

## Point Query Using BIT

```
Get single element A[i]:

Method 1: Prefix difference
  A[i] = prefix_sum(i) - prefix_sum(i-1)
  Time: O(log n)  — two prefix queries

Method 2: Keep separate array
  Maintain original A[] alongside B[]
  A[i] is O(1) but requires O(n) extra space

Method 3: Clever formula (Fenwick's trick)
  function point_query(i):
      result = B[i]
      parent = i - (i & (-i))
      i -= 1
      while i != parent:
          result -= B[i]
          i -= i & (-i)
      return result
  Time: O(log n) but often fewer iterations than Method 1
```

---

## Summary Table

| Operation | Formula | Time |
|-----------|---------|------|
| Prefix sum(i) | While i > 0: sum += B[i], i -= lowbit(i) | O(log n) |
| Range sum(L,R) | prefix_sum(R) - prefix_sum(L-1) | O(log n) |
| Point query(i) | prefix_sum(i) - prefix_sum(i-1) | O(log n) |
| Prefix XOR(i) | While i > 0: result ^= B[i], i -= lowbit(i) | O(log n) |
| Range XOR(L,R) | prefix_xor(R) ^ prefix_xor(L-1) | O(log n) |

---

## Quick Revision Questions

1. What is the prefix query algorithm? *(While i > 0: result += B[i], i -= i & (-i))*
2. How do you compute a range sum [L, R] with BIT? *(prefix_sum(R) - prefix_sum(L-1))*
3. Why don't the BIT ranges overlap during a prefix query? *(Each step's range ends exactly where the next step's range begins — guaranteed by the lowbit structure)*
4. What's the maximum number of iterations for prefix_sum(n)? *(Number of set bits in n, at most ⌊log₂ n⌋ + 1)*
5. Can BIT answer a single-element query? *(Yes: A[i] = prefix_sum(i) - prefix_sum(i-1))*

---

[← Previous: Point Update](03-point-update.md) | [Next: Range Query with BIT →](05-range-query-with-bit.md)
