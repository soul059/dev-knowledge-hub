# Chapter 8.1 — Field Extensions

[← Previous: Irreducibility Criteria](../07-Rings-Factorization/05-irreducibility-criteria.md) | [Back to README](../README.md) | [Next: Algebraic and Transcendental Extensions →](02-algebraic-and-transcendental.md)

---

## Overview

**Field extensions** are the central objects of study in field theory and Galois theory. We define extensions, their degree, the tower law, and simple extensions — building the framework for understanding algebraic numbers, finite fields, and solvability of equations.

---

## 1. Definition

**Definition.** A **field extension** $E/F$ (read "$E$ over $F$") consists of two fields $F \subseteq E$ where $F$ is a subfield of $E$, or equivalently, there is an injective ring homomorphism $\iota: F \hookrightarrow E$.

We call $F$ the **base field** and $E$ the **extension field**.

**Key observation:** $E$ is a vector space over $F$.

---

## 2. Degree of an Extension

**Definition.** The **degree** of $E/F$ is:

$$[E : F] = \dim_F(E)$$

the dimension of $E$ as an $F$-vector space.

- If $[E:F] < \infty$: $E/F$ is a **finite extension**.
- If $[E:F] = \infty$: $E/F$ is an **infinite extension**.

**Examples:**

| Extension | Degree | Basis |
|-----------|--------|-------|
| $\mathbb{C}/\mathbb{R}$ | 2 | $\{1, i\}$ |
| $\mathbb{Q}(\sqrt{2})/\mathbb{Q}$ | 2 | $\{1, \sqrt{2}\}$ |
| $\mathbb{Q}(\sqrt[3]{2})/\mathbb{Q}$ | 3 | $\{1, \sqrt[3]{2}, \sqrt[3]{4}\}$ |
| $\mathbb{R}/\mathbb{Q}$ | $\infty$ | uncountable |
| $\mathbb{F}_{p^n}/\mathbb{F}_p$ | $n$ | $\{1, \alpha, \alpha^2, \ldots, \alpha^{n-1}\}$ |
| $F/F$ | 1 | $\{1\}$ |

---

## 3. The Tower Law

**Theorem 8.1 (Tower Law / Degree Formula).** If $F \subseteq K \subseteq E$ are fields with $[E:K]$ and $[K:F]$ both finite, then:

$$[E : F] = [E : K] \cdot [K : F]$$

```
  Tower of extensions:
  
  E          [E:K] = m
  │
  K          [K:F] = n
  │
  F          [E:F] = mn
  
  If {e₁,...,eₘ} is a K-basis for E
  and {f₁,...,fₙ} is an F-basis for K,
  then {eᵢfⱼ : 1≤i≤m, 1≤j≤n} is an F-basis for E.
```

*Proof.* Let $\{e_1, \ldots, e_m\}$ be a $K$-basis for $E$ and $\{f_1, \ldots, f_n\}$ be an $F$-basis for $K$. We claim $\{e_i f_j\}$ is an $F$-basis for $E$.

*Spanning:* Any $\alpha \in E$ is $\sum_i k_i e_i$ with $k_i \in K$, and each $k_i = \sum_j a_{ij} f_j$ with $a_{ij} \in F$. So $\alpha = \sum_{i,j} a_{ij} e_i f_j$.

*Independence:* If $\sum_{i,j} a_{ij} e_i f_j = 0$, then $\sum_i (\sum_j a_{ij} f_j) e_i = 0$. By $K$-independence of $\{e_i\}$, each $\sum_j a_{ij} f_j = 0$. By $F$-independence of $\{f_j\}$, each $a_{ij} = 0$. $\blacksquare$

---

## 4. Tower Law Applications

**Corollary 1.** If $[E:F] = p$ (prime), then there is no intermediate field $K$ with $F \subsetneq K \subsetneq E$.

*Proof.* $[E:F] = [E:K][K:F]$. Since $p$ is prime, one factor is 1. $\blacksquare$

**Corollary 2.** $\sqrt{2} \notin \mathbb{Q}(\sqrt{3})$.

*Proof.* $[\mathbb{Q}(\sqrt{3}):\mathbb{Q}] = 2$. If $\sqrt{2} \in \mathbb{Q}(\sqrt{3})$, then $\mathbb{Q}(\sqrt{2}) \subseteq \mathbb{Q}(\sqrt{3})$, so $[\mathbb{Q}(\sqrt{3}):\mathbb{Q}(\sqrt{2})] \cdot [\mathbb{Q}(\sqrt{2}):\mathbb{Q}] = 2$, giving $[\mathbb{Q}(\sqrt{3}):\mathbb{Q}(\sqrt{2})] = 1$, i.e., $\mathbb{Q}(\sqrt{2}) = \mathbb{Q}(\sqrt{3})$. But then $\sqrt{6} = \sqrt{2}\cdot\sqrt{3} \in \mathbb{Q}(\sqrt{2})$, so $\sqrt{6} = a + b\sqrt{2}$, squaring gives $6 = a^2 + 2b^2 + 2ab\sqrt{2}$, forcing $ab = 0$ and $a^2 + 2b^2 = 6$. If $a = 0$: $b^2 = 3$, impossible in $\mathbb{Q}$. If $b = 0$: $a^2 = 6$, impossible. Contradiction. $\blacksquare$

