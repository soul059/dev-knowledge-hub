# Chapter 2.2: Addition and Subtraction of Polynomials

[← Previous: Types of Polynomials](01-types-of-polynomials.md) | [Back to Contents](../README.md) | [Next: Multiplication of Polynomials →](03-multiplication-of-polynomials.md)

---

## 📚 Chapter Overview

This chapter covers the fundamental operations of adding and subtracting polynomials. These operations are essential building blocks for all polynomial manipulation.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Add polynomials using horizontal and vertical methods
- Subtract polynomials correctly
- Simplify polynomial expressions
- Apply addition and subtraction in real-world contexts

---

## 1. Addition of Polynomials

### The Fundamental Rule

**Addition of polynomials = Combining like terms**

Two terms are "like terms" if they have the same variable(s) raised to the same power(s).

```
┌─────────────────────────────────────────────────────────────────────┐
│                 ADDITION OF POLYNOMIALS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Rule: Combine LIKE TERMS by adding their coefficients            │
│                                                                     │
│   (3x² + 2x - 5) + (4x² - 3x + 7)                                 │
│                                                                     │
│   Step 1: Identify like terms                                      │
│   ┌────────┬────────┬────────┐                                     │
│   │   x²   │   x    │ const  │                                     │
│   ├────────┼────────┼────────┤                                     │
│   │  3x²   │  2x    │  -5    │   ← First polynomial               │
│   │  4x²   │ -3x    │   7    │   ← Second polynomial              │
│   ├────────┼────────┼────────┤                                     │
│   │  7x²   │  -x    │   2    │   ← Sum                            │
│   └────────┴────────┴────────┘                                     │
│                                                                     │
│   Answer: 7x² - x + 2                                              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Method 1: Horizontal Addition

Remove parentheses and combine like terms in a single line.

**Example:** $(2x³ + 5x - 3) + (x³ - 2x² + 4x + 1)$

```
Step 1: Remove parentheses
        2x³ + 5x - 3 + x³ - 2x² + 4x + 1

Step 2: Rearrange to group like terms
        (2x³ + x³) + (-2x²) + (5x + 4x) + (-3 + 1)

Step 3: Combine like terms
        3x³ - 2x² + 9x - 2

Answer: 3x³ - 2x² + 9x - 2
```

### Method 2: Vertical (Column) Addition

Align like terms in columns and add vertically.

**Example:** $(2x³ + 5x - 3) + (x³ - 2x² + 4x + 1)$

```
       2x³  + 0x²  + 5x  - 3
    +  x³   - 2x²  + 4x  + 1
    ────────────────────────
       3x³  - 2x²  + 9x  - 2

Note: Use 0x² as a placeholder for the missing term
```

### 💡 Tips for Addition

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TIPS FOR ADDING POLYNOMIALS                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. Write polynomials in STANDARD FORM first                      │
│                                                                     │
│   2. Use PLACEHOLDERS (0x², 0x) for missing terms                 │
│                                                                     │
│   3. Align terms with SAME DEGREE in the same column              │
│                                                                     │
│   4. Add COEFFICIENTS only - variables stay the same              │
│                                                                     │
│   5. Check: Result should have degree ≤ max of input degrees      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Subtraction of Polynomials

### The Fundamental Rule

**Subtraction = Addition of the opposite (additive inverse)**

$$P(x) - Q(x) = P(x) + [-Q(x)]$$

To find $-Q(x)$: change the sign of every term in $Q(x)$.

```
┌─────────────────────────────────────────────────────────────────────┐
│              SUBTRACTION OF POLYNOMIALS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   (5x² + 3x - 2) - (2x² - 4x + 5)                                 │
│                                                                     │
│   Step 1: Distribute the negative sign                             │
│                                                                     │
│   = 5x² + 3x - 2 - 2x² + 4x - 5                                   │
│                    ↑     ↑    ↑                                    │
│                    Signs change for ALL terms                      │
│                    in the subtracted polynomial                    │
│                                                                     │
│   Step 2: Combine like terms                                       │
│   = (5x² - 2x²) + (3x + 4x) + (-2 - 5)                           │
│   = 3x² + 7x - 7                                                   │
│                                                                     │
│   Answer: 3x² + 7x - 7                                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### ⚠️ Common Mistake Alert

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMMON MISTAKE!                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   WRONG:  (5x² + 3x) - (2x² - 4x)                                 │
│           = 5x² + 3x - 2x² - 4x     ← Forgot to change BOTH signs │
│           = 3x² - x                  ← INCORRECT!                  │
│                                                                     │
│   RIGHT:  (5x² + 3x) - (2x² - 4x)                                 │
│           = 5x² + 3x - 2x² + 4x     ← Changed BOTH signs          │
│           = 3x² + 7x                 ← CORRECT!                    │
│                                                                     │
│   Remember: Change the sign of EVERY term being subtracted        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Vertical Subtraction Method

