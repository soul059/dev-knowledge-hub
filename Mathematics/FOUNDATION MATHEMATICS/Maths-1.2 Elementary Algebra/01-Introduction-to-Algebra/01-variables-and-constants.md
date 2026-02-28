# Chapter 1.1: Variables and Constants

[← Back to Table of Contents](../README.md) | [Next: Algebraic Expressions →](02-algebraic-expressions.md)

---

## 📚 Chapter Overview

This chapter introduces the fundamental building blocks of algebra: **variables** and **constants**. Understanding these concepts is essential as they form the foundation upon which all algebraic operations are built.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define and identify variables and constants
- Understand the role of symbols in algebra
- Distinguish between variables and constants in expressions
- Recognize the importance of algebraic notation

---

## 1. What is Algebra?

### The Transition from Arithmetic to Algebra

**Arithmetic** deals with specific numbers and their operations.
**Algebra** generalizes arithmetic by using symbols to represent numbers.

```
┌─────────────────────────────────────────────────────────────────────┐
│                  FROM ARITHMETIC TO ALGEBRA                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ARITHMETIC                          ALGEBRA                       │
│   ──────────                          ───────                       │
│   2 + 3 = 5                           a + b = c                    │
│   4 × 5 = 20                          x × y = z                    │
│   10 - 6 = 4                          m - n = p                    │
│                                                                     │
│   Specific numbers    ─────────►    General symbols                │
│   Single answer       ─────────►    Pattern/relationship           │
│   Limited scope       ─────────►    Universal application          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 💡 Key Insight

> Algebra allows us to express general mathematical truths that apply to infinitely many numbers, not just specific cases.

---

## 2. Constants

### Definition

A **constant** is a quantity whose value is fixed and does not change throughout a mathematical discussion or problem.

### Types of Constants

#### 1. Numerical Constants
These are specific numbers that have fixed values.

| Constant | Value | Description |
|----------|-------|-------------|
| 5 | 5 | Integer constant |
| -3 | -3 | Negative integer |
| 2.5 | 2.5 | Decimal constant |
| ¾ | 0.75 | Fractional constant |

#### 2. Absolute Constants
These are special mathematical constants with fixed, unchanging values.

| Symbol | Name | Approximate Value |
|--------|------|-------------------|
| π (pi) | Ratio of circumference to diameter | 3.14159... |
| e | Euler's number | 2.71828... |
| φ (phi) | Golden ratio | 1.61803... |

#### 3. Arbitrary Constants
These are constants that can take any fixed value in a particular context, often denoted by letters like a, b, c at the beginning of the alphabet.

```
Example: In y = mx + c

• m is an arbitrary constant (the slope)
• c is an arbitrary constant (the y-intercept)

For a specific line, these have fixed values:
y = 2x + 5  →  m = 2, c = 5
```

### 💡 Important Note

> Although arbitrary constants are represented by letters (like variables), they maintain a fixed value within a given problem or equation.

---

## 3. Variables

### Definition

A **variable** is a symbol (usually a letter) that represents a quantity which can take different values from a given set called its **domain**.

### Characteristics of Variables

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PROPERTIES OF VARIABLES                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. PLACEHOLDER     Variables hold the place for unknown values   │
│                                                                     │
│   2. FLEXIBLE        Can represent different values                 │
│                                                                     │
│   3. SYMBOLIC        Usually denoted by letters (x, y, z, etc.)    │
│                                                                     │
│   4. DOMAIN-BOUND    Values come from a specified set               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Common Variable Notations

| Context | Commonly Used Variables |
|---------|-------------------------|
| Unknown values | x, y, z |
| Indices/Counters | i, j, k, n |
| Time | t |
| Angles | θ (theta), α (alpha), β (beta) |
| Functions | f, g, h |

### Examples of Variables in Action

**Example 1: Area of a Rectangle**
```
Area = length × width
A = l × w

Here:
• A, l, w are variables
• They can take any positive value
• The relationship holds for ALL rectangles
```

**Example 2: Distance Formula**
```
Distance = Speed × Time
d = s × t

