# Chapter 5.3 — Limit Point Compactness and Sequential Compactness

[← Previous: Compact Subsets of ℝⁿ](02-compact-subsets-of-Rn.md) | [Back to README](../README.md) | [Next: Local Compactness →](04-local-compactness.md)

---

## 1. Overview

There are several notions related to compactness. **Limit point compactness** (every infinite set has a limit point) and **sequential compactness** (every sequence has a convergent subsequence) coincide with compactness in metric spaces but diverge in general topological spaces.

---

## 2. Definitions

> **Def 3.1 (Limit Point Compact / Bolzano-Weierstrass Property).** $X$ is **limit point compact** if every infinite subset of $X$ has a limit point in $X$.

> **Def 3.2 (Sequentially Compact).** $X$ is **sequentially compact** if every sequence in $X$ has a convergent subsequence.

> **Def 3.3 (Countably Compact).** $X$ is **countably compact** if every countable open cover has a finite subcover.

---

## 3. Relationships

```
Implication Diagram:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Compact ──→ Countably compact ──→ Limit point compact
     │              │
     │              │  (in T₁ spaces: ←→)
     │              │
     ▼              ▼
  Sequentially compact
     │
     ▼
  Limit point compact

  In METRIC SPACES:
  Compact ⟺ Sequentially compact ⟺ Limit point compact
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

| Implication | Holds in general? | Holds in metric spaces? |
|-------------|:---:|:---:|
| Compact ⟹ seq. compact | ✗ | ✓ |
| Seq. compact ⟹ compact | ✗ | ✓ |
| Compact ⟹ limit point compact | ✓ | ✓ |
| Limit point compact ⟹ compact | ✗ | ✓ |
| Compact ⟹ countably compact | ✓ | ✓ |
| Countably compact ⟹ compact | ✗ | ✓ |

---

## 4. Proofs of Key Implications

### 4.1 Compact ⟹ Limit Point Compact

**Proof:** Let $A \subseteq X$ be infinite. Suppose $A$ has no limit point. Then $A$ is closed, and for each $x \in X$, there's an open $U_x$ with $U_x \cap A \subseteq \{x\}$. The $U_x$'s cover $X$. A finite subcover can contain at most finitely many points of $A$, contradicting $A$ infinite. ∎

### 4.2 In Metric Spaces: Limit Point Compact ⟹ Sequentially Compact

**Proof sketch:** Let $(x_n)$ be a sequence. If it has finitely many distinct values, some value repeats infinitely (constant subsequence). If infinitely many distinct values, the set $\{x_n\}$ has a limit point $p$ (by limit point compactness). Construct a subsequence converging to $p$ by picking $x_{n_k}$ within distance $1/k$ of $p$. ∎

### 4.3 In Metric Spaces: Sequentially Compact ⟹ Compact

This requires the Lebesgue number lemma and total boundedness. The key steps:
1. Seq. compact ⟹ totally bounded (every $\epsilon$-net is finite).
2. Seq. compact ⟹ Lebesgue number exists.
3. Combine: cover with Lebesgue number $\delta$, then finitely many balls of radius $\delta$ cover the space.

---

## 5. Equivalence in Metric Spaces

> **Thm 3.4.** For a metric space $(X, d)$, the following are equivalent:
> 1. $X$ is compact.
> 2. $X$ is sequentially compact.
> 3. $X$ is limit point compact.
> 4. $X$ is complete and totally bounded.
> 5. $X$ is countably compact.

---

## 6. Counterexamples in General Spaces

### 6.1 Limit Point Compact but Not Compact

$\omega_1 = [0, \omega_1)$ (the ordinals less than the first uncountable ordinal, with the order topology) is limit point compact but **not** compact (the cover by initial segments $[0, \alpha)$ has no finite subcover).

### 6.2 Sequentially Compact but Not Compact

$\{0,1\}^{[0,1]}$ (the product of uncountably many copies of $\{0,1\}$) considered with a non-product topology can exhibit this. More concretely, $\omega_1$ with the order topology is sequentially compact but not compact.

### 6.3 Compact but Not Sequentially Compact

$\{0,1\}^{[0,1]}$ with the **product topology** is compact (Tychonoff) but **not** sequentially compact.

---

## 7. Summary Table

| Concept | Definition |
|---------|-----------|
| Compact | Every open cover → finite subcover |
| Limit point compact | Every infinite set has a limit point |
| Sequentially compact | Every sequence has convergent subsequence |
| Countably compact | Every countable open cover → finite subcover |
| Metric space equivalence | All four are equivalent |
| General spaces | Strict implications with counterexamples |

---

## 8. Quick Revision Questions

1. Define limit point compactness and prove that compactness implies it.

2. Give an example of a space that is limit point compact but not compact.

3. Prove the equivalence of compactness and sequential compactness for metric spaces (outline the key steps).

4. Is $\{0,1\}^{\mathbb{R}}$ (product topology) sequentially compact? Explain.

5. State the equivalence theorem for metric spaces (5 equivalent conditions).

6. Why do these notions diverge in non-metrizable spaces?

---

[← Previous: Compact Subsets of ℝⁿ](02-compact-subsets-of-Rn.md) | [Back to README](../README.md) | [Next: Local Compactness →](04-local-compactness.md)
