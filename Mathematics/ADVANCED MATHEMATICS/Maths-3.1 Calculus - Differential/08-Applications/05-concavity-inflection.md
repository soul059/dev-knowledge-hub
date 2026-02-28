# Unit 8 — Applications of Derivatives: Concavity & Points of Inflection

[← Previous: Maxima & Minima](04-maxima-minima.md) · [Next: Curve Sketching →](06-curve-sketching.md)

---

## Chapter Overview

The **second derivative** describes the shape of a curve: whether it bends upward (concave up) or downward (concave down). Points where the concavity changes are **inflection points**. These are essential for complete curve analysis.

---

## 8.5.1 Concavity

| Condition | Shape | Description |
|-----------|-------|------------|
| $f''(x) > 0$ | Concave UP (∪) | Curve lies **above** its tangent lines |
| $f''(x) < 0$ | Concave DOWN (∩) | Curve lies **below** its tangent lines |

```
  Concave UP (f'' > 0)            Concave DOWN (f'' < 0)

  y                               y
  |           *                   |  *─────────*
  |          /                    |   \       /
  |        /                      |    \     /
  |      *  ← tangent below       |     \   /
  |    / the curve                |      * ←tangent above
  |  /                            |         the curve
  +──────→ x                      +──────────→ x

  f' is increasing               f' is decreasing
```

**Geometric interpretation:**

| Concave UP | Concave DOWN |
|------------|-------------|
| $f'$ is increasing | $f'$ is decreasing |
| Slope gets steeper (or less negative) | Slope gets less steep (or more negative) |
| Curve turns left (anticlockwise) | Curve turns right (clockwise) |

---

## 8.5.2 Points of Inflection

> A **point of inflection** is where the concavity **changes** (from up to down, or down to up).

**Necessary condition:** $f''(c) = 0$ or $f''(c)$ does not exist.

**Sufficient condition:** $f''$ changes sign at $c$.

> **Warning:** $f''(c) = 0$ alone is NOT sufficient. For example, $f(x) = x^4$ has $f''(0) = 0$ but **no** inflection point (concave up on both sides).

```
  ┌──────────────────────────────────────────┐
  │           Inflection Point Test          │
  │                                          │
  │ 1. Find where f''(x) = 0 or f'' DNE     │
  │ 2. Check if f'' changes sign             │
  │    YES → inflection point               │
  │    NO  → not an inflection point        │
  └──────────────────────────────────────────┘
```

---

## 8.5.3 Worked Examples

### Example 1: $f(x) = x^3 - 3x^2 + 2$

$$f'(x) = 3x^2 - 6x, \quad f''(x) = 6x - 6$$

$f''(x) = 0 \Rightarrow x = 1$

| Interval | $f''$ | Concavity |
|----------|-------|-----------|
| $x < 1$ | $-$ | Concave down |
| $x > 1$ | $+$ | Concave up |

Sign changes at $x = 1$ → **inflection point** at $(1, 0)$.

### Example 2: $f(x) = x^4 - 6x^2$

$$f''(x) = 12x^2 - 12 = 12(x-1)(x+1)$$

$f''(x) = 0$ at $x = \pm 1$

| Interval | $f''$ | Concavity |
|----------|-------|-----------|
| $x < -1$ | $+$ | UP |
| $-1 < x < 1$ | $-$ | DOWN |
| $x > 1$ | $+$ | UP |

Inflection points: $(-1, -5)$ and $(1, -5)$.

### Example 3: $f(x) = x^4$ — No inflection at $x = 0$

$f''(x) = 12x^2$, $f''(0) = 0$

But $f''(x) \geq 0$ for all $x$ (no sign change) → **not** an inflection point.

---

## 8.5.4 Third Derivative Test for Inflection

If $f''(c) = 0$ and $f'''(c) \neq 0$, then $c$ is a point of inflection.

| Condition | Conclusion |
|-----------|------------|
| $f''(c) = 0$, $f'''(c) \neq 0$ | Inflection point ✓ |
| $f''(c) = 0$, $f'''(c) = 0$ | Inconclusive — check sign change |

### Example: $f(x) = x^3$

$f''(x) = 6x$, $f''(0) = 0$

$f'''(x) = 6 \neq 0$ → inflection point at $(0, 0)$ ✓

---

## 8.5.5 Concavity and Tangent Lines

At a point of inflection, the tangent line **crosses** the curve:

```
              *
             /     ← curve goes from below tangent
     ───────*─────── tangent line at inflection point
           /
          *        ← to above tangent (or vice-versa)
```

This is because the curve transitions from lying on one side of its tangent to the other.

---

## 8.5.6 Complete Analysis Table

| $f'(c)$ | $f''(c)$ | Type of Point |
|---------|----------|---------------|
| $0$ | $> 0$ | Local minimum |
| $0$ | $< 0$ | Local maximum |
| $0$ | $0$ | Could be max, min, or inflection |
| $\neq 0$ | $0$ (sign change) | Inflection point (not extremum) |

---

## Summary Table

| Concept | Key Test |
|---------|----------|
| Concave up | $f'' > 0$ |
| Concave down | $f'' < 0$ |
| Inflection point | $f'' = 0$ (or DNE) AND $f''$ changes sign |
| Third derivative test | $f''(c)=0$, $f'''(c)\neq 0$ → inflection |
| Tangent at inflection | Crosses the curve |

---

## Quick Revision Questions

**Q1.** Find the intervals of concavity and inflection points for $f(x) = x^3 - 6x^2 + 12x - 4$.

**Q2.** Show that $f(x) = x^4$ has no point of inflection at $x = 0$ even though $f''(0) = 0$.

**Q3.** Find all inflection points of $f(x) = x e^{-x}$.

**Q4.** Determine the concavity of $f(x) = \ln x$ for $x > 0$.

**Q5.** If $f(x) = \sin x$, find all inflection points in $[0, 2\pi]$.

**Q6.** Prove that a polynomial of degree $n$ has at most $n - 2$ inflection points.

---

[← Previous: Maxima & Minima](04-maxima-minima.md) · [Next: Curve Sketching →](06-curve-sketching.md)
