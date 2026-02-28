# Chapter 3.3: Factoring Quadratic Trinomials

[← Previous: Grouping Method](02-grouping-method.md) | [Back to Contents](../README.md) | [Next: Difference of Squares and Sum/Difference of Cubes →](04-special-products.md)

---

## 📚 Chapter Overview

**Quadratic trinomials** are polynomials of the form $ax² + bx + c$. Factoring these expressions is one of the most important skills in algebra, essential for solving quadratic equations and simplifying expressions. This chapter covers techniques for factoring trinomials when $a = 1$ and when $a ≠ 1$.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Factor trinomials of the form $x² + bx + c$
- Factor trinomials of the form $ax² + bx + c$ using multiple methods
- Recognize patterns that make factoring easier
- Determine when a trinomial is prime (unfactorable)
- Apply the AC method for complex trinomials

---

## 1. Understanding Quadratic Trinomials

### Standard Form

A **quadratic trinomial** in standard form is:

$$ax² + bx + c$$

where:
- $a$ = leading coefficient (coefficient of $x²$)
- $b$ = middle coefficient (coefficient of $x$)
- $c$ = constant term

```
┌─────────────────────────────────────────────────────────────────────┐
│              PARTS OF A QUADRATIC TRINOMIAL                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                    3x² + 7x + 2                                     │
│                    ↑     ↑    ↑                                     │
│                    a     b    c                                     │
│                    │     │    │                                     │
│                    │     │    └── constant term                     │
│                    │     └─────── linear coefficient               │
│                    └───────────── leading coefficient              │
│                                                                     │
│   Examples:                                                         │
│   • x² + 5x + 6    (a=1, b=5, c=6)                                │
│   • 2x² - 7x + 3   (a=2, b=-7, c=3)                               │
│   • x² - 9         (a=1, b=0, c=-9) ← missing middle term         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### The Goal of Factoring

We want to write $ax² + bx + c$ as a product of two binomials:

$$ax² + bx + c = (px + q)(rx + s)$$

---

## 2. Factoring When a = 1: $x² + bx + c$

### The Pattern

When $a = 1$, we look for factors of $c$ that add up to $b$:

$$x² + bx + c = (x + m)(x + n)$$

where:
- $m \cdot n = c$ (product equals $c$)
- $m + n = b$ (sum equals $b$)

```
┌─────────────────────────────────────────────────────────────────────┐
│              FACTORING x² + bx + c                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Find two numbers m and n such that:                              │
│                                                                     │
│   ┌─────────────┐     ┌─────────────┐                              │
│   │  m × n = c  │ AND │  m + n = b  │                              │
│   └─────────────┘     └─────────────┘                              │
│                                                                     │
│   Then: x² + bx + c = (x + m)(x + n)                               │
│                                                                     │
│   Example: x² + 7x + 12                                            │
│                                                                     │
│   Need: m × n = 12 and m + n = 7                                   │
│                                                                     │
│   Factors of 12:    1×12  2×6  3×4                                 │
│   Sum:              13    8    7  ← This works!                   │
│                                                                     │
│   So m = 3, n = 4                                                  │
│   x² + 7x + 12 = (x + 3)(x + 4)                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Sign Analysis

The signs of $b$ and $c$ tell us the signs of $m$ and $n$:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SIGN ANALYSIS                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   If c > 0:                                                        │
│   • m and n have the SAME sign                                     │
│   • If b > 0: both positive  → (x + m)(x + n)                     │
│   • If b < 0: both negative  → (x - m)(x - n)                     │
│                                                                     │
│   If c < 0:                                                        │
│   • m and n have DIFFERENT signs                                  │
│   • One positive, one negative → (x + m)(x - n)                   │
│   • The larger absolute value gets the sign of b                  │
│                                                                     │
│   Summary Table:                                                   │
│   ┌─────────┬─────────┬────────────────────────────┐              │
│   │  b      │  c      │  Signs of m and n          │              │
│   ├─────────┼─────────┼────────────────────────────┤              │
│   │  +      │  +      │  Both positive             │              │
│   │  -      │  +      │  Both negative             │              │
│   │  +      │  -      │  Opposite (bigger is +)    │              │
│   │  -      │  -      │  Opposite (bigger is -)    │              │
│   └─────────┴─────────┴────────────────────────────┘              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Examples of Sign Patterns

