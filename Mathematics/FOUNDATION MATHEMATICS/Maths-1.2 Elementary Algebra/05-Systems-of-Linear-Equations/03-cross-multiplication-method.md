# Chapter 5.3: Cross-Multiplication Method

[← Previous: Elimination Method](02-elimination-method.md) | [Back to Contents](../README.md) | [Next: Three-Variable Systems →](04-three-variable-systems.md)

---

## 📚 Chapter Overview

The **cross-multiplication method** provides a direct formula-based approach to solving systems of two linear equations in two variables. This method yields the solution in one step without the need for back-substitution, making it particularly efficient for theoretical work and derivations.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Derive the cross-multiplication formula
- Apply the formula to solve linear systems
- Interpret determinant-like expressions
- Handle special cases using cross-multiplication
- Recognize connections to Cramer's Rule
- Decide when cross-multiplication is efficient

---

## 1. The Cross-Multiplication Formula

### Standard Form Setup

For a system in standard form:
$$a_1x + b_1y + c_1 = 0$$
$$a_2x + b_2y + c_2 = 0$$

```
┌─────────────────────────────────────────────────────────────────────┐
│              CROSS-MULTIPLICATION FORMULA                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For:  a₁x + b₁y + c₁ = 0                                         │
│         a₂x + b₂y + c₂ = 0                                         │
│                                                                     │
│   The solution is given by:                                        │
│                                                                     │
│           x                y                 1                     │
│   ─────────────── = ─────────────── = ───────────────              │
│    b₁c₂ - b₂c₁      c₁a₂ - c₂a₁      a₁b₂ - a₂b₁                 │
│                                                                     │
│   This gives:                                                      │
│                                                                     │
│        b₁c₂ - b₂c₁                c₁a₂ - c₂a₁                     │
│   x = ─────────────    and   y = ─────────────                    │
│        a₁b₂ - a₂b₁                a₁b₂ - a₂b₁                     │
│                                                                     │
│   (provided a₁b₂ - a₂b₁ ≠ 0)                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Memory Aid: The Cross Pattern

```
┌─────────────────────────────────────────────────────────────────────┐
│              CROSS PATTERN FOR MEMORY                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Write coefficients in order: x, y, constant, x, y                │
│                                                                     │
│   Equation 1:  a₁   b₁   c₁   a₁   b₁                              │
│                                                                     │
│   Equation 2:  a₂   b₂   c₂   a₂   b₂                              │
│                                                                     │
│   Draw crosses:                                                    │
│                                                                     │
│        b₁   c₁   a₁       c₁   a₁   b₁                             │
│          ╲ ╱       ╲ ╱       ╲ ╱                                    │
│           ╳         ╳         ╳                                    │
│          ╱ ╲       ╱ ╲       ╱ ╲                                    │
│        b₂   c₂   a₂       c₂   a₂   b₂                             │
│                                                                     │
│   Formula:                                                         │
│        x              y               1                            │
│   ────────────  = ────────────  = ────────────                    │
│   b₁c₂ - b₂c₁    c₁a₂ - c₂a₁     a₁b₂ - a₂b₁                     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Derivation of the Formula

### Algebraic Proof

