# Unit 2 — Limits: Concept of Limit

[← Back to README](../README.md) · [Previous: Unit 1 — Functions](../01-Functions/01-functions.md) · [Next: Left and Right Hand Limits →](02-left-right-limits.md)

---

## Chapter Overview

The **limit** is the single most important idea in calculus. It captures the notion of *approaching* a value — what happens to $f(x)$ as $x$ gets **arbitrarily close** to some number $a$, regardless of whether $f(a)$ itself is defined. Without limits, we cannot define derivatives, integrals, or series rigorously.

---

## 2.1 Intuitive Idea of a Limit

Consider $f(x) = \frac{x^2 - 1}{x - 1}$ at $x = 1$.

- Direct substitution: $f(1) = \frac{0}{0}$ — **undefined**.
- But simplify: $f(x) = \frac{(x-1)(x+1)}{x-1} = x + 1$ for $x \ne 1$.

Let's see what happens as $x$ approaches 1:

| $x$ | 0.9 | 0.99 | 0.999 | → 1 ← | 1.001 | 1.01 | 1.1 |
|-----|-----|------|-------|--------|-------|------|-----|
| $f(x)$ | 1.9 | 1.99 | 1.999 | ? | 2.001 | 2.01 | 2.1 |

As $x \to 1$, $f(x) \to 2$. We write:

$$\lim_{x \to 1} \frac{x^2 - 1}{x - 1} = 2$$

**Graphical view:**

```
  y
  │
  3├
  │
  2├ · · · · ·○· · · · ·   ← open circle: f(1) is undefined
  │         ╱
  │        ╱
  1├      ╱
  │     ╱
  │    ╱
  │   ╱
  └──┼──┼──┼──┼──── x
     0  0.5 1  1.5
```

The open circle at $(1, 2)$ shows $f(1)$ is not defined, but the **limit** is still 2.

---

## 2.2 Formal (ε-δ) Definition

### Statement

$$\lim_{x \to a} f(x) = L$$

means: for every $\varepsilon > 0$ (no matter how small), there exists a $\delta > 0$ such that:

$$0 < |x - a| < \delta \implies |f(x) - L| < \varepsilon$$

### Visualisation

```
  y
  │
  L+ε ├─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    ← upper ε-band
  │          ╱╱╱╱╱╱╱╱╱
  L   ├─ ─ ─╱─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    ← target value
  │       ╱╱╱╱╱╱╱╱
  L-ε ├─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    ← lower ε-band
  │
  └───┼───┼───┼───┼───┼──── x
      a-δ     a     a+δ
       └──────┘└──────┘
        δ-neighbourhood
        (excluding a itself)
```

**In words:** We can make $f(x)$ as close to $L$ as we want (within $\varepsilon$) by making $x$ sufficiently close to $a$ (within $\delta$), but $x \ne a$.

### Example: Prove $\lim_{x \to 3} (2x + 1) = 7$

We need: given $\varepsilon > 0$, find $\delta > 0$ such that $0 < |x - 3| < \delta \Rightarrow |(2x+1) - 7| < \varepsilon$.

$$|(2x+1) - 7| = |2x - 6| = 2|x - 3|$$

We need $2|x - 3| < \varepsilon$, i.e., $|x - 3| < \frac{\varepsilon}{2}$.

Choose $\delta = \frac{\varepsilon}{2}$. Then:

$$0 < |x - 3| < \delta \Rightarrow |(2x+1) - 7| = 2|x-3| < 2 \cdot \frac{\varepsilon}{2} = \varepsilon \quad \blacksquare$$

---

## 2.3 When Does a Limit Exist?

$$\lim_{x \to a} f(x) = L \quad \text{exists if and only if} \quad \lim_{x \to a^-} f(x) = \lim_{x \to a^+} f(x) = L$$

Both the **left-hand limit** and the **right-hand limit** must exist and be **equal**.

### Limit Does NOT Exist — Three Common Cases

**Case 1: Left and right limits differ**

```
  y
  │
  3├        ●  ← f(2) = 3
  │         │
  2├────────○  ← left-hand limit = 2
  │              ────── ← right-hand limit = 4
  4├         ●
  │
  └─────────┼───── x
            2
  lim does NOT exist
```

**Case 2: Function oscillates**

$f(x) = \sin\left(\frac{1}{x}\right)$ near $x = 0$ oscillates infinitely — no single value is approached.

**Case 3: Function grows without bound**

$f(x) = \frac{1}{x^2}$ as $x \to 0$: $f(x) \to +\infty$. We say the limit **does not exist** (or is infinite).

---

## 2.4 Limits Involving Infinity

### $\lim_{x \to a} f(x) = \infty$

The function grows without bound near $x = a$ (vertical asymptote).

$$\lim_{x \to 0} \frac{1}{x^2} = +\infty$$

### $\lim_{x \to \infty} f(x) = L$

The function approaches $L$ as $x$ grows without bound (horizontal asymptote).

$$\lim_{x \to \infty} \frac{1}{x} = 0$$

### $\lim_{x \to \infty} f(x) = \infty$

Both input and output grow without bound.

$$\lim_{x \to \infty} x^2 = +\infty$$

---

## 2.5 Real-World Application

**Velocity as a limit:**

Average velocity over time interval $[t, t + h]$:

$$v_{\text{avg}} = \frac{s(t+h) - s(t)}{h}$$

Instantaneous velocity = limit as $h \to 0$:

$$v(t) = \lim_{h \to 0} \frac{s(t+h) - s(t)}{h}$$

This is exactly the **derivative** — limits are the bridge from algebra to calculus.

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| $\lim_{x \to a} f(x) = L$ | $f(x)$ approaches $L$ as $x$ approaches $a$ |
| ε-δ definition | Formal: $0 < \|x - a\| < \delta \Rightarrow \|f(x) - L\| < \varepsilon$ |
| Limit exists | Left-hand limit = Right-hand limit |
| Limit DNE cases | Unequal one-sided limits, oscillation, unbounded |
| Limit ≠ function value | $\lim_{x\to a} f(x)$ can exist even if $f(a)$ is undefined |
| Limits at infinity | Horizontal asymptotes: $\lim_{x\to\infty} f(x) = L$ |
| Infinite limits | Vertical asymptotes: $\lim_{x\to a} f(x) = \pm\infty$ |

---

## Quick Revision Questions

**Q1.** What is $\lim_{x \to 2} \frac{x^2 - 4}{x - 2}$? Simplify first.

**Q2.** State the ε-δ definition of a limit in your own words.

**Q3.** Does $\lim_{x \to 0} \sin\left(\frac{1}{x}\right)$ exist? Why or why not?

**Q4.** Give an example where $\lim_{x \to a} f(x)$ exists but $f(a)$ is undefined.

**Q5.** What is $\lim_{x \to \infty} \frac{3x + 1}{x - 2}$? How would you evaluate this?

**Q6.** Prove using ε-δ that $\lim_{x \to 1} (5x - 3) = 2$.

---

[← Previous: Unit 1 — Functions](../01-Functions/01-functions.md) · [Next: Left and Right Hand Limits →](02-left-right-limits.md)
