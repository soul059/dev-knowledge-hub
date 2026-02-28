# Chapter 5: Fundamental Theorem of Calculus

[← Previous: Properties of the Integral](04-properties-of-integral.md) | [Back to README](../README.md) | [Next: Improper Integrals →](06-improper-integrals.md)

---

## Overview

The **Fundamental Theorem of Calculus** (FTC) connects differentiation and integration — the two pillars of calculus. Part I shows that integration followed by differentiation recovers the original function; Part II shows that differentiation followed by integration (antidifferentiation) computes definite integrals.

---

## 5.1 FTC Part I (Differentiation of the Integral)

**Theorem 5.1 (FTC I).** Let $f \in \mathcal{R}[a,b]$. Define:

$$F(x) = \int_a^x f(t)\,dt, \quad x \in [a,b]$$

Then:
1. $F$ is **continuous** on $[a,b]$.
2. If $f$ is **continuous at** $c \in (a,b)$, then $F$ is **differentiable at** $c$ and $F'(c) = f(c)$.

**Proof of (1).** For $x, y \in [a,b]$ with $x < y$:

$$|F(y) - F(x)| = \left|\int_x^y f\right| \leq \int_x^y |f| \leq M(y - x)$$

where $M = \sup|f|$. So $F$ is Lipschitz, hence continuous. $\blacksquare$

**Proof of (2).** For $h \neq 0$ small:

$$\frac{F(c+h) - F(c)}{h} - f(c) = \frac{1}{h}\int_c^{c+h}[f(t) - f(c)]\,dt$$

Since $f$ continuous at $c$: $\forall \varepsilon > 0$, $\exists \delta$: $|t - c| < \delta \Rightarrow |f(t) - f(c)| < \varepsilon$.

For $|h| < \delta$:

$$\left|\frac{F(c+h) - F(c)}{h} - f(c)\right| \leq \frac{1}{|h|}\int_c^{c+h} |f(t) - f(c)|\,dt < \varepsilon$$

So the limit is $f(c)$, i.e., $F'(c) = f(c)$. $\blacksquare$

```
  FTC Part I — Geometric Picture:

  f(t)
   │      ╱╲
   │    ╱    ╲
   │  ╱        ╲      f continuous at c
   │╱            ╲       ⟹ F'(c) = f(c)
   └───────┬──────────── t
           c

  F(x) = ∫ₐˣ f(t)dt = area from a to x

  F(x)                Rate of area
   │        ╱          accumulation at c
   │      ╱            = height of f at c
   │    ╱
   │  ╱
   └──────────────── x
   a    c    b
```

---

## 5.2 FTC Part II (Evaluation of the Integral)

**Theorem 5.2 (FTC II).** Let $f \in \mathcal{R}[a,b]$ and suppose $\exists\, G$ differentiable on $[a,b]$ with $G'(x) = f(x)$ for all $x \in [a,b]$. Then:

$$\int_a^b f = G(b) - G(a)$$

**Proof.** Let $P = \{x_0, \ldots, x_n\}$ be any partition. By MVT on each $[x_{k-1}, x_k]$, $\exists\, t_k$:

$$G(x_k) - G(x_{k-1}) = G'(t_k)\Delta x_k = f(t_k)\Delta x_k$$

Summing (telescoping):

$$G(b) - G(a) = \sum_{k=1}^n f(t_k)\Delta x_k$$

This is a Riemann sum for $f$. Also, $m_k \Delta x_k \leq f(t_k)\Delta x_k \leq M_k \Delta x_k$, so:

$$L(f, P) \leq G(b) - G(a) \leq U(f, P)$$

for all $P$. Since $f$ is integrable, both sides squeeze to $\int_a^b f$. $\blacksquare$

**Notation.** $G(b) - G(a) = G(x)\Big|_a^b = [G(x)]_a^b$.

---

## 5.3 Relationship Between FTC I and II

