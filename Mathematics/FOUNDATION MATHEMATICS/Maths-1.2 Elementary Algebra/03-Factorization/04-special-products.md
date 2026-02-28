# Chapter 3.4: Difference of Squares and Sum/Difference of Cubes

[← Previous: Factoring Quadratic Trinomials](03-factoring-quadratic-trinomials.md) | [Back to Contents](../README.md) | [Next: Factoring by Substitution →](05-factoring-by-substitution.md)

---

## 📚 Chapter Overview

**Special products** are polynomial expressions that follow recognizable patterns and can be factored using formulas rather than trial-and-error methods. This chapter covers three essential patterns: the difference of squares, the sum of cubes, and the difference of cubes.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Recognize and factor the difference of two squares
- Understand why the sum of squares doesn't factor (over reals)
- Factor the sum of two cubes
- Factor the difference of two cubes
- Apply these patterns to complex expressions
- Combine special product patterns with other factoring techniques

---

## 1. Difference of Two Squares

### The Formula

$$a² - b² = (a + b)(a - b)$$

```
┌─────────────────────────────────────────────────────────────────────┐
│           DIFFERENCE OF TWO SQUARES                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Pattern: a² - b² = (a + b)(a - b)                                │
│                                                                     │
│   Requirements:                                                    │
│   1. Two terms                                                     │
│   2. Subtraction (difference)                                      │
│   3. Both terms are perfect squares                                │
│                                                                     │
│   Derivation (FOIL):                                               │
│   (a + b)(a - b) = a² - ab + ab - b²                              │
│                  = a² - b²                                         │
│                                                                     │
│   Visual:                                                          │
│   ┌─────────────────────────────────────────────────┐              │
│   │                        a                         │              │
│   │    ┌─────────────────────────────────────────┐  │              │
│   │    │                                         │  │              │
│   │    │            a²                           │  │ a            │
│   │    │                     ┌──────┐            │  │              │
│   │    │                     │  b²  │            │  │              │
│   │    │                     └──────┘            │  │              │
│   │    └─────────────────────────────────────────┘  │              │
│   │    Area = a² - b² = (a+b)(a-b)                  │              │
│   └─────────────────────────────────────────────────┘              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Recognizing Perfect Squares

| Expression | Perfect Square? | As a² |
|------------|----------------|-------|
| $x²$ | Yes | $(x)²$ |
| $9$ | Yes | $(3)²$ |
| $16x²$ | Yes | $(4x)²$ |
| $25y⁴$ | Yes | $(5y²)²$ |
| $x³$ | No | — |
| $7$ | No | — |

### Examples of Difference of Squares

```
x² - 9 = x² - 3² = (x + 3)(x - 3)

4x² - 25 = (2x)² - (5)² = (2x + 5)(2x - 5)

x⁴ - 16 = (x²)² - (4)² = (x² + 4)(x² - 4)
        = (x² + 4)(x + 2)(x - 2)    ← Factor again!
```

### Why Sum of Squares Doesn't Factor

$$a² + b² \neq (a + b)(a - b)$$

The sum of two squares **cannot be factored over real numbers**.

```
Proof by contradiction:

If a² + b² = (a + p)(a + q) for some real p, q:

Expanding: a² + (p+q)a + pq

For this to equal a² + b²:
• p + q = 0 (no middle term)
• pq = b²

From p + q = 0: q = -p
So: pq = -p² = b²
This means: p² = -b²

But p² ≥ 0 and -b² ≤ 0
The only solution is p = b = 0

Therefore, a² + b² is PRIME over real numbers.
```

**Exception:** Over complex numbers: $a² + b² = (a + bi)(a - bi)$

---

## 2. Sum of Two Cubes

### The Formula

$$a³ + b³ = (a + b)(a² - ab + b²)$$

```
┌─────────────────────────────────────────────────────────────────────┐
│               SUM OF TWO CUBES                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Formula: a³ + b³ = (a + b)(a² - ab + b²)                         │
│                       ─────  ─────────────                          │
│                      binomial  trinomial                            │
│                                                                     │
│   Memory aid: "SOAP"                                               │
│   S = Same sign as the original                                   │
│   O = Opposite sign                                                │
│   AP = Always Positive                                             │
│                                                                     │
│   a³ + b³ = (a + b)(a² - ab + b²)                                 │
│                 ↑       ↑     ↑                                    │
│                 S       O    AP                                    │
│                                                                     │
│   Derivation:                                                      │
│   (a + b)(a² - ab + b²)                                           │
│   = a³ - a²b + ab² + a²b - ab² + b³                               │
│   = a³ + b³  ✓                                                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Examples