Here:
• d, s, t are variables
• For s = 60 km/h and t = 2 hours: d = 120 km
• For s = 80 km/h and t = 3 hours: d = 240 km
```

---

## 4. Distinguishing Variables from Constants

### Visual Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│              VARIABLES vs CONSTANTS                                 │
├───────────────────────────┬─────────────────────────────────────────┤
│        VARIABLES          │           CONSTANTS                     │
├───────────────────────────┼─────────────────────────────────────────┤
│ Change value              │ Fixed value                             │
│ Usually at end: x, y, z   │ Usually at beginning: a, b, c          │
│ Represent unknowns        │ Represent known quantities              │
│ Domain dependent          │ Context independent                     │
│ "What we find"            │ "What we know"                          │
└───────────────────────────┴─────────────────────────────────────────┘
```

### Contextual Identification

The same letter can be a variable or constant depending on context:

**Example: The equation $y = 2x + 3$**

| Symbol | Type | Reason |
|--------|------|--------|
| x | Variable | Can take any value from its domain |
| y | Variable | Changes based on the value of x |
| 2 | Constant | Fixed numerical coefficient |
| 3 | Constant | Fixed numerical term |

**Example: The general linear equation $y = mx + c$**

| Symbol | Type | Reason |
|--------|------|--------|
| x | Variable | Independent variable |
| y | Variable | Dependent variable |
| m | Arbitrary Constant | Fixed slope for a given line |
| c | Arbitrary Constant | Fixed y-intercept for a given line |

---

## 5. Domain of Variables

### Definition

The **domain** of a variable is the set of all values that the variable can take.

### Common Domains

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMMON NUMBER SETS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ℕ (Natural Numbers)     = {1, 2, 3, 4, 5, ...}                   │
│                                                                     │
│   ℤ (Integers)            = {..., -2, -1, 0, 1, 2, ...}            │
│                                                                     │
│   ℚ (Rational Numbers)    = {p/q : p, q ∈ ℤ, q ≠ 0}                │
│                                                                     │
│   ℝ (Real Numbers)        = All points on the number line          │
│                                                                     │
│   ℂ (Complex Numbers)     = {a + bi : a, b ∈ ℝ, i² = -1}           │
│                                                                     │
│                                                                     │
│         ℕ ⊂ ℤ ⊂ ℚ ⊂ ℝ ⊂ ℂ                                          │
│                                                                     │
│   Visualization:                                                    │
│   ┌─────────────────────────────────────┐                          │
│   │  ℂ (Complex)                        │                          │
│   │   ┌─────────────────────────────┐   │                          │
│   │   │  ℝ (Real)                   │   │                          │
│   │   │   ┌─────────────────────┐   │   │                          │
│   │   │   │  ℚ (Rational)       │   │   │                          │
│   │   │   │   ┌─────────────┐   │   │   │                          │
│   │   │   │   │  ℤ (Integer)│   │   │   │                          │
│   │   │   │   │   ┌─────┐   │   │   │   │                          │
│   │   │   │   │   │  ℕ  │   │   │   │   │                          │
│   │   │   │   │   └─────┘   │   │   │   │                          │
│   │   │   │   └─────────────┘   │   │   │                          │
│   │   │   └─────────────────────┘   │   │                          │
│   │   └─────────────────────────────┘   │                          │
│   └─────────────────────────────────────┘                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Domain Examples

| Expression | Variable | Natural Domain |
|------------|----------|----------------|
| $\sqrt{x}$ | x | x ≥ 0 (non-negative reals) |
| $\frac{1}{x}$ | x | x ≠ 0 (all reals except 0) |
| $\log(x)$ | x | x > 0 (positive reals) |
| $x²$ | x | All real numbers |

---

## 6. Algebraic Notation Conventions

### Standard Conventions

