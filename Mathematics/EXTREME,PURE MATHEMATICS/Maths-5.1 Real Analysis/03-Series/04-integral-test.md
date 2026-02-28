# Chapter 4: Integral Test

[← Previous: Ratio and Root Tests](03-ratio-and-root-tests.md) | [Back to README](../README.md) | [Next: Alternating Series →](05-alternating-series.md)

---

## Overview

The Integral Test connects the convergence of a series to the convergence of an improper integral. It works best for series whose terms come from a smooth, positive, decreasing function — including the $p$-series. It also gives bounds on the sum.

---

## 4.1 The Integral Test

**Theorem 4.1 (Integral Test).** Let $f : [1, \infty) \to \mathbb{R}$ be positive, continuous, and decreasing. Then:

$$\sum_{n=1}^{\infty} f(n) \;\text{converges} \iff \int_1^{\infty} f(x)\,dx \;\text{converges}$$

**Proof.** Since $f$ is decreasing, for $n \leq x \leq n+1$:

$$f(n+1) \leq f(x) \leq f(n)$$

Integrating from $n$ to $n+1$:

$$f(n+1) \leq \int_n^{n+1} f(x)\,dx \leq f(n)$$

Summing from $n = 1$ to $N$:

$$\sum_{n=2}^{N+1} f(n) \leq \int_1^{N+1} f(x)\,dx \leq \sum_{n=1}^{N} f(n)$$

```
  Integral Test — Geometric Interpretation:

  f(x)
    ▲
    │█▓░
    │█▓▓░░
    │█▓▓▓░░░
    │████▓▓▓░░░░
    │████████▓▓▓░░░░░
    │████████████▓▓▓░░░░░░░
    └──┬──┬──┬──┬──┬──┬──┬──► x
       1  2  3  4  5  6  7

    █ = f(n) rectangles (LEFT Riemann sum = Σf(n))
    ░ = f(n+1) rectangles (RIGHT Riemann sum = Σf(n+1))
    Curve = f(x)

    RIGHT rectangles ≤ ∫f(x)dx ≤ LEFT rectangles
    Σf(n+1)          ≤ ∫f(x)dx ≤ Σf(n)
```

The left inequality gives: if $\int f$ converges, then $\sum f(n+1)$ converges, hence $\sum f(n)$ converges.

The right inequality gives: if $\sum f(n)$ converges, then $\int f$ converges. $\blacksquare$

---

## 4.2 Applications

### The $p$-Series (Revisited)

$f(x) = 1/x^p$, which is positive, continuous, and decreasing for $x \geq 1$.

$$\int_1^{\infty} \frac{1}{x^p}\,dx = \begin{cases} \frac{x^{1-p}}{1-p}\Big|_1^{\infty} = \frac{1}{p-1} & \text{if } p > 1 \\ \ln x\Big|_1^{\infty} = \infty & \text{if } p = 1 \\ \frac{x^{1-p}}{1-p}\Big|_1^{\infty} = \infty & \text{if } p < 1 \end{cases}$$

So $\sum 1/n^p$ converges iff $p > 1$. ✓

### Other Examples

| Series | $f(x)$ | $\int f$ | Convergent? |
|--------|--------|----------|:-----------:|
| $\sum \frac{1}{n \ln n}$ | $\frac{1}{x \ln x}$ | $\ln(\ln x) \to \infty$ | ❌ |
| $\sum \frac{1}{n(\ln n)^2}$ | $\frac{1}{x(\ln x)^2}$ | $\frac{-1}{\ln x} \to 0$ | ✅ |
| $\sum \frac{1}{n \ln n (\ln \ln n)^2}$ | $\cdots$ | Converges | ✅ |

---

## 4.3 Integral Bounds on Partial Sums

The proof gives us the **integral estimate**:

$$\int_1^{N+1} f(x)\,dx \leq \sum_{n=1}^{N} f(n) \leq f(1) + \int_1^{N} f(x)\,dx$$

This provides tight bounds on partial sums even when we can't compute the exact sum.

### Example: Estimating Partial Sums of Harmonic Series

$$\ln(N+1) \leq \sum_{n=1}^{N} \frac{1}{n} \leq 1 + \ln N$$

So $H_N = \sum_{n=1}^N 1/n \approx \ln N$ (up to a constant — the Euler-Mascheroni constant $\gamma \approx 0.5772$).

```
  Harmonic partial sums vs ln(n):

  Hₙ
    ▲
    │                        Hₙ = 1 + 1/2 + ... + 1/n
    │                    ╱
    │                 ╱     ← approximately ln(n) + γ
    │              ╱
    │           ╱
    │        ╱
    │     ╱
    │  ╱
    │╱
    └──────────────────────────► n
              γ ≈ 0.5772 (Euler-Mascheroni constant)
```

---

## 4.4 Remainder Estimate

If $\sum f(n)$ converges to $S$, the **remainder** $R_N = S - S_N$ satisfies:

$$\int_{N+1}^{\infty} f(x)\,dx \leq R_N \leq \int_N^{\infty} f(x)\,dx$$

This tells us how close $S_N$ is to the true sum $S$.

### Example

For $\sum 1/n^2$: To approximate $\pi^2/6$ within $0.01$:

$$R_N \leq \int_N^{\infty} \frac{1}{x^2}\,dx = \frac{1}{N}$$

Need $1/N < 0.01$, so $N > 100$. We need at least 100 terms.

---

## 4.5 Real-World Application

- **Algorithm analysis**: The integral test bounds on $H_N \approx \ln N$ are used to analyze time complexity (e.g., expected running time of randomized algorithms)
- **Number theory**: The divergence of $\sum 1/(p)$ over primes $p$ (proved via integral test ideas) shows there are "many" primes
- **Physics**: The convergence of $\sum 1/n^2 = \pi^2/6$ arises in quantum mechanics (blackbody radiation, Casimir effect)

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Integral test | $\sum f(n)$ conv. ⟺ $\int_1^\infty f(x)\,dx$ conv. |
| Requirements | $f$ positive, continuous, decreasing |
| $p$-series via integral | $\int x^{-p}\,dx$ converges ⟺ $p > 1$ |
| Integral bounds | $\int_1^{N+1} f \leq S_N \leq f(1) + \int_1^N f$ |
| Remainder estimate | $\int_{N+1}^\infty f \leq R_N \leq \int_N^\infty f$ |
| Harmonic growth | $H_N \approx \ln N + \gamma$ |

---

## Quick Revision Questions

1. **State and prove the Integral Test.** Draw the rectangle comparison diagram.

2. **Use the Integral Test** to determine convergence of $\sum \frac{1}{n(\ln n)^3}$.

3. **Estimate** $\sum_{n=1}^{1000} 1/n$ using the integral bounds.

4. **How many terms** of $\sum 1/n^3$ are needed to approximate the sum within $10^{-6}$?

5. **True or False:** The Integral Test can determine convergence even when the Ratio and Root Tests give $L = 1$.

6. **What is the Euler-Mascheroni constant?** How does it relate to the harmonic series?

---

[← Previous: Ratio and Root Tests](03-ratio-and-root-tests.md) | [Back to README](../README.md) | [Next: Alternating Series →](05-alternating-series.md)
