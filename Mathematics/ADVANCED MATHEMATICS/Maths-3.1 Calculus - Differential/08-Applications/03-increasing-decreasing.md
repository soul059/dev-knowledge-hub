# Unit 8 — Applications of Derivatives: Increasing & Decreasing Functions

[← Previous: Tangent & Normal Lines](02-tangent-normal.md) · [Next: Maxima & Minima →](04-maxima-minima.md)

---

## Chapter Overview

The **sign of the first derivative** tells us whether a function is increasing or decreasing. This chapter formalizes the increasing/decreasing test, covers monotonic functions, and lays the groundwork for extrema analysis.

---

## 8.3.1 Definitions

| Term | Definition | Derivative Condition |
|------|-----------|---------------------|
| **Increasing** on $(a,b)$ | $x_1 < x_2 \Rightarrow f(x_1) < f(x_2)$ | $f'(x) > 0$ |
| **Decreasing** on $(a,b)$ | $x_1 < x_2 \Rightarrow f(x_1) > f(x_2)$ | $f'(x) < 0$ |
| **Non-decreasing** | $x_1 < x_2 \Rightarrow f(x_1) \leq f(x_2)$ | $f'(x) \geq 0$ |
| **Non-increasing** | $x_1 < x_2 \Rightarrow f(x_1) \geq f(x_2)$ | $f'(x) \leq 0$ |
| **Monotonic** | Either always increasing or always decreasing | $f'$ doesn't change sign |

---

## 8.3.2 The First Derivative Test for Monotonicity

$$f'(x) > 0 \text{ on } (a,b) \Rightarrow f \text{ is strictly increasing on } (a,b)$$

$$f'(x) < 0 \text{ on } (a,b) \Rightarrow f \text{ is strictly decreasing on } (a,b)$$

```
  Increasing (f' > 0)         Decreasing (f' < 0)

  y         *                  y  *
  |       *                    |    *
  |     *                      |      *
  |   *                        |        *
  | *                          |          *
  +───────→ x                  +───────────→ x
    slope > 0                    slope < 0
```

---

## 8.3.3 Method: Finding Intervals of Increase/Decrease

```
  ┌────────────────────────────────────────────┐
  │ 1. Find f'(x)                             │
  │ 2. Find critical points: f'(x) = 0 or DNE │
  │ 3. Make a sign chart for f'(x)            │
  │ 4. f' > 0 → increasing, f' < 0 → decreasing│
  └────────────────────────────────────────────┘
```

### Example 1: $f(x) = x^3 - 3x^2 + 1$

$$f'(x) = 3x^2 - 6x = 3x(x-2)$$

Critical points: $x = 0, 2$

**Sign chart:**

```
  x:      -∞ ────── 0 ────── 2 ────── +∞
  3x:         -      0    +       +
  (x-2):      -      -    -    0  +
  f'(x):      +      0    -    0  +
              ↑inc    ↓dec       ↑inc
```

| Interval | $f'(x)$ | Behavior |
|----------|---------|----------|
| $(-\infty, 0)$ | $+$ | Increasing |
| $(0, 2)$ | $-$ | Decreasing |
| $(2, \infty)$ | $+$ | Increasing |

### Example 2: $f(x) = xe^{-x}$

$$f'(x) = e^{-x} - xe^{-x} = e^{-x}(1-x)$$

Since $e^{-x} > 0$ always, the sign depends on $(1-x)$:

$$f'(x) > 0 \text{ when } x < 1, \quad f'(x) < 0 \text{ when } x > 1$$

Increasing on $(-\infty, 1)$, decreasing on $(1, \infty)$.

### Example 3: $f(x) = \sin x + \cos x$ on $[0, 2\pi]$

$$f'(x) = \cos x - \sin x = \sqrt{2}\cos\left(x + \frac{\pi}{4}\right)$$

$f'(x) = 0$ when $x + \frac{\pi}{4} = \frac{\pi}{2}, \frac{3\pi}{2}$, i.e., $x = \frac{\pi}{4}, \frac{5\pi}{4}$

| Interval | $f'(x)$ | Behavior |
|----------|---------|----------|
| $(0, \pi/4)$ | $+$ | Increasing |
| $(\pi/4, 5\pi/4)$ | $-$ | Decreasing |
| $(5\pi/4, 2\pi)$ | $+$ | Increasing |

---

## 8.3.4 Proving Inequalities Using Monotonicity

A powerful application: to prove $f(x) > g(x)$, define $h(x) = f(x) - g(x)$ and show $h$ is increasing from a point where $h = 0$.

### Example: Show $e^x > 1 + x$ for $x > 0$

Let $h(x) = e^x - 1 - x$. Then $h(0) = 0$ and $h'(x) = e^x - 1 > 0$ for $x > 0$.

Since $h$ is increasing on $(0, \infty)$ and $h(0) = 0$:

$$h(x) > h(0) = 0 \Rightarrow e^x > 1 + x \quad \text{for } x > 0 \quad \blacksquare$$

### Example: Show $\sin x < x$ for $x > 0$

Let $h(x) = x - \sin x$. Then $h(0) = 0$ and $h'(x) = 1 - \cos x \geq 0$.

So $h$ is non-decreasing, and $h'(x) = 0$ only at isolated points ($x = 2n\pi$), so $h$ is strictly increasing.

$$h(x) > 0 \text{ for } x > 0 \Rightarrow x > \sin x \quad \blacksquare$$

---

## 8.3.5 Critical Points

A **critical point** of $f$ is a value $c$ in the domain of $f$ where:

$$f'(c) = 0 \quad \text{or} \quad f'(c) \text{ does not exist}$$

Critical points are the **only candidates** for local extrema.

| Type | Example |
|------|---------|
| $f'(c) = 0$ | Stationary point: $f(x) = x^2$ at $x = 0$ |
| $f'(c)$ DNE | Corner: $f(x) = \|x\|$ at $x = 0$ |
| $f'(c)$ DNE | Cusp: $f(x) = x^{2/3}$ at $x = 0$ |
| $f'(c)$ DNE | Vertical tangent: $f(x) = x^{1/3}$ at $x = 0$ |

---

## Summary Table

| Concept | Key Test |
|---------|----------|
| Increasing | $f'(x) > 0$ |
| Decreasing | $f'(x) < 0$ |
| Critical point | $f'(c) = 0$ or $f'(c)$ DNE |
| Method | Sign chart of $f'$ on intervals between critical points |
| Proving inequalities | Show $h = f - g$ is monotonic from a known value |

---

## Quick Revision Questions

**Q1.** Find the intervals where $f(x) = 2x^3 - 9x^2 + 12x - 5$ is increasing/decreasing.

**Q2.** Show that $f(x) = x - \ln(1+x)$ is increasing for $x > 0$, and deduce that $\ln(1+x) < x$ for $x > 0$.

**Q3.** Find all critical points of $f(x) = x^{4/3} - 4x^{1/3}$.

**Q4.** Is $f(x) = \tan^{-1} x$ increasing or decreasing on $\mathbb{R}$? Justify.

**Q5.** Find the intervals of monotonicity for $f(x) = x\ln x$ (for $x > 0$).

**Q6.** Prove that $\tan x > x$ for $0 < x < \frac{\pi}{2}$ using derivatives.

---

[← Previous: Tangent & Normal Lines](02-tangent-normal.md) · [Next: Maxima & Minima →](04-maxima-minima.md)