```
┌─────────────────────────────────────────────────────────────────────┐
│              DERIVATION USING ELIMINATION                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Given: a₁x + b₁y = -c₁    ... (1)                               │
│          a₂x + b₂y = -c₂    ... (2)                               │
│                                                                     │
│   To eliminate y, multiply (1) by b₂ and (2) by b₁:               │
│                                                                     │
│   a₁b₂x + b₁b₂y = -c₁b₂    ... (3)                                │
│   a₂b₁x + b₂b₁y = -c₂b₁    ... (4)                                │
│                                                                     │
│   Subtract (4) from (3):                                          │
│   (a₁b₂ - a₂b₁)x = -c₁b₂ + c₂b₁                                  │
│   (a₁b₂ - a₂b₁)x = b₁c₂ - b₂c₁                                   │
│                                                                     │
│              b₁c₂ - b₂c₁                                           │
│   ∴  x = ─────────────────                                        │
│              a₁b₂ - a₂b₁                                           │
│                                                                     │
│   Similarly, eliminating x:                                        │
│                                                                     │
│              c₁a₂ - c₂a₁                                           │
│   ∴  y = ─────────────────                                        │
│              a₁b₂ - a₂b₁                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Step-by-Step Process

### Algorithm

```
┌─────────────────────────────────────────────────────────────────────┐
│              CROSS-MULTIPLICATION STEPS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Step 1: Write both equations in the form                        │
│           a₁x + b₁y + c₁ = 0                                       │
│           a₂x + b₂y + c₂ = 0                                       │
│           (Move constant terms to left side!)                      │
│                                                                     │
│   Step 2: Identify coefficients:                                  │
│           a₁, b₁, c₁ from equation 1                              │
│           a₂, b₂, c₂ from equation 2                              │
│                                                                     │
│   Step 3: Calculate the denominator: D = a₁b₂ - a₂b₁             │
│           If D = 0, no unique solution                            │
│                                                                     │
│   Step 4: Calculate x = (b₁c₂ - b₂c₁) / D                        │
│                                                                     │
│   Step 5: Calculate y = (c₁a₂ - c₂a₁) / D                        │
│                                                                     │
│   Step 6: Verify solution in both original equations              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Detailed Example

```
Solve: { 2x + 3y = 7
       { 5x - 2y = 4

Step 1: Rewrite in form ax + by + c = 0
   2x + 3y - 7 = 0    (a₁=2, b₁=3, c₁=-7)
   5x - 2y - 4 = 0    (a₂=5, b₂=-2, c₂=-4)

Step 2: Apply formula
        x                y                1
   ────────────── = ────────────── = ──────────────
   b₁c₂ - b₂c₁     c₁a₂ - c₂a₁      a₁b₂ - a₂b₁

Calculate each cross product:
   b₁c₂ - b₂c₁ = (3)(-4) - (-2)(-7) = -12 - 14 = -26
   c₁a₂ - c₂a₁ = (-7)(5) - (-4)(2) = -35 + 8 = -27
   a₁b₂ - a₂b₁ = (2)(-2) - (5)(3) = -4 - 15 = -19

Step 3: Solve
        x        y        1
      ───── = ───── = ─────
       -26     -27     -19

   x = -26/-19 = 26/19
   y = -27/-19 = 27/19

Solution: (26/19, 27/19)
```

---

## 4. Special Cases

### No Unique Solution (D = 0)

```
┌─────────────────────────────────────────────────────────────────────┐
│              WHEN D = a₁b₂ - a₂b₁ = 0                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   If D = 0, check the numerators:                                 │
│                                                                     │
│   Case 1: D = 0 AND both numerators ≠ 0                           │
│           → NO SOLUTION (parallel lines)                          │
│                                                                     │
│   Case 2: D = 0 AND both numerators = 0                           │
│           → INFINITELY MANY SOLUTIONS (same line)                 │
│                                                                     │
│   Example of No Solution:                                         │
│   { 2x + 4y - 6 = 0      (a₁=2, b₁=4, c₁=-6)                     │
│   { x + 2y - 5 = 0       (a₂=1, b₂=2, c₂=-5)                     │
│                                                                     │
│   D = (2)(2) - (1)(4) = 4 - 4 = 0                                │
│   Numerator for x: (4)(-5) - (2)(-6) = -20 + 12 = -8 ≠ 0         │
│   → No solution                                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Connection to Determinants

### Cramer's Rule Preview

The cross-multiplication method is essentially Cramer's Rule for 2×2 systems:

```
┌─────────────────────────────────────────────────────────────────────┐
│              DETERMINANT NOTATION                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For system: a₁x + b₁y = k₁                                      │
│              a₂x + b₂y = k₂                                       │
│                                                                     │
│   Define determinants:                                             │
│                                                                     │
│       |a₁  b₁|                                                     │
│   D = |a₂  b₂| = a₁b₂ - a₂b₁                                      │
│                                                                     │
│        |k₁  b₁|                                                    │
│   Dₓ = |k₂  b₂| = k₁b₂ - k₂b₁                                     │
│                                                                     │
│        |a₁  k₁|                                                    │
│   Dᵧ = |a₂  k₂| = a₁k₂ - a₂k₁                                     │
│                                                                     │
│   Then: x = Dₓ/D  and  y = Dᵧ/D                                   │
│                                                                     │
│   (This is identical to cross-multiplication with c = -k)         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. When to Use Cross-Multiplication