**Example:** Subtract $(x² - 3x + 5)$ from $(4x² + 2x - 1)$

```
Step 1: Write as (4x² + 2x - 1) - (x² - 3x + 5)

Step 2: Change signs and add

       4x²  + 2x  - 1
    - (x²   - 3x  + 5)
    ──────────────────
       4x²  + 2x  - 1
    +  -x²  + 3x  - 5     ← Signs changed
    ──────────────────
       3x²  + 5x  - 6

Answer: 3x² + 5x - 6
```

---

## 3. Combined Operations

### Multiple Additions and Subtractions

When dealing with multiple polynomials, work left to right or use grouping.

**Example:** $(3x² - 2x + 1) + (x² + 4x - 3) - (2x² - x - 2)$

```
Step 1: Handle addition first
        (3x² - 2x + 1) + (x² + 4x - 3)
        = 4x² + 2x - 2

Step 2: Now subtract
        (4x² + 2x - 2) - (2x² - x - 2)
        = 4x² + 2x - 2 - 2x² + x + 2
        = 2x² + 3x + 0
        = 2x² + 3x

Or, all at once:
3x² - 2x + 1 + x² + 4x - 3 - 2x² + x + 2
= (3 + 1 - 2)x² + (-2 + 4 + 1)x + (1 - 3 + 2)
= 2x² + 3x + 0
= 2x² + 3x
```

### Nested Expressions

**Example:** $3x - [2x² - (x² + 3x - 1) + 4]$

```
Step 1: Work from innermost brackets
        = 3x - [2x² - x² - 3x + 1 + 4]
        = 3x - [x² - 3x + 5]

Step 2: Remove outer brackets
        = 3x - x² + 3x - 5
        = -x² + 6x - 5

Answer: -x² + 6x - 5
```

---

## 4. Properties of Polynomial Addition

### Closure Property

The sum of two polynomials is always a polynomial.

### Commutative Property

$$P(x) + Q(x) = Q(x) + P(x)$$

```
Example:
(2x + 3) + (x - 1) = 3x + 2
(x - 1) + (2x + 3) = 3x + 2  ✓ Same result
```

### Associative Property

$$[P(x) + Q(x)] + R(x) = P(x) + [Q(x) + R(x)]$$

```
Example:
[(x + 1) + (x + 2)] + (x + 3) = (2x + 3) + (x + 3) = 3x + 6
(x + 1) + [(x + 2) + (x + 3)] = (x + 1) + (2x + 5) = 3x + 6  ✓
```

### Additive Identity

$$P(x) + 0 = P(x)$$

The zero polynomial (0) is the additive identity.

### Additive Inverse

For every polynomial $P(x)$, there exists $-P(x)$ such that:

$$P(x) + [-P(x)] = 0$$

```
Example:
If P(x) = 3x² - 2x + 5
Then -P(x) = -3x² + 2x - 5

P(x) + [-P(x)] = (3x² - 2x + 5) + (-3x² + 2x - 5) = 0 ✓
```

---

## 5. Polynomials in Multiple Variables

### Adding Multivariable Polynomials

The same rules apply: combine like terms (terms with exactly the same variables and powers).

**Example:** $(3x²y + 2xy² - 5) + (x²y - 4xy² + 3)$

