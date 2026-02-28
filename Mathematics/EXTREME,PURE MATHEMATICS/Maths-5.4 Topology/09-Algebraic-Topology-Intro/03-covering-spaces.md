# Chapter 9.3 — Covering Spaces

[← Previous: The Fundamental Group](02-fundamental-group.md) | [Back to README](../README.md)

---

## 1. Overview

**Covering spaces** are the geometric counterpart of the fundamental group. They provide the cleanest proofs that $\pi_1(S^1) \cong \mathbb{Z}$, enable computation of fundamental groups via deck transformations, and connect algebra (subgroups of $\pi_1$) with topology (covering spaces).

---

## 2. Definitions

> **Def 3.1 (Covering Map).** A surjective continuous map $p : \tilde{X} \to X$ is a **covering map** if every $x \in X$ has an open neighbourhood $U$ such that $p^{-1}(U)$ is a disjoint union of open sets in $\tilde{X}$, each mapped homeomorphically onto $U$ by $p$.
>
> The sets $\tilde{U}_\alpha$ are called **sheets** over $U$, and $U$ is **evenly covered**.

```
Covering space (3-fold):

  p⁻¹(U):   ┌─Ũ₁─┐  ┌─Ũ₂─┐  ┌─Ũ₃─┐     X̃
             │     │  │     │  │     │
             └──┬──┘  └──┬──┘  └──┬──┘
                │p       │p       │p
                ▼        ▼        ▼
             ┌────────── U ──────────┐     X
             │         • x          │
             └───────────────────────┘

  Each Ũᵢ ≅ U via p (homeomorphism)
  p⁻¹(U) = Ũ₁ ⊔ Ũ₂ ⊔ Ũ₃
```

> **Def 3.2.** The **fibre** over $x$ is $p^{-1}(x)$. If $X$ is connected, all fibres have the same cardinality, called the **degree** (or **number of sheets**).

---

## 3. Classical Examples

| Covering $\tilde{X} \to X$ | Degree | Description |
|---------------------------|:------:|-------------|
| $\mathbb{R} \to S^1$, $t \mapsto e^{2\pi i t}$ | $\infty$ | Universal cover of the circle |
| $S^1 \to S^1$, $z \mapsto z^n$ | $n$ | $n$-fold cover |
| $S^n \to \mathbb{R}P^n$, $x \mapsto [x]$ | 2 | Antipodal quotient |
| $\mathbb{R}^2 \to T^2$ | $\infty$ | Universal cover of torus |
| $S^2 \to S^2/\sim$ (antipodal) | 2 | — |

### 3.1 The Helix Covering $\mathbb{R} \to S^1$

```
  X̃ = ℝ (the helix "unrolled"):
  ···─────0─────1─────2─────3─────···
           ↓p    ↓p    ↓p    ↓p
  X = S¹: ──→──→──→──→── (circle)
  
  p(t) = e^{2πit}
  p⁻¹(1) = ℤ = {..., -1, 0, 1, 2, ...}
```

---

## 4. Lifting Properties

### 4.1 Path Lifting

> **Thm 3.3 (Path Lifting).** Let $p : \tilde{X} \to X$ be a covering, $\gamma : [0,1] \to X$ a path, and $\tilde{x}_0 \in p^{-1}(\gamma(0))$. Then there exists a **unique** lift $\tilde{\gamma} : [0,1] \to \tilde{X}$ with $p \circ \tilde{\gamma} = \gamma$ and $\tilde{\gamma}(0) = \tilde{x}_0$.

```
Path lifting:
  X̃:  x̃₀ ──── γ̃ ────→ x̃₁     (unique lift)
       │                  │
       │p                 │p
       ▼                  ▼
  X:   x₀ ──── γ ────→ x₁
```

### 4.2 Homotopy Lifting

> **Thm 3.4 (Homotopy Lifting).** If $H : [0,1] \times [0,1] \to X$ is a homotopy with $H(0, \cdot)$ lifted to $\tilde{\gamma}$, then $H$ lifts uniquely to $\tilde{H} : [0,1]^2 \to \tilde{X}$.

**Consequence:** If $\gamma_0 \simeq \gamma_1$ (path homotopic), then $\tilde{\gamma}_0(1) = \tilde{\gamma}_1(1)$. The endpoint of the lift depends only on the homotopy class.

### 4.3 General Lifting Criterion

> **Thm 3.5 (Lifting Criterion).** Let $p : (\tilde{X}, \tilde{x}_0) \to (X, x_0)$ be a covering with $\tilde{X}$ connected. A continuous $f : (Y, y_0) \to (X, x_0)$ lifts to $\tilde{f} : (Y, y_0) \to (\tilde{X}, \tilde{x}_0)$ iff $f_*(\pi_1(Y, y_0)) \subseteq p_*(\pi_1(\tilde{X}, \tilde{x}_0))$.

---

## 5. Computing $\pi_1(S^1) \cong \mathbb{Z}$ via Covering Spaces

