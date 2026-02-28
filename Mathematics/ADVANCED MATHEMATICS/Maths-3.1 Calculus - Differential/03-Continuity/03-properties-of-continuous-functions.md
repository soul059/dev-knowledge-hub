# Unit 3 — Continuity: Properties of Continuous Functions

[← Previous: Types of Discontinuity](02-types-of-discontinuity.md) · [Next: Intermediate Value Theorem →](04-intermediate-value-theorem.md)

---

## Chapter Overview

Continuous functions enjoy powerful properties that make them the workhorses of calculus. This chapter covers the main theorems about what continuous functions *must* do, especially on closed intervals — boundedness, attaining extreme values, and the preservation of intervals.

---

## 3.3.1 Algebra of Continuous Functions (Recap)

If $f$ and $g$ are continuous at $x = a$, then:

| Function | Continuous at $a$? |
|----------|-------------------|
| $f + g$ | Yes |
| $f - g$ | Yes |
| $cf$ (constant multiple) | Yes |
| $f \cdot g$ | Yes |
| $f / g$ | Yes, if $g(a) \ne 0$ |
| $f \circ g$ | Yes (if $g$ cont. at $a$ and $f$ cont. at $g(a)$) |
| $|f|$ | Yes |

> **Consequence:** The set of discontinuities of $f + g$ is contained in the union of discontinuities of $f$ and $g$.

---

## 3.3.2 The Extreme Value Theorem (EVT)

### Statement

If $f$ is **continuous on a closed interval** $[a, b]$, then $f$ attains both:
- a **maximum** value $M$ at some $c \in [a, b]$
- a **minimum** value $m$ at some $d \in [a, b]$

$$\exists\, c, d \in [a, b] : \quad f(d) \le f(x) \le f(c) \quad \forall\, x \in [a, b]$$

### Graphical Interpretation

```
  y
  │
  M├─ ─ ─ ─ ─ ─●─ ─ ─ ─ ─    ← maximum at x = c
  │           ╱   ╲
  │         ╱       ╲
  │       ╱           ╲
  │     ╱               ╲
  m├─ ●─ ─ ─ ─ ─ ─ ─ ─ ─ ╲   ← minimum at x = d
  │   │                     │
  └───┼──┼────────────┼──┼── x
      a  d            c  b
      [─────── closed interval ──────]
```

### Why Closed Interval?

| Condition violated | Counterexample |
|-------------------|---------------|
| Open interval $(0, 1)$ | $f(x) = x$ — no max (approaches 1 but never reaches it) |
| $f$ not continuous | $f(x) = \begin{cases} x & x \in [0,1) \\ 0 & x = 1 \end{cases}$ — no max |
| Unbounded interval $[0, \infty)$ | $f(x) = x$ — no max |

---

## 3.3.3 Boundedness Theorem

### Statement

If $f$ is continuous on $[a, b]$, then $f$ is **bounded** on $[a, b]$.

That is, there exists $M > 0$ such that $|f(x)| \le M$ for all $x \in [a, b]$.

> This is actually a consequence of (and weaker than) the Extreme Value Theorem.

---

## 3.3.4 Preservation of Intervals

### Statement

If $f$ is continuous and $I$ is an interval, then $f(I)$ (the image of $I$) is also an interval.

In other words, continuous functions **map intervals to intervals** — they don't "skip" values.

This is the foundation for the Intermediate Value Theorem (next chapter).

---

## 3.3.5 Preservation of Sign

If $f$ is continuous at $a$ and $f(a) > 0$, then there exists a neighborhood $(a - \delta, a + \delta)$ in which $f(x) > 0$.

```
  y
  │
  │         ╭──╮
  f(a) ├────│──│────  ← f stays positive near a
  │         │  │
  0 ├───────│──│──── x
  │         │  │
  └─────────┼──┼──── x
          a-δ  a+δ
```

Similarly, if $f(a) < 0$, then $f(x) < 0$ in some neighborhood of $a$.

---

## 3.3.6 Composition of Continuous Functions

**Theorem:** If $g$ is continuous at $a$ and $f$ is continuous at $g(a)$, then $f \circ g$ is continuous at $a$.

