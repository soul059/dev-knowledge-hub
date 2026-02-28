# Chapter 8.4: Geometric Series

[← Previous: Geometric Sequences](03-geometric-sequences.md) | [Back to Contents](../README.md)

---

## 📚 Chapter Overview

This chapter explores the sum of geometric sequences. We derive formulas for finite geometric series and introduce the fascinating concept of infinite series that converge to a finite sum—a powerful idea with applications in finance, physics, and mathematics.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Calculate the sum of a finite geometric series
- Understand when an infinite geometric series converges
- Calculate the sum of an infinite geometric series
- Apply geometric series to compound interest problems
- Convert repeating decimals to fractions using geometric series
- Solve real-world problems involving geometric series

---

## 1. Finite Geometric Series

### The Sum Formula

```
┌─────────────────────────────────────────────────────────────────────┐
│              GEOMETRIC SERIES SUM FORMULA                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For a geometric series with n terms:                             │
│   Sₙ = a₁ + a₁r + a₁r² + ... + a₁rⁿ⁻¹                            │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                                                             │ │
│   │         a₁(1 - rⁿ)                                         │ │
│   │   Sₙ = ────────────    (when r ≠ 1)                       │ │
│   │           1 - r                                            │ │
│   │                                                             │ │
│   │   Alternative form:                                        │ │
│   │                                                             │ │
│   │         a₁(rⁿ - 1)                                         │ │
│   │   Sₙ = ────────────    (when r ≠ 1)                       │ │
│   │           r - 1                                            │ │
│   │                                                             │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│   Use first form when |r| < 1 (to avoid negative numerator)       │
│   Use second form when r > 1                                       │
│                                                                     │
│   When r = 1: Sₙ = a₁ × n (all terms are equal to a₁)            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Derivation

```
┌─────────────────────────────────────────────────────────────────────┐
│              DERIVING THE FORMULA                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Start with:                                                       │
│   Sₙ = a₁ + a₁r + a₁r² + ... + a₁rⁿ⁻¹  ... (1)                   │
│                                                                     │
│   Multiply both sides by r:                                        │
│   rSₙ = a₁r + a₁r² + a₁r³ + ... + a₁rⁿ  ... (2)                  │
│                                                                     │
│   Subtract (2) from (1):                                           │
│   Sₙ - rSₙ = a₁ - a₁rⁿ                                            │
│   Sₙ(1 - r) = a₁(1 - rⁿ)                                          │
│                                                                     │
│           a₁(1 - rⁿ)                                               │
│   Sₙ = ────────────                                                │
│            1 - r                                                    │
│                                                                     │
│   This works because most terms cancel!                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Example

```
Find the sum: 2 + 6 + 18 + 54 + 162

Step 1: Identify values
        a₁ = 2
        r = 6/2 = 3
        n = 5

Step 2: Apply the formula
        Sₙ = a₁(rⁿ - 1)/(r - 1)
        S₅ = 2(3⁵ - 1)/(3 - 1)
        S₅ = 2(243 - 1)/2
        S₅ = 2(242)/2
        S₅ = 242

Verify: 2 + 6 + 18 + 54 + 162 = 242 ✓
```

---

## 2. Infinite Geometric Series

### Convergence Condition

```
┌─────────────────────────────────────────────────────────────────────┐
│              INFINITE SERIES CONVERGENCE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   An infinite geometric series:                                     │
│   S∞ = a₁ + a₁r + a₁r² + a₁r³ + ... (infinite terms)             │
│                                                                     │
│   CONVERGES (has a finite sum) if and only if |r| < 1              │
│   (i.e., -1 < r < 1)                                               │
│                                                                     │
│   Why? When |r| < 1, the terms get smaller and smaller:           │
│                                                                     │
│   r = 1/2:  1, 1/2, 1/4, 1/8, 1/16, ... → 0                       │
│   r = -1/3: 1, -1/3, 1/9, -1/27, ... → 0                          │
│                                                                     │
│   If |r| ≥ 1, the series DIVERGES (sum is infinite or undefined)  │
│                                                                     │
│   r = 2:  1, 2, 4, 8, 16, ... → ∞ (diverges)                      │
│   r = -1: 1, -1, 1, -1, ... → oscillates (diverges)               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### The Infinite Sum Formula

```
┌─────────────────────────────────────────────────────────────────────┐
│              INFINITE GEOMETRIC SERIES SUM                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐ │
│   │                                                             │ │
│   │          a₁                                                │ │
│   │   S∞ = ─────     (only when |r| < 1)                      │ │
│   │         1 - r                                              │ │
│   │                                                             │ │
│   └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
│   Derivation:                                                       │
│   From Sₙ = a₁(1 - rⁿ)/(1 - r)                                    │
│                                                                     │
│   As n → ∞, if |r| < 1, then rⁿ → 0                               │
│                                                                     │
│   So S∞ = a₁(1 - 0)/(1 - r) = a₁/(1 - r)                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Visual Understanding

