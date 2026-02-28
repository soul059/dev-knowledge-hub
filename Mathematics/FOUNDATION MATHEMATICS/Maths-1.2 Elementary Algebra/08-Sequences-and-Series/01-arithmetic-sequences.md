# Chapter 8.1: Arithmetic Sequences

[← Previous: Graphing Inequalities](../07-Inequalities/04-graphing-inequalities.md) | [Back to Contents](../README.md) | [Next: Arithmetic Series →](02-arithmetic-series.md)

---

## 📚 Chapter Overview

A sequence is an ordered list of numbers following a specific pattern. Arithmetic sequences are among the most fundamental, where each term is obtained by adding a constant value to the previous term. Understanding these sequences is essential for modeling many real-world situations.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define and identify arithmetic sequences
- Find the common difference
- Write the general (nth) term formula
- Find any term of an arithmetic sequence
- Determine if a number belongs to a sequence
- Solve problems involving arithmetic sequences

---

## 1. What is a Sequence?

### Definition

```
┌─────────────────────────────────────────────────────────────────────┐
│              SEQUENCES                                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   A SEQUENCE is an ordered list of numbers following a pattern.   │
│                                                                     │
│   Each number in the sequence is called a TERM.                    │
│                                                                     │
│   Notation:                                                         │
│   a₁, a₂, a₃, a₄, ..., aₙ, ...                                    │
│                                                                     │
│   a₁ = first term                                                  │
│   a₂ = second term                                                 │
│   aₙ = nth term (general term)                                     │
│                                                                     │
│   Example: 2, 5, 8, 11, 14, ...                                   │
│   a₁ = 2, a₂ = 5, a₃ = 8, etc.                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Arithmetic Sequences

### Definition

```
┌─────────────────────────────────────────────────────────────────────┐
│              ARITHMETIC SEQUENCE                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   An ARITHMETIC SEQUENCE (or arithmetic progression, AP)           │
│   is a sequence where the difference between consecutive           │
│   terms is CONSTANT.                                               │
│                                                                     │
│   This constant is called the COMMON DIFFERENCE (d).               │
│                                                                     │
│   d = a₂ - a₁ = a₃ - a₂ = a₄ - a₃ = ... = aₙ₊₁ - aₙ             │
│                                                                     │
│   Example: 3, 7, 11, 15, 19, ...                                  │
│   Common difference d = 7 - 3 = 4                                  │
│                                                                     │
│   Each term = previous term + d                                    │
│   a₂ = a₁ + d                                                      │
│   a₃ = a₂ + d = a₁ + 2d                                           │
│   a₄ = a₃ + d = a₁ + 3d                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Visual Representation

```
Arithmetic sequence: 5, 8, 11, 14, 17, ...  (d = 3)

    +3     +3     +3     +3
 5 ──→ 8 ──→ 11 ──→ 14 ──→ 17 ──→ ...
 ↑     ↑      ↑      ↑      ↑
 a₁    a₂     a₃     a₄     a₅

On a number line:
├─────┼─────┼─────┼─────┼─────┼─────→
0     5     8    11    14    17
      ├──3──┤──3──┤──3──┤──3──┤
```

---

## 3. The General Term Formula

### Derivation

```
┌─────────────────────────────────────────────────────────────────────┐
│              DERIVING THE NTH TERM FORMULA                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Start with a₁ and add d repeatedly:                             │
│                                                                     │
│   a₁ = a₁                      = a₁ + 0·d                         │
│   a₂ = a₁ + d                  = a₁ + 1·d                         │
│   a₃ = a₁ + d + d              = a₁ + 2·d                         │
│   a₄ = a₁ + d + d + d          = a₁ + 3·d                         │
│   ...                                                               │
│   aₙ = a₁ + d + d + ... + d    = a₁ + (n-1)·d                     │
│            └──(n-1) times──┘                                       │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                                                             │ │
│   │        aₙ = a₁ + (n - 1)d                                  │ │
│   │                                                             │ │
│   │   where:                                                    │ │
│   │   aₙ = nth term                                            │ │
│   │   a₁ = first term                                          │ │
│   │   n = term number (position)                               │ │
│   │   d = common difference                                    │ │
│   │                                                             │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Example

```
Find the 20th term of the sequence: 4, 9, 14, 19, ...

Step 1: Identify a₁ and d
        a₁ = 4
        d = 9 - 4 = 5

Step 2: Apply the formula
        aₙ = a₁ + (n - 1)d
        a₂₀ = 4 + (20 - 1)(5)
        a₂₀ = 4 + 19(5)
        a₂₀ = 4 + 95
        a₂₀ = 99

