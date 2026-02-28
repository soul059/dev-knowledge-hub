# Chapter 2.7: Integrating Factors

[← Previous: Exact Equations](06-exact-equations.md) | [Back to README](../README.md) | [Next: Solvable for p, y, x →](../03-First-Order-Higher-Degree/01-solvable-for-p-y-x.md)

---

## Overview

When a first-order ODE $M \, dx + N \, dy = 0$ is **not** exact, we can often find an **integrating factor** $\mu$ such that $\mu M \, dx + \mu N \, dy = 0$ **becomes** exact. This chapter develops systematic rules for finding integrating factors.

---

## 1. Concept

### Definition

> An **integrating factor (I.F.)** is a function $\mu(x, y)$ such that when we multiply the equation $M \, dx + N \, dy = 0$ by $\mu$, the resulting equation $\mu M \, dx + \mu N \, dy = 0$ is exact.

### The Condition

For $\mu M \, dx + \mu N \, dy = 0$ to be exact:

$$\frac{\partial(\mu M)}{\partial y} = \frac{\partial(\mu N)}{\partial x}$$

Expanding:

$$\mu \frac{\partial M}{\partial y} + M \frac{\partial \mu}{\partial y} = \mu \frac{\partial N}{\partial x} + N \frac{\partial \mu}{\partial x}$$

This is a PDE for $\mu(x, y)$ — generally hard to solve. However, if we restrict $\mu$ to be a function of **one variable only**, the problem simplifies greatly.

---

## 2. Rules for Finding Integrating Factors

### Rule 1: I.F. is a function of $x$ only — $\mu = \mu(x)$

If $\dfrac{\frac{\partial M}{\partial y} - \frac{\partial N}{\partial x}}{N}$ is a function of $x$ **only**, then:

$$\boxed{\mu(x) = e^{\int \frac{M_y - N_x}{N} \, dx}}$$

### Rule 2: I.F. is a function of $y$ only — $\mu = \mu(y)$

If $\dfrac{\frac{\partial N}{\partial x} - \frac{\partial M}{\partial y}}{M}$ is a function of $y$ **only**, then:

$$\boxed{\mu(y) = e^{\int \frac{N_x - M_y}{M} \, dy}}$$

### Decision Flowchart

```
Given: M dx + N dy = 0 (NOT exact)
Compute: M_y - N_x
              │
              ▼
     ┌────────┴────────┐
     │                 │
  (M_y - N_x)/N     (N_x - M_y)/M
  = f(x) only?      = g(y) only?
     │                 │
   ┌─┴─┐            ┌─┴─┐
  YES  NO           YES  NO
   │    │            │    │
   ▼    │            ▼    │
 μ=e^∫f(x)dx        μ=e^∫g(y)dy
   │    │            │    │
   │    └────┬───────┘    │
   │         │            │
   │    Try special       │
   │    forms (Rules      │
   │    3, 4, 5)         │
   │                     │
   ▼                     ▼
 Multiply by μ      Try other methods
 and solve as exact
```

### Rule 3: I.F. is $\mu = x^m y^n$

If the equation has terms like $x^a y^b$, try $\mu = x^m y^n$ and determine $m, n$ by forcing exactness.

### Rule 4: Common Recognizable I.F.s

| Pattern | Integrating Factor | Makes it become |
|---------|-------------------|-----------------|
| $x \, dy - y \, dx$ | $1/x^2$ | $d(y/x)$ |
| $x \, dy - y \, dx$ | $1/y^2$ | $-d(x/y)$ |
| $x \, dy - y \, dx$ | $1/(x^2+y^2)$ | $d(\arctan(y/x))$ |
| $x \, dy - y \, dx$ | $1/(xy)$ | $d(\ln(y/x))$ |
| $x \, dx + y \, dy$ | $1/(x^2+y^2)$ | $\frac{1}{2}d(\ln(x^2+y^2))$ |

---

## 3. Worked Examples

### Example 1: I.F. as function of $x$

**Solve**: $(2xy + 3y^2) \, dx + (x^2 + 6xy) \, dy = 0$

**Test exactness**:
$$M_y = 2x + 6y, \quad N_x = 2x + 6y$$

Wait — this IS exact! Let me use a non-exact example.

**Solve**: $(y^2 + y) \, dx + (x) \, dy = 0$

**Test**: $M_y = 2y + 1$, $N_x = 1$

$M_y - N_x = 2y \neq 0$ → not exact.

$\dfrac{M_y - N_x}{N} = \dfrac{2y}{x}$ → depends on both $x$ and $y$. Rule 1 fails.

$\dfrac{N_x - M_y}{M} = \dfrac{1 - 2y - 1}{y^2 + y} = \dfrac{-2y}{y(y+1)} = \dfrac{-2}{y+1}$ → function of $y$ only ✓

**Rule 2**: $\mu = e^{\int \frac{-2}{y+1} dy} = e^{-2\ln|y+1|} = \dfrac{1}{(y+1)^2}$

**Multiply**: $\dfrac{y^2 + y}{(y+1)^2} \, dx + \dfrac{x}{(y+1)^2} \, dy = 0$

$$\frac{y(y+1)}{(y+1)^2} \, dx + \frac{x}{(y+1)^2} \, dy = 0$$

$$\frac{y}{y+1} \, dx + \frac{x}{(y+1)^2} \, dy = 0$$

**Verify**: $\tilde{M} = \dfrac{y}{y+1}$, $\tilde{N} = \dfrac{x}{(y+1)^2}$

