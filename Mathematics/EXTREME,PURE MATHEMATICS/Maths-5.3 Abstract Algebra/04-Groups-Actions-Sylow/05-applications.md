# Chapter 4.5 — Applications of Group Actions & Sylow Theorems

[← Previous: Sylow Theorems](04-sylow-theorems.md) | [Back to README](../README.md) | [Next: Rings — Definition & Examples →](../05-Rings-Basics/01-definition-and-examples.md)

---

## Overview

This chapter collects the most important applications of group actions and the Sylow theorems: classifying small groups, proving simplicity results, Burnside's transfer theorem, and connecting to combinatorics via Pólya enumeration.

---

## 1. Classifying Groups of Small Order

### Strategy

```
  Classification algorithm for groups of order n:
  
  1. Factor n = p₁^a₁ · p₂^a₂ · ···
  2. Apply Sylow to find nₚ for each prime
  3. Determine which Sylow subgroups are normal
  4. Build G from Sylow subgroups via:
     - Direct products (if all normal, trivial action)
     - Semidirect products (if some normal)
  5. Check for isomorphisms between candidates
```

### Groups of Order ≤ 15

| $n$ | \# groups | Groups |
|-----|----------|--------|
| 1 | 1 | $\{e\}$ |
| 2 | 1 | $\mathbb{Z}_2$ |
| 3 | 1 | $\mathbb{Z}_3$ |
| 4 | 2 | $\mathbb{Z}_4$, $V_4$ |
| 5 | 1 | $\mathbb{Z}_5$ |
| 6 | 2 | $\mathbb{Z}_6$, $S_3$ |
| 7 | 1 | $\mathbb{Z}_7$ |
| 8 | 5 | $\mathbb{Z}_8$, $\mathbb{Z}_4 \times \mathbb{Z}_2$, $\mathbb{Z}_2^3$, $D_4$, $Q_8$ |
| 9 | 2 | $\mathbb{Z}_9$, $\mathbb{Z}_3^2$ |
| 10 | 2 | $\mathbb{Z}_{10}$, $D_5$ |
| 11 | 1 | $\mathbb{Z}_{11}$ |
| 12 | 5 | $\mathbb{Z}_{12}$, $\mathbb{Z}_2 \times \mathbb{Z}_6$, $D_6$, $A_4$, $\text{Dic}_{12}$ |
| 13 | 1 | $\mathbb{Z}_{13}$ |
| 14 | 2 | $\mathbb{Z}_{14}$, $D_7$ |
| 15 | 1 | $\mathbb{Z}_{15}$ |

---

## 2. Proving Groups Are Not Simple

A group $G$ is **not simple** if it has a proper, non-trivial normal subgroup. The Sylow theorems are the main tool.

### Method

If $n_p = 1$ for any prime $p$ dividing $|G|$, then the unique Sylow $p$-subgroup is normal, so $G$ is not simple (provided $|G| \neq p^n$).

### 2.1. No Group of Order 12 Is Simple

$|G| = 12 = 2^2 \cdot 3$.

$n_3 \in \{1, 4\}$. If $n_3 = 1$, done.

If $n_3 = 4$: $G$ acts on $\text{Syl}_3(G)$ by conjugation, giving $\varphi: G \to S_4$. Since $n_3 > 1$, the action is non-trivial, so $\ker\varphi \neq G$.

$|\text{im}\,\varphi|$ divides both $|G| = 12$ and $|S_4| = 24$. If $\ker\varphi = \{e\}$, $G$ embeds in $S_4$, and $|G| = 12 = |A_4|$, so $G \cong A_4$ (the unique index-2 subgroup of $S_4$ of order 12). In $A_4$: $n_2 = 1$ (the Klein four subgroup $V_4 \trianglelefteq A_4$). So $G$ is not simple.

### 2.2. No Group of Order 56 Is Simple

$|G| = 56 = 2^3 \cdot 7$.

$n_7 \mid 8$, $n_7 \equiv 1 \pmod{7}$: $n_7 \in \{1, 8\}$.

If $n_7 = 1$, done.