| Trinomial | c | b | Find m,n where | Factored Form |
|-----------|---|---|----------------|---------------|
| $x² + 5x + 6$ | +6 | +5 | m·n=6, m+n=5 → 2,3 | $(x+2)(x+3)$ |
| $x² - 5x + 6$ | +6 | -5 | m·n=6, m+n=-5 → -2,-3 | $(x-2)(x-3)$ |
| $x² + x - 6$ | -6 | +1 | m·n=-6, m+n=1 → 3,-2 | $(x+3)(x-2)$ |
| $x² - x - 6$ | -6 | -1 | m·n=-6, m+n=-1 → -3,2 | $(x-3)(x+2)$ |

---

## 3. Factoring When a ≠ 1: $ax² + bx + c$

### Method 1: AC Method (Split the Middle Term)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THE AC METHOD                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   To factor: ax² + bx + c                                          │
│                                                                     │
│   Step 1: Find the product AC = a × c                              │
│                                                                     │
│   Step 2: Find two numbers m and n such that:                      │
│           • m × n = AC                                             │
│           • m + n = b                                              │
│                                                                     │
│   Step 3: Rewrite bx as mx + nx                                    │
│           ax² + mx + nx + c                                        │
│                                                                     │
│   Step 4: Factor by GROUPING                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### AC Method Example

```
Factor: 2x² + 7x + 3

Step 1: AC = 2 × 3 = 6

Step 2: Find m, n where m × n = 6 and m + n = 7
        Factors of 6: 1×6, 2×3
        Sums:         7,   5
        m = 1, n = 6 ✓

Step 3: Rewrite 7x as 1x + 6x
        2x² + 1x + 6x + 3

Step 4: Factor by grouping
        (2x² + 1x) + (6x + 3)
        x(2x + 1) + 3(2x + 1)
        (2x + 1)(x + 3)

Answer: 2x² + 7x + 3 = (2x + 1)(x + 3)
```

### Method 2: Trial and Error (Guess and Check)

```
┌─────────────────────────────────────────────────────────────────────┐
│                 TRIAL AND ERROR METHOD                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Factor: 2x² + 7x + 3                                             │
│                                                                     │
│   Step 1: List factors of a = 2:  (1, 2)                          │
│   Step 2: List factors of c = 3:  (1, 3)                          │
│                                                                     │
│   Step 3: Try combinations of the form (px + q)(rx + s)           │
│           where p × r = a and q × s = c                           │
│                                                                     │
│   Try: (2x + 1)(x + 3)                                             │
│        = 2x² + 6x + x + 3                                          │
│        = 2x² + 7x + 3 ✓                                            │
│                                                                     │
│   Works on first try! Answer: (2x + 1)(x + 3)                      │
│                                                                     │
│   Note: You might need to try several combinations               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Method 3: The Box Method (Organized AC)

```
Factor: 6x² + 11x + 4

Step 1: AC = 6 × 4 = 24
        Find: m × n = 24, m + n = 11
        m = 3, n = 8

Step 2: Fill in the box
        ┌─────────┬─────────┐
        │  6x²    │   3x    │
        ├─────────┼─────────┤
        │   8x    │    4    │
        └─────────┴─────────┘

Step 3: Find GCF of each row and column
              3x       1
            ┌─────────┬─────────┐
        2x  │  6x²    │   3x    │
            ├─────────┼─────────┤
        4   │   8x    │    4    │
            └─────────┴─────────┘

Step 4: Read factors from edges
        (2x + 4) and (3x + 1)
        
        Wait - check: 2x + 4 = 2(x + 2)
        Let's reread: (3x + 1) and (2x + 4)
        
        Actually: Row factors: 2x, 4 → but simplify to find true binomial
        Column headers give: (3x + 1)
        Row headers give: (2x + 4)? 
        
        Let me redo:
        GCF of 6x² and 3x is 3x → top row
        GCF of 8x and 4 is 4 → bottom row
        
        6x²/3x = 2x, 8x/4 = 2x → left column = 2x
        3x/3x = 1, 4/4 = 1 → right column = 1
        
        Factors: (3x + 4)(2x + 1)
        
Verify: (3x + 4)(2x + 1) = 6x² + 3x + 8x + 4 = 6x² + 11x + 4 ✓
```

---

## 4. Special Cases and Patterns

### Perfect Square Trinomials

```
┌─────────────────────────────────────────────────────────────────────┐
│              PERFECT SQUARE TRINOMIALS                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Pattern 1: a² + 2ab + b² = (a + b)²                              │
│   Pattern 2: a² - 2ab + b² = (a - b)²                              │
│                                                                     │
│   How to recognize:                                                │
│   1. First term is a perfect square: a²                            │
│   2. Last term is a perfect square: b²                             │
│   3. Middle term = 2 × (√first) × (√last)                         │
│                                                                     │
│   Example: x² + 6x + 9                                             │
│   • x² is a perfect square (x)²                                    │
│   • 9 is a perfect square (3)²                                     │
│   • 6x = 2 × x × 3 ✓                                               │
│                                                                     │
│   Therefore: x² + 6x + 9 = (x + 3)²                                │
│                                                                     │
│   Visual check:                                                    │
│            ┌──────┬──────┐                                         │
│            │  x²  │  3x  │                                         │
│            ├──────┼──────┤                                         │
│            │  3x  │   9  │                                         │
│            └──────┴──────┘                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Prime Trinomials

