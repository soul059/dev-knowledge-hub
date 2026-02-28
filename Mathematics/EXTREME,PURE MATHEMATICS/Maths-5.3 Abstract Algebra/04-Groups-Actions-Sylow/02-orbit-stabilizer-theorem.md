# Chapter 4.2 — Orbit-Stabiliser Theorem

[← Previous: Group Actions](01-group-actions.md) | [Back to README](../README.md) | [Next: Class Equation →](03-class-equation.md)

---

## Overview

The **Orbit-Stabiliser Theorem** is the central result connecting orbits and stabilisers. It tells us that $|\text{Orb}(x)| = [G : \text{Stab}(x)]$, giving a powerful counting tool that underpins the class equation, Burnside's lemma, and the Sylow theorems.

---

## 1. Statement

**Theorem 4.3 (Orbit-Stabiliser Theorem).** Let $G$ act on $X$, and let $x \in X$. Then:

$$|\text{Orb}(x)| = [G : \text{Stab}(x)] = \frac{|G|}{|\text{Stab}(x)|}$$

(when $G$ is finite).

```
  ┌───────────────────────────────────────────────┐
  │  |Orb(x)| · |Stab(x)| = |G|                  │
  │                                                │
  │  "Orbit size × Stabiliser size = Group order"  │
  └───────────────────────────────────────────────┘
```

---

## 2. Proof

Define $\phi: G/\text{Stab}(x) \to \text{Orb}(x)$ by $\phi(g \cdot \text{Stab}(x)) = g \cdot x$.

**Well-defined:** If $g \cdot \text{Stab}(x) = h \cdot \text{Stab}(x)$, then $h^{-1}g \in \text{Stab}(x)$, so $h^{-1}g \cdot x = x$, hence $g \cdot x = h \cdot x$. ✓

**Injective:** If $g \cdot x = h \cdot x$, then $h^{-1}g \cdot x = x$, so $h^{-1}g \in \text{Stab}(x)$, hence $g \cdot \text{Stab}(x) = h \cdot \text{Stab}(x)$. ✓

**Surjective:** Every $y \in \text{Orb}(x)$ has $y = g \cdot x$ for some $g$, so $y = \phi(g \cdot \text{Stab}(x))$. ✓

Therefore $\phi$ is a bijection, so $|\text{Orb}(x)| = |G/\text{Stab}(x)| = [G : \text{Stab}(x)]$. $\blacksquare$

```
  Bijection:   Cosets of Stab(x)   ↔   Elements of Orb(x)
  
  g·Stab(x)    ↦    g·x
  
  {e,s}·Stab(1)=Stab(1)  →  e·1 = 1
  {r,sr²}·Stab(1)         →  r·1 = 2
  {r²,sr}·Stab(1)         →  r²·1 = 3      (D₃ on triangle)
```

---

## 3. Consequences

### 3.1. Orbit Sizes Divide $|G|$

Since $|\text{Orb}(x)| = [G : \text{Stab}(x)]$, every orbit size divides $|G|$.

### 3.2. The Counting Formula

Since orbits partition $X$:

$$|X| = \sum_{\text{orbits}} |\text{Orb}(x_i)|$$

where the sum is over one representative $x_i$ from each orbit.

### 3.3. Stabilisers of Elements in the Same Orbit

**Lemma 4.4.** If $y = g \cdot x$, then $\text{Stab}(y) = g \cdot \text{Stab}(x) \cdot g^{-1}$.

Stabilisers of elements in the same orbit are **conjugate** subgroups.

---

## 4. Examples

### 4.1. $S_4$ Acting on $\{1, 2, 3, 4\}$

- $\text{Orb}(1) = \{1, 2, 3, 4\}$ (transitive), so $|\text{Orb}(1)| = 4$.
- $|\text{Stab}(1)| = |S_4|/4 = 24/4 = 6$.
- Indeed, $\text{Stab}(1) = \{\sigma \in S_4 : \sigma(1) = 1\} \cong S_3$, and $|S_3| = 6$. ✓

### 4.2. Conjugation Action of $S_3$

$S_3$ acts on itself by conjugation. Orbits = conjugacy classes.

| Conjugacy class | Representative $x$ | $\|\text{Orb}(x)\|$ | $\|\text{Stab}(x)\|$ | Check |
|----------------|---------------------|---------------------|----------------------|-------|
| $\{e\}$ | $e$ | 1 | 6 | $1 \times 6 = 6$ ✓ |
| $\{(12), (13), (23)\}$ | $(12)$ | 3 | 2 | $3 \times 2 = 6$ ✓ |
| $\{(123), (132)\}$ | $(123)$ | 2 | 3 | $2 \times 3 = 6$ ✓ |

**Total:** $1 + 3 + 2 = 6 = |S_3|$. ✓

### 4.3. Rotational Symmetries of a Cube

$G$ = rotation group of the cube, $|G| = 24$.

$G$ acts on the set of 6 faces.

