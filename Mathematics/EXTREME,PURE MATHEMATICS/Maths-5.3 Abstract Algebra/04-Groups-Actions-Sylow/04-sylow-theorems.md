# Chapter 4.4 — Sylow Theorems

[← Previous: Class Equation](03-class-equation.md) | [Back to README](../README.md) | [Next: Applications →](05-applications.md)

---

## Overview

The **Sylow theorems** are the most powerful tools in finite group theory. They guarantee the existence of subgroups of prime-power order, constrain their number, and show they are all conjugate. Combined with the class equation, they enable classification of groups of small order.

---

## 1. Definitions

Let $G$ be a finite group and $p$ a prime.

**Definition.** If $|G| = p^n m$ where $\gcd(p, m) = 1$:

- A **Sylow $p$-subgroup** of $G$ is a subgroup of order $p^n$ (the maximum $p$-power dividing $|G|$).
- $\text{Syl}_p(G)$ = set of all Sylow $p$-subgroups of $G$.
- $n_p = |\text{Syl}_p(G)|$ = number of Sylow $p$-subgroups.

```
  |G| = p^n · m,   gcd(p,m) = 1
  
  Sylow p-subgroup P:  |P| = p^n  (maximal p-power)
  
  Example: |G| = 60 = 2² · 3 · 5
  
  Sylow 2-subgroup: order 4   (n₂ = ?)
  Sylow 3-subgroup: order 3   (n₃ = ?)
  Sylow 5-subgroup: order 5   (n₅ = ?)
```

---

## 2. The Three Sylow Theorems

### Sylow's First Theorem (Existence)

**Theorem 4.12.** If $p^k$ divides $|G|$, then $G$ has a subgroup of order $p^k$. In particular, Sylow $p$-subgroups exist.

### Sylow's Second Theorem (Conjugacy)

**Theorem 4.13.** All Sylow $p$-subgroups of $G$ are **conjugate**: if $P, Q \in \text{Syl}_p(G)$, then $Q = gPg^{-1}$ for some $g \in G$.

**Corollary.** A Sylow $p$-subgroup $P$ is **normal** iff $n_p = 1$.

### Sylow's Third Theorem (Counting)

**Theorem 4.14.** The number $n_p$ of Sylow $p$-subgroups satisfies:

1. $n_p \equiv 1 \pmod{p}$
2. $n_p$ divides $m = |G|/p^n$

```
  ┌──────────────────────────────────────────────────┐
  │  SYLOW THEOREMS SUMMARY                          │
  │                                                  │
  │  |G| = pⁿ · m,  gcd(p,m) = 1                    │
  │                                                  │
  │  (1) ∃ subgroup P with |P| = pⁿ                 │
  │  (2) All such P are conjugate                    │
  │  (3) nₚ ≡ 1 (mod p)  and  nₚ | m                │
  │                                                  │
  │  Consequence: nₚ = 1 ⟺ P ◁ G                   │
  └──────────────────────────────────────────────────┘
```

---

## 3. Proof Sketch (Sylow I)

**Strategy:** Induction on $|G|$, using the class equation.

If $p \mid |Z(G)|$, then by Cauchy's theorem $Z(G)$ has element $z$ of order $p$. Consider $G/\langle z \rangle$; by induction it has a subgroup of order $p^{n-1}$. Pull it back.

If $p \nmid |Z(G)|$, then the class equation gives:

$$|G| = |Z(G)| + \sum [G : C_G(a_i)]$$

Since $p \mid |G|$ and $p \nmid |Z(G)|$, some $[G : C_G(a_i)]$ is not divisible by $p$, meaning $p^n \mid |C_G(a_i)|$ with $|C_G(a_i)| < |G|$. Apply induction to $C_G(a_i)$. $\blacksquare$

---

## 4. Applying the Sylow Theorems

### Step-by-Step Method

```
  To find nₚ for |G| = pⁿ · m:
  
  Step 1: List divisors of m
  Step 2: Filter: keep those ≡ 1 (mod p)
  Step 3: The surviving values are candidates for nₚ
  Step 4: Use additional arguments to narrow down
```

### 4.1. Groups of Order 12

$|G| = 12 = 2^2 \cdot 3$.

**Sylow 3-subgroups:** $n_3 \mid 4$ and $n_3 \equiv 1 \pmod{3}$.
- Divisors of 4: $1, 2, 4$.
- $\equiv 1 \pmod{3}$: $1, 4$.
- So $n_3 = 1$ or $n_3 = 4$.

**Sylow 2-subgroups:** $n_2 \mid 3$ and $n_2 \equiv 1 \pmod{2}$.
- Divisors of 3: $1, 3$.
- $\equiv 1 \pmod{2}$: $1, 3$.
- So $n_2 = 1$ or $n_2 = 3$.

| Group | $n_3$ | $n_2$ | Notes |
|-------|-------|-------|-------|
| $\mathbb{Z}_{12}$ | 1 | 1 | Cyclic |
| $\mathbb{Z}_2 \times \mathbb{Z}_6$ | 1 | 1 | Abelian |
| $A_4$ | 4 | 1 | No element of order 6 |
| $D_6$ | 1 | 3 | Dihedral |
| $\text{Dic}_{12}$ | 1 | 1 | Dicyclic (generalised quaternion) |

