# Chapter 3.5: Factoring by Substitution

[← Previous: Difference of Squares and Sum/Difference of Cubes](04-special-products.md) | [Back to Contents](../README.md) | [Next: One Variable Linear Equations →](../04-Linear-Equations/01-one-variable-equations.md)

---

## 📚 Chapter Overview

**Factoring by substitution** is a powerful technique that simplifies complex expressions by temporarily replacing a repeated pattern with a single variable. This makes complicated expressions look like simpler, familiar forms that we already know how to factor.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Identify expressions suitable for substitution
- Choose appropriate substitutions to simplify factoring
- Factor expressions that are "quadratic in form"
- Apply substitution to expressions with complex terms
- Combine substitution with other factoring techniques

---

## 1. The Substitution Concept

### What is Substitution?

When an expression contains a repeated complex term, we can replace that term with a simpler variable, factor, then substitute back.

```
┌─────────────────────────────────────────────────────────────────────┐
│              THE SUBSTITUTION PROCESS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Original: (x + 1)² + 5(x + 1) + 6                                │
│                                                                     │
│   Step 1: IDENTIFY the repeated pattern                            │
│           The pattern (x + 1) appears multiple times               │
│                                                                     │
│   Step 2: SUBSTITUTE                                               │
│           Let u = (x + 1)                                          │
│           Expression becomes: u² + 5u + 6                          │
│                                                                     │
│   Step 3: FACTOR the simplified expression                        │
│           u² + 5u + 6 = (u + 2)(u + 3)                            │
│                                                                     │
│   Step 4: SUBSTITUTE BACK                                          │
│           Replace u with (x + 1):                                  │
│           (x + 1 + 2)(x + 1 + 3)                                   │
│           = (x + 3)(x + 4)                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Why Substitution Works

Substitution doesn't change the mathematical structure—it just makes patterns easier to see.

```
Visual Comparison:

BEFORE SUBSTITUTION:
(x + 1)² + 5(x + 1) + 6
   ↓          ↓
 [complex] [complex]  → Hard to see the pattern

AFTER SUBSTITUTION (u = x + 1):
   u²    +   5u   + 6
   ↓          ↓
 simple   simple  → Easy! It's a quadratic trinomial!
```

---

## 2. Expressions "Quadratic in Form"

### Definition

An expression is **quadratic in form** if it can be written as:

$$A(expression)² + B(expression) + C$$

where the same expression appears with powers in ratio 2:1.

### Recognizing Quadratic Form

```
┌─────────────────────────────────────────────────────────────────────┐
│         RECOGNIZING QUADRATIC FORM                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Pattern: The expression should have:                             │
│   • A term with (something)²                                       │
│   • A term with (something)¹                                       │
│   • A constant term                                                │
│                                                                     │
│   Examples:                                                         │
│                                                                     │
│   1. x⁴ + 5x² + 6                                                  │
│      = (x²)² + 5(x²) + 6     ← quadratic in x²                    │
│      Let u = x²                                                    │
│                                                                     │
│   2. x⁶ - 7x³ + 10                                                 │
│      = (x³)² - 7(x³) + 10    ← quadratic in x³                    │
│      Let u = x³                                                    │
│                                                                     │
│   3. (x-1)² + 2(x-1) - 15                                          │
│      Already in form         ← quadratic in (x-1)                  │
│      Let u = (x-1)                                                 │
│                                                                     │
│   4. x - 3√x + 2                                                   │
│      = (√x)² - 3(√x) + 2     ← quadratic in √x                    │
│      Let u = √x                                                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Power Ratio Pattern

For an expression to be quadratic in form, look for powers in 2:1 ratio:

| Expression | Higher Power | Lower Power | Ratio | Substitution |
|------------|-------------|-------------|-------|--------------|
| $x⁴ + x² + 1$ | 4 | 2 | 4:2 = 2:1 | u = x² |
| $x⁶ + x³ - 2$ | 6 | 3 | 6:3 = 2:1 | u = x³ |
| $x⁸ + x⁴$ | 8 | 4 | 8:4 = 2:1 | u = x⁴ |
| $x + √x$ | 1 | ½ | 1:½ = 2:1 | u = √x |
| $x^(2/3) + x^(1/3)$ | 2/3 | 1/3 | 2:1 | u = x^(1/3) |