- Orbits: all faces form one orbit (can rotate any face to any other).
- $|\text{Orb}(f)| = 6$, so $|\text{Stab}(f)| = 24/6 = 4$.
- Indeed, 4 rotations fix a given face (rotations about the axis through that face by 0°, 90°, 180°, 270°).

```
  Cube with labelled faces:
  
        ┌───────┐
       /  TOP  /│
      /       / │
     ┌───────┐  │
     │       │  │  ← RIGHT
     │ FRONT │  ┘
     │       │ /
     └───────┘
  
  |G| = 24 rotations
  
  Acting on:      |Orb| × |Stab| = |G|
  6 faces:          6   ×    4   =  24  ✓
  8 vertices:       8   ×    3   =  24  ✓
  12 edges:        12   ×    2   =  24  ✓
  4 main diag.:     4   ×    6   =  24  ✓
```

---

## 5. Burnside's Lemma (Cauchy-Frobenius)

**Theorem 4.5 (Burnside's Lemma).** The number of orbits under a finite group action is:

$$|\text{orbits}| = \frac{1}{|G|} \sum_{g \in G} |\text{Fix}(g)|$$

"The number of orbits = average number of fixed points."

### Application: Counting Colourings

**Example.** How many distinct necklaces can be made with 4 beads using 2 colours?

$G = \mathbb{Z}_4$ (rotational symmetry), $X$ = set of $2^4 = 16$ colourings.

| $g$ | Description | $\|\text{Fix}(g)\|$ |
|-----|-------------|---------------------|
| $e$ | identity | $2^4 = 16$ |
| $r$ | rotate 90° | $2^1 = 2$ (all same colour) |
| $r^2$ | rotate 180° | $2^2 = 4$ (opposite pairs equal) |
| $r^3$ | rotate 270° | $2^1 = 2$ |

$$\text{Number of orbits} = \frac{1}{4}(16 + 2 + 4 + 2) = \frac{24}{4} = 6$$

Six distinct necklaces.

---

## 6. Proof of Burnside's Lemma

Count the set $S = \{(g, x) : g \cdot x = x\}$ in two ways:

$$|S| = \sum_{g \in G} |\text{Fix}(g)| = \sum_{x \in X} |\text{Stab}(x)|$$

For the right side, using orbit-stabiliser:

$$\sum_{x \in X} |\text{Stab}(x)| = \sum_{x \in X} \frac{|G|}{|\text{Orb}(x)|} = |G| \sum_{\text{orbits}} \sum_{x \in \text{Orb}} \frac{1}{|\text{Orb}|} = |G| \cdot |\text{orbits}|$$

Equating: $\sum_{g} |\text{Fix}(g)| = |G| \cdot |\text{orbits}|$, giving the result. $\blacksquare$

---

## 7. Orbits Partition the Set

**Theorem 4.6.** The orbits of a group action partition $X$:

$$X = \bigsqcup_{i} \text{Orb}(x_i)$$

where $x_i$ are orbit representatives.

*Proof.* Define $x \sim y$ iff $y = g \cdot x$ for some $g$. This is an equivalence relation:
- Reflexive: $x = e \cdot x$
- Symmetric: $y = g \cdot x \implies x = g^{-1} \cdot y$
- Transitive: $y = g \cdot x,\ z = h \cdot y \implies z = (hg) \cdot x$

The equivalence classes are precisely the orbits. $\blacksquare$

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Orbit-Stabiliser | $\|\text{Orb}(x)\| \cdot \|\text{Stab}(x)\| = \|G\|$ |
| Orbit sizes | Divide $\|G\|$ |
| Same orbit | $\text{Stab}(g \cdot x) = g\,\text{Stab}(x)\,g^{-1}$ |
| Orbits partition $X$ | $\|X\| = \sum \|\text{Orb}(x_i)\|$ |
| Burnside | $\|\text{orbits}\| = \frac{1}{\|G\|}\sum_{g}\|\text{Fix}(g)\|$ |
| Application | Counting distinct objects under symmetry |

---

## Quick Revision Questions

1. **Compute** $|\text{Stab}(x)|$ when $S_5$ acts on $\{1,2,3,4,5\}$ and $x = 3$.

2. **Use** the Orbit-Stabiliser theorem to show $|C_G(a)| \cdot |\text{cl}(a)| = |G|$ where $\text{cl}(a)$ is the conjugacy class of $a$.

3. **Apply** Burnside's lemma to count the number of distinct colourings of the faces of a triangle using 3 colours, under the action of $D_3$.

4. **Prove** that in a transitive action, $|\text{Stab}(x)| = |G|/|X|$ for any $x$.

5. **Find** all orbit sizes when $D_4$ acts on the diagonals of a square.

6. **Show** that orbits form an equivalence relation by verifying reflexivity, symmetry, and transitivity.

---

[← Previous: Group Actions](01-group-actions.md) | [Back to README](../README.md) | [Next: Class Equation →](03-class-equation.md)
