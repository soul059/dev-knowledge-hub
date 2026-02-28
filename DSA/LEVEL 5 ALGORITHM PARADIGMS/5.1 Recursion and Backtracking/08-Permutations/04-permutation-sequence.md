# Chapter 4: Permutation Sequence (K-th Permutation)

## Overview
Given $n$ and $k$, find the **k-th permutation** in lexicographic order **without generating all permutations**. This uses a clever **factorial number system** approach with O(n²) time instead of O(n! × n).

---

## Problem Statement

```
Input: n = 4, k = 14
Output: "3142"

All permutations of [1,2,3,4] in order:
  1: 1234    7: 2134   13: 3124   19: 4123
  2: 1243    8: 2143   14: 3142 ← 20: 4132
  3: 1324    9: 2314   15: 3214   21: 4213
  4: 1342   10: 2341   16: 3241   22: 4231
  5: 1423   11: 2413   17: 3412   23: 4312
  6: 1432   12: 2431   18: 3421   24: 4321
```

---

## Key Insight: Factorial Number System

```
For n = 4, total permutations = 4! = 24

Group by FIRST digit:
  1___ : positions 1-6   (3! = 6 permutations each)
  2___ : positions 7-12
  3___ : positions 13-18  ← k=14 falls here!
  4___ : positions 19-24

The first digit determines which GROUP of (n-1)! permutations.
digit_index = (k-1) / (n-1)!

After fixing the first digit, repeat for remaining positions.
```

---

## Algorithm

```
FUNCTION getPermutation(n, k):
    // Build list of available digits and factorials
    digits = [1, 2, 3, ..., n]
    factorials = precompute factorials [0!, 1!, 2!, ..., (n-1)!]
    
    k = k - 1                          ← Convert to 0-indexed
    result = ""
    
    FOR i = n DOWNTO 1:
        // Which group does k fall in?
        fact = factorials[i - 1]        ← (i-1)!
        index = k / fact                ← group index
        k = k % fact                    ← remainder for next level
        
        result += digits[index]         ← Pick the digit
        digits.remove(index)            ← Remove used digit
    
    RETURN result
```

---

## Worked Example: n=4, k=14

```
digits = [1, 2, 3, 4]
factorials = [1, 1, 2, 6]   ← [0!, 1!, 2!, 3!]
k = 14 - 1 = 13 (0-indexed)

ITERATION 1 (i=4): Pick 1st digit
  fact = 3! = 6
  index = 13 / 6 = 2        ← 3rd group (0-indexed)
  k = 13 % 6 = 1            ← position within group
  Pick digits[2] = 3
  digits = [1, 2, 4]
  result = "3"

ITERATION 2 (i=3): Pick 2nd digit
  fact = 2! = 2
  index = 1 / 2 = 0         ← 1st group
  k = 1 % 2 = 1             ← position within group
  Pick digits[0] = 1
  digits = [2, 4]
  result = "31"

ITERATION 3 (i=2): Pick 3rd digit
  fact = 1! = 1
  index = 1 / 1 = 1         ← 2nd group
  k = 1 % 1 = 0
  Pick digits[1] = 4
  digits = [2]
  result = "314"

ITERATION 4 (i=1): Pick 4th digit
  fact = 0! = 1
  index = 0 / 1 = 0
  k = 0 % 1 = 0
  Pick digits[0] = 2
  result = "3142"

Answer: "3142" ✓ (matches position 14 in the list)
```

---

## Visual Breakdown

```
n=4, k=14 (0-indexed: 13)

24 permutations divided into 4 groups of 6:
  Group 0 (1___):  k = 0-5
  Group 1 (2___):  k = 6-11
  Group 2 (3___):  k = 12-17  ← 13 falls here
  Group 3 (4___):  k = 18-23

Within Group 2 (3___), 6 perms divided into 3 groups of 2:
  Group 0 (31__):  k = 0-1   ← 1 falls here
  Group 1 (32__):  k = 2-3
  Group 2 (34__):  k = 4-5

Within Group 0 (31__), 2 perms divided into 2 groups of 1:
  Group 0 (312_):  k = 0
  Group 1 (314_):  k = 1     ← 1 falls here

Within Group 1 (314_), only one option:
  3142                        ← answer!
```

---

## Complexity

```
┌────────────────────────────────────────────┐
│ Time: O(n²)                                │
│   • n iterations                           │
│   • Each iteration: O(n) to remove digit   │
│   • Total: O(n²)                           │
│                                            │
│ Space: O(n)                                │
│   • digits list: O(n)                      │
│   • factorials: O(n)                       │
│                                            │
│ Compare with brute force:                  │
│   Generate all + find k-th: O(n! × n)      │
│   This algorithm: O(n²)                    │
│   For n=20: 10^18 vs 400 operations!       │
└────────────────────────────────────────────┘
```

---

## Summary Table

| Step | Operation | Example (n=4, k=14) |
|------|-----------|---------------------|
| Setup | k = k-1 (0-index) | k = 13 |
| Pick 1st | index = k/(n-1)! | 13/6 = 2 → pick '3' |
| Pick 2nd | index = k/(n-2)! | 1/2 = 0 → pick '1' |
| Pick 3rd | index = k/(n-3)! | 1/1 = 1 → pick '4' |
| Pick 4th | index = k/0! | 0/1 = 0 → pick '2' |
| Result | Concatenate | "3142" |

---

## Quick Revision Questions

1. **Why subtract 1** from k at the start?
2. **What is the factorial number system** and how does it apply?
3. **How many permutations** start with digit 3 when n=5?
4. **Find the 9th permutation** of [1, 2, 3, 4] using this algorithm.
5. **What is the time complexity** of this approach vs brute force?
6. **How would you find** the RANK of a given permutation (reverse problem)?

---

[← Previous: Next Permutation](03-next-permutation.md) | [Next: Beautiful Arrangement →](05-beautiful-arrangement.md)

[← Back to README](../README.md)
