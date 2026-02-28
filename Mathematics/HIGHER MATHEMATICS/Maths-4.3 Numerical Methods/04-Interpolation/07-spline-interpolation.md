# Chapter 4.7: Spline Interpolation

[← Previous: Inverse Interpolation](06-inverse-interpolation.md) | [Next: Unit 5 — Forward Difference Differentiation →](../05-Numerical-Differentiation/01-forward-difference.md)

---

## Overview

**Spline interpolation** uses piecewise low-degree polynomials (typically cubics) joined smoothly at data points, avoiding the oscillation problems (Runge phenomenon) of high-degree polynomial interpolation.

---

## 1. Motivation: Why Splines?

```
  High-degree polynomial (Runge effect):
    │  ╱╲                          ╱╲
    │ ╱  ╲   Oscillates wildly!   ╱  ╲
    ●╱    ╲────────●──────────────╱    ╲●
    │      ╲      ╱ ╲           ╱
    │       ╲────╱   ╲─────────╱
    │

  Cubic spline (smooth, stable):
    │
    ●─────────●──────────────────●
    │ smooth curve,              │
    │ no oscillation             │
```

---

## 2. Linear Spline

The simplest spline: connect data points with straight line segments.

$$S_i(x) = f_i + \frac{f_{i+1} - f_i}{x_{i+1} - x_i}(x - x_i), \quad x \in [x_i, x_{i+1}]$$

- Continuous ($C^0$) but NOT smooth (corners at data points).
- Useful but not preferred for scientific work.

---

## 3. Cubic Spline — Definition

Given $n+1$ points $(x_0, f_0), \ldots, (x_n, f_n)$, a **cubic spline** $S(x)$ satisfies:

1. $S(x)$ is a cubic polynomial $S_i(x)$ on each subinterval $[x_i, x_{i+1}]$
2. $S(x_i) = f_i$ for all $i$ (interpolation condition)
3. $S_i(x_{i+1}) = S_{i+1}(x_{i+1})$ (continuity)
4. $S_i'(x_{i+1}) = S_{i+1}'(x_{i+1})$ (smooth first derivative)
5. $S_i''(x_{i+1}) = S_{i+1}''(x_{i+1})$ (smooth second derivative)

---

## 4. Derivation

Let $h_i = x_{i+1} - x_i$ and define $M_i = S''(x_i)$ (second derivatives at knots).

On $[x_i, x_{i+1}]$, since $S_i''(x)$ is linear:

$$S_i''(x) = M_i \frac{x_{i+1} - x}{h_i} + M_{i+1} \frac{x - x_i}{h_i}$$

Integrating twice and applying $S_i(x_i) = f_i$, $S_i(x_{i+1}) = f_{i+1}$:

$$\boxed{S_i(x) = \frac{M_i}{6h_i}(x_{i+1}-x)^3 + \frac{M_{i+1}}{6h_i}(x-x_i)^3 + \left(\frac{f_{i+1}}{h_i} - \frac{M_{i+1}h_i}{6}\right)(x - x_i) + \left(\frac{f_i}{h_i} - \frac{M_i h_i}{6}\right)(x_{i+1} - x)}$$

---

## 5. The Tridiagonal System

Applying the smoothness condition $S_{i-1}'(x_i) = S_i'(x_i)$:

$$\boxed{h_{i-1} M_{i-1} + 2(h_{i-1} + h_i) M_i + h_i M_{i+1} = 6\left(\frac{f_{i+1} - f_i}{h_i} - \frac{f_i - f_{i-1}}{h_{i-1}}\right)}$$

for $i = 1, 2, \ldots, n-1$. This gives $n-1$ equations for $n+1$ unknowns $M_0, M_1, \ldots, M_n$.

### Boundary Conditions (2 more equations needed)

| Type | Condition | Description |
|------|-----------|-------------|
| **Natural** | $M_0 = 0, \; M_n = 0$ | Zero curvature at endpoints |
| **Clamped** | $S'(x_0) = f'_0, \; S'(x_n) = f'_n$ | Specified slopes |
| **Not-a-knot** | $S_0'''(x_1) = S_1'''(x_1)$ (both ends) | Cubic extends across first/last knot |

---

## 6. Tridiagonal Matrix Form (Natural Spline)

