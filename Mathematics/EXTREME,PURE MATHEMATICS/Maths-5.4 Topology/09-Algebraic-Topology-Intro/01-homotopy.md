# Chapter 9.1 — Homotopy

[← Previous: Compactness in Metric Spaces](../08-Metric-Spaces/04-compactness-in-metric-spaces.md) | [Back to README](../README.md) | [Next: The Fundamental Group →](02-fundamental-group.md)

---

## 1. Overview

**Homotopy** formalises the idea of "continuously deforming" one map into another. It is the foundational concept of algebraic topology, enabling us to classify spaces not by exact shape but by deformation type. Two spaces that can be continuously deformed into each other (homotopy equivalent) are considered "the same" from the viewpoint of algebraic topology.

---

## 2. Definitions

### 2.1 Homotopy of Maps

> **Def 1.1.** Let $f, g : X \to Y$ be continuous. A **homotopy** from $f$ to $g$ is a continuous map $H : X \times [0,1] \to Y$ with
> $$H(x, 0) = f(x), \qquad H(x, 1) = g(x) \qquad \forall\, x \in X.$$
> We write $f \simeq g$ and say $f$ is **homotopic** to $g$.

```
Homotopy H : X × [0,1] → Y

t=1  ── g(x) ──────────────────
     │         H(x,t)
     │      intermediate maps
     │
t=0  ── f(x) ──────────────────

H continuously deforms f into g as t goes from 0 to 1.
```

### 2.2 Relative Homotopy

> **Def 1.2.** If $A \subseteq X$ and $f|_A = g|_A$, a homotopy **relative to $A$** (rel $A$) satisfies $H(a, t) = f(a) = g(a)$ for all $a \in A$, $t \in [0,1]$.

### 2.3 Path Homotopy

> **Def 1.3.** Two paths $\gamma_0, \gamma_1 : [0,1] \to X$ with the same endpoints ($\gamma_0(0) = \gamma_1(0)$, $\gamma_0(1) = \gamma_1(1)$) are **path homotopic** if there is a homotopy rel $\{0, 1\}$.

```
Path homotopy (endpoints fixed):
         ╭── γ₀ ──╮
    x₀ ──┤         ├── x₁
         ╰── γ₁ ──╯

  Intermediate paths: γₜ = H(·, t)
  All share endpoints x₀ and x₁
```

---

## 3. Homotopy Is an Equivalence Relation

> **Thm 1.4.** $\simeq$ is an equivalence relation on $C(X, Y)$.

*Proof:*
- **Reflexive:** $H(x,t) = f(x)$ (constant homotopy).
- **Symmetric:** $\bar{H}(x,t) = H(x, 1-t)$.
- **Transitive:** Concatenate: $H(x,t) = \begin{cases} H_1(x, 2t) & 0 \leq t \leq 1/2 \\ H_2(x, 2t-1) & 1/2 \leq t \leq 1 \end{cases}$

(Continuity of the concatenation follows from the pasting lemma.) ∎

The equivalence class $[f]$ is the **homotopy class** of $f$.

---

## 4. Homotopy Equivalence of Spaces

> **Def 1.5.** Spaces $X$ and $Y$ are **homotopy equivalent** ($X \simeq Y$) if there exist continuous maps $f : X \to Y$ and $g : Y \to X$ with $g \circ f \simeq \mathrm{id}_X$ and $f \circ g \simeq \mathrm{id}_Y$.

Such $f$ (and $g$) are called **homotopy equivalences**.

> **Def 1.6.** $X$ is **contractible** if $X \simeq \{*\}$ (homotopy equivalent to a point).

```
Homotopy equivalence:
  f : X → Y,  g : Y → X
  
  g∘f ≃ idₓ         f∘g ≃ id_Y
  X ──f──→ Y        Y ──g──→ X
  X ──g∘f──→ X ≃ id    Y ──f∘g──→ Y ≃ id
```

---

## 5. Key Examples

| Spaces | Homotopy equivalent? | Notes |
|--------|:---:|-------|
| $\mathbb{R}^n$ and $\{0\}$ | ✓ | $\mathbb{R}^n$ is contractible via $H(x,t) = (1-t)x$ |
| $S^1$ and $\mathbb{R}^2 \setminus \{0\}$ | ✓ | Retract via $x \mapsto x/\|x\|$ |
| $D^2$ and $\{0\}$ | ✓ | Disk is contractible |
| $S^1$ and $\{*\}$ | ✗ | $\pi_1(S^1) = \mathbb{Z} \neq 0$ |
| $S^n$ and $S^m$ ($n \neq m$) | ✗ | Different homotopy groups |
| Möbius band and $S^1$ | ✓ | Deformation retract |
| Torus and $S^1 \vee S^1$ | ✗ | Different $\pi_1$ |

### Deformation Retract

> **Def 1.7.** $A \subseteq X$ is a **deformation retract** of $X$ if there is a homotopy $H : X \times [0,1] \to X$ with $H(x,0) = x$, $H(x,1) \in A$, and $H(a,t) = a$ for all $a \in A$.

Equivalently: $\mathrm{id}_X \simeq r$ rel $A$, where $r : X \to A$ is a retraction.

---

## 6. Homotopy Invariants

A property or algebraic object attached to spaces is a **homotopy invariant** if homotopy equivalent spaces share it.

| Invariant | Type |
|-----------|------|
| Path-connectedness | Yes |
| Fundamental group $\pi_1$ | Yes |
| Higher homotopy groups $\pi_n$ | Yes |
| Homology groups $H_n$ | Yes |
| Euler characteristic $\chi$ | Yes |
| Compactness | No |
| Hausdorff property | No |
| Dimension | No (in general) |

---

## 7. Null-Homotopy and Simply Connected Spaces

> **Def 1.8.** A map $f : X \to Y$ is **null-homotopic** if $f \simeq c$ for some constant map $c$.

> **Def 1.9.** $X$ is **simply connected** if it is path-connected and every loop is null-homotopic (i.e., $\pi_1(X) = 0$).

---

## 8. Summary Table

| Concept | Definition |
|---------|-----------|
| Homotopy $f \simeq g$ | Continuous deformation $H : X \times I \to Y$ |
| Path homotopy | Endpoints fixed ($\text{rel } \{0,1\}$) |
| Homotopy equivalence | $X \simeq Y$ via $f, g$ with $gf \simeq \text{id}$, $fg \simeq \text{id}$ |
| Contractible | $X \simeq \{*\}$ |
| Deformation retract | $A \hookrightarrow X$ with $\text{id}_X \simeq r$ rel $A$ |
| Null-homotopic | Homotopic to a constant |
| Simply connected | Path-connected + $\pi_1 = 0$ |

---

## 9. Quick Revision Questions

1. Define homotopy of maps. Prove it is an equivalence relation.

2. Define homotopy equivalence of spaces. Show that $\mathbb{R}^n$ is contractible.

3. What is a deformation retract? Show that $S^1$ is a deformation retract of $\mathbb{R}^2 \setminus \{0\}$.

4. List five homotopy invariants and two properties that are not homotopy invariant.

5. Define null-homotopic and simply connected. Give examples.

6. Are the torus and $S^1$ homotopy equivalent? Justify.

---

[← Previous: Compactness in Metric Spaces](../08-Metric-Spaces/04-compactness-in-metric-spaces.md) | [Back to README](../README.md) | [Next: The Fundamental Group →](02-fundamental-group.md)
