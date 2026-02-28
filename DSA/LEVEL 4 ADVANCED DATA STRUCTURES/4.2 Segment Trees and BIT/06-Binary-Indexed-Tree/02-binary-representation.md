# Binary Representation in BIT

## Chapter Overview

The elegance of BIT lies in how it uses binary representation to determine tree structure. Understanding the binary mechanics is key to grasping why BIT operations are O(log n) and how they work.

---

## The lowbit Function

```
lowbit(i) = i & (-i)

This isolates the LOWEST SET BIT of i.

How it works (two's complement):
  -i = ~i + 1  (flip all bits, add 1)

Example: i = 12 = 1100₂
  -i = 0011 + 1 = 0100₂  (but we keep same bit width for &)
  Actually in two's complement:
  ~12 = ...0011₂
  -12 = ...0100₂
  12 & -12 = ...1100 & ...0100 = ...0100 = 4

Walkthrough for several values:
  i=1  = 0001   -i = 1111   lowbit = 0001 = 1
  i=2  = 0010   -i = 1110   lowbit = 0010 = 2
  i=3  = 0011   -i = 1101   lowbit = 0001 = 1
  i=4  = 0100   -i = 1100   lowbit = 0100 = 4
  i=5  = 0101   -i = 1011   lowbit = 0001 = 1
  i=6  = 0110   -i = 1010   lowbit = 0010 = 2
  i=7  = 0111   -i = 1001   lowbit = 0001 = 1
  i=8  = 1000   -i = 1000   lowbit = 1000 = 8
  i=10 = 1010   -i = 0110   lowbit = 0010 = 2
  i=12 = 1100   -i = 0100   lowbit = 0100 = 4

Pattern: lowbit of odd numbers is always 1
         lowbit of powers of 2 is the number itself
```

---

## Query Path: Stripping Lowest Bits

```
To compute prefix_sum(i):
  Start at index i
  Add B[i] to result
  Remove lowest set bit: i -= lowbit(i)
  Repeat until i = 0

Example: prefix_sum(13)
  i = 13 = 1101₂
    Add B[13]  (covers [13,13])
    i -= lowbit(13) = 13 - 1 = 12

  i = 12 = 1100₂
    Add B[12]  (covers [9,12])
    i -= lowbit(12) = 12 - 4 = 8

  i = 8  = 1000₂
    Add B[8]   (covers [1,8])
    i -= lowbit(8) = 8 - 8 = 0

  STOP.

  prefix_sum(13) = B[13] + B[12] + B[8]
  Ranges:          [13]  + [9,12] + [1,8]  = [1,13]  ✓

Steps = number of set bits in the ORIGINAL index
  13 = 1101₂ → 3 set bits → 3 steps

Worst case: all bits set (e.g., 15 = 1111₂ → 4 steps for n=16)
  In general: at most ⌊log₂ n⌋ + 1 steps → O(log n)
```

---

## Update Path: Adding Lowest Bits

```
To update index i (add val to A[i]):
  Start at index i
  Add val to B[i]
  Add lowest set bit: i += lowbit(i)
  Repeat until i > n

Example: update index 5, array size n=16
  i = 5  = 00101₂
    B[5] += val   (B[5] covers [5,5])
    i += lowbit(5) = 5 + 1 = 6

  i = 6  = 00110₂
    B[6] += val   (B[6] covers [5,6])
    i += lowbit(6) = 6 + 2 = 8

  i = 8  = 01000₂
    B[8] += val   (B[8] covers [1,8])
    i += lowbit(8) = 8 + 8 = 16

  i = 16 = 10000₂
    B[16] += val  (B[16] covers [1,16])
    i += lowbit(16) = 16 + 16 = 32

  32 > 16 → STOP.

  Updated: B[5], B[6], B[8], B[16]
  These are exactly all the nodes whose range INCLUDES index 5!

Steps = at most ⌊log₂ n⌋ + 1 → O(log n)
```

---

## Why the Paths are Complementary

```
Query path:  i -= lowbit(i)   → strips bits  → goes to ANCESTORS
Update path: i += lowbit(i)   → adds bits    → goes to PARENTS

Query at 7:
  7 = 0111 → B[7]
  6 = 0110 → B[6]
  4 = 0100 → B[4]
  0 = stop

Update at 3:
  3  = 0011 → B[3]
  4  = 0100 → B[4]
  8  = 1000 → B[8]
  16 = stop (if n≥16)

Notice: 4 appears in BOTH paths for query(7) and update(3)
  Because B[4] covers [1,4] which includes both 3 and influences prefix(7)

This ensures correctness: every update propagates to all
BIT nodes that include it in their prefix sums.
```

