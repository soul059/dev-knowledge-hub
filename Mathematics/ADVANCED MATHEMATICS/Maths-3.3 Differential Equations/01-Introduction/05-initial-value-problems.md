# Chapter 1.5: Initial Value Problems

[← Previous: General and Particular Solutions](04-general-and-particular-solutions.md) | [Back to README](../README.md) | [Next: Variable Separable →](../02-First-Order-First-Degree/01-variable-separable.md)

---

## Overview

An **Initial Value Problem (IVP)** couples a differential equation with specific conditions at a single point. This chapter formalizes the IVP concept, contrasts it with boundary value problems, and establishes the theoretical foundations through the Existence-Uniqueness theorem.

---

## 1. Definition of an Initial Value Problem

### Formal Statement

> An **Initial Value Problem** consists of:
> 1. A differential equation
> 2. Initial conditions specifying the value of the solution and its derivatives at a **single point** $x_0$

### For an $n$-th Order ODE

$$\frac{d^n y}{dx^n} = f\left(x, y, y', y'', \ldots, y^{(n-1)}\right)$$

with initial conditions:

$$y(x_0) = y_0, \quad y'(x_0) = y_1, \quad y''(x_0) = y_2, \quad \ldots, \quad y^{(n-1)}(x_0) = y_{n-1}$$

### Why $n$ Conditions?

The general solution of an $n$-th order ODE has **$n$ arbitrary constants**. To determine a unique particular solution, we need exactly **$n$ independent conditions**.

```
   Order of DE    Constants to    Conditions
                  determine       needed
       1      →       1       →      1     (y(x₀) = y₀)
       2      →       2       →      2     (y(x₀), y'(x₀))
       3      →       3       →      3     (y(x₀), y'(x₀), y''(x₀))
       n      →       n       →      n
```

---

## 2. First-Order IVP

### Standard Form

$$\frac{dy}{dx} = f(x, y), \quad y(x_0) = y_0$$

### Geometric Meaning

The initial condition selects **one curve** from the infinite family of solution curves.

```
   y
   │
   │    ╱   ╱   ╱
   │   ╱   ╱   ╱    Family of solution curves
   │  ╱   ╱   ╱
   │ ╱   • (x₀, y₀)  ← initial condition picks
   │╱   ╱   ╱          THIS specific curve
   ├───────────── x
   │   ╱   ╱
   │  ╱   ╱
```

### Worked Example 1

**IVP**: $y' = y$, $y(0) = 2$

**Step 1** — Solve the ODE:
$$\frac{dy}{y} = dx \implies \ln|y| = x + C_0 \implies y = Ce^x$$

**Step 2** — Apply initial condition:
$$y(0) = 2 \implies Ce^0 = 2 \implies C = 2$$

**Solution**: $y = 2e^x$

### Worked Example 2

**IVP**: $y' + 2y = 4$, $y(0) = 1$

**Step 1** — This is a linear first-order ODE. Integrating factor: $e^{\int 2 \, dx} = e^{2x}$

**Step 2** — Multiply: $\frac{d}{dx}(ye^{2x}) = 4e^{2x}$

**Step 3** — Integrate: $ye^{2x} = 2e^{2x} + C$

**Step 4** — General solution: $y = 2 + Ce^{-2x}$

**Step 5** — Apply $y(0) = 1$: $1 = 2 + C \implies C = -1$

**Solution**: $y = 2 - e^{-2x}$

**Timing Diagram (Solution Behavior)**:
```
   y
   2 ──────────────────────── asymptote y = 2
   │              ___________
   │          ___/
   │       __/
   │     _/
   1 ── •  (0, 1)
   │   /
   │  /
   │ / y = 2 - e^(-2x)
   │/
   ├──┬──┬──┬──┬──┬──┬──── x
   0  1  2  3  4  5  6
   
   Solution starts at y=1 and 
   approaches y=2 as x → ∞
```

---

## 3. Second-Order IVP

### Standard Form

$$y'' = f(x, y, y'), \quad y(x_0) = y_0, \quad y'(x_0) = y_1$$

Two conditions are needed: the function value AND the slope at $x_0$.

### Worked Example 3

**IVP**: $y'' + 4y = 0$, $y(0) = 0$, $y'(0) = 6$

**Step 1** — Characteristic equation: $r^2 + 4 = 0 \implies r = \pm 2i$

**Step 2** — General solution: $y = C_1 \cos 2x + C_2 \sin 2x$

**Step 3** — Apply $y(0) = 0$:
$$0 = C_1 \cos 0 + C_2 \sin 0 = C_1 \implies C_1 = 0$$

**Step 4** — Compute $y' = -2C_1 \sin 2x + 2C_2 \cos 2x$

Apply $y'(0) = 6$:
$$6 = 2C_2 \cos 0 = 2C_2 \implies C_2 = 3$$

**Solution**: $y = 3\sin 2x$

```
   y
   3 │    ╱╲
     │   ╱  ╲              y = 3 sin(2x)
   0 ├──╱────╲────╱────── x
     │        ╲  ╱
  -3 │         ╲╱
     0   π/4  π/2  3π/4   π
   
   Starts at (0,0) with slope 6
   Oscillates with amplitude 3, period π
```

---

## 4. IVP vs BVP (Boundary Value Problem)

### Comparison

```
  Initial Value Problem (IVP)        Boundary Value Problem (BVP)
  
  All conditions at ONE point        Conditions at TWO (or more) points
  
   y                                  y
   │     ___                          │     ___
   │    /   \                         │    /   \
   │   /     \                        │   /     \
   • ─/───────\──── x                 • ─/───────•──── x
  (x₀,y₀)                           (a,yₐ)      (b,y_b)
  y(x₀)=y₀, y'(x₀)=y₁             y(a)=yₐ, y(b)=y_b
```

| Feature | IVP | BVP |
|---------|-----|-----|
| Conditions given at | One point | Two or more points |
| Typical conditions | $y(x_0), y'(x_0), \ldots$ | $y(a), y(b)$ |
| Existence guaranteed? | Yes (under Picard's theorem) | Not always |
| Uniqueness guaranteed? | Yes (under Picard's theorem) | Not always |
| Commonly arises in | Time-dependent problems | Spatial/equilibrium problems |
| Example | Spring released at $t=0$ | Beam supported at both ends |

### Example: Same DE, Different Problem Types

**DE**: $y'' + y = 0$

**As IVP**: $y(0) = 1$, $y'(0) = 0$ → Unique solution: $y = \cos x$

**As BVP**: $y(0) = 0$, $y(\pi) = 0$ → Solutions: $y = C \sin x$ (infinitely many!)

> BVPs can have **no solution**, **one solution**, or **infinitely many solutions**.

---

## 5. Picard's Existence and Uniqueness Theorem (Detailed)

### Theorem Statement

For the first-order IVP: $\dfrac{dy}{dx} = f(x, y)$, $y(x_0) = y_0$

**If** both $f(x, y)$ and $\dfrac{\partial f}{\partial y}(x, y)$ are continuous in some rectangle

$$R: |x - x_0| \leq a, \quad |y - y_0| \leq b$$

**Then** there exists a **unique** solution $y = \phi(x)$ defined on some interval $|x - x_0| \leq h$ where $h > 0$.

### Applying the Theorem: Step-by-Step

```
Given IVP: y' = f(x,y), y(x₀) = y₀
            │
            ▼
Step 1: Check if f(x,y) is continuous near (x₀, y₀)
            │
     ┌──────┴──────┐
     │ Continuous?  │
     ├──── YES ─────┤
     │              │
     ▼              ▼
Step 2: Compute     If NO: Theorem
∂f/∂y               does not apply
     │              (solution may still
     ▼              exist but not 
Step 3: Check        guaranteed)
if ∂f/∂y is
continuous
     │
     ├──── YES ─────► Solution EXISTS and is UNIQUE
     │
     └──── NO ──────► Existence may hold, uniqueness may fail
```

### Example: Checking Conditions

**IVP**: $y' = \dfrac{xy}{x^2 - 1}$, $y(0) = 2$

- $f(x,y) = \dfrac{xy}{x^2 - 1}$
- Discontinuous at $x = \pm 1$
- Near $(0, 2)$: both $f$ and $\partial f/\partial y = \dfrac{x}{x^2 - 1}$ are continuous
- **Conclusion**: Unique solution exists in interval $(-1, 1)$

**IVP**: $y' = y^{1/3}$, $y(0) = 0$

- $f(x,y) = y^{1/3}$ is continuous at $(0,0)$
- $\partial f/\partial y = \dfrac{1}{3}y^{-2/3}$ is **NOT continuous** at $y = 0$
- **Conclusion**: Uniqueness NOT guaranteed, and indeed $y = 0$ and $y = \left(\dfrac{2x}{3}\right)^{3/2}$ are both solutions.

---

## 6. Picard's Iteration Method

Beyond proving existence, Picard's theorem provides a **constructive** method to approximate solutions.

### Formula

Starting with $\phi_0(x) = y_0$, iterate:

$$\phi_{n+1}(x) = y_0 + \int_{x_0}^{x} f(t, \phi_n(t)) \, dt$$

### Example: $y' = y$, $y(0) = 1$

$$\phi_0 = 1$$

$$\phi_1 = 1 + \int_0^x 1 \, dt = 1 + x$$

$$\phi_2 = 1 + \int_0^x (1 + t) \, dt = 1 + x + \frac{x^2}{2}$$

$$\phi_3 = 1 + \int_0^x \left(1 + t + \frac{t^2}{2}\right) dt = 1 + x + \frac{x^2}{2} + \frac{x^3}{6}$$

**Pattern**: $\phi_n(x) = \sum_{k=0}^{n} \dfrac{x^k}{k!} \to e^x$ as $n \to \infty$ ✓

---

## 7. Well-Posed Problems

A problem is **well-posed** (in the sense of Hadamard) if:
1. A solution **exists**
2. The solution is **unique**
3. The solution **depends continuously** on the data (small changes in initial conditions → small changes in solution)

```
   Well-Posed                      Ill-Posed
   
   Small change in IC →            Small change in IC →
   small change in solution        LARGE change in solution
   
   y                               y
   │   ___                          │   ___
   │  / ← original                 │  /   \  ← original
   │ / ← perturbed                 │ /     \
   ├/─────────── x                 ├/───────\──── x
   │                               │        / ← perturbed
   │  Curves stay close            │  Curves diverge!
```

> Most IVPs we encounter are well-posed. Sensitivity to initial conditions becomes important in **chaotic systems**.

---

## 8. Real-World Application: Projectile Motion

**Problem**: A ball is thrown upward from the ground at 20 m/s. Find its height $y(t)$.

**Model**: $y'' = -g = -9.8$ m/s² (ignoring air resistance)

**Initial conditions**: $y(0) = 0$ (starts at ground), $y'(0) = 20$ (initial velocity)

**Solution**:
- Integrate once: $y' = -9.8t + C_1$
- Apply $y'(0) = 20$: $C_1 = 20$, so $y' = -9.8t + 20$
- Integrate again: $y = -4.9t^2 + 20t + C_2$
- Apply $y(0) = 0$: $C_2 = 0$

**Result**: $y = -4.9t^2 + 20t$

```
   y (meters)
  20 │      ___
     │    _/   \_
     │   /       \        Maximum at t = 20/9.8 ≈ 2.04 s
  15 │  /         \       Max height ≈ 20.4 m
     │ /           \
  10 │/             \
     │               \
   5 │                \
     │                 \
   0 ├──┬──┬──┬──┬──┬──── t (seconds)
     0  1  2  3  4  5
```

---

## Summary Table

| Concept | Key Point |
|---------|----------|
| **IVP** | DE + all conditions at a single point $x_0$ |
| **BVP** | DE + conditions at multiple points |
| **$n$-th order IVP** | Needs $n$ conditions: $y(x_0), y'(x_0), \ldots, y^{(n-1)}(x_0)$ |
| **Picard's theorem** | Guarantees existence and uniqueness if $f, \partial f/\partial y$ continuous |
| **Picard iteration** | Constructive approximation: $\phi_{n+1} = y_0 + \int f(t, \phi_n) \, dt$ |
| **Well-posed** | Exists, unique, continuously depends on data |
| **BVP uniqueness** | NOT guaranteed — may have 0, 1, or $\infty$ solutions |

---

## Quick Revision Questions

1. How many initial conditions are needed to solve a **fourth-order** IVP? What form do they take?

2. For $y' = x^2 + y^2$, $y(0) = 0$: does Picard's theorem guarantee a unique solution? Verify the conditions.

3. Explain why $y' = |y|^{1/2}$, $y(0) = 0$ has non-unique solutions. Which condition of the existence-uniqueness theorem fails?

4. Perform **two iterations** of Picard's method for $y' = x + y$, $y(0) = 1$.

5. What is the key difference between an IVP and a BVP? Give a physical example of each.

6. Solve the IVP: $y'' - 3y' + 2y = 0$, $y(0) = 1$, $y'(0) = 0$.

---

[← Previous: General and Particular Solutions](04-general-and-particular-solutions.md) | [Back to README](../README.md) | [Next: Variable Separable →](../02-First-Order-First-Degree/01-variable-separable.md)
