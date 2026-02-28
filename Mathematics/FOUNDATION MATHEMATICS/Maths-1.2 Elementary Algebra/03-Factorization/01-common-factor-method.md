# Chapter 3.1: Common Factor Method

[← Previous: Remainder and Factor Theorems](../02-Polynomials/05-remainder-and-factor-theorems.md) | [Back to Contents](../README.md) | [Next: Grouping Method →](02-grouping-method.md)

---

## 📚 Chapter Overview

**Factorization** is the process of expressing a polynomial as a product of its factors. The **Common Factor Method** is the most fundamental factoring technique—always check for common factors first before attempting any other method.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Identify the greatest common factor (GCF) of terms
- Factor out numerical common factors
- Factor out variable common factors
- Extract the highest common factor from polynomials
- Apply the common factor method as the first step in factorization

---

## 1. What is Factorization?

### Definition

**Factorization** (or factoring) is the reverse of multiplication—breaking down an expression into a product of simpler expressions.

```
┌─────────────────────────────────────────────────────────────────────┐
│            MULTIPLICATION vs FACTORIZATION                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   MULTIPLICATION (Expanding)                                       │
│   ────────────────────────────                                     │
│   3x(x + 2)  ──────►  3x² + 6x                                    │
│   Factors         Product                                          │
│                                                                     │
│   FACTORIZATION (Factoring)                                        │
│   ─────────────────────────                                        │
│   3x² + 6x  ──────►  3x(x + 2)                                    │
│   Expression      Factors                                          │
│                                                                     │
│        EXPAND                                                       │
│   ┌──────────────►┐                                                │
│   │               │                                                │
│   3x(x + 2)  ⟷  3x² + 6x                                         │
│   │               │                                                │
│   └──────────────◄┘                                                │
│        FACTOR                                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Why Factorization Matters

```
┌─────────────────────────────────────────────────────────────────────┐
│              IMPORTANCE OF FACTORIZATION                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. SOLVING EQUATIONS                                             │
│      x² - 5x + 6 = 0                                               │
│      (x - 2)(x - 3) = 0                                            │
│      x = 2 or x = 3                                                │
│                                                                     │
│   2. SIMPLIFYING FRACTIONS                                         │
│      x² - 4     (x+2)(x-2)                                         │
│      ────── = ─────────── = x - 2                                 │
│       x + 2       x + 2                                            │
│                                                                     │
│   3. FINDING ROOTS/ZEROS                                           │
│      If P(x) = (x - a)(x - b), then roots are x = a, b            │
│                                                                     │
│   4. GRAPHING POLYNOMIALS                                          │
│      Factored form reveals x-intercepts                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Greatest Common Factor (GCF)

### Definition

The **Greatest Common Factor** (GCF) of two or more terms is the largest factor that divides all terms exactly.

### Finding the GCF

```
┌─────────────────────────────────────────────────────────────────────┐
│             STEPS TO FIND THE GCF                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Step 1: Find GCF of NUMERICAL coefficients                      │
│           • List factors or use prime factorization               │
│           • Take the largest common factor                         │
│                                                                     │
│   Step 2: Find GCF of VARIABLE parts                              │
│           • For each variable, take the LOWEST power              │
│             that appears in ALL terms                              │
│                                                                     │
│   Step 3: Multiply numerical and variable GCFs                    │
│                                                                     │
│   Example: Find GCF of 12x³y² and 18x²y³                          │
│                                                                     │
│   Numerical: GCF(12, 18) = 6                                       │
│   Variables: x → min(3, 2) = x²                                   │
│              y → min(2, 3) = y²                                   │
│                                                                     │
│   GCF = 6x²y²                                                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Examples of Finding GCF

| Terms | Numerical GCF | Variable GCF | Overall GCF |
|-------|---------------|--------------|-------------|
| $6x, 9x²$ | GCF(6,9) = 3 | x^min(1,2) = x | 3x |
| $4a²b, 8ab², 12ab$ | GCF(4,8,12) = 4 | a^min(2,1,1) · b^min(1,2,1) = ab | 4ab |
| $15x³, 25x², 10x$ | GCF(15,25,10) = 5 | x^min(3,2,1) = x | 5x |
| $-6m²n, 9mn², -12mn$ | GCF(6,9,12) = 3 | mn | 3mn |

---

## 3. Factoring Out the GCF

### The Process

```
┌─────────────────────────────────────────────────────────────────────┐
│            FACTORING OUT THE GCF                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Expression: 6x² + 9x                                             │
│                                                                     │
│   Step 1: Find GCF                                                 │
│           GCF of 6x² and 9x = 3x                                   │
│                                                                     │
│   Step 2: Divide each term by GCF                                  │
│           6x² ÷ 3x = 2x                                            │
│           9x ÷ 3x = 3                                              │
│                                                                     │
│   Step 3: Write as product                                         │
│           6x² + 9x = 3x(2x + 3)                                    │
│                                                                     │
│   Verification:                                                     │
│           3x(2x + 3) = 6x² + 9x  ✓                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### The Reverse Distribution (Factoring)

