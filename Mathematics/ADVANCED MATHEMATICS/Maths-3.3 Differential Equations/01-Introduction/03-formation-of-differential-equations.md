# Chapter 1.3: Formation of Differential Equations

[← Previous: Order and Degree](02-order-and-degree.md) | [Back to README](../README.md) | [Next: General and Particular Solutions →](04-general-and-particular-solutions.md)

---

## Overview

If a general solution (family of curves) gives rise to a differential equation, we should be able to **reverse the process** — start from a family of curves and derive the DE that represents it. This process is called **formation** (or **derivation**) of a differential equation.

---

## 1. The Core Idea

```
    Family of Curves                    Differential Equation
    (contains arbitrary                 (no arbitrary constants)
     constants)
                                  
    y = f(x, C₁, C₂, ..., Cₙ)  ────►  F(x, y, y', y'', ..., y⁽ⁿ⁾) = 0
                                  
         Forward: Eliminate            Backward: Integration 
         constants by                  introduces constants
         differentiation               (solving the DE)
```

### Fundamental Rule

> To eliminate **$n$ arbitrary constants** from a family of curves, differentiate **$n$ times** and then eliminate the constants from the resulting $(n+1)$ equations.

---

## 2. Method for Forming a DE

### Algorithm

```
Step 1: Start with the given equation containing n arbitrary constants
Step 2: Differentiate n times to get n additional equations
Step 3: Total equations = (n + 1): original + n derivatives
Step 4: Eliminate all n arbitrary constants from these (n + 1) equations
Step 5: The result is the DE of order n
```

### Why This Works

- Original equation: 1 equation, $n$ unknowns ($C_1, C_2, \ldots, C_n$)
- After $n$ differentiations: $(n+1)$ equations, $n$ unknowns
- We can eliminate $n$ unknowns from $(n+1)$ equations →  1 equation with 0 unknowns

---

## 3. One Arbitrary Constant (First-Order DEs)

### Example 1: $y = Cx^2$

**Step 1**: Given $y = Cx^2$ ... (i)

**Step 2**: Differentiate once:
$$y' = 2Cx \quad \ldots \text{(ii)}$$

**Step 3**: Eliminate $C$ from (i) and (ii):
- From (i): $C = \dfrac{y}{x^2}$
- Substitute in (ii): $y' = 2 \cdot \dfrac{y}{x^2} \cdot x = \dfrac{2y}{x}$