```
Sum: 1 + 1/2 + 1/4 + 1/8 + 1/16 + ...

Visualize with a square of area 2:

┌───────────────────────────┐
│           │               │
│     1     │     1/2       │
│           │               │
│           ├───────┬───────┤
│           │ 1/4   │ 1/8   │
│           │       ├───┬───┤
│           │       │1/16...│
└───────────┴───────┴───┴───┘

The pieces fill up a total area of 2!

S∞ = 1/(1 - 1/2) = 1/(1/2) = 2
```

---

## 3. Examples of Infinite Series

### Example 1: Simple convergent series

```
Find the sum: 8 + 4 + 2 + 1 + 1/2 + 1/4 + ...

a₁ = 8
r = 4/8 = 1/2

Since |r| = 1/2 < 1, the series converges.

S∞ = a₁/(1 - r)
S∞ = 8/(1 - 1/2)
S∞ = 8/(1/2)
S∞ = 16
```

### Example 2: Negative ratio

```
Find the sum: 9 - 3 + 1 - 1/3 + 1/9 - ...

a₁ = 9
r = -3/9 = -1/3

Since |r| = 1/3 < 1, the series converges.

S∞ = 9/(1 - (-1/3))
S∞ = 9/(1 + 1/3)
S∞ = 9/(4/3)
S∞ = 27/4 = 6.75
```

---

## 4. Repeating Decimals as Geometric Series

### Converting to Fractions

```
┌─────────────────────────────────────────────────────────────────────┐
│              REPEATING DECIMALS                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   A repeating decimal is an infinite geometric series!             │
│                                                                     │
│   0.333... = 3/10 + 3/100 + 3/1000 + ...                          │
│            = 3/10 + 3/10 × (1/10) + 3/10 × (1/10)² + ...          │
│                                                                     │
│   a₁ = 3/10, r = 1/10                                             │
│                                                                     │
│   S∞ = (3/10)/(1 - 1/10) = (3/10)/(9/10) = 3/9 = 1/3             │
│                                                                     │
│   So 0.333... = 1/3 ✓                                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### More Examples

**0.272727...**
```
0.272727... = 27/100 + 27/10000 + 27/1000000 + ...

a₁ = 27/100, r = 1/100

S∞ = (27/100)/(1 - 1/100)
S∞ = (27/100)/(99/100)
S∞ = 27/99 = 3/11

So 0.272727... = 3/11
```

**0.9999...**
```
0.999... = 9/10 + 9/100 + 9/1000 + ...

a₁ = 9/10, r = 1/10

S∞ = (9/10)/(9/10) = 1

This proves that 0.999... = 1 exactly!
```

---

## 5. Applications

### Compound Interest (Future Value of Annuity)

```
┌─────────────────────────────────────────────────────────────────────┐
│              REGULAR DEPOSITS                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   If you deposit P dollars at the end of each period at           │
│   interest rate i per period for n periods:                       │
│                                                                     │
│   Future Value = P[(1+i)ⁿ - 1]/i                                  │
│                                                                     │
│   This is a geometric series!                                      │
│                                                                     │
│   First deposit grows for (n-1) periods: P(1+i)ⁿ⁻¹                │
│   Second deposit grows for (n-2) periods: P(1+i)ⁿ⁻²               │
│   ...                                                               │
│   Last deposit (no growth): P                                      │
│                                                                     │
│   Sum = P + P(1+i) + P(1+i)² + ... + P(1+i)ⁿ⁻¹                   │
│       = P[(1+i)ⁿ - 1]/[(1+i) - 1]                                 │
│       = P[(1+i)ⁿ - 1]/i                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Example:**
```
You save $100 per month for 10 years at 6% annual 
interest (0.5% monthly). What's the total?

P = 100, i = 0.005, n = 120

FV = 100[(1.005)¹²⁰ - 1]/0.005
FV = 100[1.8194 - 1]/0.005
FV = 100[0.8194]/0.005
FV = 100 × 163.88
FV ≈ $16,388

You deposited $12,000, but earned $4,388 in interest!
```

### Bouncing Ball

