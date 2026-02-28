# Chapter 6.1: Standard Form and Factoring

[← Previous: Applications of Systems](../05-Systems-of-Linear-Equations/05-applications.md) | [Back to Contents](../README.md) | [Next: Square Root Method →](02-square-root-method.md)

---

## 📚 Chapter Overview

A **quadratic equation** is a polynomial equation of degree 2. This chapter introduces the standard form of quadratic equations and the fundamental technique of solving by factoring, which works when the quadratic can be expressed as a product of linear factors.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Recognize and write quadratic equations in standard form
- Understand the Zero Product Property
- Solve quadratic equations by factoring
- Extract common factors before factoring trinomials
- Handle special cases (perfect squares, difference of squares)
- Verify solutions by substitution

---

## 1. The Quadratic Equation

### Standard Form

```
┌─────────────────────────────────────────────────────────────────────┐
│              STANDARD FORM OF A QUADRATIC EQUATION                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                    ax² + bx + c = 0                                │
│                                                                     │
│   where:                                                           │
│     • a, b, c are real numbers                                    │
│     • a ≠ 0 (if a = 0, it's linear, not quadratic)               │
│     • a is the coefficient of x² (leading coefficient)           │
│     • b is the coefficient of x (linear coefficient)             │
│     • c is the constant term                                      │
│                                                                     │
│   Examples:                                                        │
│     • x² - 5x + 6 = 0     (a=1, b=-5, c=6)                       │
│     • 2x² + 3x - 7 = 0    (a=2, b=3, c=-7)                       │
│     • x² - 9 = 0          (a=1, b=0, c=-9)                       │
│     • 4x² + x = 0         (a=4, b=1, c=0)                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Converting to Standard Form

```
┌─────────────────────────────────────────────────────────────────────┐
│              CONVERTING TO STANDARD FORM                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Given equation           →    Standard form                      │
│   ──────────────────────────────────────────────                  │
│   3x² = 12                 →    3x² - 12 = 0                      │
│   x² + 5x = 6              →    x² + 5x - 6 = 0                   │
│   (x+2)(x-3) = 0           →    x² - x - 6 = 0                    │
│   2x(x-1) = x + 3          →    2x² - 3x - 3 = 0                  │
│   x² = 5x - 4              →    x² - 5x + 4 = 0                   │
│                                                                     │
│   Steps:                                                           │
│   1. Expand all products                                          │
│   2. Move all terms to one side (usually left)                    │
│   3. Arrange in descending powers: x², x, constant                │
│   4. Simplify if possible                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. The Zero Product Property

### The Foundation of Factoring

```
┌─────────────────────────────────────────────────────────────────────┐
│              ZERO PRODUCT PROPERTY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   If A · B = 0, then A = 0 or B = 0 (or both)                    │
│                                                                     │
│   This is the KEY principle that makes factoring work!            │
│                                                                     │
│   Example:                                                         │
│   (x - 3)(x + 2) = 0                                              │
│                                                                     │
│   Either x - 3 = 0  →  x = 3                                      │
│   Or     x + 2 = 0  →  x = -2                                     │
│                                                                     │
│   Solutions: x = 3 or x = -2                                      │
│                                                                     │
│   ┌─────────────────────────────────────────────────┐             │
│   │ Important: The equation MUST equal zero!        │             │
│   │                                                 │             │
│   │ (x-3)(x+2) = 6 does NOT mean x-3=6 or x+2=6   │             │
│   │ You must expand and rearrange first!           │             │
│   └─────────────────────────────────────────────────┘             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Solving by Factoring: The Process

### Step-by-Step Method

```
┌─────────────────────────────────────────────────────────────────────┐
│              SOLVING QUADRATICS BY FACTORING                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Step 1: Write the equation in standard form (= 0)               │
│                                                                     │
│   Step 2: Factor the left side completely                         │
│                                                                     │
│   Step 3: Set each factor equal to zero                           │
│                                                                     │
│   Step 4: Solve each resulting equation                           │
│                                                                     │
│   Step 5: Check solutions in the original equation                │
│                                                                     │
│   ──────────────────────────────────────────────────────────────  │
│                                                                     │
│   Example: Solve x² - 5x + 6 = 0                                  │
│                                                                     │
│   Step 2: Factor: (x - 2)(x - 3) = 0                              │
│                                                                     │
│   Step 3: x - 2 = 0  or  x - 3 = 0                               │
│                                                                     │
│   Step 4: x = 2  or  x = 3                                        │
│                                                                     │
│   Step 5: Check x=2: (2)² - 5(2) + 6 = 4 - 10 + 6 = 0 ✓          │
│           Check x=3: (3)² - 5(3) + 6 = 9 - 15 + 6 = 0 ✓          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Types of Factoring for Quadratics

