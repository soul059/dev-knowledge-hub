# Chapter 5.4 — Fields (Introduction)

[← Previous: Integral Domains](03-integral-domains.md) | [Back to README](../README.md) | [Next: Characteristic →](05-characteristic.md)

---

## Overview

A **field** is a commutative ring where every non-zero element is invertible. Fields are the "nicest" rings—they support division, and linear algebra works perfectly over them. This chapter introduces fields from the ring-theoretic perspective; deeper theory (extensions, Galois theory) appears in Unit 8.

---

## 1. Definition

**Definition.** A **field** is a commutative ring $F$ with $1 \neq 0$ such that every non-zero element has a multiplicative inverse:

$$\forall a \in F,\ a \neq 0 \implies \exists a^{-1} \in F:\ a \cdot a^{-1} = 1$$

Equivalently: $F^\times = F \setminus \{0\}$ is an abelian group under multiplication.

```
  Field = Commutative ring where F \ {0} is a group under ·
  
  Ring  ⊃  Integral Domain  ⊃  Field
  
  Every field is an integral domain.
  (If ab = 0 and a ≠ 0, multiply by a⁻¹: b = 0.)
  
  Every finite integral domain is a field.
```

---

## 2. Examples of Fields

| Field | Description | $\text{char}$ |
|-------|------------|---------------|
| $\mathbb{Q}$ | Rationals | 0 |
| $\mathbb{R}$ | Reals | 0 |
| $\mathbb{C}$ | Complex numbers | 0 |
| $\mathbb{F}_p = \mathbb{Z}/p\mathbb{Z}$ | Finite field of $p$ elements | $p$ |
| $\mathbb{Q}(\sqrt{2})$ | $\{a + b\sqrt{2} : a,b \in \mathbb{Q}\}$ | 0 |
| $\mathbb{Q}(i)$ | $\{a + bi : a,b \in \mathbb{Q}\}$ | 0 |
| $\mathbb{F}_p(x)$ | Rational functions over $\mathbb{F}_p$ | $p$ |
| $\mathbb{F}_{p^n}$ | Finite field of $p^n$ elements | $p$ |

### Non-Examples

| Structure | Why not a field |
|-----------|----------------|
| $\mathbb{Z}$ | $2$ has no multiplicative inverse |
| $\mathbb{Z}_6$ | Not a domain ($2 \cdot 3 = 0$) |
| $\mathbb{H}$ | Non-commutative (division ring, not field) |
| $\mathbb{R}[x]$ | $x$ has no inverse |

---

## 3. Constructing Fields

### 3.1. Field of Fractions

If $R$ is an integral domain: $F = \text{Frac}(R)$ is a field.

$\text{Frac}(\mathbb{Z}) = \mathbb{Q}$, $\text{Frac}(\mathbb{Z}[i]) = \mathbb{Q}(i)$.

### 3.2. Quotient by Maximal Ideal

$R/M$ is a field $\iff M$ is a maximal ideal of $R$.

**Example:** $\mathbb{Z}[x]/(x^2 + 1) \cong \mathbb{Z}[i]$? No—this isn't quite right over $\mathbb{Z}$. But:

$$\mathbb{R}[x]/(x^2 + 1) \cong \mathbb{C}$$

$x^2 + 1$ is irreducible over $\mathbb{R}$, so $(x^2+1)$ is maximal in $\mathbb{R}[x]$ (PID), so the quotient is a field.

### 3.3. Finite Fields

For every prime power $q = p^n$, there exists a unique (up to isomorphism) field $\mathbb{F}_q$ of order $q$.

$\mathbb{F}_{p^n} \cong \mathbb{F}_p[x]/(f(x))$ where $f$ is irreducible of degree $n$ over $\mathbb{F}_p$.

```
  Finite fields:
  
  |F| must be a prime power p^n
  
  For each p^n, exactly ONE field (up to ≅)
  
  F₂ = {0,1}
  F₃ = {0,1,2}
  F₄ = F₂[x]/(x²+x+1) = {0, 1, α, α+1}  where α² = α+1
  F₅ = {0,1,2,3,4}
  F₇ = {0,1,...,6}
  F₈ = F₂[x]/(x³+x+1) = {0,1,α,α²,α+1,α²+1,α²+α,α²+α+1}
  F₉ = F₃[x]/(x²+1)   = {a + bα : a,b ∈ F₃}, α² = -1
```

---

## 4. $\mathbb{F}_4$ — A Detailed Example

$\mathbb{F}_4 = \mathbb{F}_2[x]/(x^2 + x + 1)$. Let $\alpha$ be a root of $x^2 + x + 1 = 0$, so $\alpha^2 = \alpha + 1$.

Elements: $\{0, 1, \alpha, \alpha+1\}$.

**Addition table** (characteristic 2: $-1 = 1$):

