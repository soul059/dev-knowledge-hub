# Chapter 1.4: Real Number System

[← Previous: Irrational Numbers](03-irrational-numbers.md) | [Back to Contents](../README.md) | [Next: Number Line Representation →](05-number-line-representation.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Understand the complete real number system
- Identify relationships between different number sets
- Apply properties of real numbers
- Work with the density property of real numbers
- Understand the completeness of the real number line

---

## 1. The Complete Picture: Real Numbers

### Definition
**Real numbers (ℝ)** include ALL numbers that can be represented on the number line. This includes both rational AND irrational numbers.

```
ℝ = ℚ ∪ (ℝ - ℚ)
Real = Rational ∪ Irrational
```

### The Grand Hierarchy
```
┌─────────────────────────────────────────────────────────────────────┐
│                        REAL NUMBERS (ℝ)                             │
│         Every point on the number line is a real number             │
│                                                                     │
│    ┌──────────────────────────┐    ┌───────────────────────────┐   │
│    │   RATIONAL NUMBERS (ℚ)   │    │  IRRATIONAL NUMBERS       │   │
│    │   Can be written as p/q  │    │  Cannot be written as p/q │   │
│    │                          │    │                           │   │
│    │   ┌──────────────────┐   │    │  Examples:                │   │
│    │   │   INTEGERS (ℤ)   │   │    │  • √2, √3, √5            │   │
│    │   │   ..., -1, 0, 1  │   │    │  • π ≈ 3.14159...        │   │
│    │   │                  │   │    │  • e ≈ 2.71828...        │   │
│    │   │   ┌───────────┐  │   │    │  • φ ≈ 1.61803...        │   │
│    │   │   │ WHOLE (W) │  │   │    │                           │   │
│    │   │   │ 0,1,2,3...│  │   │    │  Non-terminating,         │   │
│    │   │   │           │  │   │    │  non-repeating decimals   │   │
│    │   │   │ ┌───────┐ │  │   │    │                           │   │
│    │   │   │ │NAT(ℕ) │ │  │   │    │                           │   │
│    │   │   │ │1,2,3..│ │  │   │    │                           │   │
│    │   │   │ └───────┘ │  │   │    │                           │   │
│    │   │   └───────────┘  │   │    │                           │   │
│    │   └──────────────────┘   │    │                           │   │
│    │                          │    │                           │   │
│    │   Also includes:         │    │                           │   │
│    │   • Fractions: 1/2, 3/4  │    │                           │   │
│    │   • Terminating: 0.5     │    │                           │   │
│    │   • Repeating: 0.333...  │    │                           │   │
│    └──────────────────────────┘    └───────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Set Notation Summary
```
ℕ ⊂ W ⊂ ℤ ⊂ ℚ ⊂ ℝ

ℕ = Natural Numbers    {1, 2, 3, 4, ...}
W = Whole Numbers      {0, 1, 2, 3, ...}
ℤ = Integers           {..., -2, -1, 0, 1, 2, ...}
ℚ = Rational Numbers   {p/q : p, q ∈ ℤ, q ≠ 0}
ℝ = Real Numbers       {All rational and irrational numbers}
```

---

## 2. Relationship Diagram

### Venn Diagram Representation
```
                    ┌──────────────────────────────────────┐
                    │           REAL NUMBERS (ℝ)           │
                    │                                      │
    ┌───────────────────────────────────┐                  │
    │        RATIONAL NUMBERS (ℚ)       │     IRRATIONAL   │
    │                                   │                  │
    │   ┌─────────────────────────┐     │    √2, √3, √5   │
    │   │      INTEGERS (ℤ)       │     │                  │
    │   │                         │     │    π, e, φ       │
    │   │   ┌───────────────┐     │     │                  │
    │   │   │   WHOLE (W)   │     │     │    ∛2, ⁴√5      │
    │   │   │               │     │     │                  │
    │   │   │  ┌─────────┐  │     │     │                  │
    │   │   │  │  ℕ      │  │     │     │                  │
    │   │   │  │ 1,2,3.. │  │     │     │                  │
    │   │   │  └─────────┘  │     │     │                  │
    │   │   │   0           │     │     │                  │
    │   │   └───────────────┘     │     │                  │
    │   │     -1, -2, -3...       │     │                  │
    │   └─────────────────────────┘     │                  │
    │     1/2, 3/4, 0.5, 0.333...       │                  │
    └───────────────────────────────────┘                  │
                    │                                      │
                    └──────────────────────────────────────┘
```

### Quick Classification Table

| Number | ℕ | W | ℤ | ℚ | Irrational | ℝ |
|--------|---|---|---|---|------------|---|
| 5 | ✓ | ✓ | ✓ | ✓ | ✗ | ✓ |
| 0 | ✗ | ✓ | ✓ | ✓ | ✗ | ✓ |
| -3 | ✗ | ✗ | ✓ | ✓ | ✗ | ✓ |
| 2/3 | ✗ | ✗ | ✗ | ✓ | ✗ | ✓ |
| 0.75 | ✗ | ✗ | ✗ | ✓ | ✗ | ✓ |
| √2 | ✗ | ✗ | ✗ | ✗ | ✓ | ✓ |
| π | ✗ | ✗ | ✗ | ✗ | ✓ | ✓ |
| -√5 | ✗ | ✗ | ✗ | ✗ | ✓ | ✓ |

---

## 3. Properties of Real Numbers

### Algebraic Properties

| Property | Addition | Multiplication |
|----------|----------|----------------|
| **Closure** | a + b ∈ ℝ | a × b ∈ ℝ |
| **Commutative** | a + b = b + a | a × b = b × a |
| **Associative** | (a + b) + c = a + (b + c) | (a × b) × c = a × (b × c) |
| **Identity** | a + 0 = a | a × 1 = a |
| **Inverse** | a + (-a) = 0 | a × (1/a) = 1 (a ≠ 0) |
| **Distributive** | a × (b + c) = a × b + a × c |

### Examples of Properties

**Closure Property**:
```
√2 + √3 ∈ ℝ (sum of two reals is real)
π × 5 ∈ ℝ (product of two reals is real)
```

**Commutative Property**:
```
√2 + 3 = 3 + √2 ≈ 4.414
π × 2 = 2 × π ≈ 6.283
```

**Associative Property**:
```
(√2 + √3) + √5 = √2 + (√3 + √5)
Both equal approximately 5.60
```

**Distributive Property**:
```
2(√3 + √5) = 2√3 + 2√5
           ≈ 3.46 + 4.47
           ≈ 7.93
```

**Identity Elements**:
```
Additive Identity: a + 0 = a
                   π + 0 = π
                   
Multiplicative Identity: a × 1 = a
                         √2 × 1 = √2
```

**Inverse Elements**:
```
Additive Inverse: a + (-a) = 0
                  √3 + (-√3) = 0
                  
Multiplicative Inverse: a × (1/a) = 1
                        √2 × (1/√2) = 1
                        π × (1/π) = 1
```

---

## 4. Order Properties

### The Real Numbers are Ordered

For any two real numbers a and b, exactly ONE of the following is true:
- a < b
- a = b
- a > b

This is called the **Trichotomy Property**.

### Order Properties

**Transitive Property**:
```
If a < b and b < c, then a < c

Example: 
√2 < 2 and 2 < √5
Therefore: √2 < √5 ✓
```

**Addition Preserves Order**:
```
If a < b, then a + c < b + c

Example:
3 < 5
3 + √2 < 5 + √2 ✓
```

**Multiplication by Positive Preserves Order**:
```
If a < b and c > 0, then ac < bc

Example:
2 < 3 and √5 > 0
2√5 < 3√5 ✓
```

**Multiplication by Negative Reverses Order**:
```
If a < b and c < 0, then ac > bc

Example:
2 < 3 and -1 < 0
2(-1) > 3(-1)
-2 > -3 ✓
```

---

## 5. Density Property

### Key Concept
Between any two real numbers, there are **infinitely many** other real numbers.

```
No matter how close two real numbers are,
you can ALWAYS find another real number between them!
```

### Formal Statement
For any real numbers a and b where a < b:
- There exists a rational number r such that a < r < b
- There exists an irrational number s such that a < s < b

### Finding Numbers Between Two Real Numbers

**Method 1: Average (Arithmetic Mean)**
```
For any a < b:  (a + b)/2 is between a and b

Example: Find a number between 1 and 2
(1 + 2)/2 = 1.5

Find a number between 1.5 and 2
(1.5 + 2)/2 = 1.75

Find a number between 1.75 and 2
(1.75 + 2)/2 = 1.875

This can continue FOREVER!
```

**Method 2: For Rationals**
```
Find rationals between √2 and √3:

√2 ≈ 1.414
√3 ≈ 1.732

Rationals: 1.5 = 3/2
           1.6 = 8/5
           1.7 = 17/10
```

**Method 3: For Irrationals**
```
Find irrationals between 1 and 2:

√2 ≈ 1.414 ✓
√3 ≈ 1.732 ✓
(1 + √2)/2 ≈ 1.207 ✓
π/2 ≈ 1.571 ✓
```

### Visual Representation
```
Between any two points, infinitely many points exist:

    a                                          b
    |──────────────────────────────────────────|
    
    Zoom in:
    a          (a+b)/2                         b
    |────────────●───────────────────────────--|
    
    Zoom in more:
    a    (3a+b)/4    (a+b)/2    (a+3b)/4       b
    |────────●──────────●──────────●───────────|
    
    This continues INFINITELY!
```

---

## 6. Completeness Property

### What is Completeness?

The real numbers are **complete**, meaning:
- Every point on the number line corresponds to exactly one real number
- There are NO "gaps" or "holes" in the real number line
- Every bounded sequence that should converge, does converge to a real number

### Rationals are NOT Complete
```
The sequence: 1, 1.4, 1.41, 1.414, 1.4142, 1.41421, ...

This approaches √2, but √2 is NOT rational!
So within ℚ alone, this sequence has no limit.
The rationals have "holes" where irrationals should be.
```

### Reals ARE Complete
```
The same sequence: 1, 1.4, 1.41, 1.414, 1.4142, ...

Within ℝ, this converges to √2 ∈ ℝ
No holes! Every point exists.
```

### Visual: Holes in the Rationals
```
Rational Number Line (has gaps):
    ────●────●────○────●────○────●────●────→
        1   1.4  √2   1.5   √3   2   2.5
            ↑         ↑    ↑
          Exists   Missing in ℚ

Real Number Line (complete):
    ────●────●────●────●────●────●────●────→
        1   1.4   √2  1.5   √3   2   2.5
    Every point exists in ℝ!
```

---

## 7. Absolute Value in Real Numbers

### Definition
For any real number x:
```
|x| = x,    if x ≥ 0
|x| = -x,   if x < 0
```

### Geometric Interpretation
```
|x| = distance of x from 0 on the number line
```

### Properties of Absolute Value

| Property | Formula | Example |
|----------|---------|---------|
| Non-negativity | \|x\| ≥ 0 | \|-5\| = 5 ≥ 0 |
| Identity | \|x\| = 0 ⟺ x = 0 | \|0\| = 0 |
| Symmetry | \|-x\| = \|x\| | \|-3\| = \|3\| = 3 |
| Multiplicative | \|xy\| = \|x\| · \|y\| | \|(-2)(3)\| = \|-2\| · \|3\| = 6 |
| Division | \|x/y\| = \|x\|/\|y\| | \|-6/2\| = \|-6\|/\|2\| = 3 |
| Triangle Inequality | \|x + y\| ≤ \|x\| + \|y\| | \|3 + (-5)\| ≤ \|3\| + \|-5\| |

### Triangle Inequality Examples
```
|3 + (-5)| ≤ |3| + |-5|
|-2| ≤ 3 + 5
2 ≤ 8 ✓

|√2 + (-√2)| ≤ |√2| + |-√2|
|0| ≤ √2 + √2
0 ≤ 2√2 ✓
```

### Solving Absolute Value Equations

**Type 1**: |x| = a (where a > 0)
```
|x| = 5
Solution: x = 5 or x = -5
```

**Type 2**: |x - c| = a
```
|x - 3| = 7
x - 3 = 7  or  x - 3 = -7
x = 10     or  x = -4
```

**Type 3**: |x| < a (inequality)
```
|x| < 5
-5 < x < 5

On number line:
    ──────(●────────────────●)──────
         -5                  5
```

**Type 4**: |x| > a
```
|x| > 5
x < -5 or x > 5

On number line:
    ●────)──────────────────(────●
         -5                  5
```

---

## 8. Intervals on the Real Number Line

### Types of Intervals

| Interval Notation | Set Notation | Number Line |
|-------------------|--------------|-------------|
| (a, b) Open | {x : a < x < b} | `──(●────────●)──` |
| [a, b] Closed | {x : a ≤ x ≤ b} | `──[●────────●]──` |
| [a, b) Half-open | {x : a ≤ x < b} | `──[●────────●)──` |
| (a, b] Half-open | {x : a < x ≤ b} | `──(●────────●]──` |
| (a, ∞) | {x : x > a} | `──(●────────────→` |
| [a, ∞) | {x : x ≥ a} | `──[●────────────→` |
| (-∞, b) | {x : x < b} | `←────────────●)──` |
| (-∞, b] | {x : x ≤ b} | `←────────────●]──` |
| (-∞, ∞) | ℝ | `←─────────────────→` |

### Visual Examples
```
(2, 5) - Open interval: 2 and 5 are NOT included
    0    1    2    3    4    5    6
    |────|────(●═══●═══●═══●)────|
                   included

[2, 5] - Closed interval: 2 and 5 ARE included
    0    1    2    3    4    5    6
    |────|────[●═══●═══●═══●]────|

[2, 5) - Half-open: 2 included, 5 NOT included
    0    1    2    3    4    5    6
    |────|────[●═══●═══●═══●)────|

(-∞, 3) - All numbers less than 3
    ←────●════●════●════●)────|────
        -1    0    1    2    3    4