### Type 1: Common Factor (GCF)

```
Always check for a common factor first!

Example: 2x² - 6x = 0
         2x(x - 3) = 0
         2x = 0  or  x - 3 = 0
         x = 0  or  x = 3
```

### Type 2: Simple Trinomial (a = 1)

```
x² + bx + c = (x + p)(x + q)  where p + q = b and pq = c

Example: x² + 7x + 12 = 0
         Need: p + q = 7 and pq = 12
         Factors of 12: 1×12, 2×6, 3×4
         3 + 4 = 7 ✓
         (x + 3)(x + 4) = 0
         x = -3 or x = -4
```

### Type 3: General Trinomial (a ≠ 1)

```
Use the AC method or trial and error

Example: 2x² + 5x + 3 = 0
         AC = 2 × 3 = 6
         Need factors of 6 that sum to 5: 2 and 3
         Rewrite: 2x² + 2x + 3x + 3 = 0
         Factor: 2x(x + 1) + 3(x + 1) = 0
                 (2x + 3)(x + 1) = 0
         x = -3/2 or x = -1
```

### Type 4: Difference of Squares

```
a² - b² = (a + b)(a - b)

Example: x² - 25 = 0
         (x + 5)(x - 5) = 0
         x = -5 or x = 5
```

### Type 5: Perfect Square Trinomial

```
a² + 2ab + b² = (a + b)²
a² - 2ab + b² = (a - b)²

Example: x² - 10x + 25 = 0
         (x - 5)² = 0
         x - 5 = 0
         x = 5 (double root)
```

---

## 5. Flowchart for Factoring Quadratics

```
                    ax² + bx + c = 0
                          │
                          ▼
              ┌───────────────────────┐
              │  Is there a GCF?     │
              └───────────┬───────────┘
                    ┌─────┴─────┐
                   YES         NO
                    │           │
                    ▼           │
              Factor out       │
              the GCF          │
                    │          │
                    └────┬─────┘
                         │
                         ▼
              ┌───────────────────────┐
              │  Is it binomial or   │
              │  trinomial?          │
              └───────────┬───────────┘
                    ┌─────┴─────┐
              BINOMIAL      TRINOMIAL
                    │           │
                    ▼           ▼
         ┌─────────────┐   ┌─────────────┐
         │ Difference  │   │  a = 1?     │
         │ of squares? │   └──────┬──────┘
         └──────┬──────┘    ┌─────┴─────┐
                │          YES         NO
                ▼           │           │
         (a+b)(a-b)        ▼           ▼
                     Find p,q      AC Method
                     p+q=b         or trial
                     pq=c
```

---

## 6. Special Situations

### When One Solution is Zero

```
x² - 5x = 0
x(x - 5) = 0
x = 0  or  x = 5

Don't lose the x = 0 solution!
```

### Double Roots (Repeated Solution)

```
x² - 6x + 9 = 0
(x - 3)² = 0
x = 3 (double root)

The parabola touches the x-axis at one point
```

### When Factoring Doesn't Work Easily

```
x² + x + 1 = 0
This doesn't factor over real numbers!
(Need other methods: quadratic formula)
```

---

## 7. Common Mistakes to Avoid

