# Chapter 1.1: Definition and Terminology

[← Back to README](../README.md) | [Next: Order and Degree →](02-order-and-degree.md)

---

## Overview

A **differential equation** is an equation that involves one or more derivatives of an unknown function. These equations are the primary mathematical tool for describing how quantities change — from the motion of planets to the flow of current in a circuit.

This chapter establishes the foundational vocabulary you will use throughout the course.

---

## 1. What is a Differential Equation?

### Definition

> A **differential equation (DE)** is an equation containing derivatives of one or more dependent variables with respect to one or more independent variables.

**In symbols**: An equation involving $x$, $y$, $y'$, $y''$, ..., $y^{(n)}$ where $y = f(x)$.

### Simple Examples

| Equation | Type |
|----------|------|
| $\dfrac{dy}{dx} = 2x$ | First derivative only |
| $\dfrac{d^2y}{dx^2} + 3\dfrac{dy}{dx} + 2y = 0$ | Second and first derivative |
| $\dfrac{\partial^2 u}{\partial x^2} + \dfrac{\partial^2 u}{\partial y^2} = 0$ | Partial derivatives |
| $\left(\dfrac{dy}{dx}\right)^2 + y = x$ | Squared first derivative |

---

## 2. Classification of Differential Equations

### 2.1 ODE vs PDE

```
                    Differential Equations
                           │
              ┌────────────┴────────────┐
              │                         │
    Ordinary (ODE)              Partial (PDE)
   One independent           Two or more independent
     variable                    variables
              │                         │
     dy/dx = f(x,y)         ∂²u/∂x² + ∂²u/∂y² = 0
```

| Feature | ODE | PDE |
|---------|-----|-----|
| Independent variables | One ($x$ or $t$) | Two or more ($x, y, t, \ldots$) |
| Derivative notation | $dy/dx$, $d^2y/dx^2$ | $\partial u / \partial x$, $\partial^2 u / \partial t^2$ |
| Example | $y'' + y = 0$ | $u_{xx} + u_{yy} = 0$ (Laplace equation) |
| This course covers | ✓ | ✗ (covered in PDE course) |

> **This course focuses exclusively on ODEs.**

### 2.2 Linear vs Non-Linear

A differential equation is **linear** if:
1. The dependent variable $y$ and all its derivatives appear to the **first power** only
2. There are **no products** of $y$ with its derivatives
3. No transcendental functions of $y$ (like $\sin y$, $e^y$, $\ln y$)

**General form of a linear ODE of order $n$:**

$$a_n(x) \frac{d^n y}{dx^n} + a_{n-1}(x) \frac{d^{n-1} y}{dx^{n-1}} + \cdots + a_1(x) \frac{dy}{dx} + a_0(x) y = g(x)$$

where $a_0, a_1, \ldots, a_n$ and $g$ are functions of $x$ only (not $y$).

| Equation | Linear? | Reason |
|----------|---------|--------|
| $y'' + 3y' + 2y = \sin x$ | ✓ Yes | All terms in $y$ are first power |
| $y'' + y^2 = 0$ | ✗ No | $y^2$ — dependent variable squared |
| $y' \cdot y = x$ | ✗ No | Product of $y$ and $y'$ |
| $y'' + \sin(y) = 0$ | ✗ No | Transcendental function of $y$ |
| $y'' + (\sin x) y = e^x$ | ✓ Yes | $\sin x$ is coefficient, not function of $y$ |

### 2.3 Homogeneous vs Non-Homogeneous (for Linear ODEs)

A linear ODE is **homogeneous** if $g(x) = 0$:

$$a_n(x) y^{(n)} + a_{n-1}(x) y^{(n-1)} + \cdots + a_0(x) y = 0$$

It is **non-homogeneous** if $g(x) \neq 0$:

$$a_n(x) y^{(n)} + a_{n-1}(x) y^{(n-1)} + \cdots + a_0(x) y = g(x)$$

> **Note**: "Homogeneous" has a different meaning for first-order equations (Unit 2). Context matters!

---

## 3. Key Terminology

### 3.1 Dependent and Independent Variables

```
     y = f(x)
     │       │
     │       └── Independent variable (input)
     └────────── Dependent variable (output, unknown)
```

- The **independent variable** is the one we differentiate with respect to (usually $x$ or $t$).
- The **dependent variable** is the unknown function we want to find (usually $y$ or $u$).

### 3.2 Solution of a DE

> A **solution** of a DE is any function $y = \phi(x)$ that, when substituted into the equation, reduces it to an identity on some interval $I$.

**Example**: Is $y = e^{2x}$ a solution of $y' - 2y = 0$?

