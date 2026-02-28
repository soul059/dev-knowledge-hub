# 2.3 Multiplication Method

## Unit 2: Hash Functions

---

## Definition

The **Multiplication Method** hashes a key in two steps: multiply the key by a constant $A$ (where $0 < A < 1$), extract the fractional part, then multiply by the table size.

$$h(k) = \lfloor m \cdot (k \cdot A \mod 1) \rfloor$$

Where:
- $k$ = key
- $A$ = constant, $0 < A < 1$
- $m$ = table size
- $k \cdot A \mod 1$ = fractional part of $k \cdot A$

---

## How It Works

```
╔══════════════════════════════════════════════════════════════╗
║               MULTIPLICATION METHOD STEPS                   ║
║                                                             ║
║   Step 1: Multiply key by constant A                        ║
║           k × A = integer_part.fractional_part              ║
║                                                             ║
║   Step 2: Extract fractional part                           ║
║           frac = k × A - floor(k × A)                      ║
║                                                             ║
║   Step 3: Multiply by table size and floor                  ║
║           h(k) = floor(m × frac)                            ║
║                                                             ║
║   Example: k=123, A=0.6180339887, m=10                     ║
║   Step 1: 123 × 0.6180339887 = 76.018180...                ║
║   Step 2: frac = 0.018180...                                ║
║   Step 3: floor(10 × 0.018180) = floor(0.18180) = 0        ║
║           h(123) = 0                                        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## The Golden Ratio Constant

Knuth suggests using $A = \frac{\sqrt{5} - 1}{2} \approx 0.6180339887$ (the inverse of the golden ratio), which provides excellent distribution.

```
╔═══════════════════════════════════════════════════════════╗
║              GOLDEN RATIO CONSTANT                       ║
║                                                          ║
║        √5 - 1                                            ║
║   A = ────────  ≈  0.6180339887...                       ║
║          2                                               ║
║                                                          ║
║   Why golden ratio?                                      ║
║   • Most irrational number (hardest to approximate       ║
║     by fractions)                                        ║
║   • Produces maximum spread of hash values               ║
║   • Successive keys don't cluster                        ║
║                                                          ║
║   Keys:  1,  2,  3,  4,  5  (consecutive)              ║
║   Hashes spread widely despite sequential input          ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Step-by-Step Trace

```
A = 0.6180339887,  m = 8

Key 61:
  61 × 0.6180339887 = 37.700073...
  Fractional part = 0.700073...
  floor(8 × 0.700073) = floor(5.600584) = 5
  h(61) = 5

Key 62:
  62 × 0.6180339887 = 38.318107...
  Fractional part = 0.318107...
  floor(8 × 0.318107) = floor(2.544856) = 2
  h(62) = 2

Key 63:
  63 × 0.6180339887 = 38.936141...
  Fractional part = 0.936141...
  floor(8 × 0.936141) = floor(7.489128) = 7
  h(63) = 7

Key 64:
  64 × 0.6180339887 = 39.554175...
  Fractional part = 0.554175...
  floor(8 × 0.554175) = floor(4.433400) = 4
  h(64) = 4

Key 65:
  65 × 0.6180339887 = 40.172209...
  Fractional part = 0.172209...
  floor(8 × 0.172209) = floor(1.377672) = 1
  h(65) = 1

Final Table:
Index:  0    1    2    3    4    5    6    7
      ┌────┬────┬────┬────┬────┬────┬────┬────┐
      │    │ 65 │ 62 │    │ 64 │ 61 │    │ 63 │
      └────┴────┴────┴────┴────┴────┴────┴────┘

Consecutive keys → spread across different buckets!
```

---

## Pseudocode

```
CONSTANT A = 0.6180339887     // (√5 - 1) / 2

FUNCTION hash_multiplication(key, table_size):
    product = key * A
    fractional = product - FLOOR(product)
    index = FLOOR(table_size * fractional)
    RETURN index

// Integer-only version (avoids floating point)
// Uses bit manipulation: A ≈ s / 2^w where w = word size
FUNCTION hash_multiplication_int(key, table_size_power):
    // table_size = 2^p, w = 32 (word size)
    // s = floor(A * 2^32) = 2654435769
    s = 2654435769
    product = key * s               // Uses only lower 32 bits
    index = product >> (32 - p)     // Extract top p bits
    RETURN index
```

---

## Division vs Multiplication Method

| Aspect | Division Method | Multiplication Method |
|--------|:-:|:-:|
| Formula | $k \mod m$ | $\lfloor m(kA \mod 1) \rfloor$ |
| Table size dependency | Critical (must be prime) | **Any size works** |
| Speed | Very fast (1 mod) | Slightly slower (multiply + extract) |
| Distribution | Good with prime m | Good with any m |
| Implementation | Simple | Moderate |
| Consecutive keys | May cluster | Excellent spread |
| Power-of-2 table | Poor | **Works well** |

```
╔═══════════════════════════════════════════════════════╗
║   KEY ADVANTAGE: Table size m doesn't matter!        ║
║                                                      ║
║   Division:       m must be prime → less flexible    ║
║   Multiplication: m can be anything → even 2^p      ║
║                                                      ║
║   This makes multiplication method ideal when        ║
║   table size must be a power of 2 (common in         ║
║   practice for bit-shift operations).                ║
╚═══════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: Walk through the multiplication method for key=49, A=0.6180339887, m=16. What index does it produce?**

<details>
<summary>Click to reveal answer</summary>

1. **Multiply**: $49 \times 0.6180339887 = 30.283665...$
2. **Fractional part**: $0.283665...$
3. **Multiply by m**: $16 \times 0.283665 = 4.538...$
4. **Floor**: $\lfloor 4.538 \rfloor = 4$

**h(49) = 4**

The key advantage shown here: even with $m = 16$ (a power of 2), the multiplication method produces a well-distributed result, whereas the division method with $m = 16$ would only look at the last 4 bits ($49 \mod 16 = 1$).

</details>

---

| [← Previous: Division Method](02-division-method.md) | [Next: Universal Hashing →](04-universal-hashing.md) |
|:---|---:|
