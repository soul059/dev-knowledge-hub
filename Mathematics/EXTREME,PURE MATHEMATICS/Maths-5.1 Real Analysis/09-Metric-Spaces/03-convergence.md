# Chapter 3: Convergence in Metric Spaces

[← Previous: Open and Closed Sets](02-open-and-closed-sets.md) | [Back to README](../README.md) | [Next: Continuity →](04-continuity.md)

---

## Overview

Convergence of sequences in metric spaces is defined exactly as in $\mathbb{R}$, replacing $|x_n - x|$ with $d(x_n, x)$. This gives a unified framework for pointwise convergence, uniform convergence, and other types — all as convergence in different metric spaces.

---

## 3.1 Definition

**Definition 3.1.** In $(X, d)$, a sequence $(x_n)$ **converges** to $x \in X$ if:

$$\forall\, \varepsilon > 0,\; \exists\, N: \; n \geq N \Rightarrow d(x_n, x) < \varepsilon$$

Equivalently: $d(x_n, x) \to 0$ as $n \to \infty$.

**Notation:** $x_n \to x$ or $\lim x_n = x$.

---

## 3.2 Uniqueness of Limits

**Theorem 3.2.** In any metric space, limits are **unique**: if $x_n \to x$ and $x_n \to y$, then $x = y$.

**Proof.** $0 \leq d(x, y) \leq d(x, x_n) + d(x_n, y) \to 0 + 0 = 0$, so $d(x,y) = 0$, hence $x = y$. $\blacksquare$

---

## 3.3 Convergence in Specific Metric Spaces

| Space | Metric | $x_n \to x$ means |
|-------|--------|-------------------|
| $\mathbb{R}$ | $|x_n - x|$ | Usual convergence |
| $\mathbb{R}^k$ | $d_2$ | Componentwise: $x_n^{(i)} \to x^{(i)}$ for each $i$ |
| $C[a,b]$ | $\sup|f_n - f|$ | **Uniform convergence** |
| $C[a,b]$ | $\int|f_n - f|$ | $L^1$ convergence |
| $\ell^\infty$ | $\sup|x_n^{(k)} - x^{(k)}|$ | Uniform convergence of sequences |
| Discrete | $d_\text{discrete}$ | Eventually constant |

**Key insight:** Uniform convergence IS convergence in $(C[a,b], \|\cdot\|_\infty)$.

```
  Convergence in C[a,b]:

  fₙ → f in (C[a,b], d_∞)
     ⟺  sup|fₙ(x) - f(x)| → 0
     ⟺  fₙ → f uniformly

  This is why the sup-metric is natural
  for function space analysis!
```

---

## 3.4 Convergence in $\mathbb{R}^k$

**Theorem 3.4.** $\mathbf{x}_n \to \mathbf{x}$ in $(\mathbb{R}^k, d_2)$ $\iff$ $x_n^{(i)} \to x^{(i)}$ for each $i = 1, \ldots, k$.

**Proof.** $|x_n^{(i)} - x^{(i)}| \leq d_2(\mathbf{x}_n, \mathbf{x}) \leq \sum|x_n^{(j)} - x^{(j)}|$. $\blacksquare$

---

## 3.5 Subsequences

**Definition 3.5.** $(x_{n_k})$ is a **subsequence** if $n_1 < n_2 < \cdots$.

**Theorem.** $x_n \to x \Rightarrow$ every subsequence converges to $x$.

**Theorem.** $x$ is a **cluster point** of $(x_n)$ iff some subsequence converges to $x$.

---

## 3.6 Characterization of Closed Sets

**Theorem 3.6.** $F \subseteq X$ is closed $\iff$ every convergent sequence in $F$ has its limit in $F$:

$$(x_n) \subseteq F,\; x_n \to x \;\Rightarrow\; x \in F$$

This is the **sequential characterization** of closedness in metric spaces.

---

## 3.7 Bounded Sequences

**Definition 3.7.** $(x_n)$ is **bounded** if $\{x_n : n \in \mathbb{N}\}$ is a bounded set, i.e., $\exists\, M, p$: $d(x_n, p) \leq M$ for all $n$.

**Theorem.** Convergent $\Rightarrow$ bounded.

---

## Summary Table

| Property | In metric spaces |
|----------|-----------------|
| Convergence | $d(x_n, x) \to 0$ |
| Uniqueness | Limits unique (always) |
| In $\mathbb{R}^k$ | Componentwise convergence |
| In $C[a,b]$ | = Uniform convergence (sup metric) |
| Closed set | Sequentially closed |
| Convergent ⟹ bounded | Always true |

---

## Quick Revision Questions

1. **Define** convergence in a metric space. Show limits are unique.

2. **Prove** convergence in $\mathbb{R}^k$ is equivalent to componentwise convergence.

3. **Show** that convergence in $(C[0,1], d_\infty)$ is uniform convergence.

4. **Prove** a set is closed iff it is sequentially closed.

5. **Give** an example of a sequence in $C[0,1]$ that converges in $d_1$ but not in $d_\infty$.

6. **Describe** convergent sequences in a discrete metric space.

---

[← Previous: Open and Closed Sets](02-open-and-closed-sets.md) | [Back to README](../README.md) | [Next: Continuity →](04-continuity.md)
