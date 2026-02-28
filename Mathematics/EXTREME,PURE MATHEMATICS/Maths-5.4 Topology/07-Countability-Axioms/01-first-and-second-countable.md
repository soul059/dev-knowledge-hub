# Chapter 7.1 — First and Second Countable Spaces

[← Previous: Tietze Extension Theorem](../06-Separation-Axioms/04-tietze-extension-theorem.md) | [Back to README](../README.md) | [Next: Separable Spaces →](02-separable-spaces.md)

---

## 1. Overview

The **countability axioms** restrict the "size" of the topology. Spaces satisfying these axioms behave well with respect to sequences, are often metrizable, and are amenable to classical analysis.

---

## 2. Definitions

> **Def 1.1 (First Countable).** $X$ is **first countable** if every point has a countable neighbourhood basis (local basis).
>
> That is, $\forall x \in X$, $\exists$ countable collection $\{B_n\}_{n=1}^\infty$ of open sets containing $x$ such that every open $U \ni x$ satisfies $B_n \subseteq U$ for some $n$.

> **Def 1.2 (Second Countable).** $X$ is **second countable** if the topology has a countable basis (a countable collection $\mathcal{B}$ of open sets such that every open set is a union of members of $\mathcal{B}$).

**Relation:** Second countable ⟹ First countable (take the basis elements containing $x$).

---

## 3. ASCII Diagram — Local Basis

```
First Countable at x:
┌──────────────── X ────────────────┐
│                                    │
│  x ∈ B₁ ⊆ B₂ ⊆ B₃ ⊆ ...        │
│                                    │
│  For every open U ∋ x:           │
│     ┌──── U ────┐                 │
│     │  ┌─Bₙ─┐  │                 │
│     │  │  x  │  │                 │
│     │  └─────┘  │                 │
│     └───────────┘                 │
│  Some Bₙ "fits inside" U         │
└────────────────────────────────────┘
```

---

## 4. Key Examples

| Space | 1st countable | 2nd countable |
|-------|:---:|:---:|
| $\mathbb{R}^n$ (standard) | ✓ | ✓ (basis: open balls with rational center and radius) |
| Metric spaces | ✓ | depends |
| Discrete (countable) | ✓ | ✓ |
| Discrete (uncountable) | ✓ | ✗ |
| Sorgenfrey line $\mathbb{R}_\ell$ | ✓ | ✗ |
| $\mathbb{R}^\omega$ (product) | ✓ | ✓ |
| $\mathbb{R}^{\mathbb{R}}$ (product) | ✗ | ✗ |
| Uncountable product of $\mathbb{R}$'s | ✗ | ✗ |
| Cocountable on uncountable set | ✗ | ✗ |

---

## 5. The Power of First Countability: Sequences Suffice

> **Thm 1.3.** In a first-countable space:
> 1. $x \in \overline{A}$ iff there is a sequence in $A$ converging to $x$.
> 2. $f : X \to Y$ is continuous iff it preserves convergent sequences.
> 3. $X$ is compact iff every sequence has a convergent subsequence (when also Lindelöf).

Without first countability, nets or filters are required.

*Proof of (1):* ($\Leftarrow$) Clear. ($\Rightarrow$) Let $\{B_n\}$ be a countable local basis at $x$, and WLOG $B_1 \supseteq B_2 \supseteq \cdots$ (replace $B_n$ by $B_1 \cap \cdots \cap B_n$). Pick $a_n \in A \cap B_n$ (possible since $x \in \overline{A}$). Then $a_n \to x$. ∎

---

## 6. Properties of Second Countable Spaces

> **Thm 1.4.** Second countable spaces satisfy:
> 1. Separable (has a countable dense subset).
> 2. Lindelöf (every open cover has a countable subcover).
> 3. If also T₃ (regular), then normal and metrizable (Urysohn Metrization Theorem).

```
Implications (general spaces):
  2nd countable ──→ 1st countable
       │
       ├──→ Separable
       │
       └──→ Lindelöf

In METRIC SPACES, all three are equivalent:
  2nd countable ⟺ Separable ⟺ Lindelöf
```

---

## 7. Permanence Properties

| Property | Subspaces | Countable products | Quotients |
|----------|:-:|:-:|:-:|
| 1st countable | ✓ | ✓ | ✗ |
| 2nd countable | ✓ | ✓ | ✗ |

**Note:** Uncountable products of first/second countable spaces are generally **not** first/second countable (the product topology has too few basic open sets that fit into a countable basis).

---

## 8. Counting Argument: Why Sorgenfrey Line Is Not Second Countable

The Sorgenfrey line $\mathbb{R}_\ell$ has basis $\{[a,b) : a < b\}$. Suppose $\mathcal{B}$ is a countable basis. For each $x \in \mathbb{R}$, $[x, x+1)$ must contain some $B_x \in \mathcal{B}$ with $x \in B_x \subseteq [x, x+1)$. The map $x \mapsto B_x$ is injective (since $x = \min B_x$). But $\mathcal{B}$ is countable and $\mathbb{R}$ is uncountable — contradiction. ∎

---

## 9. Summary Table

| Concept | Definition | Key consequence |
|---------|-----------|----------------|
| 1st countable | Countable local basis at each point | Sequences detect closure, continuity |
| 2nd countable | Countable basis for topology | ⟹ separable, Lindelöf; +T₃ ⟹ metrizable |
| Metric spaces | Always 1st countable | 2nd ctbl ⟺ separable ⟺ Lindelöf |
| Products | Countable products preserve both | Uncountable products may fail |

---

## 10. Quick Revision Questions

1. Define first countable and second countable. How are they related?

2. Prove that in a first-countable space, $x \in \overline{A}$ iff a sequence in $A$ converges to $x$.

3. Show that every second-countable space is separable and Lindelöf.

4. Prove that the Sorgenfrey line is first countable but not second countable.

5. State the Urysohn Metrization Theorem. What role does second countability play?

6. Give an example of a first-countable space that is not second countable.

---

[← Previous: Tietze Extension Theorem](../06-Separation-Axioms/04-tietze-extension-theorem.md) | [Back to README](../README.md) | [Next: Separable Spaces →](02-separable-spaces.md)
