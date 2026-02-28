# Unit 5 — Differentiation Rules: Implicit Differentiation

[← Previous: Chain Rule](03-chain-rule.md) · [Next: Logarithmic Differentiation →](05-logarithmic-differentiation.md)

---

## Chapter Overview

So far, we've differentiated functions written **explicitly** as $y = f(x)$. But many important curves (circles, ellipses, etc.) are defined by equations relating $x$ and $y$ **implicitly** — for example, $x^2 + y^2 = 25$. **Implicit differentiation** lets us find $\frac{dy}{dx}$ directly from such equations, without first solving for $y$.

---

## 5.4.1 The Idea

Given an equation $F(x, y) = 0$ where $y$ is implicitly a function of $x$:

1. Differentiate **both sides** with respect to $x$.
2. Every time you differentiate a term involving $y$, apply the **chain rule**: $\frac{d}{dx}[g(y)] = g'(y) \cdot \frac{dy}{dx}$.
3. Solve the resulting equation for $\frac{dy}{dx}$.

---

## 5.4.2 Key Principle

$$\frac{d}{dx}[y^n] = ny^{n-1} \cdot \frac{dy}{dx}$$

Because $y$ depends on $x$, the chain rule kicks in automatically.

| Term | Derivative w.r.t. $x$ |
|------|----------------------|
| $x^3$ | $3x^2$ |
| $y^3$ | $3y^2 \cdot \frac{dy}{dx}$ |
| $xy$ | $y + x\frac{dy}{dx}$ (product rule) |
| $\sin y$ | $\cos y \cdot \frac{dy}{dx}$ |
| $e^y$ | $e^y \cdot \frac{dy}{dx}$ |
| $x^2 y^3$ | $2xy^3 + 3x^2 y^2 \frac{dy}{dx}$ |

---

## 5.4.3 Worked Examples

### Example 1: Circle $x^2 + y^2 = 25$

Differentiate both sides:

$$2x + 2y\frac{dy}{dx} = 0$$

$$\frac{dy}{dx} = -\frac{x}{y}$$

**Interpretation:** At $(3, 4)$: $\frac{dy}{dx} = -\frac{3}{4}$ (tangent slopes downward to the right on the upper semicircle).

```
  y
  4├─ ─ ─ ─●╲          ← tangent at (3,4)
  │      ╱   ╲              slope = -3/4
  │    ╱       ╲
  │  ╱   circle  ╲
  │╱    x²+y²=25   ╲
  ├──────────────────── x
  │                5
```

### Example 2: $x^3 + y^3 = 6xy$ (Folium of Descartes)

Differentiate:

$$3x^2 + 3y^2\frac{dy}{dx} = 6y + 6x\frac{dy}{dx}$$

Collect $\frac{dy}{dx}$ terms:

$$3y^2\frac{dy}{dx} - 6x\frac{dy}{dx} = 6y - 3x^2$$

$$\frac{dy}{dx}(3y^2 - 6x) = 6y - 3x^2$$

$$\frac{dy}{dx} = \frac{6y - 3x^2}{3y^2 - 6x} = \frac{2y - x^2}{y^2 - 2x}$$

### Example 3: $\sin(xy) = x + y$

Differentiate:

$$\cos(xy) \cdot \left(y + x\frac{dy}{dx}\right) = 1 + \frac{dy}{dx}$$

Expand:

$$y\cos(xy) + x\cos(xy)\frac{dy}{dx} = 1 + \frac{dy}{dx}$$

Collect:

$$\frac{dy}{dx}[x\cos(xy) - 1] = 1 - y\cos(xy)$$

$$\frac{dy}{dx} = \frac{1 - y\cos(xy)}{x\cos(xy) - 1}$$

### Example 4: $e^y + xy = e$

Differentiate:

$$e^y\frac{dy}{dx} + y + x\frac{dy}{dx} = 0$$

$$\frac{dy}{dx}(e^y + x) = -y$$

$$\frac{dy}{dx} = \frac{-y}{e^y + x}$$

---

## 5.4.4 Finding the Second Derivative Implicitly

To find $\frac{d^2y}{dx^2}$, differentiate $\frac{dy}{dx}$ again with respect to $x$, using the result for $\frac{dy}{dx}$ as needed.

### Example: $x^2 + y^2 = 25$

We found $\frac{dy}{dx} = -\frac{x}{y}$. Now:

$$\frac{d^2y}{dx^2} = \frac{d}{dx}\left(-\frac{x}{y}\right) = -\frac{y \cdot 1 - x \cdot \frac{dy}{dx}}{y^2}$$

$$= -\frac{y - x \cdot (-x/y)}{y^2} = -\frac{y + x^2/y}{y^2} = -\frac{y^2 + x^2}{y^3} = -\frac{25}{y^3}$$

---

## 5.4.5 Applications

### Finding tangent lines to implicit curves

**Example:** Find the tangent to $x^2 + y^2 = 25$ at $(3, 4)$.

- Slope: $\frac{dy}{dx}\big|_{(3,4)} = -\frac{3}{4}$
- Tangent: $y - 4 = -\frac{3}{4}(x - 3) \Rightarrow 3x + 4y = 25$

### Orthogonal trajectories

Two families of curves are **orthogonal** if at every intersection, their tangent lines are perpendicular. Implicit differentiation helps find the slope of one family, then the orthogonal family has slope $-1/m$.

---

## 5.4.6 Step-by-Step Process

```
  ┌──────────────────────────────────┐
  │ 1. Start with F(x, y) = 0       │
  │                                  │
  │ 2. Differentiate both sides      │
  │    w.r.t. x                      │
  │    • Use chain rule for y terms  │
  │    • Use product rule for xy     │
  │                                  │
  │ 3. Collect all dy/dx terms       │
  │    on one side                   │
  │                                  │
  │ 4. Factor out dy/dx              │
  │                                  │
  │ 5. Solve for dy/dx               │
  └──────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Implicit differentiation | Differentiate both sides w.r.t. $x$; chain rule for $y$ terms |
| Chain rule for $y$ | $\frac{d}{dx}[g(y)] = g'(y)\frac{dy}{dx}$ |
| Product rule for $xy$ | $\frac{d}{dx}[xy] = y + x\frac{dy}{dx}$ |
| Solving for $\frac{dy}{dx}$ | Collect, factor, divide |
| Second derivative | Differentiate $\frac{dy}{dx}$ again, substitute back |

---

## Quick Revision Questions

**Q1.** Find $\frac{dy}{dx}$ if $x^2 + y^2 = 16$.

**Q2.** Find $\frac{dy}{dx}$ if $x^2y + xy^2 = 6$.

**Q3.** For the ellipse $\frac{x^2}{9} + \frac{y^2}{4} = 1$, find the slope at $(x_0, y_0)$.

**Q4.** Find the equation of the tangent to $x^2 + xy + y^2 = 7$ at $(1, 2)$.

**Q5.** If $y = \sin(x + y)$, find $\frac{dy}{dx}$.

**Q6.** Find $\frac{d^2y}{dx^2}$ for $xy = 1$.

---

[← Previous: Chain Rule](03-chain-rule.md) · [Next: Logarithmic Differentiation →](05-logarithmic-differentiation.md)
