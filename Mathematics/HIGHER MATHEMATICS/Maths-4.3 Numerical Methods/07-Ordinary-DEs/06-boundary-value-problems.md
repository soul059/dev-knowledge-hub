# Chapter 7.6: Boundary Value Problems

[← Previous: Milne's Method](05-milnes-method.md) | [Next: Unit 8 — Power Method →](../08-Eigenvalue-Problems/01-power-method.md)

---

## Overview

A **Boundary Value Problem (BVP)** specifies conditions at **two different points** (e.g., $y(a)$ and $y(b)$), unlike an IVP where all conditions are at one point. BVPs require fundamentally different numerical techniques.

---

## 1. Standard BVP Form

$$y'' = f(x, y, y'), \quad y(a) = \alpha, \quad y(b) = \beta$$

### Examples

- Beam deflection: conditions at both supports
- Heat distribution: temperatures at both ends of a rod
- Electrostatics: potential at boundaries

---

## 2. Shooting Method

Convert the BVP into an IVP by **guessing** the missing initial condition.

### Algorithm

1. The BVP: $y'' = f(x,y,y')$, $y(a) = \alpha$, $y(b) = \beta$
2. **Guess** $y'(a) = s$ (the "shot")
3. Solve the IVP: $y'' = f$, $y(a) = \alpha$, $y'(a) = s$ using RK4
4. Check if $y(b) = \beta$
5. If not, adjust $s$ and repeat (use bisection or secant method on $s$)

```
  y
  │              ● Target: y(b) = β
  │           ╱╱╱
  │         ╱╱    Shot 3 (just right!) ●
  │       ╱╱              ╱────────────
  │     ╱╱     Shot 2   ╱    
  │    ╱╱     ╱────────╱───── ● Too high
  │   ╱╱    ╱        ╱
  │  ●    ╱  Shot 1 ╱
  │  ↑  ╱  ╱──────╱────────── ● Too low
  │  α ╱ ╱
  └──┼─────────────────────── x
     a                       b

  Adjust the "angle" (slope s) until you hit the target!
```

---

## 3. Finite Difference Method

Replace derivatives with **finite differences** to get a system of algebraic equations.

### For $y'' = p(x)y' + q(x)y + r(x)$, $y(a) = \alpha$, $y(b) = \beta$:

Discretise: $x_i = a + ih$, $h = (b-a)/n$

Replace:
$$y'' \approx \frac{y_{i-1} - 2y_i + y_{i+1}}{h^2}, \quad y' \approx \frac{y_{i+1} - y_{i-1}}{2h}$$

This gives at each interior point $i = 1, \ldots, n-1$:

$$\frac{y_{i-1} - 2y_i + y_{i+1}}{h^2} = p_i \frac{y_{i+1} - y_{i-1}}{2h} + q_i y_i + r_i$$

Rearranging:

$$\left(1 - \frac{h p_i}{2}\right)y_{i-1} + (-2 + h^2 q_i)y_i + \left(1 + \frac{h p_i}{2}\right)y_{i+1} = h^2 r_i$$

This is a **tridiagonal system** — solved efficiently in $O(n)$!

---

## 4. Worked Example (Finite Difference)

Solve $y'' = y$, $y(0) = 0$, $y(1) = \sinh(1) = 1.1752$, with $h = 0.25$ ($n = 4$).

Here $p = 0, q = 1, r = 0$.

At each interior point: $y_{i-1} - 2y_i + y_{i+1} = h^2 y_i = 0.0625 y_i$

$$y_{i-1} + (-2 - 0.0625)y_i + y_{i+1} = 0$$
$$y_{i-1} - 2.0625 y_i + y_{i+1} = 0$$

**System** ($y_0 = 0, y_4 = 1.1752$):

$i=1$: $0 - 2.0625 y_1 + y_2 = 0$

$i=2$: $y_1 - 2.0625 y_2 + y_3 = 0$

$i=3$: $y_2 - 2.0625 y_3 + 1.1752 = 0$

Solving this tridiagonal system:

From $i=1$: $y_2 = 2.0625 y_1$

From $i=2$: $y_1 - 2.0625(2.0625 y_1) + y_3 = 0$ → $y_3 = 3.2539 y_1$

From $i=3$: $2.0625 y_1 - 2.0625(3.2539 y_1) = -1.1752$

$y_1(2.0625 - 6.7112) = -1.1752$

$y_1 = \frac{-1.1752}{-4.6487} = 0.2528$

$y_2 = 0.5214, \quad y_3 = 0.8228$

**Exact:** $\sinh(0.25) = 0.2526$, $\sinh(0.5) = 0.5211$, $\sinh(0.75) = 0.8223$ ✓

---

## 5. Comparison

| Method | Pros | Cons |
|--------|------|------|
| **Shooting** | Uses existing IVP solvers (RK4) | Sensitive to initial guess; may diverge |
| **Finite Difference** | Direct solve; tridiagonal → efficient | Less flexible for nonlinear problems |
| **Collocation** | High accuracy; adaptive | More complex to implement |

---

## Summary Table

| Property | Shooting | Finite Difference |
|----------|----------|-------------------|
| **Approach** | Convert to IVP, iterate | Direct discretisation |
| **Key idea** | Guess $y'(a)$, adjust | Replace derivatives with differences |
| **System type** | Nonlinear root-finding | Tridiagonal linear system |
| **Accuracy** | $O(h^4)$ with RK4 | $O(h^2)$ with central differences |
| **Complexity** | Multiple IVP solves | Single linear solve |

---

## Quick Revision Questions

1. **What is the difference between an IVP and a BVP?**

2. **Describe the shooting method for solving a second-order BVP.**

3. **Derive the finite difference equations for $y'' + y = 0$, $y(0) = 0$, $y(\pi/2) = 1$ with $h = \pi/6$.**

4. **Why does the finite difference method produce a tridiagonal system?**

5. **What are the advantages of finite differences over the shooting method for linear BVPs?**

6. **How is the shooting method modified for nonlinear BVPs?**

---

[← Previous: Milne's Method](05-milnes-method.md) | [Next: Unit 8 — Power Method →](../08-Eigenvalue-Problems/01-power-method.md)