**Proof sketch:**
- Since $g$ is continuous at $a$: $x \to a \Rightarrow g(x) \to g(a)$
- Since $f$ is continuous at $g(a)$: $g(x) \to g(a) \Rightarrow f(g(x)) \to f(g(a))$
- Therefore $x \to a \Rightarrow (f \circ g)(x) \to (f \circ g)(a)$ $\blacksquare$

### Chain of Continuity

```
         g (continuous)      f (continuous)
  x ──────────▶ g(x) ──────────▶ f(g(x))
  ↓ close to a  ↓ close to g(a)  ↓ close to f(g(a))
  
  Result: f ∘ g is continuous
```

**Example:** $h(x) = \sin(x^2 + 1)$ is continuous on $\mathbb{R}$ because:
- $g(x) = x^2 + 1$ is continuous (polynomial)
- $f(u) = \sin u$ is continuous
- $h = f \circ g$ is continuous

---

## 3.3.7 Continuity of Inverse Functions

**Theorem:** If $f$ is continuous and strictly monotonic (always increasing or always decreasing) on an interval $I$, then:
1. $f^{-1}$ exists
2. $f^{-1}$ is also continuous on $f(I)$

**Example:** $f(x) = x^3$ is continuous and strictly increasing on $\mathbb{R}$, so $f^{-1}(x) = \sqrt[3]{x}$ is also continuous on $\mathbb{R}$.

---

## 3.3.8 Uniform Continuity (Advanced)

$f$ is **uniformly continuous** on $S$ if: for every $\varepsilon > 0$, $\exists\, \delta > 0$ such that for **all** $x, y \in S$:

$$|x - y| < \delta \implies |f(x) - f(y)| < \varepsilon$$

The key difference: $\delta$ depends only on $\varepsilon$, **not on the specific point**.

**Heine–Cantor Theorem:** Every function continuous on a **closed bounded interval** $[a, b]$ is uniformly continuous on $[a, b]$.

| Regular Continuity | Uniform Continuity |
|--------------------|--------------------|
| $\delta$ can depend on $\varepsilon$ and $a$ | $\delta$ depends only on $\varepsilon$ |
| Local property | Global property |
| $f(x) = 1/x$ on $(0,1)$: continuous | $f(x) = 1/x$ on $(0,1)$: NOT uniformly continuous |
| $f(x) = x^2$ on $\mathbb{R}$: continuous | $f(x) = x^2$ on $\mathbb{R}$: NOT uniformly continuous |

---

## Summary Table

| Property/Theorem | Statement |
|-----------------|-----------|
| Extreme Value Theorem | Continuous on $[a,b]$ ⟹ attains max and min |
| Boundedness | Continuous on $[a,b]$ ⟹ bounded |
| Preservation of intervals | Continuous image of an interval is an interval |
| Sign preservation | $f$ continuous, $f(a) > 0$ ⟹ $f > 0$ near $a$ |
| Composition | $g$ cont. at $a$, $f$ cont. at $g(a)$ ⟹ $f \circ g$ cont. at $a$ |
| Inverse | Cont. + strictly monotonic ⟹ inverse is continuous |
| Uniform continuity | On $[a,b]$, continuous ⟹ uniformly continuous |

---

## Quick Revision Questions

**Q1.** State the Extreme Value Theorem. Why is the "closed interval" condition necessary?

**Q2.** Is $f(x) = \frac{1}{x}$ bounded on $(0, 1)$? Does this contradict the Boundedness Theorem?

**Q3.** If $f$ and $g$ are continuous at $a$ but $g(a) = 0$, what can we say about $f/g$?

**Q4.** Prove that $h(x) = e^{\cos x}$ is continuous on $\mathbb{R}$.

**Q5.** Explain the difference between continuity and uniform continuity in one sentence.

**Q6.** Give an example where $f$ is continuous on an open interval but does not attain its maximum.

---

[← Previous: Types of Discontinuity](02-types-of-discontinuity.md) · [Next: Intermediate Value Theorem →](04-intermediate-value-theorem.md)