---

## Ancestor-Descendant Relationships

```
The "parent" of node i in BIT: i + lowbit(i)
The "children" relate by: i - lowbit(i) leads to the previous sibling

Example tree structure (n=8):

Index:  1    2    3    4    5    6    7    8
        │    │    │    │    │    │    │    │
        └──→ 2    │    │    │    │    │    │
             └────┼──→ 4    │    │    │    │
             3 ───┘    │    │    │    │    │
                       └────┼────┼──→ 8    │
                       5 ──→ 6    │    │    │
                             └────┘    │    │
                             7 ────────┘    │

Parent chain from any node to root:
  1 → 2 → 4 → 8
  3 → 4 → 8
  5 → 6 → 8
  7 → 8

Every chain has at most log₂(n) + 1 nodes.
```

---

## Range of Each BIT Node

```
B[i] covers indices [i - lowbit(i) + 1, i]

i    binary   lowbit   start              end   range
1    0001      1       1-1+1=1             1    [1,1]
2    0010      2       2-2+1=1             2    [1,2]
3    0011      1       3-1+1=3             3    [3,3]
4    0100      4       4-4+1=1             4    [1,4]
5    0101      1       5-1+1=5             5    [5,5]
6    0110      2       6-2+1=5             6    [5,6]
7    0111      1       7-1+1=7             7    [7,7]
8    1000      8       8-8+1=1             8    [1,8]
9    1001      1       9-1+1=9             9    [9,9]
10   1010      2       10-2+1=9           10    [9,10]
11   1011      1       11-1+1=11          11    [11,11]
12   1100      4       12-4+1=9           12    [9,12]
13   1101      1       13-1+1=13          13    [13,13]
14   1110      2       14-2+1=13          14    [13,14]
15   1111      1       15-1+1=15          15    [15,15]
16   10000     16      16-16+1=1          16    [1,16]

Powers of 2 cover the largest ranges (starting from 1).
Odd indices always cover exactly 1 element.
```

---

## Number of Steps Analysis

```
Query steps  = number of SET bits in i  (popcount)
Update steps = number of ZERO bits below the highest set bit + 1

For n = 16 (4-bit indices):
  Index   Binary  Query steps  Update steps
   1      0001       1             4
   2      0010       1             3
   3      0011       2             3
   4      0100       1             2
   5      0101       2             3
   6      0110       2             2
   7      0111       3             2
   8      1000       1             1
   9      1001       2             3
  10      1010       2             2
  11      1011       3             2
  12      1100       2             1
  13      1101       3             1
  14      1110       3             1
  15      1111       4             1

Average query steps ≈ log₂(n) / 2
Worst case: O(log n) for both operations
```

---

## Summary Table

| Concept | Formula | Purpose |
|---------|---------|---------|
| lowbit(i) | i & (-i) | Isolate lowest set bit |
| Range of B[i] | [i - lowbit(i) + 1, i] | Which elements B[i] covers |
| Range length | lowbit(i) | How many elements B[i] covers |
| Query next | i -= lowbit(i) | Move to previous non-overlapping range |
| Update next | i += lowbit(i) | Move to parent node |
| Query steps | popcount(i) | Number of iterations |
| Max steps | ⌊log₂ n⌋ + 1 | Worst case for both operations |

---

## Quick Revision Questions

1. What does i & (-i) compute? *(The lowest set bit of i — its value, not position)*
2. How does the query path traverse? *(By repeatedly removing the lowest set bit: i -= i & (-i), until i = 0)*
3. How does the update path traverse? *(By repeatedly adding the lowest set bit: i += i & (-i), until i > n)*
4. How many elements does B[i] cover? *(lowbit(i) elements)*
5. What's the maximum number of steps in a query? *(⌊log₂ n⌋ + 1, which is the maximum number of set bits)*
6. Why do odd indices always cover exactly 1 element? *(Odd numbers have lowbit = 1, so the range is [i, i])*

---

[← Previous: What is a BIT](01-what-is-a-bit.md) | [Next: Point Update →](03-point-update.md)
