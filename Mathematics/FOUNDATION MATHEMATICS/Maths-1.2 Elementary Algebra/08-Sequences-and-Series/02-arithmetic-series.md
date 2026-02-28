# Chapter 8.2: Arithmetic Series

[← Previous: Arithmetic Sequences](01-arithmetic-sequences.md) | [Back to Contents](../README.md) | [Next: Geometric Sequences →](03-geometric-sequences.md)

---

## 📚 Chapter Overview

While a sequence is a list of terms, a series is the SUM of those terms. This chapter focuses on finding the sum of arithmetic sequences efficiently, introducing formulas attributed to young Gauss and their applications.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Distinguish between sequences and series
- Understand sigma notation for series
- Derive and apply the arithmetic series formula
- Use both forms of the sum formula
- Solve problems involving partial sums
- Apply arithmetic series to real-world problems

---

## 1. Series vs. Sequence

### Definitions

```
┌─────────────────────────────────────────────────────────────────────┐
│              SEQUENCE VS. SERIES                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   SEQUENCE: A list of terms                                        │
│   2, 5, 8, 11, 14  (5 terms)                                       │
│                                                                     │
│   SERIES: The SUM of the terms                                     │
│   2 + 5 + 8 + 11 + 14 = 40                                        │
│                                                                     │
│   An ARITHMETIC SERIES is the sum of an arithmetic sequence.       │
│                                                                     │
│   Notation:                                                         │
│   Sₙ = sum of the first n terms                                   │
│   S₅ = a₁ + a₂ + a₃ + a₄ + a₅                                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Sigma Notation

### The Summation Symbol

```
┌─────────────────────────────────────────────────────────────────────┐
│              SIGMA NOTATION                                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│         n                                                           │
│        ___                                                          │
│        ╲                                                            │
│   Sₙ = ╱   aᵢ  = a₁ + a₂ + a₃ + ... + aₙ                         │
│        ‾‾‾                                                          │
│        i=1                                                          │
│                                                                     │
│   Components:                                                       │
│   • Σ (sigma) = "sum of"                                          │
│   • i = index variable (counter)                                  │
│   • 1 = starting value                                            │
│   • n = ending value                                              │
│   • aᵢ = formula for the ith term                                 │
│                                                                     │
│   Example:                                                          │
│    5                                                                │
│   ___                                                               │
│   ╲                                                                 │
│   ╱   (2i + 1) = 3 + 5 + 7 + 9 + 11 = 35                          │
│   ‾‾‾                                                               │
│   i=1                                                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Gauss's Discovery

### The Story

```
┌─────────────────────────────────────────────────────────────────────┐
│              GAUSS'S CLEVER TRICK                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Legend says young Carl Friedrich Gauss (1777-1855) was asked    │
│   to add the numbers from 1 to 100. He solved it instantly!       │
│                                                                     │
│   His method:                                                       │
│   S = 1 + 2 + 3 + ... + 98 + 99 + 100                            │
│   S = 100 + 99 + 98 + ... + 3 + 2 + 1   (write in reverse)       │
│   ─────────────────────────────────────                            │
│   2S = 101 + 101 + 101 + ... + 101 + 101 + 101                    │
│                     (100 pairs of 101)                             │
│                                                                     │
│   2S = 100 × 101                                                   │
│   S = 100 × 101 / 2 = 5050                                        │
│                                                                     │
│   This works for ANY arithmetic series!                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Visual Proof

```
Sum: 1 + 2 + 3 + 4 + 5

Visualize with blocks:
■                           = 1
■ ■                         = 2
■ ■ ■                       = 3
■ ■ ■ ■                     = 4
■ ■ ■ ■ ■                   = 5
─────────
Total = 15

Now double it and complete the rectangle:

■ □ □ □ □ □
■ ■ □ □ □ □
■ ■ ■ □ □ □
■ ■ ■ ■ □ □
■ ■ ■ ■ ■ □

Rectangle: 5 × 6 = 30
Original sum: 30/2 = 15

Formula: n(n+1)/2 where n = 5
         5(6)/2 = 15 ✓
