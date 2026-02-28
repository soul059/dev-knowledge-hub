# Chapter 1.4: General and Particular Solutions

[← Previous: Formation of DEs](03-formation-of-differential-equations.md) | [Back to README](../README.md) | [Next: Initial Value Problems →](05-initial-value-problems.md)

---

## Overview

When we solve a differential equation, we don't get a single function — we get a **family** of functions. Understanding the distinction between general solutions, particular solutions, and singular solutions is essential for correctly interpreting and applying the results.

---

## 1. General Solution

### Definition

> The **general solution** of an $n$-th order ODE is a solution that contains exactly **$n$ independent arbitrary constants** $C_1, C_2, \ldots, C_n$.

It represents the **complete family** of all solutions (under standard conditions).

### Examples

| ODE | General Solution | Constants |
|-----|-----------------|-----------|
| $y' = 2x$ | $y = x^2 + C$ | 1 |
| $y'' + y = 0$ | $y = C_1 \cos x + C_2 \sin x$ | 2 |
| $y'' - 4y = 0$ | $y = C_1 e^{2x} + C_2 e^{-2x}$ | 2 |
| $y''' = 0$ | $y = C_1 + C_2 x + C_3 x^2$ | 3 |

### Geometric Interpretation

```
   y
   │     General Solution of y' = 2x
   │     is y = x² + C
   │
   │    ╱  C = 3
   │   ╱
   │  ╱   C = 2
   │ ╱
   │╱    C = 1
   ├──────────────── x
   │╲    C = 0
   │ ╲
   │  ╲  C = -1
   │   ╲
   │    ╲ C = -2
   │
   
   Each value of C gives one curve.
   All curves together = general solution.
```

---

## 2. Particular Solution

### Definition

> A **particular solution** is obtained from the general solution by assigning **specific values** to all arbitrary constants.

### How It Arises

Particular solutions come from **additional conditions** — typically:
- Initial conditions: values of $y$, $y'$, $y''$, ... at a single point
- Boundary conditions: values at two or more points

### Example

**ODE**: $y' = 2x$

**General solution**: $y = x^2 + C$

**Condition**: $y(1) = 5$

**Finding $C$**: $5 = 1^2 + C \implies C = 4$

**Particular solution**: $y = x^2 + 4$

```
   y
   │
   │    ╱  
   │   ╱  y = x² + 4  ← particular solution
   │  ╱  •(1, 5)       passes through (1, 5)
   │ ╱
   │╱
   ├──────────────── x
   │╲    other solutions don't
   │ ╲   pass through (1, 5)
```

---

## 3. Singular Solutions

### Definition

> A **singular solution** is a solution that **cannot** be obtained from the general solution for any value of the arbitrary constants.

Singular solutions are special — they typically arise as **envelopes** of the family of curves represented by the general solution.

### Classic Example: Clairaut's Equation

Consider $y = xy' + (y')^2$

**General solution**: $y = Cx + C^2$ (family of straight lines)

**Singular solution**: $y = -x^2/4$ (the envelope parabola)

```
   y
   │   \    /           Family of lines: y = Cx + C²
   │    \  /
   │     \/              Envelope (singular solution):
   │     /\              y = -x²/4
   │    /  \
   │   /    \←── parabola touching all lines
   ├────────────── x
   │
   │  The singular solution touches
   │  every member of the family
   │  but is not a member itself.
```

> Singular solutions are covered in detail in Unit 3 (Chapter 3.4).

---

## 4. Relationship Between Solution Types

```
                    Solutions of a DE
                          │
            ┌─────────────┼─────────────┐
            │             │             │
     General Solution  Particular   Singular
     (n constants)     Solutions    Solutions
            │             │             │
     Complete family   One specific  Cannot be
     of curves         curve from    obtained from
                       the family    general solution
            │             │             │
     y = x² + C      y = x² + 4    (if it exists)
```

### Key Properties

| Property | General | Particular | Singular |
|----------|---------|-----------|----------|
| Contains arbitrary constants | Yes ($n$) | No | No |
| Obtainable from general | — | Yes (fix constants) | **No** |
| Number of solutions | Infinite family | Single curve | Typically one |
| Always exists? | Yes | Yes (given conditions) | Not always |
| Found by | Solving the DE | Applying conditions | Envelope method |

---

## 5. Existence and Uniqueness

### The Big Question

> Given a DE and an initial condition, does a solution **exist**? If so, is it **unique**?

### Picard's Existence and Uniqueness Theorem (for first-order ODEs)

Consider the initial value problem:
$$\frac{dy}{dx} = f(x, y), \quad y(x_0) = y_0$$

**Theorem**: If $f(x, y)$ and $\dfrac{\partial f}{\partial y}$ are both **continuous** in a rectangular region $R$ containing $(x_0, y_0)$, then there exists a **unique** solution $y = \phi(x)$ in some interval containing $x_0$.

```
   y
   │     ┌─────────────┐
   │     │  Region R    │
   │     │    •(x₀,y₀)  │   If f and ∂f/∂y are
   │     │              │   continuous in R, then
   │     │   ───────    │   exactly ONE solution
   │     │  solution    │   curve passes through
   │     └─────────────┘   (x₀, y₀)
   ├──────────────────── x
```

### When Uniqueness Fails

**Example**: $y' = 3y^{2/3}$, $y(0) = 0$

Here $f(x,y) = 3y^{2/3}$ and $\dfrac{\partial f}{\partial y} = 2y^{-1/3}$, which is **not continuous** at $y = 0$.

This IVP has **multiple solutions**:
- $y = 0$ (trivial solution)
- $y = x^3$
- And infinitely many others!

> **Lesson**: When the conditions of the theorem are violated, multiple solutions may exist.

---

## 6. Completeness of Solutions

### Definition

> A general solution is **complete** if every solution of the DE (except possibly singular solutions) can be obtained from it by choosing appropriate constants.

### Example of Completeness

For $y' - y = 0$: General solution $y = Ce^x$

- Every solution is of this form (for some $C$, including $C = 0$)
- The general solution is complete

### Additional (Extraneous) Solutions

Sometimes during the solution process, we divide by expressions that could be zero, or we square both sides. This can:
- **Lose solutions** (division by zero)
- **Introduce extraneous solutions** (squaring)

Always verify solutions by substituting back into the original DE.

---

## 7. Worked Example: Complete Analysis

**Problem**: Solve $y' = y/x$ and find the general, particular (with $y(1) = 3$), and verify.

**Step 1**: Separate variables: $\dfrac{dy}{y} = \dfrac{dx}{x}$

**Step 2**: Integrate: $\ln|y| = \ln|x| + \ln|C|$

**Step 3**: **General solution**: $y = Cx$

**Step 4**: Apply $y(1) = 3$: $3 = C \cdot 1 \implies C = 3$

**Step 5**: **Particular solution**: $y = 3x$

**Verification**: $y' = 3$, $y/x = 3x/x = 3$ ✓

```
   y
   │    /
   │   / C=3  ← particular solution y = 3x
   │  /
   │ / C=1
   │/
   ├────────── x      General solution: y = Cx
   │\                 (all lines through origin)
   │ \ C=-1
   │  \
   │   \ C=-2
```

> **Note**: $y = 0$ (i.e., $C = 0$) is also included in the general solution.

---

## Summary Table

| Concept | Explanation |
|---------|-----------|
| **General solution** | Solution with $n$ arbitrary constants (for $n$-th order DE) |
| **Particular solution** | General solution with constants determined by conditions |
| **Singular solution** | Solution not obtainable from the general solution |
| **Existence theorem** | If $f$ and $\partial f / \partial y$ continuous → solution exists |
| **Uniqueness theorem** | Under same conditions → solution is unique |
| **Completeness** | General solution captures all non-singular solutions |
| **Envelope** | Curve tangent to every member of a family → singular solution |

---

## Quick Revision Questions

1. The general solution of a 3rd-order ODE contains how many arbitrary constants?

2. If $y = C_1 e^{2x} + C_2 e^{-x}$ is the general solution and $y(0) = 3$, $y'(0) = 0$, find the particular solution.

3. Explain why $y' = \sqrt{y}$, $y(0) = 0$ has more than one solution. Which theorem condition fails?

4. Is $y = 0$ a solution of $y' = y^2$? Is it a singular solution or can it be obtained from the general solution?

5. What is the geometric meaning of a singular solution in terms of the family of curves?

6. For the DE $y' + y = 0$, the general solution is $y = Ce^{-x}$. Verify that $y = 5e^{-x}$ is a particular solution.

---

[← Previous: Formation of DEs](03-formation-of-differential-equations.md) | [Back to README](../README.md) | [Next: Initial Value Problems →](05-initial-value-problems.md)
