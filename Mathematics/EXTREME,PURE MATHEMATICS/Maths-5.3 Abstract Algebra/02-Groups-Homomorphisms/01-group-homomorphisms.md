# Chapter 2.1 — Group Homomorphisms

[← Previous: Generators](../01-Groups-Basics/05-generators.md) | [Back to README](../README.md) | [Next: Isomorphisms →](02-isomorphisms.md)

---

## Overview

Homomorphisms are the "structure-preserving maps" between groups. They allow us to compare groups, transfer information between them, and detect structural similarities. Understanding homomorphisms is arguably the most important skill in abstract algebra.

---

## 1. Definition

**Definition.** Let $(G, \ast)$ and $(H, \star)$ be groups. A function $\phi: G \to H$ is a **group homomorphism** if:

$$\phi(a \ast b) = \phi(a) \star \phi(b) \quad \text{for all } a, b \in G$$

```
  A homomorphism preserves the operation:
  
  In G:     a ∗ b = c
             │  │    │
          φ  │  │ φ  │ φ
             ▼  ▼    ▼
  In H:   φ(a) ⋆ φ(b) = φ(c)
  
  "Doing the operation then mapping" 
  gives the same result as 
  "mapping then doing the operation"
```

### Commutative Diagram

```
  G × G ───∗──→ G
    │              │
  φ×φ             φ
    │              │
    ▼              ▼
  H × H ───⋆──→ H

  This diagram COMMUTES: φ(a ∗ b) = φ(a) ⋆ φ(b)
```

---

## 2. Basic Properties

**Theorem 2.1.** Let $\phi: G \to H$ be a homomorphism. Then:

| Property | Statement |
|----------|-----------|
| (a) Identity maps to identity | $\phi(e_G) = e_H$ |
| (b) Inverses map to inverses | $\phi(a^{-1}) = \phi(a)^{-1}$ |
| (c) Powers are preserved | $\phi(a^n) = \phi(a)^n$ for all $n \in \mathbb{Z}$ |
| (d) Order divides | $\|\phi(a)\|$ divides $\|a\|$ |

*Proof of (a):* $\phi(e_G) = \phi(e_G \cdot e_G) = \phi(e_G) \cdot \phi(e_G)$. Cancel $\phi(e_G)$ to get $\phi(e_G) = e_H$. $\square$

*Proof of (b):* $\phi(a)\phi(a^{-1}) = \phi(aa^{-1}) = \phi(e_G) = e_H$, so $\phi(a^{-1}) = \phi(a)^{-1}$. $\square$

---

## 3. Important Examples

### 3.1 The Trivial Homomorphism

$\phi: G \to H$ defined by $\phi(g) = e_H$ for all $g$.

Always a homomorphism: $\phi(ab) = e_H = e_H \cdot e_H = \phi(a)\phi(b)$.

### 3.2 The Identity Homomorphism

$\text{id}_G: G \to G$ defined by $\text{id}_G(g) = g$. Trivially a homomorphism.

### 3.3 The Inclusion Map

If $H \leq G$, then $\iota: H \hookrightarrow G$ by $\iota(h) = h$ is a homomorphism.

### 3.4 The Determinant Map

$$\det: GL_n(\mathbb{R}) \to \mathbb{R}^\times$$

$\det(AB) = \det(A)\det(B)$ — this is precisely the homomorphism property!

```
  det: GL₂(R) ──→ R×
  
  ┌         ┐
  │  a   b  │  ──→  ad - bc
  │  c   d  │
  └         ┘
  
  ker(det) = SL₂(R) = {A : det(A) = 1}
```

### 3.5 The Sign Homomorphism

$$\text{sgn}: S_n \to \{1, -1\}$$

where $\text{sgn}(\sigma) = (-1)^k$ if $\sigma$ is a product of $k$ transpositions.

| Permutation in $S_3$ | Sign |
|----------------------|------|
| $e$ | $+1$ |
| $(1\;2)$ | $-1$ |
| $(1\;3)$ | $-1$ |
| $(2\;3)$ | $-1$ |
| $(1\;2\;3)$ | $+1$ |
| $(1\;3\;2)$ | $+1$ |

$\ker(\text{sgn}) = A_n$ (alternating group — even permutations).

### 3.6 Reduction Modulo $n$

$$\pi: \mathbb{Z} \to \mathbb{Z}_n, \quad \pi(a) = a \bmod n$$

This is a **surjective** homomorphism with $\ker(\pi) = n\mathbb{Z}$.

### 3.7 Exponentiation Map

If $G$ is abelian, the map $\phi: G \to G$ defined by $\phi(g) = g^n$ is a homomorphism:

$$\phi(ab) = (ab)^n = a^n b^n = \phi(a)\phi(b)$$

> **Warning:** This only works for abelian groups!

---

## 4. Composition of Homomorphisms

**Theorem 2.2.** If $\phi: G \to H$ and $\psi: H \to K$ are homomorphisms, then $\psi \circ \phi: G \to K$ is a homomorphism.

```
    φ          ψ
  G ───→ H ───→ K
  │               │
  └───────────────┘
       ψ ∘ φ
  
  (ψ ∘ φ)(ab) = ψ(φ(ab)) = ψ(φ(a)φ(b)) = ψ(φ(a))ψ(φ(b)) = (ψ∘φ)(a)(ψ∘φ)(b)
```

---

## 5. Image and Preimage under Homomorphisms

**Theorem 2.3.** Let $\phi: G \to H$ be a homomorphism.

| If... | Then... |
|-------|---------|
| $K \leq G$ | $\phi(K) \leq H$ |
| $L \leq H$ | $\phi^{-1}(L) \leq G$ |
| $K \trianglelefteq G$ and $\phi$ surjective | $\phi(K) \trianglelefteq H$ |
| $L \trianglelefteq H$ | $\phi^{-1}(L) \trianglelefteq G$ |