```

---

## 4. The Arithmetic Series Formula

### Two Forms

```
┌─────────────────────────────────────────────────────────────────────┐
│              ARITHMETIC SERIES FORMULAS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   FORM 1: When you know a₁ and aₙ (first and last terms)          │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │              n                                               │ │
│   │        Sₙ = ─── (a₁ + aₙ)                                   │ │
│   │              2                                               │ │
│   │                                                              │ │
│   │   "n terms times average of first and last"                 │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│   FORM 2: When you know a₁ and d (first term and common diff)     │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │              n                                               │ │
│   │        Sₙ = ─── [2a₁ + (n-1)d]                              │ │
│   │              2                                               │ │
│   │                                                              │ │
│   │   (derived by substituting aₙ = a₁ + (n-1)d into Form 1)   │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Derivation of Form 2

```
Sₙ = n/2 (a₁ + aₙ)

Substitute aₙ = a₁ + (n-1)d:

Sₙ = n/2 (a₁ + a₁ + (n-1)d)
Sₙ = n/2 (2a₁ + (n-1)d)
```

---

## 5. Using the Formulas

### Example 1: Using Form 1

```
Find the sum: 3 + 7 + 11 + ... + 99

Step 1: Identify values
        a₁ = 3, aₙ = 99, d = 4

Step 2: Find n (number of terms)
        n = (aₙ - a₁)/d + 1
        n = (99 - 3)/4 + 1
        n = 24 + 1 = 25

Step 3: Apply Form 1
        Sₙ = n/2 (a₁ + aₙ)
        S₂₅ = 25/2 (3 + 99)
        S₂₅ = 25/2 (102)
        S₂₅ = 25 × 51 = 1275

Sum = 1275
```

### Example 2: Using Form 2

```
Find the sum of the first 20 terms of: 5, 8, 11, 14, ...

Step 1: Identify values
        a₁ = 5, d = 3, n = 20

Step 2: Apply Form 2
        Sₙ = n/2 [2a₁ + (n-1)d]
        S₂₀ = 20/2 [2(5) + (20-1)(3)]
        S₂₀ = 10 [10 + 57]
        S₂₀ = 10 × 67 = 670

Sum of first 20 terms = 670
```

---

## 6. Sum of First n Natural Numbers

### Special Cases

```
┌─────────────────────────────────────────────────────────────────────┐
│              SPECIAL SERIES                                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Sum of first n natural numbers:                                  │
│   1 + 2 + 3 + ... + n = n(n+1)/2                                  │
│                                                                     │
│   Sum of first n odd numbers:                                      │
│   1 + 3 + 5 + ... + (2n-1) = n²                                   │
│                                                                     │
│   Sum of first n even numbers:                                     │
│   2 + 4 + 6 + ... + 2n = n(n+1)                                   │
│                                                                     │
│   Examples:                                                         │
│   1+2+3+...+100 = 100(101)/2 = 5050                               │
│   1+3+5+...+99 = 50² = 2500  (50 odd numbers)                     │
│   2+4+6+...+100 = 50(51) = 2550  (50 even numbers)               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. Finding n When Sum is Given

### Working Backwards

```
The sum of an arithmetic series is 210. If a₁ = 5 and d = 4, 
find the number of terms.

Using Sₙ = n/2 [2a₁ + (n-1)d]:
210 = n/2 [2(5) + (n-1)(4)]
210 = n/2 [10 + 4n - 4]
210 = n/2 [4n + 6]
420 = n(4n + 6)
420 = 4n² + 6n
4n² + 6n - 420 = 0
2n² + 3n - 210 = 0

Using the quadratic formula:
n = (-3 ± √(9 + 1680))/4
n = (-3 ± √1689)/4
n = (-3 ± 41.1)/4

n = 38.1/4 = 9.5 (approximately) or negative

Hmm, let me recalculate...
Actually: 2n² + 3n - 210 = 0
Trying to factor: (2n + 21)(n - 10) = 0
                  2n² - 20n + 21n - 210 = 2n² + n - 210 ✗

Let me use the formula more carefully:
n = (-3 ± √(9 + 4·2·210))/(2·2)
n = (-3 ± √1689)/4
√1689 ≈ 41.1
n ≈ 9.5 or negative

Since n must be a whole number, let me verify with n = 10:
S₁₀ = 10/2 [10 + 36] = 5 × 46 = 230 ✗

n = 9: S₉ = 9/2 [10 + 32] = 4.5 × 42 = 189 ✗

There might be an error in my setup. Let me recheck...

Actually, for Sₙ = 210:
2n² + 3n - 210 = 0

