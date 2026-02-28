# Chapter 6.3 — Ring Homomorphisms

[← Previous: Quotient Rings](02-quotient-rings.md) | [Back to README](../README.md) | [Next: Prime and Maximal Ideals →](04-prime-and-maximal-ideals.md)

---

## Overview

**Ring homomorphisms** are structure-preserving maps between rings, analogous to group homomorphisms. The First Isomorphism Theorem for rings parallels the group version: $R/\ker\varphi \cong \text{im}\,\varphi$.

---

## 1. Definition

**Definition.** A **ring homomorphism** is a function $\varphi: R \to S$ satisfying:

1. $\varphi(a + b) = \varphi(a) + \varphi(b)$
2. $\varphi(ab) = \varphi(a)\varphi(b)$
3. $\varphi(1_R) = 1_S$

```
  Ring homomorphism  φ: R → S
  
  (H1) φ(a + b) = φ(a) + φ(b)      preserves +
  (H2) φ(ab)    = φ(a) · φ(b)      preserves ·
  (H3) φ(1_R)   = 1_S               preserves unity
  
  Note: (H3) does NOT follow from (H1)+(H2) in general!
  Example: φ: Z → Z×Z, φ(n) = (n,0). Then φ(1) = (1,0) ≠ (1,1).
```

---

## 2. Basic Properties

**Proposition 6.5.** If $\varphi: R \to S$ is a ring homomorphism:

1. $\varphi(0_R) = 0_S$
2. $\varphi(-a) = -\varphi(a)$
3. $\varphi(na) = n\varphi(a)$ for all $n \in \mathbb{Z}$
4. If $a$ is a unit, $\varphi(a)$ is a unit with $\varphi(a^{-1}) = \varphi(a)^{-1}$
5. $\text{im}\,\varphi$ is a subring of $S$
6. $\ker\varphi$ is an ideal of $R$

---

## 3. Kernel and Image

**Kernel:** $\ker\varphi = \{r \in R : \varphi(r) = 0_S\}$ — always an ideal of $R$.

**Image:** $\text{im}\,\varphi = \{\varphi(r) : r \in R\}$ — always a subring of $S$.

**Theorem 6.6.** $\varphi$ is injective $\iff \ker\varphi = \{0\}$.

---

## 4. Examples

| Homomorphism | $\varphi$ | $\ker\varphi$ |
|-------------|-----------|---------------|
| $\mathbb{Z} \to \mathbb{Z}_n$, $a \mapsto \bar{a}$ | Reduction mod $n$ | $n\mathbb{Z}$ |
| $R[x] \to R$, $f \mapsto f(a)$ | Evaluation at $a$ | $(x - a)$ |
| $\mathbb{Z}[i] \to \mathbb{Z}_5$, $a+bi \mapsto \overline{a+2b}$ | (since $i^2 \equiv -1 \equiv 4 \equiv 2^2$) | ... |
| $M_n(R) \to R$, $A \mapsto \text{tr}(A)$ | **NOT** a homomorphism ($\text{tr}(AB) \neq \text{tr}(A)\text{tr}(B)$) | — |
| $R \to R/I$, $r \mapsto r + I$ | Natural projection | $I$ |
| $R \to R \times S$, $r \mapsto (r, 0)$ | **NOT** a homomorphism ($1 \mapsto (1,0) \neq (1,1)$) | — |
| $\det: M_n(R) \to R$ | Ring homomorphism? No: $\det(A+B) \neq \det A + \det B$ | — |

### Valid Examples

1. **Reduction mod $n$:** $\phi: \mathbb{Z} \to \mathbb{Z}_n$, $\phi(a) = a \bmod n$.
2. **Evaluation:** $\text{ev}_a: R[x] \to R$, $\text{ev}_a(f) = f(a)$. Kernel $= (x - a)$.
3. **Complex conjugation:** $\bar{\phantom{z}}: \mathbb{C} \to \mathbb{C}$, $\overline{a+bi} = a - bi$. Kernel $= \{0\}$ (isomorphism).
4. **Frobenius:** $\sigma: R \to R$, $\sigma(a) = a^p$ (char $p$, commutative).

---

## 5. First Isomorphism Theorem for Rings

**Theorem 6.7 (FIT).** If $\varphi: R \to S$ is a ring homomorphism, then:

$$R/\ker\varphi \cong \text{im}\,\varphi$$

```
  First Isomorphism Theorem:
  
       φ
  R -------→ S
  |           ↑
  π↓    ≅    /  inclusion
  |        /
  R/ker φ
  
  R/ker(φ) ≅ im(φ)
  
  via  r + ker(φ)  ↦  φ(r)
```

### Applications

