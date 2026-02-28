# Chapter 2.4: NOT Operation (~)

> **Unit 2 — Bitwise Operators**
> The NOT operator — the universal bit flipper that inverts every single bit.

---

## Overview

The **bitwise NOT** (also called complement or inversion) is the only **unary** bitwise operator — it operates on a single number. NOT **flips every bit**: 0 becomes 1, and 1 becomes 0.

---

## Truth Table

```
  ┌─────┬─────────┐
  │  A  │  NOT A  │
  ├─────┼─────────┤
  │  0  │    1    │
  │  1  │    0    │
  └─────┴─────────┘

  INTUITION: NOT is a "total flip" — every bit reverses.
  
  Symbol: ~ (tilde) in most programming languages
```

### Logic Gate Diagram

```
       A ──── ▷○ ──── ~A

  NOT gate (inverter): bubble (○) indicates negation.
```

---

## Bitwise NOT on Multi-Bit Numbers

```
  Example:  ~42  (8-bit unsigned)

      42 = 0 0 1 0 1 0 1 0
           ─────────────────
     ~42 = 1 1 0 1 0 1 0 1  = 213 (unsigned)

  Every 0 → 1, every 1 → 0
```

### In Two's Complement (Signed)

```
  ~42 in 8-bit signed:
  
      42 = 00101010
     ~42 = 11010101 = -43  (in two's complement)

  ┌─────────────────────────────────────────────┐
  │  KEY RELATIONSHIP:                          │
  │                                             │
  │     ~n = -(n + 1)                           │
  │                                             │
  │  ~0  = -1                                   │
  │  ~1  = -2                                   │
  │  ~42 = -43                                  │
  │  ~(-1) = 0                                  │
  │  ~(-5) = 4                                  │
  └─────────────────────────────────────────────┘
```

### Why ~n = -(n+1)?

```
  Proof using two's complement:
  
  n + ~n = 11111111...1  (all 1s, because every position has one 0 and one 1)
         = -1  (in two's complement, all 1s = -1)
  
  Therefore:  ~n = -1 - n = -(n + 1)  ∎
  
  Example verification:
  n  = 42  = 00101010
  ~n = -43 = 11010101
  
  n + ~n = 00101010 + 11010101 = 11111111 = -1 ✓
```

---

## Visual: NOT as a Film Negative

```
  Original:    ██░░██░░██░░██░░
  NOT:         ░░██░░██░░██░░██
  
  Like a photographic negative — everything inverts!
  
  Number:     0  0  1  0  1  0  1  0
              ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓    FLIP ALL
  NOT:        1  1  0  1  0  1  0  1
```

---

## Common NOT Patterns

### 1. Creating Inverted Masks

```
  mask   = 00001000    (bit 3 set)
  ~mask  = 11110111    (all bits EXCEPT bit 3)
  
  Use case: Clear bit 3
  n & ~mask → clears bit 3, keeps everything else
  
  n       = 10101110
  ~mask   = 11110111
  n&~mask = 10100110    bit 3 cleared ✓
```

### 2. NOT + 1 = Negation (Two's Complement)

```
  -n = ~n + 1
  
  Negate 42:
  ~42     = 11010101
  ~42 + 1 = 11010110 = -42 ✓
  
  This IS the two's complement negation formula!
```

### 3. Find Minimum of n and ~n

```
  For any n:
  n + ~n = -1  (all 1s)
  
  So if n is large, ~n is small (and negative in signed).
  They always sum to -1.
```

### 4. NOT for Bit Counting

```
  Count of ZERO bits = total bits - count of ONE bits
  
  Or equivalently:
  count_zeros(n) = count_ones(~n)
```

---

## De Morgan's Laws with NOT

