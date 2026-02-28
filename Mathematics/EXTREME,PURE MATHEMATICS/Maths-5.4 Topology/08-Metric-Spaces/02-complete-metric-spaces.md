# Chapter 8.2 — Complete Metric Spaces

[← Previous: Metrics and Metric Topology](01-metrics-and-metric-topology.md) | [Back to README](../README.md) | [Next: Baire Category Theorem →](03-baire-category-theorem.md)

---

## 1. Overview

**Completeness** — every Cauchy sequence converges — is fundamental in analysis. Unlike most topological properties, completeness depends on the specific metric, not just the topology. Complete metric spaces are the natural setting for fixed-point theorems, Baire category, and much of functional analysis.

---

## 2. Definitions

> **Def 2.1 (Cauchy Sequence).** A sequence $(x_n)$ in $(X, d)$ is **Cauchy** if $\forall \epsilon > 0$, $\exists N$ such that $m, n \geq N \Rightarrow d(x_m, x_n) < \epsilon$.

> **Def 2.2 (Complete).** $(X, d)$ is **complete** if every Cauchy sequence in $X$ converges (to a limit in $X$).

```
Cauchy sequence behaviour:
  x₁     x₂  x₃ x₄x₅x₆ → L
  ─•──────•───•──•••──────→
           ←──────→
           terms eventually
           cluster together
```

---

## 3. Key Observations

- Convergent ⟹ Cauchy (always).
- Cauchy ⟹ Convergent *only* if the space is complete.
- Completeness is a **metric property**, not a topological property: $(0,1)$ and $\mathbb{R}$ are homeomorphic, but $(0,1)$ is not complete (with standard metric) while $\mathbb{R}$ is.

---

## 4. Examples

| Space | Complete? | Notes |
|-------|:---------:|-------|
| $\mathbb{R}$ (standard) | ✓ | Axiom of completeness |
| $\mathbb{R}^n$ (Euclidean) | ✓ | Componentwise |
| $\mathbb{Q}$ | ✗ | $1, 1.4, 1.41, 1.414, \ldots \to \sqrt{2} \notin \mathbb{Q}$ |
| $(0, 1)$ (standard) | ✗ | $x_n = 1/n \to 0 \notin (0,1)$ |
| $C([0,1])$ (sup metric) | ✓ | Uniform limit of cts functions is cts |
| $\ell^p$ ($1 \leq p \leq \infty$) | ✓ | Riesz–Fischer |
| Discrete metric space | ✓ | Cauchy $\Rightarrow$ eventually constant |
| $p$-adic numbers $\mathbb{Q}_p$ | ✓ | Completion of $(\mathbb{Q}, d_p)$ |

---

## 5. Key Theorems

### 5.1 Completeness of Closed Subsets

> **Thm 2.3.** A closed subset of a complete metric space is complete. Conversely, a complete subspace of a metric space is closed.

*Proof:* ($\Rightarrow$) Cauchy in $F$ (closed) ⟹ Cauchy in $X$ ⟹ converges in $X$ ⟹ limit is in $F$ (closed).
($\Leftarrow$) Cauchy in $X$ with limit in $F$? Yes: every convergent sequence in $F$ has limit in $F$ since $F$ is complete, and complete subspace contains all its limit points. ∎

### 5.2 Completeness of Products

> **Thm 2.4.** A countable product $\prod_{n=1}^\infty (X_n, d_n)$ of complete metric spaces is complete under the product metric.

### 5.3 Banach Fixed-Point Theorem

> **Thm 2.5 (Contraction Mapping Theorem).** Let $(X, d)$ be a complete, nonempty metric space and $T : X \to X$ a contraction ($\exists \, 0 \leq k < 1$ with $d(Tx, Ty) \leq k \cdot d(x,y)$). Then $T$ has a unique fixed point $x^* = Tx^*$, and for any $x_0$, the iterates $T^n x_0 \to x^*$.

```
Contraction iteration:
 x₀ ──T──→ x₁ ──T──→ x₂ ──T──→ x₃ ──→ ··· → x*
  │                                              │
  └──────── distances shrink by factor k ────────┘
  
 d(xₙ, x*) ≤ kⁿ/(1-k) · d(x₀, x₁)
```

**Applications:** ODE existence (Picard–Lindelöf), iterative methods in numerical analysis, Nash equilibrium existence.

---

## 6. Completion of a Metric Space

> **Thm 2.6.** Every metric space $(X, d)$ has a **completion** $(\hat{X}, \hat{d})$: a complete metric space containing $X$ as a dense subspace (up to isometry).

**Construction:** $\hat{X} = \{\text{equivalence classes of Cauchy sequences}\}$ with $[(x_n)] = [(y_n)]$ iff $d(x_n, y_n) \to 0$. Define $\hat{d}([(x_n)], [(y_n)]) = \lim d(x_n, y_n)$.

| Space | Completion |
|-------|-----------|
| $\mathbb{Q}$ | $\mathbb{R}$ |
| $(0, 1)$ | $[0, 1]$ |
| $(\mathbb{Q}, d_p)$ | $\mathbb{Q}_p$ ($p$-adic numbers) |
| Polynomial space (with $L^2$ norm) | $L^2$ |

> **Uniqueness:** The completion is unique up to isometry.

---

## 7. Totally Bounded Spaces

> **Def 2.7.** $(X, d)$ is **totally bounded** if $\forall \epsilon > 0$, $X$ can be covered by finitely many open balls of radius $\epsilon$.

> **Thm 2.8.** $X$ is compact ⟺ $X$ is complete and totally bounded.

```
Compact = Complete + Totally Bounded

  Totally bounded:         NOT totally bounded:
  ┌────────────────┐      ┌────────────────┐
  │  (·)(·)(·)     │      │  •              │
  │  (·)(·)(·)     │      │       •         │
  │  (·)(·)(·)     │      │            •    │
  │  finitely many │      │                •│
  │  ε-balls cover │      │  points spread  │
  └────────────────┘      └────────────────┘
```

---

## 8. Summary Table

| Concept | Details |
|---------|---------|
| Cauchy sequence | Terms eventually arbitrarily close |
| Complete | Every Cauchy sequence converges |
| Not topological | Depends on metric, not just topology |
| Closed ⊂ complete = complete | Completeness is inherited by closed subsets |
| Contraction mapping | Unique fixed point in complete space |
| Completion | Every metric space embeds densely in a complete one |
| Compact ⟺ complete + totally bounded | Fundamental characterisation |

---

## 9. Quick Revision Questions

1. Define Cauchy sequence and completeness. Why is completeness not a topological invariant?

2. Prove that a closed subset of a complete metric space is complete.

3. State and prove the Banach Fixed-Point Theorem.

4. Describe the completion construction. What is the completion of $\mathbb{Q}$?

5. Define total boundedness and prove: compact ⟺ complete + totally bounded.

6. Is $C([0,1])$ with the sup metric complete? Justify.

---

[← Previous: Metrics and Metric Topology](01-metrics-and-metric-topology.md) | [Back to README](../README.md) | [Next: Baire Category Theorem →](03-baire-category-theorem.md)