Factor: Looking for factors of -420...
(2n - ?)(n + ?) 

By quadratic formula:
n = [-3 ± √(9 + 1680)]/4 = [-3 ± √1689]/4

√1689 = 41.097...

This doesn't give an integer. The problem might need adjustment.

Let's use S = 195 instead for a cleaner example:
195 = n/2(10 + 4n - 4) = n/2(4n + 6)
390 = 4n² + 6n
4n² + 6n - 390 = 0
2n² + 3n - 195 = 0
(2n + 15)(n - 13) = 0... let's check: 2n² - 26n + 15n - 195 = 2n² - 11n - 195 ✗

OK, let me just show a clean example:
For a₁ = 2, d = 3, S = 155:
155 = n/2[4 + 3(n-1)] = n/2[3n + 1]
310 = n(3n + 1) = 3n² + n
3n² + n - 310 = 0
(3n + 31)(n - 10) = 0
n = 10 ✓

Check: S₁₀ = 10/2[4 + 27] = 5 × 31 = 155 ✓
```

---

## ✏️ Solved Examples

### Example 1: Easy - Direct sum

**Problem:** Find the sum: 4 + 7 + 10 + 13 + 16 + 19 + 22

**Solution:**
```
a₁ = 4, aₙ = 22, d = 3

Count terms: n = (22-4)/3 + 1 = 6 + 1 = 7

Sₙ = n/2(a₁ + aₙ)
S₇ = 7/2(4 + 22)
S₇ = 7/2(26)
S₇ = 7 × 13 = 91

Sum = 91
```

### Example 2: Easy - First n terms

**Problem:** Find the sum of the first 15 terms of 6, 11, 16, 21, ...

**Solution:**
```
a₁ = 6, d = 5, n = 15

Sₙ = n/2[2a₁ + (n-1)d]
S₁₅ = 15/2[2(6) + 14(5)]
S₁₅ = 15/2[12 + 70]
S₁₅ = 15/2 × 82
S₁₅ = 15 × 41 = 615

Sum = 615
```

### Example 3: Medium - Sum of integers

**Problem:** Find the sum of all integers from 50 to 150.

**Solution:**
```
This is an AP: 50, 51, 52, ..., 150
a₁ = 50, aₙ = 150, d = 1

n = 150 - 50 + 1 = 101 terms

Sₙ = n/2(a₁ + aₙ)
S₁₀₁ = 101/2(50 + 150)
S₁₀₁ = 101/2 × 200
S₁₀₁ = 101 × 100 = 10100

Sum = 10,100
```

### Example 4: Medium - Sum of specific type

**Problem:** Find the sum of all multiples of 7 between 100 and 500.

**Solution:**
```
Multiples of 7: 105, 112, 119, ..., 497

First: 7 × 15 = 105 (smallest ≥ 100)
Last: 7 × 71 = 497 (largest ≤ 500)

a₁ = 105, aₙ = 497, d = 7

n = (497 - 105)/7 + 1 = 56 + 1 = 57

S₅₇ = 57/2(105 + 497)
S₅₇ = 57/2 × 602
S₅₇ = 57 × 301 = 17,157

Sum = 17,157
```

### Example 5: Hard - Find n from sum

**Problem:** The sum of an AP is 240. If a₁ = 3 and aₙ = 37, find n and d.

**Solution:**
```
Using Sₙ = n/2(a₁ + aₙ):
240 = n/2(3 + 37)
240 = n/2 × 40
240 = 20n
n = 12

Now find d:
aₙ = a₁ + (n-1)d
37 = 3 + 11d
34 = 11d
d = 34/11 ≈ 3.09

Hmm, d should typically be nice. Let me verify:
S₁₂ = 12/2(3 + 37) = 6 × 40 = 240 ✓

n = 12, d = 34/11
```

### Example 6: Hard - Application

**Problem:** An auditorium has 30 rows. The first row has 20 seats, and each row has 3 more seats than the previous row. Find the total seating capacity.

**Solution:**
```
Row 1: 20 seats
Row 2: 23 seats
Row 3: 26 seats
...
Row 30: ?

This is an AP: a₁ = 20, d = 3, n = 30

First, find a₃₀:
a₃₀ = 20 + 29(3) = 20 + 87 = 107

