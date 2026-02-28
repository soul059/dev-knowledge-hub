# Chapter 2.1: Types of Polynomials

[← Previous: Algebraic Operations](../01-Introduction-to-Algebra/04-algebraic-operations.md) | [Back to Contents](../README.md) | [Next: Addition and Subtraction →](02-addition-and-subtraction.md)

---

## 📚 Chapter Overview

A **polynomial** is one of the most fundamental objects in algebra. This chapter introduces the formal definition of polynomials, their classification by degree and number of terms, and standard forms for writing polynomials.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define and identify polynomials
- Classify polynomials by degree and number of terms
- Write polynomials in standard form
- Identify the leading coefficient and constant term
- Distinguish between polynomials and non-polynomials

---

## 1. What is a Polynomial?

### Formal Definition

A **polynomial** in variable $x$ is an algebraic expression of the form:

$$P(x) = a_nx^n + a_{n-1}x^{n-1} + ... + a_2x^2 + a_1x + a_0$$

where:
- $n$ is a **non-negative integer** (0, 1, 2, 3, ...)
- $a_0, a_1, a_2, ..., a_n$ are **constants** called coefficients
- $a_n ≠ 0$ (the leading coefficient is non-zero)
- $x$ is the **variable**

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ANATOMY OF A POLYNOMIAL                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│          5x³ + 2x² - 7x + 3                                        │
│          ↑      ↑     ↑    ↑                                       │
│          │      │     │    │                                       │
│          │      │     │    └── Constant term (a₀ = 3)              │
│          │      │     │                                            │
│          │      │     └────── Linear term (a₁ = -7)                │
│          │      │                                                   │
│          │      └──────────── Quadratic term (a₂ = 2)              │
│          │                                                          │
│          └─────────────────── Cubic term (a₃ = 5)                  │
│                               Leading coefficient = 5               │
│                               Degree = 3                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Key Requirements for Polynomials

```
┌─────────────────────────────────────────────────────────────────────┐
│               CONDITIONS FOR A VALID POLYNOMIAL                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ✓ Exponents must be non-negative INTEGERS (0, 1, 2, 3, ...)     │
│                                                                     │
│   ✓ Coefficients can be any real numbers                          │
│                                                                     │
│   ✓ Variable can only appear in the numerator                     │
│                                                                     │
│   ✓ No variables under roots or radicals                          │
│                                                                     │
│   ✗ No negative exponents (like x⁻¹)                              │
│                                                                     │
│   ✗ No fractional exponents (like x^(1/2))                        │
│                                                                     │
│   ✗ No variables in denominators (like 1/x)                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Examples: Polynomial vs Non-Polynomial

| Expression | Is it a Polynomial? | Reason |
|------------|---------------------|--------|
| $3x² + 2x - 5$ | ✅ Yes | Integer exponents, valid form |
| $x⁴ - 7$ | ✅ Yes | Integer exponents, valid form |
| $\sqrt{2}x³ + πx$ | ✅ Yes | Coefficients can be irrational |
| $5$ | ✅ Yes | Constant polynomial (5x⁰) |
| $x^{-2} + 3$ | ❌ No | Negative exponent |
| $\frac{1}{x} + 2$ | ❌ No | Variable in denominator (= x⁻¹) |
| $\sqrt{x} + 1$ | ❌ No | Fractional exponent (x^½) |
| $2^x + 1$ | ❌ No | Variable in exponent |

---

## 2. Classification by Degree

### Definition of Degree

The **degree** of a polynomial is the highest power of the variable with a non-zero coefficient.

### Special Names by Degree

```
┌─────────────────────────────────────────────────────────────────────┐
│              POLYNOMIALS BY DEGREE                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   DEGREE 0: CONSTANT                                               │
│   ─────────────────────                                            │
│   Examples: 5, -3, π                                               │
│   Form: P(x) = a₀                                                  │
│                                                                     │
│   Graph: Horizontal line                                           │
│   ────────────────────                                             │
│                                                                     │
│   DEGREE 1: LINEAR                                                 │
│   ────────────────────                                             │
│   Examples: 2x + 3, -x + 5, 4x                                     │
│   Form: P(x) = a₁x + a₀                                           │
│                                                                     │
│   Graph: Straight line                                             │
│       │    /                                                       │
│       │   /                                                        │
│       │  /                                                         │
│   ────┼─/─────                                                     │
│       │/                                                           │
│       │                                                            │
│                                                                     │
│   DEGREE 2: QUADRATIC                                              │
│   ───────────────────                                              │
│   Examples: x² + 2x + 1, 3x² - 7, -2x² + 5x                       │
│   Form: P(x) = a₂x² + a₁x + a₀                                    │
│                                                                     │
│   Graph: Parabola                                                  │
│       │    ╭─╮                                                     │
│       │   /   \                                                    │
│       │  /     \                                                   │
│   ────┼─/───────\───                                               │
│       │                                                            │
│                                                                     │
│   DEGREE 3: CUBIC                                                  │
│   ────────────────                                                 │
│   Examples: x³ - 1, 2x³ + x² - 3x + 4                             │
│   Form: P(x) = a₃x³ + a₂x² + a₁x + a₀                            │
│                                                                     │
│   Graph: S-curve                                                   │
│       │        /                                                   │
│       │    ╭──╯                                                    │
│   ────┼───╯─────                                                   │
│       │╭─╯                                                         │
│       ╯                                                            │
│                                                                     │
│   DEGREE 4: QUARTIC (or BIQUADRATIC)                              │
│   ──────────────────────────────────                               │
│   Examples: x⁴ - 1, x⁴ + 2x² + 1                                  │
│                                                                     │
│   DEGREE 5: QUINTIC                                                │
│   ─────────────────                                                │
│   Examples: x⁵ - x, x⁵ + x⁴ + x³ + x² + x + 1                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Summary Table: Degrees

