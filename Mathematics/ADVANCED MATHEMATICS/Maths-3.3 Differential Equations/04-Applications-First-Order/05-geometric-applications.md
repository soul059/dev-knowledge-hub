# Chapter 4.5: Geometric Applications

[← Previous: Orthogonal Trajectories](04-orthogonal-trajectories.md) | [Back to README](../README.md) | [Next: Wronskian and Linear Independence →](../05-Higher-Order-Linear-ODEs/01-wronskian-and-linear-independence.md)

---

## Overview

Differential equations arise naturally from geometric properties of curves: tangent lines, normal lines, arc length, radius of curvature, and envelopes. This chapter explores finding curves given geometric conditions.

---

## 1. Tangent and Normal Lines — Quick Reference

At a point $(x, y)$ on a curve $y = f(x)$:

| Property | Formula |
|----------|---------|
| Slope of tangent | $m = dy/dx$ |
| Slope of normal | $-1/m = -dx/dy$ |
| Equation of tangent | $Y - y = m(X - x)$ |
| Equation of normal | $Y - y = -\dfrac{1}{m}(X - x)$ |
| $x$-intercept of tangent | $x - y/m = x - y\dfrac{dx}{dy}$ |
| $y$-intercept of tangent | $y - mx = y - x\dfrac{dy}{dx}$ |
| $x$-intercept of normal | $x + ym = x + y\dfrac{dy}{dx}$ |
| $y$-intercept of normal | $y + x/m = y + x\dfrac{dx}{dy}$ |
| Length of tangent | $\dfrac{y}{m}\sqrt{1 + m^2} = \dfrac{y\sqrt{1 + (dy/dx)^2}}{dy/dx}$ |
| Length of normal | $y\sqrt{1 + m^2} = y\sqrt{1 + (dy/dx)^2}$ |
| Subtangent | $y/m = y\dfrac{dx}{dy}$ |
| Subnormal | $ym = y\dfrac{dy}{dx}$ |

```
     y
     │             Tangent line
     │           ╱
     │     P(x,y)
     │      •╱
     │     ╱│╲
     │    ╱ │  ╲  Normal line
     │   ╱  │   ╲
     │  ╱   │    ╲
  ───┼─╱T───┼─N───╲──── x
     │      │
     │  ←─→ ←──→
     │  Sub-  Sub-
     │  tangent normal
```

**Key**: 
- **T** = x-intercept of tangent
- **N** = x-intercept of normal
- **Subtangent** = horizontal projection of tangent segment from $P$ to $T$
- **Subnormal** = horizontal projection of normal segment from $P$ to $N$

---

## 2. Problems on Subtangent

### Type 1: Subtangent is Constant

*Find the curve whose subtangent is constant $= a$.*

Subtangent $= y\dfrac{dx}{dy} = a$

$$\frac{dy}{y} = \frac{dx}{a} \implies \ln y = \frac{x}{a} + C$$

$$\boxed{y = Ce^{x/a}} \quad (\text{exponential curves})$$

### Type 2: Subtangent = Twice the Abscissa

$y\dfrac{dx}{dy} = 2x$

$$\frac{dx}{x} = \frac{2\,dy}{y} \implies \ln x = 2\ln y + C \implies x = ky^2$$

$$\boxed{x = ky^2} \quad (\text{parabolas with horizontal axis})$$

---

## 3. Problems on Subnormal

### Type 1: Subnormal is Constant

*Find the curve whose subnormal is constant $= a$.*

Subnormal $= y\dfrac{dy}{dx} = a$

$$y\,dy = a\,dx \implies \frac{y^2}{2} = ax + C$$

$$\boxed{y^2 = 2ax + C} \quad (\text{parabolas})$$

> **Important Result**: The only curves with constant subnormal are **parabolas**.

### Type 2: Subnormal = Abscissa

$y\dfrac{dy}{dx} = x$

$$y\,dy = x\,dx \implies y^2 = x^2 + C$$

$$\boxed{x^2 - y^2 = k} \quad (\text{rectangular hyperbolas})$$

---

## 4. Problems on Tangent Intercepts

### Example 1: $y$-intercept of Tangent = Twice the Ordinate

$y$-intercept of tangent $= y - x\dfrac{dy}{dx} = 2y$

$$-x\frac{dy}{dx} = y \implies \frac{dy}{y} = -\frac{dx}{x}$$

$$\ln y = -\ln x + C \implies xy = k$$

$$\boxed{xy = k} \quad (\text{rectangular hyperbolas})$$

### Example 2: $x$-intercept of Tangent = Twice the Abscissa

$x$-intercept $= x - y\dfrac{dx}{dy} = 2x$

$$-y\frac{dx}{dy} = x \implies \frac{dx}{x} = -\frac{dy}{y}$$

$$\boxed{xy = k}$$

### Example 3: Tangent at Any Point Passes Through a Fixed Point

*The tangent at $(x, y)$ passes through the origin.*

Tangent: $Y - y = m(X - x)$. At $(0, 0)$:

$$-y = m(-x) \implies m = y/x \implies \frac{dy}{dx} = \frac{y}{x}$$

$$\boxed{y = cx} \quad (\text{lines through origin})$$

---

## 5. Problems on Length of Tangent and Normal

