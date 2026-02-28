# Unit 4 — Differentiability: Geometric Interpretation

[← Previous: Definition (First Principles)](01-definition-first-principles.md) · [Next: Differentiability vs Continuity →](03-differentiability-vs-continuity.md)

---

## Chapter Overview

The derivative has a beautiful geometric meaning: it is the **slope of the tangent line** to the curve $y = f(x)$ at a given point. This chapter develops that connection and shows how the derivative captures local behavior of a curve.

---

## 4.2.1 Slope of a Secant Line

The **secant line** through two points on a curve provides an *average* rate of change.

For points $P = (a, f(a))$ and $Q = (a+h, f(a+h))$:

$$m_{\text{sec}} = \frac{f(a+h) - f(a)}{h}$$

```
  y
  │                  Q ● (a+h, f(a+h))
  │                ╱ ╱
  │              ╱ ╱
  │            ╱ ╱  ← secant line
  │     P ●──╱ ╱
  │    (a, f(a))
  │        ╱ ╱
  │      ╱ ╱
  └────────────────── x
         a   a+h
```

---

## 4.2.2 Slope of the Tangent Line

As $h \to 0$, $Q$ slides along the curve toward $P$, and the secant line rotates into the **tangent line**:

$$m_{\text{tan}} = \lim_{h \to 0} \frac{f(a+h) - f(a)}{h} = f'(a)$$

```
  y         Q₁
  │        ╱  Q₂
  │       ╱ ╱  Q₃
  │      ╱╱╱╱  ← secant lines rotating
  │     P●╱╱╱╱╱╱ ← tangent line (limit)
  │    ╱╱╱╱
  │   ╱╱╱
  └──────────── x
       a
  As Q → P, secant → tangent
```

**The derivative $f'(a)$ = slope of the tangent line to $y = f(x)$ at $x = a$.**

---

## 4.2.3 Equation of the Tangent Line

At point $(a, f(a))$, the tangent line has slope $f'(a)$:

$$\boxed{y - f(a) = f'(a)(x - a)}$$

or equivalently:

$$y = f(a) + f'(a)(x - a)$$

### Worked Example

Find the equation of the tangent to $y = x^2$ at $(3, 9)$.

- $f(x) = x^2$, $f'(x) = 2x$, $f'(3) = 6$
- Tangent: $y - 9 = 6(x - 3) \Rightarrow y = 6x - 9$

```
  y
  │                  ╱ ╱ y = x²
  │                ╱ ╱
  9├─ ─ ─ ─ ─ ─ ●╱     ← (3, 9)
  │            ╱╱
  │          ╱╱  ← tangent: y = 6x - 9
  │        ╱╱
  │      ╱╱
  │    ╱╱
  └──╱───────────── x
     0   1  2  3
```

---

## 4.2.4 Equation of the Normal Line

The **normal** to a curve at a point is the line **perpendicular** to the tangent at that point.

If the tangent has slope $m = f'(a)$, the normal has slope $-\frac{1}{m}$ (negative reciprocal):

$$\boxed{y - f(a) = -\frac{1}{f'(a)}(x - a)}$$

### Worked Example

Normal to $y = x^2$ at $(3, 9)$:

- Tangent slope: $6$
- Normal slope: $-\frac{1}{6}$
- Normal: $y - 9 = -\frac{1}{6}(x - 3)$

```
  y
  │       normal ╲    ╱ tangent
  │               ╲  ╱
  │                ╲╱
  9├────────────────●── (3, 9)
  │               ╱ ╲
  │              ╱   ╲
  │             ╱     ╲
  └──────────────────── x
```

---

## 4.2.5 Geometric Meaning of Sign of Derivative

| $f'(a)$ | Geometric Meaning | Tangent Slope |
|---------|-------------------|---------------|
| $f'(a) > 0$ | Function is **increasing** at $a$ | Upward slope ╱ |
| $f'(a) < 0$ | Function is **decreasing** at $a$ | Downward slope ╲ |
| $f'(a) = 0$ | **Horizontal tangent** at $a$ | Flat ── |

```
  y
  │    f'<0      f'=0       f'>0
  │     ╲         ─          ╱
  │      ╲      ╱   ╲       ╱
  │       ╲   ╱       ╲   ╱
  │        ╲╱           ╲╱
  │      local          local
  │      max            min
  └──────────────────────── x
```

---

## 4.2.6 Linear Approximation

Near $x = a$, a differentiable function is well approximated by its tangent line:

$$f(x) \approx f(a) + f'(a)(x - a) \quad \text{for } x \text{ near } a$$

This is the foundation of **Taylor series** (Unit 9).

### Example

Approximate $\sqrt{4.02}$ using linear approximation.

Let $f(x) = \sqrt{x}$, $a = 4$, $x = 4.02$.

- $f(4) = 2$
- $f'(x) = \frac{1}{2\sqrt{x}}$, $f'(4) = \frac{1}{4}$

$$\sqrt{4.02} \approx 2 + \frac{1}{4}(4.02 - 4) = 2 + \frac{0.02}{4} = 2.005$$

(Actual value: $2.00499...$) — excellent approximation!

---

## 4.2.7 The Derivative as a Magnification Factor

Another geometric interpretation: if we "zoom in" on a differentiable function near $x = a$, the curve looks more and more like its tangent line. The function is **locally linear**.

```
  Zoomed out                Zoomed in at point P
  y                         y
  │    ╭─╮                  │     ╱
  │   ╱   ╲                 │    ╱
  │  ╱  P  ╲                │   ╱  ← looks linear!
  │ ╱   ●   ╲               │  ●
  │╱         ╲              │ ╱
  └──────────── x           └─────── x
```

---

## Summary Table

| Concept | Formula / Meaning |
|---------|-------------------|
| Slope of secant | $\frac{f(a+h) - f(a)}{h}$ |
| Slope of tangent | $f'(a) = \lim_{h \to 0} \frac{f(a+h) - f(a)}{h}$ |
| Tangent line equation | $y - f(a) = f'(a)(x - a)$ |
| Normal line equation | $y - f(a) = -\frac{1}{f'(a)}(x - a)$ |
| $f'(a) > 0$ | Function increasing (upslope) |
| $f'(a) < 0$ | Function decreasing (downslope) |
| $f'(a) = 0$ | Horizontal tangent |
| Linear approximation | $f(x) \approx f(a) + f'(a)(x - a)$ |

---

## Quick Revision Questions

**Q1.** What is the geometric meaning of $f'(3) = -2$?

**Q2.** Find the equation of the tangent and normal to $y = x^3$ at $x = 1$.

**Q3.** Use linear approximation to estimate $\sin(0.1)$ (take $a = 0$).

**Q4.** For what value(s) of $x$ does $f(x) = x^3 - 3x$ have a horizontal tangent?

**Q5.** Explain why the graph of a differentiable function looks like a straight line when you zoom in enough.

**Q6.** If the tangent to $y = f(x)$ at $(2, 5)$ has slope 3, what is the equation of the normal?

---

[← Previous: Definition (First Principles)](01-definition-first-principles.md) · [Next: Differentiability vs Continuity →](03-differentiability-vs-continuity.md)
