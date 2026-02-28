# Chapter 3.3 — Quotient Groups

[← Previous: Normal Subgroups](02-normal-subgroups.md) | [Back to README](../README.md) | [Next: Simple Groups →](04-simple-groups.md)

---

## Overview

Given a normal subgroup $N \trianglelefteq G$, we can "collapse" $N$ to a single point and form a new, smaller group called the **quotient group** $G/N$. This construction is one of the most powerful tools in algebra — it lets us study $G$ by breaking it into simpler pieces.

---

## 1. Construction

**Definition.** Let $N \trianglelefteq G$. The **quotient group** (or factor group) is:

$$G/N = \{gN : g \in G\}$$

with the operation $(aN)(bN) = (ab)N$.

**Why normality is needed:** The operation must be well-defined. If $aN = a'N$ and $bN = b'N$, we need $(ab)N = (a'b')N$. This holds precisely when $N$ is normal.

```
  Quotient Group Construction:
  
  Step 1: Start with G and N ◁ G
  
  Step 2: Partition G into cosets of N:
  ┌─────┬─────┬─────┬─────┐
  │  N  │ g₁N │ g₂N │ g₃N │  ← elements of G/N
  └─────┴─────┴─────┴─────┘
  
  Step 3: Define multiplication:
  (g₁N)(g₂N) = (g₁g₂)N
  
  Step 4: Verify this is well-defined (requires N ◁ G)
  
  Result: G/N is a group of order |G|/|N|
```

---

## 2. Properties of Quotient Groups

| Property | Value |
|----------|-------|
| Identity | $eN = N$ |
| Inverse of $gN$ | $(gN)^{-1} = g^{-1}N$ |
| Order | $\|G/N\| = [G:N] = \|G\|/\|N\|$ |
| Natural projection | $\pi: G \to G/N$, $\pi(g) = gN$ is a surjective homomorphism |
| $\ker \pi$ | $= N$ |

---

## 3. Examples

### Example 1: $\mathbb{Z}/n\mathbb{Z}$

$G = \mathbb{Z}$, $N = n\mathbb{Z}$.

Elements: $\{0 + n\mathbb{Z},\, 1 + n\mathbb{Z},\, \ldots,\, (n-1) + n\mathbb{Z}\}$

Operation: $(a + n\mathbb{Z}) + (b + n\mathbb{Z}) = (a+b) + n\mathbb{Z}$

This is just $\mathbb{Z}_n$! So $\mathbb{Z}/n\mathbb{Z} \cong \mathbb{Z}_n$.

### Example 2: $S_3/A_3$

$A_3 = \{e, (123), (132)\} \trianglelefteq S_3$.

Cosets: $A_3$ and $(12)A_3 = \{(12), (13), (23)\}$.

```
  Cayley table for S₃/A₃:
  
       ·     │  A₃      (12)A₃
  ───────────┼─────────────────
    A₃       │  A₃      (12)A₃
    (12)A₃   │  (12)A₃   A₃
  
  This is ≅ Z₂!
```

### Example 3: $\mathbb{Z}_6 / \langle 2 \rangle$

$\langle 2 \rangle = \{0, 2, 4\}$. Cosets:

| Coset | Elements |
|-------|----------|
| $0 + \langle 2 \rangle$ | $\{0, 2, 4\}$ |
| $1 + \langle 2 \rangle$ | $\{1, 3, 5\}$ |

$\mathbb{Z}_6/\langle 2 \rangle \cong \mathbb{Z}_2$.

### Example 4: $D_4 / \langle r^2 \rangle$

$\langle r^2 \rangle = \{e, r^2\} \trianglelefteq D_4$.

$|D_4/\langle r^2 \rangle| = 8/2 = 4$.

Cosets: $\{e, r^2\}$, $\{r, r^3\}$, $\{s, r^2s\}$, $\{rs, r^3s\}$.

```
  Cayley table for D₄/⟨r²⟩:
  
  Let E = {e,r²}, R = {r,r³}, S = {s,r²s}, T = {rs,r³s}
  
    ·  │  E   R   S   T
  ─────┼────────────────
   E   │  E   R   S   T
   R   │  R   E   T   S
   S   │  S   T   E   R
   T   │  T   S   R   E
  
  Every non-identity element has order 2 → V₄!
  
  D₄/⟨r²⟩ ≅ V₄ ≅ Z₂ × Z₂
```

---

## 4. The Natural Projection

The map $\pi: G \to G/N$ defined by $\pi(g) = gN$ is called the **natural projection** (or canonical homomorphism).

