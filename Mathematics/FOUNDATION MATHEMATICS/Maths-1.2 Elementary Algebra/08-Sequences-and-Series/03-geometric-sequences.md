# Chapter 8.3: Geometric Sequences

[← Previous: Arithmetic Series](02-arithmetic-series.md) | [Back to Contents](../README.md) | [Next: Geometric Series →](04-geometric-series.md)

---

## 📚 Chapter Overview

While arithmetic sequences grow by adding a constant, geometric sequences grow by multiplying by a constant. This type of sequence models exponential growth and decay, compound interest, population dynamics, and many other real-world phenomena.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define and identify geometric sequences
- Find the common ratio
- Write the general (nth) term formula
- Find any term of a geometric sequence
- Insert geometric means between two numbers
- Solve problems involving geometric sequences

---

## 1. Definition of Geometric Sequence

### What Makes It Geometric?

```
┌─────────────────────────────────────────────────────────────────────┐
│              GEOMETRIC SEQUENCE                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   A GEOMETRIC SEQUENCE (or geometric progression, GP)              │
│   is a sequence where the ratio between consecutive terms          │
│   is CONSTANT.                                                     │
│                                                                     │
│   This constant is called the COMMON RATIO (r).                    │
│                                                                     │
│   r = a₂/a₁ = a₃/a₂ = a₄/a₃ = ... = aₙ₊₁/aₙ                      │
│                                                                     │
│   Example: 3, 6, 12, 24, 48, ...                                  │
│   Common ratio r = 6/3 = 2                                         │
│                                                                     │
│   Each term = previous term × r                                    │
│   a₂ = a₁ × r                                                      │
│   a₃ = a₂ × r = a₁ × r²                                           │
│   a₄ = a₃ × r = a₁ × r³                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Visual Comparison

```
ARITHMETIC (d = 3):     2 ──+3──> 5 ──+3──> 8 ──+3──> 11
                           ADD         ADD         ADD

GEOMETRIC (r = 2):      2 ──×2──> 4 ──×2──> 8 ──×2──> 16
                         MULTIPLY   MULTIPLY   MULTIPLY
```

---

## 2. Types of Common Ratios

### Positive vs. Negative Ratios

```
┌─────────────────────────────────────────────────────────────────────┐
│              TYPES OF COMMON RATIOS                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   r > 1: Sequence increases (growth)                               │
│   Example: 2, 6, 18, 54, ... (r = 3)                              │
│                                                                     │
│   0 < r < 1: Sequence decreases toward 0 (decay)                  │
│   Example: 81, 27, 9, 3, 1, ... (r = 1/3)                         │
│                                                                     │
│   r < 0: Terms alternate in sign                                   │
│   Example: 2, -6, 18, -54, ... (r = -3)                           │
│                                                                     │
│   -1 < r < 0: Alternating and approaching 0                       │
│   Example: 100, -50, 25, -12.5, ... (r = -1/2)                    │
│                                                                     │
│   r = 1: Constant sequence                                         │
│   Example: 5, 5, 5, 5, ... (r = 1)                                │
│                                                                     │
│   Note: r ≠ 0 (dividing by previous term must be defined)         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. The General Term Formula

### Derivation

```
┌─────────────────────────────────────────────────────────────────────┐
│              DERIVING THE NTH TERM FORMULA                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Start with a₁ and multiply by r repeatedly:                     │
│                                                                     │
│   a₁ = a₁                      = a₁ × r⁰                          │
│   a₂ = a₁ × r                  = a₁ × r¹                          │
│   a₃ = a₁ × r × r              = a₁ × r²                          │
│   a₄ = a₁ × r × r × r          = a₁ × r³                          │
│   ...                                                               │
│   aₙ = a₁ × r × r × ... × r    = a₁ × rⁿ⁻¹                        │
│            └──(n-1) times──┘                                       │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                                                             │ │
│   │        aₙ = a₁ × rⁿ⁻¹                                      │ │
│   │                                                             │ │
│   │   where:                                                    │ │
│   │   aₙ = nth term                                            │ │
│   │   a₁ = first term                                          │ │
│   │   r = common ratio                                         │ │
│   │   n = term number (position)                               │ │
│   │                                                             │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Example

```
Find the 8th term of the sequence: 5, 10, 20, 40, ...

