# Chapter 5.3: Two Numbers Appearing Once

> **Unit 5 — XOR Applications**
> Every element appears twice except TWO elements. Find both in O(n) time, O(1) space.

---

## Overview

This is an extension of the Single Number problem. Given an array where every element appears **exactly twice** except **two** elements that appear once, find both unique elements. Simple XOR gives us `a ^ b` — but we need to **separate** a and b.

---

## The Trick: Divide by Differentiating Bit

```
  ┌────────────────────────────────────────────────────────────┐
  │  Step 1: XOR all elements → get xor = a ^ b               │
  │  Step 2: Find any set bit in xor (a and b DIFFER here)     │
  │  Step 3: Partition all elements by that bit                │
  │  Step 4: XOR each group separately → get a and b           │
  └────────────────────────────────────────────────────────────┘
```

### Why does this work?

```
  Since a ≠ b, they differ in at least one bit position.
  
  If we partition by that bit:
    - a goes into one group
    - b goes into the other group
    - Each paired element goes into ONE group (both copies have same bit)
    - Within each group, pairs cancel → only the unique survives!
```

---

## Step-by-Step Trace

### Example: [2, 4, 6, 8, 10, 4, 6, 8]

Unique elements: 2 and 10.

```
  Step 1: XOR all
  2 ^ 4 ^ 6 ^ 8 ^ 10 ^ 4 ^ 6 ^ 8
  = 2 ^ 10 ^ (4^4) ^ (6^6) ^ (8^8)
  = 2 ^ 10
  = 0010 ^ 1010
  = 1000 = 8

  Step 2: Find a set bit in 8 = 1000
  Rightmost set bit: bit 3  (using xor & (-xor) = 1000 & 1000 = 1000)
  diff_bit = 8

  Step 3: Partition by bit 3

  Group A (bit 3 = 1):     Group B (bit 3 = 0):
    8  = 1000                 2  = 0010
    8  = 1000                 4  = 0100
    10 = 1010                 4  = 0100
                              6  = 0110
                              6  = 0110

  Step 4: XOR each group
  Group A: 8 ^ 8 ^ 10 = 10   ✓
  Group B: 2 ^ 4 ^ 4 ^ 6 ^ 6 = 2   ✓

  Answer: {2, 10}
```

---

## Pseudocode

```
FUNCTION twoSingleNumbers(nums[]):
    // Step 1: XOR all to get a ^ b
    xor = 0
    FOR each num in nums:
        xor = xor ^ num
    
    // Step 2: Find rightmost set bit (any differing bit works)
    diff_bit = xor & (-xor)
    
    // Step 3 & 4: Partition and XOR each group
    a = 0
    b = 0
    FOR each num in nums:
        IF num & diff_bit != 0:
            a = a ^ num
        ELSE:
            b = b ^ num
    
    RETURN (a, b)
```

---

## Why the Partition Works

```
  ┌─────────────────────────────────────────────────────────┐
  │  For any paired element p (appears twice):              │
  │    Both copies have the SAME bit k → both go to same   │
  │    group → they cancel: p ^ p = 0                      │
  │                                                         │
  │  For unique elements a and b:                           │
  │    They DIFFER at bit k (because xor has bit k set)     │
  │    → a goes to one group, b to the other               │
  │    → each group has exactly one unique element          │
  └─────────────────────────────────────────────────────────┘
```

---

## Complexity

| | Time | Space |
|---|------|-------|
| This approach | O(n) — two passes | O(1) |
| HashMap | O(n) | O(n) |
| Sorting | O(n log n) | O(1) |

---

## Edge Cases

| Case | Handling |
|------|----------|
| a and b differ in only 1 bit | Works fine — that's the diff_bit |
| a and b differ in all bits | Any set bit of xor works for partitioning |
| Array has only 2 elements | [a, b] → xor = a^b, partition separates them |
| Elements include 0 | No issue — 0 has specific bit pattern |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| XOR all elements | Gives a ^ b (unique pair XOR) |
| Find differentiating bit | xor & (-xor) isolates rightmost set bit |
| Partition by that bit | Separates a and b into different groups |
| XOR each group | Pairs cancel; unique element survives in each |
| Two passes total | Both O(n), O(1) extra space |

---

## Quick Revision Questions

1. **In [1, 2, 3, 1], find the two unique numbers.**
   <details><summary>Answer</summary>XOR all: 1^2^3^1 = 2^3 = 01^11 = 10 = 2. diff_bit = 2 (bit 1). Group A (bit 1 set): {2,3} → 2^3... Wait, repartition: 2=10 (bit1=1), 3=11 (bit1=1), 1=01 (bit1=0). Group A: 2^3 = 1? No. Let me redo. XOR = 2^3 = 01. diff_bit = 01 (bit 0). Group A (bit 0 set): {1,1,3} → 1^1^3 = 3. Group B (bit 0 clear): {2} → 2. Answer: {2, 3}.</details>

2. **Why do we use xor & (-xor) specifically?**
   <details><summary>Answer</summary>It isolates the rightmost set bit, giving us a single-bit mask. Any set bit of xor works — rightmost is just the easiest to extract.</details>

3. **Can this be extended to three unique numbers?**
   <details><summary>Answer</summary>Not directly with this technique. Three unique numbers require more complex approaches (e.g., using two differentiating bits and 3-way partitioning).</details>

4. **What if a = 5 and b = 3? What is the diff_bit?**
   <details><summary>Answer</summary>5^3 = 101^011 = 110. diff_bit = 110 & 010 = 010 (bit 1). We'd partition by bit 1.</details>

5. **How many passes through the array does this require?**
   <details><summary>Answer</summary>Two passes: first to compute the total XOR, second to partition and XOR each group. Both are O(n).</details>

---

[← Previous: Single Number](02-single-number.md) | [Next: Find Missing Number →](04-find-missing-number.md)
