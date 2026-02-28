# Chapter 2.4 — First Isomorphism Theorem

[← Previous: Kernel and Image](03-kernel-and-image.md) | [Back to README](../README.md) | [Next: Second & Third Isomorphism Theorems →](05-second-third-isomorphism-theorems.md)

---

## Overview

The **First Isomorphism Theorem** is the most important structural result in group theory. It says that the image of a homomorphism is isomorphic to the quotient of the domain by the kernel. It unifies homomorphisms, normal subgroups, and quotient groups into one powerful framework.

---

## 1. Statement

**Theorem 2.11 (First Isomorphism Theorem).** Let $\phi: G \to H$ be a group homomorphism. Then:

$$G / \ker \phi \;\cong\; \text{im}\,\phi$$

More precisely, the map $\bar{\phi}: G/\ker\phi \to \text{im}\,\phi$ defined by $\bar{\phi}(g \cdot \ker\phi) = \phi(g)$ is a well-defined isomorphism.

```
  The First Isomorphism Theorem:
  
           φ
    G ──────────────→ H
    │                 ↑
    │ π               │ inclusion
    │                 │
    ▼      φ̄         │
  G/ker φ ─────≅────→ im φ  ≤  H
  
  The diagram says:
  φ = inclusion ∘ φ̄ ∘ π
  
  "Every homomorphism factors through the quotient by its kernel"
```

---

## 2. Proof

**Well-defined:** If $g_1 \ker\phi = g_2 \ker\phi$, then $g_1^{-1}g_2 \in \ker\phi$, so $\phi(g_1^{-1}g_2) = e$, giving $\phi(g_1) = \phi(g_2)$.

**Homomorphism:** $\bar{\phi}(g_1\ker\phi \cdot g_2\ker\phi) = \bar{\phi}(g_1g_2 \ker\phi) = \phi(g_1g_2) = \phi(g_1)\phi(g_2) = \bar{\phi}(g_1\ker\phi)\bar{\phi}(g_2\ker\phi)$.

**Injective:** $\bar{\phi}(g\ker\phi) = e_H \implies \phi(g) = e_H \implies g \in \ker\phi \implies g\ker\phi = \ker\phi$ (identity of $G/\ker\phi$).

**Surjective:** By definition of $\text{im}\,\phi$, every $\phi(g) \in \text{im}\,\phi$ is the image of $g\ker\phi$. $\square$

---

## 3. Visual Intuition

```
  G is partitioned into cosets of K = ker φ:
  
  ┌───────┬───────┬───────┬───────┬───────┐
  │  K    │ g₁K   │ g₂K   │ g₃K   │ g₄K   │    ← G
  │  ↓    │  ↓    │  ↓    │  ↓    │  ↓    │
  │ eH    │ φ(g₁) │ φ(g₂) │ φ(g₃) │ φ(g₄) │    ← im φ
  └───────┴───────┴───────┴───────┴───────┘
  
  Each coset maps to exactly one element.
  The coset multiplication in G/K corresponds 
  exactly to multiplication in im φ.
  
  G/K  ≅  im φ
```

---

## 4. Applications

### Application 1: $\mathbb{Z}/n\mathbb{Z} \cong \mathbb{Z}_n$

$\phi: \mathbb{Z} \to \mathbb{Z}_n$, $\phi(k) = k \bmod n$.
- $\ker\phi = n\mathbb{Z}$
- $\text{im}\,\phi = \mathbb{Z}_n$
- **FIT:** $\mathbb{Z}/n\mathbb{Z} \cong \mathbb{Z}_n$ ✓

### Application 2: $GL_n(\mathbb{R})/SL_n(\mathbb{R}) \cong \mathbb{R}^\times$

$\phi = \det: GL_n(\mathbb{R}) \to \mathbb{R}^\times$
- $\ker\phi = SL_n(\mathbb{R})$
- $\text{im}\,\phi = \mathbb{R}^\times$ (surjective)
- **FIT:** $GL_n(\mathbb{R})/SL_n(\mathbb{R}) \cong \mathbb{R}^\times$