### Advantages and Limitations

| Advantages | Limitations |
|------------|-------------|
| Direct formula - no back-substitution | Only works for 2×2 systems |
| Good for theoretical work | More calculations for simple systems |
| Formula easily memorized | Must convert to standard form first |
| Useful for deriving conditions | Prone to arithmetic errors |

---

## ✏️ Solved Examples

### Example 1: Easy - Integer Solution

**Problem:** Solve using cross-multiplication:
$\begin{cases} x + y = 5 \\ x - y = 1 \end{cases}$

**Solution:**
```
Convert to standard form (ax + by + c = 0):
   x + y - 5 = 0    (a₁=1, b₁=1, c₁=-5)
   x - y - 1 = 0    (a₂=1, b₂=-1, c₂=-1)

Apply formula:
        x                 y                 1
   ───────────── = ───────────── = ─────────────
   b₁c₂ - b₂c₁    c₁a₂ - c₂a₁     a₁b₂ - a₂b₁

Calculate:
   b₁c₂ - b₂c₁ = (1)(-1) - (-1)(-5) = -1 - 5 = -6
   c₁a₂ - c₂a₁ = (-5)(1) - (-1)(1) = -5 + 1 = -4
   a₁b₂ - a₂b₁ = (1)(-1) - (1)(1) = -1 - 1 = -2

So:    x      y      1
      ─── = ─── = ────
       -6    -4    -2

   x = -6/-2 = 3
   y = -4/-2 = 2

Answer: (3, 2)
```

### Example 2: Easy - Fraction Solution

**Problem:** Solve:
$\begin{cases} 2x + y = 5 \\ 3x + 2y = 7 \end{cases}$

**Solution:**
```
Standard form:
   2x + y - 5 = 0    (a₁=2, b₁=1, c₁=-5)
   3x + 2y - 7 = 0   (a₂=3, b₂=2, c₂=-7)

Calculate cross products:
   b₁c₂ - b₂c₁ = (1)(-7) - (2)(-5) = -7 + 10 = 3
   c₁a₂ - c₂a₁ = (-5)(3) - (-7)(2) = -15 + 14 = -1
   a₁b₂ - a₂b₁ = (2)(2) - (3)(1) = 4 - 3 = 1

   x = 3/1 = 3
   y = -1/1 = -1

Check: 2(3) + (-1) = 6 - 1 = 5 ✓
       3(3) + 2(-1) = 9 - 2 = 7 ✓

Answer: (3, -1)
```

### Example 3: Medium - Negative Coefficients

**Problem:** Solve:
$\begin{cases} 3x - 4y = 2 \\ 5x + 3y = 12 \end{cases}$

**Solution:**
```
Standard form:
   3x - 4y - 2 = 0    (a₁=3, b₁=-4, c₁=-2)
   5x + 3y - 12 = 0   (a₂=5, b₂=3, c₂=-12)

Calculate:
   b₁c₂ - b₂c₁ = (-4)(-12) - (3)(-2) = 48 + 6 = 54
   c₁a₂ - c₂a₁ = (-2)(5) - (-12)(3) = -10 + 36 = 26
   a₁b₂ - a₂b₁ = (3)(3) - (5)(-4) = 9 + 20 = 29

   x = 54/29
   y = 26/29

Answer: (54/29, 26/29)
```

### Example 4: Medium - Coefficients Need Care

**Problem:** Solve:
$\begin{cases} 4x - 3y = 11 \\ 5x + 7y = -3 \end{cases}$

