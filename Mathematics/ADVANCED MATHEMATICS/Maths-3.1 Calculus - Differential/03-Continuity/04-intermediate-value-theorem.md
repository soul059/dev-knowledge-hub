# Unit 3 — Continuity: Intermediate Value Theorem

[← Previous: Properties of Continuous Functions](03-properties-of-continuous-functions.md) · [Next: Unit 4 — Differentiability →](../04-Differentiability/01-definition-first-principles.md)

---

## Chapter Overview

The **Intermediate Value Theorem (IVT)** is one of the most intuitive yet powerful theorems in calculus. It says that a continuous function cannot "jump over" any value — if it takes the value $A$ and later the value $B$, it must take every value between $A$ and $B$ at some point.

---

## 3.4.1 Statement of the Theorem

If $f$ is **continuous on** $[a, b]$ and $N$ is any number between $f(a)$ and $f(b)$ (i.e., $f(a) < N < f(b)$ or $f(b) < N < f(a)$), then there exists at least one $c \in (a, b)$ such that:

$$\boxed{f(c) = N}$$

### Graphical Interpretation

```
  y
  │
  f(b)├─ ─ ─ ─ ─ ─ ─ ─ ─●
  │                     ╱
  │                   ╱
  N ├─ ─ ─ ─ ─ ─●─ ─      ← must cross y = N at some c
  │            ╱
  │          ╱
  f(a)├────●
  │
  └────┼───┼───────┼──── x
       a   c       b
```

**The key insight:** A continuous curve going from $(a, f(a))$ to $(b, f(b))$ must cross every horizontal line between $f(a)$ and $f(b)$.

---

## 3.4.2 Special Case: Bolzano's Theorem (Root-Finding)

If $f$ is continuous on $[a, b]$ and $f(a)$ and $f(b)$ have **opposite signs** (one positive, one negative), then there exists $c \in (a, b)$ such that:

$$f(c) = 0$$

(Here $N = 0$.)

### Graphical View

```
  y
  │
  f(a)├──●
  │      ╲
  │       ╲
  0 ├──────╲────────────── x-axis
  │         ╲         ╱
  │          ╲       ╱
  f(b)├───────╲─●──╱        ← there must be a root
  │            ╲╱
  └───┼────┼───┼──┼──── x
      a    c₁     b
           ↑
         f(c₁) = 0
```

---

## 3.4.3 Proving Existence of Roots — Worked Examples

### Example 1: Show $x^3 - x - 1 = 0$ has a root in $[1, 2]$

Let $f(x) = x^3 - x - 1$.

- $f(1) = 1 - 1 - 1 = -1 < 0$
- $f(2) = 8 - 2 - 1 = 5 > 0$
- $f$ is a polynomial → continuous on $[1, 2]$