Step 1: Identify a₁ and r
        a₁ = 5
        r = 10/5 = 2

Step 2: Apply the formula
        aₙ = a₁ × rⁿ⁻¹
        a₈ = 5 × 2⁸⁻¹
        a₈ = 5 × 2⁷
        a₈ = 5 × 128
        a₈ = 640

The 8th term is 640.
```

---

## 4. Finding the Common Ratio

### Methods

```
┌─────────────────────────────────────────────────────────────────────┐
│              FINDING r                                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Method 1: Divide consecutive terms                               │
│   r = a₂/a₁ = a₃/a₂ = any aₙ₊₁/aₙ                                │
│                                                                     │
│   Method 2: Using two terms aₘ and aₙ (where m < n)               │
│                                                                     │
│   From: aₙ = a₁ × rⁿ⁻¹  and  aₘ = a₁ × rᵐ⁻¹                      │
│                                                                     │
│   Divide: aₙ/aₘ = rⁿ⁻¹/rᵐ⁻¹ = rⁿ⁻ᵐ                               │
│                                                                     │
│   So: r = (aₙ/aₘ)^(1/(n-m))   or   r^(n-m) = aₙ/aₘ               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Example:**
```
In a GP, the 2nd term is 6 and the 5th term is 162. Find r.

a₂ = 6, a₅ = 162

a₅/a₂ = r^(5-2) = r³
162/6 = r³
27 = r³
r = 3
```

---

## 5. Finding the First Term

### When Given Other Information

```
Example: The 4th term of a GP is 24 and r = 2. Find a₁.

Using aₙ = a₁ × rⁿ⁻¹:
a₄ = a₁ × 2³
24 = a₁ × 8
a₁ = 3

Check: 3, 6, 12, 24 ✓
```

---

## 6. Finding the Number of Terms

### Rearranging the Formula

```
Example: In the GP 2, 6, 18, ..., 1458, find the number of terms.

a₁ = 2, r = 3, aₙ = 1458

aₙ = a₁ × rⁿ⁻¹
1458 = 2 × 3ⁿ⁻¹
729 = 3ⁿ⁻¹
3⁶ = 3ⁿ⁻¹
6 = n - 1
n = 7

There are 7 terms.
Check: a₇ = 2 × 3⁶ = 2 × 729 = 1458 ✓
```

---

## 7. Geometric Means

### Definition

```
┌─────────────────────────────────────────────────────────────────────┐
│              GEOMETRIC MEANS                                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   The geometric means between two numbers are the terms            │
│   that form a geometric sequence with those numbers.               │
│                                                                     │
│   If a and b are end terms with k means between them:              │
│   a, m₁, m₂, ..., mₖ, b                                           │
│                                                                     │
│   Total terms = k + 2                                              │
│                                                                     │
│   r = (b/a)^(1/(k+1))                                              │
│                                                                     │
│   Special case: ONE geometric mean between a and b                 │
│                                                                     │
│   m = √(ab)   (the geometric mean)                                 │
│                                                                     │
│   Note: For real means, a and b must have the same sign           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Example:**
```
Insert 2 geometric means between 3 and 81.

We want: 3, _, _, 81 (4 terms total)

k = 2 means, so k + 1 = 3 intervals

r = (81/3)^(1/3) = 27^(1/3) = 3

The sequence is: 3, 9, 27, 81

Geometric means: 9 and 27
```

**Single Mean Example:**
```
Find the geometric mean of 4 and 16.

m = √(4 × 16) = √64 = 8