Using the covering $p : \mathbb{R} \to S^1$:

1. **Lift loops:** A loop $\gamma$ at $1 \in S^1$ lifts to a path $\tilde{\gamma}$ in $\mathbb{R}$ starting at $0$.
2. **Endpoint:** $\tilde{\gamma}(1) \in p^{-1}(1) = \mathbb{Z}$. The integer $n = \tilde{\gamma}(1)$ is the **winding number**.
3. **Well-defined on $\pi_1$:** Homotopic loops lift to paths with the same endpoint (homotopy lifting).
4. **Homomorphism:** $[\gamma] \mapsto n$ is a group homomorphism $\pi_1(S^1) \to \mathbb{Z}$.
5. **Isomorphism:** Surjective (wind $n$ times) and injective (if $n = 0$, lift is a loop in $\mathbb{R}$, which is contractible). ∎

---

## 6. Deck Transformations

> **Def 3.6.** A **deck transformation** (or covering transformation) is a homeomorphism $\phi : \tilde{X} \to \tilde{X}$ with $p \circ \phi = p$.

The deck transformations form a group $\mathrm{Aut}(\tilde{X}/X)$ (or $\mathrm{Deck}(p)$).

| Covering | Deck group |
|----------|-----------|
| $\mathbb{R} \to S^1$ | $\mathbb{Z}$ (translations $t \mapsto t + n$) |
| $S^n \to \mathbb{R}P^n$ | $\mathbb{Z}/2\mathbb{Z}$ (antipodal map) |
| $z \mapsto z^n : S^1 \to S^1$ | $\mathbb{Z}/n\mathbb{Z}$ (rotations) |

> **Thm 3.7.** If $\tilde{X}$ is the **universal cover** (simply connected covering), then $\mathrm{Deck}(p) \cong \pi_1(X)$.

---

## 7. The Galois Correspondence for Covering Spaces

> **Thm 3.8 (Classification of Covering Spaces).** Let $X$ be connected, locally path-connected, and semi-locally simply connected. There is a bijection:
>
> $$\left\{ \begin{array}{c} \text{Connected coverings } \tilde{X} \to X \\ \text{(up to isomorphism)} \end{array} \right\} \longleftrightarrow \left\{ \begin{array}{c} \text{Subgroups of } \pi_1(X) \\ \text{(up to conjugacy)} \end{array} \right\}$$
>
> given by $p : \tilde{X} \to X \;\longmapsto\; p_*(\pi_1(\tilde{X})) \leq \pi_1(X)$.

| Subgroup $H$ | Covering | Degree |
|-------------|---------|:------:|
| $\{0\}$ | Universal cover | $|\pi_1|$ |
| $\pi_1(X)$ | $X$ itself (trivial cover) | 1 |
| $n\mathbb{Z} \leq \mathbb{Z}$ | $S^1 \xrightarrow{z^n} S^1$ | $n$ |

```
Lattice of subgroups ↔ Lattice of coverings

  {0} ──→ Universal cover (ℝ → S¹)
   |
  nℤ ──→ n-fold cover (S¹ →^{zⁿ} S¹)
   |
   ℤ  ──→ Trivial cover (S¹ → S¹)
```

---

## 8. Universal Cover

> **Def 3.9.** The **universal cover** $\tilde{X}$ is the unique (up to isomorphism) simply connected covering space of $X$.

**Existence:** Requires $X$ to be connected, locally path-connected, and semi-locally simply connected.

**Construction:** Points of $\tilde{X}$ are homotopy classes of paths from $x_0$:
$$\tilde{X} = \{[\gamma] : \gamma \text{ is a path from } x_0\}$$
with $p([\gamma]) = \gamma(1)$.

---

## 9. Summary Table

| Concept | Details |
|---------|---------|
| Covering map | Local homeomorphism; evenly covered neighbourhoods |
| Fibre | $p^{-1}(x)$; all same cardinality if $X$ connected |
| Path lifting | Unique lift given starting point |
| Homotopy lifting | Uniqueness implies $\pi_1$ computation |
| Deck group | $\mathrm{Aut}(\tilde{X}/X)$; $\cong \pi_1(X)$ for universal cover |
| Classification | Coverings ↔ subgroups of $\pi_1$ (Galois correspondence) |
| Universal cover | Simply connected; unique |

---

## 10. Quick Revision Questions

1. Define covering map. Give three examples with their degrees.

2. State the path-lifting and homotopy-lifting theorems.

3. Use the covering $\mathbb{R} \to S^1$ to prove $\pi_1(S^1) \cong \mathbb{Z}$.

4. Define deck transformations. Compute the deck group of $S^n \to \mathbb{R}P^n$.

5. State the classification theorem for covering spaces (Galois correspondence).

6. What conditions on $X$ guarantee existence of a universal cover? Describe the construction.

---

[← Previous: The Fundamental Group](02-fundamental-group.md) | [Back to README](../README.md)