**Solution:**
```
Standard form:
   4x - 3y - 11 = 0   (a₁=4, b₁=-3, c₁=-11)
   5x + 7y + 3 = 0    (a₂=5, b₂=7, c₂=3)

Calculate:
   b₁c₂ - b₂c₁ = (-3)(3) - (7)(-11) = -9 + 77 = 68
   c₁a₂ - c₂a₁ = (-11)(5) - (3)(4) = -55 - 12 = -67
   a₁b₂ - a₂b₁ = (4)(7) - (5)(-3) = 28 + 15 = 43

   x = 68/43
   y = -67/43

Check in equation 1:
4(68/43) - 3(-67/43) = 272/43 + 201/43 = 473/43 = 11 ✓

Answer: (68/43, -67/43)
```

### Example 5: Hard - No Solution

**Problem:** Determine the solution:
$\begin{cases} 2x + 6y = 4 \\ x + 3y = 8 \end{cases}$

**Solution:**
```
Standard form:
   2x + 6y - 4 = 0   (a₁=2, b₁=6, c₁=-4)
   x + 3y - 8 = 0    (a₂=1, b₂=3, c₂=-8)

Calculate denominator first:
   D = a₁b₂ - a₂b₁ = (2)(3) - (1)(6) = 6 - 6 = 0

Since D = 0, check numerators:
   b₁c₂ - b₂c₁ = (6)(-8) - (3)(-4) = -48 + 12 = -36 ≠ 0

D = 0 but numerator ≠ 0

Answer: No solution (parallel lines)
```

### Example 6: Hard - Infinitely Many Solutions

**Problem:** Solve:
$\begin{cases} 3x - 6y = 9 \\ x - 2y = 3 \end{cases}$

**Solution:**
```
Standard form:
   3x - 6y - 9 = 0   (a₁=3, b₁=-6, c₁=-9)
   x - 2y - 3 = 0    (a₂=1, b₂=-2, c₂=-3)

Calculate:
   D = a₁b₂ - a₂b₁ = (3)(-2) - (1)(-6) = -6 + 6 = 0

Check numerators:
   b₁c₂ - b₂c₁ = (-6)(-3) - (-2)(-9) = 18 - 18 = 0
   c₁a₂ - c₂a₁ = (-9)(1) - (-3)(3) = -9 + 9 = 0

All three expressions = 0

Note: Equation 1 = 3 × Equation 2 (same line!)

Answer: Infinitely many solutions
General solution: x = 2y + 3, y ∈ ℝ
```

---

## ❓ Practice Problems

### Easy Level

1. Solve using cross-multiplication:
   $\begin{cases} x + 2y = 7 \\ x - y = 1 \end{cases}$

2. Solve:
   $\begin{cases} 2x + y = 8 \\ x + y = 5 \end{cases}$

3. Solve:
   $\begin{cases} 3x - y = 5 \\ x + y = 3 \end{cases}$

### Medium Level

4. Solve:
   $\begin{cases} 5x - 3y = 1 \\ 2x + 7y = 11 \end{cases}$

5. Solve:
   $\begin{cases} 4x + 5y = 14 \\ 3x + 4y = 11 \end{cases}$

6. Determine if the system has a unique solution, no solution, or infinitely many:
   $\begin{cases} 4x - 2y = 10 \\ 6x - 3y = 15 \end{cases}$

### Hard Level

7. Solve:
   $\begin{cases} 7x - 5y = -1 \\ 2x + 3y = 11 \end{cases}$

8. Find the value of k for which the system has no solution:
   $\begin{cases} 3x + 5y = 1 \\ kx + 10y = 5 \end{cases}$

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. x + 2y - 7 = 0, x - y - 1 = 0
   D = (1)(-1) - (1)(2) = -3
   x = [(2)(-1) - (-1)(-7)]/-3 = (-2 - 7)/-3 = 3
   y = [(-7)(1) - (-1)(1)]/-3 = -6/-3 = 2
   **$(3, 2)$**

2. 2x + y - 8 = 0, x + y - 5 = 0
   D = (2)(1) - (1)(1) = 1
   x = (1)(-5) - (1)(-8) = 3
   y = (-8)(1) - (-5)(2) = 2
   **$(3, 2)$**

