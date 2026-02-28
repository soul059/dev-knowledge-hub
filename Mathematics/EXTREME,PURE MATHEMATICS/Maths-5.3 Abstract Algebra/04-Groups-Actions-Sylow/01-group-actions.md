# Chapter 4.1 — Group Actions

[← Previous: Direct Products](../03-Groups-Structure/05-direct-products.md) | [Back to README](../README.md) | [Next: Orbit-Stabiliser Theorem →](02-orbit-stabilizer-theorem.md)

---

## Overview

A **group action** is one of the most powerful ideas in algebra: it lets a group $G$ "act on" a set $X$ by permutations. This connects abstract algebra to geometry, combinatorics, and physics. Many fundamental results—Lagrange, Sylow, Burnside—are best understood through group actions.

---

## 1. Definition

**Definition.** A (left) **action** of a group $G$ on a set $X$ is a function $G \times X \to X$, written $(g, x) \mapsto g \cdot x$, satisfying:

1. **Identity:** $e \cdot x = x$ for all $x \in X$
2. **Compatibility:** $(gh) \cdot x = g \cdot (h \cdot x)$ for all $g, h \in G$, $x \in X$

We say "$G$ **acts on** $X$", or $X$ is a **$G$-set**.

```
  Group Action Axioms:
  
  (A1)  e · x = x               ∀ x ∈ X
  (A2)  (gh) · x = g · (h · x)  ∀ g,h ∈ G, x ∈ X
  
  Equivalently: the map  G → Sym(X)
                         g ↦ σ_g   where σ_g(x) = g·x
  is a group homomorphism.
```

---

## 2. Equivalent Formulation

**Theorem 4.1.** An action of $G$ on $X$ is the same as a group homomorphism $\varphi: G \to \text{Sym}(X)$.

Given an action, define $\varphi(g)(x) = g \cdot x$. Then:
- $\varphi(e) = \text{id}_X$ — follows from (A1)
- $\varphi(gh) = \varphi(g) \circ \varphi(h)$ — follows from (A2)

The **kernel** of the action is $\ker \varphi = \{g \in G : g \cdot x = x \text{ for all } x \in X\}$.

An action is **faithful** (or effective) if $\ker \varphi = \{e\}$.

---

## 3. Examples of Group Actions

### 3.1. Left Regular Action

$G$ acts on itself by left multiplication: $g \cdot x = gx$.

- This is **faithful**: if $gx = x$ for all $x$, then $g = e$.
- This gives **Cayley's Theorem**: every group embeds in $\text{Sym}(G)$.

### 3.2. Conjugation Action

$G$ acts on itself by conjugation: $g \cdot x = gxg^{-1}$.

- Fixed points = centre $Z(G)$
- Orbits = conjugacy classes
- Kernel = $Z(G)$

### 3.3. Action on Subgroups by Conjugation

$G$ acts on $\text{Sub}(G) = \{\text{subgroups of } G\}$ by $g \cdot H = gHg^{-1}$.

- Fixed points = normal subgroups
- Stabiliser of $H$ = normaliser $N_G(H)$

### 3.4. Action on Cosets

If $H \leq G$, then $G$ acts on $G/H = \{gH : g \in G\}$ by left multiplication: $g \cdot (aH) = (ga)H$.

- This is **always transitive** (only one orbit)

### 3.5. $S_n$ Acts on $\{1, 2, \ldots, n\}$

The natural action: $\sigma \cdot i = \sigma(i)$.

### 3.6. $D_n$ Acts on Vertices of Regular $n$-gon

$D_n = \langle r, s \mid r^n = s^2 = e, srs = r^{-1} \rangle$ acts on $\{1, 2, \ldots, n\}$ (vertices).

```
  Example: D₄ acting on vertices of a square
  
      1 -------- 2
      |          |
      |          |
      4 -------- 3
  
  r (90° rotation):   1→2, 2→3, 3→4, 4→1
  s (reflection):     1→1, 2→4, 3→3, 4→2  (about vertical axis)
  
  r = (1234), s = (24)  in S₄
```

### 3.7. Matrix Groups Acting on Vectors

$GL_n(\mathbb{R})$ acts on $\mathbb{R}^n$ by matrix multiplication: $A \cdot \mathbf{v} = A\mathbf{v}$.

---

## 4. Summary Table of Key Actions

| Group | Set | Action $g \cdot x$ | Kernel | Faithful? |
|-------|-----|---------------------|--------|-----------|
| $G$ | $G$ | $gx$ (left mult.) | $\{e\}$ | Yes |
| $G$ | $G$ | $gxg^{-1}$ (conj.) | $Z(G)$ | Iff $Z(G) = \{e\}$ |
| $G$ | $G/H$ | $g(aH) = gaH$ | $\bigcap_{g} gHg^{-1}$ | Sometimes |
| $G$ | Sub($G$) | $gHg^{-1}$ | — | Sometimes |
| $S_n$ | $\{1,\ldots,n\}$ | $\sigma(i)$ | $\{e\}$ | Yes |
| $D_n$ | vertices | geometric | $\{e\}$ (usually) | Yes (usually) |
| $GL_n$ | $\mathbb{R}^n$ | $A\mathbf{v}$ | $\{I\}$ | Yes |

