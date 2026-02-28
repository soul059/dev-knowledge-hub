# Chapter 1.2 — Order of an Element

[← Previous: Definition and Examples](01-definition-and-examples.md) | [Back to README](../README.md) | [Next: Subgroups →](03-subgroups.md)

---

## Overview

The **order** of an element measures how many times you must apply the group operation before returning to the identity. This concept is central to understanding group structure, cyclic behaviour, and element classification.

---

## 1. Definition

**Definition.** Let $(G, \ast)$ be a group and $a \in G$. The **order** of $a$, written $\text{ord}(a)$ or $|a|$, is the smallest positive integer $n$ such that:

$$a^n = e$$

If no such $n$ exists, we say $a$ has **infinite order** and write $|a| = \infty$.

### Notation in Additive vs Multiplicative Groups

| Notation | Multiplicative | Additive |
|----------|---------------|----------|
| Repeated operation | $a^n = \underbrace{a \cdot a \cdots a}_{n}$ | $na = \underbrace{a + a + \cdots + a}_{n}$ |
| Order condition | $a^n = e$ | $na = 0$ |

---

## 2. Computing Orders — Examples

### Example 1: $(\mathbb{Z}_{12}, +)$

Find the order of each element:

| Element $a$ | Smallest $n$ with $na \equiv 0 \pmod{12}$ | $\|a\|$ |
|------------|-------------------------------------------|---------|
| $0$ | $1 \cdot 0 = 0$ | $1$ |
| $1$ | $12 \cdot 1 = 12 \equiv 0$ | $12$ |
| $2$ | $6 \cdot 2 = 12 \equiv 0$ | $6$ |
| $3$ | $4 \cdot 3 = 12 \equiv 0$ | $4$ |
| $4$ | $3 \cdot 4 = 12 \equiv 0$ | $3$ |
| $5$ | $12 \cdot 5 = 60 \equiv 0$ | $12$ |
| $6$ | $2 \cdot 6 = 12 \equiv 0$ | $2$ |
| $7$ | $12 \cdot 7 = 84 \equiv 0$ | $12$ |
| $8$ | $3 \cdot 8 = 24 \equiv 0$ | $3$ |
| $9$ | $4 \cdot 9 = 36 \equiv 0$ | $4$ |
| $10$ | $6 \cdot 10 = 60 \equiv 0$ | $6$ |
| $11$ | $12 \cdot 11 = 132 \equiv 0$ | $12$ |

**Formula:** In $(\mathbb{Z}_n, +)$, the order of element $a$ is:

$$|a| = \frac{n}{\gcd(a, n)}$$

### Example 2: $(\mathbb{Z}_8^\times, \cdot)$

$\mathbb{Z}_8^\times = \{1, 3, 5, 7\}$

| Element $a$ | Powers $a, a^2, a^3, \ldots$ | $\|a\|$ |
|------------|------------------------------|---------|
| $1$ | $1$ | $1$ |
| $3$ | $3, 9 \equiv 1$ | $2$ |
| $5$ | $5, 25 \equiv 1$ | $2$ |
| $7$ | $7, 49 \equiv 1$ | $2$ |

Every non-identity element has order 2! This group is $V_4 \cong \mathbb{Z}_2 \times \mathbb{Z}_2$.

### Example 3: $S_3$

| Element | Cycle notation | Order |
|---------|---------------|-------|
| $e$ | $()$ | $1$ |
| $(1\;2)$ | transposition | $2$ |
| $(1\;3)$ | transposition | $2$ |
| $(2\;3)$ | transposition | $2$ |
| $(1\;2\;3)$ | 3-cycle | $3$ |
| $(1\;3\;2)$ | 3-cycle | $3$ |

**Rule for permutations:** The order of a permutation is the **LCM** of its cycle lengths.

```
  Order of a permutation:
  ┌─────────────────────────────────────────────────┐
  │  σ = (a₁ a₂ ··· aₖ)(b₁ b₂ ··· bₘ)···          │
  │                                                  │
  │  |σ| = lcm(k, m, ...)                           │
  │                                                  │
  │  Example: σ = (1 2 3)(4 5) ∈ S₅                 │
  │  |σ| = lcm(3, 2) = 6                            │
  └─────────────────────────────────────────────────┘
```

---

## 3. Key Theorems on Order

### Theorem 1.5: $a^k = e$ if and only if $|a|$ divides $k$

*Proof.* ($\Leftarrow$) If $k = |a| \cdot q$, then $a^k = (a^{|a|})^q = e^q = e$.

($\Rightarrow$) Write $k = |a| \cdot q + r$ with $0 \leq r < |a|$. Then:
$$e = a^k = a^{|a| \cdot q + r} = (a^{|a|})^q \cdot a^r = e^q \cdot a^r = a^r$$

Since $0 \leq r < |a|$ and $|a|$ is the smallest positive integer with $a^{|a|} = e$, we must have $r = 0$. $\square$

### Theorem 1.6: Order of Powers

If $|a| = n$, then:

$$|a^k| = \frac{n}{\gcd(n, k)}$$

**Diagram — Order relationships in $\mathbb{Z}_{12}$:**

```
                    12
                   / | \
                  /  |  \
                 6   4   3          ← Orders of elements
                / \ / \   \
               3   2   1            ← gcd structure
                \ /  \ /
                 1
                 
  Element:  |1| = 12    |2| = 6     |3| = 4     |4| = 3
            |5| = 12    |6| = 2     |7| = 12    |8| = 3
            |9| = 4     |10| = 6   |11| = 12
```

### Theorem 1.7: Order of Inverses