3. 3x - y - 5 = 0, x + y - 3 = 0
   D = (3)(1) - (1)(-1) = 4
   x = [(-1)(-3) - (1)(-5)]/4 = 8/4 = 2
   y = [(-5)(1) - (-3)(3)]/4 = 4/4 = 1
   **$(2, 1)$**

4. 5x - 3y - 1 = 0, 2x + 7y - 11 = 0
   D = (5)(7) - (2)(-3) = 41
   x = [(-3)(-11) - (7)(-1)]/41 = 40/41
   y = [(-1)(2) - (-11)(5)]/41 = 53/41
   **$(40/41, 53/41)$**

5. 4x + 5y - 14 = 0, 3x + 4y - 11 = 0
   D = (4)(4) - (3)(5) = 1
   x = (5)(-11) - (4)(-14) = 1
   y = (-14)(3) - (-11)(4) = 2
   **$(1, 2)$**

6. 4x - 2y - 10 = 0, 6x - 3y - 15 = 0
   D = (4)(-3) - (6)(-2) = -12 + 12 = 0
   Check: b₁c₂ - b₂c₁ = (-2)(-15) - (-3)(-10) = 30 - 30 = 0
   **Infinitely many solutions**

7. 7x - 5y + 1 = 0, 2x + 3y - 11 = 0
   D = (7)(3) - (2)(-5) = 31
   x = [(-5)(-11) - (3)(1)]/31 = 52/31
   y = [(1)(2) - (-11)(7)]/31 = 79/31
   **$(52/31, 79/31)$**

8. For no solution: D = 0 but numerators ≠ 0
   D = (3)(10) - (k)(5) = 30 - 5k = 0
   k = 6
   Check numerators with k = 6 are not both zero.
   **k = 6**

</details>

---

## 📋 Summary Table

| Expression | Formula | Interpretation |
|------------|---------|----------------|
| a₁b₂ - a₂b₁ | Denominator D | If 0, no unique solution |
| b₁c₂ - b₂c₁ | Numerator for x | Divide by D for x value |
| c₁a₂ - c₂a₁ | Numerator for y | Divide by D for y value |

| D | Numerators | Result |
|---|------------|--------|
| ≠ 0 | Any | Unique solution |
| = 0 | At least one ≠ 0 | No solution |
| = 0 | Both = 0 | Infinitely many |

---

## 🔄 Quick Revision Questions

1. **What form must equations be in for cross-multiplication?**

2. **What is the formula for x in cross-multiplication?**

3. **If a₁b₂ - a₂b₁ = 0, what does it tell you?**

4. **How do you distinguish no solution from infinitely many when D = 0?**

5. **What is the memory pattern for cross products?**

6. **How is cross-multiplication related to determinants?**

<details>
<summary>Quick Answers</summary>

1. a₁x + b₁y + c₁ = 0 (constants on left side)
2. x = (b₁c₂ - b₂c₁)/(a₁b₂ - a₂b₁)
3. No unique solution exists
4. If numerators ≠ 0, no solution; if numerators = 0, infinitely many
5. Write coefficients in order: x, y, c, x, y and draw diagonal crosses
6. The denominator is the 2×2 determinant of coefficients

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Cross-multiplication provides a direct formula solution       │
│                                                                     │
│   ★ Equations must be in form: a₁x + b₁y + c₁ = 0                 │
│                                                                     │
│   ★ Key formula:                                                   │
│         x              y               1                           │
│     ───────────  = ───────────  = ───────────                     │
│     b₁c₂-b₂c₁     c₁a₂-c₂a₁      a₁b₂-a₂b₁                       │
│                                                                     │
│   ★ If D = a₁b₂ - a₂b₁ = 0:                                       │
│     • Numerators ≠ 0 → No solution                                │
│     • Numerators = 0 → Infinitely many                            │
│                                                                     │
│   ★ This method is equivalent to Cramer's Rule for 2×2           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Elimination Method](02-elimination-method.md) | [Back to Contents](../README.md) | [Next: Three-Variable Systems →](04-three-variable-systems.md)