```
A ball is dropped from 10 meters. Each bounce reaches 
3/4 of the previous height. Find the total distance 
traveled (until it comes to rest).

Distance down: 10 + 10(3/4) + 10(3/4)² + ...
Distance up:       10(3/4) + 10(3/4)² + ...

Total = 10 + 2[10(3/4) + 10(3/4)² + ...]
      = 10 + 2 × 10(3/4)/(1 - 3/4)
      = 10 + 2 × 10(3/4)/(1/4)
      = 10 + 2 × 10 × 3
      = 10 + 60
      = 70 meters
```

---

## ✏️ Solved Examples

### Example 1: Easy - Finite sum

**Problem:** Find the sum: 3 + 6 + 12 + 24 + 48

**Solution:**
```
a₁ = 3, r = 2, n = 5

S₅ = 3(2⁵ - 1)/(2 - 1)
S₅ = 3(32 - 1)/1
S₅ = 3 × 31
S₅ = 93
```

### Example 2: Easy - Infinite sum

**Problem:** Find the sum: 27 + 9 + 3 + 1 + ...

**Solution:**
```
a₁ = 27, r = 9/27 = 1/3

Since |r| < 1, converges.

S∞ = 27/(1 - 1/3)
S∞ = 27/(2/3)
S∞ = 27 × 3/2
S∞ = 81/2 = 40.5
```

### Example 3: Medium - Find n from sum

**Problem:** The sum of a GP is 728. If a₁ = 2 and r = 3, find n.

**Solution:**
```
Sₙ = a₁(rⁿ - 1)/(r - 1)
728 = 2(3ⁿ - 1)/(3 - 1)
728 = 2(3ⁿ - 1)/2
728 = 3ⁿ - 1
729 = 3ⁿ
3⁶ = 3ⁿ
n = 6

There are 6 terms.
Check: 2 + 6 + 18 + 54 + 162 + 486 = 728 ✓
```

### Example 4: Medium - Repeating decimal

**Problem:** Express 0.454545... as a fraction.

**Solution:**
```
0.454545... = 45/100 + 45/10000 + 45/1000000 + ...

a₁ = 45/100 = 9/20
r = 1/100

S∞ = (45/100)/(1 - 1/100)
S∞ = (45/100)/(99/100)
S∞ = 45/99 = 5/11

So 0.454545... = 5/11
```

### Example 5: Hard - Finding ratio from sums

**Problem:** In an infinite GP, the sum is 4 and the sum of the cubes of the terms is 192. Find a₁ and r.

**Solution:**
```
Sum of GP: S = a₁/(1-r) = 4 ... (1)

The cubes form another GP with first term a₁³ and ratio r³.
Sum of cubes: S' = a₁³/(1-r³) = 192 ... (2)

From (1): a₁ = 4(1-r)

Substitute in (2):
[4(1-r)]³/(1-r³) = 192
64(1-r)³/(1-r³) = 192
64(1-r)³/[(1-r)(1+r+r²)] = 192
64(1-r)²/(1+r+r²) = 192
(1-r)²/(1+r+r²) = 3

(1-r)² = 3(1+r+r²)
1 - 2r + r² = 3 + 3r + 3r²
0 = 2r² + 5r + 2
0 = (2r + 1)(r + 2)
r = -1/2 or r = -2

Since |r| < 1 for convergence: r = -1/2

a₁ = 4(1 - (-1/2)) = 4(3/2) = 6

Answer: a₁ = 6, r = -1/2
Check: S = 6/(1 + 1/2) = 6/(3/2) = 4 ✓
```

### Example 6: Hard - Application

**Problem:** A square has side 10 cm. A second square is inscribed by joining the midpoints of the first. This continues infinitely. Find the total area of all squares.

**Solution:**
```
First square: side = 10, area = 100

Second square: The diagonal of the inscribed square 
equals the side of the outer square.
If inner side = s, then s√2 = 10
s = 10/√2 = 5√2
Area = (5√2)² = 50

Third square: side = 5, area = 25

The areas form a GP: 100, 50, 25, ...
a₁ = 100, r = 1/2

Total area:
S∞ = 100/(1 - 1/2)
S∞ = 100/(1/2)
S∞ = 200 cm²
```

---

## ❓ Practice Problems

### Easy Level

1. Find the sum: 1 + 2 + 4 + 8 + 16 + 32

2. Find the sum: 12 + 6 + 3 + 1.5 + ...

3. Determine if the series converges: 5 + 10 + 20 + 40 + ...

### Medium Level

4. Find the sum of the first 8 terms of 3, 6, 12, 24, ...

5. Express 0.636363... as a fraction.

6. Find the sum: 1 - 1/2 + 1/4 - 1/8 + ...