By IVT (Bolzano's Theorem), $\exists\, c \in (1, 2)$ where $f(c) = 0$.

### Example 2: Show $e^x = 3 - x$ has a solution

Rewrite as $f(x) = e^x - 3 + x = 0$.

- $f(0) = 1 - 3 + 0 = -2 < 0$
- $f(1) = e - 3 + 1 = e - 2 \approx 0.718 > 0$

By IVT, $\exists\, c \in (0, 1)$ where $e^c = 3 - c$.

### Example 3: Show $\cos x = x$ has a solution in $[0, \pi/2]$

Let $f(x) = \cos x - x$.

- $f(0) = 1 - 0 = 1 > 0$
- $f(\pi/2) = 0 - \pi/2 \approx -1.57 < 0$

By IVT, there exists a root. (It's approximately $x \approx 0.739$, called the **Dottie number**.)

---

## 3.4.4 The Bisection Method (Numerical Root-Finding)

IVT gives us the **bisection method** — a systematic way to narrow down where a root is.

### Algorithm

```
Given: f continuous on [a, b], f(a) · f(b) < 0

1. Compute midpoint: m = (a + b) / 2
2. Evaluate f(m)
3. If f(m) = 0 → root found!
4. If f(a) · f(m) < 0 → root is in [a, m], set b = m
5. If f(m) · f(b) < 0 → root is in [m, b], set a = m
6. Repeat until |b - a| < desired tolerance
```

### Example: Find root of $x^3 - x - 1 = 0$ in $[1, 2]$

| Iteration | $a$ | $b$ | $m = \frac{a+b}{2}$ | $f(m)$ | New interval |
|-----------|------|------|------|--------|--------------|
| 1 | 1.0 | 2.0 | 1.5 | $3.375 - 1.5 - 1 = 0.875$ | $[1.0, 1.5]$ |
| 2 | 1.0 | 1.5 | 1.25 | $1.953 - 1.25 - 1 = -0.297$ | $[1.25, 1.5]$ |
| 3 | 1.25 | 1.5 | 1.375 | $2.600 - 1.375 - 1 = 0.224$ | $[1.25, 1.375]$ |
| 4 | 1.25 | 1.375 | 1.3125 | $2.260 - 1.3125 - 1 = -0.053$ | $[1.3125, 1.375]$ |

Root $\approx 1.3247$ (exact root is the real root of $x^3 - x - 1$).

Each iteration halves the interval, so after $n$ iterations, the error $\le \frac{b - a}{2^n}$.

---

## 3.4.5 Other Applications of IVT

### Fixed Point Theorem

If $f : [0, 1] \to [0, 1]$ is continuous, then $f$ has a **fixed point** — a value $c$ where $f(c) = c$.

**Proof:** Let $g(x) = f(x) - x$.
- $g(0) = f(0) - 0 = f(0) \ge 0$ (since $f(0) \in [0, 1]$)
- $g(1) = f(1) - 1 \le 0$ (since $f(1) \in [0, 1]$)
- By IVT, $\exists\, c$ where $g(c) = 0$, i.e., $f(c) = c$.

```
  y
  1├──────────────●
  │             ╱ │
  │           ╱   │
  │    f(x) ╱     │
  │       ╱  ╱y=x │
  │     ●╱ ╱      │   ← intersection = fixed point
  │    ╱ ╱        │
  │  ╱ ╱          │
  │╱ ╱            │
  └──┼────────┼───┼── x
     0   c    1
```

### Temperature Example

At any moment, there exist two diametrically opposite points on the Earth's equator with the **same temperature**.

**Proof sketch:** Let $T(\theta)$ be the temperature at angle $\theta$ on the equator. Define $g(\theta) = T(\theta) - T(\theta + \pi)$.

- $g(0) = T(0) - T(\pi)$
- $g(\pi) = T(\pi) - T(2\pi) = T(\pi) - T(0) = -g(0)$

If $g(0) \ne 0$, then $g(0)$ and $g(\pi)$ have opposite signs. By IVT, $\exists\, \theta_0$ where $g(\theta_0) = 0$, i.e., $T(\theta_0) = T(\theta_0 + \pi)$.

---

## 3.4.6 Conditions Are Necessary

| Missing Condition | Counterexample |
|-------------------|---------------|
| Not continuous | $f(x) = \begin{cases} -1 & x < 0 \\ 1 & x \ge 0 \end{cases}$; $f(-1) = -1$, $f(1) = 1$, but $f(x) = 0$ has no solution |
| Not a closed interval | $f(x) = \frac{1}{x}$ on $(0, 1]$; continuous there but never achieves values $>$ any bound |

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| IVT | If $f$ continuous on $[a,b]$, then $f$ takes every value between $f(a)$ and $f(b)$ |
| Bolzano's Theorem | $f(a) \cdot f(b) < 0$ ⟹ root exists in $(a, b)$ |
| Bisection Method | Repeatedly halve the interval to locate the root |
| Fixed Point Theorem | $f : [0,1] \to [0,1]$ continuous ⟹ $\exists\, c$ with $f(c) = c$ |
| Necessary conditions | $f$ must be continuous; interval must be closed |
| Error in bisection | After $n$ steps: $\le \frac{b-a}{2^n}$ |

---

## Quick Revision Questions

**Q1.** State the Intermediate Value Theorem precisely.

**Q2.** Show that $x^5 + x - 1 = 0$ has at least one root in $[0, 1]$.

**Q3.** Using the bisection method, find the first 3 iterations for the root of $x^2 - 2 = 0$ in $[1, 2]$.

**Q4.** Why does IVT not guarantee a root for $f(x) = x^2 + 1$ on $[-1, 1]$?

**Q5.** Explain the fixed point theorem using IVT.

**Q6.** Can a discontinuous function still satisfy the IVT conclusion? Give an example or counter-argument.

---

[← Previous: Properties of Continuous Functions](03-properties-of-continuous-functions.md) · [Next: Unit 4 — Differentiability →](../04-Differentiability/01-definition-first-principles.md)
