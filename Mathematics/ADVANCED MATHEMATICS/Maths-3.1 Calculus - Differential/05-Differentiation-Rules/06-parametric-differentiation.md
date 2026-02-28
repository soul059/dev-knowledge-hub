# Unit 5 — Differentiation Rules: Parametric Differentiation

[← Previous: Logarithmic Differentiation](05-logarithmic-differentiation.md) · [Next: Unit 6 — Derivatives of Polynomials →](../06-Derivatives-Standard/01-polynomial-derivatives.md)

---

## Chapter Overview

When a curve is defined **parametrically** — i.e., both $x$ and $y$ are given as functions of a third variable (the **parameter**, usually $t$ or $\theta$) — we need a special approach to find $\frac{dy}{dx}$.

---

## 5.6.1 Parametric Equations

A curve is given parametrically as:

$$x = f(t), \quad y = g(t)$$

```
  y
  |       * (x(t₂), y(t₂))
  |      /
  |     *  ← parametric curve traced as t increases
  |    /
  |   * (x(t₁), y(t₁))
  |  /
  | *
  +----------------------------→ x
     t increases along the curve
```

**Common parametric curves:**

| Curve | Parametric Form |
|-------|----------------|
| Circle (radius $a$) | $x = a\cos t$, $y = a\sin t$ |
| Ellipse ($a$, $b$) | $x = a\cos t$, $y = b\sin t$ |
| Parabola | $x = at^2$, $y = 2at$ |
| Cycloid | $x = a(t - \sin t)$, $y = a(1 - \cos t)$ |
| Astroid | $x = a\cos^3 t$, $y = a\sin^3 t$ |

---

## 5.6.2 First Derivative — The Chain Rule Connection

By the chain rule:

$$\frac{dy}{dx} = \frac{dy/dt}{dx/dt} \quad \text{(provided } dx/dt \neq 0\text{)}$$

```
                     dy/dt
   dy/dx   =   ─────────────
                     dx/dt

   "Rate of y w.r.t. t"  divided by  "Rate of x w.r.t. t"
```

### Example 1: Circle

$$x = a\cos t, \quad y = a\sin t$$

$$\frac{dx}{dt} = -a\sin t, \quad \frac{dy}{dt} = a\cos t$$

$$\frac{dy}{dx} = \frac{a\cos t}{-a\sin t} = -\cot t$$

### Example 2: Parabola

$$x = at^2, \quad y = 2at$$

$$\frac{dx}{dt} = 2at, \quad \frac{dy}{dt} = 2a$$

$$\frac{dy}{dx} = \frac{2a}{2at} = \frac{1}{t}$$

### Example 3: Cycloid

$$x = a(t - \sin t), \quad y = a(1 - \cos t)$$

$$\frac{dx}{dt} = a(1 - \cos t), \quad \frac{dy}{dt} = a\sin t$$

$$\frac{dy}{dx} = \frac{a\sin t}{a(1 - \cos t)} = \frac{\sin t}{1 - \cos t}$$

Using identities: $\sin t = 2\sin\frac{t}{2}\cos\frac{t}{2}$ and $1 - \cos t = 2\sin^2\frac{t}{2}$:

$$\frac{dy}{dx} = \frac{2\sin\frac{t}{2}\cos\frac{t}{2}}{2\sin^2\frac{t}{2}} = \cot\frac{t}{2}$$

---

## 5.6.3 Second Derivative of Parametric Curves

$$\frac{d^2y}{dx^2} = \frac{d}{dx}\left(\frac{dy}{dx}\right) = \frac{\frac{d}{dt}\left(\frac{dy}{dx}\right)}{\frac{dx}{dt}}$$

> **Warning:** $\frac{d^2y}{dx^2} \neq \frac{d^2y/dt^2}{d^2x/dt^2}$. This is a common error!

### Process

```
    Step 1: Find dy/dx = (dy/dt) / (dx/dt)

    Step 2: Let  u = dy/dx  (express in terms of t)

    Step 3: Find du/dt

    Step 4: d²y/dx² = (du/dt) / (dx/dt)
```

### Example: Second Derivative for the Parabola

Given: $x = at^2$, $y = 2at$.

We found $\frac{dy}{dx} = \frac{1}{t}$.

Let $u = \frac{1}{t} = t^{-1}$:

$$\frac{du}{dt} = -t^{-2} = -\frac{1}{t^2}$$

$$\frac{d^2y}{dx^2} = \frac{du/dt}{dx/dt} = \frac{-1/t^2}{2at} = -\frac{1}{2at^3}$$

### Example: Second Derivative for the Circle

Given: $x = a\cos t$, $y = a\sin t$, so $\frac{dy}{dx} = -\cot t$.

Let $u = -\cot t$:

$$\frac{du}{dt} = \csc^2 t$$

$$\frac{d^2y}{dx^2} = \frac{\csc^2 t}{-a\sin t} = \frac{1}{-a\sin^3 t} = -\frac{\csc^3 t}{a}$$

---

## 5.6.4 Tangent and Normal to Parametric Curves

At the point corresponding to $t = t_0$:

**Gradient of tangent:**

