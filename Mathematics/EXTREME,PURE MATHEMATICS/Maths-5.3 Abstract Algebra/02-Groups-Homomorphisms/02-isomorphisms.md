# Chapter 2.2 — Isomorphisms

[← Previous: Group Homomorphisms](01-group-homomorphisms.md) | [Back to README](../README.md) | [Next: Kernel and Image →](03-kernel-and-image.md)

---

## Overview

Two groups are **isomorphic** if they are structurally identical — they differ only in the labelling of their elements. An isomorphism is a bijective homomorphism. Proving (or disproving) isomorphism is one of the most important tasks in group theory.

---

## 1. Definition

**Definition.** A homomorphism $\phi: G \to H$ is an **isomorphism** if $\phi$ is bijective (injective and surjective). We write $G \cong H$ and say "$G$ is isomorphic to $H$".

**Definition.** An isomorphism $\phi: G \to G$ (from a group to itself) is called an **automorphism**.

```
  Isomorphism = Bijective Homomorphism
  
       φ
  G ═══════► H       (bijective)
  
  │ a ──→ φ(a)  │    one-to-one
  │ b ──→ φ(b)  │    onto
  │ ab──→ φ(a)φ(b) │ operation-preserving
  
  G and H have identical algebraic structure!
```

---

## 2. What Isomorphism Preserves

If $\phi: G \to H$ is an isomorphism, then $G$ and $H$ share **all** group-theoretic properties:

| Property | Preserved? |
|----------|-----------|
| Order of group | $\|G\| = \|H\|$ |
| Abelian/non-abelian | Yes |
| Cyclic/non-cyclic | Yes |
| Order of elements | $\|a\| = \|\phi(a)\|$ |
| Number of elements of each order | Yes |
| Number of subgroups of each order | Yes |
| Subgroup lattice | Same structure |
| Centre | $\phi(Z(G)) = Z(H)$ |
| Solvability | Yes |

---

## 3. Proving Two Groups Are Isomorphic

**Strategy:** Construct an explicit bijective homomorphism.

### Example 1: $\mathbb{Z}_4 \cong \langle i \rangle$ in $\mathbb{C}^\times$

Define $\phi: \mathbb{Z}_4 \to \langle i \rangle$ by $\phi(k) = i^k$:

```
  φ: Z₄ → ⟨i⟩
  
  0 ──→ i⁰ = 1
  1 ──→ i¹ = i
  2 ──→ i² = -1
  3 ──→ i³ = -i
  
  Check: φ(2 + 3) = φ(1) = i
         φ(2)·φ(3) = (-1)(-i) = i  ✓
```

### Example 2: $(\mathbb{R}, +) \cong (\mathbb{R}^+, \cdot)$ via $\exp$

$$\phi: (\mathbb{R}, +) \to (\mathbb{R}^+, \cdot), \quad \phi(x) = e^x$$

- **Homomorphism:** $\phi(x + y) = e^{x+y} = e^x \cdot e^y = \phi(x)\phi(y)$ ✓
- **Injective:** $e^x = e^y \implies x = y$ ✓
- **Surjective:** For any $r > 0$, $\phi(\ln r) = r$ ✓

Inverse: $\phi^{-1} = \ln: (\mathbb{R}^+, \cdot) \to (\mathbb{R}, +)$

### Example 3: $S_3 \cong D_3$

The symmetric group $S_3$ and the dihedral group $D_3$ are isomorphic:

```
  S₃ = {e, (12), (13), (23), (123), (132)}
  D₃ = {e, r, r², s, rs, r²s}
  
  Correspondence:
  e    ↔  e
  (123) ↔  r
  (132) ↔  r²
  (12)  ↔  s
  (13)  ↔  r²s
  (23)  ↔  rs
```

---

## 4. Proving Two Groups Are NOT Isomorphic

**Strategy:** Find a group-theoretic property that differs.

### Invariants to Check (in order of ease)

```
  ┌─────────────────────────────────────────┐
  │ 1. |G| ≠ |H|  →  Not isomorphic        │
  │                                          │
  │ 2. One abelian, other not               │
  │                                          │
  │ 3. Different order profiles              │
  │    (count elements of each order)        │
  │                                          │
  │ 4. One cyclic, other not                │
  │                                          │
  │ 5. Different number of subgroups        │
  │    of some order                         │
  │                                          │
  │ 6. Different centre sizes               │
  └─────────────────────────────────────────┘
```

### Example: $\mathbb{Z}_4 \not\cong V_4$

Both have order 4, both are abelian. But:

| Property | $\mathbb{Z}_4$ | $V_4$ |
|----------|----------------|-------|
| Elements of order 1 | 1 | 1 |
| Elements of order 2 | 1 | 3 |
| Elements of order 4 | 2 | 0 |
| Cyclic? | Yes | No |

Different order profiles $\implies \mathbb{Z}_4 \not\cong V_4$.

### Example: $Q_8 \not\cong D_4$

Both have order 8, both non-abelian.

| Property | $Q_8$ | $D_4$ |
|----------|-------|-------|
| Elements of order 1 | 1 | 1 |
| Elements of order 2 | 1 ($-1$) | 5 ($r^2, s, rs, r^2s, r^3s$) |
| Elements of order 4 | 6 | 2 |

Different order profiles $\implies Q_8 \not\cong D_4$. ✓

---

## 5. The Automorphism Group

