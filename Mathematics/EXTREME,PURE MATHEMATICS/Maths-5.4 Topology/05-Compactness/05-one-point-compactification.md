# Chapter 5.5 — One-Point Compactification

[← Previous: Local Compactness](04-local-compactness.md) | [Back to README](../README.md) | [Next: Unit 6 — Separation Axioms →](../06-Separation-Axioms/01-hausdorff-and-T1-spaces.md)

---

## 1. Overview

The **one-point (Alexandroff) compactification** adds a single "point at infinity" to a non-compact space to make it compact. This construction is fundamental: it identifies $S^n$ as the compactification of $\mathbb{R}^n$, $\hat{\mathbb{C}}$ as the Riemann sphere, and appears throughout analysis and geometry.

---

## 2. Construction

Let $X$ be a topological space that is not compact. Define $X^* = X \cup \{\infty\}$ where $\infty \notin X$.

> **Def 5.1 (One-Point Compactification Topology).** The topology on $X^*$ consists of:
> 1. All open sets of $X$, plus
> 2. Sets of the form $(X \setminus C) \cup \{\infty\}$ where $C \subseteq X$ is compact and closed.

```
X* = X ∪ {∞}

Open sets in X*:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Type 1: U ⊆ X, U open in X

  ┌──────── X* ────────┐
  │  ┌── U ──┐         │
  │  │       │    ∞    │
  │  └───────┘         │
  └─────────────────────┘

Type 2: (X \ C) ∪ {∞}, C compact closed in X

  ┌──────── X* ────────┐
  │  ┌── C ──┐  ░░░░░░│
  │  │ closed │  ░░░░░░│
  │  │compact │  ░∞░░░░│
  │  └───────┘  ░░░░░░│
  └─────────────────────┘
  ░ = open neighbourhood of ∞
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Why This Works

> **Thm 5.2.** $X^*$ is a compact topological space. The inclusion $X \hookrightarrow X^*$ is an embedding (homeomorphism onto image).

**Proof sketch:**
1. **Topology axioms:** ∅ and $X^*$ are open. Unions/intersections close by checking both types.
2. **Compactness:** Let $\{U_\alpha\}$ cover $X^*$. Some $U_{\alpha_0}$ contains $\infty$, so $U_{\alpha_0} = (X \setminus C) \cup \{\infty\}$ with $C$ compact. The remaining $U_\alpha$'s cover $C$; extract a finite subcover since $C$ is compact. Together with $U_{\alpha_0}$, this gives a finite cover of $X^*$.
3. **Embedding:** The subspace topology on $X \subseteq X^*$ is exactly the original topology of $X$. ∎

---

## 4. When Is $X^*$ Hausdorff?

> **Thm 5.3.** $X^*$ is Hausdorff if and only if $X$ is locally compact Hausdorff.

*Proof:*
- ($\Leftarrow$) Separate two points of $X$ using Hausdorff of $X$. Separate $x \in X$ from $\infty$ using a compact neighbourhood $C$ of $x$: then $x \in \mathrm{int}(C)$ and $\infty \in (X \setminus C) \cup \{\infty\}$.
- ($\Rightarrow$) If $X^*$ is Hausdorff, then $X$ (as a subspace) is Hausdorff, and separating $x$ from $\infty$ yields a compact neighbourhood, giving local compactness. ∎

---

## 5. Classical Examples

| Space $X$ | One-point compactification $X^*$ |
|-----------|--------------------------------|
| $\mathbb{R}$ | $S^1$ (circle) |
| $\mathbb{R}^2$ | $S^2$ (sphere) |
| $\mathbb{R}^n$ | $S^n$ ($n$-sphere) |
| $\mathbb{C}$ | $\hat{\mathbb{C}}$ (Riemann sphere) |
| $(0,1)$ | $S^1$ |
| $\mathbb{Z}$ (discrete) | $\mathbb{Z} \cup \{\infty\}$ (converging sequence) |
| $\mathbb{N}$ (discrete) | Converging sequence $\{0, 1, 1/2, 1/3, \ldots\}$ |

### 5.1 $\mathbb{R}^2 \cup \{\infty\} \cong S^2$ via Stereographic Projection

```
        N (= ∞)
       /|\
      / | \
     /  |  \
    /   |   \
   /    |    \
  /     |     \
 ───────┼──────     ← equatorial plane ≅ ℝ²
  \     |     /
   \    |    /
    \   |   /
     \  |  /
      \ | /
       \|/
        S

  Stereographic projection from N:
  S² \ {N} → ℝ²   is a homeomorphism
  Adding ∞ ↔ N gives S² ≅ ℝ² ∪ {∞}
```

---

## 6. Uniqueness

> **Thm 5.4.** If $X$ is LCH and non-compact, then $X^*$ is the unique (up to homeomorphism) compact Hausdorff space containing $X$ as a dense subspace such that $X^* \setminus X$ is a single point.

---

## 7. Functoriality

A **proper** continuous map $f : X \to Y$ (preimages of compact sets are compact) extends to a continuous map $f^* : X^* \to Y^*$ by setting $f^*(\infty_X) = \infty_Y$.

```
  X ──f──→ Y
  ↓         ↓
  X* ─f*─→ Y*    (∞_X ↦ ∞_Y)
```

---

## 8. Comparison: One-Point vs Stone–Čech

| Feature | One-point $X^*$ | Stone–Čech $\beta X$ |
|---------|:-:|:-:|
| Points added | 1 | Potentially many |
| Hausdorff | Only if $X$ is LCH | If $X$ is Tychonoff |
| Universal property | For proper maps | For bounded continuous functions |
| Size | Minimal compactification | Maximal compactification |
| $X$ dense in? | ✓ | ✓ |

---

## 9. Summary Table

| Concept | Details |
|---------|---------|
| $X^* = X \cup \{\infty\}$ | Add one point |
| Topology | Open sets of $X$ + complements of compact closed sets |
| $X^*$ compact | Always |
| $X^*$ Hausdorff | ⟺ $X$ is LCH |
| $\mathbb{R}^n \cup \{\infty\}$ | $\cong S^n$ |
| Stereographic proj. | Exhibits the homeomorphism |
| Uniqueness | Unique minimal compactification (LCH case) |

---

## 10. Quick Revision Questions

1. Define the one-point compactification topology and verify the topology axioms.

2. Prove that $X^*$ is compact.

3. When is $X^*$ Hausdorff? State and sketch the proof.

4. Show that $\mathbb{R} \cup \{\infty\} \cong S^1$ using stereographic projection.

5. Compare the one-point compactification with the Stone–Čech compactification.

6. What is the one-point compactification of $\mathbb{N}$ (discrete)? Describe the topology explicitly.

---

[← Previous: Local Compactness](04-local-compactness.md) | [Back to README](../README.md) | [Next: Unit 6 — Separation Axioms →](../06-Separation-Axioms/01-hausdorff-and-T1-spaces.md)