| $+$ | $0$ | $1$ | $\alpha$ | $\alpha{+}1$ |
|-----|-----|-----|----------|-------------|
| $0$ | $0$ | $1$ | $\alpha$ | $\alpha{+}1$ |
| $1$ | $1$ | $0$ | $\alpha{+}1$ | $\alpha$ |
| $\alpha$ | $\alpha$ | $\alpha{+}1$ | $0$ | $1$ |
| $\alpha{+}1$ | $\alpha{+}1$ | $\alpha$ | $1$ | $0$ |

**Multiplication table:**

| $\cdot$ | $0$ | $1$ | $\alpha$ | $\alpha{+}1$ |
|---------|-----|-----|----------|-------------|
| $0$ | $0$ | $0$ | $0$ | $0$ |
| $1$ | $0$ | $1$ | $\alpha$ | $\alpha{+}1$ |
| $\alpha$ | $0$ | $\alpha$ | $\alpha{+}1$ | $1$ |
| $\alpha{+}1$ | $0$ | $\alpha{+}1$ | $1$ | $\alpha$ |

Note: $\alpha^{-1} = \alpha + 1$ and $(\alpha+1)^{-1} = \alpha$.

$\mathbb{F}_4^\times = \{1, \alpha, \alpha+1\} \cong \mathbb{Z}_3$ (cyclic!).

---

## 5. The Multiplicative Group of a Field

**Theorem 5.11.** If $F$ is a field, then $F^\times = F \setminus \{0\}$ is a group under multiplication.

**Theorem 5.12.** Every finite subgroup of $F^\times$ is **cyclic**.

*Proof sketch.* A finite abelian group that is not cyclic has too many solutions to $x^d = 1$ for some $d$ (more than $d$ solutions), contradicting the fact that a polynomial of degree $d$ over a field has at most $d$ roots. $\blacksquare$

**Corollary.** If $F$ is a finite field, then $F^\times$ is cyclic. A generator is called a **primitive element**.

---

## 6. Prime Fields

**Definition.** A **prime field** is a field with no proper subfields.

**Theorem 5.13.** The prime fields are:
- $\mathbb{Q}$ (characteristic 0)
- $\mathbb{F}_p$ (characteristic $p$)

Every field contains a unique copy of its prime field.

```
  Every field F contains a prime subfield:
  
  char(F) = 0  ⟹  prime subfield ≅ ℚ
  char(F) = p  ⟹  prime subfield ≅ F_p
```

---

## 7. Division Rings

**Definition.** A **division ring** (or **skew field**) is a ring where every non-zero element is invertible (not necessarily commutative).

**Wedderburn's Theorem 5.14.** Every finite division ring is a field (i.e., is commutative).

| Structure | Commutative $\cdot$ | Every $a \neq 0$ invertible |
|-----------|---------------------|----------------------------|
| Ring | Not required | Not required |
| Integral domain | Yes | Not required |
| Division ring | Not required | Yes |
| Field | Yes | Yes |

---

## 8. Subfields

**Definition.** A **subfield** $K$ of $F$ is a subring that is itself a field.

Equivalently: $K \subseteq F$ with $1 \in K$, closed under $+, -, \cdot$, and $a^{-1}$ (for $a \neq 0$).

**Subfield test:** $K \neq \{0\}$, $a - b \in K$, $ab^{-1} \in K$ (for $b \neq 0$).

### Examples

- $\mathbb{Q} \subseteq \mathbb{R} \subseteq \mathbb{C}$: each is a subfield of the next.
- $\mathbb{Q}(\sqrt{2}) \subseteq \mathbb{R}$: subfield.
- $\mathbb{F}_p \subseteq \mathbb{F}_{p^n}$ for all $n$.
- $\mathbb{F}_{p^m} \subseteq \mathbb{F}_{p^n}$ iff $m \mid n$.

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Field | Commutative ring, $1 \neq 0$, every $a \neq 0$ invertible |
| Field $\implies$ domain | $ab = 0, a \neq 0 \implies b = a^{-1}(ab) = 0$ |
| Finite domain $\implies$ field | Pigeonhole / injective on finite set |
| $\mathbb{F}_q$ exists | For every prime power $q = p^n$; unique up to $\cong$ |
| $F^\times$ cyclic | For finite fields |
| Prime fields | $\mathbb{Q}$ or $\mathbb{F}_p$ |
| Wedderburn | Finite division ring $\implies$ field |

---

## Quick Revision Questions

1. **Prove** that every field is an integral domain.

2. **Construct** $\mathbb{F}_9$ as $\mathbb{F}_3[x]/(x^2 + 1)$ and write out the multiplication table.

3. **Show** that $\mathbb{Q}(\sqrt{3}) = \{a + b\sqrt{3} : a, b \in \mathbb{Q}\}$ is a field.

4. **Prove** that every finite subgroup of $F^\times$ is cyclic.

5. **Find** a primitive element (generator of $F^\times$) for $\mathbb{F}_7$.

6. **State** Wedderburn's theorem and explain why it does not apply to $\mathbb{H}$.

---

[← Previous: Integral Domains](03-integral-domains.md) | [Back to README](../README.md) | [Next: Characteristic →](05-characteristic.md)
