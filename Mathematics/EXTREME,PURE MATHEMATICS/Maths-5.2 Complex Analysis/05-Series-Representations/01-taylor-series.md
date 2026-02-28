# 5.1 Taylor Series

[← Previous: Morera's Theorem](../04-Complex-Integration/06-moreras-theorem.md) | [Next: Laurent Series →](02-laurent-series.md)

---

## Chapter Overview

Every analytic function can be represented as a convergent **power series** (Taylor series) around any point in its domain. This is a uniquely complex-analytic phenomenon — real-differentiable functions need not have convergent Taylor series. We study the form, radius of convergence, and uniqueness of Taylor expansions.

---

## 1. Taylor's Theorem for Analytic Functions

### 1.1 Statement

> **Theorem.** If $f$ is analytic in the disk $|z - z_0| < R$, then
> $$\boxed{f(z) = \sum_{n=0}^{\infty} a_n(z - z_0)^n, \quad |z - z_0| < R}$$
> where
> $$a_n = \frac{f^{(n)}(z_0)}{n!} = \frac{1}{2\pi i}\oint_C \frac{f(z)}{(z - z_0)^{n+1}}\,dz$$

### 1.2 Proof Sketch

From the Cauchy integral formula:

$$f(z) = \frac{1}{2\pi i}\oint_C \frac{f(\zeta)}{\zeta - z}\,d\zeta$$

Write $\frac{1}{\zeta - z} = \frac{1}{(\zeta - z_0) - (z - z_0)} = \frac{1}{\zeta - z_0}\cdot\frac{1}{1 - \frac{z-z_0}{\zeta - z_0}}$.

Expand the geometric series (valid since $|z - z_0| < |\zeta - z_0|$):

$$= \sum_{n=0}^{\infty} \frac{(z-z_0)^n}{(\zeta-z_0)^{n+1}}$$

Substitute and interchange sum and integral. $\square$

---

## 2. Radius of Convergence

### 2.1 Definition

The **radius of convergence** $R$ of $\sum a_n(z-z_0)^n$ is:

$$R = \frac{1}{\limsup_{n\to\infty} |a_n|^{1/n}}$$

Or by the **ratio test**: $R = \lim_{n\to\infty}\left|\frac{a_n}{a_{n+1}}\right|$ (when this limit exists).

### 2.2 Geometric Interpretation

The series converges in the largest disk centered at $z_0$ that avoids all singularities of $f$.

```
Radius of convergence = distance to nearest singularity:

            Im
             ▲
             │         × singularity at z₁
             │       ╱
             │     ╱  R = |z₁ - z₀|
             │   ╱
       ──────z₀────────────► Re
             │╲
             │  ╲  convergent
             │    ╲ inside disk
             │
             │              × another singularity
```

### 2.3 Key Facts

| Property | Statement |
|----------|-----------|
| Inside disk | Series converges absolutely and uniformly on compact subsets |
| Outside disk | Series diverges |
| On boundary | Anything can happen (convergent, divergent, or conditional) |
| $R = \infty$ | $f$ is entire → Taylor series converges everywhere |

---

## 3. Important Taylor Series

### 3.1 Table of Standard Expansions (about $z_0 = 0$)

| Function | Taylor Series | Radius |
|----------|--------------|--------|
| $e^z$ | $\sum_{n=0}^\infty \frac{z^n}{n!}$ | $\infty$ |
| $\sin z$ | $\sum_{n=0}^\infty \frac{(-1)^n z^{2n+1}}{(2n+1)!}$ | $\infty$ |
| $\cos z$ | $\sum_{n=0}^\infty \frac{(-1)^n z^{2n}}{(2n)!}$ | $\infty$ |
| $\frac{1}{1-z}$ | $\sum_{n=0}^\infty z^n$ | $1$ |
| $\frac{1}{1+z}$ | $\sum_{n=0}^\infty (-1)^n z^n$ | $1$ |
| $\frac{1}{(1-z)^2}$ | $\sum_{n=0}^\infty (n+1)z^n$ | $1$ |
| $\text{Log}(1+z)$ | $\sum_{n=1}^\infty \frac{(-1)^{n+1}z^n}{n}$ | $1$ |
| $\sinh z$ | $\sum_{n=0}^\infty \frac{z^{2n+1}}{(2n+1)!}$ | $\infty$ |
| $\cosh z$ | $\sum_{n=0}^\infty \frac{z^{2n}}{(2n)!}$ | $\infty$ |

---

## 4. Uniqueness of Taylor Series

### 4.1 Identity Theorem (Preview)

If two analytic functions agree on a set with an accumulation point in their common domain, they are identical.

**Consequence:** The Taylor series of $f$ about $z_0$ is **unique**. Any method that produces a power series equal to $f$ near $z_0$ must give the Taylor series.

### 4.2 Practical Implication

You can find Taylor coefficients by *any* method — direct formula, algebraic manipulation, substitution, multiplication of known series — and the result is guaranteed to be the unique Taylor series.

---

## 5. Operations on Power Series

| Operation | Result |
|-----------|--------|
| **Addition** | $\sum a_n z^n + \sum b_n z^n = \sum (a_n+b_n)z^n$; $R \geq \min(R_1, R_2)$ |
| **Multiplication** | $(\sum a_n z^n)(\sum b_n z^n) = \sum c_n z^n$ where $c_n = \sum_{k=0}^n a_k b_{n-k}$ |
| **Differentiation** | $\frac{d}{dz}\sum a_n z^n = \sum na_n z^{n-1}$; same $R$ |
| **Integration** | $\int \sum a_n z^n\,dz = \sum \frac{a_n}{n+1}z^{n+1} + C$; same $R$ |
| **Substitution** | $f(g(z)) = \sum a_n[g(z)]^n$ if $|g(z)| < R$ |

---

## 6. Design Examples

### Example 1: Find the Taylor series of $f(z) = \frac{1}{z}$ about $z_0 = 1$.

Write $\frac{1}{z} = \frac{1}{1 + (z-1)} = \sum_{n=0}^\infty (-1)^n(z-1)^n$, valid for $|z - 1| < 1$.

**Verification:** Nearest singularity is $z = 0$, with $|0 - 1| = 1 = R$. ✓

### Example 2: Find the Taylor series of $f(z) = \frac{z}{(1-z)(2-z)}$ about $z_0 = 0$.

**Step 1.** Partial fractions:

$$\frac{z}{(1-z)(2-z)} = \frac{1}{1-z} - \frac{2}{2-z} = \frac{1}{1-z} - \frac{1}{1-z/2}$$

**Step 2.** Expand each term:

$$= \sum_{n=0}^\infty z^n - \sum_{n=0}^\infty \frac{z^n}{2^n} = \sum_{n=0}^\infty \left(1 - \frac{1}{2^n}\right)z^n$$

**Step 3.** Radius: $R = \min(1, 2) = 1$ (distance to nearest singularity at $z = 1$).

### Example 3: Find the first few terms of the Taylor series of $\tan z$ about $z = 0$.

$$\tan z = \frac{\sin z}{\cos z} = \frac{z - z^3/6 + z^5/120 - \cdots}{1 - z^2/2 + z^4/24 - \cdots}$$

By long division:

$$\tan z = z + \frac{z^3}{3} + \frac{2z^5}{15} + \cdots$$

Radius $R = \pi/2$ (nearest singularity at $z = \pi/2$).

---

## 7. Real-World Applications

| Application | Taylor Series Role |
|-------------|-------------------|
| **Numerical computation** | Evaluating special functions via truncated series |
| **Physics** | Small-angle approximation ($\sin\theta \approx \theta$) |
| **Engineering** | Transfer function expansion for frequency analysis |
| **Optics** | Aberration theory via power series of wavefront |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Taylor series | $f(z) = \sum \frac{f^{(n)}(z_0)}{n!}(z-z_0)^n$ |
| Radius of convergence | $R$ = distance to nearest singularity |
| Uniqueness | Taylor series is unique (Identity theorem) |
| Analytic ↔ power series | In complex analysis, these are equivalent |
| Term-by-term operations | Differentiation, integration preserve $R$ |
| Multiplication | Cauchy product formula; $R \geq \min(R_1, R_2)$ |

---

## Quick Revision Questions

1. **State** Taylor's theorem for analytic functions, giving the formula for the coefficients.

2. **Find** the Taylor series of $\frac{1}{1+z^2}$ about $z = 0$ and determine the radius of convergence.

3. **Explain** why the radius of convergence of $\frac{1}{1+z^2}$ about $z = 0$ is $1$, even though the function is perfectly smooth on $\mathbb{R}$.

4. **Find** the Taylor series of $e^z \sin z$ about $z = 0$ up to the $z^4$ term.

5. **Prove** that differentiation of a power series preserves the radius of convergence.

6. **Find** the Taylor series of $\frac{1}{z}$ about $z_0 = 2$ and determine $R$.

---

[← Previous: Morera's Theorem](../04-Complex-Integration/06-moreras-theorem.md) | [Next: Laurent Series →](02-laurent-series.md)
