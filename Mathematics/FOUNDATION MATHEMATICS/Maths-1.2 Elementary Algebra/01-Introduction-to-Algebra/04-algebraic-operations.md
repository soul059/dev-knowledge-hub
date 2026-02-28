# Chapter 1.4: Algebraic Operations

[← Previous: Terms, Coefficients, Like Terms](03-terms-coefficients-like-terms.md) | [Back to Contents](../README.md) | [Next: Types of Polynomials →](../02-Polynomials/01-types-of-polynomials.md)

---

## 📚 Chapter Overview

This chapter covers the fundamental operations on algebraic expressions: **addition**, **subtraction**, **multiplication**, and **division**. Mastering these operations is essential for all further work in algebra.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Add and subtract algebraic expressions
- Multiply monomials and simple expressions
- Divide algebraic terms
- Apply laws of exponents in algebraic operations
- Simplify complex algebraic expressions

---

## 1. Laws of Exponents (Review)

Before diving into algebraic operations, let's review the essential laws of exponents that govern these operations.

### The Seven Laws of Exponents

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LAWS OF EXPONENTS                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. PRODUCT RULE          aᵐ × aⁿ = aᵐ⁺ⁿ                         │
│      Example: x³ × x² = x⁵                                         │
│                                                                     │
│   2. QUOTIENT RULE         aᵐ ÷ aⁿ = aᵐ⁻ⁿ  (a ≠ 0)               │
│      Example: x⁵ ÷ x² = x³                                         │
│                                                                     │
│   3. POWER OF A POWER      (aᵐ)ⁿ = aᵐⁿ                            │
│      Example: (x²)³ = x⁶                                           │
│                                                                     │
│   4. POWER OF A PRODUCT    (ab)ⁿ = aⁿbⁿ                           │
│      Example: (xy)³ = x³y³                                         │
│                                                                     │
│   5. POWER OF A QUOTIENT   (a/b)ⁿ = aⁿ/bⁿ  (b ≠ 0)                │
│      Example: (x/y)² = x²/y²                                       │
│                                                                     │
│   6. ZERO EXPONENT         a⁰ = 1  (a ≠ 0)                        │
│      Example: x⁰ = 1                                               │
│                                                                     │
│   7. NEGATIVE EXPONENT     a⁻ⁿ = 1/aⁿ  (a ≠ 0)                    │
│      Example: x⁻² = 1/x²                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Derivation of Product Rule

**Why does $a^m \times a^n = a^{m+n}$?**

```
By definition of exponents:
aᵐ = a × a × a × ... × a  (m times)
aⁿ = a × a × a × ... × a  (n times)

Therefore:
aᵐ × aⁿ = (a × a × ... × a) × (a × a × ... × a)
              m times           n times
        = a × a × ... × a
          (m + n) times
        = aᵐ⁺ⁿ  ✓
```

### Derivation of Zero Exponent Rule

**Why does $a^0 = 1$?**

```
Using the quotient rule:
aⁿ ÷ aⁿ = aⁿ⁻ⁿ = a⁰

But also:
aⁿ ÷ aⁿ = 1  (any number divided by itself equals 1)

Therefore:
a⁰ = 1  ✓
```

---

## 2. Addition of Algebraic Expressions

### Rule for Addition

To add algebraic expressions:
1. Remove parentheses (if any)
2. Identify like terms
3. Combine like terms by adding their coefficients

```
┌─────────────────────────────────────────────────────────────────────┐
│                   ADDITION OF EXPRESSIONS                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   (3x² + 2x - 5) + (2x² - 4x + 3)                                 │
│                                                                     │
│   Step 1: Remove parentheses (no sign changes)                     │
│           3x² + 2x - 5 + 2x² - 4x + 3                              │
│                                                                     │
│   Step 2: Group like terms                                         │
│           (3x² + 2x²) + (2x - 4x) + (-5 + 3)                      │
│                                                                     │
│   Step 3: Combine                                                  │
│           5x² + (-2x) + (-2)                                       │
│           = 5x² - 2x - 2                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Column Method for Addition

For complex expressions, the column method is helpful:

```
        3x² + 2x - 5
    +   2x² - 4x + 3
    ─────────────────
        5x² - 2x - 2
