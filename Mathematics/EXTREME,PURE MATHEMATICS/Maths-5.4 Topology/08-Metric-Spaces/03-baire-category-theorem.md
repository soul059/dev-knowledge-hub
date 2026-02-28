# Chapter 8.3 — Baire Category Theorem

[← Previous: Complete Metric Spaces](02-complete-metric-spaces.md) | [Back to README](../README.md) | [Next: Compactness in Metric Spaces →](04-compactness-in-metric-spaces.md)

---

## 1. Overview

The **Baire Category Theorem** (BCT) is one of the cornerstones of analysis and topology. It says that complete metric spaces (and locally compact Hausdorff spaces) are "large" — they cannot be written as a countable union of nowhere dense sets. This result underpins the Open Mapping Theorem, Closed Graph Theorem, Uniform Boundedness Principle, and more.

---

## 2. Definitions

> **Def 3.1 (Nowhere Dense).** $A \subseteq X$ is **nowhere dense** if $\mathrm{int}(\overline{A}) = \emptyset$, i.e., $\overline{A}$ contains no open set.

> **Def 3.2 (Meagre / First Category).** $A$ is **meagre** (of the **first category**) if $A = \bigcup_{n=1}^\infty A_n$ with each $A_n$ nowhere dense.

> **Def 3.3 (Residual / Comeagre).** $A$ is **residual** if $X \setminus A$ is meagre.

> **Def 3.4 (Second Category).** $A$ is of the **second category** if it is not meagre.

```
Classification of subsets:
┌─────────────────────────────────────────────┐
│              All subsets of X               │
│                                             │
│  ┌──────── Meagre (1st category) ────────┐  │
│  │  Countable union of nowhere dense     │  │
│  │  "Thin" / "negligible"                │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  ┌──────── 2nd category ─────────────────┐  │
│  │  NOT a countable union of nwd sets    │  │
│  │  "Thick" / "substantial"              │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

---

## 3. Statement of the Baire Category Theorem

> **Theorem 3.5 (BCT, Version 1 — Intersection Form).** If $X$ is a complete metric space (or a locally compact Hausdorff space), and $\{G_n\}_{n=1}^\infty$ are dense open sets, then $\bigcap_{n=1}^\infty G_n$ is dense in $X$.

> **Theorem 3.5' (BCT, Version 2 — Union Form).** A complete metric space (or LCH space) is **not** meagre in itself. Equivalently, if $X = \bigcup_{n=1}^\infty F_n$ with each $F_n$ closed, then some $F_n$ has nonempty interior.

These are equivalent: take complements ($G_n$ dense open ↔ $X \setminus G_n$ nowhere dense closed).

---

## 4. Proof (Complete Metric Space Version)

**Claim:** $\bigcap_{n=1}^\infty G_n$ is dense, i.e., meets every nonempty open $U$.

*Proof:*
1. $G_1$ dense ⟹ $G_1 \cap U \neq \emptyset$. Pick $x_1 \in G_1 \cap U$. Choose $\epsilon_1 < 1$ with $\overline{B(x_1, \epsilon_1)} \subseteq G_1 \cap U$.
2. $G_2$ dense ⟹ $G_2 \cap B(x_1, \epsilon_1) \neq \emptyset$. Pick $x_2$, $\epsilon_2 < 1/2$ with $\overline{B(x_2, \epsilon_2)} \subseteq G_2 \cap B(x_1, \epsilon_1)$.
3. Inductively: $\overline{B(x_n, \epsilon_n)} \subseteq G_n \cap B(x_{n-1}, \epsilon_{n-1})$, $\epsilon_n < 1/n$.

The sequence $(x_n)$ is Cauchy ($\epsilon_n \to 0$), hence converges to some $x \in X$ (completeness!).

Then $x \in \overline{B(x_n, \epsilon_n)} \subseteq G_n$ for all $n$, and $x \in U$. So $x \in U \cap \bigcap G_n$. ∎

```
Nested balls:
U ⊇ B̄(x₁,ε₁) ⊇ B̄(x₂,ε₂) ⊇ B̄(x₃,ε₃) ⊇ ···
                                              → {x}
ε₁ > ε₂ > ε₃ > ··· → 0
Each B̄(xₙ,εₙ) ⊆ Gₙ
```

---

## 5. Classical Applications

### 5.1 Nowhere-Differentiable Continuous Functions Are "Generic"

> **Thm 3.6 (Banach, 1931).** The set of continuous functions $f : [0,1] \to \mathbb{R}$ that are **nowhere differentiable** is residual (comeagre) in $C([0,1])$ with the sup norm.

Implication: "most" continuous functions are nowhere differentiable!

### 5.2 $\mathbb{R} \ne \bigcup_{n} A_n$ (Nowhere Dense $A_n$)

In particular, $\mathbb{R} \neq \mathbb{Q}$: but more strongly, $\mathbb{Q}$ is meagre in $\mathbb{R}$ (countable union of singletons, each nowhere dense), while $\mathbb{R} \setminus \mathbb{Q}$ is residual.

### 5.3 Uniform Boundedness Principle

> **Thm 3.7 (Banach–Steinhaus).** If $(T_n)$ is a sequence of bounded linear operators $X \to Y$ ($X$ Banach) with $\sup_n \|T_n x\| < \infty$ for each $x$, then $\sup_n \|T_n\| < \infty$.

*Proof uses BCT:* $X = \bigcup_k \{x : \sup_n \|T_n x\| \leq k\}$. Each set is closed; by BCT, some has interior.

### 5.4 Open Mapping Theorem

> **Thm 3.8.** A surjective bounded linear operator between Banach spaces is an open map.

---

## 6. Baire Space (General Definition)

> **Def 3.9.** A topological space is a **Baire space** if countable intersections of dense open sets are dense (equivalently, it is not meagre in itself).

The BCT says: complete metric spaces and locally compact Hausdorff spaces are Baire spaces.

---

## 7. Summary Table

| Concept | Details |
|---------|---------|
| Nowhere dense | $\mathrm{int}(\overline{A}) = \emptyset$ |
| Meagre (1st category) | Countable union of nowhere dense sets |
| Residual (comeagre) | Complement of meagre |
| BCT | Complete metric / LCH ⟹ not meagre |
| Equivalent form | Countable intersection of dense open sets is dense |
| Applications | Banach–Steinhaus, open mapping, generic non-differentiability |

---

## 8. Quick Revision Questions

1. Define nowhere dense, meagre, and residual sets with examples.

2. State both forms of the Baire Category Theorem and prove they are equivalent.

3. Prove BCT for complete metric spaces using the nested balls argument.

4. Show that $\mathbb{Q}$ is meagre in $\mathbb{R}$. What does BCT imply about $\mathbb{R} \setminus \mathbb{Q}$?

5. State the Uniform Boundedness Principle and outline how BCT is used in its proof.

6. Explain why "most" continuous functions are nowhere differentiable (in the Baire category sense).

---

[← Previous: Complete Metric Spaces](02-complete-metric-spaces.md) | [Back to README](../README.md) | [Next: Compactness in Metric Spaces →](04-compactness-in-metric-spaces.md)
