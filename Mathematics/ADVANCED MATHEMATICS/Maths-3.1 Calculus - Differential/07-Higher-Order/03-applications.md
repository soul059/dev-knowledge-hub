# Unit 7 — Higher-Order Derivatives: Applications

[← Previous: Leibniz Theorem](02-leibniz-theorem.md) · [Next: Unit 8 — Rate of Change →](../08-Applications/01-rate-of-change.md)

---

## Chapter Overview

Higher-order derivatives have deep geometric and physical meaning. This chapter covers curvature, radius of curvature, and the connection between second derivatives and the shape of curves (convexity/concavity), preparing the ground for the full applications unit.

---

## 7.3.1 Physical Meaning of Higher Derivatives

| Derivative | Physical Meaning (motion) | Geometric Meaning |
|------------|---------------------------|-------------------|
| $f(t)$ | Position | Height of curve |
| $f'(t)$ | Velocity | Slope of curve |
| $f''(t)$ | Acceleration | Curvature / concavity |
| $f'''(t)$ | Jerk (rate of change of accel.) | Rate of change of curvature |

```
  Position s(t)          Velocity v(t)         Acceleration a(t)
       *                      *                       *───*
      / \                   *   *                    /     \
     /   \                 /     \                  *       *
    /     \               /       \
   *       *             *         *
   
   d/dt →                d/dt →                  d/dt →
```

---

## 7.3.2 Curvature

The **curvature** $\kappa$ measures how rapidly a curve bends.

$$\kappa = \frac{|y''|}{(1 + (y')^2)^{3/2}}$$

- $\kappa = 0$: straight line (no bending)
- Large $\kappa$: sharp bend
- $\kappa$ is always non-negative

### Radius of Curvature

$$R = \frac{1}{\kappa} = \frac{(1 + (y')^2)^{3/2}}{|y''|}$$

The **radius of curvature** is the radius of the **osculating circle** — the circle that best approximates the curve at that point.

```
    y
    |         Osculating circle
    |        .  .  .
    |      .        .
    |     .    R ←───• Center
    |      .        .
    |   ────*────── curve
    |        .  .
    +───────────────→ x
```

### Example: Curvature of $y = x^2$ at origin

$y' = 2x$, $y'' = 2$

At $x = 0$: $y' = 0$, $y'' = 2$

$$\kappa = \frac{|2|}{(1+0)^{3/2}} = 2$$

$$R = \frac{1}{2}$$

### Example: Curvature of $y = \sin x$ at $x = 0$

$y' = \cos x$, $y'' = -\sin x$

At $x = 0$: $y' = 1$, $y'' = 0$

$$\kappa = \frac{0}{(1+1)^{3/2}} = 0$$

The curve has zero curvature at the inflection point $x = 0$.

---

## 7.3.3 Curvature in Parametric Form

For $x = f(t)$, $y = g(t)$:

$$\kappa = \frac{|\dot{x}\ddot{y} - \ddot{x}\dot{y}|}{(\dot{x}^2 + \dot{y}^2)^{3/2}}$$

where dots denote derivatives with respect to $t$.

### Example: Circle $x = a\cos t$, $y = a\sin t$

$$\dot{x} = -a\sin t, \quad \ddot{x} = -a\cos t$$
$$\dot{y} = a\cos t, \quad \ddot{y} = -a\sin t$$

$$\kappa = \frac{|(-a\sin t)(-a\sin t) - (-a\cos t)(a\cos t)|}{(a^2\sin^2 t + a^2\cos^2 t)^{3/2}}$$

$$= \frac{|a^2\sin^2 t + a^2\cos^2 t|}{(a^2)^{3/2}} = \frac{a^2}{a^3} = \frac{1}{a}$$

So $R = a$ — the radius of curvature of a circle equals its radius! ✓

---

## 7.3.4 Centre of Curvature

The centre of curvature $(\bar{x}, \bar{y})$ at a point $(x_0, y_0)$ is:

$$\bar{x} = x_0 - \frac{y'(1 + (y')^2)}{y''}, \quad \bar{y} = y_0 + \frac{1 + (y')^2}{y''}$$

The locus of all centres of curvature as the point moves along the curve is called the **evolute** of the curve.

---

## 7.3.5 Concavity and Points of Inflection (Preview)

| Condition | Shape | Name |
|-----------|-------|------|
| $f''(x) > 0$ | Concave up (∪) | "Holds water" |
| $f''(x) < 0$ | Concave down (∩) | "Spills water" |
| $f''(x) = 0$ (changes sign) | Point of inflection | Curvature = 0 |

```
  Concave UP (f'' > 0)         Concave DOWN (f'' < 0)
  
       *   *                    *───────*
      /     \                  /         \
     /       \                *           *
    *         \
                              Point of inflection:
                              where concavity changes
                                    *
                                   / \
                              ────*   *────
```

This topic is covered fully in [Unit 8: Concavity & Inflection](../08-Applications/05-concavity-inflection.md).

---

## 7.3.6 Envelope of a Family of Curves

If a family of curves is given by $F(x, y, c) = 0$ (where $c$ is a parameter), the **envelope** satisfies:

$$F(x, y, c) = 0 \quad \text{and} \quad \frac{\partial F}{\partial c} = 0$$

Higher-order derivatives help analyze these envelopes, which are curves tangent to every member of the family.

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Curvature | $\kappa = \frac{\|y''\|}{(1+(y')^2)^{3/2}}$ |
| Radius of curvature | $R = 1/\kappa$ |
| Parametric curvature | $\kappa = \frac{\|\dot{x}\ddot{y} - \ddot{x}\dot{y}\|}{(\dot{x}^2+\dot{y}^2)^{3/2}}$ |
| Circle: $R = a$ | Curvature is constant $= 1/a$ |
| Straight line | Curvature $= 0$ everywhere |
| Inflection point | $y'' = 0$ and sign changes |
| Physical meaning | $f'$ = velocity, $f''$ = acceleration, $f'''$ = jerk |

---

## Quick Revision Questions

**Q1.** Find the curvature of $y = e^x$ at $x = 0$.

**Q2.** Show that the radius of curvature of the parabola $y = x^2$ at the vertex is $\frac{1}{2}$.

**Q3.** Find the curvature of the ellipse $x = 3\cos t$, $y = 2\sin t$ at $t = 0$.

**Q4.** At what points does $y = x^3$ have zero curvature?

**Q5.** Find the radius of curvature of $y = \ln(\sec x)$ at $x = 0$.

**Q6.** Why is the curvature of a straight line zero? Explain in terms of $y''$.

---

[← Previous: Leibniz Theorem](02-leibniz-theorem.md) · [Next: Unit 8 — Rate of Change →](../08-Applications/01-rate-of-change.md)
