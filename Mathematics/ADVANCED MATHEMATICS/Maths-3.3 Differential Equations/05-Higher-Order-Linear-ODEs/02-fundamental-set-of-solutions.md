# Chapter 5.2: Fundamental Set of Solutions

[← Previous: Wronskian and Linear Independence](01-wronskian-and-linear-independence.md) | [Back to README](../README.md) | [Next: Reduction of Order →](03-reduction-of-order.md)

---

## Overview

A fundamental set (or fundamental system) of solutions forms a basis for the solution space of a linear homogeneous ODE. Every solution can be expressed as a linear combination of these fundamental solutions.

---

## 1. The $n$th-Order Linear Homogeneous ODE

### Standard Form

$$\boxed{a_n(x)y^{(n)} + a_{n-1}(x)y^{(n-1)} + \cdots + a_1(x)y' + a_0(x)y = 0}$$

Or in operator notation: $L[y] = 0$

### Existence and Uniqueness Theorem

If $a_0, a_1, \ldots, a_n$ are continuous on $I$ and $a_n(x) \neq 0$ on $I$, then the IVP:

$$L[y] = 0, \quad y(x_0) = b_0, \; y'(x_0) = b_1, \; \ldots, \; y^{(n-1)}(x_0) = b_{n-1}$$

has a **unique** solution on $I$.

---

## 2. Superposition Principle

### Theorem

If $y_1, y_2, \ldots, y_k$ are solutions of $L[y] = 0$, then for any constants $c_1, c_2, \ldots, c_k$:

$$\boxed{y = c_1 y_1 + c_2 y_2 + \cdots + c_k y_k}$$

is also a solution.

```
SUPERPOSITION PRINCIPLE

  Solution y₁  ─────┐
                     │
  Solution y₂  ─────┼─── c₁y₁ + c₂y₂ + ... + cₖyₖ  ──→  Also a solution!
                     │
  Solution yₖ  ─────┘

  (Only works for LINEAR HOMOGENEOUS equations)
```

> **Warning**: Superposition does NOT hold for:
> - Non-homogeneous equations ($L[y] = g(x) \neq 0$)
> - Nonlinear equations

---

## 3. Fundamental Set of Solutions

### Definition

A set $\{y_1, y_2, \ldots, y_n\}$ of $n$ solutions of an $n$th-order linear homogeneous ODE is called a **fundamental set** if they are linearly independent.

### Why Exactly $n$ Solutions?

| Order of ODE | Dimension of solution space | # functions in fundamental set |
|:---:|:---:|:---:|
| 1 | 1 | 1 |
| 2 | 2 | 2 |
| 3 | 3 | 3 |
| $n$ | $n$ | $n$ |

### General Solution

If $\{y_1, y_2, \ldots, y_n\}$ is a fundamental set, then the **general solution** is:

$$\boxed{y = c_1 y_1 + c_2 y_2 + \cdots + c_n y_n}$$

where $c_1, \ldots, c_n$ are arbitrary constants.

```
SOLUTION SPACE STRUCTURE (2nd order example)

  y₂
  ↑          Every point (c₁, c₂) gives a
  │  •       particular solution y = c₁y₁ + c₂y₂
  │    •
  │      •   The fundamental set {y₁, y₂}
  │        • spans this 2D space
  │──────────→ y₁
  │
  │  This is a VECTOR SPACE!
```

### Fundamental Set Existence Theorem

For any $n$th-order linear homogeneous ODE with continuous coefficients on $I$, a fundamental set of solutions **always exists**.

---

## 4. Constructing a Fundamental Set

### Method: Choose $n$ Linearly Independent IVPs

At $x_0 \in I$, define $n$ solutions by:

$$y_1: \; y(x_0) = 1, \; y'(x_0) = 0, \; y''(x_0) = 0, \ldots$$
$$y_2: \; y(x_0) = 0, \; y'(x_0) = 1, \; y''(x_0) = 0, \ldots$$
$$y_n: \; y(x_0) = 0, \; y'(x_0) = 0, \; \ldots, \; y^{(n-1)}(x_0) = 1$$

The Wronskian at $x_0$ is the identity matrix determinant $= 1 \neq 0$, so these are LI.

---

## 5. Worked Examples

### Example 1: $y'' - 5y' + 6y = 0$

**Verify**: $y_1 = e^{2x}$ and $y_2 = e^{3x}$ form a fundamental set.

Check solutions:
- $y_1'': (e^{2x})'' - 5(e^{2x})' + 6e^{2x} = 4e^{2x} - 10e^{2x} + 6e^{2x} = 0$ ✓
- $y_2'': (e^{3x})'' - 5(e^{3x})' + 6e^{3x} = 9e^{3x} - 15e^{3x} + 6e^{3x} = 0$ ✓

Wronskian: $W = e^{2x}(3e^{3x}) - e^{3x}(2e^{2x}) = e^{5x} \neq 0$ ✓

**General solution**: $y = c_1 e^{2x} + c_2 e^{3x}$

### Example 2: $y'' + y = 0$

Fundamental set: $\{\sin x, \cos x\}$

$W = -\sin^2 x - \cos^2 x = -1 \neq 0$

**General solution**: $y = c_1 \cos x + c_2 \sin x$

### Example 3: Third-Order — $y''' - 6y'' + 11y' - 6y = 0$

Verify $\{e^x, e^{2x}, e^{3x}\}$:

$$W = \begin{vmatrix} e^x & e^{2x} & e^{3x} \\ e^x & 2e^{2x} & 3e^{3x} \\ e^x & 4e^{2x} & 9e^{3x} \end{vmatrix}$$

$$= e^{6x}\begin{vmatrix} 1 & 1 & 1 \\ 1 & 2 & 3 \\ 1 & 4 & 9 \end{vmatrix}$$

$$= e^{6x}[1(18-12) - 1(9-3) + 1(4-2)] = e^{6x}[6 - 6 + 2] = 2e^{6x} \neq 0$$

**General solution**: $y = c_1 e^x + c_2 e^{2x} + c_3 e^{3x}$

### Example 4: Particular Solution from Initial Conditions

For $y'' - 5y' + 6y = 0$ with $y(0) = 1$, $y'(0) = 0$:

$y = c_1 e^{2x} + c_2 e^{3x}$

$y(0) = c_1 + c_2 = 1$

$y'(0) = 2c_1 + 3c_2 = 0$

Solving: $c_1 = 3$, $c_2 = -2$

$$\boxed{y = 3e^{2x} - 2e^{3x}}$$

---

## 6. Solution Space as a Vector Space

The set of all solutions of $L[y] = 0$ forms a **vector space** of dimension $n$:

| Vector Space Axiom | Solution Space Analog |
|--------------------|-----------------------|
| Closure under addition | Sum of solutions is a solution |
| Closure under scalar multiplication | Constant × solution is a solution |
| Zero vector | $y = 0$ (trivial solution) |
| Basis | Fundamental set |
| Dimension | Order of the ODE |
| Coordinates | Constants $c_1, c_2, \ldots, c_n$ |

---

## 7. Non-Uniqueness of Fundamental Sets

The fundamental set is **not unique**. For $y'' + y = 0$:

- $\{\sin x, \cos x\}$ ← standard
- $\{\sin(x + 1), \cos(x + 1)\}$ ← shifted
- $\{e^{ix}, e^{-ix}\}$ ← complex exponential
- $\{\sin x + \cos x, \sin x - \cos x\}$ ← rotated

All are valid fundamental sets (all LI), giving the same general solution written differently.

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Superposition** | $c_1y_1 + c_2y_2$ is a solution if $y_1, y_2$ are |
| **Fundamental set** | $n$ LI solutions of an $n$th-order ODE |
| **LI test** | $W \neq 0$ at one point |
| **General solution** | $y = c_1y_1 + \cdots + c_ny_n$ |
| **Solution space** | $n$-dimensional vector space |
| **IVP solution** | Determine $c_1, \ldots, c_n$ from initial conditions |
| **Uniqueness** | Fundamental set is not unique; general solution is |

---

## Quick Revision Questions

1. Show that $\{e^x, xe^x\}$ is a fundamental set for $y'' - 2y' + y = 0$.

2. What is the dimension of the solution space for a 4th-order linear homogeneous ODE?

3. If $y_1 = e^{2x}$ is one solution of a 2nd-order ODE, and $W = e^{5x}$, find $y_2$.

4. Does the superposition principle apply to $y'' + y = \sin x$? Why or why not?

5. For $y'' + 4y = 0$, verify that $\{\cos 2x, \sin 2x\}$ and $\{e^{2ix}, e^{-2ix}\}$ both form fundamental sets.

6. Given $y_1 = x$ and $y_2 = x^2$ as solutions of a 2nd-order ODE on $(0, \infty)$, find the general solution and solve the IVP $y(1) = 3$, $y'(1) = 5$.

---

[← Previous: Wronskian and Linear Independence](01-wronskian-and-linear-independence.md) | [Back to README](../README.md) | [Next: Reduction of Order →](03-reduction-of-order.md)