$$m = \frac{dy/dt}{dx/dt}\bigg|_{t = t_0}$$

**Equation of tangent:**

$$y - g(t_0) = m\big[x - f(t_0)\big]$$

**Equation of normal:**

$$y - g(t_0) = -\frac{1}{m}\big[x - f(t_0)\big]$$

### Example: Tangent to ellipse at $t = \frac{\pi}{4}$

$x = 3\cos t$, $y = 2\sin t$

At $t = \pi/4$: $x_0 = \frac{3}{\sqrt{2}}$, $y_0 = \frac{2}{\sqrt{2}} = \sqrt{2}$

$$\frac{dy}{dx} = \frac{2\cos t}{-3\sin t} = -\frac{2}{3}\cot t$$

At $t = \pi/4$: $m = -\frac{2}{3}\cot\frac{\pi}{4} = -\frac{2}{3}$

Tangent: $y - \sqrt{2} = -\frac{2}{3}\left(x - \frac{3}{\sqrt{2}}\right)$

---

## 5.6.5 Special Points on Parametric Curves

| Condition | Meaning |
|-----------|---------|
| $\frac{dy}{dt} = 0$, $\frac{dx}{dt} \neq 0$ | Horizontal tangent |
| $\frac{dx}{dt} = 0$, $\frac{dy}{dt} \neq 0$ | Vertical tangent |
| Both $= 0$ | Indeterminate — use L'Hôpital or further analysis |

```
   y                   y                   y
   |    ───*───         |      *            |     *
   |       ↑            |     /|            |    / \
   |   horizontal       |    / |            |   *   *
   |   tangent          |   /  |            |  singular point
   +──────→ x           +──→ ↑ → x          + (both = 0)
                           vertical
                           tangent
```

---

## 5.6.6 Worked Example: Complete Analysis

**Curve:** $x = t^2 - 1$, $y = t^3 - 3t$

**First derivative:**

$$\frac{dx}{dt} = 2t, \quad \frac{dy}{dt} = 3t^2 - 3$$

$$\frac{dy}{dx} = \frac{3t^2 - 3}{2t} = \frac{3(t^2 - 1)}{2t}$$

**Horizontal tangent:** $3t^2 - 3 = 0 \Rightarrow t = \pm 1$
- At $t = 1$: $(0, -2)$
- At $t = -1$: $(0, 2)$

**Vertical tangent:** $2t = 0 \Rightarrow t = 0$
- At $t = 0$: $(-1, 0)$

**Second derivative:**

Let $u = \frac{3(t^2 - 1)}{2t} = \frac{3}{2}\left(t - \frac{1}{t}\right)$

$$\frac{du}{dt} = \frac{3}{2}\left(1 + \frac{1}{t^2}\right) = \frac{3(t^2 + 1)}{2t^2}$$

$$\frac{d^2y}{dx^2} = \frac{3(t^2+1)/(2t^2)}{2t} = \frac{3(t^2+1)}{4t^3}$$

At $t = 1$: $\frac{d^2y}{dx^2} = \frac{3(2)}{4} = \frac{3}{2} > 0$ → local minimum at $(0, -2)$

At $t = -1$: $\frac{d^2y}{dx^2} = \frac{3(2)}{-4} = -\frac{3}{2} < 0$ → local maximum at $(0, 2)$

---

## Summary Table

| Concept | Formula |
|---------|---------|
| First derivative (parametric) | $\frac{dy}{dx} = \frac{dy/dt}{dx/dt}$ |
| Second derivative (parametric) | $\frac{d^2y}{dx^2} = \frac{(d/dt)(dy/dx)}{dx/dt}$ |
| Horizontal tangent | $dy/dt = 0$, $dx/dt \neq 0$ |
| Vertical tangent | $dx/dt = 0$, $dy/dt \neq 0$ |
| Tangent line | $y - y_0 = m(x - x_0)$ where $m = dy/dx$ |
| Normal line | $y - y_0 = -(1/m)(x - x_0)$ |

---

## Quick Revision Questions

**Q1.** For $x = 3t$, $y = t^2$, find $\frac{dy}{dx}$ and $\frac{d^2y}{dx^2}$.

**Q2.** Find the equation of the tangent to the curve $x = \cos t$, $y = \sin t$ at $t = \frac{\pi}{6}$.

**Q3.** For the astroid $x = a\cos^3 t$, $y = a\sin^3 t$, show that $\frac{dy}{dx} = -\tan t$.

**Q4.** Find the points on $x = 2t + 1$, $y = t^3 - 12t$ where the tangent is horizontal.

**Q5.** Find $\frac{d^2y}{dx^2}$ for the cycloid $x = a(t - \sin t)$, $y = a(1 - \cos t)$.

**Q6.** Why is $\frac{d^2y}{dx^2} \neq \frac{d^2y/dt^2}{d^2x/dt^2}$? Explain briefly.

---

[← Previous: Logarithmic Differentiation](05-logarithmic-differentiation.md) · [Next: Unit 6 — Derivatives of Polynomials →](../06-Derivatives-Standard/01-polynomial-derivatives.md)
