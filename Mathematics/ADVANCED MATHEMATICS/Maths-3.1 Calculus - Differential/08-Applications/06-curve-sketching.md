# Unit 8 — Applications of Derivatives: Curve Sketching

[← Previous: Concavity & Points of Inflection](05-concavity-inflection.md) · [Next: Mean Value Theorems →](07-mean-value-theorems.md)

---

## Chapter Overview

**Curve sketching** combines ALL derivative information — intercepts, symmetry, asymptotes, monotonicity, extrema, concavity, and inflection — to draw accurate graphs without a calculator. It is the synthesis of everything in this unit.

---

## 8.6.1 Complete Curve Sketching Checklist

```
  ┌──────────────────────────────────────────────┐
  │        CURVE SKETCHING ALGORITHM              │
  │                                               │
  │  1. Domain: where is f defined?               │
  │  2. Intercepts: x-intercepts, y-intercept     │
  │  3. Symmetry: even, odd, periodic?            │
  │  4. Asymptotes: vertical, horizontal, oblique │
  │  5. First derivative: f'(x)                   │
  │     → Critical points                         │
  │     → Intervals of increase/decrease          │
  │     → Local maxima and minima                 │
  │  6. Second derivative: f''(x)                  │
  │     → Concavity                               │
  │     → Inflection points                       │
  │  7. Special values and behavior               │
  │  8. Plot key points and sketch                │
  └──────────────────────────────────────────────┘
```

---

## 8.6.2 Symmetry

| Type | Condition | Graph Property |
|------|-----------|---------------|
| Even function | $f(-x) = f(x)$ | Symmetric about $y$-axis |
| Odd function | $f(-x) = -f(x)$ | Symmetric about origin |
| Periodic | $f(x+T) = f(x)$ | Repeats every $T$ units |

---

## 8.6.3 Asymptotes

| Type | How to Find |
|------|------------|
| **Vertical** $x = a$ | Where $f(x) \to \pm\infty$ (usually where denominator $= 0$) |
| **Horizontal** $y = L$ | $\lim_{x\to\pm\infty} f(x) = L$ |
| **Oblique** $y = mx + c$ | $m = \lim_{x\to\infty}\frac{f(x)}{x}$, $c = \lim_{x\to\infty}[f(x) - mx]$ |

---

## 8.6.4 Worked Example 1: $f(x) = \frac{x^2}{x^2 - 4}$

**1. Domain:** $x \neq \pm 2$

**2. Intercepts:** $f(0) = 0$ (origin). $x$-intercept: $x = 0$.

**3. Symmetry:** $f(-x) = f(x)$ → **even function** (symmetric about $y$-axis).

**4. Asymptotes:**

- Vertical: $x = 2, x = -2$ (denominator = 0)
- Horizontal: $\lim_{x\to\pm\infty}\frac{x^2}{x^2-4} = 1$, so $y = 1$

**5. First derivative:**

$$f'(x) = \frac{2x(x^2-4) - x^2(2x)}{(x^2-4)^2} = \frac{-8x}{(x^2-4)^2}$$

$f'(x) = 0 \Rightarrow x = 0$

| Interval | $f'$ sign | Behavior |
|----------|-----------|----------|
| $(-\infty, -2)$ | $+$ | Increasing |
| $(-2, 0)$ | $+$ | Increasing |
| $(0, 2)$ | $-$ | Decreasing |
| $(2, \infty)$ | $-$ | Decreasing |

Local max at $(0, 0)$.

**6. Second derivative:** (for concavity — computation omitted for brevity)

$$f''(x) = \frac{8(3x^2 + 4)}{(x^2-4)^3}$$

Since $3x^2 + 4 > 0$ always, the sign depends on $(x^2-4)^3$:

| Interval | $(x^2-4)^3$ | $f''$ | Concavity |
|----------|-------------|-------|-----------|
| $|x| < 2$ | $-$ | $-$ | Down |
| $|x| > 2$ | $+$ | $+$ | Up |

No inflection (asymptotes interrupt).

**Sketch:**

```
  y
  |   *                              *   ← concave up, approaching y=1
  |    *                            *
  1 ----*---------asymptote--------*---- y = 1
  |      *                        *
  |       *                      *
  |        *   ← concave down  *
  0 +-------*───────*───────*──────→ x
  |          ↑              ↑
  |         x=-2           x=2
  |          (vertical      (vertical
  |          asymptote)     asymptote)
```

---