```

Align like terms in columns and add vertically.

---

## 3. Subtraction of Algebraic Expressions

### Rule for Subtraction

To subtract algebraic expressions:
1. Change the sign of each term in the expression being subtracted
2. Then add as usual

### 💡 Key Concept: Subtracting is Adding the Opposite

$$A - B = A + (-B)$$

```
┌─────────────────────────────────────────────────────────────────────┐
│                 SUBTRACTION OF EXPRESSIONS                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   (5x² + 3x - 2) - (2x² - x + 4)                                  │
│                                                                     │
│   Step 1: Distribute the negative sign                             │
│           5x² + 3x - 2 - 2x² + x - 4                               │
│                ↑           ↑     ↑   ↑                             │
│                │           │     │   │                             │
│           unchanged    sign changes for all                        │
│                        terms after minus                           │
│                                                                     │
│   Step 2: Group like terms                                         │
│           (5x² - 2x²) + (3x + x) + (-2 - 4)                       │
│                                                                     │
│   Step 3: Combine                                                  │
│           3x² + 4x - 6                                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Column Method for Subtraction

```
        5x² + 3x - 2
    -  (2x² - x  + 4)      Change all signs in second row
    ─────────────────
        5x² + 3x - 2
    +  -2x² + x  - 4       Add the changed expression
    ─────────────────
        3x² + 4x - 6
```

---

## 4. Multiplication of Algebraic Expressions

### Types of Multiplication

```
┌─────────────────────────────────────────────────────────────────────┐
│              TYPES OF ALGEBRAIC MULTIPLICATION                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   TYPE 1: Monomial × Monomial                                      │
│   ─────────────────────────────                                    │
│   (3x²)(4x³) = 12x⁵                                                │
│                                                                     │
│   TYPE 2: Monomial × Polynomial                                    │
│   ────────────────────────────                                     │
│   2x(3x² + 4x - 5) = 6x³ + 8x² - 10x                              │
│                                                                     │
│   TYPE 3: Binomial × Binomial (FOIL)                               │
│   ───────────────────────────────                                  │
│   (x + 2)(x + 3) = x² + 5x + 6                                    │
│                                                                     │
│   TYPE 4: Polynomial × Polynomial                                  │
│   ──────────────────────────────                                   │
│   Each term in first × Each term in second                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.1 Monomial × Monomial

**Rule:** Multiply coefficients, add exponents of like bases

$$a^m \cdot b^n \cdot a^p = a^{m+p} \cdot b^n$$

```
Example: (3x²y)(4xy³)

Step 1: Multiply coefficients
        3 × 4 = 12

Step 2: Multiply variable parts (add exponents)
        x² × x = x²⁺¹ = x³
        y × y³ = y¹⁺³ = y⁴

Result: 12x³y⁴
```

### 4.2 Monomial × Polynomial

**Rule:** Use the distributive property

$$a(b + c + d) = ab + ac + ad$$

```
Example: 3x(2x² - 4x + 5)

Distribute 3x to each term:
= 3x(2x²) + 3x(-4x) + 3x(5)
= 6x³ - 12x² + 15x
```

### 4.3 Binomial × Binomial (FOIL Method)

**FOIL** stands for: **F**irst, **O**uter, **I**nner, **L**ast

```
┌─────────────────────────────────────────────────────────────────────┐
│                      FOIL METHOD                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   (a + b)(c + d)                                                   │
│                                                                     │
│   F: First terms      a × c = ac                                   │
│   O: Outer terms      a × d = ad                                   │
│   I: Inner terms      b × c = bc                                   │
│   L: Last terms       b × d = bd                                   │
│                                                                     │
│   Result: ac + ad + bc + bd                                        │
│                                                                     │
│   Visual:                                                          │
│                                                                     │
│        (a + b)(c + d)                                              │
│         ↓   ↓  ↓   ↓                                               │
│         ├───┼──┼───┤  F: First (a·c)                              │
│         ├───┼──┤   │  O: Outer (a·d)                              │
│         │   ├──┼───┤  I: Inner (b·c)                              │
│         │   ├──┤   │  L: Last  (b·d)                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Example:** $(x + 3)(x + 5)$

