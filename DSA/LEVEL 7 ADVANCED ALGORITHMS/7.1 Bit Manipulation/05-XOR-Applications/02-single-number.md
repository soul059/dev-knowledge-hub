# Chapter 5.2: Single Number (LeetCode #136)

> **Unit 5 — XOR Applications**
> Every element appears twice except one. Find it in O(n) time, O(1) space.

---

## Overview

This is the classic "find the unique element" problem. Given an array where every element appears **exactly twice** except for one element that appears **once**, find that single element.

---

## The XOR Solution

```
  ┌──────────────────────────────────────────────────────────┐
  │  XOR all elements together.                              │
  │                                                          │
  │  Pairs cancel:    a ^ a = 0                              │
  │  Unique survives: 0 ^ x = x                             │
  │                                                          │
  │  Result = the single number                              │
  └──────────────────────────────────────────────────────────┘
```

---

## Pseudocode

```
FUNCTION singleNumber(nums[]):
    result = 0
    FOR each num in nums:
        result = result ^ num
    RETURN result
```

---

## Step-by-Step Trace

### Example: [4, 1, 2, 1, 2]

```
  Start: result = 0

  Step 1: result = 0 ^ 4 = 4
             0000
           ^ 0100
           ──────
             0100 (4)

  Step 2: result = 4 ^ 1 = 5
             0100
           ^ 0001
           ──────
             0101 (5)

  Step 3: result = 5 ^ 2 = 7
             0101
           ^ 0010
           ──────
             0111 (7)

  Step 4: result = 7 ^ 1 = 6
             0111
           ^ 0001
           ──────
             0110 (6)

  Step 5: result = 6 ^ 2 = 4
             0110
           ^ 0010
           ──────
             0100 (4)

  Answer: 4 ✓
```

### Verification by rearranging:

```
  4 ^ 1 ^ 2 ^ 1 ^ 2
  = 4 ^ (1 ^ 1) ^ (2 ^ 2)    (commutativity + associativity)
  = 4 ^ 0 ^ 0
  = 4  ✓
```

---

## Why Other Approaches Are Worse

| Approach | Time | Space | Problem |
|----------|------|-------|---------|
| **XOR** | **O(n)** | **O(1)** | **Optimal** |
| HashMap (count) | O(n) | O(n) | Extra memory |
| Sorting | O(n log n) | O(1)* | Slow |
| Math (2·sum_set - sum_all) | O(n) | O(n) | Needs set |

---

## Variations

### Array with negatives

```
  [-1, 2, -1, 3, 2]
  XOR: (-1) ^ 2 ^ (-1) ^ 3 ^ 2
     = (-1 ^ -1) ^ (2 ^ 2) ^ 3
     = 0 ^ 0 ^ 3
     = 3  ✓

  XOR works on signed integers too!
```

### Array with zeros

```
  [0, 5, 0, 3, 5]
  XOR: 0 ^ 5 ^ 0 ^ 3 ^ 5
     = (0^0) ^ (5^5) ^ 3
     = 3  ✓

  Zero is XOR's identity, not a special case.
```

---

## Edge Cases

| Case | Input | Output |
|------|-------|--------|
| Single element | [42] | 42 |
| All same except one | [7, 7, 7, 7, 3] | — Not valid (7 appears 4 times = even, but 3 appears once ✓) → 3 |
| Large array | [1..10⁵ with pairs] | O(n) XOR |
| Zero is the unique | [5, 5, 0] | 0 |

---

## Implementation Notes

```
  Language-specific reduce/fold:
  
  Python:  from functools import reduce; reduce(lambda a,b: a^b, nums)
  Java:    Arrays.stream(nums).reduce(0, (a,b) -> a^b)
  C++:     accumulate(nums.begin(), nums.end(), 0, bit_xor<>())
  JS:      nums.reduce((a, b) => a ^ b, 0)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Core trick | XOR all elements; pairs cancel |
| Time complexity | O(n) — single pass |
| Space complexity | O(1) — just one variable |
| Requirement | Each duplicate appears exactly 2 times |
| Works with negatives | Yes, XOR operates on bit patterns |

---

## Quick Revision Questions

1. **Find the single number in [7, 3, 5, 3, 7].**
   <details><summary>Answer</summary>7^3^5^3^7 = (7^7)^(3^3)^5 = 0^0^5 = 5</details>

2. **What if two elements appear once? Does this still work?**
   <details><summary>Answer</summary>No. You'd get a^b (the XOR of the two unique elements), not either one individually. You need a different technique (see Chapter 5.3).</details>

3. **Can this find the element appearing 3 times when others appear 2 times?**
   <details><summary>Answer</summary>Yes! 3 is odd, 2 is even. XOR cancels even-count elements, odd-count survives. a^a^a = a.</details>

4. **What if ALL elements appear 3 times except one appears once?**
   <details><summary>Answer</summary>Simple XOR won't work because 3 is also odd. You need bit-counting mod 3 for each bit position (different algorithm).</details>

5. **Why is the mathematical approach (2×sum_set − sum_all) inferior?**
   <details><summary>Answer</summary>It requires O(n) extra space for the set, and can suffer from integer overflow with large values. XOR avoids both issues.</details>

---

[← Previous: XOR Properties](01-xor-properties.md) | [Next: Two Numbers Appearing Once →](03-two-numbers-appearing-once.md)