```

---

## 9. Solved Examples

### Example 1: Classify each number into all applicable sets

Numbers: 7, -3, 4/5, √9, √7, 0, -2/3, π

**Solution**:
```
7:    ℕ, W, ℤ, ℚ, ℝ (Natural number)
-3:   ℤ, ℚ, ℝ (Negative integer)
4/5:  ℚ, ℝ (Proper fraction)
√9:   = 3 → ℕ, W, ℤ, ℚ, ℝ (Perfect square root)
√7:   Irrational, ℝ (Non-perfect square root)
0:    W, ℤ, ℚ, ℝ (Whole number, not natural)
-2/3: ℚ, ℝ (Negative fraction)
π:    Irrational, ℝ (Famous irrational)
```

### Example 2: Find 5 rational and 3 irrational numbers between 2 and 3

**Solution**:
```
Rational numbers (can be written as p/q):
1. 2.5 = 5/2
2. 2.25 = 9/4
3. 2.75 = 11/4
4. 2.1 = 21/10
5. 2.8 = 14/5

Irrational numbers:
1. √5 ≈ 2.236
2. √6 ≈ 2.449
3. √7 ≈ 2.646

Verification: 2 < √5 < √6 < √7 < 3 ✓
```

### Example 3: Solve |2x - 5| = 9

**Solution**:
```
Case 1: 2x - 5 = 9
        2x = 14
        x = 7

