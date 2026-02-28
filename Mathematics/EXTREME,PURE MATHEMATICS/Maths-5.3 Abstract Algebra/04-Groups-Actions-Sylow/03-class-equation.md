# Chapter 4.3 — The Class Equation

[← Previous: Orbit-Stabiliser Theorem](02-orbit-stabilizer-theorem.md) | [Back to README](../README.md) | [Next: Sylow Theorems →](04-sylow-theorems.md)

---

## Overview

The **class equation** applies the orbit-stabiliser theorem to the conjugation action of $G$ on itself. It relates $|G|$, the centre $|Z(G)|$, and the sizes of non-trivial conjugacy classes. This single formula proves that $p$-groups have non-trivial centres, which is the foundation for the Sylow theorems.

---

## 1. Conjugacy Classes

When $G$ acts on itself by **conjugation** ($g \cdot x = gxg^{-1}$):

- **Orbits** = conjugacy classes: $\text{cl}(a) = \{gag^{-1} : g \in G\}$
- **Stabiliser** of $a$ = centraliser: $C_G(a) = \{g \in G : ga = ag\}$

By orbit-stabiliser:

$$|\text{cl}(a)| = [G : C_G(a)] = \frac{|G|}{|C_G(a)|}$$

---

## 2. The Class Equation

**Theorem 4.7 (Class Equation).** Let $G$ be a finite group. Then:

$$|G| = |Z(G)| + \sum_{i=1}^{k} [G : C_G(a_i)]$$

where $a_1, \ldots, a_k$ are representatives of the conjugacy classes of size $> 1$.

```
  G partitioned by conjugacy classes:
  
  |G| = |{e}| + |cl(a₁)| + |cl(a₂)| + ··· + |cl(aₖ)| + ··· 
        \_____________/   \________________________________/
         size-1 classes     non-trivial conjugacy classes
         = Z(G)             each has size | |G|, size > 1
  
  ┌────────────────────────────────────────────────────┐
  │  |G| = |Z(G)| + Σ [G : C_G(aᵢ)]                  │
  │                                                    │
  │  where each [G : C_G(aᵢ)] > 1 and divides |G|     │
  └────────────────────────────────────────────────────┘
```

*Proof.* The conjugacy classes partition $G$. An element $a$ is in a singleton class iff $gag^{-1} = a$ for all $g$, iff $a \in Z(G)$. The remaining classes have size $> 1$. $\blacksquare$

---

## 3. Example: Class Equation of $S_3$

$|S_3| = 6$, $Z(S_3) = \{e\}$.

| Element | Conjugacy class | Size | $\|C_{S_3}(a)\|$ |
|---------|----------------|------|------------------|
| $e$ | $\{e\}$ | 1 | 6 |
| $(12)$ | $\{(12), (13), (23)\}$ | 3 | 2 |
| $(123)$ | $\{(123), (132)\}$ | 2 | 3 |

$$6 = 1 + 3 + 2$$

or equivalently:

$$6 = |Z(S_3)| + [S_3 : C(12)] + [S_3 : C(123)] = 1 + 3 + 2$$

---

## 4. Example: Class Equation of $D_4$

$|D_4| = 8$, $D_4 = \{e, r, r^2, r^3, s, sr, sr^2, sr^3\}$.

| Class | Elements | Size |
|-------|----------|------|
| $\{e\}$ | $e$ | 1 |
| $\{r^2\}$ | $r^2$ | 1 |
| $\{r, r^3\}$ | $r, r^3$ | 2 |
| $\{s, sr^2\}$ | $s, sr^2$ | 2 |
| $\{sr, sr^3\}$ | $sr, sr^3$ | 2 |

$Z(D_4) = \{e, r^2\}$, $|Z(D_4)| = 2$.

$$8 = 2 + 2 + 2 + 2$$

Check: each non-trivial class size (2) divides 8. ✓

---

## 5. $p$-Groups Have Non-Trivial Centre

**Definition.** A **$p$-group** is a group whose order is a power of a prime $p$: $|G| = p^n$ for some $n \geq 1$.

**Theorem 4.8.** If $G$ is a $p$-group with $|G| > 1$, then $Z(G) \neq \{e\}$.

*Proof.* From the class equation:

$$|G| = |Z(G)| + \sum [G : C_G(a_i)]$$

Each $[G : C_G(a_i)] > 1$ divides $|G| = p^n$, so each summand is divisible by $p$.

Since $|G|$ is divisible by $p$, we get $|Z(G)| \equiv 0 \pmod{p}$.

Since $e \in Z(G)$, we have $|Z(G)| \geq 1$. Combined with $p \mid |Z(G)|$, we get $|Z(G)| \geq p > 1$. $\blacksquare$

```
  Class equation mod p:
  
  |G|    ≡ |Z(G)| + Σ [G : C(aᵢ)]   (mod p)
   0     ≡ |Z(G)| +       0           (mod p)
  
  ⟹ p | |Z(G)|  ⟹  |Z(G)| ≥ p  ⟹  Z(G) ≠ {e}
```

---

## 6. Consequences

### 6.1. Groups of Order $p^2$ Are Abelian

**Corollary 4.9.** Every group of order $p^2$ is abelian. In fact, $G \cong \mathbb{Z}_{p^2}$ or $G \cong \mathbb{Z}_p \times \mathbb{Z}_p$.