The 20th term is 99.
```

---

## 4. Finding the Common Difference

### Methods

```
┌─────────────────────────────────────────────────────────────────────┐
│              FINDING d                                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Method 1: Subtract consecutive terms                             │
│   d = a₂ - a₁ = a₃ - a₂ = any aₙ₊₁ - aₙ                          │
│                                                                     │
│   Method 2: Using two terms aₘ and aₙ (where m < n)               │
│                                                                     │
│   From: aₙ = a₁ + (n-1)d  and  aₘ = a₁ + (m-1)d                  │
│                                                                     │
│   Subtract: aₙ - aₘ = (n-1)d - (m-1)d = (n-m)d                    │
│                                                                     │
│              aₙ - aₘ                                               │
│   So:   d = ─────────                                              │
│               n - m                                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Example:**
```
In an AP, the 3rd term is 7 and the 7th term is 19. Find d.

d = (a₇ - a₃)/(7 - 3)
d = (19 - 7)/4
d = 12/4
d = 3
```

---

## 5. Finding the First Term

### When Given Other Information

```
Example: The 5th term of an AP is 23 and d = 4. Find a₁.

Using aₙ = a₁ + (n-1)d:
a₅ = a₁ + (5-1)(4)
23 = a₁ + 16
a₁ = 7

Check: 7, 11, 15, 19, 23 ✓
```

---

## 6. Finding the Number of Terms

### Rearranging the Formula

```
┌─────────────────────────────────────────────────────────────────────┐
│              FINDING n (NUMBER OF TERMS)                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   From aₙ = a₁ + (n-1)d, solve for n:                             │
│                                                                     │
│   aₙ - a₁ = (n-1)d                                                 │
│                                                                     │
│   (aₙ - a₁)/d = n - 1                                              │
│                                                                     │
│        aₙ - a₁                                                      │
│   n = ───────── + 1                                                │
│           d                                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Example:**
```
How many terms are in the sequence: 5, 11, 17, ..., 95?

a₁ = 5, d = 6, aₙ = 95

n = (aₙ - a₁)/d + 1
n = (95 - 5)/6 + 1
n = 90/6 + 1
n = 15 + 1
n = 16

There are 16 terms.

Check: a₁₆ = 5 + 15(6) = 5 + 90 = 95 ✓
```

---

## 7. Checking if a Number is in the Sequence

### Method

To check if a value L is in an arithmetic sequence, solve for n:

```
Is 50 in the sequence 7, 12, 17, 22, ...?

a₁ = 7, d = 5

If 50 is a term, then aₙ = 50 for some positive integer n.

50 = 7 + (n-1)(5)
43 = 5(n-1)
43/5 = n-1
8.6 = n-1
n = 9.6

Since n is NOT a positive integer, 50 is NOT in the sequence.

Check nearby: a₉ = 7 + 8(5) = 47, a₁₀ = 7 + 9(5) = 52
```

---

## 8. Arithmetic Means

### Definition

```
┌─────────────────────────────────────────────────────────────────────┐
│              ARITHMETIC MEANS                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   The arithmetic means between two numbers are the terms           │
│   that form an arithmetic sequence with those numbers.             │
│                                                                     │
│   If a and b are end terms with k means between them:              │
│   a, m₁, m₂, ..., mₖ, b                                           │
│                                                                     │
│   Total terms = k + 2                                              │
│                                                                     │
│           b - a                                                     │
│   d = ───────────                                                  │
│          k + 1                                                      │
│                                                                     │
│   Special case: ONE mean between a and b                           │
│                                                                     │
│        a + b                                                        │
│   m = ───────  (the arithmetic mean or average)                   │
│          2                                                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Example:**
```
Insert 3 arithmetic means between 5 and 21.

We want: 5, _, _, _, 21 (5 terms total)

k = 3 means, so k + 1 = 4 intervals

d = (21 - 5)/(3 + 1) = 16/4 = 4

The sequence is: 5, 9, 13, 17, 21

Arithmetic means: 9, 13, 17
```

---

## ✏️ Solved Examples

### Example 1: Easy - Find common difference

**Problem:** Find d for the sequence: 12, 8, 4, 0, -4, ...

**Solution:**
```
d = a₂ - a₁ = 8 - 12 = -4

or

d = a₃ - a₂ = 4 - 8 = -4

Common difference: d = -4
```

### Example 2: Easy - Find the nth term

**Problem:** Find the 15th term of 3, 8, 13, 18, ...

**Solution:**
```
a₁ = 3, d = 5

a₁₅ = a₁ + (15-1)d
a₁₅ = 3 + 14(5)
a₁₅ = 3 + 70
a₁₅ = 73

The 15th term is 73.
```

### Example 3: Medium - Find a₁ given two terms

**Problem:** The 4th term of an AP is 14 and the 8th term is 26. Find a₁ and d.

**Solution:**
```
Set up equations:
a₄ = a₁ + 3d = 14  ... (1)
a₈ = a₁ + 7d = 26  ... (2)

Subtract (1) from (2):
4d = 12
d = 3

Substitute in (1):
a₁ + 3(3) = 14
a₁ = 14 - 9 = 5

Answer: a₁ = 5, d = 3
Sequence: 5, 8, 11, 14, 17, 20, 23, 26, ...
```

### Example 4: Medium - Find number of terms

**Problem:** The first term of an AP is 3 and the last term is 99. If d = 4, find the number of terms.

**Solution:**
```
aₙ = a₁ + (n-1)d
99 = 3 + (n-1)(4)
96 = 4(n-1)
24 = n-1
n = 25

There are 25 terms.
```