```
Identify like terms:
• x²y terms: 3x²y and x²y
• xy² terms: 2xy² and -4xy²
• Constants: -5 and 3

Combine:
= (3x²y + x²y) + (2xy² - 4xy²) + (-5 + 3)
= 4x²y - 2xy² - 2

Answer: 4x²y - 2xy² - 2
```

### ⚠️ Remember: x²y ≠ xy²

```
x²y and xy² are NOT like terms!

x²y → x appears twice, y appears once
xy² → x appears once, y appears twice

These cannot be combined.
```

---

## ✏️ Solved Examples

### Example 1: Easy - Simple Addition

**Problem:** Add $(4x + 5)$ and $(3x - 2)$

**Solution:**
```
(4x + 5) + (3x - 2)
= 4x + 5 + 3x - 2
= (4x + 3x) + (5 - 2)
= 7x + 3

Answer: 7x + 3
```

### Example 2: Easy - Simple Subtraction

**Problem:** Subtract $(2x - 3)$ from $(5x + 4)$

**Solution:**
```
(5x + 4) - (2x - 3)

Step 1: Change signs of subtracted polynomial
        = 5x + 4 - 2x + 3

Step 2: Combine like terms
        = (5x - 2x) + (4 + 3)
        = 3x + 7

Answer: 3x + 7
```

### Example 3: Medium - Vertical Method

**Problem:** Add $(3x³ - 2x + 5)$ and $(x³ + 4x² - 3x - 1)$

**Solution:**
```
Arrange in columns (add 0x² placeholder):

       3x³  + 0x²  - 2x  + 5
    +  x³   + 4x²  - 3x  - 1
    ─────────────────────────
       4x³  + 4x²  - 5x  + 4

Answer: 4x³ + 4x² - 5x + 4
```

### Example 4: Medium - Multiple Operations

**Problem:** Simplify: $(5a² - 3a + 2) - (2a² + a - 4) + (a² - 2a + 1)$

**Solution:**
```
Step 1: Handle the subtraction first
        = 5a² - 3a + 2 - 2a² - a + 4 + a² - 2a + 1

Step 2: Group like terms
        = (5a² - 2a² + a²) + (-3a - a - 2a) + (2 + 4 + 1)

Step 3: Combine
        = 4a² + (-6a) + 7
        = 4a² - 6a + 7

Answer: 4a² - 6a + 7
```

### Example 5: Hard - Nested Brackets

**Problem:** Simplify: $2x² - \{3x - [x² - (2x - 1) + 3x²] - 2\}$

**Solution:**
```
Work from innermost to outermost:

Step 1: Simplify innermost ( )
        2x - 1 stays as is

Step 2: Simplify [ ]
        x² - (2x - 1) + 3x²
        = x² - 2x + 1 + 3x²
        = 4x² - 2x + 1

Step 3: Simplify { }
        3x - [4x² - 2x + 1] - 2
        = 3x - 4x² + 2x - 1 - 2
        = -4x² + 5x - 3

Step 4: Final expression
        2x² - {-4x² + 5x - 3}
        = 2x² + 4x² - 5x + 3
        = 6x² - 5x + 3

Answer: 6x² - 5x + 3
```

### Example 6: Hard - Application Problem

**Problem:** The perimeter of a triangle is $(7x² + 3x - 2)$ cm. Two sides measure $(2x² + x + 1)$ cm and $(3x² - 2x + 4)$ cm. Find the third side.

**Solution:**
```
Let the third side = P

Perimeter = Sum of all three sides
7x² + 3x - 2 = (2x² + x + 1) + (3x² - 2x + 4) + P

Step 1: Find sum of known sides
        (2x² + x + 1) + (3x² - 2x + 4)
        = 5x² - x + 5

Step 2: Third side = Perimeter - Sum of two sides
        P = (7x² + 3x - 2) - (5x² - x + 5)
        P = 7x² + 3x - 2 - 5x² + x - 5
        P = 2x² + 4x - 7

Answer: Third side = (2x² + 4x - 7) cm
```

---

## ❓ Practice Problems

### Easy Level

1. Add: $(6x - 4)$ and $(2x + 7)$

2. Subtract $(3y + 2)$ from $(5y - 1)$

3. Find: $(a² + 3a) + (2a² - a + 5)$

### Medium Level

