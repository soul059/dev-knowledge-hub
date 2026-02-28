# Chapter 7.3 — Lindelöf Spaces

[← Previous: Separable Spaces](02-separable-spaces.md) | [Back to README](../README.md) | [Next: Unit 8 — Metrics and Metric Topology →](../08-Metric-Spaces/01-metrics-and-metric-topology.md)

---

## 1. Overview

A **Lindelöf** space is one in which every open cover has a *countable* subcover — a weakening of compactness that nonetheless provides powerful results, especially in conjunction with separation axioms.

---

## 2. Definition

> **Def 3.1.** $X$ is **Lindelöf** if every open cover of $X$ has a countable subcover.

Equivalently: every open cover $\{U_\alpha\}_{\alpha \in A}$ admits a countable $A_0 \subseteq A$ with $\bigcup_{\alpha \in A_0} U_\alpha = X$.

```
Compactness hierarchy:
  Compact ──→ σ-compact ──→ Lindelöf
     ↓                         ↑
 Sequentially compact    2nd countable
```

---

## 3. Examples

| Space | Lindelöf? | Why |
|-------|:---------:|-----|
| Any compact space | ✓ | Finite subcover ⟹ countable subcover |
| $\mathbb{R}^n$ | ✓ | Second countable |
| 2nd countable spaces | ✓ | Always (Thm 3.2) |
| Sorgenfrey line $\mathbb{R}_\ell$ | ✓ | Direct proof using rationals |
| $\mathbb{R}_\ell \times \mathbb{R}_\ell$ | ✗ | Not even normal! |
| Discrete uncountable | ✗ | Cover by singletons has no countable subcover |
| $\omega_1$ (order top.) | ✗ | $\{[0,\alpha) : \alpha < \omega_1\}$ has no ctbl subcover |
| $\sigma$-compact spaces | ✓ | Union of countably many compact sets |

---

## 4. Key Theorems

### 4.1 Second Countable ⟹ Lindelöf

> **Thm 3.2.** Every second-countable space is Lindelöf.

*Proof:* Let $\mathcal{B} = \{B_n\}$ be a countable basis and $\{U_\alpha\}$ an open cover. Each $U_\alpha = \bigcup_n B_n$ (where indices come from $\mathcal{B}$). For each $B_n$ that is contained in some $U_\alpha$, choose one such $U_{\alpha(n)}$. This gives a countable subcover. ∎

### 4.2 Lindelöf + Regular ⟹ Normal

> **Thm 3.3.** A regular Lindelöf space is normal.

*Proof sketch:* Given disjoint closed $F_1, F_2$: for each $x \in F_1$, regularity gives $U_x$ with $x \in U_x \subseteq \overline{U_x} \subseteq X \setminus F_2$. The $U_x$'s cover $F_1$; Lindelöf gives countable subcover $\{U_n\}$. Similarly get $\{V_n\}$ covering $F_2$. Then "shrink" to make them disjoint:
$$U_n' = U_n \setminus \bigcup_{k=1}^n \overline{V_k}, \qquad V_n' = V_n \setminus \bigcup_{k=1}^n \overline{U_k}$$
Set $U = \bigcup U_n'$, $V = \bigcup V_n'$. ∎

### 4.3 Metric Spaces: Lindelöf ⟺ Separable ⟺ 2nd Countable

> **Thm 3.4.** For metric spaces: Lindelöf ⟺ separable ⟺ second countable.

---

## 5. σ-Compactness

> **Def 3.5.** $X$ is **σ-compact** if $X = \bigcup_{n=1}^\infty K_n$ with each $K_n$ compact.

$\sigma$-compact spaces are always Lindelöf. Examples: $\mathbb{R}^n = \bigcup_{n=1}^\infty \overline{B(0,n)}$.

```
σ-compact:
  X = K₁ ∪ K₂ ∪ K₃ ∪ ···

  ┌──K₁──┐ ┌──K₂──┐ ┌──K₃──┐
  │      │ │      │ │      │  ···
  └──────┘ └──────┘ └──────┘
```

---

## 6. Permanence Properties

| Property | Closed subspaces | Open subspaces | Products | Quotients |
|----------|:---:|:---:|:---:|:---:|
| Lindelöf | ✓ | ✓ (in regular spaces) | ✗ | ✓ |

**Key failure:** $\mathbb{R}_\ell$ is Lindelöf, but $\mathbb{R}_\ell \times \mathbb{R}_\ell$ is not.

---

## 7. Lindelöf Number

> **Def 3.6.** The **Lindelöf number** $L(X)$ is the smallest infinite cardinal $\kappa$ such that every open cover has a subcover of cardinality $\leq \kappa$.

- $L(X) = \aleph_0$ means $X$ is Lindelöf.
- $L(X) \leq |X|$ always holds.

---

## 8. Summary Table

| Concept | Details |
|---------|---------|
| Lindelöf | Every open cover → countable subcover |
| 2nd countable ⟹ Lindelöf | Always |
| Compact ⟹ Lindelöf | Finite ⟹ countable |
| Metric: Lindelöf ⟺ separable | ⟺ 2nd countable |
| Lindelöf + regular ⟹ normal | Via countable shrinking argument |
| σ-compact ⟹ Lindelöf | Union of countably many compact sets |
| Products | Not preserved in general |

---

## 9. Quick Revision Questions

1. Define Lindelöf spaces. How does this relate to compactness?

2. Prove that second-countable spaces are Lindelöf.

3. Prove that a regular Lindelöf space is normal. (Outline the "shrinking" argument.)

4. Show the equivalence of Lindelöf, separable, and 2nd countable for metric spaces.

5. Prove that the Sorgenfrey line is Lindelöf, but the Sorgenfrey plane is not.

6. Define σ-compactness and prove it implies the Lindelöf property.

---

[← Previous: Separable Spaces](02-separable-spaces.md) | [Back to README](../README.md) | [Next: Unit 8 — Metrics and Metric Topology →](../08-Metric-Spaces/01-metrics-and-metric-topology.md)
