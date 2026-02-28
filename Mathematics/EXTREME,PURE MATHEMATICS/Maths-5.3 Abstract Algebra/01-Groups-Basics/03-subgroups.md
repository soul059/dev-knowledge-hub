# Chapter 1.3 — Subgroups

[← Previous: Order of an Element](02-order-of-element.md) | [Back to README](../README.md) | [Next: Cyclic Groups →](04-cyclic-groups.md)

---

## Overview

A **subgroup** is a subset of a group that is itself a group under the same operation. Subgroups are the fundamental building blocks for understanding group structure. We develop efficient tests for identifying subgroups and visualise their relationships using **subgroup lattice diagrams**.

---

## 1. Definition

**Definition.** Let $(G, \ast)$ be a group and $H \subseteq G$. We say $H$ is a **subgroup** of $G$, written $H \leq G$, if $(H, \ast)$ is itself a group under the same operation.

**Trivial subgroups:** Every group $G$ has at least two subgroups:
- $\{e\} \leq G$ — the **trivial subgroup**
- $G \leq G$ — the **improper subgroup**

A subgroup $H$ with $\{e\} \neq H \neq G$ is called a **proper, non-trivial subgroup**.

---

## 2. Subgroup Tests

Rather than checking all four group axioms, we have efficient criteria:

### Two-Step Subgroup Test

**Theorem 1.10.** A non-empty subset $H$ of a group $G$ is a subgroup if and only if:
1. $a, b \in H \implies ab \in H$ (closure)
2. $a \in H \implies a^{-1} \in H$ (closed under inverses)

*Proof.* Associativity is inherited from $G$. Closure + inverses guarantee $e = aa^{-1} \in H$. $\square$

### One-Step Subgroup Test

**Theorem 1.11.** A non-empty subset $H$ of $G$ is a subgroup if and only if:

$$a, b \in H \implies ab^{-1} \in H$$

*Proof.* Take $a = b$: $e = aa^{-1} \in H$. Take $a = e$: $b^{-1} = eb^{-1} \in H$. Then for $a, b \in H$: $b^{-1} \in H$ so $ab = a(b^{-1})^{-1} \in H$ by applying the condition to $a$ and $b^{-1}$. $\square$

### Finite Subgroup Test

**Theorem 1.12.** If $G$ is a group and $H$ is a **finite** non-empty subset of $G$ closed under the operation, then $H \leq G$.

*Proof.* We only need inverses. Let $a \in H$. The elements $a, a^2, a^3, \ldots$ are all in $H$ (closure). Since $H$ is finite, $a^i = a^j$ for some $i > j$, so $a^{i-j} = e \in H$. Then $a^{i-j-1} = a^{-1} \in H$. $\square$

```
  Subgroup Test Decision:
  
       Is H non-empty?
            │
       No ──┤── Yes
       ↓    │
  Not a     ▼
  subgroup  Is H finite?
            │
       Yes──┤── No
       ↓    │
    Check   ▼
    only:   Check:
  Closure   ab⁻¹ ∈ H
    only    for all a,b ∈ H
```

---

## 3. Important Subgroup Examples

### 3.1 Subgroups of $(\mathbb{Z}, +)$

**Theorem 1.13.** Every subgroup of $(\mathbb{Z}, +)$ has the form $n\mathbb{Z} = \{nk : k \in \mathbb{Z}\}$ for some $n \geq 0$.

```
  Subgroups of (Z, +):

  0Z = {0}
  1Z = Z
  2Z = {..., -4, -2, 0, 2, 4, ...}        (even integers)
  3Z = {..., -6, -3, 0, 3, 6, ...}         (multiples of 3)
  nZ = {..., -2n, -n, 0, n, 2n, ...}       (multiples of n)

  Containment:  nZ ⊆ mZ  ⟺  m | n
```

### 3.2 Subgroups of $\mathbb{Z}_n$

The subgroups of $\mathbb{Z}_n$ correspond to divisors of $n$. For each divisor $d$ of $n$, there is exactly one subgroup of order $d$:

$$\langle n/d \rangle = \{0, n/d, 2n/d, \ldots, (d-1) \cdot n/d\}$$

**Example: Subgroups of $\mathbb{Z}_{12}$**

| Divisor $d$ of 12 | Generator $12/d$ | Subgroup | Order |
|-------------------|-------------------|----------|-------|
| 1 | 12 ≡ 0 | $\{0\}$ | 1 |
| 2 | 6 | $\{0, 6\}$ | 2 |
| 3 | 4 | $\{0, 4, 8\}$ | 3 |
| 4 | 3 | $\{0, 3, 6, 9\}$ | 4 |
| 6 | 2 | $\{0, 2, 4, 6, 8, 10\}$ | 6 |
| 12 | 1 | $\mathbb{Z}_{12}$ | 12 |

### 3.3 Centre of a Group

**Definition.** The **centre** of $G$ is $Z(G) = \{z \in G : zg = gz \text{ for all } g \in G\}$.

**Proposition.** $Z(G) \leq G$.

*Proof.* $e \in Z(G)$, so $Z(G) \neq \emptyset$. If $a, b \in Z(G)$, then for all $g \in G$:
$(ab^{-1})g = a(b^{-1}g) = a(gb^{-1}) = (ag)b^{-1} = (ga)b^{-1} = g(ab^{-1})$. $\square$

### 3.4 Centraliser

**Definition.** For $a \in G$, the **centraliser** of $a$ is $C_G(a) = \{g \in G : ga = ag\}$.

Then $Z(G) = \bigcap_{a \in G} C_G(a)$.

---

## 4. Subgroup Lattice Diagrams

A **subgroup lattice** displays all subgroups with lines indicating containment (subgroup at bottom is contained in subgroup at top).