### Application 3: $S_n/A_n \cong \mathbb{Z}_2$

$\phi = \text{sgn}: S_n \to \{+1, -1\} \cong \mathbb{Z}_2$
- $\ker\phi = A_n$
- $\text{im}\,\phi = \{+1, -1\}$ (surjective)
- **FIT:** $S_n/A_n \cong \mathbb{Z}_2$

### Application 4: Cyclic Group Quotients

$\phi: \mathbb{Z}_{12} \to \mathbb{Z}_{12}$, $\phi(x) = 4x$
- $\ker\phi = \{0, 3, 6, 9\} \cong \mathbb{Z}_4$? No, $\cong \langle 3 \rangle$
- Wait: $\ker\phi = \{x : 4x \equiv 0 \pmod{12}\} = \{0, 3, 6, 9\}$
- $\text{im}\,\phi = \{0, 4, 8\} \cong \mathbb{Z}_3$
- **FIT:** $\mathbb{Z}_{12}/\langle 3 \rangle \cong \mathbb{Z}_3$

```
  Summary of FIT applications:
  
  Homomorphism         │ Kernel    │ Image    │ Quotient ≅ Image
  ─────────────────────┼───────────┼──────────┼──────────────────
  Z → Zₙ (mod n)      │ nZ        │ Zₙ       │ Z/nZ ≅ Zₙ
  det: GLₙ → R×       │ SLₙ      │ R×       │ GLₙ/SLₙ ≅ R×
  sgn: Sₙ → {±1}     │ Aₙ        │ Z₂       │ Sₙ/Aₙ ≅ Z₂
  exp: R → R⁺         │ {0}       │ R⁺       │ R/{0} ≅ R⁺ (trivial)
  Z₁₂ →[×4] Z₁₂      │ ⟨3⟩       │ ⟨4⟩      │ Z₁₂/⟨3⟩ ≅ Z₃
```

---

## 5. The Universal Property

The FIT gives a **universal property**: any homomorphism from $G$ whose kernel contains $N$ factors uniquely through $G/N$.

**Theorem 2.12 (Universal Property of Quotients).** Let $N \trianglelefteq G$ and $\phi: G \to H$ a homomorphism with $N \subseteq \ker\phi$. Then there exists a unique homomorphism $\bar{\phi}: G/N \to H$ with $\bar{\phi} \circ \pi = \phi$.

```
           φ
    G ──────────→ H
    │             ↑
    │ π           │ ∃! φ̄
    │             │
    ▼             │
   G/N ───────────┘
   
  If N ⊆ ker φ, then φ factors through G/N.
  
  Moreover:
  • ker φ̄ = ker φ / N
  • φ̄ injective ⟺ N = ker φ
  • φ̄ surjective ⟺ φ surjective
```

---

## 6. Correspondence Theorem

**Theorem 2.13 (Fourth Isomorphism Theorem / Lattice Isomorphism).** Let $N \trianglelefteq G$. There is a bijection:

$$\{\text{subgroups of } G \text{ containing } N\} \;\longleftrightarrow\; \{\text{subgroups of } G/N\}$$

given by $H \mapsto H/N$ (and $\bar{H} \mapsto \pi^{-1}(\bar{H})$).

This bijection preserves:
- Containment: $H_1 \leq H_2 \iff H_1/N \leq H_2/N$
- Normality: $H \trianglelefteq G \iff H/N \trianglelefteq G/N$
- Index: $[G:H] = [G/N : H/N]$

```
  Correspondence Theorem — Lattice Isomorphism
  
  Subgroups of G containing N    ↔    Subgroups of G/N
  
         G                              G/N
        / \                            / \
       /   \                          /   \
      H₁    H₂                    H₁/N   H₂/N
       \   /                        \   /
        \ /                          \ /
         N                           {N}
         
  The lattice structure is preserved!
```

---

## 7. Step-by-Step: Using the First Isomorphism Theorem