Check: 4, 8, 16 forms a GP with r = 2 ✓
```

---

## 8. Relationship Between AM and GM

### Arithmetic Mean vs. Geometric Mean

```
┌─────────────────────────────────────────────────────────────────────┐
│              AM-GM INEQUALITY                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For any two positive numbers a and b:                            │
│                                                                     │
│   Arithmetic Mean (AM) = (a + b)/2                                 │
│   Geometric Mean (GM) = √(ab)                                      │
│                                                                     │
│   Key relationship:                                                 │
│                                                                     │
│        AM ≥ GM                                                     │
│                                                                     │
│   (a + b)/2 ≥ √(ab)                                               │
│                                                                     │
│   Equality holds when a = b                                        │
│                                                                     │
│   Example: For a = 4, b = 16:                                     │
│   AM = (4 + 16)/2 = 10                                            │
│   GM = √(64) = 8                                                  │
│   10 ≥ 8 ✓                                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✏️ Solved Examples

### Example 1: Easy - Find common ratio

**Problem:** Find r for the sequence: 5, 15, 45, 135, ...

**Solution:**
```
r = a₂/a₁ = 15/5 = 3

Verify: 45/15 = 3, 135/45 = 3 ✓

Common ratio: r = 3
```

### Example 2: Easy - Find the nth term

**Problem:** Find the 6th term of 4, 12, 36, ...

**Solution:**
```
a₁ = 4, r = 12/4 = 3

a₆ = a₁ × r⁵
a₆ = 4 × 3⁵
a₆ = 4 × 243
a₆ = 972

The 6th term is 972.
```

### Example 3: Medium - Decreasing sequence

**Problem:** Find the 5th term of 64, 32, 16, ...

**Solution:**
```
a₁ = 64, r = 32/64 = 1/2

a₅ = 64 × (1/2)⁴
a₅ = 64 × 1/16
a₅ = 4

The 5th term is 4.
```

### Example 4: Medium - Find a₁ given two terms

**Problem:** The 3rd term of a GP is 12 and the 6th term is 324. Find a₁ and r.

**Solution:**
```
Set up equations:
a₃ = a₁ × r² = 12   ... (1)
a₆ = a₁ × r⁵ = 324  ... (2)

Divide (2) by (1):
r⁵/r² = 324/12
r³ = 27
r = 3

Substitute in (1):
a₁ × 9 = 12
a₁ = 12/9 = 4/3

Answer: a₁ = 4/3, r = 3
Sequence: 4/3, 4, 12, 36, 108, 324, ...
```

### Example 5: Hard - Alternating sequence

**Problem:** Find the 7th term of 3, -6, 12, -24, ...

**Solution:**
```
a₁ = 3, r = -6/3 = -2

a₇ = 3 × (-2)⁶
a₇ = 3 × 64
a₇ = 192

Note: (-2)⁶ is positive because the exponent is even.
The 7th term is 192.

Pattern: 3, -6, 12, -24, 48, -96, 192
         +   -   +   -   +    -   +
```

### Example 6: Hard - Three terms problem

**Problem:** Three numbers are in GP. Their sum is 26 and their product is 216. Find the numbers.

**Solution:**
```
Let the three terms be: a/r, a, ar
(This is a useful trick for three consecutive GP terms)

Product: (a/r) × a × ar = 216
        a³ = 216
        a = 6

Sum: a/r + a + ar = 26
     6/r + 6 + 6r = 26
     6/r + 6r = 20
     6 + 6r² = 20r    (multiply by r)
     6r² - 20r + 6 = 0
     3r² - 10r + 3 = 0
     (3r - 1)(r - 3) = 0
     r = 1/3 or r = 3

If r = 3: 6/3, 6, 18 = 2, 6, 18
If r = 1/3: 18, 6, 2

Both give the same numbers: 2, 6, 18

Check: Sum = 26 ✓, Product = 216 ✓
```

---

## ❓ Practice Problems

### Easy Level

1. Find the common ratio: 4, 20, 100, 500, ...

2. Find the 5th term of: 3, 9, 27, 81, ...

3. Write the first 5 terms of a GP with a₁ = 2 and r = 4.

