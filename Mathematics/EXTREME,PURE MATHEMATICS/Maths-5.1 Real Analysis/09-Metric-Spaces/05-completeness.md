# Chapter 5: Completeness in Metric Spaces

[← Previous: Continuity](04-continuity.md) | [Back to README](../README.md) | [Next: Compactness →](06-compactness.md)

---

## Overview

A metric space is **complete** if every Cauchy sequence converges (within the space). Completeness is crucial for fixed-point theorems, existence proofs, and analysis. $\mathbb{R}$ is complete; $\mathbb{Q}$ is not. We examine which familiar spaces are complete and how to complete those that aren't.

---

## 5.1 Cauchy Sequences

**Definition 5.1.** $(x_n)$ in $(X, d)$ is **Cauchy** if:

$$\forall\, \varepsilon > 0,\; \exists\, N: \; m, n \geq N \Rightarrow d(x_m, x_n) < \varepsilon$$

**Theorem.** Every convergent sequence is Cauchy.

**Proof.** $d(x_m, x_n) \leq d(x_m, x) + d(x, x_n) < \varepsilon/2 + \varepsilon/2 = \varepsilon$. $\blacksquare$

The converse is what defines completeness.

---

## 5.2 Complete Metric Spaces

**Definition 5.2.** $(X, d)$ is **complete** if every Cauchy sequence in $X$ converges to a point in $X$.

```
  Complete vs. Incomplete:

  Complete (ℝ):       Incomplete (ℚ):

  ──●──●─●●●──→ L     ──●──●─●●●──→ L = √2
                 ∈ ℝ                   ∉ ℚ ✗

  Cauchy sequences     Cauchy sequences may
  always converge      converge to a "gap"
```

---

## 5.3 Examples

| Space | Complete? | Reason |
|-------|-----------|--------|
| $(\mathbb{R}, |\cdot|)$ | **Yes** | Axiom of completeness |
| $(\mathbb{Q}, |\cdot|)$ | No | $1, 1.4, 1.41, \ldots \to \sqrt{2} \notin \mathbb{Q}$ |
| $(\mathbb{R}^n, d_2)$ | **Yes** | Componentwise: each component Cauchy in $\mathbb{R}$ |
| $(C[a,b], d_\infty)$ | **Yes** | Uniform limit of continuous = continuous |
| $(C[a,b], d_1)$ | No | $L^1$-Cauchy doesn't imply uniform |
| $(\ell^\infty, d_\infty)$ | **Yes** | Componentwise convergence |
| Discrete $(X, d)$ | **Yes** | Cauchy $\Rightarrow$ eventually constant $\Rightarrow$ convergent |
| $(0, 1)$ with $|\cdot|$ | No | $1/n$ is Cauchy, converges to $0 \notin (0,1)$ |

---

## 5.4 Completeness of $C[a,b]$

**Theorem 5.4.** $(C[a,b], \|\cdot\|_\infty)$ is complete.

**Proof.** Let $(f_n)$ be Cauchy: $\forall \varepsilon$, $\exists N$: $\|f_m - f_n\|_\infty < \varepsilon$ for $m, n \geq N$.

1. For each fixed $x$, $(f_n(x))$ is Cauchy in $\mathbb{R}$ (since $|f_m(x) - f_n(x)| \leq \|f_m - f_n\|_\infty$). Define $f(x) = \lim f_n(x)$.

2. Letting $m \to \infty$ in $|f_m(x) - f_n(x)| < \varepsilon$: $|f(x) - f_n(x)| \leq \varepsilon$ for all $x$, so $\|f - f_n\|_\infty \leq \varepsilon$. Thus $f_n \to f$ uniformly.

3. Uniform limit of continuous functions is continuous (Theorem from Unit 8), so $f \in C[a,b]$. $\blacksquare$

---

## 5.5 Properties of Complete Spaces

**Theorem 5.5.1.** A closed subset of a complete metric space is complete.

**Proof.** Cauchy in $F$ $\Rightarrow$ Cauchy in $X$ $\Rightarrow$ convergent in $X$ $\Rightarrow$ limit in $F$ (since $F$ closed). $\blacksquare$

**Theorem 5.5.2.** A complete subspace of any metric space is closed.

**Theorem 5.5.3 (Nested Set Theorem).** If $(X, d)$ complete and $F_1 \supseteq F_2 \supseteq \cdots$ are non-empty closed sets with $\text{diam}(F_n) \to 0$, then $\bigcap F_n = \{x\}$ (a single point).

---

## 5.6 Baire Category Theorem

**Theorem 5.6 (Baire).** In a complete metric space, the countable intersection of open dense sets is dense.

Equivalently: a complete metric space is **not** the countable union of nowhere dense sets (i.e., complete spaces are of **second category**).

**Consequences:**
- $\mathbb{R}$ is not a countable union of nowhere dense sets.
- The set of continuous nowhere-differentiable functions is dense in $C[0,1]$ (most continuous functions are nowhere differentiable!).

---

## 5.7 Completion

**Theorem 5.7.** Every metric space $(X, d)$ has a **completion**: a complete metric space $(\hat{X}, \hat{d})$ containing an isometric copy of $X$ as a dense subset. The completion is unique up to isometry.

**Construction.** Define equivalence classes of Cauchy sequences in $X$:

$$(x_n) \sim (y_n) \iff d(x_n, y_n) \to 0$$

$$\hat{d}([(x_n)], [(y_n)]) = \lim_{n\to\infty} d(x_n, y_n)$$

**Examples:**
- Completion of $\mathbb{Q}$ = $\mathbb{R}$
- Completion of $(0, 1)$ = $[0, 1]$

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Cauchy sequence | $d(x_m, x_n) < \varepsilon$ for large $m, n$ |
| Complete | Every Cauchy converges |
| $\mathbb{R}, \mathbb{R}^n, C[a,b]_{d_\infty}$ | Complete |
| $\mathbb{Q}, (0,1), C[a,b]_{d_1}$ | Incomplete |
| Closed ⊂ complete | Complete |
| Complete ⊂ any | Closed |
| Baire | $\bigcap$ open dense = dense |
| Completion | Every space embeds densely in a complete space |

---

## Quick Revision Questions

1. **Define** Cauchy sequence and completeness in a metric space.

2. **Prove** $(C[a,b], \|\cdot\|_\infty)$ is complete.

3. **Show** that a closed subset of a complete metric space is complete.

4. **Show** $(\mathbb{Q}, |\cdot|)$ is NOT complete by exhibiting a Cauchy sequence with no rational limit.

5. **State** the Baire Category Theorem and give one consequence.

6. **Describe** the completion construction. What is the completion of $\mathbb{Q}$?

---

[← Previous: Continuity](04-continuity.md) | [Back to README](../README.md) | [Next: Compactness →](06-compactness.md)