$$ab + ac = a(b + c)$$

This is the distributive property used in reverse.

```
Visual Representation:

  6x² + 9x
    ↓   ↓
  3x·2x + 3x·3    ← Write each term showing the GCF
    ↓     ↓
  3x(2x + 3)      ← Factor out the GCF
```

---

## 4. Factoring with Negative Signs

### When the Leading Term is Negative

Sometimes it's useful to factor out a negative GCF.

```
┌─────────────────────────────────────────────────────────────────────┐
│           FACTORING WITH NEGATIVES                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Example: -4x² + 8x - 12                                          │
│                                                                     │
│   Option 1: Factor out 4                                           │
│             = 4(-x² + 2x - 3)                                      │
│                                                                     │
│   Option 2: Factor out -4 (preferred when leading term is negative)│
│             = -4(x² - 2x + 3)                                      │
│                                                                     │
│   Both are correct! Option 2 gives a positive leading term inside. │
│                                                                     │
│   Rule of thumb:                                                   │
│   • If the first term is negative, consider factoring out         │
│     the negative of the GCF                                        │
│   • This often makes further factoring easier                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Sign Changes When Factoring -1

When you factor out $-1$ (or any negative number), **all signs change**:

| Original | Factor out -1 |
|----------|---------------|
| $-a + b$ | $-(a - b)$ |
| $-x - y$ | $-(x + y)$ |
| $-3x² + 6x - 9$ | $-3(x² - 2x + 3)$ |

---

## 5. Factoring Polynomials Completely

### The Complete Factoring Strategy

```
┌─────────────────────────────────────────────────────────────────────┐
│           COMPLETE FACTORIZATION STRATEGY                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ALWAYS START BY FACTORING OUT THE GCF!                           │
│                                                                     │
│   1. Factor out the GCF (this chapter)                             │
│   2. Look at what remains:                                         │
│      • 2 terms → difference of squares? sum/diff of cubes?        │
│      • 3 terms → trinomial factoring?                              │
│      • 4+ terms → grouping method?                                 │
│   3. Check if any factor can be factored further                  │
│   4. Verify by multiplying                                         │
│                                                                     │
│   Flowchart:                                                       │
│                                                                     │
│   START                                                             │
│     │                                                               │
│     ▼                                                               │
│   ┌─────────────────┐                                              │
│   │ Factor out GCF  │ ◄── ALWAYS DO THIS FIRST!                   │
│   └────────┬────────┘                                              │
│            │                                                        │
│            ▼                                                        │
│   Check remaining factor(s)                                        │
│            │                                                        │
│            ▼                                                        │
│   Can it be factored more?                                         │
│       │           │                                                 │
│      YES         NO                                                 │
│       │           │                                                 │
│       ▼           ▼                                                 │
│    Factor       DONE                                                │
│    again                                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✏️ Solved Examples

### Example 1: Easy - Simple GCF

**Problem:** Factor $8x + 12$

**Solution:**
```
Step 1: Find GCF
        GCF of 8 and 12 = 4
        No common variables
        GCF = 4

Step 2: Factor out GCF
        8x ÷ 4 = 2x
        12 ÷ 4 = 3

        8x + 12 = 4(2x + 3)

Verification: 4(2x + 3) = 8x + 12 ✓

Answer: 4(2x + 3)
```

### Example 2: Easy - Variable GCF

**Problem:** Factor $x³ + x²$

**Solution:**
```
Step 1: Find GCF
        Numerical: GCF(1, 1) = 1
        Variable: x^min(3,2) = x²
        GCF = x²

Step 2: Factor out GCF
        x³ ÷ x² = x
        x² ÷ x² = 1

        x³ + x² = x²(x + 1)

Answer: x²(x + 1)
```

### Example 3: Medium - Multiple Variables

**Problem:** Factor $15a²b³ - 25ab² + 10ab$

**Solution:**
```
Step 1: Find GCF
        Numerical: GCF(15, 25, 10) = 5
        Variable a: a^min(2,1,1) = a
        Variable b: b^min(3,2,1) = b
        GCF = 5ab

Step 2: Factor out GCF
        15a²b³ ÷ 5ab = 3ab²
        -25ab² ÷ 5ab = -5b
        10ab ÷ 5ab = 2

        15a²b³ - 25ab² + 10ab = 5ab(3ab² - 5b + 2)

Verification: 
5ab(3ab²) = 15a²b³ ✓
5ab(-5b) = -25ab² ✓
5ab(2) = 10ab ✓

Answer: 5ab(3ab² - 5b + 2)
```

