# Chapter 3.4 — Simple Groups

[← Previous: Quotient Groups](03-quotient-groups.md) | [Back to README](../README.md) | [Next: Direct Products →](05-direct-products.md)

---

## Overview

**Simple groups** are the "atoms" of group theory — groups that cannot be broken down further via quotient constructions. Just as prime numbers are the building blocks of integers, simple groups are the building blocks from which all finite groups are constructed.

---

## 1. Definition

**Definition.** A group $G$ is **simple** if $|G| > 1$ and its only normal subgroups are $\{e\}$ and $G$ itself.

```
  Simple Group:
  ┌──────────────────────────────────┐
  │       G     ← only these two    │
  │       │        normal subgroups  │
  │      {e}                        │
  │                                 │
  │  No "interesting" quotients!    │
  │  G/{e} ≅ G  and  G/G ≅ {e}     │
  │  — both trivial.                │
  └──────────────────────────────────┘
  
  Non-Simple Group:
  ┌──────────────────────────────────┐
  │       G                         │
  │      / \                        │
  │     N   ...   ← N ◁ G, N ≠ {e} │
  │      \                          │
  │      {e}                        │
  │                                 │
  │  Can form G/N — a smaller group │
  └──────────────────────────────────┘
```

---

## 2. Abelian Simple Groups

**Theorem 3.15.** The only abelian simple groups are $\mathbb{Z}_p$ for $p$ prime.

*Proof.* In an abelian group, every subgroup is normal. So $G$ is simple $\iff$ $G$ has no proper, non-trivial subgroups $\iff$ $|G| = p$ (prime), by Lagrange. $\square$

```
  Abelian simple groups:
  
  Z₂, Z₃, Z₅, Z₇, Z₁₁, Z₁₃, Z₁₇, Z₁₉, Z₂₃, ...
  
  (one for each prime p)
```

---

## 3. The Alternating Groups $A_n$

**Theorem 3.16.** $A_n$ is simple for $n \geq 5$.

| $n$ | $\|A_n\|$ | Simple? | Normal subgroups |
|-----|----------|---------|-----------------|
| 1 | 1 | No (trivial) | $\{e\}$ |
| 2 | 1 | No (trivial) | $\{e\}$ |
| 3 | 3 | **Yes** ($= \mathbb{Z}_3$) | $\{e\}, A_3$ |
| 4 | 12 | **No** | $\{e\}, V_4, A_4$ |
| 5 | 60 | **Yes** | $\{e\}, A_5$ |
| $\geq 5$ | $n!/2$ | **Yes** | $\{e\}, A_n$ |

**Why $A_4$ is not simple:** $V_4 = \{e, (12)(34), (13)(24), (14)(23)\} \trianglelefteq A_4$.

**Why $A_5$ IS simple (proof sketch):** The conjugacy classes of $A_5$ have sizes $1, 12, 12, 15, 20$. A normal subgroup must be a union of classes including $\{e\}$. No sub-sum of $\{1, 12, 12, 15, 20\}$ including $1$ divides $60$ — except $1$ and $60$. $\square$

---

## 4. Composition Series

**Definition.** A **composition series** for a group $G$ is a chain:

$$\{e\} = G_0 \trianglelefteq G_1 \trianglelefteq G_2 \trianglelefteq \cdots \trianglelefteq G_n = G$$

where each **factor** $G_{i+1}/G_i$ is simple. These factors are called **composition factors**.

### Example: $S_4$

$$\{e\} \trianglelefteq V_4 \trianglelefteq A_4 \trianglelefteq S_4$$

Composition factors:
- $V_4/\{e\} \cong V_4 \cong \mathbb{Z}_2 \times \mathbb{Z}_2$ — **not simple!**

Refine:
$$\{e\} \trianglelefteq \langle(12)(34)\rangle \trianglelefteq V_4 \trianglelefteq A_4 \trianglelefteq S_4$$

Factors:
| Factor | $\cong$ | Simple? |
|--------|---------|---------|
| $\langle(12)(34)\rangle / \{e\}$ | $\mathbb{Z}_2$ | Yes |
| $V_4 / \langle(12)(34)\rangle$ | $\mathbb{Z}_2$ | Yes |
| $A_4 / V_4$ | $\mathbb{Z}_3$ | Yes |
| $S_4 / A_4$ | $\mathbb{Z}_2$ | Yes |

```
  Composition Series of S₄:
  
  S₄          ─── S₄/A₄ ≅ Z₂
   │
  A₄          ─── A₄/V₄ ≅ Z₃
   │
  V₄          ─── V₄/⟨(12)(34)⟩ ≅ Z₂
   │
  ⟨(12)(34)⟩  ─── Z₂/{e} ≅ Z₂
   │
  {e}
  
  Composition factors: Z₂, Z₂, Z₃, Z₂
```

---

## 5. The Jordan-Hölder Theorem

**Theorem 3.17 (Jordan-Hölder).** Any two composition series of a finite group $G$ have the same length and the same composition factors (up to permutation and isomorphism).

