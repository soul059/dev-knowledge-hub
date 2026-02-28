# Chapter 2.3 — Kernel and Image

[← Previous: Isomorphisms](02-isomorphisms.md) | [Back to README](../README.md) | [Next: First Isomorphism Theorem →](04-first-isomorphism-theorem.md)

---

## Overview

The **kernel** and **image** of a homomorphism are its most important structural features. The kernel measures "how far from injective" the map is, while the image measures "how far from surjective". Together they completely determine the homomorphism's behaviour.

---

## 1. Definitions

**Definition.** Let $\phi: G \to H$ be a group homomorphism.

The **kernel** of $\phi$:
$$\ker \phi = \{g \in G : \phi(g) = e_H\} = \phi^{-1}(\{e_H\})$$

The **image** of $\phi$:
$$\text{im}\,\phi = \{\phi(g) : g \in G\} = \phi(G)$$

```
         G                           H
  ┌──────────────┐            ┌──────────────┐
  │              │     φ      │              │
  │  ┌────────┐  │ ────────→  │  ┌────────┐  │
  │  │ker(φ)  │  │            │  │ im(φ)  │  │
  │  │ maps   │  │            │  │        │  │
  │  │ to eH  │  │            │  │ = φ(G) │  │
  │  └────────┘  │            │  └────────┘  │
  │              │            │              │
  └──────────────┘            └──────────────┘
  
  ker(φ) ◁ G       (always normal in G)
  im(φ)  ≤ H       (always a subgroup of H)
```

---

## 2. Kernel is a Normal Subgroup