```
  HOW TO USE THE FIT:
  
  Step 1: Identify or construct a homomorphism φ: G → H
  
  Step 2: Compute ker φ (must be normal in G)
  
  Step 3: Compute im φ (a subgroup of H)
  
  Step 4: Conclude G/ker φ ≅ im φ
  
  ═══════════════════════════════════════════════
  
  GOING THE OTHER WAY (proving G/N ≅ H):
  
  Step 1: Construct φ: G → H  (a surjective homomorphism)
  
  Step 2: Show ker φ = N
  
  Step 3: FIT gives G/N ≅ H
```

### Worked Example

**Claim:** $\mathbb{Z} \times \mathbb{Z} / \langle(1,1)\rangle \cong \mathbb{Z}$

*Proof.* Define $\phi: \mathbb{Z} \times \mathbb{Z} \to \mathbb{Z}$ by $\phi(a,b) = a - b$.

- **Homomorphism:** $\phi((a_1,b_1)+(a_2,b_2)) = \phi(a_1+a_2, b_1+b_2) = (a_1+a_2)-(b_1+b_2) = (a_1-b_1)+(a_2-b_2) = \phi(a_1,b_1)+\phi(a_2,b_2)$ ✓
- **Surjective:** $\phi(n, 0) = n$ for any $n \in \mathbb{Z}$ ✓
- **Kernel:** $\phi(a,b) = 0 \iff a = b \iff (a,b) = k(1,1)$ for some $k$, so $\ker\phi = \langle(1,1)\rangle$ ✓
- **FIT:** $\mathbb{Z}^2/\langle(1,1)\rangle \cong \mathbb{Z}$. $\square$

---

## 8. Real-World Applications

| Application | FIT Instance |
|-------------|-------------|
| Modular arithmetic | $\mathbb{Z}/n\mathbb{Z} \cong \mathbb{Z}_n$ |
| Error-correcting codes | Syndrome decoding uses $\mathbb{Z}_2^n / C \cong \mathbb{Z}_2^k$ |
| Quotient spaces (topology) | Analogous construction |
| Signal processing | $\mathbb{R}/\mathbb{Z} \cong$ unit circle (circular signals) |

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| First Isomorphism Theorem | $G/\ker\phi \cong \text{im}\,\phi$ |
| The induced map | $\bar{\phi}(g\ker\phi) = \phi(g)$ is well-defined |
| Universal property | $N \subseteq \ker\phi \implies \phi$ factors through $G/N$ |
| Correspondence theorem | Subgroups of $G/N \leftrightarrow$ subgroups of $G$ containing $N$ |
| $\mathbb{Z}/n\mathbb{Z}$ | $\cong \mathbb{Z}_n$ |
| $GL_n/SL_n$ | $\cong \mathbb{R}^\times$ |
| $S_n/A_n$ | $\cong \mathbb{Z}_2$ |

---

## Quick Revision Questions

1. **State** the First Isomorphism Theorem and draw the commutative diagram.

2. **Use the FIT** to prove $\mathbb{Z}/5\mathbb{Z} \cong \mathbb{Z}_5$.

3. **Apply the FIT** to $\det: GL_2(\mathbb{R}) \to \mathbb{R}^\times$ to identify $GL_2(\mathbb{R})/SL_2(\mathbb{R})$.

4. **Find** a surjective homomorphism $\phi: \mathbb{Z}_{12} \to \mathbb{Z}_4$ and verify the FIT.

5. **Prove** that $\mathbb{R}/\mathbb{Z}$ is isomorphic to the circle group $S^1 = \{z \in \mathbb{C} : |z| = 1\}$.

6. **State** the Correspondence Theorem and use it to list all subgroups of $\mathbb{Z}_{12}/\langle 4 \rangle$.

---

[← Previous: Kernel and Image](03-kernel-and-image.md) | [Back to README](../README.md) | [Next: Second & Third Isomorphism Theorems →](05-second-third-isomorphism-theorems.md)