**Corollary 3.** $[\mathbb{Q}(\sqrt{2}, \sqrt{3}) : \mathbb{Q}] = 4$.

```
  ℚ(√2, √3)         degree 4 over ℚ
    /    \
  ℚ(√2)  ℚ(√3)      each degree 2 over ℚ
    \    /
      ℚ
```

---

## 5. Simple Extensions

**Definition.** $E/F$ is a **simple extension** if $E = F(\alpha)$ for some $\alpha \in E$.

$$F(\alpha) = \text{smallest field containing } F \text{ and } \alpha$$

Two cases:
1. $\alpha$ **algebraic** over $F$: $F(\alpha) \cong F[x]/(m_\alpha(x))$ where $m_\alpha$ is the minimal polynomial.
2. $\alpha$ **transcendental** over $F$: $F(\alpha) \cong F(x)$ (rational function field).

---

## 6. Constructing $F(\alpha)$ for Algebraic $\alpha$

If $\alpha$ is a root of an irreducible $f(x) \in F[x]$ of degree $n$:

$$F(\alpha) = F[x]/(f(x)) = \{a_0 + a_1\alpha + \cdots + a_{n-1}\alpha^{n-1} : a_i \in F\}$$

This is a field of degree $n$ over $F$ with basis $\{1, \alpha, \alpha^2, \ldots, \alpha^{n-1}\}$.

**Example:** $\mathbb{Q}(\sqrt{2}) = \mathbb{Q}[x]/(x^2 - 2) = \{a + b\sqrt{2} : a, b \in \mathbb{Q}\}$.

Arithmetic:
- $(a + b\sqrt{2}) + (c + d\sqrt{2}) = (a+c) + (b+d)\sqrt{2}$
- $(a + b\sqrt{2})(c + d\sqrt{2}) = (ac + 2bd) + (ad + bc)\sqrt{2}$
- $(a + b\sqrt{2})^{-1} = \frac{a - b\sqrt{2}}{a^2 - 2b^2}$

**Example:** $\mathbb{Q}[x]/(x^3 - 2) \cong \mathbb{Q}(\sqrt[3]{2}) = \{a + b\sqrt[3]{2} + c\sqrt[3]{4} : a, b, c \in \mathbb{Q}\}$, degree 3.

---

## 7. Compositum of Fields

**Definition.** For extensions $E_1/F$ and $E_2/F$ inside some common overfield $L$:

$$E_1 E_2 = E_1(E_2) = \text{smallest subfield of } L \text{ containing } E_1 \text{ and } E_2$$

**Degree inequality:** $[E_1 E_2 : F] \leq [E_1:F] \cdot [E_2:F]$, with equality when $\gcd([E_1:F], [E_2:F]) = 1$.

```
  E₁E₂ (compositum)
  /    \
 E₁    E₂
  \    /
    F
  
  [E₁E₂ : F] ≤ [E₁:F]·[E₂:F]
  Equality when gcd([E₁:F], [E₂:F]) = 1
```

---

## 8. Lattice of Subfields

**Example:** Subfields of $\mathbb{Q}(\sqrt{2}, \sqrt{3})$:

```
         ℚ(√2, √3)
        /    |    \
     ℚ(√2) ℚ(√3) ℚ(√6)
        \    |    /
           ℚ
  
  All intermediate extensions have degree 2 over ℚ.
  Each is ℚ(√d) for d = 2, 3, or 6.
  No others exist (by degree counting).
```

---

## Summary Table

| Concept | Definition / Result |
|---------|-------------------|
| Field extension $E/F$ | $F \subseteq E$, both fields |
| Degree $[E:F]$ | $\dim_F(E)$ |
| Tower law | $[E:F] = [E:K][K:F]$ |
| Simple extension | $E = F(\alpha)$ |
| Algebraic $\alpha$ | $F(\alpha) \cong F[x]/(m_\alpha)$, $[F(\alpha):F] = \deg m_\alpha$ |
| Transcendental $\alpha$ | $F(\alpha) \cong F(x)$, $[F(\alpha):F] = \infty$ |
| Compositum | $E_1 E_2 = $ smallest field containing both |
| Degree inequality | $[E_1E_2:F] \leq [E_1:F][E_2:F]$ |

---

## Quick Revision Questions

1. **Compute** $[\mathbb{Q}(\sqrt[4]{2}):\mathbb{Q}]$ and give a basis.

2. **Prove** the tower law for finite extensions.

3. **Show** that $[\mathbb{Q}(\sqrt{2}, \sqrt{3}):\mathbb{Q}] = 4$ using the tower law.

4. **Construct** $\mathbb{Q}(\sqrt[3]{2})$ as a quotient of $\mathbb{Q}[x]$ and describe its elements.

5. **Determine** whether there exists an intermediate field strictly between $\mathbb{Q}$ and $\mathbb{Q}(\sqrt[3]{5})$.

6. **State** the relationship between the degrees of a compositum and its constituent extensions.

---

[← Previous: Irreducibility Criteria](../07-Rings-Factorization/05-irreducibility-criteria.md) | [Back to README](../README.md) | [Next: Algebraic and Transcendental Extensions →](02-algebraic-and-transcendental.md)