| Degree | Name | General Form | Graph Shape |
|--------|------|--------------|-------------|
| 0 | Constant | $a$ | Horizontal line |
| 1 | Linear | $ax + b$ | Straight line |
| 2 | Quadratic | $ax² + bx + c$ | Parabola |
| 3 | Cubic | $ax³ + bx² + cx + d$ | S-curve |
| 4 | Quartic | $ax⁴ + bx³ + cx² + dx + e$ | W or M shape |
| 5 | Quintic | $ax⁵ + ...$ | Wave-like |
| n | nth degree | $axⁿ + ...$ | Complex curves |

---

## 3. Classification by Number of Terms

### Terminology

```
┌─────────────────────────────────────────────────────────────────────┐
│            POLYNOMIALS BY NUMBER OF TERMS                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   MONOMIAL (1 term)                                                │
│   ─────────────────                                                │
│   Examples: 5x³, -2y², 7, ab                                       │
│                                                                     │
│   BINOMIAL (2 terms)                                               │
│   ──────────────────                                               │
│   Examples: x + 5, a² - b², 3x² - 2x                              │
│                                                                     │
│   TRINOMIAL (3 terms)                                              │
│   ───────────────────                                              │
│   Examples: x² + 5x + 6, a² + 2ab + b²                            │
│                                                                     │
│   POLYNOMIAL (4+ terms)                                            │
│   ─────────────────────                                            │
│   Examples: x³ + 2x² + 3x + 4, a⁴ - a³ + a² - a + 1               │
│                                                                     │
│   Visual:                                                          │
│   ┌───┐    ┌───┬───┐    ┌───┬───┬───┐    ┌───┬───┬───┬───┐       │
│   │ 1 │    │ 1 │ 2 │    │ 1 │ 2 │ 3 │    │ 1 │ 2 │ 3 │...│       │
│   └───┘    └───┴───┘    └───┴───┴───┘    └───┴───┴───┴───┘       │
│   Mono     Bi           Tri              Poly                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Combined Classification

We can describe polynomials using both degree and number of terms:

| Expression | By Terms | By Degree | Combined Name |
|------------|----------|-----------|---------------|
| $5$ | Monomial | Constant | Constant monomial |
| $3x$ | Monomial | Linear | Linear monomial |
| $x² - 4$ | Binomial | Quadratic | Quadratic binomial |
| $x² + 2x + 1$ | Trinomial | Quadratic | Quadratic trinomial |
| $x³ + x$ | Binomial | Cubic | Cubic binomial |
| $x⁴ + 3x² + 1$ | Trinomial | Quartic | Quartic trinomial |

---

## 4. Standard Form of Polynomials

### Definition

A polynomial is in **standard form** (or **descending form**) when its terms are arranged from highest degree to lowest degree.

```
Standard Form: aₙxⁿ + aₙ₋₁xⁿ⁻¹ + ... + a₂x² + a₁x + a₀
               ↓                                    ↓
           Highest                               Lowest
           Degree                                Degree