```
F: x × x = x²
O: x × 5 = 5x
I: 3 × x = 3x
L: 3 × 5 = 15

Result: x² + 5x + 3x + 15
      = x² + 8x + 15
```

### 4.4 Polynomial × Polynomial

**Rule:** Every term in the first polynomial multiplies every term in the second.

```
Example: (x² + 2x + 1)(x + 3)

Method: Distribute each term

= x²(x + 3) + 2x(x + 3) + 1(x + 3)
= x³ + 3x² + 2x² + 6x + x + 3
= x³ + 5x² + 7x + 3
```

**Box/Grid Method:**

```
            │   x    │   3    │
────────────┼────────┼────────┤
    x²      │  x³    │  3x²   │
────────────┼────────┼────────┤
    2x      │  2x²   │  6x    │
────────────┼────────┼────────┤
    1       │   x    │   3    │
────────────┴────────┴────────┘

Sum: x³ + 3x² + 2x² + 6x + x + 3 = x³ + 5x² + 7x + 3
```

---

## 5. Division of Algebraic Expressions

### 5.1 Monomial ÷ Monomial

**Rule:** Divide coefficients, subtract exponents of like bases

$$\frac{a^m}{a^n} = a^{m-n}$$

```
Example: 12x⁵y³ ÷ 4x²y

Step 1: Divide coefficients
        12 ÷ 4 = 3

Step 2: Divide variable parts (subtract exponents)
        x⁵ ÷ x² = x⁵⁻² = x³
        y³ ÷ y = y³⁻¹ = y²

Result: 3x³y²
```

### 5.2 Polynomial ÷ Monomial

**Rule:** Divide each term of the polynomial by the monomial

$$\frac{a + b + c}{d} = \frac{a}{d} + \frac{b}{d} + \frac{c}{d}$$

```
Example: (6x³ - 9x² + 12x) ÷ 3x

= 6x³/3x - 9x²/3x + 12x/3x
= 2x² - 3x + 4
```

### 5.3 Polynomial ÷ Polynomial (Long Division)

This is covered in detail in Chapter 2.4.

---

## 6. Special Products (Introduction)

### Common Special Products

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SPECIAL PRODUCTS                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. SQUARE OF A SUM                                               │
│      (a + b)² = a² + 2ab + b²                                      │
│                                                                     │
│   2. SQUARE OF A DIFFERENCE                                        │
│      (a - b)² = a² - 2ab + b²                                      │
│                                                                     │
│   3. DIFFERENCE OF SQUARES                                         │
│      (a + b)(a - b) = a² - b²                                      │
│                                                                     │
│   4. SUM OF CUBES                                                  │
│      (a + b)(a² - ab + b²) = a³ + b³                               │
│                                                                     │
│   5. DIFFERENCE OF CUBES                                           │
│      (a - b)(a² + ab + b²) = a³ - b³                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Derivation of $(a + b)²$

```
(a + b)² = (a + b)(a + b)

Using FOIL:
F: a × a = a²
O: a × b = ab
I: b × a = ab
L: b × b = b²

Sum: a² + ab + ab + b²
   = a² + 2ab + b²  ✓
```

### Visual Proof of $(a + b)²$

```
┌───────────────────────────────────────┐
│                                       │
│    ┌─────────────┬─────────┐          │
│    │             │         │          │
│    │     a²      │   ab    │  ← a     │
│    │             │         │          │
│    ├─────────────┼─────────┤          │
│    │             │         │          │
│    │     ab      │   b²    │  ← b     │
│    │             │         │          │
│    └─────────────┴─────────┘          │
│          ↑           ↑                │
│          a           b                │
│                                       │
│   Total Area = (a + b)²               │
│              = a² + ab + ab + b²      │
│              = a² + 2ab + b²          │
│                                       │
└───────────────────────────────────────┘
```

