# Chapter 5.1: Wronskian and Linear Independence

[← Previous: Geometric Applications](../04-Applications-First-Order/05-geometric-applications.md) | [Back to README](../README.md) | [Next: Fundamental Set of Solutions →](02-fundamental-set-of-solutions.md)

---

## Overview

The Wronskian determinant is the key tool for testing whether a set of functions (solutions) is linearly independent. It underpins the entire theory of higher-order linear ODEs, determining whether we have "enough" solutions to form the general solution.

---

## 1. Linear Dependence and Independence

### Definitions

A set of functions $\{y_1, y_2, \ldots, y_n\}$ defined on an interval $I$ is:

- **Linearly dependent (LD)** if there exist constants $c_1, c_2, \ldots, c_n$, **not all zero**, such that:
$$c_1 y_1 + c_2 y_2 + \cdots + c_n y_n = 0 \quad \forall\, x \in I$$

- **Linearly independent (LI)** if the only such constants are $c_1 = c_2 = \cdots = c_n = 0$.

### Intuition

```
LINEARLY DEPENDENT                    LINEARLY INDEPENDENT
(one is a combo of others)            (no redundancy)

  y₁ = 2x                              y₁ = eˣ
  y₂ = 6x    ← y₂ = 3·y₁              y₂ = e²ˣ
                                        (no constant multiple relation)
  3y₁ - y₂ = 0  (c₁=3, c₂=-1)         c₁eˣ + c₂e²ˣ = 0 ⟹ c₁=c₂=0
```

### Quick Tests for Two Functions

- $y_1$ and $y_2$ are LD $\iff$ $y_1/y_2 = $ constant (or $y_2/y_1 = $ constant)
- $y_1$ and $y_2$ are LI $\iff$ $y_1/y_2$ is **not** constant

---

## 2. The Wronskian

### Definition for Two Functions

$$W(y_1, y_2) = \begin{vmatrix} y_1 & y_2 \\ y_1' & y_2' \end{vmatrix} = y_1 y_2' - y_2 y_1'$$

### Definition for $n$ Functions

$$W(y_1, y_2, \ldots, y_n) = \begin{vmatrix} y_1 & y_2 & \cdots & y_n \\ y_1' & y_2' & \cdots & y_n' \\ \vdots & \vdots & \ddots & \vdots \\ y_1^{(n-1)} & y_2^{(n-1)} & \cdots & y_n^{(n-1)} \end{vmatrix}$$

### Wronskian Matrix Structure

```
Row 0:   y₁        y₂        ...  yₙ         ← functions
Row 1:   y₁'       y₂'       ...  yₙ'        ← 1st derivatives
Row 2:   y₁''      y₂''      ...  yₙ''       ← 2nd derivatives
  ⋮       ⋮         ⋮              ⋮
Row n-1: y₁⁽ⁿ⁻¹⁾  y₂⁽ⁿ⁻¹⁾  ...  yₙ⁽ⁿ⁻¹⁾   ← (n-1)th derivatives
```

---

## 3. Key Theorems

### Theorem 1: Wronskian Test for Solutions

If $y_1, y_2, \ldots, y_n$ are solutions of an $n$th-order linear homogeneous ODE on interval $I$, then:

$$\boxed{y_1, \ldots, y_n \text{ are LI on } I \iff W(y_1, \ldots, y_n)(x_0) \neq 0 \text{ for some } x_0 \in I}$$

**Equivalently**: They are LD $\iff$ $W = 0$ for **all** $x \in I$.

> **Warning**: This theorem requires that the functions be solutions of the same linear ODE. For arbitrary functions, $W = 0$ does NOT always imply linear dependence.

### Theorem 2: Abel's Formula

For solutions of $y'' + P(x)y' + Q(x)y = 0$:

$$\boxed{W(x) = W(x_0)\exp\left(-\int_{x_0}^{x} P(t)\,dt\right)}$$

**Consequence**: $W$ is either always zero or never zero on $I$.

### Decision Flowchart

```
Given: y₁, y₂, ..., yₙ on interval I
              │
              ▼
    Are they solutions of a
    linear homogeneous ODE?
        ╱           ╲
      YES            NO
       │              │
       ▼              ▼
  Compute W at     Compute W
  any ONE point    everywhere
       │              │
       ▼              ▼
   W ≠ 0?         W ≡ 0 on I?
   ╱    ╲         ╱        ╲
 YES    NO      YES         NO
  │      │       │           │
  ▼      ▼       ▼           ▼
  LI    LD    MIGHT be    Definitely
              LD (need     LI
              more info)
```

---

## 4. Worked Examples

### Example 1: $y_1 = e^x$, $y_2 = e^{2x}$

$$W = \begin{vmatrix} e^x & e^{2x} \\ e^x & 2e^{2x} \end{vmatrix} = 2e^{3x} - e^{3x} = e^{3x} \neq 0$$