```
┌─────────────────────────────────────────────────────────────────────┐
│                  NOTATION CONVENTIONS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. MULTIPLICATION                                                 │
│      • 3 × x  →  3x  or  3·x                                       │
│      • a × b  →  ab  or  a·b                                       │
│      • The multiplication sign is often omitted                     │
│                                                                     │
│   2. DIVISION                                                       │
│      • a ÷ b  →  a/b  or  ─                                        │
│                          b                                          │
│                                                                     │
│   3. EXPONENTS                                                      │
│      • x × x  →  x²                                                │
│      • x × x × x  →  x³                                            │
│                                                                     │
│   4. COEFFICIENTS                                                   │
│      • 1x  →  x  (coefficient of 1 is not written)                 │
│      • -1x  →  -x  (only the sign is written)                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Examples of Simplified Notation

| Full Form | Simplified Form |
|-----------|-----------------|
| $3 \times x$ | $3x$ |
| $a \times b \times c$ | $abc$ |
| $x \times x \times x$ | $x³$ |
| $1 \times x$ | $x$ |
| $(-1) \times y$ | $-y$ |
| $\frac{a}{b}$ | $\frac{a}{b}$ or $a/b$ |

---

## ✏️ Solved Examples

### Example 1: Easy - Identifying Variables and Constants

**Problem:** Identify the variables and constants in the expression $5x + 3y - 7$

**Solution:**
```
Expression: 5x + 3y - 7

Step 1: Identify numerical values
        • 5, 3, and 7 are CONSTANTS (fixed values)

Step 2: Identify letter symbols that can change
        • x and y are VARIABLES (can take different values)

Answer:
┌─────────────────────────────────────┐
│  Constants: 5, 3, -7                │
│  Variables: x, y                    │
└─────────────────────────────────────┘
```

### Example 2: Easy - Determining Values

**Problem:** If $x = 3$ in the expression $2x + 5$, find the value.

**Solution:**
```
Expression: 2x + 5

Step 1: Substitute x = 3
        2(3) + 5

Step 2: Multiply
        6 + 5

Step 3: Add
        11

Answer: When x = 3, the expression equals 11
```

### Example 3: Medium - Multiple Substitutions

**Problem:** For the expression $3a - 2b + c$, find the value when:
- $a = 4$, $b = 3$, $c = 5$

**Solution:**
```
Expression: 3a - 2b + c

Step 1: Substitute all variables
        3(4) - 2(3) + 5

Step 2: Perform multiplications
        12 - 6 + 5

Step 3: Perform operations left to right
        12 - 6 = 6
        6 + 5 = 11

Answer: When a = 4, b = 3, c = 5, the expression equals 11
```

### Example 4: Medium - Working with Domain

**Problem:** For what values of x is the expression $\frac{3}{x-2}$ defined?

**Solution:**
```
Expression: 3/(x-2)

Step 1: Identify restriction
        Division by zero is undefined
        Therefore, x - 2 ≠ 0

Step 2: Solve for x
        x - 2 ≠ 0
        x ≠ 2

Answer: The expression is defined for all real numbers except x = 2
Domain: {x ∈ ℝ : x ≠ 2} or (-∞, 2) ∪ (2, ∞)
```

### Example 5: Hard - Contextual Analysis

**Problem:** In the formula for the area of a circle $A = πr²$, identify:
1. The variables
2. The constants
3. What happens to A when r doubles?

**Solution:**
```
Formula: A = πr²

Part 1: Variables
        • A (area) - changes based on the radius
        • r (radius) - can take any positive value
        Variables: A, r

Part 2: Constants
        • π (pi) - approximately 3.14159...
        • This is an absolute constant
        Constants: π

Part 3: Effect of doubling r
        Original: A₁ = πr²
        
        When r doubles (becomes 2r):
        A₂ = π(2r)²
        A₂ = π(4r²)
        A₂ = 4πr²
        A₂ = 4A₁

Answer: When the radius doubles, the area becomes 4 times larger

Visual representation:
┌─────────────┐     ┌─────────────────────────┐
│     ○       │     │           ○             │
│   r = 1     │     │         r = 2           │
│   A = π     │     │         A = 4π          │
└─────────────┘     └─────────────────────────┘
```

### Example 6: Hard - Variable Analysis in Physics

**Problem:** The equation of motion $s = ut + \frac{1}{2}at²$ relates distance (s), initial velocity (u), acceleration (a), and time (t). Classify each symbol.

**Solution:**
```
Equation: s = ut + ½at²