- Compute $y' = 2e^{2x}$
- Substitute: $2e^{2x} - 2e^{2x} = 0$ ✓
- Yes, $y = e^{2x}$ is a solution.

### 3.3 Explicit vs Implicit Solutions

| Type | Form | Example |
|------|------|---------|
| **Explicit** | $y = \phi(x)$ | $y = Ce^{2x}$ |
| **Implicit** | $F(x, y) = 0$ | $x^2 + y^2 = r^2$ |

Some DEs have solutions that cannot be written in explicit form — the implicit form is perfectly valid.

### 3.4 Interval of Existence

A solution is valid on an **interval** $I = (a, b)$ where:
- The solution exists
- The solution is continuous
- The solution satisfies the DE at every point in $I$

The largest such interval is called the **interval of existence** or **domain of the solution**.

---

## 4. Families of Curves and Differential Equations

A key conceptual link: **a DE represents a family of curves, and each solution is one member**.

```
   y                    Family: y = Cx²  (solutions of xy' = 2y)
   │     /              
   │    / C=2           Each curve corresponds to
   │   /                a different value of C
   │  /  C=1
   │ /                  
   │/ C=0.5             The DE xy' = 2y describes
   ├──────── x          ALL these curves at once
   │\ C=-0.5
   │ \
   │  \ C=-1
```

- **General solution**: $y = Cx^2$ (one arbitrary constant → one-parameter family)
- **Particular solution**: $y = 3x^2$ (specific value $C = 3$)

---

## 5. Origins of Differential Equations

DEs arise naturally in modeling:

| Source | Physical Law | Resulting ODE |
|--------|-------------|---------------|
| Population growth | Rate ∝ population | $\dfrac{dP}{dt} = kP$ |
| Radioactive decay | Rate ∝ amount present | $\dfrac{dN}{dt} = -\lambda N$ |
| Newton's 2nd law | $F = ma$ | $m\dfrac{d^2x}{dt^2} = F(t, x, x')$ |
| Circuit (RL) | Kirchhoff's voltage law | $L\dfrac{di}{dt} + Ri = V(t)$ |
| Cooling | Rate ∝ temperature difference | $\dfrac{dT}{dt} = -k(T - T_s)$ |

### RL Circuit Example (ASCII Diagram)

```
        R (Resistor)
   ┌───/\/\/\/───┐
   │             │
 + │             │
 V(t)          L (Inductor)
 - │           ═══
   │             │
   └─────────────┘

   KVL: V(t) = Ri + L(di/dt)
   
   ODE: L(di/dt) + Ri = V(t)    ← First order linear ODE
```

---

## 6. Direction Fields (Slope Fields)

For a first-order ODE $y' = f(x, y)$, the **direction field** shows the slope at each point $(x, y)$.

```
   y
   3 │  /  /  /  |  \  \  \
     │                        At each point, draw a
   2 │  /  /  /  |  \  \  \   short line segment with
     │                        slope = f(x,y)
   1 │  /  /  -- -- -- \  \
     │                        Solution curves follow
   0 ├──────────────────── x  these slope marks
     │  \  \  -- -- -- /  /
  -1 │                       
     │  \  \  \  |  /  /  /
  -2 │                       
    -3 -2 -1  0  1  2  3

   Example: y' = -x/y  (circular solutions)
```

Direction fields give a **qualitative picture** of solutions without solving the equation.

---

## Summary Table

| Concept | Definition |
|---------|-----------|
| **Differential Equation** | Equation involving derivatives of an unknown function |
| **ODE** | DE with one independent variable |
| **PDE** | DE with multiple independent variables |
| **Linear DE** | $y$ and derivatives appear to first power, no products |
| **Non-linear DE** | $y^2$, $yy'$, $\sin y$, etc. appear |
| **Homogeneous** | Right-hand side $g(x) = 0$ (for linear DEs) |
| **Solution** | Function satisfying the DE on an interval |
| **General solution** | Contains arbitrary constants |
| **Particular solution** | Specific constant values |
| **Direction field** | Visual representation of slopes $y' = f(x,y)$ |

---

## Quick Revision Questions

1. **Classify** the equation $y'' + xy' - y^3 = 0$. Is it ODE or PDE? Linear or non-linear? Why?

2. **Verify** that $y = \sin(2x)$ is a solution of $y'' + 4y = 0$.

3. What is the **difference** between a general solution and a particular solution? How are they related?

4. Is $x^2 y'' + 3xy' + y = \ln x$ a linear ODE? Justify your answer.

5. Given a one-parameter family of curves $y = Ce^{-x}$, what differential equation does this family satisfy?

6. Explain why $y' + y \cdot y'' = x$ is a **non-linear** ODE.

---

[← Back to README](../README.md) | [Next: Order and Degree →](02-order-and-degree.md)