### Medium Level

4. The 2nd term of a GP is 10 and the 4th term is 40. Find a₁ and r.

5. How many terms are in the GP: 1, 2, 4, ..., 512?

6. Insert one geometric mean between 3 and 27.

### Hard Level

7. Insert 3 geometric means between 2 and 162.

8. The sum of three numbers in GP is 21 and the sum of their squares is 189. Find the numbers.

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. r = 20/4 = **5**

2. a₁ = 3, r = 3
   a₅ = 3 × 3⁴ = 3 × 81 = **243**

3. 2, 8, 32, 128, 512

4. a₂ = a₁r = 10
   a₄ = a₁r³ = 40
   Divide: r² = 4, r = ±2
   If r = 2: a₁ = 5
   If r = -2: a₁ = -5
   **a₁ = 5, r = 2** (or a₁ = -5, r = -2)

5. a₁ = 1, r = 2, aₙ = 512 = 2⁹
   1 × 2^(n-1) = 512
   2^(n-1) = 2⁹
   n - 1 = 9
   **n = 10 terms**

6. m = √(3 × 27) = √81 = **9**

7. 4 terms total: 2, _, _, _, 162
   r = (162/2)^(1/4) = 81^(1/4) = 3
   Means: **6, 18, 54**

8. Let terms be a/r, a, ar
   Product of terms: a³ (but we're not given product)
   
   Sum: a/r + a + ar = 21
   a(1/r + 1 + r) = 21
   
   Sum of squares: a²/r² + a² + a²r² = 189
   a²(1/r² + 1 + r²) = 189
   
   From first equation: a = 21/(1/r + 1 + r) = 21r/(1 + r + r²)
   
   This leads to: Let's try values. If r = 2:
   Sum = a/2 + a + 2a = 3.5a = 21, so a = 6
   Terms: 3, 6, 12
   Sum of squares: 9 + 36 + 144 = 189 ✓
   
   **Numbers: 3, 6, 12**

</details>

---

## 📋 Summary Table

| To Find | Formula |
|---------|---------|
| nth term | aₙ = a₁ × rⁿ⁻¹ |
| Common ratio | r = a₂/a₁ = (aₙ/aₘ)^(1/(n-m)) |
| Number of terms | Solve aₙ = a₁ × rⁿ⁻¹ for n |
| Geometric mean of a and b | m = √(ab) |
| r for k means between a and b | r = (b/a)^(1/(k+1)) |

---

## 🔄 Quick Revision Questions

1. **What is the common ratio in 2, -6, 18, -54, ...?**

2. **Write the formula for the nth term of a GP.**

3. **If a₁ = 3 and r = 2, what is a₅?**

4. **What is the geometric mean of 9 and 16?**

5. **Can r be negative in a GP?**

6. **If r = 1/2, is the sequence increasing or decreasing?**

<details>
<summary>Quick Answers</summary>

1. r = -6/2 = -3
2. aₙ = a₁ × rⁿ⁻¹
3. a₅ = 3 × 2⁴ = 48
4. √(9 × 16) = √144 = 12
5. Yes! Terms will alternate in sign.
6. Decreasing (approaching 0)

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Geometric Sequence: constant ratio between terms              │
│                                                                     │
│   ★ Key formula: aₙ = a₁ × rⁿ⁻¹                                   │
│                                                                     │
│   ★ Common ratio r can be:                                         │
│     • r > 1: increasing sequence                                  │
│     • 0 < r < 1: decreasing (decay)                               │
│     • r < 0: alternating signs                                    │
│                                                                     │
│   ★ Geometric mean of a and b: √(ab)                              │
│                                                                     │
│   ★ AM ≥ GM always (for positive numbers)                         │
│                                                                     │
│   ★ For 3 consecutive terms, use a/r, a, ar                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Arithmetic Series](02-arithmetic-series.md) | [Back to Contents](../README.md) | [Next: Geometric Series →](04-geometric-series.md)