| Expression | Identify a and b | Factored Form |
|------------|-----------------|---------------|
| $x³ + 8$ | a=x, b=2 | $(x+2)(x²-2x+4)$ |
| $27 + y³$ | a=3, b=y | $(3+y)(9-3y+y²)$ |
| $8x³ + 125$ | a=2x, b=5 | $(2x+5)(4x²-10x+25)$ |

---

## 3. Difference of Two Cubes

### The Formula

$$a³ - b³ = (a - b)(a² + ab + b²)$$

```
┌─────────────────────────────────────────────────────────────────────┐
│            DIFFERENCE OF TWO CUBES                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Formula: a³ - b³ = (a - b)(a² + ab + b²)                         │
│                       ─────  ─────────────                          │
│                      binomial  trinomial                            │
│                                                                     │
│   Using SOAP:                                                      │
│   a³ - b³ = (a - b)(a² + ab + b²)                                 │
│                 ↑       ↑     ↑                                    │
│                 S       O    AP                                    │
│                                                                     │
│   Derivation:                                                      │
│   (a - b)(a² + ab + b²)                                           │
│   = a³ + a²b + ab² - a²b - ab² - b³                               │
│   = a³ - b³  ✓                                                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Comparing Sum and Difference of Cubes

```
┌─────────────────────────────────────────────────────────────────────┐
│         SUM vs DIFFERENCE OF CUBES                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   SUM:        a³ + b³ = (a + b)(a² - ab + b²)                      │
│                            ↑        ↑                               │
│                          PLUS    MINUS                              │
│                                                                     │
│   DIFFERENCE: a³ - b³ = (a - b)(a² + ab + b²)                      │
│                            ↑        ↑                               │
│                         MINUS    PLUS                               │
│                                                                     │
│   Pattern:                                                          │
│   • First factor: Same sign as original                            │
│   • Middle term of trinomial: Opposite sign                        │
│   • Last term of trinomial: Always positive                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Perfect Cubes Reference Table

| Number | Cube | | Variable | Cube |
|--------|------|---|----------|------|
| 1 | 1 | | x | x³ |
| 2 | 8 | | 2x | 8x³ |
| 3 | 27 | | 3x | 27x³ |
| 4 | 64 | | 4x | 64x³ |
| 5 | 125 | | 5x | 125x³ |
| 10 | 1000 | | x² | x⁶ |

---

## 4. Step-by-Step Factoring Process

### For Difference of Squares

```
Factor: 49x² - 81

Step 1: Confirm it's a difference of squares
        • Two terms? Yes
        • Subtraction? Yes
        • Both perfect squares? 49x² = (7x)², 81 = (9)² Yes

Step 2: Identify a and b
        a = 7x, b = 9

Step 3: Apply formula a² - b² = (a + b)(a - b)
        49x² - 81 = (7x + 9)(7x - 9)

Answer: (7x + 9)(7x - 9)
```

### For Sum/Difference of Cubes

```
Factor: 8x³ - 27

Step 1: Confirm it's a difference of cubes
        • Two terms? Yes
        • Subtraction? Yes
        • Both perfect cubes? 8x³ = (2x)³, 27 = (3)³ Yes

Step 2: Identify a and b
        a = 2x, b = 3

Step 3: Apply formula a³ - b³ = (a - b)(a² + ab + b²)
        • First factor: (a - b) = (2x - 3)
        • a² = (2x)² = 4x²
        • ab = (2x)(3) = 6x
        • b² = 3² = 9
        • Second factor: (4x² + 6x + 9)

Answer: 8x³ - 27 = (2x - 3)(4x² + 6x + 9)
```

---

## 5. Factoring Completely with Special Products

### Multiple Applications

Sometimes you need to apply patterns multiple times or combine with other techniques.

```
┌─────────────────────────────────────────────────────────────────────┐
│           FACTORING COMPLETELY                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Example: x⁴ - 81                                                  │
│                                                                     │
│   Step 1: Recognize as difference of squares                       │
│           x⁴ - 81 = (x²)² - (9)²                                   │
│                   = (x² + 9)(x² - 9)                               │
│                                                                     │
│   Step 2: Check if factors can be factored further                │
│           • x² + 9 is sum of squares → PRIME (over reals)         │
│           • x² - 9 is difference of squares!                      │
│             x² - 9 = (x + 3)(x - 3)                                │
│                                                                     │
│   Complete factorization:                                          │
│           x⁴ - 81 = (x² + 9)(x + 3)(x - 3)                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Combining with GCF

```
Factor completely: 2x⁴ - 32