Total seats:
S₃₀ = 30/2(20 + 107)
S₃₀ = 15 × 127
S₃₀ = 1905

Total capacity = 1905 seats
```

---

## ❓ Practice Problems

### Easy Level

1. Find the sum: 2 + 5 + 8 + 11 + 14 + 17

2. Find the sum of the first 10 terms of 7, 12, 17, 22, ...

3. Find 1 + 2 + 3 + ... + 50

### Medium Level

4. Find the sum of all even numbers from 2 to 200.

5. Find the sum: 100 + 95 + 90 + ... + 5

6. The 8th term of an AP is 23 and the sum of first 8 terms is 92. Find a₁ and d.

### Hard Level

7. The sum of the first n terms of an AP is 3n² + 5n. Find a₁, d, and the 10th term.

8. A contractor is paid $100 for the first day of work, with a $5 raise each subsequent day. How much total is earned after working 30 days?

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. a₁ = 2, aₙ = 17, n = 6
   S₆ = 6/2(2+17) = 3×19 = **57**

2. a₁ = 7, d = 5, n = 10
   S₁₀ = 10/2[14 + 45] = 5×59 = **295**

3. n = 50
   S = 50(51)/2 = **1275**

4. 2, 4, 6, ..., 200 (100 terms)
   S = 100/2(2+200) = 50×202 = **10,100**

5. a₁ = 100, aₙ = 5, d = -5
   n = (5-100)/(-5) + 1 = 19 + 1 = 20
   S = 20/2(100+5) = 10×105 = **1050**

6. a₈ = a₁ + 7d = 23
   S₈ = 8/2(a₁ + a₈) = 4(a₁ + 23) = 92
   a₁ + 23 = 23, so a₁ = 0
   Then 7d = 23, d = 23/7
   **a₁ = 0, d = 23/7**
   (Alternative: S₈ = 4[2a₁ + 7d] = 92 → 2a₁ + 7d = 23)

7. Sₙ = 3n² + 5n
   S₁ = a₁ = 3 + 5 = 8, so **a₁ = 8**
   S₂ = a₁ + a₂ = 12 + 10 = 22
   a₂ = 22 - 8 = 14
   **d = 14 - 8 = 6**
   a₁₀ = 8 + 9(6) = **62**

8. a₁ = 100, d = 5, n = 30
   S₃₀ = 30/2[200 + 145] = 15×345 = **$5175**

</details>

---

## 📋 Summary Table

| Formula | When to Use |
|---------|-------------|
| Sₙ = n/2(a₁ + aₙ) | Know first term, last term, and count |
| Sₙ = n/2[2a₁ + (n-1)d] | Know first term, common difference, and count |
| n(n+1)/2 | Sum of first n natural numbers |
| n² | Sum of first n odd numbers |
| n(n+1) | Sum of first n even numbers |

---

## 🔄 Quick Revision Questions

1. **What is the sum 1 + 2 + 3 + ... + 100?**

2. **If a₁ = 5, aₙ = 45, and n = 9, what is Sₙ?**

3. **What's the difference between a sequence and a series?**

4. **Write the formula for Sₙ when you know a₁ and d.**

5. **The sum of first n odd numbers equals what?**

6. **Who is credited with discovering the arithmetic series formula?**

<details>
<summary>Quick Answers</summary>

1. 100(101)/2 = 5050
2. S₉ = 9/2(5+45) = 9/2(50) = 225
3. Sequence is a list; series is the sum of the list
4. Sₙ = n/2[2a₁ + (n-1)d]
5. n²
6. Carl Friedrich Gauss

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ A series is the SUM of sequence terms                         │
│                                                                     │
│   ★ Two main formulas for arithmetic series:                       │
│     • Sₙ = n/2(a₁ + aₙ) - when you know first and last terms     │
│     • Sₙ = n/2[2a₁ + (n-1)d] - when you know first term and d    │
│                                                                     │
│   ★ Special formulas:                                              │
│     • 1+2+...+n = n(n+1)/2                                        │
│     • Sum of first n odd = n²                                     │
│                                                                     │
│   ★ The key insight: Sum = (number of terms) × (average term)     │
│                                                                     │
│   ★ Always find n first if not given                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Arithmetic Sequences](01-arithmetic-sequences.md) | [Back to Contents](../README.md) | [Next: Geometric Sequences →](03-geometric-sequences.md)