| Homomorphism | FIT gives |
|-------------|-----------|
| $\mathbb{Z} \to \mathbb{Z}_n$ | $\mathbb{Z}/n\mathbb{Z} \cong \mathbb{Z}_n$ |
| $\mathbb{R}[x] \xrightarrow{\text{ev}_a} \mathbb{R}$ | $\mathbb{R}[x]/(x-a) \cong \mathbb{R}$ |
| $\mathbb{R}[x] \xrightarrow{\text{ev}_i} \mathbb{C}$ | $\mathbb{R}[x]/(x^2+1) \cong \mathbb{C}$ |
| $\mathbb{Z}[x] \xrightarrow{\text{ev}_i} \mathbb{Z}[i]$ | $\mathbb{Z}[x]/(x^2+1) \cong \mathbb{Z}[i]$ |

---

## 6. Second and Third Isomorphism Theorems

**Theorem 6.8 (Second Isomorphism Theorem).** If $S$ is a subring and $I$ is an ideal of $R$, then:

$$S/(S \cap I) \cong (S + I)/I$$

**Theorem 6.9 (Third Isomorphism Theorem).** If $I \subseteq J$ are both ideals of $R$, then $J/I$ is an ideal of $R/I$ and:

$$(R/I)/(J/I) \cong R/J$$

---

## 7. Types of Ring Homomorphisms

| Type | Definition |
|------|-----------|
| **Monomorphism** | Injective homomorphism |
| **Epimorphism** | Surjective homomorphism |
| **Isomorphism** | Bijective homomorphism |
| **Endomorphism** | Homomorphism $R \to R$ |
| **Automorphism** | Isomorphism $R \to R$ |

**Theorem 6.10.** If $\varphi: R \to S$ is an isomorphism, then $\varphi^{-1}: S \to R$ is also an isomorphism.

### Ring Automorphisms

- $\text{Aut}(\mathbb{C}/\mathbb{R}) = \{\text{id}, \bar{\phantom{z}}\} \cong \mathbb{Z}_2$ (identity and conjugation).
- $\text{Aut}(\mathbb{Q}) = \{\text{id}\}$ (the only automorphism of $\mathbb{Q}$ is the identity).
- $\text{Aut}(\mathbb{Q}(\sqrt{2})) = \{\text{id}, \sigma\}$ where $\sigma(a + b\sqrt{2}) = a - b\sqrt{2}$.

---

## 8. Ring Isomorphism Invariants

If $R \cong S$, then:

| Invariant | Preserved |
|-----------|-----------|
| $\|R\| = \|S\|$ | Cardinality |
| $\text{char}(R) = \text{char}(S)$ | Characteristic |
| $R$ commutative $\iff S$ commutative | Commutativity |
| $R$ domain $\iff S$ domain | No zero divisors |
| $R$ field $\iff S$ field | Every non-zero invertible |
| $\|R^\times\| = \|S^\times\|$ | Number of units |
| Ideal lattices isomorphic | Ideal structure |

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Ring homomorphism | Preserves $+$, $\cdot$, and $1$ |
| Kernel | $\ker\varphi$ is an ideal |
| Image | $\text{im}\,\varphi$ is a subring |
| Injective | $\iff \ker\varphi = \{0\}$ |
| FIT | $R/\ker\varphi \cong \text{im}\,\varphi$ |
| 2nd IT | $S/(S \cap I) \cong (S+I)/I$ |
| 3rd IT | $(R/I)/(J/I) \cong R/J$ |

---

## Quick Revision Questions

1. **Verify** that $\text{ev}_0: \mathbb{R}[x] \to \mathbb{R}$ is a ring homomorphism and find its kernel.

2. **Apply** the FIT to prove $\mathbb{R}[x]/(x^2 + 1) \cong \mathbb{C}$.

3. **Show** that $\det: GL_n(\mathbb{R}) \to \mathbb{R}^\times$ is a group homomorphism but NOT a ring homomorphism on $M_n(\mathbb{R})$.

4. **Prove** that $\varphi: \mathbb{Z}[i] \to \mathbb{Z}_5$ defined by $\varphi(a + bi) = \overline{a + 2b}$ is a ring homomorphism.

5. **Use** the Third Isomorphism Theorem to show $(\mathbb{Z}/12\mathbb{Z})/(4\mathbb{Z}/12\mathbb{Z}) \cong \mathbb{Z}/4\mathbb{Z}$.

6. **Find** all ring homomorphisms $\mathbb{Z}_6 \to \mathbb{Z}_{15}$.

---

[← Previous: Quotient Rings](02-quotient-rings.md) | [Back to README](../README.md) | [Next: Prime and Maximal Ideals →](04-prime-and-maximal-ideals.md)
