# Chapter 4.2: Find the Odd Occurring Element

> **Unit 4 — Bit Manipulation Tricks**
> Every element appears an even number of times except one — find it in O(n) time, O(1) space.

---

## Overview

Given an array where every element appears an **even number of times** except one element that appears an **odd number of times**, find that element. XOR's cancellation property makes this trivially elegant.

---

## The Trick

```
  ┌────────────────────────────────────────────────────┐
  │  XOR all elements together.                       │
  │                                                    │
  │  Even-count elements cancel: a ^ a = 0            │
  │  Odd-count element survives: 0 ^ x = x            │
  └────────────────────────────────────────────────────┘
```

## Step-by-Step Trace

```
  Array: [4, 3, 4, 3, 4, 7, 7]

  XOR all:
  4 ^ 3 ^ 4 ^ 3 ^ 4 ^ 7 ^ 7
  
  Rearrange (commutative + associative):
  = (3 ^ 3) ^ (7 ^ 7) ^ (4 ^ 4 ^ 4)
  =    0    ^    0    ^ (0 ^ 4)
  =    0    ^    0    ^   4
  = 4

  Answer: 4 (appears 3 times = odd) ✓
```

---

## Pseudocode

```
FUNCTION findOddOccurring(arr[]):
    result = 0
    FOR each element x in arr:
        result = result ^ x
    RETURN result
```

---

## Complexity

| | Time | Space |
|---|------|-------|
| XOR approach | O(n) | O(1) |
| HashMap approach | O(n) | O(n) |
| Sorting approach | O(n log n) | O(1) |

> XOR is optimal — linear time with constant space.

---

## Applications

- **Single Number** (LeetCode #136) — every element appears twice except one
- **Error detection** — find corrupted packet in duplicate stream
- **Missing element** — combine with XOR of expected range

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Core idea | XOR all elements; pairs cancel to 0 |
| Why it works | `a^a = 0` and `0^x = x` |
| Time/Space | O(n) time, O(1) space |
| Requirement | Exactly one element has odd frequency |

---

## Quick Revision Questions

1. **Find the odd-occurring element in [1, 2, 1, 2, 3, 3, 5].**
   <details><summary>Answer</summary>XOR all: (1^1) ^ (2^2) ^ (3^3) ^ 5 = 0 ^ 0 ^ 0 ^ 5 = 5</details>

2. **Does this work if the odd element appears 5 times?**
   <details><summary>Answer</summary>Yes. 5 is odd, so a^a^a^a^a = a (since pairs cancel, one remains).</details>

3. **What if TWO elements appear odd times?**
   <details><summary>Answer</summary>Simple XOR gives a^b (they both survive). You need a different technique (see Unit 5: XOR Applications).</details>

4. **Why can't we use this for elements appearing 3 times, others once?**
   <details><summary>Answer</summary>Both 1 and 3 are odd. All elements survive XOR. The trick only works when all but one have even frequency.</details>

5. **What is the result of XOR-ing all elements if ALL appear even times?**
   <details><summary>Answer</summary>0 — everything cancels out.</details>

---

[← Previous: Swap Without Temp](01-swap-without-temp.md) | [Next: Check Power of 4 →](03-check-power-of-4.md)