---

## 3. Step-by-Step Substitution Method

### Complete Process

```
┌─────────────────────────────────────────────────────────────────────┐
│           FACTORING BY SUBSTITUTION - STEPS                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Step 1: IDENTIFY the substitution                               │
│           • Look for repeated expressions                          │
│           • Check for powers in 2:1 ratio                         │
│                                                                     │
│   Step 2: DEFINE the substitution                                 │
│           • Write: Let u = [expression]                           │
│           • Note: u² = [expression]²                              │
│                                                                     │
│   Step 3: REWRITE the original in terms of u                      │
│           • Replace all occurrences                                │
│           • Should now look like a simple quadratic               │
│                                                                     │
│   Step 4: FACTOR the expression in u                              │
│           • Use appropriate factoring technique                   │
│                                                                     │
│   Step 5: SUBSTITUTE BACK                                          │
│           • Replace u with the original expression               │
│           • Simplify each factor if possible                      │
│                                                                     │
│   Step 6: CHECK for further factoring                             │
│           • Can any factor be factored more?                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Common Substitution Patterns

### Pattern 1: Higher Powers of x

```
Factor: x⁴ - 5x² + 4

Let u = x²

Then: u² - 5u + 4

Factor: (u - 1)(u - 4)

Substitute back:
(x² - 1)(x² - 4)

Factor further (both are difference of squares!):
(x + 1)(x - 1)(x + 2)(x - 2)

Answer: (x + 1)(x - 1)(x + 2)(x - 2)
```

### Pattern 2: Binomial Expressions

```
Factor: (x + 3)² - 7(x + 3) + 12

Let u = (x + 3)

Then: u² - 7u + 12

Factor: (u - 3)(u - 4)

Substitute back:
(x + 3 - 3)(x + 3 - 4)
= (x)(x - 1)

Answer: x(x - 1)
```

### Pattern 3: Radical Expressions

```
Factor: x - 5√x + 6

Let u = √x  (so x = u²)

Then: u² - 5u + 6

Factor: (u - 2)(u - 3)

Substitute back:
(√x - 2)(√x - 3)

Answer: (√x - 2)(√x - 3)
```

### Pattern 4: Fraction Exponents

```
Factor: x^(2/3) - 5x^(1/3) + 6

Let u = x^(1/3)  (so x^(2/3) = u²)

Then: u² - 5u + 6

Factor: (u - 2)(u - 3)

Substitute back:
(x^(1/3) - 2)(x^(1/3) - 3)

Answer: (x^(1/3) - 2)(x^(1/3) - 3)
```

---

## 5. Substitution with Special Products

### Difference of Squares via Substitution

```
Factor: (x + 2)² - 9

Method 1: Direct (difference of squares)
= (x + 2)² - 3²
= [(x + 2) + 3][(x + 2) - 3]
= (x + 5)(x - 1)

Method 2: Substitution
Let u = (x + 2)
u² - 9 = (u + 3)(u - 3)
= (x + 2 + 3)(x + 2 - 3)
= (x + 5)(x - 1)

Same answer: (x + 5)(x - 1)
```

### Sum/Difference of Cubes via Substitution

```
Factor: (x + 1)³ + 8

Let u = (x + 1)

u³ + 8 = u³ + 2³
       = (u + 2)(u² - 2u + 4)

Substitute back:
= (x + 1 + 2)[(x + 1)² - 2(x + 1) + 4]
= (x + 3)[x² + 2x + 1 - 2x - 2 + 4]
= (x + 3)(x² + 3)

Answer: (x + 3)(x² + 3)
```

---

## 6. Complex Substitutions

### Double Substitution

Sometimes you may need to substitute twice.

```
Factor: x⁸ - 1

Step 1: Let u = x⁴
        u² - 1 = (u + 1)(u - 1)
        = (x⁴ + 1)(x⁴ - 1)

Step 2: Factor x⁴ - 1 further
        Let v = x²
        v² - 1 = (v + 1)(v - 1)
        = (x² + 1)(x² - 1)
        = (x² + 1)(x + 1)(x - 1)

Complete factorization:
x⁸ - 1 = (x⁴ + 1)(x² + 1)(x + 1)(x - 1)
```

### Nested Expressions

```
Factor: (x² + x)² - 8(x² + x) + 12

