# Chapter 4: Taylor's Theorem

[← Previous: L'Hôpital's Rule](03-lhopitals-rule.md) | [Back to README](../README.md) | [Next: Higher-Order Derivatives →](05-higher-order-derivatives.md)

---

## Overview

**Taylor's Theorem** approximates differentiable functions by polynomials and gives precise control of the error (remainder). It connects local derivative information to global function behavior and is the theoretical foundation of power series representations.

---

## 4.1 Taylor Polynomials

**Definition 4.1.** If $f$ is $n$-times differentiable at $c$, the **$n$-th Taylor polynomial** centered at $c$ is:

$$P_n(x) = \sum_{k=0}^{n} \frac{f^{(k)}(c)}{k!}(x - c)^k = f(c) + f'(c)(x-c) + \frac{f''(c)}{2!}(x-c)^2 + \cdots + \frac{f^{(n)}(c)}{n!}(x-c)^n$$

**Key property:** $P_n$ is the unique polynomial of degree $\leq n$ satisfying $P_n^{(k)}(c) = f^{(k)}(c)$ for $k = 0, 1, \ldots, n$.

```
  Taylor polynomials for e^x at c = 0:

  P₀(x) = 1
  P₁(x) = 1 + x
  P₂(x) = 1 + x + x²/2
  P₃(x) = 1 + x + x²/2 + x³/6

  y ▲
    │              ╱ eˣ
  3 │           ╱╱  P₃
    │         ╱╱╱
  2 │      ╱╱╱╱   P₂
    │    ╱╱╱╱
  1 │●╱╱╱╱╱ P₁ (tangent line)
    │╱╱
    └───────────────► x

  Each Pₙ matches f more closely near c = 0
```

---

## 4.2 Taylor's Theorem (Lagrange Remainder)

**Theorem 4.2 (Taylor, Lagrange form).** If $f$ is $(n+1)$-times differentiable on an interval containing $c$ and $x$, then:

$$f(x) = P_n(x) + R_n(x)$$

where the **Lagrange remainder** is:

$$R_n(x) = \frac{f^{(n+1)}(\xi)}{(n+1)!}(x - c)^{n+1}$$

for some $\xi$ between $c$ and $x$.

**Proof.** Fix $x$ and define:

$$F(t) = f(x) - \sum_{k=0}^{n} \frac{f^{(k)}(t)}{k!}(x-t)^k - M(x-t)^{n+1}$$

where $M$ is chosen so that $F(c) = 0$ (i.e., $M = R_n(x)/(x-c)^{n+1}$). Also $F(x) = 0$.

By Rolle's Theorem applied to $F$ on $[c, x]$ (or $[x, c]$): $\exists\, \xi$ with $F'(\xi) = 0$.

Computing $F'(t)$ (most terms cancel telescopically):

$$F'(t) = -\frac{f^{(n+1)}(t)}{n!}(x-t)^n + M(n+1)(x-t)^n = 0$$

So $M = \frac{f^{(n+1)}(\xi)}{(n+1)!}$. $\blacksquare$

---

## 4.3 Other Remainder Forms

| Form | Remainder $R_n(x)$ | Use case |
|------|--------------------:|---------|
| **Lagrange** | $\frac{f^{(n+1)}(\xi)}{(n+1)!}(x-c)^{n+1}$ | Error bounds |
| **Cauchy** | $\frac{f^{(n+1)}(\xi)}{n!}(x-\xi)^n(x-c)$ | Theoretical proofs |
| **Integral** | $\frac{1}{n!}\int_c^x f^{(n+1)}(t)(x-t)^n\,dt$ | Most precise; works with $f^{(n+1)} \in L^1$ |

The integral form is the most general and exact.

---

## 4.4 Applications of Taylor's Theorem

**Application 1: Error estimation.** For $e^x$ at $c = 0$ with $n$ terms:

$$|R_n(x)| \leq \frac{e^{|x|}}{(n+1)!}|x|^{n+1}$$

Since $(n+1)!$ grows faster than $|x|^{n+1}$, $R_n(x) \to 0$ for all $x$. So:

$$e^x = \sum_{n=0}^{\infty} \frac{x^n}{n!} \quad \forall\, x \in \mathbb{R}$$

**Application 2: Proving inequalities.**
- $e^x \geq 1 + x$ (use $P_1$ and $R_1 \geq 0$)
- $\sin x \leq x$ for $x \geq 0$ (use Taylor with remainder)
- $\ln(1+x) \leq x$ for $x > -1$

**Application 3: Convergence of series.** A function equals its Taylor series iff $R_n \to 0$.

```
  Taylor series convergence:

  f(x) = Σ f⁽ⁿ⁾(c)/n! (x-c)ⁿ  ⟺  Rₙ(x) → 0 as n → ∞
         n=0

  ┌──────────────────────────────────────┐
  │ Not all smooth functions equal their │
  │ Taylor series!                       │
  │                                      │
  │ f(x) = e^(-1/x²) (x≠0), f(0) = 0   │
  │ has f⁽ⁿ⁾(0) = 0 for all n          │
  │ Taylor series ≡ 0, but f ≢ 0        │
  └──────────────────────────────────────┘
```

---

## 4.5 Common Taylor Series

| Function | Taylor series (at $c = 0$) | Radius |
|----------|---------------------------|:------:|
| $e^x$ | $\sum \frac{x^n}{n!}$ | $\infty$ |
| $\sin x$ | $\sum \frac{(-1)^n x^{2n+1}}{(2n+1)!}$ | $\infty$ |
| $\cos x$ | $\sum \frac{(-1)^n x^{2n}}{(2n)!}$ | $\infty$ |
| $\frac{1}{1-x}$ | $\sum x^n$ | $1$ |
| $\ln(1+x)$ | $\sum \frac{(-1)^{n+1} x^n}{n}$ | $1$ |
| $(1+x)^\alpha$ | $\sum \binom{\alpha}{n} x^n$ | $1$ |

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| Taylor polynomial $P_n$ | Best degree-$n$ polynomial matching $f$ at $c$ (up to $n$-th derivative) |
| Taylor's Theorem | $f(x) = P_n(x) + R_n(x)$ |
| Lagrange remainder | $R_n = \frac{f^{(n+1)}(\xi)}{(n+1)!}(x-c)^{n+1}$ |
| Series convergence | $f(x) = \sum \frac{f^{(n)}(c)}{n!}(x-c)^n$ iff $R_n \to 0$ |
| Smooth $\neq$ analytic | $e^{-1/x^2}$ is smooth but not analytic at $0$ |

---

## Quick Revision Questions

1. **State** Taylor's Theorem with the Lagrange remainder.

2. **Prove** Taylor's Theorem using Rolle's Theorem.

3. **Use** Taylor's Theorem to show $e^x \geq 1 + x$ for all $x$.

4. **Show** $e^x = \sum x^n/n!$ converges for all $x$ by bounding $R_n$.

5. **Give** an example of a $C^\infty$ function not equal to its Taylor series.

6. **Derive** the Taylor polynomial of degree 4 for $\cos x$ at $c = 0$ and estimate $\cos(0.1)$.

---

[← Previous: L'Hôpital's Rule](03-lhopitals-rule.md) | [Back to README](../README.md) | [Next: Higher-Order Derivatives →](05-higher-order-derivatives.md)