### Length of Tangent = Constant $a$

$$\frac{y}{m}\sqrt{1 + m^2} = a \implies y^2(1 + m^2) = a^2m^2$$

Let $p = dy/dx$:

$$y^2(1 + p^2) = a^2 p^2 \implies y^2 = p^2(a^2 - y^2) \implies p = \frac{y}{\sqrt{a^2 - y^2}}$$

$$\frac{\sqrt{a^2 - y^2}}{y}\,dy = dx$$

Substitute $y = a\sin\theta$:

$$x = a\ln\left(\frac{a + \sqrt{a^2 - y^2}}{y}\right) - \sqrt{a^2 - y^2} + C$$

This is the **tractrix** (or pursuit curve).

### Length of Normal = Constant $a$

$$y\sqrt{1 + (dy/dx)^2} = a$$

$$1 + \left(\frac{dy}{dx}\right)^2 = \frac{a^2}{y^2} \implies \frac{dy}{dx} = \pm\frac{\sqrt{a^2 - y^2}}{y}$$

$$\frac{y\,dy}{\sqrt{a^2 - y^2}} = \pm dx \implies -\sqrt{a^2 - y^2} = \pm x + C$$

$$(x - h)^2 + y^2 = a^2$$

$$\boxed{\text{Circles of radius } a}$$

> **Remarkable Result**: Circles are the only curves with constant normal length.

---

## 6. Curves with Given Arc Length Properties

### Arc Length Formula

$$ds = \sqrt{dx^2 + dy^2} = \sqrt{1 + \left(\frac{dy}{dx}\right)^2}\,dx$$

### Example: Arc Length = Twice the Abscissa

$$s = 2x \implies \frac{ds}{dx} = 2 \implies \sqrt{1 + (y')^2} = 2$$

$$(y')^2 = 3 \implies y' = \pm\sqrt{3}$$

$$\boxed{y = \pm\sqrt{3}\,x + C} \quad (\text{straight lines at 60° to x-axis})$$

---

## 7. Reflection Problems (Mirrors)

*Find the curve such that light rays from the origin, after reflection, travel parallel to the $x$-axis.*

```
        Reflected ray (horizontal)
     ←────────────────── •P(x,y)
                        ╱│
                       ╱ │
          Incident    ╱  │ Normal
          ray from   ╱   │
          origin    ╱    │
                   ╱     │
                  O(0,0)
```

By the law of reflection (angle of incidence = angle of reflection):

$$\frac{dy}{dx} = \frac{y}{x + \sqrt{x^2 + y^2}}$$

Substituting $y = vx$, this becomes solvable and yields:

$$\boxed{y^2 = 4a(x + a)} \quad (\text{parabolas with focus at origin})$$

This is why **parabolic mirrors** focus parallel light to a point!

---

## 8. Curve Through Given Points with Slope Condition

### Example: Slope = Sum of Coordinates, Passing Through (0, 1)

$$\frac{dy}{dx} = x + y, \quad y(0) = 1$$

This is linear: $\dfrac{dy}{dx} - y = x$

I.F.: $e^{-x}$

$$ye^{-x} = \int xe^{-x}\,dx = -xe^{-x} - e^{-x} + C$$

$$y = -x - 1 + Ce^x$$

$y(0) = 1$: $1 = -1 + C \implies C = 2$

$$\boxed{y = 2e^x - x - 1}$$

---

## Summary Table

| Geometric Property | ODE | Solution |
|-------------------|-----|----------|
| Subtangent $= a$ | $y\,dx/dy = a$ | $y = Ce^{x/a}$ (exponentials) |
| Subnormal $= a$ | $y\,dy/dx = a$ | $y^2 = 2ax + C$ (parabolas) |
| Tangent passes through origin | $dy/dx = y/x$ | $y = cx$ (lines) |
| Normal length $= a$ | $y\sqrt{1+y'^2} = a$ | $(x-h)^2 + y^2 = a^2$ (circles) |
| $y$-int of tangent $= 2y$ | $y - xy' = 2y$ | $xy = k$ (hyperbolas) |
| Arc length $= 2x$ | $\sqrt{1+y'^2} = 2$ | $y = \pm\sqrt{3}x + C$ (lines) |
| Tangent length $= a$ | Complex ODE | Tractrix |
| Reflective curve (parabolic) | $y' = y/(x+\sqrt{x^2+y^2})$ | $y^2 = 4a(x+a)$ |

---

## Quick Revision Questions

1. Find the curve whose subnormal equals three times its abscissa.

2. The subtangent at any point of a curve equals twice the abscissa. Find the curve.

3. Find the curve whose normal at every point passes through the origin.

4. Show that the curve with constant normal length is a circle.

5. A curve passes through $(1, 2)$ and has the property that the $y$-intercept of the tangent equals $x \cdot dy/dx$. Find the curve.

6. Find the curve whose tangent at any point $(x, y)$ makes intercepts on the axes whose sum is $2a$.

---

[← Previous: Orthogonal Trajectories](04-orthogonal-trajectories.md) | [Back to README](../README.md) | [Next: Wronskian and Linear Independence →](../05-Higher-Order-Linear-ODEs/01-wronskian-and-linear-independence.md)