If $n_7 = 8$: eight Sylow 7-subgroups, each of order 7, pairwise intersecting in $\{e\}$ (since they're of prime order). This gives $8 \times 6 = 48$ elements of order 7.

Remaining elements: $56 - 48 = 8$. These must form the **unique** Sylow 2-subgroup. So $n_2 = 1$, and $G$ is not simple. ✓

```
  Order 56 element count:
  
  48 elements of order 7  (from 8 Sylow 7-subgroups)
  +  8 remaining elements (form unique Sylow 2-subgroup)
  ────────────────────────
  56 total                ⟹ n₂ = 1 ⟹ not simple
```

---

## 3. Simplicity of $A_5$

**Theorem 4.17.** $A_5$ is simple.

$|A_5| = 60 = 2^2 \cdot 3 \cdot 5$.

*Proof.* Suppose $N \trianglelefteq A_5$ with $N \neq \{e\}, A_5$. Then $|N|$ divides 60.

Conjugacy classes in $A_5$:

| Class | Cycle type | Size |
|-------|-----------|------|
| $\{e\}$ | $(1^5)$ | 1 |
| 3-cycles | $(3, 1^2)$ | 20 |
| prod. of 2-transpositions | $(2^2, 1)$ | 15 |
| 5-cycles (type I) | $(5)$ | 12 |
| 5-cycles (type II) | $(5)$ | 12 |

$N$ must be a union of conjugacy classes containing $\{e\}$, with $|N|$ dividing 60.

Possible sums $1 + \text{subset of } \{20, 15, 12, 12\}$ that divide 60:

$1+15 = 16$, $1+20 = 21$, $1+12 = 13$, $1+12+12 = 25$, $1+15+12=28$... none divide 60.

No valid $|N|$ exists, so $A_5$ is simple. $\blacksquare$

---

## 4. The $p$-Group Fixed-Point Theorem

**Theorem 4.18.** If $P$ is a $p$-group acting on a finite set $X$, then:

$$|X^P| \equiv |X| \pmod{p}$$

where $X^P = \{x \in X : g \cdot x = x \text{ for all } g \in P\}$.

*Proof.* Each orbit has size dividing $|P| = p^k$, so size $1$ or divisible by $p$. The size-1 orbits are exactly the fixed points. $\blacksquare$

### Application: Wilson's Theorem

Let $p$ be prime. $(p-1)! \equiv -1 \pmod{p}$.

### Application: Counting Fixed Points

If $\mathbb{Z}_p$ acts on a set of size not divisible by $p$, there must be fixed points.

---

## 5. Pólya Enumeration Theorem

A generalisation of Burnside's lemma using **cycle index** polynomials.

**Theorem 4.19 (Pólya).** The number of distinct colourings with $k$ colours under a group $G$ acting on $n$ positions is:

$$P_G(k) = \frac{1}{|G|} \sum_{g \in G} k^{c(g)}$$

where $c(g)$ is the number of cycles of $g$ (as a permutation of positions).

### Example: Colourings of a Square

$G = D_4$ acting on 4 vertices, $k = 2$ colours (B, W):

| $g$ | Permutation | Cycles $c(g)$ | $2^{c(g)}$ |
|-----|------------|---------------|------------|
| $e$ | $(1)(2)(3)(4)$ | 4 | 16 |
| $r$ | $(1234)$ | 1 | 2 |
| $r^2$ | $(13)(24)$ | 2 | 4 |
| $r^3$ | $(1432)$ | 1 | 2 |
| $s$ | $(24)(1)(3)$ | 3 | 8 |
| $sr$ | $(12)(34)$ | 2 | 4 |
| $sr^2$ | $(13)(2)(4)$ | 3 | 8 |
| $sr^3$ | $(14)(23)$ | 2 | 4 |

$$P_{D_4}(2) = \frac{1}{8}(16 + 2 + 4 + 2 + 8 + 4 + 8 + 4) = \frac{48}{8} = 6$$

Six distinct 2-colourings of a square (up to $D_4$ symmetry).

---

## 6. Finite Simple Groups (Overview)

The classification of finite simple groups is one of the greatest achievements of 20th-century mathematics:

```
  Finite Simple Groups:
  
  1. Cyclic: Z_p  (p prime)
  2. Alternating: A_n  (n ≥ 5)
  3. Groups of Lie type: PSL_n(q), PSp_{2n}(q), ...
  4. 26 Sporadic groups:
     - Mathieu groups: M₁₁, M₁₂, M₂₂, M₂₃, M₂₄
     - Janko groups: J₁, J₂, J₃, J₄
     - Monster: order ≈ 8 × 10⁵³
     - ... and 17 others
  
  Total proof: ~10,000 pages across hundreds of journal articles
```

---

## 7. Applications Beyond Pure Algebra

| Application | Group Action / Sylow |
|------------|---------------------|
| Crystallography | 32 point groups (3D), 230 space groups |
| Molecular symmetry | Point groups classify molecular shapes |
| Combinatorics | Burnside/Pólya: counting under symmetry |
| Coding theory | Automorphism groups of codes |
| Physics | Particle physics: gauge groups $SU(2), SU(3)$ |
| Rubik's cube | Group of order $\approx 4.3 \times 10^{19}$ |

---

## 8. Worked Example: Groups of Order 20

$|G| = 20 = 2^2 \cdot 5$.

$n_5 \mid 4$, $n_5 \equiv 1 \pmod{5}$: $n_5 = 1$ only.

So the Sylow 5-subgroup $P \cong \mathbb{Z}_5$ is **normal**.

$n_2 \mid 5$, $n_2 \equiv 1 \pmod{2}$: $n_2 \in \{1, 5\}$.

A Sylow 2-subgroup $Q$ has order 4: $Q \cong \mathbb{Z}_4$ or $\mathbb{Z}_2^2$.

Since $P \trianglelefteq G$, $P \cap Q = \{e\}$, $PQ = G$ (by order count), $G \cong P \rtimes Q$.

The semidirect product depends on $\varphi: Q \to \text{Aut}(P) \cong \text{Aut}(\mathbb{Z}_5) \cong \mathbb{Z}_4$.

| $Q$ | $\varphi$ | Group | Description |
|-----|-----------|-------|-------------|
| $\mathbb{Z}_4$ | trivial | $\mathbb{Z}_{20}$ | cyclic |
| $\mathbb{Z}_4$ | non-trivial | $\mathbb{Z}_5 \rtimes \mathbb{Z}_4$ | Frobenius group $F_{20}$ |
| $\mathbb{Z}_2^2$ | trivial | $\mathbb{Z}_{10} \times \mathbb{Z}_2$ | abelian |
| $\mathbb{Z}_2^2$ | one factor acts | $D_{10}$ | dihedral |
| $\mathbb{Z}_2^2$ | — | $\text{Dic}_{20}$ | generalised dicyclic |

There are **5 groups of order 20** (up to isomorphism).

---

## Summary Table

| Application | Key Technique |
|------------|---------------|
| Classify order $n$ | Factor $n$, apply Sylow, build via (semi)direct products |
| Prove not simple | Find $n_p = 1$ for some $p$, or element counting |
| Prove simple ($A_5$) | Check no union of conjugacy classes has valid size |
| Pólya enumeration | $P_G(k) = \frac{1}{\|G\|}\sum k^{c(g)}$ |
| $p$-group action | $\|X^P\| \equiv \|X\| \pmod{p}$ |

---

## Quick Revision Questions

1. **Classify** all groups of order 21 using Sylow theorems.

2. **Prove** that $A_5$ is the smallest non-abelian simple group.

3. **Use** element counting to show that no group of order 30 is simple.

4. **Apply** Pólya enumeration to count distinct necklaces of 6 beads with 3 colours (rotational symmetry only).

5. **List** all groups of order 8 and identify which are non-isomorphic.

6. **Explain** why every group of prime order is simple, and give an example of a non-abelian simple group.

---

[← Previous: Sylow Theorems](04-sylow-theorems.md) | [Back to README](../README.md) | [Next: Rings — Definition & Examples →](../05-Rings-Basics/01-definition-and-examples.md)