```
┌─────────────────────────────────────────────────────────────────────┐
│              COMMON ERRORS                                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ✗ WRONG: x² = 5x → x = 5 (lost x = 0!)                         │
│   ✓ RIGHT: x² = 5x → x² - 5x = 0 → x(x-5) = 0 → x = 0 or x = 5  │
│                                                                     │
│   ✗ WRONG: Dividing both sides by x (loses x = 0 solution)        │
│   ✓ RIGHT: Move everything to one side, then factor              │
│                                                                     │
│   ✗ WRONG: (x-2)(x-3) = 2 → x-2 = 2 or x-3 = 2                  │
│   ✓ RIGHT: Must equal ZERO to use zero product property          │
│            Expand: x² - 5x + 6 = 2 → x² - 5x + 4 = 0             │
│                                                                     │
│   ✗ WRONG: x² + 9 = 0 → (x+3)(x-3) = 0                           │
│   ✓ RIGHT: x² + 9 ≠ (x+3)(x-3); this has no real solutions       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✏️ Solved Examples

### Example 1: Easy - Simple Trinomial

**Problem:** Solve x² - 7x + 10 = 0

**Solution:**
```
Factor: Find two numbers that multiply to 10 and add to -7
        -2 × -5 = 10 ✓
        -2 + (-5) = -7 ✓

        (x - 2)(x - 5) = 0

Apply zero product property:
        x - 2 = 0  →  x = 2
        x - 5 = 0  →  x = 5

Check: x = 2: (2)² - 7(2) + 10 = 4 - 14 + 10 = 0 ✓
       x = 5: (5)² - 7(5) + 10 = 25 - 35 + 10 = 0 ✓

Answer: x = 2 or x = 5
```

### Example 2: Easy - Common Factor

**Problem:** Solve 3x² + 12x = 0

**Solution:**
```
Factor out GCF: 3x(x + 4) = 0

Apply zero product property:
        3x = 0  →  x = 0
        x + 4 = 0  →  x = -4

Check: x = 0: 3(0)² + 12(0) = 0 ✓
       x = -4: 3(16) + 12(-4) = 48 - 48 = 0 ✓

Answer: x = 0 or x = -4
```

### Example 3: Easy - Difference of Squares

**Problem:** Solve x² - 49 = 0

**Solution:**
```
Factor: x² - 49 = (x + 7)(x - 7) = 0

Apply zero product property:
        x + 7 = 0  →  x = -7
        x - 7 = 0  →  x = 7

Check: x = 7: 49 - 49 = 0 ✓
       x = -7: 49 - 49 = 0 ✓

Answer: x = ±7
```

### Example 4: Medium - Leading Coefficient ≠ 1

**Problem:** Solve 2x² - 7x + 3 = 0

**Solution:**
```
Use AC method: A = 2, C = 3, AC = 6
Find factors of 6 that add to -7: -1 and -6

Rewrite middle term:
2x² - x - 6x + 3 = 0

Factor by grouping:
x(2x - 1) - 3(2x - 1) = 0
(x - 3)(2x - 1) = 0

Apply zero product property:
        x - 3 = 0  →  x = 3
        2x - 1 = 0  →  x = 1/2

Check: x = 3: 2(9) - 7(3) + 3 = 18 - 21 + 3 = 0 ✓
       x = 1/2: 2(1/4) - 7(1/2) + 3 = 1/2 - 7/2 + 3 = 0 ✓

Answer: x = 3 or x = 1/2
```

### Example 5: Medium - Equation Not in Standard Form

**Problem:** Solve (x + 3)(x - 2) = 14

**Solution:**
```
Expand and rearrange:
x² + x - 6 = 14
x² + x - 20 = 0

Factor: Find factors of -20 that add to 1: 5 and -4
(x + 5)(x - 4) = 0

Apply zero product property:
        x = -5  or  x = 4

Check: x = 4: (4+3)(4-2) = 7 × 2 = 14 ✓
       x = -5: (-5+3)(-5-2) = (-2)(-7) = 14 ✓

Answer: x = -5 or x = 4
```

### Example 6: Hard - Multiple Steps Required

**Problem:** Solve 6x² + x - 12 = 0

**Solution:**
```
AC method: A = 6, C = -12, AC = -72
Find factors of -72 that add to 1: 9 and -8

Rewrite:
6x² + 9x - 8x - 12 = 0

Factor by grouping:
3x(2x + 3) - 4(2x + 3) = 0
(3x - 4)(2x + 3) = 0

Apply zero product property:
        3x - 4 = 0  →  x = 4/3
        2x + 3 = 0  →  x = -3/2

