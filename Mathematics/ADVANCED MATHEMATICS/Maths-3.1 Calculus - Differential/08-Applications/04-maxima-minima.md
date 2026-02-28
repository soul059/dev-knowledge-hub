# Unit 8 — Applications of Derivatives: Maxima & Minima

[← Previous: Increasing & Decreasing Functions](03-increasing-decreasing.md) · [Next: Concavity & Points of Inflection →](05-concavity-inflection.md)

---

## Chapter Overview

Finding the maximum and minimum values of a function is one of the most important applications of calculus. This chapter covers local and absolute extrema, the first and second derivative tests, and optimization problems.

---

## 8.4.1 Definitions

| Term | Definition |
|------|-----------|
| **Local maximum** | $f(c) \geq f(x)$ for all $x$ near $c$ |
| **Local minimum** | $f(c) \leq f(x)$ for all $x$ near $c$ |
| **Absolute maximum** | $f(c) \geq f(x)$ for all $x$ in the domain |
| **Absolute minimum** | $f(c) \leq f(x)$ for all $x$ in the domain |
| **Extremum** | Either a maximum or minimum |

```
  y
  |     * ← absolute max
  |    / \      local max
  |   /   \    * 
  |  /     \  / \
  | /       *    \     ← local min
  |/              \
  * ← absolute min \*
  +─────────────────→ x
```

---

## 8.4.2 Fermat's Theorem

> If $f$ has a local extremum at $c$ and $f'(c)$ exists, then $f'(c) = 0$.

**Contrapositive:** If $f'(c) \neq 0$, there is no local extremum at $c$.

**Warning:** The converse is FALSE. $f'(c) = 0$ does NOT guarantee an extremum (e.g., $f(x) = x^3$ at $x = 0$).

---

## 8.4.3 First Derivative Test

At a critical point $c$ where $f'(c) = 0$:

| $f'$ changes from | Conclusion |
|--------------------|------------|
| $+$ to $-$ | **Local maximum** at $c$ |
| $-$ to $+$ | **Local minimum** at $c$ |
| $+$ to $+$ or $-$ to $-$ | **No extremum** (inflection point) |

```
  Local Maximum        Local Minimum        No Extremum
  (+ to -)             (- to +)             (+ to +)

      *                                         *
     / \                                       /
    /   \               \   /                 *
   /     \               \ /                 /
  / f'>0  \ f'<0    f'<0  *  f'>0           / f'>0
                                            *
```

### Example: $f(x) = x^3 - 3x + 1$

$$f'(x) = 3x^2 - 3 = 3(x-1)(x+1)$$

Critical points: $x = -1, 1$

| Interval | $f'$ sign | Behavior |
|----------|-----------|----------|
| $(-\infty, -1)$ | $+$ | Increasing |
| $(-1, 1)$ | $-$ | Decreasing |
| $(1, \infty)$ | $+$ | Increasing |

- At $x = -1$: $f'$ changes from $+$ to $-$ → **local max**, $f(-1) = 3$
- At $x = 1$: $f'$ changes from $-$ to $+$ → **local min**, $f(1) = -1$

---

## 8.4.4 Second Derivative Test

At a critical point $c$ where $f'(c) = 0$:

| Condition | Conclusion |
|-----------|------------|
| $f''(c) < 0$ | **Local maximum** |
| $f''(c) > 0$ | **Local minimum** |
| $f''(c) = 0$ | **Test inconclusive** (use first derivative test) |

**Mnemonic:** "Smile positive, frown negative"
- $f'' > 0$: concave UP (∪ = smile) → minimum
- $f'' < 0$: concave DOWN (∩ = frown) → maximum

### Example: $f(x) = x^4 - 4x^3 + 6$

$$f'(x) = 4x^3 - 12x^2 = 4x^2(x - 3)$$

Critical points: $x = 0, 3$

$$f''(x) = 12x^2 - 24x$$

- $f''(0) = 0$ → inconclusive!
- $f''(3) = 108 - 72 = 36 > 0$ → **local min** at $x = 3$, $f(3) = -21$

For $x = 0$: Use first derivative test. $f'$ goes from $-$ to $-$ (check: $f'(-1) = -16$, $f'(1) = -8$). **No extremum** at $x = 0$.

---

## 8.4.5 Absolute Extrema on Closed Intervals

The **Extreme Value Theorem** guarantees that a continuous function on $[a, b]$ attains its absolute max and min.

```
  ┌──────────────────────────────────────────┐
  │ To find absolute extrema on [a, b]:     │
  │ 1. Find all critical points in (a, b)   │
  │ 2. Evaluate f at critical points         │
  │ 3. Evaluate f at endpoints: f(a), f(b)  │
  │ 4. Largest = absolute max               │
  │ 5. Smallest = absolute min              │
  └──────────────────────────────────────────┘
```