Let u = x² + x

u² - 8u + 12 = (u - 2)(u - 6)

Substitute back:
(x² + x - 2)(x² + x - 6)

Factor each trinomial:
• x² + x - 2 = (x + 2)(x - 1)
• x² + x - 6 = (x + 3)(x - 2)

Answer: (x + 2)(x - 1)(x + 3)(x - 2)
```

---

## ✏️ Solved Examples

### Example 1: Easy - Basic Substitution

**Problem:** Factor $x⁴ + 7x² + 10$

**Solution:**
```
Step 1: Identify - this is quadratic in x²

Step 2: Let u = x²
        Expression becomes: u² + 7u + 10

Step 3: Factor
        u² + 7u + 10 = (u + 2)(u + 5)

Step 4: Substitute back
        (x² + 2)(x² + 5)

Step 5: Check for further factoring
        Neither factor is a difference of squares.
        Both are sums → prime over reals

Answer: (x² + 2)(x² + 5)
```

### Example 2: Easy - Binomial Substitution

**Problem:** Factor $(x - 2)² + 5(x - 2) + 6$

**Solution:**
```
Step 1: Let u = (x - 2)
        Expression: u² + 5u + 6

Step 2: Factor
        (u + 2)(u + 3)

Step 3: Substitute back
        (x - 2 + 2)(x - 2 + 3)
        = (x)(x + 1)
        = x(x + 1)

Answer: x(x + 1)
```

### Example 3: Medium - With Further Factoring

**Problem:** Factor $x⁴ - 13x² + 36$

**Solution:**
```
Step 1: Let u = x²
        Expression: u² - 13u + 36

Step 2: Factor (find factors of 36 that sum to -13)
        -4 × -9 = 36, -4 + -9 = -13 ✓
        (u - 4)(u - 9)

Step 3: Substitute back
        (x² - 4)(x² - 9)

Step 4: Factor further - both are difference of squares!
        x² - 4 = (x + 2)(x - 2)
        x² - 9 = (x + 3)(x - 3)

Complete factorization:
(x + 2)(x - 2)(x + 3)(x - 3)

Answer: (x + 2)(x - 2)(x + 3)(x - 3)
```

### Example 4: Medium - Radical Substitution

**Problem:** Factor $x - 7√x + 12$

**Solution:**
```
Step 1: Recognize as quadratic in √x
        Let u = √x  (so x = u²)

Step 2: Rewrite
        u² - 7u + 12

Step 3: Factor
        (u - 3)(u - 4)

Step 4: Substitute back
        (√x - 3)(√x - 4)

Note: For this to be defined, x ≥ 0

Answer: (√x - 3)(√x - 4)
```

### Example 5: Hard - Sum of Cubes

**Problem:** Factor $(2x - 1)³ + 27$

**Solution:**
```
Step 1: Let u = (2x - 1)
        Expression: u³ + 27 = u³ + 3³

Step 2: Apply sum of cubes formula
        a³ + b³ = (a + b)(a² - ab + b²)
        
        u³ + 27 = (u + 3)(u² - 3u + 9)

Step 3: Substitute back
        = (2x - 1 + 3)[(2x - 1)² - 3(2x - 1) + 9]
        = (2x + 2)[4x² - 4x + 1 - 6x + 3 + 9]
        = (2x + 2)(4x² - 10x + 13)

Step 4: Factor further
        2x + 2 = 2(x + 1)

Answer: 2(x + 1)(4x² - 10x + 13)
```

### Example 6: Hard - Complex Nested Expression

**Problem:** Factor $(x² + 3x)² + 2(x² + 3x) - 8$

**Solution:**
```
Step 1: Let u = x² + 3x
        Expression: u² + 2u - 8

Step 2: Factor
        (u + 4)(u - 2)

Step 3: Substitute back
        (x² + 3x + 4)(x² + 3x - 2)

Step 4: Check each factor
        x² + 3x + 4: discriminant = 9 - 16 = -7 < 0 → prime
        x² + 3x - 2: discriminant = 9 + 8 = 17 → not a perfect square → prime over integers

Answer: (x² + 3x + 4)(x² + 3x - 2)

