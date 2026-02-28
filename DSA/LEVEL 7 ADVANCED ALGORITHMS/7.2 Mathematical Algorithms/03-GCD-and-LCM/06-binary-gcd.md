# Chapter 6: Binary GCD (Stein's Algorithm)

[← Previous: Properties and Theorems](05-properties-and-theorems.md) | [Back to README](../README.md) | [Next: Unit 4 — Modular Addition →](../04-Modular-Arithmetic/01-modular-addition.md)

---

## Overview

The Binary GCD (Stein's Algorithm) computes GCD using only subtraction, comparison, and bit shifts — avoiding expensive division/modulo operations. It is especially efficient on hardware where division is slow.

---

## 6.1 Motivation

```
  Standard Euclidean: uses mod (division) → expensive on some hardware
  Binary GCD: uses ONLY:
    ✓ Subtraction
    ✓ Comparison
    ✓ Right shift (÷2)
    ✓ Parity check (x & 1)

  Modern CPUs: division ≈ 20-90 cycles, shift ≈ 1 cycle
  Although compilers optimize well, binary GCD can still be
  faster for large arbitrary-precision integers (BigInt).
```

---

## 6.2 Key Observations

```
  ┌──────────────────────────────────────────────────────────┐
  │  FOUR RULES:                                             │
  │                                                          │
  │  1. gcd(0, b) = b  and  gcd(a, 0) = a                  │
  │                                                          │
  │  2. Both even: gcd(2a, 2b) = 2 × gcd(a, b)             │
  │     (factor out common 2)                                │
  │                                                          │
  │  3. One even, one odd: gcd(2a, b) = gcd(a, b) if b odd  │
  │     (2 is not a common factor)                           │
  │                                                          │
  │  4. Both odd: gcd(a, b) = gcd(|a-b|/2, min(a,b))       │
  │     (difference is even, so divide by 2)                 │
  └──────────────────────────────────────────────────────────┘
```

---

## 6.3 Algorithm

```
FUNCTION binaryGCD(a, b):
    IF a == 0: RETURN b
    IF b == 0: RETURN a

    // Count common factors of 2
    shift = 0
    WHILE (a | b) is even:      // both even
        a = a >> 1
        b = b >> 1
        shift = shift + 1

    // Remove remaining factors of 2 from a
    WHILE a is even:
        a = a >> 1

    // Main loop
    REPEAT:
        // Remove factors of 2 from b
        WHILE b is even:
            b = b >> 1
        // Now both a and b are odd
        IF a > b:
            SWAP(a, b)
        b = b - a           // b becomes even (odd - odd = even)
    UNTIL b == 0

    RETURN a << shift        // restore common factors of 2

Time: O(log(a) × log(b))  worst case
      O((log max(a,b))²)
Space: O(1)
```

---

## 6.4 Optimized Version Using CTZ

```
  CTZ = Count Trailing Zeros = number of trailing 0-bits
  CTZ is a single CPU instruction on modern hardware!

FUNCTION binaryGCD_optimized(a, b):
    IF a == 0: RETURN b
    IF b == 0: RETURN a

    az = ctz(a)     // trailing zeros of a
    bz = ctz(b)     // trailing zeros of b
    shift = min(az, bz)
    a = a >> az      // remove all factors of 2 from a
    
    REPEAT:
        b = b >> ctz(b)    // remove all factors of 2 from b
        IF a > b:
            SWAP(a, b)
        b = b - a
    UNTIL b == 0

    RETURN a << shift
```

---

## 6.5 Detailed Trace: binaryGCD(48, 36)

```
  a = 48 = 110000₂
  b = 36 = 100100₂

  Step 1: Factor out common 2s
  Both even:  a=24, b=18, shift=1
  Both even:  a=12, b=9,  shift=2
  Now b is odd → stop common factoring

  Step 2: Remove remaining 2s from a
  a=12 (even) → a=6 (even) → a=3 (odd)
  Now a=3, b=9

  Step 3: Main loop
  ┌─────┬──────┬──────┬─────────────────────────┐
  │ Iter│  a   │  b   │ Action                   │
  ├─────┼──────┼──────┼─────────────────────────┤
  │  1  │  3   │  9   │ Both odd                 │
  │     │      │      │ a ≤ b → b = 9-3 = 6     │
  │     │      │  6   │ b even → b = 3           │
  │  2  │  3   │  3   │ Both odd                 │
  │     │      │      │ a ≤ b → b = 3-3 = 0     │
  │     │      │  0   │ b = 0 → STOP             │
  └─────┴──────┴──────┴─────────────────────────┘

  Result: a << shift = 3 << 2 = 12

  GCD(48, 36) = 12 ✓
```

---

## 6.6 Trace: binaryGCD(105, 63)

```
  a = 105 = 1101001₂
  b =  63 = 0111111₂

  Step 1: Common 2s
  105 is odd, 63 is odd → shift = 0

  Step 2: a is already odd. Skip.

  Step 3: Main loop
  ┌─────┬──────┬──────┬─────────────────────────┐
  │ Iter│  a   │  b   │ Action                   │
  ├─────┼──────┼──────┼─────────────────────────┤
  │  1  │ 63   │ 105  │ a>b? No,swap a↔b:63,105│
  │     │      │      │ wait, 63<105 so no swap │
  │     │      │      │ b=105-63=42             │
  │     │      │  42  │ b even→21               │
  │  2  │ 21   │  63  │ wait, a=63,b=21? Let me │
  │     │      │      │ redo: a=63,b=21         │
  │     │      │      │ a>b→ swap: a=21,b=63    │
  │     │      │      │ b=63-21=42              │
  │     │      │  42  │ b even→21               │
  │  3  │ 21   │  21  │ b=21-21=0               │
  │     │      │  0   │ STOP                     │
  └─────┴──────┴──────┴─────────────────────────┘

  Let me redo cleanly:
  a=105, b=63, shift=0 (both odd)

  Iter 1: remove 2s from b: b=63 (already odd)
          a=105 > b=63 → swap: a=63, b=105
          b = 105 - 63 = 42
  
  Iter 2: remove 2s from b: 42→21
          a=63 > b=21 → swap: a=21, b=63
          b = 63 - 21 = 42

  Iter 3: remove 2s from b: 42→21
          a=21 ≤ b=21 → no swap
          b = 21 - 21 = 0

  b=0 → STOP. Result = 21 << 0 = 21

  GCD(105, 63) = 21 ✓
```

---

## 6.7 Comparison: Euclidean vs Binary GCD

```
  ┌────────────────────┬───────────────────┬─────────────────┐
  │ Aspect             │ Euclidean         │ Binary GCD      │
  ├────────────────────┼───────────────────┼─────────────────┤
  │ Operations         │ Division / Modulo │ Shift, Sub, Cmp │
  │ Time complexity    │ O(log min(a,b))   │ O(log²(max))    │
  │ # Iterations       │ ≤ log_φ(max)      │ ≤ 2×log₂(max)   │
  │ Hardware-friendly  │ Needs divider     │ Shift-only      │
  │ BigInt performance │ O(n²) per step    │ O(n) per step   │
  │ Practical speed    │ Faster (compiled) │ Faster (BigInt) │
  │ Code simplicity    │ Very simple       │ More complex    │
  └────────────────────┴───────────────────┴─────────────────┘

  n = number of digits in the input.
  For fixed-size integers: Euclidean is typically faster.
  For arbitrary-precision: Binary GCD wins (no expensive division).
```

---

## 6.8 Bit Manipulation Recap

```
  Operations used in Binary GCD:

  x & 1         → check if odd (last bit)
  x >> 1        → divide by 2
  x << k        → multiply by 2^k
  x | y         → bitwise OR
  ctz(x)        → count trailing zeros
                   __builtin_ctz(x) in C/C++
                   Integer.numberOfTrailingZeros(x) in Java

  Example:
  48 = 110000₂  → ctz(48) = 4 (four trailing zeros)
  36 = 100100₂  → ctz(36) = 2
```

---

## 6.9 When to Use Binary GCD

| Scenario | Recommendation |
|----------|---------------|
| Standard competitive programming | Use Euclidean (simpler) |
| Big integer libraries | Binary GCD (no division) |
| Embedded systems without divider | Binary GCD |
| Multiple-precision arithmetic | Binary GCD |
| Interview problems | Know both, use Euclidean |

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Core principle | Replace division with shifts and subtraction |
| Rule: both even | gcd(2a, 2b) = 2×gcd(a, b) |
| Rule: one even | gcd(2a, b_odd) = gcd(a, b_odd) |
| Rule: both odd | gcd(a, b) = gcd(\|a-b\|/2, min(a,b)) |
| Time complexity | O(log²(max(a,b))) |
| Best for | Arbitrary-precision integers |
| Hardware ops | shift, subtract, compare, ctz |

---

## Quick Revision Questions

1. **List the four rules** that Binary GCD is based on.
2. **trace binaryGCD(24, 18)** step by step.
3. **Why is odd minus odd always even?** (Think in terms of parity.)
4. **When is Binary GCD preferred** over Euclidean?
5. **What CPU instruction** counts trailing zeros?
6. **What is the time complexity** of Binary GCD, and why is it O(log²)?

---

[← Previous: Properties and Theorems](05-properties-and-theorems.md) | [Back to README](../README.md) | [Next: Unit 4 — Modular Addition →](../04-Modular-Arithmetic/01-modular-addition.md)