Not all trinomials can be factored using integers. A trinomial that cannot be factored is called **prime** or **irreducible**.

**Test:** For $x² + bx + c$, if no two integers multiply to $c$ and add to $b$, the trinomial is prime.

**Test using discriminant:** $b² - 4ac$
- If $b² - 4ac < 0$: No real factors (prime over reals)
- If $b² - 4ac$ is not a perfect square: No rational factors

Example: $x² + x + 1$
- Need: m × n = 1, m + n = 1
- Only factors of 1: (1, 1) → sum = 2 ≠ 1
- This trinomial is **prime**

---

## 5. Factoring Strategy Flowchart

```
┌─────────────────────────────────────────────────────────────────────┐
│           TRINOMIAL FACTORING STRATEGY                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   START: ax² + bx + c                                              │
│      │                                                              │
│      ▼                                                              │
│   Factor out GCF if any                                            │
│      │                                                              │
│      ▼                                                              │
│   Is a = 1?                                                        │
│      │                                                              │
│    ┌─┴─┐                                                           │
│   YES  NO                                                          │
│    │    │                                                          │
│    ▼    ▼                                                          │
│  Find m,n    Is it a perfect                                       │
│  m×n = c     square trinomial?                                     │
│  m+n = b         │                                                  │
│    │        ┌────┴────┐                                            │
│    ▼       YES       NO                                            │
│ (x+m)(x+n)  │         │                                            │
│             ▼         ▼                                             │
│         (√a·x ± √c)² Use AC Method                                 │
│                       or Trial/Error                               │
│                           │                                         │
│                           ▼                                         │
│                    Can you factor?                                 │
│                       │                                             │
│                  ┌────┴────┐                                       │
│                 YES       NO                                       │
│                  │         │                                        │
│                  ▼         ▼                                        │
│             (px+q)(rx+s)  PRIME                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✏️ Solved Examples

### Example 1: Easy - Simple Trinomial (a = 1)

**Problem:** Factor $x² + 8x + 15$

**Solution:**
```
Need two numbers that:
• Multiply to 15
• Add to 8

Factors of 15: 1×15, 3×5
Sums:          16,   8 ✓

m = 3, n = 5

x² + 8x + 15 = (x + 3)(x + 5)

Verification:
(x + 3)(x + 5) = x² + 5x + 3x + 15 = x² + 8x + 15 ✓

Answer: (x + 3)(x + 5)
```

### Example 2: Easy - Negative Middle Term

**Problem:** Factor $x² - 7x + 12$

**Solution:**
```
Need two numbers that:
• Multiply to 12 (positive)
• Add to -7 (negative)

Since c > 0 and b < 0, both numbers are negative.

Factors of 12: -1×-12, -2×-6, -3×-4
Sums:          -13,    -8,    -7 ✓

m = -3, n = -4

x² - 7x + 12 = (x - 3)(x - 4)

Answer: (x - 3)(x - 4)
```

### Example 3: Medium - Negative Constant

**Problem:** Factor $x² + 2x - 15$

**Solution:**
```
Need two numbers that:
• Multiply to -15
• Add to 2

Since c < 0, numbers have opposite signs.
Since b > 0, the larger number is positive.

Factors of -15: 1×-15, -1×15, 3×-5, -3×5
Sums:           -14,    14,   -2,    2 ✓

m = -3, n = 5

x² + 2x - 15 = (x - 3)(x + 5)

Answer: (x - 3)(x + 5)
```

### Example 4: Medium - Using AC Method

**Problem:** Factor $3x² + 10x + 8$

**Solution:**
```
AC Method:

Step 1: AC = 3 × 8 = 24

Step 2: Find m, n where m × n = 24 and m + n = 10
        Factors of 24: 1×24, 2×12, 3×8, 4×6
        Sums:          25,   14,   11,  10 ✓
        m = 4, n = 6

Step 3: Rewrite: 3x² + 4x + 6x + 8

Step 4: Factor by grouping
        (3x² + 4x) + (6x + 8)
        x(3x + 4) + 2(3x + 4)
        (3x + 4)(x + 2)

Verification:
(3x + 4)(x + 2) = 3x² + 6x + 4x + 8 = 3x² + 10x + 8 ✓