Analysis:
┌──────────┬──────────────┬─────────────────────────────────────┐
│  Symbol  │     Type     │           Explanation               │
├──────────┼──────────────┼─────────────────────────────────────┤
│    s     │   Variable   │ Distance - depends on other values  │
│    u     │   Constant   │ Initial velocity - fixed at start   │
│    t     │   Variable   │ Time - independent variable         │
│    a     │   Constant   │ Acceleration - constant in problem  │
│    ½     │   Constant   │ Numerical constant                  │
└──────────┴──────────────┴─────────────────────────────────────┘

Note: u and a are constants for a specific motion scenario,
      but they could be considered parameters that vary
      between different scenarios.
      
Key Relationships:
• s is the dependent variable (output)
• t is the independent variable (input)
• u, a are parameters (fixed for a given problem)
• ½ is an absolute numerical constant
```

---

## ❓ Practice Problems

### Easy Level

1. Identify the variables and constants in: $7x - 4$

2. Find the value of $3y + 2$ when $y = 5$

3. What is the value of $x² - 1$ when $x = 3$?

### Medium Level

4. For the expression $\frac{x+1}{x-3}$, find all values of x for which it is undefined.

5. If $a = 2$ and $b = -3$, find the value of $a² - 2ab + b²$

6. In the formula $V = \frac{4}{3}πr³$, identify all constants and variables.

### Hard Level

7. For what values of x is $\sqrt{x-4}$ defined? Express as an interval.

8. Analyze the expression $\frac{\sqrt{x}}{x-1}$ and determine its domain.

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. **Variables:** x | **Constants:** 7, -4

2. $3(5) + 2 = 15 + 2 = 17$

3. $(3)² - 1 = 9 - 1 = 8$

4. Undefined when $x - 3 = 0$, i.e., $x = 3$

5. $(2)² - 2(2)(-3) + (-3)² = 4 + 12 + 9 = 25$

6. **Constants:** $\frac{4}{3}$, π | **Variables:** V, r

7. $x - 4 ≥ 0$, so $x ≥ 4$. Domain: $[4, ∞)$

8. Need $x ≥ 0$ (for $\sqrt{x}$) AND $x ≠ 1$ (for denominator)
   Domain: $[0, 1) ∪ (1, ∞)$

</details>

---

## 📋 Summary Table

| Concept | Definition | Example |
|---------|------------|---------|
| **Constant** | A quantity with a fixed value | 5, π, e |
| **Variable** | A symbol representing changing values | x, y, z |
| **Numerical Constant** | Specific fixed numbers | 3, -7, 2.5 |
| **Absolute Constant** | Universal mathematical constants | π, e |
| **Arbitrary Constant** | Parameters with fixed values in context | m, c in y = mx + c |
| **Domain** | Set of all possible values for a variable | ℝ, ℕ, [0, ∞) |

---

## 🔄 Quick Revision Questions

1. **What is the main difference between a variable and a constant?**
   
2. **Give three examples of absolute constants.**

3. **In $y = 3x + 7$, which are variables and which are constants?**

4. **What is meant by the "domain" of a variable?**

5. **Why is the expression $\frac{1}{x}$ undefined at $x = 0$?**

6. **Convert $a \times a \times a \times b \times b$ to simplified algebraic notation.**

<details>
<summary>Quick Answers</summary>

1. Variables can take different values; constants have fixed values.
2. π, e, φ (pi, Euler's number, golden ratio)
3. Variables: x, y | Constants: 3, 7
4. The set of all possible values the variable can take
5. Division by zero is undefined in mathematics
6. $a³b²$

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Constants are fixed values (numbers like 5 or symbols like π)  │
│                                                                     │
│   ★ Variables are symbols representing unknown or changing values  │
│                                                                     │
│   ★ The domain defines what values a variable can take             │
│                                                                     │
│   ★ Context determines whether a symbol is a variable or constant  │
│                                                                     │
│   ★ Algebra generalizes arithmetic using symbolic notation         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Back to Table of Contents](../README.md) | [Next: Algebraic Expressions →](02-algebraic-expressions.md)