For equal spacing $h$:

$$\begin{bmatrix} 1 & 0 & & \\ 1 & 4 & 1 & & \\ & 1 & 4 & 1 & \\ & & \ddots & \ddots & \ddots \\ & & & 1 & 4 & 1 \\ & & & & 0 & 1 \end{bmatrix} \begin{bmatrix} M_0 \\ M_1 \\ M_2 \\ \vdots \\ M_{n-1} \\ M_n \end{bmatrix} = \begin{bmatrix} 0 \\ d_1 \\ d_2 \\ \vdots \\ d_{n-1} \\ 0 \end{bmatrix}$$

where $d_i = \frac{6}{h^2}(f_{i-1} - 2f_i + f_{i+1})$.

This **tridiagonal system** can be solved in $O(n)$ time using the Thomas algorithm!

---

## 7. Worked Example

Given:

| $x$ | 0 | 1 | 2 | 3 |
|-----|---|---|---|---|
| $f(x)$ | 1 | 2 | 33 | 244 |

Find the natural cubic spline.

$h = 1$ (equal spacing), $n = 3$.

**Step 1**: Compute $d_i$:

$$d_1 = \frac{6}{1}(f_0 - 2f_1 + f_2) = 6(1 - 4 + 33) = 180$$

$$d_2 = \frac{6}{1}(f_1 - 2f_2 + f_3) = 6(2 - 66 + 244) = 1080$$

**Step 2**: Solve the tridiagonal system (with $M_0 = 0, M_3 = 0$):

$$4M_1 + M_2 = 180$$
$$M_1 + 4M_2 = 1080$$

From equation 1: $M_1 = (180 - M_2)/4 = 45 - M_2/4$

Substituting: $45 - M_2/4 + 4M_2 = 1080 \Rightarrow 15M_2/4 = 1035 \Rightarrow M_2 = 276$

$M_1 = 45 - 69 = -24$

**Step 3**: Write spline pieces using the formula:

| Interval | $M_i$ | $M_{i+1}$ |
|----------|--------|-----------|
| $[0,1]$ | $M_0=0$ | $M_1=-24$ |
| $[1,2]$ | $M_1=-24$ | $M_2=276$ |
| $[2,3]$ | $M_2=276$ | $M_3=0$ |

For $[0,1]$:
$$S_0(x) = \frac{0}{6}(1-x)^3 + \frac{-24}{6}(x)^3 + (2 + 4)(x) + (1-0)(1-x)$$
$$= -4x^3 + 6x + 1 - x = -4x^3 + 5x + 1$$

---

## 8. Comparison

```
  ┌──────────────────┬──────────────┬──────────────────┐
  │   Method         │  Continuity  │  Oscillation     │
  ├──────────────────┼──────────────┼──────────────────┤
  │ Linear spline    │  C⁰          │  None            │
  │ Quadratic spline │  C¹          │  Minimal         │
  │ Cubic spline     │  C²          │  None (typical)  │
  │ High-degree poly │  C∞          │  Severe (Runge)  │
  └──────────────────┴──────────────┴──────────────────┘
```

---

## Summary Table

| Property | Value |
|----------|-------|
| **Piece type** | Cubic polynomial per subinterval |
| **Continuity** | $C^2$ (function, 1st and 2nd derivatives continuous) |
| **System** | Tridiagonal — $O(n)$ solve |
| **Boundary types** | Natural ($M=0$), Clamped ($S'$ given), Not-a-knot |
| **Advantage** | No oscillation, smooth, efficient |
| **Error** | $O(h^4)$ for clamped spline |

---

## Quick Revision Questions

1. **What is the Runge phenomenon and how do cubic splines avoid it?**

2. **State the continuity conditions a cubic spline must satisfy at each interior knot.**

3. **Derive the tridiagonal system for the natural cubic spline with equal spacing.**

4. **What are the three common types of boundary conditions for cubic splines?**

5. **Given $(0,0), (1,1), (2,0)$, find the natural cubic spline.**

6. **Why is the tridiagonal system efficient to solve? What is its complexity?**

---

[← Previous: Inverse Interpolation](06-inverse-interpolation.md) | [Next: Unit 5 — Forward Difference Differentiation →](../05-Numerical-Differentiation/01-forward-difference.md)