```

### Examples of Standard Form

| Not Standard | Standard Form |
|--------------|---------------|
| $3 + 2x - x²$ | $-x² + 2x + 3$ |
| $5x - 3x³ + x²$ | $-3x³ + x² + 5x$ |
| $7 - x + 4x³$ | $4x³ - x + 7$ |

### Key Terms in Standard Form

```
For P(x) = 4x³ - 2x² + 5x - 7

┌─────────────────────────────────────────────────────────────────────┐
│                    KEY TERMINOLOGY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   LEADING TERM: 4x³                                                │
│   ─────────────────                                                │
│   The term with the highest degree                                 │
│                                                                     │
│   LEADING COEFFICIENT: 4                                           │
│   ──────────────────────                                           │
│   The coefficient of the leading term                              │
│                                                                     │
│   DEGREE: 3                                                        │
│   ─────────                                                        │
│   The exponent of the leading term                                 │
│                                                                     │
│   CONSTANT TERM: -7                                                │
│   ─────────────────                                                │
│   The term with no variable (x⁰ term)                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Polynomials in Multiple Variables

### Definition

A **polynomial in multiple variables** contains two or more different variables.

### Examples

```
Two variables (x and y):
P(x, y) = 3x²y + 2xy² - 5x + 4y - 7

Three variables (x, y, z):
Q(x, y, z) = x² + y² + z² - 2xy - 2yz - 2xz
```

### Degree in Multiple Variables

The degree of a term with multiple variables is the **sum of all exponents**.
The degree of the polynomial is the **maximum degree among all terms**.

```
Example: P(x, y) = 4x³y² - 2xy³ + 5x²y - 3x + 7

Term Analysis:
┌─────────────┬─────────────────────────────────────┬────────┐
│    Term     │         Degree Calculation          │ Degree │
├─────────────┼─────────────────────────────────────┼────────┤
│   4x³y²     │   3 (from x³) + 2 (from y²) = 5    │   5    │
│   -2xy³     │   1 (from x) + 3 (from y³) = 4     │   4    │
│   5x²y      │   2 (from x²) + 1 (from y) = 3     │   3    │
│   -3x       │   1 (from x)                        │   1    │
│   7         │   0 (constant)                      │   0    │
└─────────────┴─────────────────────────────────────┴────────┘

Degree of P(x, y) = max{5, 4, 3, 1, 0} = 5
```

---

## 6. Evaluating Polynomials

### Substitution Method

To evaluate a polynomial at a specific value, substitute that value for the variable.

**Notation:** $P(a)$ means "evaluate P at x = a"

```
Example: P(x) = 2x³ - 5x² + 3x - 1

Find P(2):
P(2) = 2(2)³ - 5(2)² + 3(2) - 1
     = 2(8) - 5(4) + 6 - 1
     = 16 - 20 + 6 - 1
     = 1
```

### Horner's Method (Efficient Evaluation)

For large polynomials, Horner's method is more efficient.

**Idea:** Rewrite the polynomial in nested form.

```
P(x) = 2x³ - 5x² + 3x - 1

Horner's form:
P(x) = ((2x - 5)x + 3)x - 1

Evaluate P(2):
Step 1: 2(2) - 5 = -1
Step 2: (-1)(2) + 3 = 1
Step 3: (1)(2) - 1 = 1

Answer: P(2) = 1 ✓
```