### Example 4: Medium - Negative Leading Term

**Problem:** Factor $-6x² + 18x - 24$

**Solution:**
```
Step 1: Find GCF
        GCF of 6, 18, 24 = 6
        No common variables
        Since leading term is negative, factor out -6

Step 2: Factor out -6
        -6x² ÷ (-6) = x²
        18x ÷ (-6) = -3x
        -24 ÷ (-6) = 4

        -6x² + 18x - 24 = -6(x² - 3x + 4)

Verification: 
-6(x²) = -6x² ✓
-6(-3x) = 18x ✓
-6(4) = -24 ✓

Answer: -6(x² - 3x + 4)
```

### Example 5: Hard - Binomial Common Factor

**Problem:** Factor $3x(a + b) + 5y(a + b)$

**Solution:**
```
Notice that (a + b) is a common factor!

Think of (a + b) as a single unit, let's call it "u"
3x(u) + 5y(u) = u(3x + 5y)

Substituting back:
= (a + b)(3x + 5y)

Answer: (a + b)(3x + 5y)

This is an example of a BINOMIAL common factor!
```

### Example 6: Hard - Complete Factorization

**Problem:** Factor completely: $2x³ - 8x$

**Solution:**
```
Step 1: Factor out GCF
        GCF = 2x
        2x³ - 8x = 2x(x² - 4)

Step 2: Look at what remains
        x² - 4 is a DIFFERENCE OF SQUARES!
        x² - 4 = (x + 2)(x - 2)

Step 3: Write complete factorization
        2x³ - 8x = 2x(x + 2)(x - 2)

Verification:
2x(x + 2)(x - 2) = 2x(x² - 4) = 2x³ - 8x ✓

Answer: 2x(x + 2)(x - 2)
```

---

## ❓ Practice Problems

### Easy Level

1. Factor: $6x + 18$

2. Factor: $5y² - 15y$

3. Factor: $12a³ + 4a²$

### Medium Level

4. Factor: $8x²y - 12xy² + 4xy$

5. Factor: $-9m² + 27m - 18$

6. Factor: $2x(y - 3) + 5(y - 3)$

### Hard Level

7. Factor completely: $3x³ - 27x$

8. Factor completely: $-2a⁴ + 8a³ - 8a²$

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. GCF = 6: **$6(x + 3)$**

2. GCF = 5y: **$5y(y - 3)$**

3. GCF = 4a²: **$4a²(3a + 1)$**

4. GCF = 4xy: **$4xy(2x - 3y + 1)$**

5. Factor out -9: **$-9(m² - 3m + 2) = -9(m - 1)(m - 2)$**

6. Common factor (y - 3): **$(y - 3)(2x + 5)$**

7. GCF = 3x, then difference of squares:
   $3x(x² - 9) = $ **$3x(x + 3)(x - 3)$**

8. GCF = -2a², then factor quadratic:
   $-2a²(a² - 4a + 4) = -2a²(a - 2)² = $ **$-2a²(a - 2)²$**

</details>

---

## 📋 Summary Table

| Expression Type | GCF | Factored Form |
|-----------------|-----|---------------|
| $ax + ay$ | $a$ | $a(x + y)$ |
| $x² + x$ | $x$ | $x(x + 1)$ |
| $6x² - 9x$ | $3x$ | $3x(2x - 3)$ |
| $-ax - ay$ | $-a$ | $-a(x + y)$ |
| $a(x) + b(x)$ | $(x)$ | $(x)(a + b)$ |

---

## 🔄 Quick Revision Questions

1. **What is the first step in any factorization problem?**

2. **Find the GCF of $12x³y$ and $18x²y²$**

3. **Factor: $7x + 7$**

4. **When should you factor out a negative GCF?**

5. **Factor: $x(a+1) + y(a+1)$**

6. **Is $x² + 3$ factorable using real numbers?**

<details>
<summary>Quick Answers</summary>

1. Factor out the GCF
2. GCF = 6x²y
3. $7(x + 1)$
4. When the leading term is negative
5. $(a + 1)(x + y)$
6. No, it cannot be factored over real numbers

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ ALWAYS factor out the GCF first                               │
│                                                                     │
│   ★ GCF = GCF of coefficients × lowest power of each variable     │
│                                                                     │
│   ★ Factor out negative GCF if leading term is negative           │
│                                                                     │
│   ★ Check if the remaining expression can be factored further     │
│                                                                     │
│   ★ Verify by multiplying to get the original expression          │
│                                                                     │
│   ★ Binomials can also be common factors                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Remainder and Factor Theorems](../02-Polynomials/05-remainder-and-factor-theorems.md) | [Back to Contents](../README.md) | [Next: Grouping Method →](02-grouping-method.md)
