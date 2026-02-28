# Chapter 3.1: Check if Bit is Set

> **Unit 3 — Basic Bit Techniques**
> The most fundamental bit operation — testing whether a specific bit is ON or OFF.

---

## Overview

Checking if a particular bit is set (equals 1) is the gateway technique for all bit manipulation. It's used everywhere: permission checks, feature flags, bitmap operations, and as a building block for more complex algorithms.

---

## The Core Idea

```
  To check if bit at position k is set in number n:
  
  ┌───────────────────────────────────────────────────────┐
  │                                                       │
  │  METHOD 1:  (n >> k) & 1                              │
  │  → Shift bit k to position 0, then mask it           │
  │                                                       │
  │  METHOD 2:  n & (1 << k)                              │
  │  → Create mask with only bit k, AND with n           │
  │  → Returns 0 (not set) or 2ᵏ (set)                   │
  │                                                       │
  └───────────────────────────────────────────────────────┘
```

---

## Method 1: Shift and Mask

### Pseudocode

```
FUNCTION isBitSet(n, k):
    RETURN (n >> k) & 1    // Returns 0 or 1
```

### Step-by-Step Trace: Is bit 3 set in 42?

```
  n = 42 = 00101010
  k = 3
  
  Step 1: Shift right by 3
    42 >> 3 = 00000101
    
    00101010
          ↘↘↘   (bits 0,1,2 fall off)
    00000101     (bit 3 is now at position 0)
  
  Step 2: AND with 1
    00000101
  & 00000001
  ──────────
    00000001 = 1
    
  Result: 1 → bit 3 IS set ✓
  
  Verify: 42 = 00101010
                   ↑
                 bit 3 = 1 ✓
```

### Another Trace: Is bit 2 set in 42?

```
  n = 42 = 00101010
  k = 2
  
  Step 1: 42 >> 2 = 00001010
  Step 2: 00001010 & 00000001 = 0
  
  Result: 0 → bit 2 is NOT set ✓
  
  Verify: 42 = 00101010
                    ↑
                  bit 2 = 0 ✓
```

---

## Method 2: Mask and AND

### Pseudocode

```
FUNCTION isBitSet(n, k):
    mask = 1 << k
    RETURN (n & mask) != 0    // Returns true or false
```

### Step-by-Step Trace: Is bit 5 set in 42?

```
  n = 42 = 00101010
  k = 5
  
  Step 1: Create mask
    1 << 5 = 00100000
  
  Step 2: AND with n
    00101010
  & 00100000
  ──────────
    00100000  ≠ 0 → bit IS set ✓

  The result is nonzero (specifically 32 = 2⁵)
```

### Visual: The Mask as a Window

```
  Number:  0  0  1  0  1  0  1  0     (42)
           ─  ─  ─  ─  ─  ─  ─  ─
  Mask:    0  0  1  0  0  0  0  0     (1 << 5)
                 ↑
           Only this position matters
           
  AND:     0  0  1  0  0  0  0  0     = 32 (nonzero → bit is set)
  
  If bit 5 were 0:
  Number:  0  0  0  0  1  0  1  0     (10)
  Mask:    0  0  1  0  0  0  0  0     (1 << 5)
  AND:     0  0  0  0  0  0  0  0     = 0 (zero → bit not set)
```

---

## Comparing Both Methods

```
  ┌────────────────────────────────────────────────────┐
  │  METHOD        │ RETURNS    │ USE WHEN             │
  ├────────────────┼────────────┼──────────────────────┤
  │ (n >> k) & 1   │ 0 or 1     │ Need exact 0/1 value │
  │ n & (1 << k)   │ 0 or 2ᵏ   │ Only need true/false │
  └────────────────┴────────────┴──────────────────────┘
  
  Both are O(1) and equally fast.
  Method 1 is preferred when you need a clean 0/1 result.
```

---

## Checking Multiple Bits

### Check if ANY of several bits are set

```
  mask = (1 << 2) | (1 << 5) | (1 << 7);  // bits 2, 5, 7
  
  if (n & mask) != 0:
      // At least one of bits 2, 5, 7 is set
```

### Check if ALL of several bits are set

```
  mask = (1 << 2) | (1 << 5) | (1 << 7);
  
  if (n & mask) == mask:
      // ALL of bits 2, 5, 7 are set
```

### Trace: Permission Check

```
  READ  = 1 << 0 = 001
  WRITE = 1 << 1 = 010
  EXEC  = 1 << 2 = 100
  
  user_perms = 0b101 = 5  (read + execute)
  
  Check read access:
    5 & READ = 101 & 001 = 001 ≠ 0 → Has read ✓
  
  Check write access:
    5 & WRITE = 101 & 010 = 000 = 0 → No write ✓
  
  Check read AND execute:
    mask = READ | EXEC = 101
    5 & mask = 101 & 101 = 101 = mask → Has both ✓
```

---

## Iterating Over All Set Bits

```
FUNCTION printSetBits(n):
    FOR k = 0 TO 31:
        IF (n >> k) & 1:
            PRINT "Bit", k, "is set"

Trace: n = 42 (00101010)

  k=0: (42>>0)&1 = 0
  k=1: (42>>1)&1 = 1  → Bit 1 is set
  k=2: (42>>2)&1 = 0
  k=3: (42>>3)&1 = 1  → Bit 3 is set
  k=4: (42>>4)&1 = 0
  k=5: (42>>5)&1 = 1  → Bit 5 is set
  k=6+: 0

  Set bits: {1, 3, 5}
  Verify: 2¹ + 2³ + 2⁵ = 2 + 8 + 32 = 42 ✓
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Check single bit | O(1) | O(1) |
| Check all n bits | O(n) | O(1) |
| Check against multi-bit mask | O(1) | O(1) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Method 1 | `(n >> k) & 1` → returns 0 or 1 |
| Method 2 | `n & (1 << k)` → returns 0 or 2ᵏ |
| Multiple bits (ANY) | `(n & mask) != 0` |
| Multiple bits (ALL) | `(n & mask) == mask` |
| Iterate set bits | Loop k from 0 to 31, check each |

---

## Quick Revision Questions

1. **Is bit 4 set in the number 26?**
   <details><summary>Answer</summary>26 = 11010₂. Bit 4 = 1. Yes, it's set. (26 >> 4) & 1 = 1.</details>

2. **What does `n & (1 << 3)` return if bit 3 of n is set?**
   <details><summary>Answer</summary>8 (which is 2³). It returns the mask value, not 1.</details>

3. **How do you check if ALL of bits 0, 1, and 2 are set?**
   <details><summary>Answer</summary>mask = 7 (0b111). Check: `(n & mask) == mask`</details>

4. **What is the time complexity of checking a single bit?**
   <details><summary>Answer</summary>O(1) — shift and AND are constant-time CPU operations</details>

5. **Why is Method 1 sometimes preferred over Method 2?**
   <details><summary>Answer</summary>Method 1 returns a clean 0 or 1, making it directly usable as an index or boolean without comparison.</details>

6. **In 42 = 00101010, list all positions that are set.**
   <details><summary>Answer</summary>Positions 1, 3, and 5 (from right, zero-indexed)</details>

---

[← Previous Unit: Right Shift](../02-Bitwise-Operators/06-right-shift.md) | [Next: Set a Bit →](02-set-a-bit.md)