Case 2: 2x - 5 = -9
        2x = -4
        x = -2

Answer: x = 7 or x = -2

Verification:
|2(7) - 5| = |14 - 5| = |9| = 9 ✓
|2(-2) - 5| = |-4 - 5| = |-9| = 9 ✓
```

### Example 4: Represent the solution of |x - 2| < 3 on a number line

**Solution**:
```
|x - 2| < 3
-3 < x - 2 < 3
-3 + 2 < x < 3 + 2
-1 < x < 5

Number Line:
    -2   -1    0    1    2    3    4    5    6
    |────(●═══●════●════●════●════●════●)────|
         ↑                                 ↑
    Not included                    Not included

Answer: (-1, 5) or {x ∈ ℝ : -1 < x < 5}
```

### Example 5: Prove that between any two rational numbers, there is an irrational number

**Solution**:
```
Let a and b be rational numbers with a < b.

Consider: c = a + (b - a)/√2

Since a and b are rational, (b - a) is rational.
Since √2 is irrational, (b - a)/√2 is irrational.
Since a is rational and (b - a)/√2 is irrational,
their sum c is irrational.

Now prove a < c < b:
• c = a + (b - a)/√2
• Since (b - a) > 0 and √2 > 1:
  (b - a)/√2 < (b - a)
