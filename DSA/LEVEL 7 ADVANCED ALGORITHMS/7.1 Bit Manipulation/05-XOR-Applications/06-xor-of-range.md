# Chapter 5.6: XOR of a Range [L..R]

> **Unit 5 — XOR Applications**
> Compute XOR of all integers from L to R in O(1) time using the mod-4 pattern.

---

## Overview

Computing L ^ (L+1) ^ (L+2) ^ ... ^ R naively takes O(R - L) time. But using the cyclic pattern of XOR(0..n), we can do it in O(1).

---

## XOR(0..n) Pattern

```
  XOR of 0 to n depends only on n % 4:

  n % 4 │ XOR(0..n) │ Example
  ──────┼───────────┼────────────────────────────
    0   │    n      │ XOR(0..4) = 0^1^2^3^4 = 4
    1   │    1      │ XOR(0..5) = ...^5 = 1
    2   │   n + 1   │ XOR(0..6) = ...^6 = 7
    3   │    0      │ XOR(0..7) = ...^7 = 0
```

### Why the Pattern Repeats

```
  Consider 4 consecutive numbers at positions 4k, 4k+1, 4k+2, 4k+3:

  In binary, the lowest 2 bits cycle: 00, 01, 10, 11
  XOR of these lowest 2 bits: 00 ^ 01 ^ 10 ^ 11 = 00

  Every group of 4 consecutive numbers XORs to 0!
  So XOR(0..n) only depends on the "remainder" numbers after
  the last complete group of 4.
```

---

## XOR(L..R) Formula

```
  ┌──────────────────────────────────────────────────┐
  │  XOR(L..R) = XOR(0..R) ^ XOR(0..L-1)            │
  │                                                   │
  │  Because:                                         │
  │  XOR(0..R) = XOR(0..L-1) ^ XOR(L..R)            │
  │  → XOR(L..R) = XOR(0..R) ^ XOR(0..L-1)         │
  └──────────────────────────────────────────────────┘
```

---

## Pseudocode

```
FUNCTION xorUpTo(n):
    SWITCH (n % 4):
        CASE 0: RETURN n
        CASE 1: RETURN 1
        CASE 2: RETURN n + 1
        CASE 3: RETURN 0

FUNCTION xorRange(L, R):
    IF L == 0:
        RETURN xorUpTo(R)
    RETURN xorUpTo(R) ^ xorUpTo(L - 1)
```

---

## Step-by-Step Traces

### Example 1: XOR(3..7)

```
  XOR(0..7): 7 % 4 = 3 → 0
  XOR(0..2): 2 % 4 = 2 → 2 + 1 = 3

  XOR(3..7) = 0 ^ 3 = 3

  Verify: 3^4^5^6^7
  = 011 ^ 100 ^ 101 ^ 110 ^ 111
  = 011 ^ (100^101) ^ (110^111)
  = 011 ^ 001 ^ 001
  = 011 ^ 000
  = 011 = 3  ✓
```

### Example 2: XOR(5..9)

```
  XOR(0..9): 9 % 4 = 1 → 1
  XOR(0..4): 4 % 4 = 0 → 4

  XOR(5..9) = 1 ^ 4 = 5

  Verify: 5^6^7^8^9
  = 0101 ^ 0110 ^ 0111 ^ 1000 ^ 1001
  = (0101^0110) ^ (0111^1000) ^ 1001
  = 0011 ^ 1111 ^ 1001
  = 1100 ^ 1001
  = 0101 = 5  ✓
```

### Example 3: XOR(1..1)

```
  XOR(0..1): 1 % 4 = 1 → 1
  XOR(0..0): 0 % 4 = 0 → 0

  XOR(1..1) = 1 ^ 0 = 1  ✓  (single element)
```

---

## Complexity

| Method | Time | Space |
|--------|------|-------|
| Formula (n%4) | **O(1)** | O(1) |
| Naive loop | O(R - L) | O(1) |

---

## Applications

- **Competitive programming** — range XOR queries
- **Prefix XOR arrays** — like prefix sums but for XOR
- **Find missing number in range** — XOR expected range with actual elements
- **Segment trees** — XOR as a mergeable operation

---

## Prefix XOR Array

```
  Similar to prefix sums, but using XOR:

  arr    = [3, 5, 2, 7, 4]
  prefix = [0, 3, 6, 4, 3, 7]
           └──────────────────── prefix[0] = 0
               └────────────── prefix[1] = 0^3 = 3
                  └──────────── prefix[2] = 3^5 = 6
                     └────────── prefix[3] = 6^2 = 4
                        └──────── prefix[4] = 4^7 = 3
                           └──── prefix[5] = 3^4 = 7

  XOR of arr[L..R] = prefix[R+1] ^ prefix[L]

  Example: XOR(arr[1..3]) = prefix[4] ^ prefix[1] = 3 ^ 3 = 0
  Verify: 5^2^7 = 0  ✓
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| XOR(0..n) pattern | Depends on n%4: {n, 1, n+1, 0} |
| Range XOR formula | XOR(L..R) = XOR(0..R) ^ XOR(0..L-1) |
| Complexity | O(1) time and space |
| Prefix XOR | Enables O(1) range XOR queries on arrays |

---

## Quick Revision Questions

1. **What is XOR(0..12)?**
   <details><summary>Answer</summary>12 % 4 = 0, so XOR(0..12) = 12.</details>

2. **What is XOR(10..15)?**
   <details><summary>Answer</summary>XOR(0..15) = 15%4=3 → 0. XOR(0..9) = 9%4=1 → 1. XOR(10..15) = 0^1 = 1.</details>

3. **Why does every group of 4 consecutive integers XOR to 0?**
   <details><summary>Answer</summary>The last 2 bits cycle through 00,01,10,11. XOR of these 4 patterns = 00. Higher bits are all the same across the group (or differ by a complete cycle), so they also cancel.</details>

4. **How would you compute XOR of all even numbers from 2 to 100?**
   <details><summary>Answer</summary>Even numbers: 2,4,...,100 = 2×(1,2,...,50). Factor out: each is 2×k. For even numbers 2k, XOR is 2×XOR(1..50) only if... Actually this doesn't simplify cleanly. Best to note 2k = k<<1, so XOR of all (k<<1) for k=1..50 = (XOR(1..50))<<1 = (50%4=2 → 51)<<1 = 102.</details>

5. **Build a prefix XOR for [1, 2, 3] and find XOR(arr[0..2]).**
   <details><summary>Answer</summary>prefix = [0, 1, 3, 0]. XOR(arr[0..2]) = prefix[3]^prefix[0] = 0^0 = 0. Verify: 1^2^3 = 0 ✓.</details>

---

[← Previous: Find Duplicate Number](05-find-duplicate-number.md) | [Next: Unit 6 — Generate All Subsets →](../06-Subsets-With-Bits/01-generate-all-subsets.md)