$\tilde{M}_y = \dfrac{1}{(y+1)^2}$, $\tilde{N}_x = \dfrac{1}{(y+1)^2}$ ✓ Exact!

**Solve**: $F = \int \tilde{M} \, dx = \dfrac{xy}{y+1} + g(y)$

$F_y = \dfrac{x}{(y+1)^2} + g'(y) = \dfrac{x}{(y+1)^2}$

$g'(y) = 0 \implies g(y) = 0$

**Solution**: $\boxed{\dfrac{xy}{y+1} = C}$

### Example 2: I.F. as function of $x$

**Solve**: $y \, dx - x \, dy + 2x^2 y \, dx = 0$

Rearrange: $(y + 2x^2 y) \, dx + (-x) \, dy = 0$

$M = y(1 + 2x^2)$, $N = -x$

$M_y = 1 + 2x^2$, $N_x = -1$

$\dfrac{M_y - N_x}{N} = \dfrac{1 + 2x^2 + 1}{-x} = \dfrac{2 + 2x^2}{-x} = \dfrac{-2(1+x^2)}{x}$

This depends only on $x$... but it's complicated. Let's try recognizing patterns instead.

Rewrite: $y \, dx - x \, dy + 2x^2 y \, dx = 0$

Multiply by $\dfrac{1}{x^2}$: $\dfrac{y \, dx - x \, dy}{x^2} + 2y \, dx = 0$

Recognize: $\dfrac{y \, dx - x \, dy}{x^2} = -d\left(\dfrac{y}{x}\right)$... wait, actually $d(y/x) = \dfrac{x\,dy - y\,dx}{x^2}$, so $\dfrac{y\,dx - x\,dy}{x^2} = -d(y/x)$.

$$-d\left(\frac{y}{x}\right) + 2y \, dx = 0$$

This doesn't separate cleanly. Let's try $\mu = 1/y^2$:

$\dfrac{1}{y} \, dx - \dfrac{x}{y^2} \, dy + 2x^2 \, dx = 0$

$(1/y + 2x^2) \, dx - (x/y^2) \, dy = 0$

$M_y = -1/y^2$, $N_x = -1/y^2$ ✓ Exact!

**Solve**: $F = \int (1/y + 2x^2) \, dx = x/y + 2x^3/3 + g(y)$

$F_y = -x/y^2 + g'(y) = -x/y^2$

$g'(y) = 0$

**Solution**: $\boxed{\dfrac{x}{y} + \dfrac{2x^3}{3} = C}$

### Example 3: Using Rule 1

**Solve**: $(x^2 + y^2 + x) \, dx + xy \, dy = 0$

$M_y = 2y$, $N_x = y$

$\dfrac{M_y - N_x}{N} = \dfrac{2y - y}{xy} = \dfrac{1}{x}$ → function of $x$ only ✓

$\mu = e^{\int dx/x} = x$

**Multiply by $x$**: $(x^3 + xy^2 + x^2) \, dx + x^2 y \, dy = 0$

**Verify**: $\tilde{M}_y = 2xy = \tilde{N}_x$ ✓

$F = \int x^2 y \, dy = \dfrac{x^2 y^2}{2} + h(x)$

$F_x = xy^2 + h'(x) = x^3 + xy^2 + x^2$

$h'(x) = x^3 + x^2 \implies h(x) = \dfrac{x^4}{4} + \dfrac{x^3}{3}$

**Solution**: $\boxed{\dfrac{x^2 y^2}{2} + \dfrac{x^4}{4} + \dfrac{x^3}{3} = C}$

---

## 4. Summary of All I.F. Rules

```
Not Exact: M dx + N dy = 0
                │
   ┌────────────┼────────────────┐
   │            │                │
Rule 1:      Rule 2:          Rule 3/4:
(M_y-N_x)/N  (N_x-M_y)/M     Pattern
= f(x)?      = g(y)?         recognition
   │            │                │
   ▼            ▼                ▼
μ=e^∫f dx   μ=e^∫g dy      Try x^m y^n or
                             recognize d(y/x),
                             d(xy), etc.
```

---

## Summary Table

| Rule | Condition | I.F. Formula |
|------|-----------|-------------|
| **Rule 1** | $(M_y - N_x) / N = f(x)$ | $\mu = e^{\int f(x) \, dx}$ |
| **Rule 2** | $(N_x - M_y) / M = g(y)$ | $\mu = e^{\int g(y) \, dy}$ |
| **Rule 3** | Polynomial terms | Try $\mu = x^m y^n$ |
| **Rule 4** | $(x \, dy - y \, dx)$ pattern | $1/x^2$, $1/y^2$, $1/(x^2+y^2)$, $1/(xy)$ |
| **After finding I.F.** | Multiply through | Solve as exact equation |

---

## Quick Revision Questions

1. Find the I.F. for $(y + 1) \, dx - x \, dy = 0$ and solve.

2. For $(2y) \, dx + (x) \, dy = 0$, compute $(M_y - N_x)/N$. What is the I.F.?

3. If $(M_y - N_x)/N = 3$, what is the integrating factor?

4. Show that $1/(x^2 + y^2)$ is an I.F. for $y \, dx - x \, dy = 0$.

5. Find an I.F. of the form $x^m y^n$ for $(3y) \, dx + (2x) \, dy = 0$.

6. Why is finding a general I.F. $\mu(x,y)$ harder than finding $\mu(x)$ or $\mu(y)$?

---

[← Previous: Exact Equations](06-exact-equations.md) | [Back to README](../README.md) | [Next: Solvable for p, y, x →](../03-First-Order-Higher-Degree/01-solvable-for-p-y-x.md)