**Definition.** $\text{Aut}(G) = \{\phi: G \to G : \phi \text{ is an isomorphism}\}$ under composition forms a group.

### Key Results

| Group $G$ | $\text{Aut}(G)$ |
|-----------|-----------------|
| $\mathbb{Z}_n$ | $\mathbb{Z}_n^\times$ (order $\varphi(n)$) |
| $\mathbb{Z}$ | $\mathbb{Z}_2 = \{\text{id}, x \mapsto -x\}$ |
| $\mathbb{Z}_p$ ($p$ prime) | $\mathbb{Z}_{p-1}$ |
| $V_4$ | $S_3$ |
| $S_3$ | $S_3$ (inner automorphisms only) |

**Inner Automorphisms:** For $g \in G$, define $\phi_g: G \to G$ by $\phi_g(x) = gxg^{-1}$ (conjugation). Then $\text{Inn}(G) = \{\phi_g : g \in G\} \trianglelefteq \text{Aut}(G)$.

```
  Automorphism group structure:
  
  Inn(G) ◁ Aut(G)
  
  Inn(G) ≅ G/Z(G)
  
  Aut(G)/Inn(G) ≅ Out(G)  (outer automorphism group)
```

---

## 6. Classification of Groups of Small Order

| Order | Groups (up to isomorphism) | Count |
|-------|---------------------------|-------|
| 1 | $\{e\}$ | 1 |
| 2 | $\mathbb{Z}_2$ | 1 |
| 3 | $\mathbb{Z}_3$ | 1 |
| 4 | $\mathbb{Z}_4$, $V_4$ | 2 |
| 5 | $\mathbb{Z}_5$ | 1 |
| 6 | $\mathbb{Z}_6$, $S_3$ | 2 |
| 7 | $\mathbb{Z}_7$ | 1 |
| 8 | $\mathbb{Z}_8$, $\mathbb{Z}_4 \times \mathbb{Z}_2$, $\mathbb{Z}_2^3$, $D_4$, $Q_8$ | 5 |

```
  Groups of order ≤ 8:
  
  Order:  1   2   3   4   5   6   7   8
  Count:  1   1   1   2   1   2   1   5
  
  Observations:
  • Prime order → unique group (cyclic)
  • Order 4: Z₄ vs V₄ (cyclic vs non-cyclic)
  • Order 6: Z₆ vs S₃ (abelian vs non-abelian)
  • Order 8: richest variety so far
```

---

## 7. Cayley's Theorem

**Theorem 2.5 (Cayley).** Every group $G$ is isomorphic to a subgroup of some symmetric group. If $|G| = n$, then $G$ is isomorphic to a subgroup of $S_n$.

*Proof sketch.* Define $\phi: G \to S_G$ (permutations of the set $G$) by $\phi(g) = \lambda_g$ where $\lambda_g(x) = gx$ (left multiplication). Then $\phi$ is an injective homomorphism. $\square$

```
  Cayley's Theorem — Embedding G into Sₙ:
  
  G = {e, a, b, ...}    (|G| = n)
       │
       ▼
  Each g ∈ G acts on G by left multiplication:
  
  λ_g: G → G,   λ_g(x) = gx
  
  This is a permutation of G!
  
  φ: G → S_G,   φ(g) = λ_g
  
  φ is injective: λ_g = λ_h ⟹ gx = hx for all x ⟹ g = h
  φ is a homomorphism: λ_{gh}(x) = ghx = λ_g(hx) = (λ_g ∘ λ_h)(x)
  
  Therefore G ≅ φ(G) ≤ S_G ≅ Sₙ
```

---

## 8. Real-World Applications

| Application | Isomorphism |
|-------------|-------------|
| Circuit symmetries | Identifying equivalent circuit layouts |
| Cryptographic equivalence | Protocols based on isomorphic groups are equally secure |
| Chemical stereoisomers | Molecules with same bond structure |
| Data structure design | Isomorphic algebraic structures → same algorithms |

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Isomorphism | Bijective homomorphism |
| $G \cong H$ | Structurally identical groups |
| Preserved invariants | Order, abelianness, element orders, subgroup structure |
| Disproving $G \cong H$ | Find a differing invariant |
| $\text{Aut}(G)$ | Group of automorphisms of $G$ |
| $\text{Inn}(G)$ | Inner automorphisms, $\cong G/Z(G)$ |
| $\text{Aut}(\mathbb{Z}_n)$ | $\cong \mathbb{Z}_n^\times$ |
| Cayley's theorem | $G \hookrightarrow S_{\|G\|}$ |

---

## Quick Revision Questions

1. **Prove** that $(\mathbb{R}, +)$ and $(\mathbb{R}^+, \cdot)$ are isomorphic using the exponential map.

2. **Show** that $\mathbb{Z}_4$ and $V_4$ are not isomorphic by comparing element orders.

3. **Find** $\text{Aut}(\mathbb{Z}_6)$.

4. **State** Cayley's theorem and embed $\mathbb{Z}_3$ into $S_3$ explicitly.

5. **Determine** all groups of order 6, up to isomorphism, and prove there are exactly two.

6. **If** $G \cong H$ and $G$ has exactly 3 subgroups, how many subgroups does $H$ have? Why?

---

[← Previous: Group Homomorphisms](01-group-homomorphisms.md) | [Back to README](../README.md) | [Next: Kernel and Image →](03-kernel-and-image.md)