Answer: (3x + 4)(x + 2)
```

### Example 5: Hard - Perfect Square Trinomial

**Problem:** Factor $4x² - 20x + 25$

**Solution:**
```
Check for perfect square pattern:

• First term: 4x² = (2x)² ✓
• Last term: 25 = (5)² ✓
• Middle term: -20x = 2 × (2x) × (-5) = -20x ✓

This is a perfect square trinomial!

4x² - 20x + 25 = (2x - 5)²

Verification:
(2x - 5)² = 4x² - 20x + 25 ✓

Answer: (2x - 5)²
```

### Example 6: Hard - Complete Factoring

**Problem:** Factor completely: $2x³ + 10x² + 12x$

**Solution:**
```
Step 1: Factor out GCF
        GCF = 2x
        2x(x² + 5x + 6)

Step 2: Factor the trinomial x² + 5x + 6
        Need: m × n = 6, m + n = 5
        m = 2, n = 3
        x² + 5x + 6 = (x + 2)(x + 3)

Step 3: Write complete factorization
        2x(x + 2)(x + 3)

Answer: 2x(x + 2)(x + 3)
```

---

## ❓ Practice Problems

### Easy Level

1. Factor: $x² + 9x + 20$

2. Factor: $x² - 6x + 8$

3. Factor: $x² + 3x - 10$

### Medium Level

4. Factor: $2x² + 5x + 3$

5. Factor: $3x² - 11x + 6$

6. Factor: $x² - 10x + 25$

### Hard Level

7. Factor completely: $5x³ - 45x² + 100x$

8. Factor: $6x² + x - 12$

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. m×n=20, m+n=9 → 4,5: **$(x + 4)(x + 5)$**

2. m×n=8, m+n=-6 → -2,-4: **$(x - 2)(x - 4)$**

3. m×n=-10, m+n=3 → 5,-2: **$(x + 5)(x - 2)$**

4. AC=6, m+n=5, m×n=6 → 2,3
   2x² + 2x + 3x + 3 = (2x + 3)(x + 1): **$(2x + 3)(x + 1)$**

5. AC=18, m+n=-11, m×n=18 → -2,-9
   3x² - 2x - 9x + 6 = (3x - 2)(x - 3): **$(3x - 2)(x - 3)$**

6. Perfect square: (x)², (5)², 2×x×5=10x: **$(x - 5)²$**

7. GCF=5x: 5x(x² - 9x + 20)
   Factor trinomial: 5x(x - 4)(x - 5): **$5x(x - 4)(x - 5)$**

8. AC=6×(-12)=-72, m+n=1, m×n=-72 → 9,-8
   6x² + 9x - 8x - 12 = (3x - 4)(2x + 3): **$(3x - 4)(2x + 3)$**

</details>

---

## 📋 Summary Table

| Form | Method | Key Steps |
|------|--------|-----------|
| $x² + bx + c$ | Find factors | m×n=c, m+n=b → (x+m)(x+n) |
| $ax² + bx + c$ | AC Method | Find m×n=ac, m+n=b, then group |
| $a² + 2ab + b²$ | Perfect square | = $(a+b)²$ |
| $a² - 2ab + b²$ | Perfect square | = $(a-b)²$ |

---

## 🔄 Quick Revision Questions

1. **For $x² + bx + c$, what two conditions must m and n satisfy?**

2. **If c is positive and b is negative, what are the signs of the factors?**

3. **What is the AC in the AC method for $5x² - 7x + 2$?**

4. **Identify: Is $x² + 4x + 4$ a perfect square trinomial?**

5. **Factor: $x² - 4x - 5$**

6. **When is a trinomial called "prime"?**

<details>
<summary>Quick Answers</summary>

1. m × n = c and m + n = b
2. Both negative
3. AC = 5 × 2 = 10
4. Yes: $(x + 2)²$
5. $(x - 5)(x + 1)$
6. When it cannot be factored using integers/rationals

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Always factor out GCF first                                   │
│                                                                     │
│   ★ For x² + bx + c: Find m,n where m×n=c and m+n=b               │
│                                                                     │
│   ★ For ax² + bx + c: Use AC method                               │
│     - Find m×n = ac, m+n = b                                       │
│     - Rewrite and factor by grouping                              │
│                                                                     │
│   ★ Check for perfect square patterns                             │
│                                                                     │
│   ★ Use sign analysis to determine factor signs                   │
│                                                                     │
│   ★ Verify by multiplying factors back                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Grouping Method](02-grouping-method.md) | [Back to Contents](../README.md) | [Next: Difference of Squares and Sum/Difference of Cubes →](04-special-products.md)