### Lattice of $\mathbb{Z}_{12}$

```
                  Z₁₂
                 / | \
                /  |  \
               /   |   \
             ⟨2⟩  ⟨3⟩  ⟨4⟩       orders: 6, 4, 3
              |  \/ |  / |
              | /\  | /  |
             ⟨4⟩  ⟨6⟩              orders: 3, 2
              |   \/ 
              |   /\
              | /    \
             ⟨6⟩    ⟨4⟩
              \      /
               \    /
                {0}                 order: 1

  Corrected lattice:

                    Z₁₂                    order 12
                   / | \
                 /   |   \
              ⟨2⟩   |   ⟨3⟩                orders 6, 4
               | \  |  / |
               |  ⟨3⟩  / |
               | / | \/  |
              ⟨4⟩  ⟨6⟩   |                 orders 3, 2
                \  |  /
                 \ | /
                  {0}                       order 1
```

### Lattice of $S_3$

```
                    S₃                     order 6
                   /  \
                 /      \
           ⟨(123)⟩   ⟨(12)⟩ ⟨(13)⟩ ⟨(23)⟩   orders 3, 2, 2, 2
             = A₃     
                \     |    |    /
                 \    |    |   /
                  \   |    |  /
                    {e}                    order 1

  (A₃ is the unique subgroup of order 3)
```

### Lattice of $D_4$ (order 8)

```
                         D₄                          order 8
                    /   / | \   \
                  /   /   |   \   \
               ⟨r⟩  ⟨r²,s⟩ ⟨r²,rs⟩  ...            order 4
                |   / \ / \  / \
                | /   ⟨r²⟩  \   \                    order 2
                |/   / | \   \   \
              ⟨r²⟩⟨s⟩⟨r²s⟩⟨rs⟩⟨r³s⟩                 order 2
                 \  \ | /  /
                   \  |  /
                    {e}                               order 1
```

---

## 5. Intersection of Subgroups

**Theorem 1.14.** If $H \leq G$ and $K \leq G$, then $H \cap K \leq G$.

*Proof.* $e \in H \cap K$, so it's non-empty. If $a, b \in H \cap K$, then $ab^{-1} \in H$ (since $H \leq G$) and $ab^{-1} \in K$ (since $K \leq G$), so $ab^{-1} \in H \cap K$. $\square$

> **Warning:** $H \cup K$ is generally **not** a subgroup!

**Theorem 1.15.** $H \cup K \leq G$ if and only if $H \subseteq K$ or $K \subseteq H$.

---

## 6. Subgroup Generated by a Subset

**Definition.** For $S \subseteq G$, the **subgroup generated by $S$** is:

$$\langle S \rangle = \bigcap_{\substack{H \leq G \\ S \subseteq H}} H$$

This is the smallest subgroup of $G$ containing $S$.

**Explicit description:**

$$\langle S \rangle = \{s_1^{\epsilon_1} s_2^{\epsilon_2} \cdots s_k^{\epsilon_k} : s_i \in S,\; \epsilon_i = \pm 1,\; k \geq 0\}$$

(All finite products of elements of $S$ and their inverses.)

---

## 7. Centraliser, Normaliser, and Centre — Comparison

| Subgroup | Notation | Definition | Always a subgroup? |
|----------|----------|------------|-------------------|
| Centre | $Z(G)$ | $\{g : gx = xg\;\forall x \in G\}$ | Yes |
| Centraliser of $a$ | $C_G(a)$ | $\{g : ga = ag\}$ | Yes |
| Normaliser of $H$ | $N_G(H)$ | $\{g : gHg^{-1} = H\}$ | Yes |

**Containment:** $Z(G) \subseteq C_G(a) \subseteq G$ and $H \subseteq N_G(H) \subseteq G$.

---

## 8. Real-World Applications

| Application | Subgroup Used |
|-------------|--------------|
| Symmetry breaking in physics | Residual symmetry is a subgroup |
| Crystallography | Point groups are subgroups of $O(3)$ |
| Coding theory | Subgroups of $\mathbb{Z}_2^n$ give linear codes |
| Key exchange (Diffie-Hellman) | Subgroups of $\mathbb{Z}_p^\times$ |

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Subgroup $H \leq G$ | $(H, \ast)$ is a group, same operation |
| One-step test | $H \neq \emptyset$ and $ab^{-1} \in H$ for all $a, b \in H$ |
| Finite subgroup test | $H$ finite, non-empty, closed $\implies H \leq G$ |
| Subgroups of $\mathbb{Z}$ | All of the form $n\mathbb{Z}$ |
| Subgroups of $\mathbb{Z}_n$ | One per divisor of $n$ |
| $H \cap K$ | Always a subgroup |
| $H \cup K$ | Subgroup $\iff H \subseteq K$ or $K \subseteq H$ |
| Centre $Z(G)$ | Elements commuting with everything |
| $\langle S \rangle$ | Smallest subgroup containing $S$ |

---

## Quick Revision Questions

1. **State** the one-step subgroup test and prove it is equivalent to the two-step test.

2. **List** all subgroups of $\mathbb{Z}_{18}$ and draw the subgroup lattice.

3. **Show** that $\{1, -1, i, -i\}$ is a subgroup of $(\mathbb{C} \setminus \{0\}, \cdot)$.

4. **Prove** that the intersection of any collection of subgroups is a subgroup.

5. **Find** the centre $Z(D_4)$.

6. **Give** an example showing that $H \cup K$ need not be a subgroup, even when both $H$ and $K$ are subgroups.

---

[← Previous: Order of an Element](02-order-of-element.md) | [Back to README](../README.md) | [Next: Cyclic Groups →](04-cyclic-groups.md)