### Example: $f(x) = 2x^3 - 9x^2 + 12x$ on $[0, 4]$

$f'(x) = 6x^2 - 18x + 12 = 6(x-1)(x-2)$

Critical points in $(0, 4)$: $x = 1, 2$

| $x$ | $f(x)$ |
|-----|--------|
| $0$ | $0$ |
| $1$ | $5$ |
| $2$ | $4$ |
| $4$ | $16$ |

**Absolute max = 16** at $x = 4$, **Absolute min = 0** at $x = 0$.

---

## 8.4.6 Optimization Problems

**Strategy:**

```
  ┌──────────────────────────────────────────┐
  │ 1. Understand: what to maximize/minimize?│
  │ 2. Variables: define and relate them     │
  │ 3. Objective function: express in ONE var│
  │ 4. Domain: find valid range              │
  │ 5. Differentiate and find critical points│
  │ 6. Test: verify max/min                  │
  │ 7. Answer: give the requested value      │
  └──────────────────────────────────────────┘
```

### Example 1: Maximum Area Rectangle

Find the rectangle of maximum area inscribed in a circle of radius $R$.

Let half-sides be $x, y$ with $x^2 + y^2 = R^2$.

$A = 4xy = 4x\sqrt{R^2 - x^2}$

$$\frac{dA}{dx} = 4\sqrt{R^2 - x^2} + 4x \cdot \frac{-x}{\sqrt{R^2-x^2}} = \frac{4(R^2 - 2x^2)}{\sqrt{R^2 - x^2}}$$

$\frac{dA}{dx} = 0 \Rightarrow x^2 = R^2/2 \Rightarrow x = R/\sqrt{2}$, so $y = R/\sqrt{2}$.

**The rectangle is a square** with side $R\sqrt{2}$ and area $2R^2$.

### Example 2: Box with Maximum Volume

An open box is made from a square sheet (side $a$) by cutting equal squares from corners.

```
  ┌──┬────────┬──┐
  │x │        │x │
  ├──┘        └──┤
  │              │
  │     a-2x     │
  │              │
  ├──┐        ┌──┤
  │x │        │x │
  └──┴────────┴──┘
```

$V = x(a-2x)^2$, domain: $0 < x < a/2$

$$V'(x) = (a-2x)^2 + x \cdot 2(a-2x)(-2) = (a-2x)(a-6x)$$

$V'(x) = 0 \Rightarrow x = a/2$ (endpoint) or $x = a/6$

$V''(a/6) = -2(a - a/3) - 4(a/3 - a/3) + \cdots$ (or just check: second derivative is negative → maximum)

$$V_{\max} = \frac{a}{6}\left(\frac{2a}{3}\right)^2 = \frac{2a^3}{27}$$

---

## 8.4.7 Comparison of Tests

| | First Derivative Test | Second Derivative Test |
|---|---|---|
| **Information needed** | Sign of $f'$ around $c$ | Values of $f'(c)$ and $f''(c)$ |
| **Always works?** | Yes | No (fails if $f''(c) = 0$) |
| **Shows increasing/decreasing** | Yes | No |
| **Easier when** | $f''$ is hard to compute | $f''$ is easy to compute |

---

## Summary Table

| Concept | Key Test |
|---------|----------|
| Local max | $f'$ changes $+$ to $-$, or $f'' < 0$ |
| Local min | $f'$ changes $-$ to $+$, or $f'' > 0$ |
| Absolute extrema on $[a,b]$ | Compare $f$ at critical points and endpoints |
| Optimization | Reduce to single-variable, differentiate, test |
| Fermat's theorem | $f'(c) = 0$ at any local extremum (if $f'$ exists) |

---

## Quick Revision Questions

**Q1.** Find the local extrema of $f(x) = x^4 - 8x^2 + 3$ using the second derivative test.

**Q2.** Find the absolute max and min of $f(x) = x^3 - 3x + 1$ on $[-2, 2]$.

**Q3.** A farmer has 200 m of fencing. Find dimensions of the rectangular field of maximum area.

**Q4.** Find two positive numbers whose sum is 20 and whose product is maximum.

**Q5.** Show that among all rectangles with a given perimeter, the square has the largest area.

**Q6.** A tin can with fixed volume $V$ is to have minimum surface area. Find the ratio of height to radius.

---

[← Previous: Increasing & Decreasing Functions](03-increasing-decreasing.md) · [Next: Concavity & Points of Inflection →](05-concavity-inflection.md)