```
  Subgroup mapping:
  
         G                    H
  ┌──────────────┐    ┌──────────────┐
  │              │    │              │
  │   K ≤ G ─────────→ φ(K) ≤ H     │
  │              │    │              │
  └──────────────┘    └──────────────┘
  
  Subgroups map to subgroups!
  Normal subgroups map to normal subgroups (if φ surjective)!
```

---

## 6. Types of Homomorphisms

| Name | Condition | Set Theory |
|------|-----------|-----------|
| **Monomorphism** | Injective (one-to-one) | $\phi(a) = \phi(b) \implies a = b$ |
| **Epimorphism** | Surjective (onto) | For all $h \in H$, $\exists g: \phi(g) = h$ |
| **Isomorphism** | Bijective | Both injective and surjective |
| **Endomorphism** | $G \to G$ | Homomorphism to itself |
| **Automorphism** | Bijective, $G \to G$ | Isomorphism to itself |

```
  Classification of homomorphisms φ: G → H
  
              Surjective?
              │
        Yes ──┤── No
        │     │
    Injective?│   Injective?
    │         │       │
  Yes─┤─No    │   Yes─┤─No
  │   │       │   │   │
  ISO EPIM    │  MONO  General
              │        HOM
```

---

## 7. Constructing Homomorphisms from Cyclic Groups

**Theorem 2.4.** Let $G = \langle a \rangle$ be cyclic of order $n$, and $H$ any group. A function $\phi: G \to H$ defined by $\phi(a) = h$ extends to a homomorphism if and only if $h^n = e_H$ (equivalently, $|h|$ divides $n$).

**Example:** Homomorphisms $\phi: \mathbb{Z}_6 \to \mathbb{Z}_6$:

$\phi$ determined by $\phi(1) = k$. Need $6k \equiv 0 \pmod{6}$ — always true. So there are 6 homomorphisms.

| $\phi(1)$ | $\phi$ explicitly | $\ker \phi$ | $\text{im}\,\phi$ |
|-----------|------------------|------------|-------------------|
| 0 | $\phi(x) = 0$ | $\mathbb{Z}_6$ | $\{0\}$ |
| 1 | $\phi(x) = x$ | $\{0\}$ | $\mathbb{Z}_6$ |
| 2 | $\phi(x) = 2x$ | $\{0, 3\}$ | $\{0, 2, 4\}$ |
| 3 | $\phi(x) = 3x$ | $\{0, 2, 4\}$ | $\{0, 3\}$ |
| 4 | $\phi(x) = 4x$ | $\{0, 3\}$ | $\{0, 2, 4\}$ |
| 5 | $\phi(x) = 5x$ | $\{0\}$ | $\mathbb{Z}_6$ |

---

## 8. Homomorphisms and Order

```
  Order relationships:
  
  ┌──────────────────────────────────────────────┐
  │  |φ(a)|  divides  |a|                        │
  │                                               │
  │  |φ(a)|  divides  |H|   (if H finite)        │
  │                                               │
  │  Therefore |φ(a)| divides gcd(|a|, |H|)      │
  └──────────────────────────────────────────────┘
  
  Example: φ: Z₆ → Z₄
  
  |φ(1)| divides gcd(6, 4) = 2
  So φ(1) ∈ {0, 2}  (elements of order dividing 2 in Z₄)
  
  Only 2 homomorphisms exist!
```

---

## 9. Real-World Applications

| Application | Homomorphism |
|-------------|-------------|
| Parity check | $\text{sgn}: S_n \to \{+1, -1\}$ |
| Determinant | $\det: GL_n \to \mathbb{R}^\times$ |
| Modular reduction | $\mathbb{Z} \to \mathbb{Z}_n$ |
| Logarithm | $\log: (\mathbb{R}^+, \cdot) \to (\mathbb{R}, +)$ |
| Trace (additive) | $\text{tr}: M_n(\mathbb{R}) \to \mathbb{R}$ |
| Exponential | $\exp: (\mathbb{R}, +) \to (\mathbb{R}^+, \cdot)$ |

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Homomorphism | $\phi(ab) = \phi(a)\phi(b)$ |
| $\phi(e_G)$ | $= e_H$ |
| $\phi(a^{-1})$ | $= \phi(a)^{-1}$ |
| $\|\phi(a)\|$ | divides $\|a\|$ |
| Composition | $\psi \circ \phi$ is a homomorphism |
| Subgroups map forward | $K \leq G \implies \phi(K) \leq H$ |
| Subgroups pull back | $L \leq H \implies \phi^{-1}(L) \leq G$ |
| From cyclic groups | Determined by image of generator |

---

## Quick Revision Questions

1. **Verify** that $\det: GL_2(\mathbb{R}) \to \mathbb{R}^\times$ is a homomorphism.

2. **Count** all homomorphisms from $\mathbb{Z}_{12}$ to $\mathbb{Z}_4$.

3. **Prove** that $\phi(a^{-1}) = \phi(a)^{-1}$ for any group homomorphism.

4. **Show** that the map $\phi: \mathbb{Z} \to \mathbb{Z}_n$ given by $\phi(a) = a \bmod n$ is a surjective homomorphism.

5. **If** $\phi: G \to H$ is a homomorphism and $|G| = 15$, $|H| = 10$, what are the possible values of $|\text{im}\,\phi|$?

6. **Prove** or disprove: The map $\phi: S_3 \to S_3$ given by $\phi(\sigma) = \sigma^2$ is a homomorphism.

---

[← Previous: Generators](../01-Groups-Basics/05-generators.md) | [Back to README](../README.md) | [Next: Isomorphisms →](02-isomorphisms.md)
