# Chapter 3.2 — Normal Subgroups

[← Previous: Cosets and Lagrange's Theorem](01-cosets-and-lagranges-theorem.md) | [Back to README](../README.md) | [Next: Quotient Groups →](03-quotient-groups.md)

---

## Overview

**Normal subgroups** are the subgroups for which left cosets equal right cosets. They are precisely the kernels of homomorphisms and the subgroups from which we can form quotient groups. Normality is the key that unlocks the entire quotient group construction.

---

## 1. Definition

**Definition.** A subgroup $N$ of $G$ is **normal**, written $N \trianglelefteq G$, if for all $g \in G$:

$$gN = Ng \quad (\text{left coset = right coset})$$

Equivalently: $gNg^{-1} = N$ for all $g \in G$.

Equivalently: $gng^{-1} \in N$ for all $g \in G$, $n \in N$.

> Note: $gN = Ng$ does **not** mean $gn = ng$ for each $n$. It means the **sets** are equal.

---

## 2. Equivalent Conditions for Normality

**Theorem 3.9.** The following are equivalent for $N \leq G$:

| # | Condition |
|---|-----------|
| (a) | $N \trianglelefteq G$ (every left coset is a right coset) |
| (b) | $gNg^{-1} = N$ for all $g \in G$ |
| (c) | $gNg^{-1} \subseteq N$ for all $g \in G$ |
| (d) | $N = \ker\phi$ for some homomorphism $\phi: G \to H$ |
| (e) | The coset multiplication $(aN)(bN) = (ab)N$ is well-defined |

```
  Normal ⟺ invariant under conjugation:
  
  For all g ∈ G:
  
       g · N · g⁻¹  =  N
       ─────────────    ─
       conjugate of N   same N back!
  
  ┌──────────────────────────────────────┐
  │  N ◁ G means N is "the same from    │
  │  every element's perspective"        │
  └──────────────────────────────────────┘
```

---

## 3. Examples of Normal Subgroups

### Always Normal

| Subgroup | Why Normal |
|----------|-----------|
| $\{e\}$ | $g e g^{-1} = e$ |
| $G$ itself | $gGg^{-1} = G$ |
| $Z(G)$ (centre) | $gzg^{-1} = z$ for $z \in Z(G)$ |
| Any subgroup of index 2 | Theorem below |
| Kernel of any homomorphism | Proved earlier |

### Specific Examples

| Group | Normal Subgroup | Reason |
|-------|----------------|--------|
| $S_n$ | $A_n$ | $[S_n : A_n] = 2$ |
| $GL_n(\mathbb{R})$ | $SL_n(\mathbb{R})$ | $\ker(\det)$ |
| $D_n$ | $\langle r \rangle$ | $[D_n : \langle r \rangle] = 2$ |
| $S_4$ | $V_4 = \{e, (12)(34), (13)(24), (14)(23)\}$ | Verify conjugation |
| Every abelian group | All subgroups | $gng^{-1} = n$ always |

### Non-Normal Example

In $S_3$: $H = \langle (12) \rangle = \{e, (12)\}$.

$(123)(12)(123)^{-1} = (123)(12)(132) = (23) \notin H$.

So $H$ is **not** normal in $S_3$.

---

## 4. Index-2 Subgroups Are Always Normal

**Theorem 3.10.** If $[G : N] = 2$, then $N \trianglelefteq G$.

*Proof.* There are exactly 2 left cosets: $N$ and $gN$ (for $g \notin N$). Similarly 2 right cosets: $N$ and $Ng$. Since $gN \neq N$ and $Ng \neq N$, and the cosets partition $G$ into two sets, we must have $gN = Ng$. $\square$

```
  Index 2 ⟹ Normal:
  
  G = N ⊔ gN        (left cosets)
  G = N ⊔ Ng        (right cosets)
  
  Since gN ≠ N and Ng ≠ N:
  gN = G \ N = Ng
  
  ∴ gN = Ng for all g  ⟹  N ◁ G
```

---

## 5. The Normaliser

**Definition.** The **normaliser** of $H$ in $G$:

$$N_G(H) = \{g \in G : gHg^{-1} = H\}$$

Then:
- $H \leq N_G(H) \leq G$
- $H \trianglelefteq N_G(H)$ (always!)
- $H \trianglelefteq G \iff N_G(H) = G$

```
  Normaliser measures "how normal" H is:
  
  ┌───────────────────────────────────┐
  │  H ⊆ N_G(H) ⊆ G                 │
  │                                    │
  │  N_G(H) = G   ⟺  H ◁ G          │
  │  N_G(H) = H   ⟺  H is "as far   │
  │                    from normal     │
  │                    as possible"    │
  └───────────────────────────────────┘
```

---

## 6. Conjugacy and Normal Subgroups

**Definition.** Elements $a, b \in G$ are **conjugate** if $b = gag^{-1}$ for some $g \in G$. The **conjugacy class** of $a$ is:

$$\text{cl}(a) = \{gag^{-1} : g \in G\}$$

**Theorem 3.11.** $N \trianglelefteq G$ if and only if $N$ is a union of conjugacy classes.

*Proof.* $N \trianglelefteq G \iff gNg^{-1} = N$ for all $g \iff$ if $n \in N$ then $gng^{-1} \in N$ for all $g \iff$ the entire conjugacy class of each $n \in N$ lies in $N$. $\square$

### Example: Conjugacy Classes of $S_3$

| Class | Elements | Size |
|-------|----------|------|
| $\{e\}$ | Identity | 1 |
| $\{(12), (13), (23)\}$ | Transpositions | 3 |
| $\{(123), (132)\}$ | 3-cycles | 2 |

Normal subgroups must be unions of classes:
- $\{e\}$: sizes $1 = 1$ ✓ ($\trianglelefteq S_3$)
- $\{e, (123), (132)\}$: sizes $1 + 2 = 3$ divides $6$ ✓ ($= A_3 \trianglelefteq S_3$)
- $\{e, (12)\}$: sizes $1 + 1 = 2$, but $\{(12)\}$ is not a conjugacy class! ✗
- $S_3$: sizes $1 + 3 + 2 = 6$ ✓

So the normal subgroups of $S_3$ are: $\{e\}$, $A_3$, $S_3$.

---

## 7. Products of Normal Subgroups

**Theorem 3.12.** If $M, N \trianglelefteq G$, then:
- $M \cap N \trianglelefteq G$
- $MN \trianglelefteq G$

**Theorem 3.13.** If $M, N \trianglelefteq G$ with $M \cap N = \{e\}$, then elements of $M$ and $N$ commute: $mn = nm$ for all $m \in M$, $n \in N$.

*Proof.* Consider $mnm^{-1}n^{-1}$. Since $N \trianglelefteq G$: $mnm^{-1} \in N$, so $mnm^{-1}n^{-1} \in N$. Since $M \trianglelefteq G$: $nm^{-1}n^{-1} \in M$, so $mnm^{-1}n^{-1} \in M$. Thus $mnm^{-1}n^{-1} \in M \cap N = \{e\}$, giving $mn = nm$. $\square$

---

## 8. Real-World Applications

| Application | Normal Subgroup |
|-------------|----------------|
| Even/odd permutations | $A_n \trianglelefteq S_n$ |
| Special matrices | $SL_n \trianglelefteq GL_n$ |
| Modular arithmetic | $n\mathbb{Z} \trianglelefteq \mathbb{Z}$ |
| Symmetry breaking (physics) | Residual symmetry is normal |
| Error-correcting codes | Code $C$ is normal in ambient group |

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| $N \trianglelefteq G$ | $gNg^{-1} = N$ for all $g$ |
| Equivalent | $gN = Ng$ for all $g$ |
| Equivalent | $N = \ker\phi$ for some $\phi$ |
| Index 2 | $\implies$ normal |
| Abelian groups | All subgroups normal |
| Normal = union of conjugacy classes | |
| $N_G(H)$ | $= \{g : gHg^{-1} = H\}$ |
| $H \trianglelefteq G$ | $\iff N_G(H) = G$ |
| $M, N \trianglelefteq G$ | $\implies M \cap N, MN \trianglelefteq G$ |

---

## Quick Revision Questions

1. **Prove** that every subgroup of an abelian group is normal.

2. **Show** that $\langle (12) \rangle$ is NOT normal in $S_3$ by finding a conjugate not in the subgroup.

3. **List** all normal subgroups of $D_4$.

4. **Prove** that if $[G:N] = 2$, then $N \trianglelefteq G$.

5. **Find** the normal subgroups of $S_4$ using conjugacy classes.

6. **If** $M, N \trianglelefteq G$ with $M \cap N = \{e\}$, prove that $mn = nm$ for all $m \in M$, $n \in N$.

---

[← Previous: Cosets and Lagrange's Theorem](01-cosets-and-lagranges-theorem.md) | [Back to README](../README.md) | [Next: Quotient Groups →](03-quotient-groups.md)