- $\pi$ is a surjective homomorphism
- $\ker \pi = N$
- $\pi$ is the "simplest" way to map out of $G$ with kernel $N$

```
  Natural projection:
  
  π: G ──→ G/N
     g ├──→ gN
  
  π is surjective: every coset gN = π(g)
  ker π = {g : gN = N} = N
  
  The FIT applied to π gives:
  G/ker(π) = G/N ≅ im(π) = G/N  (trivially true)
```

---

## 5. Quotient Groups and Homomorphisms

**Key Principle:** For every normal subgroup $N \trianglelefteq G$, quotient group $G/N$ arises. Conversely, for every surjective homomorphism $\phi: G \to H$, we get $H \cong G/\ker\phi$.

```
  ╔══════════════════════════════════════════════╗
  ║  NORMAL SUBGROUPS   ←──→   QUOTIENT GROUPS  ║
  ║  N ◁ G              one-to-one    G/N       ║
  ║                  correspondence              ║
  ║                                              ║
  ║  {e}               ←──→         G/{e} ≅ G   ║
  ║  G                 ←──→         G/G ≅ {e}   ║
  ║  N                 ←──→         G/N         ║
  ╚══════════════════════════════════════════════╝
```

---

## 6. When Is $G/N$ Abelian?

**Theorem 3.14.** $G/N$ is abelian if and only if $N$ contains the **commutator subgroup** $G' = [G, G]$.

The **commutator subgroup** $G' = \langle [a,b] : a,b \in G \rangle$ where $[a,b] = aba^{-1}b^{-1}$.

```
  Abelianisation:
  
  G/G' = G^ab = the largest abelian quotient of G
  
  Examples:
  • S₃/A₃ ≅ Z₂  (A₃ = S₃')
  • D₄/D₄' ≅ Z₂ × Z₂  (D₄' = ⟨r²⟩)
  • G abelian ⟹ G' = {e}, so G/{e} ≅ G is abelian
```

---

## 7. Visualising Quotients

### As a "Collapsing" Operation

```
  Z₁₂ with N = ⟨4⟩ = {0, 4, 8}:
  
  BEFORE (Z₁₂):
  0 ─ 1 ─ 2 ─ 3 ─ 4 ─ 5 ─ 6 ─ 7 ─ 8 ─ 9 ─ 10 ─ 11
  
  AFTER (Z₁₂/⟨4⟩):
  Each coset becomes one point:
  
  {0,4,8} ─ {1,5,9} ─ {2,6,10} ─ {3,7,11}
     ●          ●          ●          ●
     0̄          1̄          2̄          3̄
  
  Z₁₂/⟨4⟩ ≅ Z₄
```

---

## 8. Real-World Applications

| Application | Quotient Group |
|-------------|---------------|
| Clock arithmetic | $\mathbb{Z}/12\mathbb{Z} \cong \mathbb{Z}_{12}$ |
| Parity (even/odd) | $\mathbb{Z}/2\mathbb{Z} \cong \mathbb{Z}_2$ |
| Angle addition | $\mathbb{R}/2\pi\mathbb{Z} \cong S^1$ |
| Error detection | Syndromes live in $\mathbb{F}_2^n / C$ |
| Topology | Fundamental group of quotient space |

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| $G/N$ | $= \{gN : g \in G\}$ with $(aN)(bN) = abN$ |
| Requires | $N \trianglelefteq G$ |
| $\|G/N\|$ | $= [G:N] = \|G\|/\|N\|$ |
| Natural projection | $\pi: G \to G/N$, $\pi(g) = gN$, $\ker\pi = N$ |
| $G/N$ abelian | $\iff G' \leq N$ |
| $G/\{e\}$ | $\cong G$ |
| $G/G$ | $\cong \{e\}$ |
| $\mathbb{Z}/n\mathbb{Z}$ | $\cong \mathbb{Z}_n$ |

---

## Quick Revision Questions

1. **Construct** the quotient group $\mathbb{Z}_{12}/\langle 3 \rangle$ and identify it.

2. **Show** that the operation on $G/N$ is well-defined if and only if $N \trianglelefteq G$.

3. **Compute** $D_4/Z(D_4)$ and determine which known group it is isomorphic to.

4. **Prove** that $G/Z(G)$ cyclic implies $G$ is abelian.

5. **Find** the commutator subgroup of $S_3$ and identify $S_3/S_3'$.

6. **If** $|G| = 12$ and $|N| = 4$ with $N \trianglelefteq G$, what can you say about $G/N$?

---

[← Previous: Normal Subgroups](02-normal-subgroups.md) | [Back to README](../README.md) | [Next: Simple Groups →](04-simple-groups.md)
