# Chapter 3.1 — Cosets and Lagrange's Theorem

[← Previous: Second & Third Isomorphism Theorems](../02-Groups-Homomorphisms/05-second-third-isomorphism-theorems.md) | [Back to README](../README.md) | [Next: Normal Subgroups →](02-normal-subgroups.md)

---

## Overview

**Lagrange's Theorem** is one of the most important results in finite group theory. It states that the order of a subgroup always divides the order of the group. The proof relies on the concept of **cosets**, which partition the group into equal-sized pieces.

---

## 1. Left and Right Cosets

**Definition.** Let $H \leq G$ and $g \in G$.

- **Left coset:** $gH = \{gh : h \in H\}$
- **Right coset:** $Hg = \{hg : h \in H\}$

### Example: Cosets of $\langle 3 \rangle$ in $\mathbb{Z}_{12}$

$H = \langle 3 \rangle = \{0, 3, 6, 9\}$

| Coset | Elements |
|-------|----------|
| $0 + H$ | $\{0, 3, 6, 9\}$ |
| $1 + H$ | $\{1, 4, 7, 10\}$ |
| $2 + H$ | $\{2, 5, 8, 11\}$ |
| $3 + H$ | $\{3, 6, 9, 0\} = 0 + H$ |

Only 3 distinct cosets, partitioning $\mathbb{Z}_{12}$.

```
  Coset partition of Z₁₂ by H = ⟨3⟩:
  
  ┌──────────────┬──────────────┬──────────────┐
  │  0 + H       │  1 + H       │  2 + H       │
  │  {0,3,6,9}   │  {1,4,7,10}  │  {2,5,8,11}  │
  └──────────────┴──────────────┴──────────────┘
  
  3 cosets × 4 elements each = 12 = |Z₁₂|  ✓
```

### Example: Cosets of $H = \{e, s\}$ in $S_3$

Left cosets:
| $gH$ | Elements |
|------|----------|
| $eH$ | $\{e, s\} = \{e, (12)\}$ |
| $rH$ | $\{r, rs\} = \{(123), (13)\}$ |
| $r^2H$ | $\{r^2, r^2s\} = \{(132), (23)\}$ |

Right cosets:
| $Hg$ | Elements |
|------|----------|
| $He$ | $\{e, (12)\}$ |
| $Hr$ | $\{r, sr\} = \{(123), (23)\}$ |
| $Hr^2$ | $\{r^2, sr^2\} = \{(132), (13)\}$ |

**Note:** Left cosets $\neq$ right cosets in general! Here $rH \neq Hr$.

---

## 2. Properties of Cosets

**Theorem 3.1.** Let $H \leq G$. For $a, b \in G$:

| # | Property |
|---|----------|
| 1 | $aH = bH \iff a^{-1}b \in H \iff a \in bH$ |
| 2 | $aH = H \iff a \in H$ |
| 3 | $\|aH\| = \|H\|$ (all cosets have the same size) |
| 4 | $aH = bH$ or $aH \cap bH = \emptyset$ (cosets are disjoint or identical) |
| 5 | The left cosets partition $G$ |
| 6 | $aH = bH \iff aH \cap bH \neq \emptyset$ |

```
  Cosets partition the group:
  
  G = g₁H  ⊔  g₂H  ⊔  g₃H  ⊔  ···  ⊔  gₖH
      ──────    ──────    ──────         ──────
       |H|      |H|       |H|            |H|
  
  k cosets × |H| elements = |G|
  
  ⊔ = disjoint union
```

---

## 3. The Index

**Definition.** The **index** of $H$ in $G$, written $[G : H]$, is the number of distinct left cosets of $H$ in $G$.

$$[G : H] = \frac{|G|}{|H|} \quad \text{(for finite } G\text{)}$$

The number of left cosets equals the number of right cosets (even for infinite groups).

---

## 4. Lagrange's Theorem

**Theorem 3.2 (Lagrange).** If $G$ is a finite group and $H \leq G$, then $|H|$ divides $|G|$.

Moreover: $|G| = [G:H] \cdot |H|$.

*Proof.* The left cosets of $H$ partition $G$ into $[G:H]$ disjoint sets, each of size $|H|$. Therefore $|G| = [G:H] \cdot |H|$. $\square$

```
  LAGRANGE'S THEOREM
  ═══════════════════════════════════════
  
  |H| divides |G|     for all  H ≤ G
  
  |G| = [G : H] · |H|
  
  ═══════════════════════════════════════
```

---

## 5. Corollaries of Lagrange's Theorem

### Corollary 3.3: Order of Elements Divides Group Order

If $G$ is finite and $a \in G$, then $|a|$ divides $|G|$.

*Proof.* $|a| = |\langle a \rangle|$, and $\langle a \rangle \leq G$. $\square$

### Corollary 3.4: $a^{|G|} = e$ for All $a \in G$

*Proof.* $|a|$ divides $|G|$, so $a^{|G|} = (a^{|a|})^{|G|/|a|} = e$. $\square$

### Corollary 3.5: Groups of Prime Order Are Cyclic

