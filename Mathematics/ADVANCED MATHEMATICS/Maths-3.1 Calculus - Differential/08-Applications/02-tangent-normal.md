# Unit 8 — Applications of Derivatives: Tangent & Normal Lines

[← Previous: Rate of Change](01-rate-of-change.md) · [Next: Increasing & Decreasing Functions →](03-increasing-decreasing.md)

---

## Chapter Overview

The derivative gives the **slope of the tangent line** at any point on a curve. This chapter formalizes tangent and normal line equations, explores special cases, and covers the angle of intersection between curves.

---

## 8.2.1 Equation of the Tangent Line

At the point $(x_0, y_0)$ on $y = f(x)$, the tangent line has slope $m = f'(x_0)$:

$$\boxed{y - y_0 = f'(x_0)(x - x_0)}$$

```
         y
         |          *  ← curve y = f(x)
         |         *
         |        * ← tangent line
         |       *•(x₀, y₀)
         |      *
         |     *        slope = f'(x₀)
         +──────────────→ x
```

---

## 8.2.2 Equation of the Normal Line

The **normal** is perpendicular to the tangent at the point of tangency:

$$\boxed{y - y_0 = -\frac{1}{f'(x_0)}(x - x_0)} \quad \text{(provided } f'(x_0) \neq 0\text{)}$$

If $f'(x_0) = 0$ (horizontal tangent), the normal is the vertical line $x = x_0$.

```
         y
         |     Normal
         |       |
         |       | (perpendicular
         |───────•────── Tangent    to the tangent)
         |       |
         |       |
         +──────────────→ x
```

---

## 8.2.3 Worked Examples

### Example 1: $y = x^3 - 3x + 2$ at $x = 1$

$y(1) = 1 - 3 + 2 = 0$, so the point is $(1, 0)$.

$y' = 3x^2 - 3$, so $y'(1) = 0$.

**Tangent:** $y = 0$ (horizontal line, the $x$-axis)

**Normal:** $x = 1$ (vertical line)

### Example 2: $y = e^x$ at $x = 0$

Point: $(0, 1)$. Slope: $y'(0) = e^0 = 1$.

**Tangent:** $y - 1 = 1(x - 0) \Rightarrow y = x + 1$

**Normal:** $y - 1 = -1(x - 0) \Rightarrow y = -x + 1$

### Example 3: $x^2 + y^2 = 25$ at $(3, 4)$

Implicit differentiation: $2x + 2y\frac{dy}{dx} = 0 \Rightarrow \frac{dy}{dx} = -\frac{x}{y}$

At $(3, 4)$: slope $= -\frac{3}{4}$

**Tangent:** $y - 4 = -\frac{3}{4}(x - 3) \Rightarrow 3x + 4y = 25$

**Normal:** $y - 4 = \frac{4}{3}(x - 3) \Rightarrow 4x - 3y = 0$

> The normal to a circle passes through the center $(0,0)$. ✓

---

## 8.2.4 Length of Tangent, Normal, Subtangent, Subnormal

```
         y
         |
         |         P (x₀, y₀)
         |        /|
         |       / |
         |      /  | y₀
         | ____/   |
  ───────T─────────M───N──────→ x
         ←  ST  →← SN →
         ←── Length of Tangent ──→
                   ←── Length of Normal ──→
```

At point $P(x_0, y_0)$ with slope $m = f'(x_0)$:

| Quantity | Formula |
|----------|---------|
| **Subtangent** $TM$ | $\dfrac{y_0}{f'(x_0)}$ |
| **Subnormal** $MN$ | $y_0 \cdot f'(x_0)$ |
| **Length of tangent** $TP$ | $\dfrac{y_0}{f'(x_0)}\sqrt{1 + (f'(x_0))^2}$ |
| **Length of normal** $PN$ | $y_0\sqrt{1 + (f'(x_0))^2}$ |

### Example: $y^2 = 4ax$ (parabola)

$2yy' = 4a \Rightarrow y' = \frac{2a}{y}$

At point $(at^2, 2at)$: $y' = \frac{2a}{2at} = \frac{1}{t}$

- Subtangent $= \frac{2at}{1/t} = 2at^2$
- Subnormal $= 2at \cdot \frac{1}{t} = 2a$ (constant!)

> The subnormal of a parabola is constant and equals the semi-latus rectum.

---

## 8.2.5 Angle of Intersection of Two Curves

If two curves $y = f(x)$ and $y = g(x)$ intersect at a point, the **angle of intersection** $\theta$ is:

$$\tan\theta = \left|\frac{m_1 - m_2}{1 + m_1 m_2}\right|$$

where $m_1 = f'(x_0)$ and $m_2 = g'(x_0)$.

**Special cases:**

| Condition | Result |
|-----------|--------|
| $m_1 = m_2$ | Curves are tangent ($\theta = 0$) |
| $m_1 m_2 = -1$ | Curves are orthogonal ($\theta = 90°$) |

### Example: Show that $xy = 4$ and $x^2 + y^2 = 8$ are orthogonal

**Intersection:** Substituting $y = 4/x$ into $x^2 + 16/x^2 = 8$, giving $x^4 - 8x^2 + 16 = 0$, so $(x^2 - 4)^2 = 0$, $x = \pm 2$.

At $(2, 2)$:

Curve 1: $xy = 4 \Rightarrow y + xy' = 0 \Rightarrow y' = -y/x = -1$

Curve 2: $x^2 + y^2 = 8 \Rightarrow 2x + 2yy' = 0 \Rightarrow y' = -x/y = -1$

Wait—both give $m = -1$ at $(2,2)$, so they're tangent here, not orthogonal. Let me recheck.

Actually at $(2,2)$: curve 1 gives $m_1 = -2/2 = -1$; curve 2 gives $m_2 = -2/2 = -1$. So they are tangent at this point.

**Better example:** $y^2 = 4x$ and $2x^2 + y^2 = 6$.

Intersection: $2x^2 + 4x = 6 \Rightarrow x^2 + 2x - 3 = 0 \Rightarrow x = 1$ (taking positive root).

At $(1, 2)$:

Curve 1: $2yy' = 4 \Rightarrow y' = 2/y = 1$, so $m_1 = 1$

Curve 2: $4x + 2yy' = 0 \Rightarrow y' = -2x/y = -1$, so $m_2 = -1$

$m_1 \cdot m_2 = 1 \cdot (-1) = -1$ → **Orthogonal** ✓

---

## 8.2.6 Orthogonal Trajectories

Two families of curves are **orthogonal trajectories** if every curve of one family intersects every curve of the other at right angles.

**Method:** Given $F(x, y, c) = 0$:
1. Differentiate to eliminate $c$, get ODE $\frac{dy}{dx} = f(x,y)$
2. Replace $f(x,y)$ with $-\frac{1}{f(x,y)}$
3. Solve the new ODE

### Example: Orthogonal trajectories of $y = cx^2$

$y' = 2cx$. Eliminate $c = y/x^2$: $y' = 2y/x$

Orthogonal family: $y' = -x/(2y)$

$2y\,dy = -x\,dx \Rightarrow y^2 = -x^2/2 + k \Rightarrow x^2 + 2y^2 = C$

Family of ellipses — orthogonal to the family of parabolas.

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Tangent line | $y - y_0 = f'(x_0)(x - x_0)$ |
| Normal line | $y - y_0 = -\frac{1}{f'(x_0)}(x - x_0)$ |
| Subtangent | $y_0 / f'(x_0)$ |
| Subnormal | $y_0 \cdot f'(x_0)$ |
| Angle of intersection | $\tan\theta = \|(m_1 - m_2)/(1 + m_1 m_2)\|$ |
| Orthogonal curves | $m_1 \cdot m_2 = -1$ |

---

## Quick Revision Questions

**Q1.** Find the tangent and normal to $y = \ln x$ at $x = 1$.

**Q2.** At what point on $y = x^2$ is the tangent parallel to $y = 6x - 1$?

**Q3.** Find the subtangent and subnormal of $y = e^x$ at $x = 0$.

**Q4.** Find the angle of intersection of $y = x^2$ and $y = x^3$ at their common point (other than origin).

**Q5.** Show that the tangent to $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$ at $(x_0, y_0)$ is $\frac{xx_0}{a^2} + \frac{yy_0}{b^2} = 1$.

**Q6.** Find the orthogonal trajectories of the family $y = Ae^x$.

---

[← Previous: Rate of Change](01-rate-of-change.md) · [Next: Increasing & Decreasing Functions →](03-increasing-decreasing.md)
