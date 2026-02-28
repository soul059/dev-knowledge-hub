# Chapter 8.4 — Compactness in Metric Spaces

[← Previous: Baire Category Theorem](03-baire-category-theorem.md) | [Back to README](../README.md) | [Next: Unit 9 — Homotopy →](../09-Algebraic-Topology-Intro/01-homotopy.md)

---

## 1. Overview

In metric spaces, the various notions of compactness collapse into a single equivalence. This chapter ties together results from earlier units and adds the Arzelà–Ascoli theorem — the key compactness criterion for function spaces.

---

## 2. The Grand Equivalence

> **Theorem 4.1.** For a metric space $(X, d)$, the following are equivalent:
> 1. $X$ is **compact** (every open cover has a finite subcover).
> 2. $X$ is **sequentially compact** (every sequence has a convergent subsequence).
> 3. $X$ is **limit point compact** (every infinite subset has a limit point).
> 4. $X$ is **countably compact** (every countable open cover has a finite subcover).
> 5. $X$ is **complete and totally bounded**.

```
In metric spaces:
┌──────────────────────────────────────────┐
│                                          │
│  Compact ⟺ Seq. compact ⟺ Lim pt compact│
│      ⟺ Countably compact                │
│      ⟺ Complete + Totally bounded        │
│                                          │
│  ALL EQUIVALENT!                         │
└──────────────────────────────────────────┘
```

---

## 3. Proof Outline of Key Implications

### 3.1 Compact ⟹ Complete + Totally Bounded

**Complete:** Cauchy sequence in compact space has a convergent subsequence (by sequential compactness); Cauchy + convergent subseq ⟹ convergent.

**Totally bounded:** If not, $\exists \epsilon > 0$ such that no finite $\epsilon$-net covers $X$. Build a sequence $x_1, x_2, \ldots$ with $d(x_i, x_j) \geq \epsilon$ for $i \neq j$. No convergent subsequence — contradicting sequential compactness.

### 3.2 Complete + Totally Bounded ⟹ Sequentially Compact

Given $(x_n)$: total boundedness gives a nested sequence of $\epsilon_k$-balls ($\epsilon_k \to 0$) each containing infinitely many terms. Diagonal argument extracts a Cauchy subsequence. Completeness gives convergence.

### 3.3 Sequentially Compact ⟹ Compact

Uses the **Lebesgue number lemma** and total boundedness:
1. Every open cover has a Lebesgue number $\delta > 0$ (proved using sequential compactness).
2. Cover $X$ with finitely many balls of radius $\delta/2$ (total boundedness).
3. Each ball lies in some cover element. Finite collection suffices.

---

## 4. Lebesgue Number Lemma

> **Lemma 4.2.** Let $(X, d)$ be a sequentially compact (equivalently, compact) metric space and $\{U_\alpha\}$ an open cover. Then there exists $\delta > 0$ (the **Lebesgue number**) such that every subset of diameter $< \delta$ is contained in some $U_\alpha$.

*Proof:* Suppose not. Then $\forall n$, $\exists$ set of diameter $< 1/n$ not contained in any $U_\alpha$. Pick $x_n$ from such a set. Sequential compactness: $x_{n_k} \to x$. Then $x \in U_\alpha$ for some $\alpha$. For large $k$, the $1/n_k$-ball around $x_{n_k}$ lies in $U_\alpha$ — contradiction. ∎

---

## 5. Compactness in $\mathbb{R}^n$: Heine–Borel (Recap)

> **Thm 4.3 (Heine–Borel).** $A \subseteq \mathbb{R}^n$ is compact iff $A$ is closed and bounded.

In general metric spaces, "closed and bounded" does **not** imply compact ($\ell^\infty$: the closed unit ball is closed and bounded but not compact).

---

## 6. Arzelà–Ascoli Theorem

The Arzelà–Ascoli theorem characterises compact subsets of $C(X)$ — the space of continuous functions with the sup metric.

> **Def 4.4.** A family $\mathcal{F} \subseteq C(X, \mathbb{R})$ is:
> - **Pointwise bounded:** $\forall x \in X$, $\sup_{f \in \mathcal{F}} |f(x)| < \infty$.
> - **Equicontinuous:** $\forall \epsilon > 0$, $\forall x \in X$, $\exists$ neighbourhood $U$ of $x$ s.t. $|f(y) - f(x)| < \epsilon$ $\forall f \in \mathcal{F}$, $\forall y \in U$.

> **Thm 4.5 (Arzelà–Ascoli).** Let $X$ be a compact space. A subset $\mathcal{F} \subseteq C(X, \mathbb{R}^n)$ has compact closure iff $\mathcal{F}$ is **pointwise bounded** and **equicontinuous**.

```
Arzelà–Ascoli summary:
┌────────────────────────────────────────┐
│  F̄ compact in C(X)                    │
│                                        │
│  ⟺  F is { pointwise bounded          │
│           { equicontinuous             │
│                                        │
│  Consequence: every sequence in F      │
│  has a uniformly convergent subseq.    │
└────────────────────────────────────────┘
```

### Applications:
- **Peano's existence theorem** for ODEs: solutions form an equicontinuous family.
- **Montel's theorem** in complex analysis: normal families.
- **Compactness of integral operators** in functional analysis.

---

## 7. Total Boundedness — Further Properties

> **Prop 4.6.** 
> - Every subset of a totally bounded space is totally bounded.
> - $X$ totally bounded ⟹ $X$ separable.
> - $X$ complete + separable ⟹ $X$ is a Polish space.

| Property | Compact | Totally bounded | Complete |
|----------|:-------:|:---------------:|:--------:|
| Every sequence has convergent subseq. | ✓ | ✗ | ✗ |
| Cauchy subseq. exists | ✓ | ✓ | — |
| Cauchy sequences converge | ✓ | ✗ | ✓ |
| Closed + bounded ⟹ compact | Only in $\mathbb{R}^n$ | — | — |

---

## 8. Summary Table

| Concept | Details |
|---------|---------|
| Metric compactness | 5 equivalent conditions |
| Lebesgue number | Min "resolution" of a cover; exists in compact metric spaces |
| Heine–Borel | Compact ⟺ closed + bounded (in $\mathbb{R}^n$ only) |
| Arzelà–Ascoli | Compact ⊂ $C(X)$ ⟺ pointwise bounded + equicontinuous |
| Complete + totally bounded | ⟺ compact (in metric spaces) |

---

## 9. Quick Revision Questions

1. State the five equivalent characterisations of compactness in metric spaces.

2. Prove the Lebesgue Number Lemma.

3. Why does "closed and bounded" not imply compact in general metric spaces? Give an example.

4. State the Arzelà–Ascoli theorem. Define equicontinuity and pointwise boundedness.

5. Outline the proof: complete + totally bounded ⟹ sequentially compact.

6. How is Arzelà–Ascoli used in the existence theory for ODEs?

---

[← Previous: Baire Category Theorem](03-baire-category-theorem.md) | [Back to README](../README.md) | [Next: Unit 9 — Homotopy →](../09-Algebraic-Topology-Intro/01-homotopy.md)