If $|G| = p$ (prime), then $G \cong \mathbb{Z}_p$.

*Proof.* Any $a \neq e$ has $|a| > 1$ and $|a| \mid p$, so $|a| = p$. Thus $G = \langle a \rangle$. $\square$

### Corollary 3.6: Fermat-Euler Theorem

If $\gcd(a, n) = 1$, then $a^{\varphi(n)} \equiv 1 \pmod{n}$.

*Proof.* $a \in \mathbb{Z}_n^\times$ with $|\mathbb{Z}_n^\times| = \varphi(n)$. Apply Corollary 3.4. $\square$

### Corollary 3.7: Fermat's Little Theorem

If $p$ is prime and $p \nmid a$, then $a^{p-1} \equiv 1 \pmod{p}$.

*Proof.* $\varphi(p) = p - 1$. $\square$

---

## 6. Converse of Lagrange's Theorem

> **The converse of Lagrange's theorem is FALSE in general!**

If $d \mid |G|$, there need not exist a subgroup of order $d$.

**Counterexample:** $A_4$ has order 12, and $6 \mid 12$, but $A_4$ has **no** subgroup of order 6.

```
  Converse of Lagrange:
  
  ┌────────────────────────────────────────────────┐
  │  d | |G|  does NOT imply  ∃ H ≤ G with |H|=d  │
  │                                                 │
  │  Counterexample: A₄, |A₄| = 12                 │
  │  6 | 12 but A₄ has no subgroup of order 6       │
  │                                                 │
  │  HOWEVER: Sylow theorems guarantee subgroups    │
  │  of prime-power order dividing |G|              │
  └────────────────────────────────────────────────┘
```

Groups where the converse holds are called **CLT groups** (Converse Lagrange Theorem groups). All abelian groups and all supersolvable groups are CLT.

---

## 7. The Tower Law for Indices

**Theorem 3.8.** If $K \leq H \leq G$ with $[G:K]$ finite, then:

$$[G : K] = [G : H] \cdot [H : K]$$

```
  Tower Law:
  
     G     ───  [G:K] cosets of K in G
     │
  [G:H]
     │
     H     ───  [H:K] cosets of K in H
     │
  [H:K]
     │
     K
  
  Total cosets: [G:K] = [G:H] · [H:K]
```

---

## 8. Examples and Applications

### Example 1: Possible Orders of Elements in $S_5$

$|S_5| = 120 = 2^3 \cdot 3 \cdot 5$

By Lagrange, element orders must divide 120. The actual orders occurring:

| Cycle type | Order ($= \text{lcm}$ of lengths) | Occurs? |
|-----------|----------------------------------|---------|
| $(1)(1)(1)(1)(1)$ | 1 | Yes |
| $(ab)(1)(1)(1)$ | 2 | Yes |
| $(abc)(1)(1)$ | 3 | Yes |
| $(abcd)(1)$ | 4 | Yes |
| $(abcde)$ | 5 | Yes |
| $(ab)(cd)(1)$ | 2 | Yes |
| $(ab)(cde)$ | 6 | Yes |
| $(abcd)(e)$... | 4 | Yes |

Not every divisor of 120 occurs (e.g., 8, 40, 120 don't occur as element orders).

### Example 2: Application to Number Theory

**Proving Fermat's Little Theorem algebraically:**

$\mathbb{Z}_p^\times = \{1, 2, \ldots, p-1\}$ under multiplication mod $p$.

$|\mathbb{Z}_p^\times| = p - 1$.

For any $a \not\equiv 0$: $a \in \mathbb{Z}_p^\times$, so $a^{p-1} \equiv 1 \pmod{p}$. ∎

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Left coset $gH$ | $= \{gh : h \in H\}$ |
| Cosets partition $G$ | Disjoint, all same size $\|H\|$ |
| Index $[G:H]$ | $= \|G\|/\|H\|$ (finite case) |
| **Lagrange** | $\|H\| \mid \|G\|$ |
| Element order | $\|a\| \mid \|G\|$ |
| $a^{\|G\|} = e$ | For all $a \in G$ (finite) |
| Prime order | $\|G\| = p \implies G \cong \mathbb{Z}_p$ |
| Converse | **FALSE** in general ($A_4$ has no order-6 subgroup) |
| Tower law | $[G:K] = [G:H][H:K]$ |

---

## Quick Revision Questions

1. **List** all distinct left cosets of $\langle 4 \rangle$ in $\mathbb{Z}_{20}$.

2. **Prove** Lagrange's theorem from the coset partition.

3. **Use** Lagrange's theorem to show that any group of order 35 has elements of order 5 and 7.

4. **Prove** Fermat's Little Theorem using Lagrange's theorem.

5. **Show** that $A_4$ has no subgroup of order 6 (the converse of Lagrange fails).

6. **If** $[G:H] = 2$, show that $H \trianglelefteq G$.

---

[← Previous: Second & Third Isomorphism Theorems](../02-Groups-Homomorphisms/05-second-third-isomorphism-theorems.md) | [Back to README](../README.md) | [Next: Normal Subgroups →](02-normal-subgroups.md)