Step 1: Factor out GCF
        = 2(x⁴ - 16)

Step 2: Factor x⁴ - 16 (difference of squares)
        = 2(x² + 4)(x² - 4)

Step 3: Factor x² - 4 (difference of squares)
        = 2(x² + 4)(x + 2)(x - 2)

Answer: 2(x² + 4)(x + 2)(x - 2)
```

---

## 6. Special Cases and Tricks

### Difference of Squares with Expressions

```
Factor: (x + 1)² - (x - 1)²

Let a = (x + 1) and b = (x - 1)

a² - b² = (a + b)(a - b)
        = [(x+1) + (x-1)][(x+1) - (x-1)]
        = [2x][2]
        = 4x

Answer: 4x
```

### Creating a Difference of Squares

```
Factor: x² - 6x + 9 - y²

Rearrange: (x² - 6x + 9) - y²
         = (x - 3)² - y²     ← Perfect square trinomial!
         = [(x-3) + y][(x-3) - y]
         = (x - 3 + y)(x - 3 - y)

Answer: (x + y - 3)(x - y - 3)
```

### Sum and Difference Pattern

```
a⁶ - b⁶ can be factored two ways!

Method 1: As difference of squares
(a³)² - (b³)² = (a³ + b³)(a³ - b³)
Then factor each cube expression further.

Method 2: As difference of cubes
(a²)³ - (b²)³ = (a² - b²)(a⁴ + a²b² + b⁴)
Then factor a² - b² as difference of squares.

Both lead to the same complete factorization:
a⁶ - b⁶ = (a + b)(a - b)(a² + ab + b²)(a² - ab + b²)
```

---

## ✏️ Solved Examples

### Example 1: Easy - Difference of Squares

**Problem:** Factor $x² - 49$

**Solution:**
```
Identify: a² - b² where a = x, b = 7

Apply formula:
x² - 49 = (x + 7)(x - 7)

Verification:
(x + 7)(x - 7) = x² - 7x + 7x - 49 = x² - 49 ✓

Answer: (x + 7)(x - 7)
```

### Example 2: Easy - Sum of Cubes

**Problem:** Factor $x³ + 64$

**Solution:**
```
Identify: a³ + b³ where a = x, b = 4 (since 64 = 4³)

Apply formula: a³ + b³ = (a + b)(a² - ab + b²)

x³ + 64 = (x + 4)(x² - 4x + 16)

Verification:
(x + 4)(x² - 4x + 16)
= x³ - 4x² + 16x + 4x² - 16x + 64
= x³ + 64 ✓

Answer: (x + 4)(x² - 4x + 16)
```

### Example 3: Medium - Difference of Cubes

**Problem:** Factor $27a³ - 8b³$

**Solution:**
```
Identify perfect cubes:
27a³ = (3a)³
8b³ = (2b)³

So this is (3a)³ - (2b)³

Apply formula: a³ - b³ = (a - b)(a² + ab + b²)
where a = 3a, b = 2b

= (3a - 2b)[(3a)² + (3a)(2b) + (2b)²]
= (3a - 2b)(9a² + 6ab + 4b²)

Answer: (3a - 2b)(9a² + 6ab + 4b²)
```

### Example 4: Medium - Factor Completely

**Problem:** Factor completely: $16x⁴ - 1$

**Solution:**
```
Step 1: Recognize as difference of squares
        16x⁴ - 1 = (4x²)² - (1)²
                 = (4x² + 1)(4x² - 1)

Step 2: Check each factor
        • 4x² + 1: Sum of squares → PRIME
        • 4x² - 1: Difference of squares!
          4x² - 1 = (2x)² - (1)² = (2x + 1)(2x - 1)

Complete factorization:
16x⁴ - 1 = (4x² + 1)(2x + 1)(2x - 1)

Answer: (4x² + 1)(2x + 1)(2x - 1)
```

### Example 5: Hard - With GCF

**Problem:** Factor completely: $3x³ + 24$

**Solution:**
```
Step 1: Factor out GCF
        GCF = 3
        3x³ + 24 = 3(x³ + 8)

Step 2: Factor x³ + 8 (sum of cubes)
        x³ + 8 = x³ + 2³
               = (x + 2)(x² - 2x + 4)

