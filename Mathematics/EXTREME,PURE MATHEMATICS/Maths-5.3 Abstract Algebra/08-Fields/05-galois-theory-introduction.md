# Chapter 8.5 — Introduction to Galois Theory

[← Previous: Splitting Fields](04-splitting-fields.md) | [Back to README](../README.md)

---

## Overview

**Galois theory** is the crown jewel of abstract algebra, connecting field extensions to group theory via the **Galois correspondence**. The Fundamental Theorem of Galois Theory establishes a bijection between intermediate fields and subgroups of the Galois group, translating field-theoretic problems into group-theoretic ones. This chapter introduces the basic framework and states the main theorem.

---

## 1. Galois Extensions

**Definition.** A finite extension $E/F$ is a **Galois extension** if it is both **normal** and **separable**.

Equivalently: $E$ is the splitting field of a separable polynomial over $F$.

In **characteristic 0** (e.g., over $\mathbb{Q}$): Galois $\iff$ normal $\iff$ splitting field of some polynomial.

---

## 2. The Galois Group

**Definition.** The **Galois group** of a Galois extension $E/F$ is:

$$\text{Gal}(E/F) = \{\sigma: E \to E : \sigma \text{ is a field automorphism with } \sigma|_F = \text{id}\}$$

These are the automorphisms of $E$ that fix every element of $F$.

**Theorem 8.19.** If $E/F$ is Galois, then $|\text{Gal}(E/F)| = [E:F]$.

---

## 3. Examples of Galois Groups

### 3.1 $\mathbb{Q}(\sqrt{2})/\mathbb{Q}$

Galois (splitting field of $x^2 - 2$). Automorphisms fixing $\mathbb{Q}$:
- $\text{id}$: $\sqrt{2} \mapsto \sqrt{2}$
- $\sigma$: $\sqrt{2} \mapsto -\sqrt{2}$

$\text{Gal}(\mathbb{Q}(\sqrt{2})/\mathbb{Q}) \cong \mathbb{Z}/2\mathbb{Z}$.

### 3.2 $\mathbb{Q}(\sqrt{2}, \sqrt{3})/\mathbb{Q}$

Galois (splitting field of $(x^2-2)(x^2-3)$). Four automorphisms:

| $\sigma$ | $\sqrt{2} \mapsto$ | $\sqrt{3} \mapsto$ |
|----------|-------------------|-------------------|
| $e$ | $\sqrt{2}$ | $\sqrt{3}$ |
| $\sigma_1$ | $-\sqrt{2}$ | $\sqrt{3}$ |
| $\sigma_2$ | $\sqrt{2}$ | $-\sqrt{3}$ |
| $\sigma_3$ | $-\sqrt{2}$ | $-\sqrt{3}$ |

$\text{Gal}(\mathbb{Q}(\sqrt{2},\sqrt{3})/\mathbb{Q}) \cong \mathbb{Z}/2\mathbb{Z} \times \mathbb{Z}/2\mathbb{Z} \cong V_4$ (Klein four-group).

### 3.3 $\mathbb{Q}(\sqrt[3]{2}, \omega)/\mathbb{Q}$

Splitting field of $x^3 - 2$. Degree 6.

Roots: $\alpha = \sqrt[3]{2}$, $\omega\alpha$, $\omega^2\alpha$. An automorphism is determined by where it sends $\alpha$ and $\omega$.

$\text{Gal} \cong S_3$ (symmetric group on 3 letters, permuting the 3 roots).

```
  Gal(ℚ(∛2,ω)/ℚ) ≅ S₃:
  
  σ: α ↦ ωα, ω ↦ ω        (order 3, rotation)
  τ: α ↦ α,  ω ↦ ω²       (order 2, complex conjugation)
  
  |S₃| = 6 = [ℚ(∛2,ω) : ℚ]  ✓
  
  Elements: {e, σ, σ², τ, στ, σ²τ}
```

### 3.4 $\mathbb{F}_{p^n}/\mathbb{F}_p$

$\text{Gal}(\mathbb{F}_{p^n}/\mathbb{F}_p) = \langle \phi \rangle \cong \mathbb{Z}/n\mathbb{Z}$

where $\phi: x \mapsto x^p$ is the Frobenius automorphism.

---

## 4. The Fundamental Theorem of Galois Theory

**Theorem 8.20 (Fundamental Theorem).** Let $E/F$ be a finite Galois extension with Galois group $G = \text{Gal}(E/F)$. There is an inclusion-reversing bijection:

$$\{\text{intermediate fields } F \subseteq K \subseteq E\} \longleftrightarrow \{\text{subgroups } H \leq G\}$$

given by:

$$K \mapsto \text{Gal}(E/K) = \{\sigma \in G : \sigma|_K = \text{id}\}$$

$$H \mapsto E^H = \{\alpha \in E : \sigma(\alpha) = \alpha \; \forall \sigma \in H\}$$

Moreover:
1. $[E:K] = |H|$ and $[K:F] = [G:H]$ (index).
2. $K/F$ is normal $\iff$ $H \trianglelefteq G$ (normal subgroup).
3. If $H \trianglelefteq G$, then $\text{Gal}(K/F) \cong G/H$.

```
  The Galois Correspondence (inclusion-reversing):
  
  Fields:                    Groups:
  
  E ◄───────────────────────▸ {e}
  │                            │
  K ◄───────────────────────▸ H = Gal(E/K)
  │                            │
  F ◄───────────────────────▸ G = Gal(E/F)
  
  Larger field ↔ Smaller group  (order-reversing!)
  [E:K] = |H|,  [K:F] = [G:H]
  K/F normal ⟺ H ◁ G
```

---

## 5. Worked Example: $\mathbb{Q}(\sqrt{2}, \sqrt{3})/\mathbb{Q}$

$G = \{e, \sigma_1, \sigma_2, \sigma_3\} \cong V_4$.

Subgroups of $V_4$: $\{e\}$, $\langle\sigma_1\rangle$, $\langle\sigma_2\rangle$, $\langle\sigma_3\rangle$, $V_4$.

| Subgroup $H$ | $|H|$ | Fixed field $E^H$ | $[E^H:F]$ |
|--------------|--------|-------------------|-----------|
| $\{e\}$ | 1 | $\mathbb{Q}(\sqrt{2},\sqrt{3})$ | 4 |
| $\langle\sigma_1\rangle$ | 2 | $\mathbb{Q}(\sqrt{3})$ | 2 |
| $\langle\sigma_2\rangle$ | 2 | $\mathbb{Q}(\sqrt{2})$ | 2 |
| $\langle\sigma_3\rangle$ | 2 | $\mathbb{Q}(\sqrt{6})$ | 2 |
| $V_4$ | 4 | $\mathbb{Q}$ | 1 |

```
  Fields:                    Subgroups:
  
  ℚ(√2,√3)                  {e}
   /  |  \                  /  |  \
ℚ(√2) ℚ(√3) ℚ(√6)    ⟨σ₂⟩ ⟨σ₁⟩ ⟨σ₃⟩
   \  |  /                  \  |  /
     ℚ                        V₄
  
  All subgroups are normal (V₄ is abelian), 
  so all intermediate extensions are Galois over ℚ.
```

---

## 6. Worked Example: $x^3 - 2$

$E = \mathbb{Q}(\sqrt[3]{2}, \omega)$, $G = \text{Gal}(E/\mathbb{Q}) \cong S_3$.

Subgroups of $S_3$:

| Subgroup $H$ | $|H|$ | Index | Fixed field | Normal? | $K/\mathbb{Q}$ Galois? |
|--------------|--------|-------|-------------|---------|----------------------|
| $\{e\}$ | 1 | 6 | $E$ | ✓ | ✓ |
| $\langle\tau\rangle \cong \mathbb{Z}_2$ | 2 | 3 | $\mathbb{Q}(\sqrt[3]{2})$ | ✗ | ✗ |
| $\langle\sigma\tau\rangle \cong \mathbb{Z}_2$ | 2 | 3 | $\mathbb{Q}(\omega\sqrt[3]{2})$ | ✗ | ✗ |
| $\langle\sigma^2\tau\rangle \cong \mathbb{Z}_2$ | 2 | 3 | $\mathbb{Q}(\omega^2\sqrt[3]{2})$ | ✗ | ✗ |
| $\langle\sigma\rangle \cong \mathbb{Z}_3$ | 3 | 2 | $\mathbb{Q}(\omega)$ | ✓ | ✓ |
| $S_3$ | 6 | 1 | $\mathbb{Q}$ | ✓ | ✓ |

```
  Fields:                    Subgroups of S₃:
  
     ℚ(∛2, ω)                    {e}
     / | | \                    / | | \
  ℚ(∛2) ℚ(ω∛2) ℚ(ω²∛2) ℚ(ω)   ⟨τ⟩ ⟨στ⟩ ⟨σ²τ⟩ ⟨σ⟩
      \   |   /    /              \   |   /    /
           ℚ                          S₃
  
  Only ℚ(ω)/ℚ is Galois among the intermediate extensions,
  corresponding to the unique normal subgroup ⟨σ⟩ = A₃ ◁ S₃.
```

---

## 7. Applications of Galois Theory

### 7.1 Impossibility of Classical Constructions