---

## 5. Orbits and Stabilisers

### 5.1. Orbit

The **orbit** of $x \in X$ under $G$ is:

$$\text{Orb}(x) = G \cdot x = \{g \cdot x : g \in G\}$$

### 5.2. Stabiliser (Isotropy Subgroup)

The **stabiliser** of $x \in X$ is:

$$\text{Stab}(x) = G_x = \{g \in G : g \cdot x = x\}$$

**Lemma 4.2.** $\text{Stab}(x)$ is a subgroup of $G$.

*Proof.* 
- $e \cdot x = x$, so $e \in \text{Stab}(x)$.
- If $g, h \in \text{Stab}(x)$, then $(gh) \cdot x = g \cdot (h \cdot x) = g \cdot x = x$.
- If $g \cdot x = x$, then $x = g^{-1} \cdot (g \cdot x) = g^{-1} \cdot x$, so $g^{-1} \in \text{Stab}(x)$. $\blacksquare$

### 5.3. Transitive Actions

An action is **transitive** if there is exactly **one orbit**: for any $x, y \in X$, there exists $g \in G$ with $g \cdot x = y$.

```
  Orbits partition X:
  
  X = ┌──────────────────────────────────────┐
      │  Orb(x₁)  │  Orb(x₂)  │  Orb(x₃)  │
      │  ● ● ●    │  ○ ○      │  △ △ △ △   │
      └──────────────────────────────────────┘
  
  Transitive = only one orbit (whole set)
```

---

## 6. Fixed Points

The **fixed-point set** of $g \in G$ is:

$$\text{Fix}(g) = X^g = \{x \in X : g \cdot x = x\}$$

The set of elements fixed by the entire group:

$$X^G = \{x \in X : g \cdot x = x \text{ for all } g \in G\}$$

---

## 7. Examples Worked Out

### D₃ Acting on Vertices of Triangle

$D_3 = \{e, r, r^2, s, sr, sr^2\}$ acting on $\{1, 2, 3\}$:

| $g$ | $g \cdot 1$ | $g \cdot 2$ | $g \cdot 3$ | Fix($g$) |
|-----|-----------|-----------|-----------|----------|
| $e$ | 1 | 2 | 3 | $\{1, 2, 3\}$ |
| $r$ | 2 | 3 | 1 | $\emptyset$ |
| $r^2$ | 3 | 1 | 2 | $\emptyset$ |
| $s$ | 1 | 3 | 2 | $\{1\}$ |
| $sr$ | 3 | 2 | 1 | $\{2\}$ |
| $sr^2$ | 2 | 1 | 3 | $\{3\}$ |

- **Orbits:** One orbit $\{1, 2, 3\}$ — action is transitive.
- **Stabilisers:** $\text{Stab}(1) = \{e, s\}$, $\text{Stab}(2) = \{e, sr\}$, $\text{Stab}(3) = \{e, sr^2\}$.
- $|\text{Orb}(1)| \cdot |\text{Stab}(1)| = 3 \cdot 2 = 6 = |D_3|$. ✓

---

## 8. Right Actions and Other Variants

A **right action** $X \times G \to X$, $(x, g) \mapsto x \cdot g$, satisfies:
- $x \cdot e = x$
- $x \cdot (gh) = (x \cdot g) \cdot h$

Right actions correspond to **anti-homomorphisms** $G \to \text{Sym}(X)$ (or equivalently, homomorphisms $G^{\text{op}} \to \text{Sym}(X)$).

Every right action gives a left action by $g \cdot x = x \cdot g^{-1}$ (and vice versa).

---

## Summary Table

| Concept | Definition / Key Result |
|---------|------------------------|
| Group action | $G \times X \to X$ satisfying $e \cdot x = x$, $(gh) \cdot x = g \cdot (h \cdot x)$ |
| Equivalent to | Homomorphism $\varphi: G \to \text{Sym}(X)$ |
| Faithful | $\ker \varphi = \{e\}$ |
| Orbit | $\text{Orb}(x) = \{g \cdot x : g \in G\}$ |
| Stabiliser | $\text{Stab}(x) = \{g : g \cdot x = x\} \leq G$ |
| Transitive | Only one orbit |
| Fixed points | $\text{Fix}(g) = \{x : g \cdot x = x\}$ |

---

## Quick Revision Questions

1. **Verify** that the conjugation action of $S_3$ on itself satisfies the axioms.

2. **Find** all orbits and stabilisers for $\mathbb{Z}_4 = \langle r \rangle$ acting on the vertices of a square by rotation.

3. **Prove** that the kernel of a group action is a normal subgroup of $G$.

4. **Give** an example of a non-faithful action.

5. **Show** that the action of $G$ on $G/H$ by left multiplication is transitive.

6. **List** all elements of $\text{Fix}(g)$ for each $g \in D_4$ acting on the vertices of a square.

---

[← Previous: Direct Products](../03-Groups-Structure/05-direct-products.md) | [Back to README](../README.md) | [Next: Orbit-Stabiliser Theorem →](02-orbit-stabilizer-theorem.md)