**Result**: $\boxed{xy' = 2y}$ or equivalently $xy' - 2y = 0$

> **Check**: Order = 1, and we eliminated 1 constant ✓

### Example 2: $y = A e^{3x}$

**Step 1**: Given $y = Ae^{3x}$ ... (i)

**Step 2**: Differentiate: $y' = 3Ae^{3x} = 3y$

**Result**: $\boxed{y' - 3y = 0}$

> Here $A$ was eliminated instantly because $y' = 3 \times (\text{original})$.

### Example 3: $x^2 + y^2 = r^2$ (circles centered at origin)

**Step 1**: $x^2 + y^2 = r^2$ ... (i) — one constant $r$

**Step 2**: Differentiate: $2x + 2yy' = 0$

**Result**: $\boxed{x + yy' = 0}$

> This DE represents **all circles centered at the origin**, regardless of radius.

---

## 4. Two Arbitrary Constants (Second-Order DEs)

### Example 4: $y = C_1 e^x + C_2 e^{-x}$

**Step 1**: $y = C_1 e^x + C_2 e^{-x}$ ... (i)

**Step 2**: Differentiate twice:
$$y' = C_1 e^x - C_2 e^{-x} \quad \ldots \text{(ii)}$$
$$y'' = C_1 e^x + C_2 e^{-x} \quad \ldots \text{(iii)}$$

**Step 3**: Compare (i) and (iii): $y'' = y$

**Result**: $\boxed{y'' - y = 0}$

> We didn't even need equation (ii)! Sometimes the elimination is immediate.

### Example 5: $y = C_1 \sin x + C_2 \cos x$

**Step 1**: $y = C_1 \sin x + C_2 \cos x$ ... (i)

**Step 2**: Differentiate:
$$y' = C_1 \cos x - C_2 \sin x \quad \ldots \text{(ii)}$$
$$y'' = -C_1 \sin x - C_2 \cos x \quad \ldots \text{(iii)}$$

**Step 3**: Add (i) and (iii):
$$y + y'' = (C_1 \sin x + C_2 \cos x) + (-C_1 \sin x - C_2 \cos x) = 0$$

**Result**: $\boxed{y'' + y = 0}$

> This is the **simple harmonic motion** equation — one of the most important in physics!

### Example 6: $y = Ax^2 + Bx$ (two constants)

**Step 1**: $y = Ax^2 + Bx$ ... (i)

**Step 2**: $y' = 2Ax + B$ ... (ii)

**Step 3**: $y'' = 2A$ ... (iii)

**Step 4**: From (iii): $A = y''/2$

From (ii): $B = y' - 2Ax = y' - xy''$

Substitute into (i): $y = \dfrac{y''}{2}x^2 + (y' - xy'')x$

$$y = \frac{x^2 y''}{2} + xy' - x^2 y''$$

$$y = xy' - \frac{x^2 y''}{2}$$

**Result**: $\boxed{x^2 y'' - 2xy' + 2y = 0}$

> This is a **Cauchy-Euler equation** (covered in Unit 6).

---

## 5. Using Determinant Method

For families involving constants linearly, we can use a **determinant** to eliminate constants elegantly.

### Method

If the original equation and its derivatives form a linear system in the constants, set up the determinant:

For $y = C_1 f_1(x) + C_2 f_2(x)$, the system from (i), (ii), (iii) is:

$$\begin{vmatrix} y & f_1 & f_2 \\ y' & f_1' & f_2' \\ y'' & f_1'' & f_2'' \end{vmatrix} = 0$$

### Example: $y = C_1 e^{2x} + C_2 e^{-3x}$

$$\begin{vmatrix} y & e^{2x} & e^{-3x} \\ y' & 2e^{2x} & -3e^{-3x} \\ y'' & 4e^{2x} & 9e^{-3x} \end{vmatrix} = 0$$

Expanding along the first column:

$$y(2 \cdot 9 \cdot e^{-x} - (-3) \cdot 4 \cdot e^{-x}) - y'(9e^{-x} - (-3) \cdot e^{-x} \cdot \ldots)$$

It's easier to just differentiate twice and eliminate:

- $y = C_1 e^{2x} + C_2 e^{-3x}$ ... (i)
- $y' = 2C_1 e^{2x} - 3C_2 e^{-3x}$ ... (ii)
- $y'' = 4C_1 e^{2x} + 9C_2 e^{-3x}$ ... (iii)

From (i) and (ii): (ii) + 3(i) gives $y' + 3y = 5C_1 e^{2x}$ → $C_1 e^{2x} = \dfrac{y' + 3y}{5}$

From (i) and (ii): (ii) − 2(i) gives $y' - 2y = -5C_2 e^{-3x}$ → $C_2 e^{-3x} = \dfrac{2y - y'}{5}$

Substitute into (iii):
$$y'' = 4 \cdot \frac{y' + 3y}{5} + 9 \cdot \frac{2y - y'}{5} = \frac{4y' + 12y + 18y - 9y'}{5} = \frac{-5y' + 30y}{5}$$

$$y'' = -y' + 6y$$

**Result**: $\boxed{y'' + y' - 6y = 0}$

> **Verification**: Characteristic equation $r^2 + r - 6 = (r+3)(r-2) = 0$ gives $r = 2, -3$ ✓

---

## 6. Special Cases and Techniques

### Case 1: Constant appears as an exponent

**Example**: $y = e^{Cx}$

Taking $\ln$: $\ln y = Cx$ → $C = \dfrac{\ln y}{x}$

Differentiate the original: $y' = Ce^{Cx} = Cy$

So $C = \dfrac{y'}{y}$, and also $C = \dfrac{\ln y}{x}$

$$\frac{y'}{y} = \frac{\ln y}{x} \implies xy' = y \ln y$$

**Result**: $\boxed{xy' - y\ln y = 0}$

### Case 2: Constant inside a trigonometric function

**Example**: $y = \sin(x + C)$

Differentiate: $y' = \cos(x + C)$

Use identity: $\sin^2(x+C) + \cos^2(x+C) = 1$

$$y^2 + (y')^2 = 1$$

**Result**: $\boxed{y^2 + (y')^2 = 1}$

### Case 3: Implicit equation

**Example**: $xy = C$ (rectangular hyperbolas)

Differentiate: $y + xy' = 0$

**Result**: $\boxed{y + xy' = 0}$

---

## 7. Summary of Procedure

```
         Given: Family of curves F(x, y, C₁, ..., Cₙ) = 0
                          │
                  ┌───────┴───────┐
                  │  How many     │
                  │  constants?   │
                  └───────┬───────┘
                          │
              ┌───────────┼───────────┐
              │           │           │
          n = 1       n = 2       n = k
              │           │           │
         Diff 1x     Diff 2x     Diff kx
         (2 eqns)    (3 eqns)    (k+1 eqns)
              │           │           │
         Eliminate    Eliminate   Eliminate
          1 const    2 consts    k consts
              │           │           │
         Order 1      Order 2     Order k
           DE           DE          DE
```

---

## Summary Table

| Scenario | Constants | Differentiations | Equations | Resulting Order |
|----------|-----------|-----------------|-----------|-----------------|
| $y = Cf(x)$ | 1 | 1 | 2 | 1 |
| $y = C_1 f_1 + C_2 f_2$ | 2 | 2 | 3 | 2 |
| $y = C_1 f_1 + C_2 f_2 + C_3 f_3$ | 3 | 3 | 4 | 3 |
| General: $n$ constants | $n$ | $n$ | $n+1$ | $n$ |

| Technique | When to Use |
|-----------|-------------|
| Direct substitution | Constants appear linearly |
| Logarithmic differentiation | Constant in exponent ($e^{Cx}$, $x^C$) |
| Trigonometric identity | Constant in trig argument |
| Determinant method | Constants appear linearly (systematic) |

---

## Quick Revision Questions

1. Form the DE for the family of parabolas $y^2 = 4ax$ ($a$ is the arbitrary constant).

2. Eliminate $A$ and $B$ from $y = A\cos 3x + B\sin 3x$ to form the corresponding DE.

3. How many times must you differentiate a family containing **4** arbitrary constants?

4. Form the DE from $y = Ce^x + De^{2x}$ (two constants).

5. Find the DE of all circles of radius 5 centered on the x-axis: $(x-a)^2 + y^2 = 25$.

6. Why does the family $y = mx$ (lines through the origin) give a **first-order** DE, but $y = mx + c$ (general lines) gives a **second-order** DE?

---

[← Previous: Order and Degree](02-order-and-degree.md) | [Back to README](../README.md) | [Next: General and Particular Solutions →](04-general-and-particular-solutions.md)