---

## ✏️ Solved Examples

### Example 1: Easy - Identifying Polynomials

**Problem:** Determine which are polynomials:
(a) $3x⁴ - 2x + 7$
(b) $\frac{1}{x²} + x$
(c) $x^{2/3} + 1$

**Solution:**
```
(a) 3x⁴ - 2x + 7
    All exponents are non-negative integers (4, 1, 0)
    ✓ This IS a polynomial

(b) 1/x² + x = x⁻² + x
    Contains x⁻² (negative exponent)
    ✗ This is NOT a polynomial

(c) x^(2/3) + 1
    Contains fractional exponent (2/3)
    ✗ This is NOT a polynomial
```

### Example 2: Easy - Classification

**Problem:** Classify $4x² - 9$ by:
(a) Number of terms
(b) Degree
(c) Combined name

**Solution:**
```
Expression: 4x² - 9

(a) Number of terms: 2 → BINOMIAL

(b) Highest power: 2 → DEGREE 2 (QUADRATIC)

(c) Combined name: QUADRATIC BINOMIAL

Additional information:
• Leading term: 4x²
• Leading coefficient: 4
• Constant term: -9
```

### Example 3: Medium - Standard Form

**Problem:** Write in standard form and identify key features:
$7 - 3x + 5x³ - x²$

**Solution:**
```
Original: 7 - 3x + 5x³ - x²

Step 1: Arrange by descending powers
        5x³ - x² - 3x + 7

Step 2: Identify key features
┌────────────────────────────────┐
│ Standard form: 5x³ - x² - 3x + 7 │
│ Degree: 3 (cubic)              │
│ Leading term: 5x³              │
│ Leading coefficient: 5         │
│ Constant term: 7               │
│ Number of terms: 4 (polynomial)│
└────────────────────────────────┘
```

### Example 4: Medium - Degree of Multivariable

**Problem:** Find the degree of: $3x²y³ - 2xy⁴ + 5x³ - y²$

**Solution:**
```
Expression: 3x²y³ - 2xy⁴ + 5x³ - y²

Calculate degree of each term:

Term 1: 3x²y³
        Degree = 2 + 3 = 5

Term 2: -2xy⁴
        Degree = 1 + 4 = 5

Term 3: 5x³
        Degree = 3 + 0 = 3

Term 4: -y²
        Degree = 0 + 2 = 2

Maximum degree = max{5, 5, 3, 2} = 5

Answer: The polynomial has degree 5
```

### Example 5: Hard - Polynomial Evaluation

**Problem:** For $P(x) = x³ - 4x² + 5x - 2$, find:
(a) P(1)
(b) P(-2)
(c) P(0)

**Solution:**
```
P(x) = x³ - 4x² + 5x - 2

(a) P(1):
    P(1) = (1)³ - 4(1)² + 5(1) - 2
         = 1 - 4 + 5 - 2
         = 0
    
(b) P(-2):
    P(-2) = (-2)³ - 4(-2)² + 5(-2) - 2
          = -8 - 4(4) - 10 - 2
          = -8 - 16 - 10 - 2
          = -36

(c) P(0):
    P(0) = (0)³ - 4(0)² + 5(0) - 2
         = 0 - 0 + 0 - 2
         = -2

Note: P(0) always equals the constant term!
```

### Example 6: Hard - Creating Polynomials

**Problem:** Write a cubic polynomial in x with:
- Leading coefficient 2
- Constant term -5
- P(1) = 0

**Solution:**
```
General cubic: P(x) = 2x³ + bx² + cx - 5

Given: P(1) = 0
Substitute x = 1:
2(1)³ + b(1)² + c(1) - 5 = 0
2 + b + c - 5 = 0
b + c = 3

We need one equation but have two unknowns.
Choose any values where b + c = 3.

Example solutions:
• b = 0, c = 3: P(x) = 2x³ + 3x - 5
• b = 1, c = 2: P(x) = 2x³ + x² + 2x - 5
• b = 3, c = 0: P(x) = 2x³ + 3x² - 5

Verification for first: P(1) = 2 + 3 - 5 = 0 ✓
```

