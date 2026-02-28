# Chapter 5.4 — Local Compactness

[← Previous: Limit Point Compactness](03-limit-point-compactness.md) | [Back to README](../README.md) | [Next: One-Point Compactification →](05-one-point-compactification.md)

---

## 1. Overview

Local compactness is a weakening of compactness that retains many useful properties. It plays a crucial role in the one-point compactification, the Arzelà–Ascoli theorem, and the theory of locally compact groups.

---

## 2. Definitions

> **Def 4.1 (Locally Compact at a Point).** $X$ is **locally compact at $x$** if there exists a compact set $C \subseteq X$ that contains a neighbourhood of $x$.

> **Def 4.2 (Locally Compact).** $X$ is **locally compact** if it is locally compact at every point.

Equivalently (for Hausdorff spaces): every point has an open neighbourhood whose closure is compact.

> **Def 4.3 (Locally Compact Hausdorff — LCH).** A space that is both locally compact and Hausdorff. This is the setting where the theory works best.

---

## 3. ASCII Diagram — Local Compactness Illustrated

```
Point x in space X:
┌──────────────────────────────── X ──────────────────────────────┐
│                                                                  │
│         ┌───────── C (compact) ─────────┐                       │
│         │                                │                       │
│         │      ┌── U (open) ──┐         │                       │
│         │      │              │         │                       │
│         │      │     • x      │         │                       │
│         │      │              │         │                       │
│         │      └──────────────┘         │                       │
│         │                                │                       │
│         └────────────────────────────────┘                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
   x ∈ U ⊆ C ⊆ X,  U open, C compact
```

---

## 4. Key Examples

| Space | Locally Compact? | Hausdorff? | LCH? |
|-------|:-:|:-:|:-:|
| $\mathbb{R}^n$ | ✓ | ✓ | ✓ |
| Any compact space | ✓ | depends | — |
| Discrete space | ✓ | ✓ | ✓ |
| $\mathbb{Q}$ (usual) | ✗ | ✓ | ✗ |
| $\ell^2$ | ✗ | ✓ | ✗ |
| $\mathbb{R}^\omega$ (product) | ✗ | ✓ | ✗ |

**Why is $\mathbb{Q}$ not locally compact?** Every open neighbourhood of a rational contains irrationals, and no compact subset of $\mathbb{Q}$ has interior (since every compact subset of $\mathbb{Q}$ is nowhere dense in $\mathbb{Q}$ — this follows from the Baire category theorem applied to $\mathbb{R}$).

---

## 5. Key Theorems

### 5.1 Open Subsets of LCH Spaces

> **Thm 4.4.** An open subspace of a locally compact Hausdorff space is locally compact Hausdorff.

*Proof:* Let $U$ be open in LCH $X$, and $x \in U$. There exists open $V$ with $x \in V \subseteq \overline{V} \subseteq X$, $\overline{V}$ compact. Then $V \cap U$ is open and $\overline{V \cap U} \cap U$ gives a compact neighbourhood of $x$ in (the subspace) $U$. ∎

### 5.2 Closed Subsets of LC Spaces

> **Thm 4.5.** A closed subspace of a locally compact space is locally compact.

### 5.3 Products

> **Thm 4.6.** A **finite** product of locally compact spaces is locally compact.

*Proof:* If $C_i$ is a compact neighbourhood of $x_i$ in $X_i$ for $i = 1,\ldots,n$, then $C_1 \times \cdots \times C_n$ is a compact neighbourhood of $(x_1,\ldots,x_n)$ in $X_1 \times \cdots \times X_n$. ∎

**Warning:** An infinite product of locally compact non-compact spaces is generally **not** locally compact (e.g., $\mathbb{R}^\omega$).

### 5.4 Quotient of LC Spaces

> **Thm 4.7.** An open quotient map preserves local compactness.

---

## 6. Regularity Lemma for LCH Spaces

> **Lemma 4.8.** Let $X$ be LCH. For any $x \in U$ (open), there exists open $V$ with $x \in V \subseteq \overline{V} \subseteq U$, $\overline{V}$ compact.

```
Nesting for LCH:
x ∈ V ⊆ V̄ ⊆ U
      ↑    ↑
    open  compact
```

This shows that LCH spaces are **regular** (T₃). Combined with other arguments, one can show they are even **completely regular** (T₃½ / Tychonoff).

---

## 7. Locally Compact Groups (Preview)

A **locally compact group** is a topological group that is locally compact and Hausdorff. Examples: $\mathbb{R}^n$, $\mathrm{GL}_n(\mathbb{R})$, $S^1$, $p$-adic numbers $\mathbb{Q}_p$. On such groups one can define the **Haar measure**, enabling integration and harmonic analysis.

---

## 8. Summary Table

| Property | Details |
|----------|---------|
| Locally compact | $\forall x$, $\exists$ compact nbhd |
| LCH | Locally compact + Hausdorff; the "right" setting |
| Open subsets | LCH is preserved |
| Closed subsets | LC is preserved |
| Finite products | LC is preserved |
| Infinite products | LC need not be preserved |
| LCH ⟹ T₃½ | Every LCH space is completely regular |

---

## 9. Quick Revision Questions

1. Define local compactness. Give three examples of LCH spaces and one non-example.

2. Prove that an open subspace of an LCH space is locally compact.

3. Why is $\mathbb{Q}$ not locally compact?

4. State and prove the regularity lemma (Lemma 4.8) for LCH spaces.

5. Is an arbitrary product of locally compact spaces locally compact? Justify.

6. What additional structure does a locally compact group carry?

---

[← Previous: Limit Point Compactness](03-limit-point-compactness.md) | [Back to README](../README.md) | [Next: One-Point Compactification →](05-one-point-compactification.md)
