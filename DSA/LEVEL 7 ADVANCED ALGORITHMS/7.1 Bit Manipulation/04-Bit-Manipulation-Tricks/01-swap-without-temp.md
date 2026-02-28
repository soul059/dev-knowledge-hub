# Chapter 4.1: Swap Without Temp Variable

> **Unit 4 — Bit Manipulation Tricks**
> Using XOR to swap two values without any extra memory.

---

## Overview

The XOR swap is a classic bit manipulation trick that swaps two variables without using a temporary variable. It exploits XOR's self-inverse property: `a ^ b ^ b = a`.

---

## The Algorithm

```
  ┌─────────────────────────────────────────┐
  │  STEP 1:  a = a ^ b                    │
  │  STEP 2:  b = a ^ b   → b gets old a   │
  │  STEP 3:  a = a ^ b   → a gets old b   │
  └─────────────────────────────────────────┘
```

## Step-by-Step Trace: Swap a=5, b=9

```
  Initial: a = 5 (0101), b = 9 (1001)

  Step 1: a = a ^ b = 0101 ^ 1001 = 1100 (12)
          a = 12, b = 9

  Step 2: b = a ^ b = 1100 ^ 1001 = 0101 (5)
          a = 12, b = 5    ← b now has original a!

  Step 3: a = a ^ b = 1100 ^ 0101 = 1001 (9)
          a = 9,  b = 5    ← a now has original b!

  Final: a = 9, b = 5  (Swapped!) ✓
```

## Mathematical Proof

```
  Let original values be A and B.

  After step 1:  a = A ^ B
  After step 2:  b = (A ^ B) ^ B = A ^ (B ^ B) = A ^ 0 = A  ✓
  After step 3:  a = (A ^ B) ^ A = (A ^ A) ^ B = 0 ^ B = B  ✓  ∎
```

---

## Pitfall: Swapping a Variable with Itself

```
  ╔══════════════════════════════════════════════════╗
  ║  WARNING: If a and b are the SAME variable       ║
  ║  (same memory location), the value becomes 0!    ║
  ║                                                  ║
  ║  a = a ^ a = 0   (step 1 zeroes it out!)         ║
  ║                                                  ║
  ║  Always check: if (&a != &b) before XOR swap     ║
  ╚══════════════════════════════════════════════════╝
```

---

## Pseudocode

```
FUNCTION xorSwap(a, b):
    IF a is same reference as b:
        RETURN                    // avoid zeroing out
    a = a ^ b
    b = a ^ b
    a = a ^ b
```

---

## Complexity

| | Time | Space |
|---|------|-------|
| XOR Swap | O(1) | O(1) — no temp variable |
| Temp Swap | O(1) | O(1) — one temp variable |

> In practice, compilers optimize both to the same machine code. The temp-variable approach is cleaner and preferred in modern code.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Algorithm | `a^=b; b^=a; a^=b;` |
| Why it works | XOR is self-inverse: `a^b^b = a` |
| Pitfall | Fails if a and b are same memory location |
| Practical use | Rarely needed today; temp swap is cleaner |

---

## Quick Revision Questions

1. **Swap a=7, b=3 using XOR. Show each step.**
   <details><summary>Answer</summary>a=7^3=4, b=4^3=7, a=4^7=3. Result: a=3, b=7</details>

2. **What happens if you XOR-swap a variable with itself?**
   <details><summary>Answer</summary>It becomes 0. x^x=0 in step 1 destroys the value.</details>

3. **Why is temp-variable swap preferred in practice?**
   <details><summary>Answer</summary>It's clearer, has no aliasing bug, and compilers optimize both to the same machine code anyway.</details>

4. **Can XOR swap work on floating-point numbers?**
   <details><summary>Answer</summary>Not directly — XOR operates on integer bit patterns. You'd need to reinterpret the float bits as integers first.</details>

5. **How many XOR operations does the swap require?**
   <details><summary>Answer</summary>3 XOR operations</details>

---

[← Previous Unit: Count Set Bits](../03-Basic-Bit-Techniques/06-count-set-bits.md) | [Next: Find Odd Occurring Element →](02-find-odd-occurring-element.md)