---

## ✏️ Solved Examples

### Example 1: Easy - Addition

**Problem:** Add $(4x² + 3x - 2)$ and $(2x² - 5x + 7)$

**Solution:**
```
(4x² + 3x - 2) + (2x² - 5x + 7)

Step 1: Remove parentheses
        4x² + 3x - 2 + 2x² - 5x + 7

Step 2: Combine like terms
        (4x² + 2x²) + (3x - 5x) + (-2 + 7)
        = 6x² + (-2x) + 5
        = 6x² - 2x + 5

Answer: 6x² - 2x + 5
```

### Example 2: Easy - Subtraction

**Problem:** Subtract $(3a - 2b)$ from $(7a + 5b)$

**Solution:**
```
(7a + 5b) - (3a - 2b)

Step 1: Change signs in the expression being subtracted
        7a + 5b - 3a + 2b

Step 2: Combine like terms
        (7a - 3a) + (5b + 2b)
        = 4a + 7b

Answer: 4a + 7b
```

### Example 3: Medium - Multiplication (FOIL)

**Problem:** Multiply $(2x - 3)(3x + 5)$

**Solution:**
```
(2x - 3)(3x + 5)

Using FOIL:
F: (2x)(3x) = 6x²
O: (2x)(5) = 10x
I: (-3)(3x) = -9x
L: (-3)(5) = -15

Combine:
6x² + 10x - 9x - 15
= 6x² + x - 15

Answer: 6x² + x - 15
```

### Example 4: Medium - Division

**Problem:** Divide $15x⁴y³ - 20x³y² + 10x²y$ by $5x²y$

**Solution:**
```
(15x⁴y³ - 20x³y² + 10x²y) ÷ 5x²y

Divide each term separately:

15x⁴y³/5x²y = (15/5)(x⁴⁻²)(y³⁻¹) = 3x²y²

20x³y²/5x²y = (20/5)(x³⁻²)(y²⁻¹) = 4xy

10x²y/5x²y = (10/5)(x²⁻²)(y¹⁻¹) = 2(1)(1) = 2

Result: 3x²y² - 4xy + 2

Answer: 3x²y² - 4xy + 2
```

### Example 5: Hard - Complex Multiplication

**Problem:** Expand $(x + 2)(x - 3)(x + 1)$

**Solution:**
```
(x + 2)(x - 3)(x + 1)

Step 1: Multiply first two binomials
        (x + 2)(x - 3)
        = x² - 3x + 2x - 6
        = x² - x - 6

Step 2: Multiply result by third binomial
        (x² - x - 6)(x + 1)
        
        = x²(x + 1) - x(x + 1) - 6(x + 1)
        = x³ + x² - x² - x - 6x - 6
        = x³ + 0x² - 7x - 6
        = x³ - 7x - 6

Answer: x³ - 7x - 6
```

### Example 6: Hard - Special Products Application

**Problem:** Simplify $(3x + 2y)² - (3x - 2y)²$

**Solution:**
```
(3x + 2y)² - (3x - 2y)²

Method 1: Using special product formulas

Recall: (a + b)² = a² + 2ab + b²
        (a - b)² = a² - 2ab + b²

Here a = 3x, b = 2y

(3x + 2y)² = (3x)² + 2(3x)(2y) + (2y)²
           = 9x² + 12xy + 4y²

(3x - 2y)² = (3x)² - 2(3x)(2y) + (2y)²
           = 9x² - 12xy + 4y²

Subtracting:
(9x² + 12xy + 4y²) - (9x² - 12xy + 4y²)
= 9x² + 12xy + 4y² - 9x² + 12xy - 4y²
= 24xy

Method 2: Using a² - b² = (a + b)(a - b)

Let A = (3x + 2y) and B = (3x - 2y)

A² - B² = (A + B)(A - B)
        = [(3x + 2y) + (3x - 2y)][(3x + 2y) - (3x - 2y)]
        = [6x][4y]
        = 24xy  ✓

Answer: 24xy
```

---

## ❓ Practice Problems

### Easy Level

