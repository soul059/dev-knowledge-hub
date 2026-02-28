# Chapter 3.1: Equations Solvable for p, y, x

[← Previous: Integrating Factors](../02-First-Order-First-Degree/07-integrating-factors.md) | [Back to README](../README.md) | [Next: Clairaut's Equation →](02-clairauts-equation.md)

---

## Overview

First-order ODEs of **degree greater than 1** take the form $F(x, y, p) = 0$ where $p = dy/dx$. These cannot always be reduced to the methods of Unit 2. The approach depends on whether we can solve the equation explicitly for $p$, $y$, or $x$.

---

## 1. General Form

A first-order ODE of degree $n$:

$$F(x, y, p) = 0 \quad \text{where } p = \frac{dy}{dx}$$

This may be written as:

$$p^n + a_1(x,y) p^{n-1} + \cdots + a_{n-1}(x,y) p + a_n(x,y) = 0$$

---

## 2. Equations Solvable for $p$

### Method

If $F(x, y, p) = 0$ can be factored or solved to give:

$$p = f_1(x, y), \quad p = f_2(x, y), \quad \ldots, \quad p = f_n(x, y)$$

Then solve **each** first-order equation separately and combine the solutions.

### General Solution

If $\phi_1(x, y, C) = 0$, $\phi_2(x, y, C) = 0$, ..., $\phi_n(x, y, C) = 0$ are the solutions, then the general solution is:

$$\phi_1(x, y, C) \cdot \phi_2(x, y, C) \cdots \phi_n(x, y, C) = 0$$

### Example 1: $p^2 - 5p + 6 = 0$

**Factor**: $(p - 2)(p - 3) = 0$

**Case 1**: $p = 2 \implies dy/dx = 2 \implies y = 2x + C$

**Case 2**: $p = 3 \implies dy/dx = 3 \implies y = 3x + C$

**General solution**: $(y - 2x - C)(y - 3x - C) = 0$

> The same constant $C$ appears in both factors.

### Example 2: $p^2 - (x + y)p + xy = 0$

**Factor**: $(p - x)(p - y) = 0$

**Case 1**: $p = x \implies dy = x \, dx \implies y = x^2/2 + C$

**Case 2**: $p = y \implies dy/y = dx \implies y = Ce^x$

**General solution**: $(2y - x^2 - C)(y - Ce^x) = 0$

### Example 3: $p^2 + 2xp - 3x^2 = 0$

**Factor**: $(p + 3x)(p - x) = 0$

**Case 1**: $p = -3x \implies y = -3x^2/2 + C$

**Case 2**: $p = x \implies y = x^2/2 + C$

---

## 3. Equations Solvable for $y$

### Method

If the equation can be written as $y = f(x, p)$:

**Step 1**: Differentiate with respect to $x$:
$$p = \frac{dy}{dx} = \frac{\partial f}{\partial x} + \frac{\partial f}{\partial p} \cdot \frac{dp}{dx}$$

**Step 2**: This gives an ODE in $x$ and $p$. Solve for $p$ as a function of $x$ (with constant $C$).

**Step 3**: Substitute back into $y = f(x, p)$ to get the solution.

### Process

```
y = f(x, p)     ────differentiate w.r.t. x────►     ODE in x and p
                                                          │
                                                     Solve for p = g(x, C)
                                                          │
                                                     Substitute back
                                                          │
                                             y = f(x, g(x, C)) = solution
```

### Example 4: $y = 2px + p^2$

**Differentiate** w.r.t. $x$:
$$p = 2p + 2x\frac{dp}{dx} + 2p\frac{dp}{dx}$$

$$p = 2p + (2x + 2p)\frac{dp}{dx}$$

$$-p = (2x + 2p)\frac{dp}{dx}$$

$$\frac{dp}{dx} = \frac{-p}{2(x + p)}$$

This is a first-order ODE in $p$ and $x$. Let's try $x = vp$:

$\dfrac{dx}{dp} = \dfrac{-2(x+p)}{p} = \dfrac{-2x}{p} - 2$

$\dfrac{dx}{dp} + \dfrac{2x}{p} = -2$ — this is linear in $x$!

I.F. $= e^{\int 2/p \, dp} = p^2$

$\dfrac{d}{dp}(xp^2) = -2p^2$

$xp^2 = -\dfrac{2p^3}{3} + C$

$x = -\dfrac{2p}{3} + \dfrac{C}{p^2}$

Parametric solution with parameter $p$:
$$x = -\frac{2p}{3} + \frac{C}{p^2}, \quad y = 2px + p^2$$

---

## 4. Equations Solvable for $x$

### Method

If the equation can be written as $x = f(y, p)$:

**Step 1**: Differentiate with respect to $y$:
$$\frac{1}{p} = \frac{dx}{dy} = \frac{\partial f}{\partial y} + \frac{\partial f}{\partial p} \cdot \frac{dp}{dy}$$

**Step 2**: Solve the resulting ODE in $y$ and $p$.

**Step 3**: Substitute back.

### Example 5: $x = y + p^2$

**Differentiate** w.r.t. $y$ (since $dx/dy = 1/p$):
$$\frac{1}{p} = 1 + 2p\frac{dp}{dy}$$

$$2p\frac{dp}{dy} = \frac{1}{p} - 1 = \frac{1 - p}{p}$$

$$\frac{2p^2}{1 - p} \, dp = dy$$

This can be separated and integrated. Use partial fractions:

$$\frac{2p^2}{1-p} = -2p - 2 + \frac{2}{1-p} = -2(p + 1) + \frac{2}{1-p}$$

Integrate:
$$y = -p^2 - 2p - 2\ln|1-p| + C$$

The solution is in parametric form: $x = y + p^2$ and the relation above.

---

## 5. Summary of Approaches

```
Given: F(x, y, p) = 0     (higher degree in p)
              │
    ┌─────────┼─────────┐
    │         │         │
 Solvable  Solvable  Solvable
  for p     for y     for x
    │         │         │
    ▼         ▼         ▼
 Factor:   y=f(x,p)  x=f(y,p)
 p=f₁,f₂  Diff w.r.t  Diff w.r.t
    │      x→ODE in   y→ODE in
    ▼      x,p        y,p
 Solve each    │         │
 1st order     ▼         ▼
 DE         Parametric Parametric
    │       solution   solution
    ▼       (x,y in    (x,y in
 Product     terms      terms
 of factors  of p)      of p)
```

---

## Summary Table

| Approach | When to Use | Key Step |
|----------|-------------|----------|
| **Solvable for $p$** | Can factor/solve $F = 0$ for $p$ | Solve each $p = f_i(x,y)$ separately |
| **Solvable for $y$** | Can write $y = f(x, p)$ | Differentiate w.r.t. $x$, get ODE in $(x, p)$ |
| **Solvable for $x$** | Can write $x = f(y, p)$ | Differentiate w.r.t. $y$, get ODE in $(y, p)$ |
| **Combined solution** | Solvable for $p$ | Product of all factor solutions |
| **Parametric form** | Solvable for $y$ or $x$ | Express $x, y$ both in terms of $p$ |

---

## Quick Revision Questions

1. Solve: $p^2 - 7p + 12 = 0$ (find the general solution).

2. For $p^2 = x + y$, which method (solvable for $p$, $y$, or $x$) is most natural?

3. Solve: $p^2 + p - 2 = 0$ (factor and solve).

4. Given $y = xp + p^3$, differentiate w.r.t. $x$ and set up the resulting ODE.

5. If $F(x,y,p) = 0$ factors into 3 linear factors in $p$, how many first-order DEs do you solve?

6. Why are solutions of higher-degree first-order ODEs often given in **parametric form**?

---

[← Previous: Integrating Factors](../02-First-Order-First-Degree/07-integrating-factors.md) | [Back to README](../README.md) | [Next: Clairaut's Equation →](02-clairauts-equation.md)
