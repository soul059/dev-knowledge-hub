# Chapter 1: Definition and Examples of Metric Spaces

[← Previous: Unit 8 — Taylor Series](../08-Sequences-and-Series-of-Functions/06-taylor-series.md) | [Back to README](../README.md) | [Next: Open and Closed Sets →](02-open-and-closed-sets.md)

---

## Overview

A **metric space** abstracts the notion of distance from $\mathbb{R}$ to arbitrary sets. This generalization unifies the theory of convergence, continuity, compactness, and completeness into a single framework.

---

## 1.1 Definition

**Definition 1.1.** A **metric space** is a pair $(X, d)$ where $X$ is a non-empty set and $d: X \times X \to [0, \infty)$ is a **metric** (distance function) satisfying:

| Axiom | Statement | Name |
|-------|-----------|------|
| (M1) | $d(x, y) = 0 \iff x = y$ | Identity of indiscernibles |
| (M2) | $d(x, y) = d(y, x)$ | Symmetry |
| (M3) | $d(x, z) \leq d(x, y) + d(y, z)$ | Triangle inequality |

**Non-negativity** $d(x,y) \geq 0$ follows: $0 = d(x,x) \leq d(x,y) + d(y,x) = 2d(x,y)$.

---

## 1.2 Examples

### Real line

$(\mathbb{R}, d)$ with $d(x, y) = |x - y|$. This is the metric we've used throughout the course.

### Euclidean space

$(\mathbb{R}^n, d_2)$ with $d_2(\mathbf{x}, \mathbf{y}) = \sqrt{\sum_{k=1}^n (x_k - y_k)^2}$.

### Other metrics on $\mathbb{R}^n$

| Metric | Formula | Name |
|--------|---------|------|
| $d_1$ | $\sum_{k=1}^n |x_k - y_k|$ | Taxicab / Manhattan |
| $d_2$ | $\sqrt{\sum (x_k - y_k)^2}$ | Euclidean |
| $d_\infty$ | $\max_k |x_k - y_k|$ | Chebyshev / sup |
| $d_p$ | $\left(\sum |x_k - y_k|^p\right)^{1/p}$ | $\ell^p$ metric ($p \geq 1$) |

```
  Unit balls in ℝ² under different metrics:

  d₁ (diamond)      d₂ (circle)      d∞ (square)

      ◆                 ●               ■■■
     ◆ ◆              ●   ●            ■   ■
    ◆   ◆            ●     ●           ■   ■
     ◆ ◆              ●   ●            ■   ■
      ◆                 ●               ■■■
```

### Function spaces

**$C[a,b]$** with supremum metric: $d(f, g) = \|f - g\|_\infty = \sup_{x \in [a,b]} |f(x) - g(x)|$.

**$C[a,b]$** with $L^1$ metric: $d_1(f, g) = \int_a^b |f(x) - g(x)|\,dx$.

### Discrete metric

On any set $X$: $d(x,y) = \begin{cases} 0 & x = y \\ 1 & x \neq y \end{cases}$

This satisfies all axioms. Every subset is open; every sequence converges iff eventually constant.

### Sequence spaces

$\ell^\infty$ = bounded sequences with $d(\mathbf{x}, \mathbf{y}) = \sup_n |x_n - y_n|$.

$\ell^2$ = square-summable sequences with $d(\mathbf{x}, \mathbf{y}) = \sqrt{\sum |x_n - y_n|^2}$.

---

## 1.3 Verifying the Triangle Inequality

The hardest axiom to verify is (M3). Key tools:

**Cauchy–Schwarz inequality:** $\left|\sum x_k y_k\right| \leq \sqrt{\sum x_k^2}\sqrt{\sum y_k^2}$

**Minkowski's inequality:** $\left(\sum|x_k + y_k|^p\right)^{1/p} \leq \left(\sum|x_k|^p\right)^{1/p} + \left(\sum|y_k|^p\right)^{1/p}$ for $p \geq 1$.

---

## 1.4 Subspaces

**Definition 1.4.** If $(X, d)$ is a metric space and $Y \subseteq X$, then $(Y, d|_{Y \times Y})$ is a **metric subspace**.

Example: $(\mathbb{Q}, |\cdot|)$ is a subspace of $(\mathbb{R}, |\cdot|)$.

---

## 1.5 Equivalent Metrics

**Definition 1.5.** Metrics $d_1, d_2$ on $X$ are **equivalent** if they generate the same open sets:

$$\exists\, \alpha, \beta > 0: \alpha\, d_1(x,y) \leq d_2(x,y) \leq \beta\, d_1(x,y) \quad \forall\, x, y$$

**Theorem.** On $\mathbb{R}^n$, all $\ell^p$ metrics are equivalent:

$$d_\infty(\mathbf{x},\mathbf{y}) \leq d_2(\mathbf{x},\mathbf{y}) \leq d_1(\mathbf{x},\mathbf{y}) \leq n\, d_\infty(\mathbf{x},\mathbf{y})$$

---

## 1.6 Bounded and Diameter

**Definition 1.6.** $A \subseteq X$ is **bounded** if $\text{diam}(A) = \sup\{d(x,y) : x, y \in A\} < \infty$.

---

## Summary Table

| Example | Space | Metric |
|---------|-------|--------|
| Real line | $\mathbb{R}$ | $|x - y|$ |
| Euclidean | $\mathbb{R}^n$ | $\sqrt{\sum(x_k-y_k)^2}$ |
| Continuous functions | $C[a,b]$ | $\sup|f-g|$ |
| Discrete | Any $X$ | $d = 0$ or $1$ |
| Sequences | $\ell^p$ | $(\sum|x_n-y_n|^p)^{1/p}$ |

---

## Quick Revision Questions

1. **State** the three axioms for a metric. Verify them for $d(x,y) = |x - y|$ on $\mathbb{R}$.

2. **Verify** the discrete metric satisfies all metric axioms.

3. **Show** $d_\infty \leq d_2 \leq d_1$ on $\mathbb{R}^n$.

4. **Prove** that $d(f, g) = \sup|f - g|$ is a metric on $C[a,b]$.

5. **Give** three different metrics on $\mathbb{R}^2$ and sketch their unit balls.

6. **Define** equivalent metrics and explain why all $\ell^p$ metrics on $\mathbb{R}^n$ are equivalent.

---

[← Previous: Unit 8 — Taylor Series](../08-Sequences-and-Series-of-Functions/06-taylor-series.md) | [Back to README](../README.md) | [Next: Open and Closed Sets →](02-open-and-closed-sets.md)