**Linearly independent** on any interval.

### Example 2: $y_1 = \sin x$, $y_2 = \cos x$

$$W = \begin{vmatrix} \sin x & \cos x \\ \cos x & -\sin x \end{vmatrix} = -\sin^2 x - \cos^2 x = -1 \neq 0$$

**Linearly independent.** (Wronskian is constant!)

### Example 3: $y_1 = x^2$, $y_2 = 3x^2$

$$W = \begin{vmatrix} x^2 & 3x^2 \\ 2x & 6x \end{vmatrix} = 6x^3 - 6x^3 = 0$$

**Linearly dependent** ($y_2 = 3y_1$).

### Example 4: Three Functions — $y_1 = 1$, $y_2 = x$, $y_3 = x^2$

$$W = \begin{vmatrix} 1 & x & x^2 \\ 0 & 1 & 2x \\ 0 & 0 & 2 \end{vmatrix} = 1 \cdot \begin{vmatrix} 1 & 2x \\ 0 & 2 \end{vmatrix} = 2 \neq 0$$

**Linearly independent.** These form a basis for polynomials of degree ≤ 2.

### Example 5: Verifying Abel's Formula

For $y'' - 3y' + 2y = 0$ (so $P(x) = -3$):

Solutions: $y_1 = e^x$, $y_2 = e^{2x}$

$W = e^{3x}$

Abel's formula: $W = W(0)e^{-\int_0^x (-3)\,dt} = 1 \cdot e^{3x} = e^{3x}$ ✓

### Example 6: Using Abel's to Compute $W$ Without Solutions

For $y'' + \dfrac{1}{x}y' + y = 0$, $P(x) = 1/x$:

$$W(x) = W(x_0)\exp\left(-\int_{x_0}^{x}\frac{1}{t}\,dt\right) = W(x_0)\exp(-\ln(x/x_0)) = W(x_0)\frac{x_0}{x} = \frac{C}{x}$$

Even without knowing the solutions, we know $W$ is proportional to $1/x$.

---

## 5. Common Pitfalls

| Pitfall | Correction |
|---------|-----------|
| "W = 0 at a point ⟹ LD" | Only for solutions; need $W = 0$ everywhere |
| "W ≠ 0 ⟹ LI for any functions" | True for any functions (Wronskian nonzero ⟹ LI always) |
| "W = 0 everywhere ⟹ LD" | Only guaranteed for solutions of a linear ODE |
| Forgetting to check interval | LI/LD is interval-dependent |
| Computing $W$ incorrectly | Check determinant signs carefully |

---

## 6. Wronskian for Common Function Sets

| Functions | Wronskian | LI/LD |
|-----------|-----------|-------|
| $e^{ax}, e^{bx}$ ($a \neq b$) | $(b-a)e^{(a+b)x}$ | LI |
| $\sin x, \cos x$ | $-1$ | LI |
| $e^{ax}\sin bx, e^{ax}\cos bx$ | $-be^{2ax}$ | LI ($b \neq 0$) |
| $x^m, x^n$ ($m \neq n$, $x > 0$) | $(n-m)x^{m+n-1}$ | LI |
| $1, x, x^2, \ldots, x^n$ | $\prod_{k=1}^n k!$ | LI |
| $f(x), cf(x)$ | $0$ | LD |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **LI definition** | $\sum c_k y_k = 0 \implies$ all $c_k = 0$ |
| **Wronskian** | Determinant of functions and their derivatives |
| **LI test (solutions)** | $W \neq 0$ at one point ⟹ LI |
| **LD test (solutions)** | $W = 0$ everywhere ⟹ LD |
| **Abel's formula** | $W(x) = W(x_0)e^{-\int P\,dx}$ |
| **Key property** | For solutions: $W$ is either always 0 or never 0 |

---

## Quick Revision Questions

1. Compute the Wronskian of $e^{3x}$ and $e^{-x}$ and determine if they are LI.

2. Use Abel's formula to find the Wronskian of two solutions of $y'' + 4y = 0$ given that $W(0) = 2$.

3. Show that $\{1, \sin^2 x, \cos^2 x\}$ is linearly dependent (without computing $W$).

4. For $y'' - y = 0$, verify that the Wronskian of $\cosh x$ and $\sinh x$ is constant.

5. Why can't we use $W = 0$ at one point to conclude LD for arbitrary (non-solution) functions?

6. Compute the $3 \times 3$ Wronskian of $\{e^x, e^{2x}, e^{3x}\}$.

---

[← Previous: Geometric Applications](../04-Applications-First-Order/05-geometric-applications.md) | [Back to README](../README.md) | [Next: Fundamental Set of Solutions →](02-fundamental-set-of-solutions.md)