1. Add: $(5x + 3)$ and $(2x - 7)$

2. Subtract $(2a - 3b)$ from $(5a + 4b)$

3. Multiply: $(4x³)(3x²)$

### Medium Level

4. Multiply using FOIL: $(3x + 4)(2x - 5)$

5. Divide: $(8a³b² - 12a²b + 4ab)$ by $4ab$

6. Simplify: $2x(x + 3) - 3x(x - 2)$

### Hard Level

7. Expand: $(2x - 1)³$

8. Simplify: $(x + y + z)(x - y + z)$

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. $5x + 3 + 2x - 7 = 7x - 4$

2. $(5a + 4b) - (2a - 3b) = 5a + 4b - 2a + 3b = 3a + 7b$

3. $(4)(3) \cdot x^{3+2} = 12x^5$

4. $6x² - 15x + 8x - 20 = 6x² - 7x - 20$

5. $2a²b - 3a + 1$

6. $2x² + 6x - 3x² + 6x = -x² + 12x$

7. $(2x-1)³ = (2x)³ - 3(2x)²(1) + 3(2x)(1)² - 1³$
   $= 8x³ - 12x² + 6x - 1$

8. Group as $[(x + z) + y][(x + z) - y]$
   $= (x + z)² - y²$
   $= x² + 2xz + z² - y²$

</details>

---

## 📋 Summary Table

| Operation | Rule | Example |
|-----------|------|---------|
| **Addition** | Combine like terms | $(3x + 2) + (x - 5) = 4x - 3$ |
| **Subtraction** | Change signs, then add | $(5x - 3) - (2x + 1) = 3x - 4$ |
| **Mono × Mono** | Multiply coefficients, add exponents | $3x² \cdot 2x³ = 6x⁵$ |
| **Mono × Poly** | Distribute | $2x(x + 3) = 2x² + 6x$ |
| **Bi × Bi** | FOIL | $(x+2)(x+3) = x² + 5x + 6$ |
| **Mono ÷ Mono** | Divide coefficients, subtract exponents | $6x⁵ ÷ 2x² = 3x³$ |
| **Poly ÷ Mono** | Divide each term | $(6x² + 4x) ÷ 2x = 3x + 2$ |

---

## 🔄 Quick Revision Questions

1. **What is $x³ \times x⁴$?**

2. **What is $x⁶ ÷ x²$?**

3. **What does FOIL stand for?**

4. **When subtracting expressions, what happens to the signs?**

5. **What is $(a + b)(a - b)$ equal to?**

6. **Can you divide $x³$ by $x⁵$? What's the result?**

<details>
<summary>Quick Answers</summary>

1. $x^7$ (add exponents: 3 + 4 = 7)
2. $x^4$ (subtract exponents: 6 - 2 = 4)
3. First, Outer, Inner, Last
4. All signs in the subtracted expression change
5. $a² - b²$ (difference of squares)
6. Yes, $x^{3-5} = x^{-2} = 1/x²$

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Addition/Subtraction: Only combine LIKE terms                 │
│                                                                     │
│   ★ Multiplication: Multiply coefficients, ADD exponents          │
│                                                                     │
│   ★ Division: Divide coefficients, SUBTRACT exponents             │
│                                                                     │
│   ★ FOIL helps multiply two binomials                             │
│                                                                     │
│   ★ When subtracting, change ALL signs in the subtracted part     │
│                                                                     │
│   ★ Special products are shortcuts worth memorizing               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Unit 1 Complete!

Congratulations! You have completed **Unit 1: Introduction to Algebra**. You now understand:

- ✅ Variables and Constants
- ✅ Algebraic Expressions
- ✅ Terms, Coefficients, and Like Terms
- ✅ Basic Algebraic Operations

**Next up:** [Unit 2: Polynomials](../02-Polynomials/01-types-of-polynomials.md)

---

[← Previous: Terms, Coefficients, Like Terms](03-terms-coefficients-like-terms.md) | [Back to Contents](../README.md) | [Next: Types of Polynomials →](../02-Polynomials/01-types-of-polynomials.md)