## 8.6.5 Worked Example 2: $f(x) = xe^{-x}$

**1. Domain:** $\mathbb{R}$

**2. Intercepts:** $f(0) = 0$. $x$-intercept at $x = 0$.

**3. Symmetry:** Neither even nor odd.

**4. Asymptotes:**
- No vertical asymptotes
- $\lim_{x\to\infty}xe^{-x} = 0$ (horizontal asymptote $y = 0$ as $x \to \infty$)
- $\lim_{x\to -\infty}xe^{-x} = -\infty$ (no asymptote on left)

**5. First derivative:**

$$f'(x) = e^{-x}(1-x)$$

$f'(x) = 0 \Rightarrow x = 1$

At $x = 1$: $f(1) = e^{-1} \approx 0.368$ → local max (changes from $+$ to $-$)

**6. Second derivative:**

$$f''(x) = e^{-x}(x-2)$$

$f''(x) = 0 \Rightarrow x = 2$, $f(2) = 2e^{-2} \approx 0.271$

Inflection point at $(2, 2e^{-2})$.

**Sketch:**

```
  y
  |
  |       * (1, 1/e) ← local max
  |      / \
  |     /   \
  |    /     *  (2, 2/e²) ← inflection
  |   /       \
  |  /         \_______  → y = 0
  | /
  */
  +──────────────────────→ x
  0     1     2     3
```

---

## 8.6.6 Worked Example 3: $f(x) = x^3 - 3x^2 + 4$

**1. Domain:** $\mathbb{R}$

**2. Intercepts:** $f(0) = 4$. Factor: $f(x) = (x+1)(x-2)^2$, so $x$-intercepts at $x = -1, 2$.

**3. Symmetry:** None.

**4. Asymptotes:** None (polynomial).

**5. First derivative:**

$$f'(x) = 3x^2 - 6x = 3x(x-2)$$

- $x = 0$: local max, $f(0) = 4$
- $x = 2$: local min, $f(2) = 0$

**6. Second derivative:**

$$f''(x) = 6x - 6$$

$f''(x) = 0$ at $x = 1$: inflection point at $(1, 2)$.

**Sketch:**

```
  y
  4 +* (0, 4) ← local max
    | \
    |  \
  2 +   * (1, 2) ← inflection
    |    \
    |     \
  0 +──────*────→ x  (2, 0) ← local min (touches x-axis)
    |     / \
 -1 *    /
   (-1,0)
```

---

## 8.6.7 Summary of Key Features to Identify

```
  ┌─────────────────────────────────────────┐
  │              f(x)                        │
  │  • Domain, intercepts, symmetry         │
  │  • End behavior                          │
  ├─────────────────────────────────────────┤
  │              f'(x)                       │
  │  • Critical points (f' = 0 or DNE)      │
  │  • Increasing (f' > 0)                  │
  │  • Decreasing (f' < 0)                  │
  │  • Local max/min                         │
  ├─────────────────────────────────────────┤
  │              f''(x)                      │
  │  • Concave up (f'' > 0)                 │
  │  • Concave down (f'' < 0)               │
  │  • Inflection points (f'' sign change)  │
  └─────────────────────────────────────────┘
```

---

## Summary Table

| Step | What to Find |
|------|-------------|
| Domain | Values where $f$ is defined |
| Intercepts | $y = f(0)$; solve $f(x) = 0$ |
| Symmetry | Even/odd/periodic |
| Asymptotes | VA, HA, oblique |
| $f'(x)$ | Monotonicity, extrema |
| $f''(x)$ | Concavity, inflection |
| End behavior | $\lim_{x\to\pm\infty}f(x)$ |

---

## Quick Revision Questions

**Q1.** Sketch $f(x) = \frac{x}{x^2+1}$ using the complete checklist.

**Q2.** Sketch $f(x) = x^2 e^{-x}$ for $x \geq 0$.

**Q3.** Find all key features and sketch $f(x) = \frac{x^2 - 1}{x^2 + 1}$.

**Q4.** Sketch $f(x) = x - \ln(1+x)$ for $x > -1$.

**Q5.** How many inflection points does $f(x) = x^5 - 5x^3$ have? Find them and sketch.

**Q6.** Describe the differences in the graphs of $y = x^2$, $y = x^4$, and $y = x^6$ near $x = 0$ in terms of concavity and "flatness."

---

[← Previous: Concavity & Points of Inflection](05-concavity-inflection.md) · [Next: Mean Value Theorems →](07-mean-value-theorems.md)