---

## ❓ Practice Problems

### Easy Level

1. Classify as polynomial or non-polynomial:
   (a) $5x³ - \sqrt{2}x + 1$
   (b) $x^{-1} + x^2$
   (c) $\frac{x³ + 1}{2}$

2. State the degree of: $7x⁵ - 3x² + 2x - 9$

3. Identify the leading coefficient: $-4x⁴ + x³ - 2x + 8$

### Medium Level

4. Write in standard form: $3 - 2x² + 5x⁴ - x$

5. Find the degree of: $4a²b³ - 3ab⁴ + 2a³b - 5$

6. If $P(x) = 2x² - 3x + 1$, find P(2) and P(-1)

### Hard Level

7. Find a value of k such that the polynomial $kx³ - 2x² + 4x - 1$ has degree 2

8. If $P(x) = x³ - 6x² + ax + b$ and P(1) = 0, P(2) = 0, find a and b

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. (a) ✅ Polynomial (irrational coefficients are allowed)
   (b) ❌ Not a polynomial (negative exponent)
   (c) ✅ Polynomial (same as $\frac{1}{2}x³ + \frac{1}{2}$)

2. Degree = 5 (highest power)

3. Leading coefficient = -4

4. Standard form: $5x⁴ - 2x² - x + 3$

5. Term degrees: 5, 5, 4, 0. Degree = 5

6. P(2) = 2(4) - 3(2) + 1 = 8 - 6 + 1 = 3
   P(-1) = 2(1) - 3(-1) + 1 = 2 + 3 + 1 = 6

7. For degree to be 2, coefficient of x³ must be 0
   Therefore k = 0

8. P(1) = 1 - 6 + a + b = 0 → a + b = 5
   P(2) = 8 - 24 + 2a + b = 0 → 2a + b = 16
   Solving: a = 11, b = -6

</details>

---

## 📋 Summary Table

| Concept | Definition | Example |
|---------|------------|---------|
| **Polynomial** | Expression with non-negative integer exponents | $3x² + 2x - 5$ |
| **Degree** | Highest exponent | Degree of $x³ + x$ is 3 |
| **Monomial** | 1 term | $5x²$ |
| **Binomial** | 2 terms | $x + 3$ |
| **Trinomial** | 3 terms | $x² + x + 1$ |
| **Leading coefficient** | Coefficient of highest degree term | In $4x³ - x$: it's 4 |
| **Constant term** | Term with no variable | In $x² + 5$: it's 5 |
| **Standard form** | Descending order of degree | $x³ + x² + x + 1$ |

---

## 🔄 Quick Revision Questions

1. **Is $\frac{5}{x}$ a polynomial? Why or why not?**

2. **What is the degree of the constant polynomial $7$?**

3. **How many terms does a binomial have?**

4. **What is the leading coefficient of $-x³ + 5x$?**

5. **In a multivariable term $x²y³$, what is the degree?**

6. **What is $P(0)$ for any polynomial $P(x) = ax² + bx + c$?**

<details>
<summary>Quick Answers</summary>

1. No, because $\frac{5}{x} = 5x^{-1}$ has a negative exponent
2. Degree 0
3. Two terms
4. -1 (the coefficient of x³)
5. 2 + 3 = 5
6. P(0) = c (the constant term)

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Polynomials have non-negative INTEGER exponents only          │
│                                                                     │
│   ★ Degree = highest power of the variable                        │
│                                                                     │
│   ★ Classified by terms: mono- (1), bi- (2), tri- (3)             │
│                                                                     │
│   ★ Classified by degree: constant, linear, quadratic, cubic...   │
│                                                                     │
│   ★ Standard form: descending order of powers                      │
│                                                                     │
│   ★ P(0) = constant term for any polynomial                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Algebraic Operations](../01-Introduction-to-Algebra/04-algebraic-operations.md) | [Back to Contents](../README.md) | [Next: Addition and Subtraction →](02-addition-and-subtraction.md)