```
  ┌──────────────────────────────────────────────┐
  │  DE MORGAN'S LAWS (Fundamental!)             │
  │                                              │
  │  ~(a & b) = ~a | ~b                          │
  │  ~(a | b) = ~a & ~b                          │
  │                                              │
  │  "NOT of AND = OR of NOTs"                   │
  │  "NOT of OR  = AND of NOTs"                  │
  └──────────────────────────────────────────────┘

  Verification with truth table:
  
  a  b │ a&b  ~(a&b) │ ~a  ~b  ~a|~b │ Match?
  ─────┼─────────────┼───────────────┼───────
  0  0 │  0     1    │  1   1    1   │  ✓
  0  1 │  0     1    │  1   0    1   │  ✓
  1  0 │  0     1    │  0   1    1   │  ✓
  1  1 │  1     0    │  0   0    0   │  ✓
```

---

## NOT in Different Contexts

```
  ┌────────────────────────────────────────────────────┐
  │  CONTEXT-DEPENDENT BEHAVIOR OF NOT                │
  ├────────────────────────────────────────────────────┤
  │                                                    │
  │  8-bit unsigned:   ~42 = 213                       │
  │  8-bit signed:     ~42 = -43                       │
  │  32-bit signed:    ~42 = -43                       │
  │  32-bit unsigned:  ~42 = 4,294,967,253             │
  │                                                    │
  │  The bit pattern is the SAME (all bits flipped),   │
  │  but the INTERPRETATION depends on signed/unsigned │
  │  and bit width.                                    │
  └────────────────────────────────────────────────────┘
```

---

## NOT Bit Width Pitfall

```
  DANGER: NOT depends on the integer size!
  
  ~5 in 8-bit:   ~(00000101) = 11111010 = 250 (unsigned)
  ~5 in 16-bit:  ~(00000000 00000101) = 11111111 11111010 = 65530
  ~5 in 32-bit:  ~(00...00000101) = 11...11111010 = 4294967290
  
  The number of leading 1s changes with bit width!
  
  ┌─────────────────────────────────────────────┐
  │  When using NOT for masks, always consider  │
  │  the integer width of your platform.        │
  │                                             │
  │  To make width-independent masks:           │
  │  Use XOR with a known width mask instead    │
  │  0xFF ^ 0x08 = 0xF7  (8-bit safe)          │
  └─────────────────────────────────────────────┘
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| ~a | O(1) | O(1) |

> Single CPU instruction — all bits flip in parallel.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| NOT rule | Flips every bit (0→1, 1→0) |
| Formula | `~n = -(n+1)` in two's complement |
| Negation | `-n = ~n + 1` |
| Inverted mask | `~(1 << k)` clears bit k position |
| n + ~n | Always equals -1 (all ones) |
| De Morgan's | `~(a&b) = ~a\|~b` and `~(a\|b) = ~a&~b` |
| Width matters | NOT result depends on integer size |

---

## Quick Revision Questions

1. **What is ~0 in 8-bit signed?**
   <details><summary>Answer</summary>11111111 = -1 (all bits set = -1 in two's complement)</details>

2. **What is ~(-1) in any bit width?**
   <details><summary>Answer</summary>0 (flip all 1s to all 0s)</details>

3. **If ~n = -43, what is n?**
   <details><summary>Answer</summary>n = -(~n + 1) → Actually, ~n = -(n+1), so -43 = -(n+1), thus n = 42</details>

4. **Express De Morgan's law for ~(a | b).**
   <details><summary>Answer</summary>`~(a | b) = ~a & ~b` — NOT of OR equals AND of NOTs</details>

5. **Create a mask with all bits set EXCEPT bit 4.**
   <details><summary>Answer</summary>`~(1 << 4)` = ~00010000 = 11101111</details>

6. **Why does ~5 give different results in 8-bit vs 32-bit?**
   <details><summary>Answer</summary>NOT flips ALL bits. In 8-bit there are 8 bits to flip, in 32-bit there are 32 bits. More leading 1s in wider integers.</details>

---

[← Previous: XOR Operation](03-xor-operation.md) | [Next: Left Shift →](05-left-shift.md)
