# Chapter 3: Weierstrass M-Test

[← Previous: Uniform Convergence](02-uniform-convergence.md) | [Back to README](../README.md) | [Next: Integration and Differentiation of Series →](04-integration-and-differentiation-of-series.md)

---

## Overview

The **Weierstrass M-test** is the most widely used tool for establishing **uniform convergence of series of functions**. It works by bounding each term by a constant and reducing to convergence of a numerical series.

---

## 3.1 Statement and Proof

**Theorem 3.1 (Weierstrass M-Test).** Let $g_n: A \to \mathbb{R}$ and suppose $\exists\, M_n \geq 0$ with:

1. $|g_n(x)| \leq M_n$ for all $x \in A$, all $n$
2. $\sum_{n=1}^\infty M_n$ converges

Then $\sum_{n=1}^\infty g_n(x)$ converges **uniformly** (and absolutely) on $A$.

**Proof.** Let $S_N(x) = \sum_{n=1}^N g_n(x)$. For $m > n$:

$$|S_m(x) - S_n(x)| = \left|\sum_{k=n+1}^m g_k(x)\right| \leq \sum_{k=n+1}^m |g_k(x)| \leq \sum_{k=n+1}^m M_k$$

Since $\sum M_k$ converges, the tail $\sum_{k=n+1}^m M_k \to 0$ as $n, m \to \infty$, independently of $x$.

So $(S_N)$ is uniformly Cauchy $\Rightarrow$ uniformly convergent. $\blacksquare$

```
  Weierstrass M-Test — Bounding strategy:

  |g₁(x)| ≤ M₁     |g₂(x)| ≤ M₂     |g₃(x)| ≤ M₃    ...
     │                  │                  │
     ▼                  ▼                  ▼
  ∑ Mₙ converges  (numerical series)
     │
     ▼
  ∑ gₙ(x) converges uniformly and absolutely
```

---

## 3.2 How to Apply

**Step 1.** Find $M_n = \sup_{x \in A}|g_n(x)|$ (or any upper bound).

**Step 2.** Check whether $\sum M_n < \infty$.

**Step 3.** If yes, conclude uniform + absolute convergence on $A$.

---

## 3.3 Examples

**Example 1.** $\sum_{n=1}^\infty \frac{\sin(nx)}{n^2}$ on $\mathbb{R}$.

$$\left|\frac{\sin(nx)}{n^2}\right| \leq \frac{1}{n^2} = M_n, \qquad \sum \frac{1}{n^2} = \frac{\pi^2}{6} < \infty$$

Uniform convergence on $\mathbb{R}$. ✓

**Example 2.** $\sum_{n=0}^\infty x^n$ on $[-r, r]$ for $0 < r < 1$.

$$|x^n| \leq r^n = M_n, \qquad \sum r^n = \frac{1}{1-r} < \infty$$

Uniform on $[-r, r]$. ✓ (But NOT uniform on $(-1, 1)$ since $\sup |x^n| = 1$.)

**Example 3.** $\sum_{n=1}^\infty \frac{x^n}{n!}$ on $[-R, R]$ for any $R > 0$.

$$\left|\frac{x^n}{n!}\right| \leq \frac{R^n}{n!} = M_n, \qquad \sum \frac{R^n}{n!} = e^R < \infty$$

Uniform on every $[-R, R]$. ✓ (This series defines $e^x$.)

**Example 4.** $\sum_{n=1}^\infty \frac{(-1)^n}{n + x^2}$ on $\mathbb{R}$.

$M_n = 1/n$, $\sum 1/n = \infty$. M-test FAILS. (But the series converges pointwise by AST.) The M-test is sufficient, not necessary.

---

## 3.4 M-Test Limitations

The M-test establishes **absolute** convergence. It **cannot** detect:
- Conditionally convergent series (like $\sum (-1)^n / (n + x^2)$)
- Series that converge uniformly but not absolutely

For such cases, use **Dirichlet's test** or **Abel's test** for series of functions.

---

## 3.5 Abel and Dirichlet Tests for Series

**Theorem 3.5 (Dirichlet).** $\sum a_n(x) b_n(x)$ converges uniformly if:
1. Partial sums $\sum_{k=1}^N a_k(x)$ are **uniformly bounded**
2. $b_n(x) \to 0$ **uniformly** and $b_n$ is monotone in $n$ for each $x$

**Theorem 3.5' (Abel).** $\sum a_n(x) b_n(x)$ converges uniformly if:
1. $\sum a_n(x)$ converges **uniformly**
2. $(b_n(x))$ is **uniformly bounded** and monotone in $n$ for each $x$

---

## 3.6 Weierstrass Approximation Theorem (Preview)

**Theorem 3.6 (Weierstrass).** Every continuous function $f: [a,b] \to \mathbb{R}$ can be uniformly approximated by polynomials:

$$\forall\, \varepsilon > 0,\; \exists\, \text{polynomial } p: \|f - p\|_\infty < \varepsilon$$

This is a deep result showing that polynomials are **dense** in $C[a,b]$.

---

## Summary Table

| Tool | Hypothesis | Conclusion |
|------|-----------|------------|
| Weierstrass M-test | $|g_n(x)| \leq M_n$, $\sum M_n < \infty$ | $\sum g_n$ unif. + abs. convergent |
| Dirichlet (functions) | Partial sums bdd, $b_n \downarrow 0$ uniformly | $\sum a_n b_n$ unif. convergent |
| Abel (functions) | $\sum a_n$ unif., $b_n$ bdd + monotone | $\sum a_n b_n$ unif. convergent |
| M-test limitation | Cannot detect conditional convergence | Use Dirichlet/Abel instead |

---

## Quick Revision Questions

1. **State and prove** the Weierstrass M-test.

2. **Show** $\sum \frac{\cos(nx)}{n^2}$ converges uniformly on $\mathbb{R}$.

3. **Explain** why $\sum x^n$ converges uniformly on $[-r, r]$ ($r < 1$) but NOT on $(-1, 1)$.

4. **Give** an example where the M-test fails but the series converges uniformly (use Dirichlet).

5. **Apply** the M-test to $\sum \frac{x^n}{n!}$ on $[-R, R]$.

6. **State** the Weierstrass Approximation Theorem. Why is it remarkable?

---

[← Previous: Uniform Convergence](02-uniform-convergence.md) | [Back to README](../README.md) | [Next: Integration and Differentiation of Series →](04-integration-and-differentiation-of-series.md)