$$|a^{-1}| = |a|$$

*Proof.* If $a^n = e$, raise both sides to the power $-1$: $(a^{-1})^n = e$. The minimality of $n$ follows symmetrically. $\square$

### Theorem 1.8: Order in Abelian Groups

If $G$ is abelian, $|a| = m$, $|b| = n$, and $\gcd(m, n) = 1$, then:

$$|ab| = mn$$

---

## 4. Elements of Finite Order vs Infinite Order

### Infinite Order Example: $(\mathbb{Z}, +)$

Consider $a = 1$. The multiples $1, 2, 3, \ldots$ never equal $0$ (the identity). So $|1| = \infty$.

In fact, in $(\mathbb{Z}, +)$, every non-zero element has infinite order.

### Torsion Elements

**Definition.** An element $a \in G$ is a **torsion element** if $|a| < \infty$. The set of all torsion elements of an abelian group forms a subgroup called the **torsion subgroup**.

```
  Classification of elements by order:
  
  ┌───────────────────────────────────────────┐
  │              Elements of G                 │
  ├───────────────────┬───────────────────────┤
  │  Finite Order     │  Infinite Order        │
  │  (Torsion)        │                        │
  │                   │                        │
  │  a^n = e for      │  a^n ≠ e for all      │
  │  some n > 0       │  n > 0                │
  │                   │                        │
  │  Ex: rotations    │  Ex: translations     │
  │  of a polygon     │  of the plane         │
  └───────────────────┴───────────────────────┘
```

---

## 5. The Exponent of a Group

**Definition.** The **exponent** of a finite group $G$ is the smallest positive integer $m$ such that $a^m = e$ for all $a \in G$.

- The exponent divides $|G|$.
- The exponent equals $\text{lcm}\{|a| : a \in G\}$.

**Example:**

| Group | Orders present | Exponent |
|-------|---------------|----------|
| $\mathbb{Z}_6$ | $1, 2, 3, 6$ | $6$ |
| $V_4$ | $1, 2$ | $2$ |
| $S_3$ | $1, 2, 3$ | $6$ |
| $\mathbb{Z}_2 \times \mathbb{Z}_4$ | $1, 2, 4$ | $4$ |

---

## 6. Order and Group Structure

The distribution of element orders reveals much about a group's structure:

```
  Group orders and structure:
  
  All elements     Every non-identity      Orders divide a
  have order 1     element has order 2     single prime p
       │                  │                      │
       ▼                  ▼                      ▼
  Trivial {e}     Elementary abelian        p-group
                   2-group (≅ Z₂ᵏ)
```

**Theorem 1.9.** If every non-identity element of $G$ has order 2, then $G$ is abelian.

*Proof.* For all $a, b \in G$: $a^2 = e$ means $a = a^{-1}$. Then
$$ab = (ab)^{-1} = b^{-1}a^{-1} = ba. \quad \square$$

---

## 7. Worked Example: Finding All Element Orders in $D_4$

$D_4 = \{e, r, r^2, r^3, s, rs, r^2s, r^3s\}$ where $r^4 = e$, $s^2 = e$, $srs = r^{-1}$.

| Element | Computation | Order |
|---------|------------|-------|
| $e$ | — | $1$ |
| $r$ | $r, r^2, r^3, r^4 = e$ | $4$ |
| $r^2$ | $r^2, r^4 = e$ | $2$ |
| $r^3$ | $r^3, r^6 = r^2, r^9 = r, r^{12} = e$ | $4$ |
| $s$ | $s, s^2 = e$ | $2$ |
| $rs$ | $(rs)^2 = rssr^{-1} \cdot r = \ldots = e$ | $2$ |
| $r^2s$ | $(r^2s)^2 = e$ | $2$ |
| $r^3s$ | $(r^3s)^2 = e$ | $2$ |

**Order profile of $D_4$:**

| Order | Count | Elements |
|-------|-------|----------|
| 1 | 1 | $e$ |
| 2 | 5 | $r^2, s, rs, r^2s, r^3s$ |
| 4 | 2 | $r, r^3$ |

Compare with $\mathbb{Z}_8$: has elements of orders $1, 2, 4, 8$. So $D_4 \not\cong \mathbb{Z}_8$.

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| $\|a\|$ | Smallest $n > 0$ with $a^n = e$ |
| $a^k = e$ | $\iff |a|$ divides $k$ |
| $\|a^k\|$ | $= n / \gcd(n, k)$ where $\|a\| = n$ |
| $\|a^{-1}\|$ | $= \|a\|$ |
| Permutation order | $= \text{lcm}$ of cycle lengths |
| Order in $\mathbb{Z}_n$ | $\|a\| = n / \gcd(a, n)$ |
| Exponent of $G$ | $= \text{lcm}\{\|a\| : a \in G\}$ |
| All elements order 2 | $\implies G$ is abelian |

---

## Quick Revision Questions

1. **Compute** the order of $5$ in $(\mathbb{Z}_{20}, +)$.

2. **Find** the order of the permutation $(1\;3\;5\;7)(2\;4\;6)$ in $S_7$.

3. **Prove** that if $|a| = n$ and $a^k = e$, then $n \mid k$.

4. **Determine** all elements of order $4$ in $\mathbb{Z}_{20}$.

5. **Show** that in $\mathbb{Z}_n$, the number of elements of order $n$ is $\varphi(n)$.

6. **If** $|a| = 12$, what is $|a^8|$? What is $|a^5|$?

---

[← Previous: Definition and Examples](01-definition-and-examples.md) | [Back to README](../README.md) | [Next: Subgroups →](03-subgroups.md)