• So c < a + (b - a) = b
• Also c > a (since we're adding a positive amount)

Therefore a < c < b, and c is irrational. ∎
```

### Example 6: If √3 = 1.732, find the value of 1/(√3 - 1)

**Solution**:
```
Method: Rationalize the denominator

1/(√3 - 1) × (√3 + 1)/(√3 + 1)

= (√3 + 1)/[(√3)² - (1)²]
= (√3 + 1)/(3 - 1)
= (√3 + 1)/2
= (1.732 + 1)/2
= 2.732/2
= 1.366

Answer: 1.366
```

---

## 10. Mental Math Tricks 🧠

### Trick 1: Quick Classification
```
See a square root? Check if it's a perfect square:
√16 = 4 → Rational (natural number)
√15 → Irrational (15 not a perfect square)

See decimals?
Terminating (0.25) → Rational
Repeating (0.333...) → Rational
Random pattern → Likely Irrational
```

### Trick 2: Identifying Set Membership
```
Question: "Is the number a natural number?"
Test: Is it positive AND a whole number?
  7 → Yes (positive whole)
  0 → No (not positive)
  -3 → No (not positive)
  1/2 → No (not whole)
```

### Trick 3: Sum/Product Classification
```
Rational + Rational = Rational
Rational + Irrational = Irrational
Irrational + Irrational = Could be either!
  (√2 + (-√2) = 0, rational)
  (√2 + √3 ≈ 3.15, irrational)

Similar rules for multiplication!
```

### Trick 4: Quick Interval Notation
```
"Greater than 5" → (5, ∞)
"At least 5" → [5, ∞)
"Less than 5" → (-∞, 5)
"At most 5" → (-∞, 5]
"Between 3 and 7" → (3, 7)
"From 3 to 7 inclusive" → [3, 7]
```

---

## 📊 Summary Table

### Complete Number System Hierarchy

| Set | Symbol | Contains | Example |
|-----|--------|----------|---------|
| Natural | ℕ | Counting numbers | 1, 2, 3, ... |
| Whole | W | Natural + 0 | 0, 1, 2, 3, ... |
| Integer | ℤ | Whole + negatives | ..., -2, -1, 0, 1, 2, ... |
| Rational | ℚ | All p/q (q≠0) | 1/2, -3/4, 5, 0.333... |
| Irrational | ℝ\ℚ | Non-repeating decimals | √2, π, e |
| Real | ℝ | Rational ∪ Irrational | Every number line point |

### Properties Summary

| Property | + | × | Description |
|----------|---|---|-------------|
| Closure | ✓ | ✓ | Result stays in ℝ |
| Commutative | ✓ | ✓ | Order doesn't matter |
| Associative | ✓ | ✓ | Grouping doesn't matter |
| Identity | 0 | 1 | Doesn't change the number |
| Inverse | -a | 1/a | Returns to identity |
| Distributive | × over + | a(b+c) = ab + ac |

### Interval Types

| Notation | Meaning | Endpoints |
|----------|---------|-----------|
| (a, b) | Open | Excluded |
| [a, b] | Closed | Included |
| [a, b) | Left-closed | a in, b out |
| (a, b] | Right-closed | a out, b in |

---

## ❓ Quick Revision Questions

1. **True or False**: Every integer is a rational number.

2. **Classify**: To which sets does -√4 belong?

3. **Find**: An irrational number between √2 and √3.

4. **Solve**: |3x + 2| = 11

5. **Express in interval notation**: -2 ≤ x < 7

6. **Prove**: Show that the sum of a rational and an irrational number is always irrational.

<details>
<summary>Click to see answers</summary>

1. **True** - Every integer n can be written as n/1, which is the form p/q.

2. **-√4 = -2** belongs to: **ℤ, ℚ, ℝ**
   (Not ℕ or W because it's negative)

3. Between √2 ≈ 1.414 and √3 ≈ 1.732:
   **√2.5 ≈ 1.581** is irrational and lies between them.
   (Also valid: (√2 + √3)/2 ≈ 1.573)

4. |3x + 2| = 11
   Case 1: 3x + 2 = 11 → 3x = 9 → x = 3
   Case 2: 3x + 2 = -11 → 3x = -13 → x = -13/3
   **Answer: x = 3 or x = -13/3**

5. -2 ≤ x < 7 means x is at least -2 and less than 7
   **Answer: [-2, 7)**

6. **Proof**:
   Let r be rational and i be irrational.
   Assume r + i = q where q is rational.
   Then i = q - r.
   Since rationals are closed under subtraction, q - r is rational.
   But this contradicts that i is irrational.
   Therefore, r + i must be irrational. ∎

</details>

---

## 🔗 Navigation

[← Previous: Irrational Numbers](03-irrational-numbers.md) | [Back to Contents](../README.md) | [Next: Number Line Representation →](05-number-line-representation.md)