### 4.2. Groups of Order 15

$|G| = 15 = 3 \cdot 5$.

- $n_3 \mid 5$, $n_3 \equiv 1 \pmod{3}$: $n_3 \in \{1, 5\}$, only $1$ works.
- $n_5 \mid 3$, $n_5 \equiv 1 \pmod{5}$: $n_5 \in \{1, 3\}$, only $1$ works.

So $n_3 = n_5 = 1$: both Sylow subgroups are **normal**.

$G \cong \mathbb{Z}_3 \times \mathbb{Z}_5 \cong \mathbb{Z}_{15}$. (Unique group of order 15.)

---

## 5. Important Consequences

### 5.1. $n_p = 1 \implies P \trianglelefteq G$

If there is a unique Sylow $p$-subgroup $P$, then $P$ is normal (since $gPg^{-1}$ is also a Sylow $p$-subgroup, it must equal $P$).

### 5.2. Counting Elements

If $n_p > 1$, distinct Sylow $p$-subgroups of prime order $p$ intersect only in $\{e\}$ (since subgroups of prime order are cyclic). This gives:

$$\text{elements of order } p \geq n_p(p - 1)$$

### 5.3. Groups of Order $pq$

**Theorem 4.15.** Let $p < q$ be primes and $|G| = pq$.

- If $p \nmid (q-1)$: $G \cong \mathbb{Z}_{pq}$ (unique).
- If $p \mid (q-1)$: either $G \cong \mathbb{Z}_{pq}$ or a non-abelian group $\mathbb{Z}_q \rtimes \mathbb{Z}_p$.

*Proof.* $n_q \mid p$, $n_q \equiv 1 \pmod{q}$. Since $p < q$, the only possibility is $n_q = 1$. So the Sylow $q$-subgroup is normal. Now $n_p \mid q$, $n_p \equiv 1 \pmod{p}$: so $n_p = 1$ or $n_p = q$ (when $q \equiv 1 \pmod{p}$). $\blacksquare$

---

## 6. Sylow Subgroups of $S_n$

| $n$ | $p$ | $\|S_n\| = n!$ | Sylow $p$-subgroup |
|-----|-----|-------------|-------------------|
| 3 | 2 | 6 | $\langle(12)\rangle \cong \mathbb{Z}_2$ |
| 3 | 3 | 6 | $\langle(123)\rangle \cong \mathbb{Z}_3$ |
| 4 | 2 | 24 | $D_4$ (order 8), $n_2 = 3$ |
| 4 | 3 | 24 | $\langle(123)\rangle \cong \mathbb{Z}_3$, $n_3 = 4$ |
| 5 | 5 | 120 | $\langle(12345)\rangle \cong \mathbb{Z}_5$, $n_5 = 6$ |

---

## 7. Normaliser and Sylow

**Theorem 4.16.** $n_p = [G : N_G(P)]$ where $P \in \text{Syl}_p(G)$.

This gives another way to compute $n_p$: it equals the index of the normaliser of any Sylow $p$-subgroup.

```
  nₚ = |G| / |N_G(P)|
  
  Since P ≤ N_G(P) ≤ G:
  
  |P| divides |N_G(P)| divides |G|
  p^n divides |N_G(P)| divides p^n · m
```

---

## 8. Worked Example: Groups of Order 36

$|G| = 36 = 2^2 \cdot 3^2$.

**Sylow 3-subgroups (order 9):**
$n_3 \mid 4$, $n_3 \equiv 1 \pmod{3}$: $n_3 \in \{1, 4\}$.

**Sylow 2-subgroups (order 4):**
$n_2 \mid 9$, $n_2 \equiv 1 \pmod{2}$: $n_2 \in \{1, 3, 9\}$.

If $n_3 = 4$: four subgroups of order 9. Each is abelian ($p^2$-group), so $\cong \mathbb{Z}_9$ or $\mathbb{Z}_3^2$.

This analysis, combined with semidirect product constructions, leads to a complete classification (there are 14 groups of order 36).

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Sylow $p$-subgroup | Subgroup of order $p^n$ (max. $p$-power dividing $\|G\|$) |
| Existence (Sylow I) | Sylow $p$-subgroups always exist |
| Conjugacy (Sylow II) | All Sylow $p$-subgroups are conjugate |
| Counting (Sylow III) | $n_p \equiv 1 \pmod{p}$ and $n_p \mid m$ |
| Normality | $P \trianglelefteq G \iff n_p = 1$ |
| $n_p$ via normaliser | $n_p = [G : N_G(P)]$ |
| Order $pq$ ($p < q$) | $n_q = 1$; when $p \nmid (q{-}1)$, $G \cong \mathbb{Z}_{pq}$ |

---

## Quick Revision Questions

1. **Find** all possible values of $n_2, n_3, n_5$ for a group of order 60.

2. **Prove** that every group of order 35 is cyclic.

3. **Show** that a group of order 99 has a normal Sylow 11-subgroup.

4. **Determine** $n_7$ for a group of order 168. Which groups can arise?

5. **Use** Sylow's theorems to show that no group of order 12 is simple.

6. **State** the three Sylow theorems and explain the role of each in classifying groups.

---

[← Previous: Class Equation](03-class-equation.md) | [Back to README](../README.md) | [Next: Applications →](05-applications.md)