**Theorem 2.6.** For any homomorphism $\phi: G \to H$:
1. $\ker \phi \leq G$ (it's a subgroup)
2. $\ker \phi \trianglelefteq G$ (it's a **normal** subgroup)

*Proof of normality.* Let $k \in \ker \phi$ and $g \in G$. Then:
$$\phi(gkg^{-1}) = \phi(g)\phi(k)\phi(g^{-1}) = \phi(g) \cdot e_H \cdot \phi(g)^{-1} = e_H$$

So $gkg^{-1} \in \ker \phi$. Thus $g(\ker \phi)g^{-1} \subseteq \ker \phi$ for all $g \in G$. $\square$

### The Deep Significance

```
  ┌─────────────────────────────────────────────────┐
  │                                                  │
  │   ker(φ) ◁ G   is the FUNDAMENTAL reason        │
  │   why normal subgroups matter!                   │
  │                                                  │
  │   Fact: N ◁ G  ⟺  N = ker(φ) for some          │
  │         homomorphism φ from G                    │
  │                                                  │
  └─────────────────────────────────────────────────┘
```

---

## 3. Image is a Subgroup

**Theorem 2.7.** $\text{im}\,\phi \leq H$.

*Proof.* $\phi(e_G) = e_H \in \text{im}\,\phi$, so $\text{im}\,\phi \neq \emptyset$. If $\phi(a), \phi(b) \in \text{im}\,\phi$, then $\phi(a)\phi(b)^{-1} = \phi(a)\phi(b^{-1}) = \phi(ab^{-1}) \in \text{im}\,\phi$. $\square$

---

## 4. Kernel and Injectivity

**Theorem 2.8.** $\phi$ is injective $\iff$ $\ker \phi = \{e_G\}$.

*Proof.* ($\Rightarrow$) If $\phi$ is injective and $\phi(g) = e_H = \phi(e_G)$, then $g = e_G$.

($\Leftarrow$) If $\phi(a) = \phi(b)$, then $\phi(ab^{-1}) = \phi(a)\phi(b)^{-1} = e_H$, so $ab^{-1} \in \ker \phi = \{e_G\}$, giving $a = b$. $\square$

```
  Relationship between ker(φ) and injectivity:
  
  ker(φ) = {e}     ⟺    φ injective (monomorphism)
  
  ker(φ) = G       ⟺    φ is the trivial homomorphism
  
  {e} ⊂ ker(φ) ⊂ G  ⟹   φ is neither trivial nor injective
```

---

## 5. The Fibre Decomposition

**Definition.** For $h \in \text{im}\,\phi$, the **fibre** (or preimage) over $h$ is:

$$\phi^{-1}(h) = \{g \in G : \phi(g) = h\}$$

**Theorem 2.9.** Each fibre is a **coset** of $\ker \phi$:

$$\phi^{-1}(h) = g \cdot \ker \phi \quad \text{where } \phi(g) = h$$

All fibres have the same cardinality: $|\phi^{-1}(h)| = |\ker \phi|$.

```
  Fibre decomposition of G by φ: G → H
  
  G is partitioned into cosets of ker(φ):
  
  ┌─────────┬─────────┬─────────┬─────────┐
  │  ker φ  │ g₁·kerφ │ g₂·kerφ │ g₃·kerφ │  ...
  │  ↓ eH   │  ↓ h₁   │  ↓ h₂   │  ↓ h₃   │
  └─────────┴─────────┴─────────┴─────────┘
       │          │          │          │
       ▼          ▼          ▼          ▼
  ┌────────────────────────────────────────┐
  │    eH        h₁        h₂        h₃   │  ⊂ H
  └────────────────────────────────────────┘
  
  Each column has |ker φ| elements.
  Number of columns = |im φ| = [G : ker φ]
  
  |G| = |ker φ| · |im φ|
```

---

## 6. The Rank-Nullity Analogue

Just as in linear algebra ($\dim V = \dim \ker T + \dim \text{im}\,T$), we have:

$$|G| = |\ker \phi| \cdot |\text{im}\,\phi|$$

or equivalently:

$$[G : \ker \phi] = |\text{im}\,\phi|$$

This is a group-theoretic "rank-nullity theorem".

| Linear Algebra | Group Theory |
|---------------|-------------|
| $\ker T$ (null space) | $\ker \phi$ |
| $\text{im}\,T$ (range) | $\text{im}\,\phi$ |
| $\dim V = \dim \ker T + \dim \text{im}\,T$ | $\|G\| = \|\ker \phi\| \cdot \|\text{im}\,\phi\|$ |
| $T$ injective $\iff \ker T = \{0\}$ | $\phi$ injective $\iff \ker \phi = \{e\}$ |

---

## 7. Computing Kernel and Image — Examples

### Example 1: $\phi: \mathbb{Z} \to \mathbb{Z}_n$, $\phi(k) = k \bmod n$

- $\ker \phi = n\mathbb{Z} = \{\ldots, -2n, -n, 0, n, 2n, \ldots\}$
- $\text{im}\,\phi = \mathbb{Z}_n$ (surjective)
- $|\mathbb{Z}/n\mathbb{Z}| = n = |\text{im}\,\phi|$ ✓

### Example 2: $\det: GL_n(\mathbb{R}) \to \mathbb{R}^\times$

- $\ker(\det) = SL_n(\mathbb{R}) = \{A : \det A = 1\}$
- $\text{im}(\det) = \mathbb{R}^\times$ (surjective — every nonzero det is achievable)

### Example 3: $\text{sgn}: S_n \to \{+1, -1\}$

- $\ker(\text{sgn}) = A_n$ (alternating group)
- $\text{im}(\text{sgn}) = \{+1, -1\}$ (surjective for $n \geq 2$)
- $|A_n| = |S_n|/|\text{im}| = n!/2$

### Example 4: $\phi: \mathbb{Z}_{12} \to \mathbb{Z}_{12}$, $\phi(x) = 3x$

- $\ker \phi = \{x : 3x \equiv 0 \pmod{12}\} = \{0, 4, 8\}$
- $\text{im}\,\phi = \{0, 3, 6, 9\}$
- $|\ker \phi| \cdot |\text{im}\,\phi| = 3 \times 4 = 12 = |\mathbb{Z}_{12}|$ ✓

### Summary of Examples

```
  Homomorphism          │ Kernel              │ Image
  ──────────────────────┼─────────────────────┼──────────────
  Z → Zₙ (mod n)       │ nZ                  │ Zₙ
  det: GLₙ → R×        │ SLₙ(R)             │ R×
  sgn: Sₙ → {±1}       │ Aₙ                  │ {±1}
  φ(x) = 3x on Z₁₂    │ {0, 4, 8}           │ {0, 3, 6, 9}
  exp: (R,+) → (R⁺,·)  │ {0}                 │ R⁺
```

---

## 8. Normal Subgroups Are Exactly Kernels

**Theorem 2.10.** A subgroup $N \leq G$ is normal if and only if $N = \ker \phi$ for some homomorphism $\phi$ from $G$.

*Proof.*
- ($\Leftarrow$) Already shown: kernels are normal.
- ($\Rightarrow$) If $N \trianglelefteq G$, define $\pi: G \to G/N$ by $\pi(g) = gN$ (the natural projection). Then $\ker \pi = N$. $\square$

```
  Equivalence:
  
  N ◁ G   ⟺   N = ker(π)  where  π: G → G/N
  
  This connects:
  • Normal subgroups (internal structure)
  • Kernels of homomorphisms (maps between groups)
  • Quotient groups (new group construction)
  
  THREE perspectives on the SAME concept!
```

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| $\ker \phi$ | $= \{g \in G : \phi(g) = e_H\}$ |
| $\text{im}\,\phi$ | $= \{\phi(g) : g \in G\}$ |
| $\ker \phi$ | is always normal in $G$ |
| $\text{im}\,\phi$ | is always a subgroup of $H$ |
| $\phi$ injective | $\iff \ker \phi = \{e\}$ |
| Fibres | $= $ cosets of $\ker \phi$ |
| Counting | $\|G\| = \|\ker \phi\| \cdot \|\text{im}\,\phi\|$ |
| Normal = Kernel | $N \trianglelefteq G \iff N = \ker\phi$ for some $\phi$ |

---

## Quick Revision Questions

1. **Find** the kernel and image of $\phi: \mathbb{Z} \to \mathbb{Z}$, $\phi(n) = 3n$.

2. **Prove** that $\ker \phi \trianglelefteq G$ for any homomorphism $\phi$.

3. **Show** that $\phi: GL_2(\mathbb{R}) \to \mathbb{R}^\times$ via $\det$ has kernel $SL_2(\mathbb{R})$.

4. **If** $\phi: \mathbb{Z}_{20} \to \mathbb{Z}_{20}$ is defined by $\phi(x) = 4x$, find $\ker \phi$ and $\text{im}\,\phi$.

5. **Prove** that a homomorphism from a simple group is either trivial or injective.

6. **Verify** the counting formula $|G| = |\ker \phi| \cdot |\text{im}\,\phi|$ for $\text{sgn}: S_4 \to \{+1, -1\}$.

---

[← Previous: Isomorphisms](02-isomorphisms.md) | [Back to README](../README.md) | [Next: First Isomorphism Theorem →](04-first-isomorphism-theorem.md)