### Hard Level

7. The sum of an infinite GP is 15 and the sum of the squares of its terms is 45. Find a₁ and r.

8. A pendulum swings 40 cm on its first swing. Each subsequent swing is 90% of the previous. Find the total distance traveled before coming to rest.

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. a₁ = 1, r = 2, n = 6
   S₆ = 1(2⁶ - 1)/(2-1) = 64 - 1 = **63**

2. a₁ = 12, r = 1/2
   S∞ = 12/(1/2) = **24**

3. r = 2, |r| > 1, so **diverges (no finite sum)**

4. a₁ = 3, r = 2, n = 8
   S₈ = 3(2⁸ - 1)/1 = 3(255) = **765**

5. 0.636363... = (63/100)/(99/100) = 63/99 = **7/11**

6. a₁ = 1, r = -1/2
   S∞ = 1/(1 + 1/2) = 1/(3/2) = **2/3**

7. S = a₁/(1-r) = 15
   S' = a₁²/(1-r²) = 45
   
   From first: a₁ = 15(1-r)
   Substitute: [15(1-r)]²/(1-r²) = 45
   225(1-r)²/[(1-r)(1+r)] = 45
   225(1-r)/(1+r) = 45
   5(1-r) = 1+r
   5-5r = 1+r
   4 = 6r
   r = 2/3
   
   a₁ = 15(1 - 2/3) = 15(1/3) = 5
   **a₁ = 5, r = 2/3**

8. Total distance = 40 + 2(40 × 0.9 + 40 × 0.9² + ...)
   = 40 + 2 × 40(0.9)/(1-0.9)
   = 40 + 80 × 0.9/0.1
   = 40 + 720
   = **760 cm** or **7.6 m**

</details>

---

## 📋 Summary Table

| Type | Condition | Formula |
|------|-----------|---------|
| Finite GP sum | r ≠ 1 | Sₙ = a₁(1 - rⁿ)/(1 - r) |
| Finite GP sum (alt) | r > 1 | Sₙ = a₁(rⁿ - 1)/(r - 1) |
| Infinite GP sum | \|r\| < 1 | S∞ = a₁/(1 - r) |
| Divergent | \|r\| ≥ 1 | No finite sum |

---

## 🔄 Quick Revision Questions

1. **What condition must r satisfy for an infinite GP to converge?**

2. **Find the sum: 1/2 + 1/4 + 1/8 + ...**

3. **Does 3 + 6 + 12 + 24 + ... have a finite sum?**

4. **What fraction equals 0.999...?**

5. **Write the formula for the sum of an infinite GP.**

6. **If S∞ = 10 and a₁ = 2, what is r?**

<details>
<summary>Quick Answers</summary>

1. |r| < 1
2. S∞ = (1/2)/(1/2) = 1
3. No, |r| = 2 > 1, so it diverges
4. 1 (exactly!)
5. S∞ = a₁/(1 - r)
6. 10 = 2/(1-r), so 1-r = 0.2, r = 0.8 or 4/5

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Finite geometric series:                                       │
│     Sₙ = a₁(1 - rⁿ)/(1 - r)                                       │
│                                                                     │
│   ★ Infinite geometric series converges when |r| < 1:             │
│     S∞ = a₁/(1 - r)                                               │
│                                                                     │
│   ★ When |r| ≥ 1, the infinite series diverges                    │
│                                                                     │
│   ★ Repeating decimals = infinite geometric series                │
│     → Can convert to fractions using S∞ formula                   │
│                                                                     │
│   ★ Applications:                                                   │
│     • Compound interest and annuities                             │
│     • Physics (bouncing balls, pendulums)                         │
│     • Fractals and geometric patterns                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🎉 Congratulations!

You have completed the **Elementary Algebra** course! You've mastered:

1. ✅ **Introduction to Algebra** - Variables, expressions, and basic operations
2. ✅ **Polynomials** - Adding, subtracting, multiplying, and dividing
3. ✅ **Factorization** - Breaking down polynomials into factors
4. ✅ **Linear Equations** - Solving equations in one variable
5. ✅ **Systems of Linear Equations** - Solving multiple equations simultaneously
6. ✅ **Quadratic Equations** - Various solving methods and applications
7. ✅ **Inequalities** - Linear, compound, absolute value, and graphing
8. ✅ **Sequences and Series** - Arithmetic and geometric patterns

**Keep practicing, and you'll build a strong mathematical foundation!**

---

[← Previous: Geometric Sequences](03-geometric-sequences.md) | [Back to Contents](../README.md)