4. Simplify: $(4m² - 3m + 7) - (2m² + 5m - 3) + (m² - m)$

5. Add using vertical method:
   - $(2x³ - 5x² + 3x - 1)$
   - $(x³ + 2x² - 4x + 6)$
   - $(-x³ + x² + x - 2)$

6. Subtract $(3xy - 2x²y + y²)$ from $(5xy + x²y - 3y²)$

### Hard Level

7. Simplify: $3a - [5a - \{7a - (4a - 3a - 2)\} + 4]$

8. If $P(x) = 2x² - 3x + 1$ and $Q(x) = x² + 4x - 5$, find:
   (a) $P(x) + Q(x)$
   (b) $P(x) - Q(x)$
   (c) $2P(x) + 3Q(x)$

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. $6x - 4 + 2x + 7 = 8x + 3$

2. $(5y - 1) - (3y + 2) = 5y - 1 - 3y - 2 = 2y - 3$

3. $a² + 3a + 2a² - a + 5 = 3a² + 2a + 5$

4. $4m² - 3m + 7 - 2m² - 5m + 3 + m² - m = 3m² - 9m + 10$

5. Vertical addition:
   ```
      2x³ - 5x² + 3x - 1
      x³  + 2x² - 4x + 6
     -x³  + x²  + x  - 2
   ─────────────────────
      2x³ - 2x² + 0x + 3 = 2x³ - 2x² + 3
   ```

6. $(5xy + x²y - 3y²) - (3xy - 2x²y + y²)$
   $= 5xy + x²y - 3y² - 3xy + 2x²y - y²$
   $= 2xy + 3x²y - 4y²$

7. Working from inside:
   - $(4a - 3a - 2) = a - 2$
   - $7a - (a - 2) = 7a - a + 2 = 6a + 2$
   - $5a - \{6a + 2\} + 4 = 5a - 6a - 2 + 4 = -a + 2$
   - $3a - [-a + 2] = 3a + a - 2 = 4a - 2$

8. (a) $P(x) + Q(x) = 3x² + x - 4$
   (b) $P(x) - Q(x) = x² - 7x + 6$
   (c) $2P(x) + 3Q(x) = 4x² - 6x + 2 + 3x² + 12x - 15 = 7x² + 6x - 13$

</details>

---

## 📋 Summary Table

| Operation | Process | Key Point |
|-----------|---------|-----------|
| **Addition** | Remove parentheses, combine like terms | Signs stay the same |
| **Subtraction** | Change signs of subtracted polynomial, then add | Change ALL signs |
| **Vertical Method** | Align like terms in columns | Use 0 placeholders |
| **Nested Brackets** | Work from innermost to outermost | Careful with signs |
| **Like Terms** | Same variables with same powers | Only these can combine |

---

## 🔄 Quick Revision Questions

1. **What must be true for two terms to be "like terms"?**

2. **When subtracting polynomials, what happens to the signs?**

3. **Is $5xy$ a like term with $5x²y$?**

4. **What is the additive inverse of $(3x² - 2x + 1)$?**

5. **In the expression $a - [b - c]$, what does this simplify to?**

6. **Can the sum of two cubic polynomials be quadratic?**

<details>
<summary>Quick Answers</summary>

1. Same variables raised to the same powers
2. All signs in the subtracted polynomial change
3. No, the power of x is different (1 vs 2)
4. $-3x² + 2x - 1$
5. $a - b + c$
6. Yes, if the x³ terms cancel (e.g., $x³ + x$ and $-x³ + 2x$)

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Only LIKE TERMS can be combined                                │
│                                                                     │
│   ★ Addition: Remove parentheses, combine like terms               │
│                                                                     │
│   ★ Subtraction: Change ALL signs, then add                        │
│                                                                     │
│   ★ Vertical method: Align by degree, use 0 placeholders          │
│                                                                     │
│   ★ Nested brackets: Work from inside out                         │
│                                                                     │
│   ★ Always write final answer in standard form                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Types of Polynomials](01-types-of-polynomials.md) | [Back to Contents](../README.md) | [Next: Multiplication of Polynomials →](03-multiplication-of-polynomials.md)