Check: x = 4/3: 6(16/9) + 4/3 - 12 = 32/3 + 4/3 - 36/3 = 0 ✓
       x = -3/2: 6(9/4) - 3/2 - 12 = 27/2 - 3/2 - 24/2 = 0 ✓

Answer: x = 4/3 or x = -3/2
```

---

## ❓ Practice Problems

### Easy Level

1. Solve: x² - 9x + 14 = 0

2. Solve: x² + 4x = 0

3. Solve: x² - 81 = 0

4. Solve: x² + 6x + 9 = 0

### Medium Level

5. Solve: 3x² - 10x + 8 = 0

6. Solve: x² - 3x = 28

7. Solve: 5x² + 13x - 6 = 0

### Hard Level

8. Solve: (2x - 1)(x + 3) = 9

9. Solve: 6x² - 7x - 3 = 0

10. Solve: x(x + 1) = 2(x + 8)

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. (x - 2)(x - 7) = 0
   **x = 2 or x = 7**

2. x(x + 4) = 0
   **x = 0 or x = -4**

3. (x + 9)(x - 9) = 0
   **x = ±9**

4. (x + 3)² = 0
   **x = -3 (double root)**

5. AC = 24, factors -4 and -6
   3x² - 4x - 6x + 8 = 0
   (3x - 4)(x - 2) = 0
   **x = 4/3 or x = 2**

6. x² - 3x - 28 = 0
   (x - 7)(x + 4) = 0
   **x = 7 or x = -4**

7. AC = -30, factors 15 and -2
   5x² + 15x - 2x - 6 = 0
   5x(x + 3) - 2(x + 3) = 0
   (5x - 2)(x + 3) = 0
   **x = 2/5 or x = -3**

8. 2x² + 5x - 3 = 9
   2x² + 5x - 12 = 0
   (2x - 3)(x + 4) = 0
   **x = 3/2 or x = -4**

9. AC = -18, factors 2 and -9
   6x² + 2x - 9x - 3 = 0
   2x(3x + 1) - 3(3x + 1) = 0
   (2x - 3)(3x + 1) = 0
   **x = 3/2 or x = -1/3**

10. x² + x = 2x + 16
    x² - x - 16 = 0
    This doesn't factor nicely (irrational roots)
    **x = (1 ± √65)/2**

</details>

---

## 📋 Summary Table

| Form | Factoring Method |
|------|------------------|
| x² + bx = 0 | Factor out x: x(x + b) = 0 |
| x² - c² = 0 | Difference of squares: (x+c)(x-c) = 0 |
| x² + bx + c = 0 | Find p, q where p+q=b, pq=c |
| ax² + bx + c = 0 | AC method or trial and error |
| x² + 2bx + b² = 0 | Perfect square: (x + b)² = 0 |

---

## 🔄 Quick Revision Questions

1. **What is the standard form of a quadratic equation?**

2. **What is the Zero Product Property?**

3. **If x² = 9x, what's the first step?**

4. **How many solutions can a quadratic equation have?**

5. **What type of factoring is used for x² - 16?**

6. **If (x - 4)² = 0, how many distinct solutions are there?**

<details>
<summary>Quick Answers</summary>

1. ax² + bx + c = 0, where a ≠ 0
2. If AB = 0, then A = 0 or B = 0
3. Move all terms to one side: x² - 9x = 0, then factor
4. 0, 1, or 2 real solutions
5. Difference of squares
6. One (x = 4 is a double root)

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Standard form: ax² + bx + c = 0 (a ≠ 0)                       │
│                                                                     │
│   ★ Zero Product Property: If AB = 0, then A = 0 or B = 0        │
│     This is why we need equation = 0 before factoring            │
│                                                                     │
│   ★ Steps to solve by factoring:                                  │
│     1. Get standard form (= 0)                                    │
│     2. Factor completely                                          │
│     3. Set each factor = 0                                        │
│     4. Solve and check                                            │
│                                                                     │
│   ★ Always factor out GCF first                                   │
│                                                                     │
│   ★ Never divide by x (you'll lose the x = 0 solution)           │
│                                                                     │
│   ★ Not all quadratics factor nicely - use other methods         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Applications of Systems](../05-Systems-of-Linear-Equations/05-applications.md) | [Back to Contents](../README.md) | [Next: Square Root Method →](02-square-root-method.md)