### Example 5: Hard - Problem solving

**Problem:** The sum of three consecutive terms of an AP is 33 and their product is 1287. Find the terms.

**Solution:**
```
Let the three terms be: (a-d), a, (a+d)
(This is a useful trick for three consecutive AP terms)

Sum: (a-d) + a + (a+d) = 33
     3a = 33
     a = 11

Product: (a-d) · a · (a+d) = 1287
        (11-d)(11)(11+d) = 1287
        11(121 - d²) = 1287
        121 - d² = 117
        d² = 4
        d = ±2

If d = 2: terms are 9, 11, 13
If d = -2: terms are 13, 11, 9

Check: 9 + 11 + 13 = 33 ✓
       9 × 11 × 13 = 1287 ✓

The terms are 9, 11, 13 (or 13, 11, 9).
```

### Example 6: Hard - Real-world application

**Problem:** A theater has 25 rows. The first row has 18 seats, and each subsequent row has 2 more seats than the row in front of it. How many seats are in the 25th row?

**Solution:**
```
This is an AP where:
a₁ = 18 (first row)
d = 2 (2 more seats each row)
n = 25

a₂₅ = a₁ + (25-1)d
a₂₅ = 18 + 24(2)
a₂₅ = 18 + 48
a₂₅ = 66

The 25th row has 66 seats.
```

---

## ❓ Practice Problems

### Easy Level

1. Find the common difference: 7, 3, -1, -5, ...

2. Find the 10th term of: 2, 7, 12, 17, ...

3. Write the first 5 terms of an AP with a₁ = -3 and d = 4.

### Medium Level

4. The 6th term of an AP is 17 and the 10th term is 33. Find a₁ and d.

5. How many terms are in the sequence: 11, 17, 23, ..., 101?

6. Is 85 a term in the sequence 4, 9, 14, 19, ...?

### Hard Level

7. Insert 4 arithmetic means between 3 and 28.

8. The sum of three numbers in AP is 27 and the sum of their squares is 293. Find the numbers.

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. d = 3 - 7 = **-4**

2. a₁ = 2, d = 5
   a₁₀ = 2 + 9(5) = **47**

3. -3, 1, 5, 9, 13

4. a₆ = a₁ + 5d = 17
   a₁₀ = a₁ + 9d = 33
   Subtract: 4d = 16, d = 4
   a₁ = 17 - 20 = **-3**
   **d = 4, a₁ = -3**

5. a₁ = 11, d = 6, aₙ = 101
   n = (101 - 11)/6 + 1 = 15 + 1 = **16 terms**

6. n = (85 - 4)/5 + 1 = 81/5 + 1 = 17.2
   **No**, 85 is not in the sequence (n must be an integer)

7. 5 terms total: 3, _, _, _, _, 28
   d = (28 - 3)/5 = 5
   Means: **8, 13, 18, 23**

8. Let terms be (a-d), a, (a+d)
   Sum: 3a = 27, so a = 9
   Sum of squares: (9-d)² + 81 + (9+d)² = 293
   81 - 18d + d² + 81 + 81 + 18d + d² = 293
   243 + 2d² = 293
   d² = 25, d = ±5
   **Numbers: 4, 9, 14 (or 14, 9, 4)**

</details>

---

## 📋 Summary Table

| To Find | Formula |
|---------|---------|
| nth term | aₙ = a₁ + (n-1)d |
| Common difference | d = a₂ - a₁ = (aₙ - aₘ)/(n - m) |
| Number of terms | n = (aₙ - a₁)/d + 1 |
| First term | a₁ = aₙ - (n-1)d |
| Arithmetic mean of a and b | m = (a + b)/2 |

---

## 🔄 Quick Revision Questions

1. **What is the common difference in 10, 7, 4, 1, ...?**

2. **Write the formula for the nth term of an AP.**

3. **If a₁ = 5 and d = -2, what is a₄?**

4. **How many terms are between a₁ and aₙ in an AP?**

5. **What is the arithmetic mean of 8 and 20?**

6. **Can d be negative in an AP?**

<details>
<summary>Quick Answers</summary>

1. d = -3
2. aₙ = a₁ + (n-1)d
3. a₄ = 5 + 3(-2) = -1
4. n - 2 terms (the first and last are not "between")
5. (8 + 20)/2 = 14
6. Yes! The sequence decreases.

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Arithmetic Sequence: constant difference between terms        │
│                                                                     │
│   ★ Key formula: aₙ = a₁ + (n-1)d                                 │
│                                                                     │
│   ★ Common difference d can be positive, negative, or zero        │
│                                                                     │
│   ★ To find if a value is in the sequence:                        │
│     Solve for n and check if it's a positive integer              │
│                                                                     │
│   ★ Arithmetic mean of two numbers = their average                │
│                                                                     │
│   ★ For problems with 3 consecutive terms, use (a-d), a, (a+d)    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Graphing Inequalities](../07-Inequalities/04-graphing-inequalities.md) | [Back to Contents](../README.md) | [Next: Arithmetic Series →](02-arithmetic-series.md)