```
  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │  FTC I:  Integration → Differentiation           │
  │          d/dx ∫ₐˣ f(t)dt = f(x)                 │
  │          (if f continuous at x)                  │
  │                                                  │
  │  FTC II: Differentiation → Integration           │
  │          ∫ₐᵇ G'(x)dx = G(b) - G(a)             │
  │          (if G' integrable)                      │
  │                                                  │
  │  Together: Integration ⊣⊢ Differentiation        │
  │           (inverse operations, with caveats)     │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

**Key differences:**
- FTC I requires $f$ **continuous** (at the point) to get differentiability.
- FTC II requires $G'$ to **exist everywhere** and be **integrable**.
- Not every integrable function has an antiderivative.
- Not every derivative is Riemann integrable (though derivatives are always Darboux continuous).

---

## 5.4 Consequences

**Corollary 5.4.1 (Existence of Antiderivatives).** Every continuous function has an antiderivative: $F(x) = \int_a^x f(t)\,dt$.

**Corollary 5.4.2 (Integration by Parts).** If $f, g$ differentiable with $f', g' \in \mathcal{R}[a,b]$:

$$\int_a^b fg' = [fg]_a^b - \int_a^b f'g$$

**Proof.** $(fg)' = f'g + fg'$ is integrable. Apply FTC II to $(fg)' $. $\blacksquare$

**Corollary 5.4.3 (Change of Variables / Substitution).** If $\varphi: [\alpha, \beta] \to [a, b]$ is $C^1$ and $f \in C[a,b]$:

$$\int_{\varphi(\alpha)}^{\varphi(\beta)} f(x)\,dx = \int_\alpha^\beta f(\varphi(t))\,\varphi'(t)\,dt$$

---

## 5.5 Worked Examples

**Example 1.** $\frac{d}{dx}\int_0^x e^{-t^2}\,dt = e^{-x^2}$ by FTC I (since $e^{-t^2}$ is continuous).

**Example 2.** $\frac{d}{dx}\int_0^{x^2} \sin(t)\,dt$. Let $F(u) = \int_0^u \sin t\,dt$. Then $\frac{d}{dx}F(x^2) = F'(x^2)\cdot 2x = \sin(x^2)\cdot 2x$ by chain rule + FTC I.

**Example 3.** $\frac{d}{dx}\int_{x}^{x^2} e^t\,dt = e^{x^2}\cdot 2x - e^x$ (Leibniz rule).

**Example 4.** $\int_0^1 x^3\,dx = \frac{x^4}{4}\Big|_0^1 = \frac{1}{4}$ by FTC II since $G(x) = x^4/4$ has $G'(x) = x^3$.

---

## 5.6 Leibniz Integral Rule

**Theorem 5.6.** If $a(x), b(x)$ are differentiable and $f$ is continuous:

$$\frac{d}{dx}\int_{a(x)}^{b(x)} f(t)\,dt = f(b(x))\,b'(x) - f(a(x))\,a'(x)$$

---

## 5.7 When FTC Fails

| Issue | Example | Problem |
|-------|---------|---------|
| $f$ not continuous | $f = \mathbf{1}_{\mathbb{Q}}$ | $F(x) = \int_0^x f = 0$ is differentiable, but $F' = 0 \neq f$ |
| $G'$ not integrable | Volterra's function | $G'$ exists everywhere but is not Riemann integrable |
| $G'$ exists a.e. only | $G(x) = x^2\sin(1/x^2)$ | $G'$ unbounded, improper integral needed |

---

## Summary Table

| Theorem | Hypothesis | Conclusion |
|---------|-----------|------------|
| FTC I | $f \in \mathcal{R}$, $F(x) = \int_a^x f$ | $F$ continuous; $f$ cont. at $c \Rightarrow F'(c) = f(c)$ |
| FTC II | $G' = f$ on $[a,b]$, $f \in \mathcal{R}$ | $\int_a^b f = G(b) - G(a)$ |
| Integration by parts | $f, g$ diff., $f', g' \in \mathcal{R}$ | $\int fg' = [fg] - \int f'g$ |
| Substitution | $\varphi \in C^1$, $f \in C$ | $\int f(x)dx = \int f(\varphi(t))\varphi'(t)dt$ |
| Leibniz rule | $f$ cont., $a(x), b(x)$ diff. | $\frac{d}{dx}\int_{a(x)}^{b(x)}f = f(b)b' - f(a)a'$ |

---

## Quick Revision Questions

1. **State and prove** FTC Part I. Where do you use continuity of $f$?

2. **State and prove** FTC Part II. Where do you use MVT?

3. **Compute** $\frac{d}{dx}\int_1^{x^3} \frac{\sin t}{t}\,dt$.

4. **Derive** integration by parts from FTC II.

5. **Give** an example where $F(x) = \int_0^x f$ is differentiable but $F'(x) \neq f(x)$ at some point.

6. **Explain** why FTC I does not require $f$ to be continuous everywhere.

---

[← Previous: Properties of the Integral](04-properties-of-integral.md) | [Back to README](../README.md) | [Next: Improper Integrals →](06-improper-integrals.md)