Note: If factoring over irrational numbers is allowed:
x² + 3x - 2 = (x + (3+√17)/2)(x + (3-√17)/2)
```

---

## ❓ Practice Problems

### Easy Level

1. Factor: $x⁴ + 3x² + 2$

2. Factor: $(y + 1)² + 4(y + 1) + 3$

3. Factor: $x⁶ - 9$

### Medium Level

4. Factor: $x⁴ - 10x² + 9$

5. Factor: $a - 4√a + 3$

6. Factor: $(x - 5)² - 4(x - 5) - 21$

### Hard Level

7. Factor completely: $(x² - x)² - 18(x² - x) + 72$

8. Factor: $x⁴ + x² + 1$ (Hint: Add and subtract x²)

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. Let u = x²: u² + 3u + 2 = (u + 1)(u + 2)
   **$(x² + 1)(x² + 2)$**

2. Let u = y + 1: u² + 4u + 3 = (u + 1)(u + 3)
   = (y + 2)(y + 4): **$(y + 2)(y + 4)$**

3. x⁶ - 9 = (x³)² - 3²
   = (x³ + 3)(x³ - 3): **$(x³ + 3)(x³ - 3)$**

4. Let u = x²: u² - 10u + 9 = (u - 1)(u - 9)
   = (x² - 1)(x² - 9) = (x+1)(x-1)(x+3)(x-3)
   **$(x + 1)(x - 1)(x + 3)(x - 3)$**

5. Let u = √a: u² - 4u + 3 = (u - 1)(u - 3)
   **$(√a - 1)(√a - 3)$**

6. Let u = x - 5: u² - 4u - 21 = (u - 7)(u + 3)
   = (x - 12)(x - 2): **$(x - 12)(x - 2)$**

7. Let u = x² - x: u² - 18u + 72 = (u - 6)(u - 12)
   = (x² - x - 6)(x² - x - 12)
   = (x - 3)(x + 2)(x - 4)(x + 3)
   **$(x - 3)(x + 2)(x - 4)(x + 3)$**

8. x⁴ + x² + 1 = x⁴ + 2x² + 1 - x² = (x² + 1)² - x²
   = (x² + 1 + x)(x² + 1 - x)
   **$(x² + x + 1)(x² - x + 1)$**

</details>

---

## 📋 Summary Table

| Expression Type | Substitution | Example |
|-----------------|--------------|---------|
| Higher even powers | u = x^(half power) | x⁴: u = x² |
| Binomial squared | u = (binomial) | (x+1)²: u = x+1 |
| Radical | u = √x | x - √x: u = √x |
| Fractional exponents | u = x^(lower power) | x^(2/3): u = x^(1/3) |
| Complex expressions | u = (whole expression) | (x²+x)²: u = x²+x |

---

## 🔄 Quick Revision Questions

1. **When is an expression "quadratic in form"?**

2. **What substitution would you use for $x⁶ + 5x³ + 6$?**

3. **Factor: $(x + 2)² - 9$**

4. **What is the ratio of powers for quadratic form?**

5. **What substitution simplifies $x - 6√x + 8$?**

6. **After substituting u = x², factor $u² - 5u + 6$, then give final answer.**

<details>
<summary>Quick Answers</summary>

1. When it has form A(expr)² + B(expr) + C
2. u = x³
3. (x + 5)(x - 1)
4. 2:1
5. u = √x
6. (u - 2)(u - 3) = (x² - 2)(x² - 3)

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Substitution simplifies complex expressions                   │
│                                                                     │
│   ★ Look for repeated patterns or 2:1 power ratios                │
│                                                                     │
│   ★ Steps: Identify → Substitute → Factor → Back-substitute       │
│                                                                     │
│   ★ Always check if factors can be factored further               │
│                                                                     │
│   ★ Common substitutions:                                          │
│     • u = x² for x⁴ terms                                         │
│     • u = (binomial) for squared binomials                        │
│     • u = √x for radical expressions                              │
│                                                                     │
│   ★ Remember to simplify each factor after substituting back      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Difference of Squares and Sum/Difference of Cubes](04-special-products.md) | [Back to Contents](../README.md) | [Next: One Variable Linear Equations →](../04-Linear-Equations/01-one-variable-equations.md)