*Proof.* By Theorem 4.8, $|Z(G)| \geq p$. So $|Z(G)| = p$ or $p^2$.

If $|Z(G)| = p^2$, then $G = Z(G)$, so $G$ is abelian.

If $|Z(G)| = p$, then $G/Z(G)$ has order $p$, hence is cyclic: $G/Z(G) = \langle \bar{g} \rangle$. But then $G$ is abelian (any two elements commute via elements of $Z(G)$ and powers of $g$), contradicting $|Z(G)| = p < p^2$. So this case is impossible.

Thus $G$ is abelian, and by FTFGAG: $G \cong \mathbb{Z}_{p^2}$ or $\mathbb{Z}_p \times \mathbb{Z}_p$. $\blacksquare$

### 6.2. Cauchy's Theorem

**Theorem 4.10 (Cauchy).** If $p$ is a prime dividing $|G|$, then $G$ has an element of order $p$.

*Proof sketch (via class equation for $p$-subgroups, or McKay's combinatorial proof).* Consider the set:

$$S = \{(a_1, \ldots, a_p) \in G^p : a_1 a_2 \cdots a_p = e\}$$

$|S| = |G|^{p-1}$ (choose $a_1, \ldots, a_{p-1}$ freely, then $a_p$ is determined).

$\mathbb{Z}_p$ acts on $S$ by cyclic permutation. Each orbit has size 1 or $p$.

Size-1 orbits: $(a, a, \ldots, a)$ with $a^p = e$.

$|S| = |G|^{p-1} \equiv 0 \pmod{p}$, and the orbit $(e, e, \ldots, e)$ contributes 1.

So the number of size-1 orbits $\equiv 0 \pmod{p}$, meaning there exists $a \neq e$ with $a^p = e$. $\blacksquare$

---

## 7. Conjugacy in $S_n$

**Theorem 4.11.** Two permutations in $S_n$ are conjugate iff they have the same **cycle type**.

The number of conjugacy classes of $S_n$ = number of partitions of $n$.

| $n$ | Partitions | \# Conjugacy classes |
|-----|-----------|---------------------|
| 1 | $1$ | 1 |
| 2 | $2,\ 1+1$ | 2 |
| 3 | $3,\ 2+1,\ 1+1+1$ | 3 |
| 4 | $4,\ 3+1,\ 2+2,\ 2+1+1,\ 1+1+1+1$ | 5 |
| 5 | 7 partitions | 7 |

### Size of Conjugacy Classes in $S_n$

For cycle type $(1^{c_1} 2^{c_2} \cdots n^{c_n})$:

$$|\text{cl}| = \frac{n!}{\prod_{k=1}^{n} k^{c_k} \cdot c_k!}$$

**Example.** Cycle type $(2, 2, 1)$ in $S_5$:

$$|\text{cl}| = \frac{5!}{2^2 \cdot 2! \cdot 1^1 \cdot 1!} = \frac{120}{4 \cdot 2 \cdot 1} = 15$$

---

## 8. Class Equation of $A_4$

$|A_4| = 12$, $Z(A_4) = \{e\}$.

| Cycle type | Elements | Class size | $\|C_{A_4}(a)\|$ |
|-----------|----------|-----------|------------------|
| $(1,1,1,1)$ | $e$ | 1 | 12 |
| $(2,2)$ | $(12)(34), (13)(24), (14)(23)$ | 3 | 4 |
| $(3)_+$ | $(123), (134), (142), (243)$ | 4 | 3 |
| $(3)_-$ | $(132), (143), (124), (234)$ | 4 | 3 |

$$12 = 1 + 3 + 4 + 4$$

Note: 3-cycles split into two conjugacy classes in $A_4$ (but form one class in $S_4$).

**Key observation:** There is no subgroup of order 6 in $A_4$ (since the class equation would require a union of classes including $\{e\}$ summing to 6, but $1 + 3 = 4 \neq 6$, $1 + 4 = 5 \neq 6$). This shows the **converse of Lagrange's theorem fails**.

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Class equation | $\|G\| = \|Z(G)\| + \sum [G : C_G(a_i)]$ |
| $p$-group centre | $\|G\| = p^n > 1 \implies Z(G) \neq \{e\}$ |
| Order $p^2$ | Always abelian: $\mathbb{Z}_{p^2}$ or $\mathbb{Z}_p^2$ |
| Cauchy's theorem | $p \mid \|G\| \implies \exists a$ of order $p$ |
| Conjugacy in $S_n$ | Same cycle type $\iff$ conjugate |
| \# classes in $S_n$ | = number of partitions of $n$ |

---

## Quick Revision Questions

1. **Write** the class equation for $Q_8 = \{\pm 1, \pm i, \pm j, \pm k\}$.

2. **Prove** that every group of order $p^2$ is abelian using the class equation.

3. **Compute** the number of conjugacy classes of $S_5$ and list the cycle types.

4. **Find** $|Z(G)|$ if $G$ has order 27 and exactly 5 conjugacy classes.

5. **Use** Cauchy's theorem to prove that a group of order 15 has elements of order 3 and 5.

6. **Explain** why $A_4$ has no subgroup of order 6, using the class equation.

---

[← Previous: Orbit-Stabiliser Theorem](02-orbit-stabilizer-theorem.md) | [Back to README](../README.md) | [Next: Sylow Theorems →](04-sylow-theorems.md)
