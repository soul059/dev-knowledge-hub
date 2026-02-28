# Chapter 5: Power Series

[← Previous: Integration and Differentiation of Series](04-integration-and-differentiation-of-series.md) | [Back to README](../README.md) | [Next: Taylor Series →](06-taylor-series.md)

---

## Overview

**Power series** $\sum a_n x^n$ are the best-behaved series of functions. They have a natural **radius of convergence**, converge uniformly on compact subsets of the interval of convergence, and can be differentiated and integrated term by term within that interval.

---

## 5.1 Definition

**Definition 5.1.** A **power series** centered at $c$ is:

$$\sum_{n=0}^\infty a_n(x - c)^n = a_0 + a_1(x-c) + a_2(x-c)^2 + \cdots$$

We often write centered at $0$: $\sum a_n x^n$.

---

## 5.2 Radius of Convergence

**Theorem 5.2 (Cauchy–Hadamard).** For $\sum a_n x^n$, define:

$$R = \frac{1}{\limsup_{n\to\infty} |a_n|^{1/n}} \in [0, \infty]$$

Then:
- $|x| < R$: series converges **absolutely**
- $|x| > R$: series **diverges**
- $|x| = R$: anything can happen

$R$ is the **radius of convergence**.

```
  Convergence behavior:

  ◄──── diverges ────┤ may vary ├──── converges (abs.) ────┤ may vary ├──── diverges ────►
                    -R          -R                         R           R
                     ◄──────────── interval of convergence ────────────►
                                        (open)
```

**Alternative formula (Ratio).** If $\lim |a_{n+1}/a_n|$ exists:

$$R = \lim_{n\to\infty}\left|\frac{a_n}{a_{n+1}}\right|$$

---

## 5.3 Examples

| Power Series | $a_n$ | $R$ | Interval | At endpoints |
|-------------|-------|-----|----------|-------------|
| $\sum x^n$ | $1$ | $1$ | $(-1,1)$ | Both diverge |
| $\sum x^n/n$ | $1/n$ | $1$ | $[-1, 1)$ | $x=-1$: conv. (AST); $x=1$: div. (harmonic) |
| $\sum x^n/n^2$ | $1/n^2$ | $1$ | $[-1, 1]$ | Both converge (abs.) |
| $\sum x^n/n!$ | $1/n!$ | $\infty$ | $\mathbb{R}$ | — |
| $\sum n!\, x^n$ | $n!$ | $0$ | $\{0\}$ | — |
| $\sum x^{2n}/n$ | — | $1$ | $(-1, 1)$ | Check separately |

---

## 5.4 Uniform Convergence of Power Series

**Theorem 5.4.** If $\sum a_n x^n$ has radius $R > 0$, then it converges **uniformly** on $[-r, r]$ for any $0 < r < R$.

**Proof.** $|a_n x^n| \leq |a_n| r^n = M_n$. Since $r < R$, $\sum |a_n|r^n$ converges. By M-test. $\blacksquare$

**Note:** Uniform convergence on $(-R, R)$ itself may fail (e.g., $\sum x^n$).

---

## 5.5 Term-by-Term Operations

**Theorem 5.5.** Let $f(x) = \sum_{n=0}^\infty a_n x^n$ with radius $R > 0$. Then on $(-R, R)$:

**(a) $f$ is continuous.**

**(b) $f$ is differentiable** and:

$$f'(x) = \sum_{n=1}^\infty n\, a_n\, x^{n-1}$$

with the **same** radius $R$.

**(c) $f$ is integrable** and:

$$\int_0^x f(t)\,dt = \sum_{n=0}^\infty \frac{a_n}{n+1}\, x^{n+1}$$

with the **same** radius $R$.

**(d)** $f$ is **infinitely differentiable** ($C^\infty$) on $(-R, R)$, and:

$$a_n = \frac{f^{(n)}(0)}{n!}$$

**Proof of same radius for $f'$.** $\limsup |n\, a_n|^{1/n} = \limsup |a_n|^{1/n}$ since $n^{1/n} \to 1$. $\blacksquare$

```
  Power series operations preserve radius:

  f(x) = Σ aₙxⁿ         radius R
           │
     ┌─────┼─────┐
     ▼     ▼     ▼
   f'(x)  ∫f   f''(x)    ALL have radius R
  Σ naₙxⁿ⁻¹  Σ aₙxⁿ⁺¹/(n+1)
```

---

## 5.6 Abel's Theorem

**Theorem 5.6 (Abel).** If $\sum a_n$ converges (i.e., the series converges at $x = R = 1$), then:

$$\lim_{x \to 1^-} \sum_{n=0}^\infty a_n x^n = \sum_{n=0}^\infty a_n$$

The power series is **continuous from the left** at the boundary if the series converges there.

**Application.** $\sum (-1)^n/(n+1) = \ln 2$ (from integrating $1/(1+x) = \sum (-x)^n$).

---

## 5.7 Algebra of Power Series

**Theorem 5.7.** If $f(x) = \sum a_n x^n$ (radius $R_1$) and $g(x) = \sum b_n x^n$ (radius $R_2$):

**Addition:** $f + g = \sum (a_n + b_n)x^n$ with radius $\geq \min(R_1, R_2)$.

**Cauchy Product:** $f \cdot g = \sum c_n x^n$ where $c_n = \sum_{k=0}^n a_k b_{n-k}$, radius $\geq \min(R_1, R_2)$.

**Example.** $e^x \cdot e^x = e^{2x}$:

$$\sum \frac{x^n}{n!}\cdot\sum\frac{x^n}{n!} = \sum c_n x^n, \quad c_n = \sum_{k=0}^n \frac{1}{k!(n-k)!} = \frac{2^n}{n!}$$

---

## 5.8 Identity Theorem

**Theorem 5.8.** If $\sum a_n x^n = \sum b_n x^n$ for all $x$ in some interval $(-\delta, \delta)$, then $a_n = b_n$ for all $n$.

Power series representations are **unique**.

---

## Summary Table

| Property | Statement |
|----------|-----------|
| Radius | $R = 1/\limsup|a_n|^{1/n}$ |
| Absolute convergence | On $(-R, R)$ |
| Uniform convergence | On $[-r, r]$, $r < R$ |
| Differentiation | $f' = \sum n a_n x^{n-1}$, same $R$ |
| Integration | $\int f = \sum \frac{a_n}{n+1}x^{n+1}$, same $R$ |
| $C^\infty$ | Power series are infinitely differentiable |
| Abel's theorem | Continuity at boundary if series converges |
| Uniqueness | $a_n = f^{(n)}(0)/n!$ |

---

## Quick Revision Questions

1. **State** the Cauchy–Hadamard formula for the radius of convergence.

2. **Find** the radius of convergence of $\sum \frac{n^2}{3^n}x^n$.

3. **Prove** that differentiation preserves the radius of convergence.

4. **Derive** $\ln(1+x) = x - x^2/2 + x^3/3 - \cdots$ by integrating $1/(1+x) = \sum(-x)^n$.

5. **State** Abel's theorem. Use it to show $\ln 2 = 1 - 1/2 + 1/3 - \cdots$.

6. **Explain** why power series are $C^\infty$ on their interval of convergence.

---

[← Previous: Integration and Differentiation of Series](04-integration-and-differentiation-of-series.md) | [Back to README](../README.md) | [Next: Taylor Series →](06-taylor-series.md)