This means: the "prime factorisation" of a group into simple pieces is **unique** (up to order).

```
  Jordan-Hölder Theorem:
  ╔══════════════════════════════════════════╗
  ║  Composition factors are UNIQUE          ║
  ║  (up to ordering and isomorphism)        ║
  ║                                          ║
  ║  Analogy:  12 = 2 × 2 × 3               ║
  ║            12 = 3 × 2 × 2  (same primes)║
  ║                                          ║
  ║  Similarly: S₄ has factors Z₂,Z₂,Z₃,Z₂ ║
  ║  regardless of which series we choose    ║
  ╚══════════════════════════════════════════╝
```

---

## 6. The Classification of Finite Simple Groups

The **Classification Theorem** (completed ~2004, spanning ~15,000 pages) states that every finite simple group is one of:

| Family | Examples |
|--------|---------|
| **Cyclic** $\mathbb{Z}_p$ | $\mathbb{Z}_2, \mathbb{Z}_3, \mathbb{Z}_5, \ldots$ |
| **Alternating** $A_n$ ($n \geq 5$) | $A_5, A_6, A_7, \ldots$ |
| **Groups of Lie type** | $PSL_n(q)$, $PSU_n(q)$, $PSp_{2n}(q)$, ... |
| **26 Sporadic groups** | Mathieu groups, Monster, Baby Monster, ... |

```
  CLASSIFICATION OF FINITE SIMPLE GROUPS
  
  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐
  │  Cyclic Z_p  │  │ Alternating  │  │ Lie type     │
  │  (infinite   │  │  A_n (n≥5)   │  │ 16 families  │
  │   family)    │  │  (infinite   │  │ (infinite    │
  │              │  │   family)    │  │  families)   │
  └──────────────┘  └──────────────┘  └──────────────┘
  
  ┌───────────────────────────────────────────────┐
  │  26 SPORADIC GROUPS                            │
  │                                                │
  │  Mathieu: M₁₁, M₁₂, M₂₂, M₂₃, M₂₄          │
  │  Conway: Co₁, Co₂, Co₃                        │
  │  Fischer: Fi₂₂, Fi₂₃, Fi₂₄'                  │
  │  Janko: J₁, J₂, J₃, J₄                       │
  │  Monster: M  (|M| ≈ 8 × 10⁵³)               │
  │  Baby Monster: B                               │
  │  ... and others                                │
  └───────────────────────────────────────────────┘
```

The **Monster group** has order:
$$|M| = 2^{46} \cdot 3^{20} \cdot 5^9 \cdot 7^6 \cdot 11^2 \cdot 13^3 \cdot 17 \cdot 19 \cdot 23 \cdot 29 \cdot 31 \cdot 41 \cdot 47 \cdot 59 \cdot 71$$
$$\approx 8.08 \times 10^{53}$$

---

## 7. Solvable Groups

**Definition.** A group $G$ is **solvable** if it has a composition series with all factors abelian (i.e., all composition factors are $\mathbb{Z}_p$ for various primes $p$).

| Group | Solvable? | Why |
|-------|-----------|-----|
| $S_3$ | Yes | Factors: $\mathbb{Z}_2, \mathbb{Z}_3$ |
| $S_4$ | Yes | Factors: $\mathbb{Z}_2, \mathbb{Z}_2, \mathbb{Z}_3, \mathbb{Z}_2$ |
| $A_4$ | Yes | Factors: $\mathbb{Z}_2, \mathbb{Z}_2, \mathbb{Z}_3$ |
| $S_5$ | **No** | $A_5$ is simple, non-abelian |
| $A_5$ | **No** | Simple, non-abelian |

**Historical importance:** Galois proved that a polynomial is solvable by radicals if and only if its Galois group is solvable. Since $S_5$ is not solvable, the general quintic cannot be solved by radicals!

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Simple group | Only normal subgroups are $\{e\}$ and $G$ |
| Abelian simple | $\cong \mathbb{Z}_p$ ($p$ prime) |
| $A_n$ simple | for $n \geq 5$ |
| Composition series | Chain with simple factors |
| Jordan-Hölder | Composition factors are unique |
| Classification | $\mathbb{Z}_p$, $A_n$, Lie type, 26 sporadics |
| Solvable | All composition factors abelian |
| $S_n$ solvable | iff $n \leq 4$ |

---

## Quick Revision Questions

1. **Prove** that $\mathbb{Z}_p$ is simple for every prime $p$.

2. **Find** a composition series for $\mathbb{Z}_{12}$ and list the composition factors.

3. **Explain** why $A_4$ is not simple (find a proper normal subgroup).

4. **State** the Jordan-Hölder theorem and explain its analogy with prime factorisation.

5. **Why** is $S_5$ not solvable? What does this imply about solving quintic equations?

6. **Give** an example of a non-abelian simple group with fewer than 100 elements.

---

[← Previous: Quotient Groups](03-quotient-groups.md) | [Back to README](../README.md) | [Next: Direct Products →](05-direct-products.md)