Complete factorization:
3x³ + 24 = 3(x + 2)(x² - 2x + 4)

Answer: 3(x + 2)(x² - 2x + 4)
```

### Example 6: Hard - Expression with Difference of Squares

**Problem:** Factor $x² + 2xy + y² - 9$

**Solution:**
```
Step 1: Recognize the perfect square trinomial
        x² + 2xy + y² = (x + y)²

Step 2: Rewrite the expression
        (x + y)² - 9

Step 3: Apply difference of squares
        = (x + y)² - 3²
        = [(x + y) + 3][(x + y) - 3]
        = (x + y + 3)(x + y - 3)

Answer: (x + y + 3)(x + y - 3)
```

---

## ❓ Practice Problems

### Easy Level

1. Factor: $x² - 36$

2. Factor: $y³ + 1$

3. Factor: $m³ - 125$

### Medium Level

4. Factor: $25a² - 4b²$

5. Factor: $64x³ + 27y³$

6. Factor completely: $x⁴ - 16$

### Hard Level

7. Factor completely: $2x⁵ - 128x²$

8. Factor: $a² - 4ab + 4b² - c²$

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. $x² - 6² = $ **$(x + 6)(x - 6)$**

2. $y³ + 1³ = $ **$(y + 1)(y² - y + 1)$**

3. $m³ - 5³ = $ **$(m - 5)(m² + 5m + 25)$**

4. $(5a)² - (2b)² = $ **$(5a + 2b)(5a - 2b)$**

5. $(4x)³ + (3y)³ = $ **$(4x + 3y)(16x² - 12xy + 9y²)$**

6. $(x²)² - (4)² = (x² + 4)(x² - 4)$
   $= (x² + 4)(x + 2)(x - 2)$: **$(x² + 4)(x + 2)(x - 2)$**

7. GCF = 2x²: $2x²(x³ - 64)$
   $= 2x²(x - 4)(x² + 4x + 16)$: **$2x²(x - 4)(x² + 4x + 16)$**

8. $(a - 2b)² - c² = $ **$(a - 2b + c)(a - 2b - c)$**

</details>

---

## 📋 Summary Table

| Pattern | Formula | Example |
|---------|---------|---------|
| Difference of Squares | $a² - b² = (a+b)(a-b)$ | $x²-9=(x+3)(x-3)$ |
| Sum of Squares | **Not factorable** (over ℝ) | $x²+9$ is prime |
| Sum of Cubes | $a³ + b³ = (a+b)(a²-ab+b²)$ | $x³+8=(x+2)(x²-2x+4)$ |
| Difference of Cubes | $a³ - b³ = (a-b)(a²+ab+b²)$ | $x³-8=(x-2)(x²+2x+4)$ |

---

## 🔄 Quick Revision Questions

1. **Factor: $x² - 100$**

2. **What is the formula for $a³ + b³$?**

3. **Can $x² + 4$ be factored over real numbers?**

4. **Apply SOAP: What are the signs in $(a³ - b³)$?**

5. **Factor: $8 - x³$**

6. **Factor completely: $x⁴ - 1$**

<details>
<summary>Quick Answers</summary>

1. $(x + 10)(x - 10)$
2. $(a + b)(a² - ab + b²)$
3. No, sum of squares is prime over reals
4. Binomial: minus; Trinomial: plus, plus
5. $(2 - x)(4 + 2x + x²)$
6. $(x² + 1)(x + 1)(x - 1)$

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ DIFFERENCE OF SQUARES: a² - b² = (a + b)(a - b)               │
│                                                                     │
│   ★ SUM OF SQUARES: a² + b² is PRIME (over real numbers)          │
│                                                                     │
│   ★ SUM OF CUBES: a³ + b³ = (a + b)(a² - ab + b²)                 │
│                                                                     │
│   ★ DIFFERENCE OF CUBES: a³ - b³ = (a - b)(a² + ab + b²)          │
│                                                                     │
│   ★ Remember SOAP for cubes:                                       │
│     Same sign, Opposite sign, Always Positive                     │
│                                                                     │
│   ★ Always factor out GCF first                                   │
│                                                                     │
│   ★ Check if factors can be factored again                        │
│                                                                     │
│   ★ The trinomial factor in cube formulas is always prime         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Factoring Quadratic Trinomials](03-factoring-quadratic-trinomials.md) | [Back to Contents](../README.md) | [Next: Factoring by Substitution →](05-factoring-by-substitution.md)