**Theorem 8.21.** A length is constructible by straightedge and compass $\iff$ it lies in a tower $\mathbb{Q} = F_0 \subset F_1 \subset \cdots \subset F_n$ with each $[F_{i+1}:F_i] = 2$.

**Corollaries:**
- **Doubling the cube** is impossible: $\sqrt[3]{2}$ has degree 3, not a power of 2.
- **Trisecting a general angle** is impossible: $\cos(20°)$ has minimal polynomial $8x^3 - 6x - 1$, degree 3.
- **Squaring the circle** is impossible: $\sqrt{\pi}$ is transcendental.

### 7.2 Solvability by Radicals

**Definition.** A polynomial $f \in F[x]$ is **solvable by radicals** if its roots can be expressed using $+, -, \times, \div$ and $\sqrt[n]{}$ (for various $n$).

**Theorem 8.22 (Galois).** $f \in F[x]$ is solvable by radicals $\iff$ $\text{Gal}(E/F)$ is a **solvable group** (has a composition series with abelian factors).

**Theorem 8.23 (Abel–Ruffini).** The general polynomial of degree $\geq 5$ is **not** solvable by radicals.

*Proof.* The Galois group of the "generic" degree-$n$ polynomial is $S_n$. For $n \geq 5$, $S_n$ is not solvable (since $A_n$ is simple for $n \geq 5$). $\blacksquare$

```
  Solvability by radicals:
  
  Degree 1: ✓  (trivially)
  Degree 2: ✓  quadratic formula     Gal ≤ S₂ ≅ ℤ₂ (solvable)
  Degree 3: ✓  Cardano's formula     Gal ≤ S₃ (solvable)
  Degree 4: ✓  Ferrari's formula     Gal ≤ S₄ (solvable)
  Degree 5: ✗  in general!           Gal = S₅ (NOT solvable)
  
  Specific degree-5 example: x⁵ - 4x + 2
  Gal ≅ S₅ (not solvable) ⟹ no radical formula for roots.
```

### 7.3 Fundamental Theorem of Algebra

Galois theory gives an elegant proof that $\mathbb{C}$ is algebraically closed, using only:
- $\mathbb{R}$ has no odd-degree extension (intermediate value theorem)
- Every element of $\mathbb{C}$ has a square root
- Sylow's theorem (to handle 2-power degrees)

---

## 8. Summary: The Big Picture

```
  THE GALOIS THEORY PIPELINE:
  
  Polynomial  ──▸  Splitting field  ──▸  Galois group  ──▸  Answer
  f(x) ∈ F[x]      E/F                  G = Gal(E/F)
  
  Questions about         translated to      Questions about
  roots & fields    ◄──────────────────    groups & subgroups
  
  ┌──────────────────────────────────────────────┐
  │  Field theory        ◄═══▸  Group theory     │
  │  Intermediate fields ◄═══▸  Subgroups        │
  │  Normal extensions   ◄═══▸  Normal subgroups │
  │  Degree [K:F]        ◄═══▸  Index [G:H]      │
  │  Solvable by radicals◄═══▸  Solvable group   │
  └──────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Definition / Result |
|---------|-------------------|
| Galois extension | Normal + separable |
| Galois group | $\text{Gal}(E/F) = \text{Aut}_F(E)$; $|\text{Gal}(E/F)| = [E:F]$ |
| FTGT | Bijection: intermediate fields $\leftrightarrow$ subgroups |
| Normal extension $\leftrightarrow$ normal subgroup | $K/F$ normal $\iff$ $\text{Gal}(E/K) \trianglelefteq G$ |
| Constructibility | Degree must be a power of 2 |
| Solvable by radicals | $\iff$ Galois group is solvable |
| Abel–Ruffini | General degree $\geq 5$ not solvable ($S_n$ not solvable for $n \geq 5$) |

---

## Quick Revision Questions

1. **State** the definition of a Galois extension and the Galois group.

2. **Compute** $\text{Gal}(\mathbb{Q}(\sqrt{5})/\mathbb{Q})$ and list all automorphisms.

3. **Apply** the Fundamental Theorem: list all subgroups of $V_4 = \text{Gal}(\mathbb{Q}(\sqrt{2},\sqrt{3})/\mathbb{Q})$ and the corresponding intermediate fields.

4. **Explain** why $\mathbb{Q}(\sqrt[3]{2})/\mathbb{Q}$ is not a Galois extension.

5. **State** the Abel–Ruffini theorem and its connection to the simplicity of $A_5$.

6. **Prove** that doubling the cube is impossible using degree arguments and the tower law.

---

[← Previous: Splitting Fields](04-splitting-fields.md) | [Back to README](../README.md)
