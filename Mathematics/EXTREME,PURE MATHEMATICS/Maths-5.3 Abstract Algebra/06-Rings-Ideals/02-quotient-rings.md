# Chapter 6.2 — Quotient Rings

[← Previous: Ideals](01-ideals.md) | [Back to README](../README.md) | [Next: Ring Homomorphisms →](03-ring-homomorphisms.md)

---

## Overview

Just as we form quotient groups $G/N$ using normal subgroups, we form **quotient rings** $R/I$ using ideals. The resulting structure is a ring, and the natural projection $\pi: R \to R/I$ is a surjective ring homomorphism.

---

## 1. Construction

Let $I \trianglelefteq R$. The **quotient ring** (or **factor ring**) is:

$$R/I = \{r + I : r \in R\}$$

with operations:

$$(a + I) + (b + I) = (a + b) + I$$
$$(a + I)(b + I) = ab + I$$

```
  Elements of R/I are cosets:  r + I = {r + a : a ∈ I}
  
  Two cosets are equal:  r + I = s + I  ⟺  r - s ∈ I
  
  Addition:        (a+I) + (b+I) = (a+b) + I
  Multiplication:  (a+I) · (b+I) = (ab) + I
  Zero:            0 + I = I
  Unity:           1 + I
```

---

## 2. Well-Definedness

**Why ideal is needed for multiplication:**

If $a + I = a' + I$ and $b + I = b' + I$, then $a - a' \in I$ and $b - b' \in I$.

$ab - a'b' = ab - a'b + a'b - a'b' = (a - a')b + a'(b - b')$

Since $I$ is an ideal: $(a-a')b \in I$ (absorption) and $a'(b-b') \in I$. So $ab - a'b' \in I$. ✓

**Note:** This is why we need ideals (not just subrings) for quotient rings—absorption ensures multiplication is well-defined.

---

## 3. Natural Projection

The **natural projection** $\pi: R \to R/I$ defined by $\pi(r) = r + I$ is:

- A surjective ring homomorphism
- $\ker \pi = I$

This connects to the First Isomorphism Theorem: $R/\ker\varphi \cong \text{im}\,\varphi$.

---

## 4. Examples

### 4.1. $\mathbb{Z}/n\mathbb{Z} = \mathbb{Z}_n$

$I = n\mathbb{Z}$. Cosets: $\{0 + I, 1 + I, \ldots, (n-1) + I\}$.

$\mathbb{Z}/n\mathbb{Z} \cong \mathbb{Z}_n$ — the ring we already know.

### 4.2. $\mathbb{R}[x]/(x^2 + 1) \cong \mathbb{C}$

$I = (x^2 + 1)$. Every polynomial mod $x^2 + 1$ reduces to $a + bx$ (since $x^2 \equiv -1$).

Elements: $\{a + bx + I : a, b \in \mathbb{R}\}$.

Multiplication: $(a + bx)(c + dx) = ac + (ad + bc)x + bdx^2 \equiv (ac - bd) + (ad + bc)x$.

This is exactly $\mathbb{C}$ with $x \leftrightarrow i$!

### 4.3. $\mathbb{Z}[x]/(x^2 + 1) \cong \mathbb{Z}[i]$

Gaussian integers: $a + bx + I \leftrightarrow a + bi$.

### 4.4. $F[x]/(f(x))$

For $F$ a field and $f(x)$ irreducible of degree $n$:

$F[x]/(f(x))$ is a field extension of $F$ of degree $n$.

Elements: $\{a_0 + a_1 x + \cdots + a_{n-1}x^{n-1} + I : a_i \in F\}$.

### 4.5. $\mathbb{Z}[x]/(x) \cong \mathbb{Z}$

Map: $f(x) + (x) \mapsto f(0)$ (evaluation at 0).

### 4.6. $R/(1) = \{0\}$

The quotient by the whole ring is the zero ring.

$R/(0) \cong R$ — quotient by zero ideal gives back $R$.

---

## 5. Properties of $R/I$

| Property of $R/I$ | Condition |
|-------------------|-----------|
| $R/I$ is commutative | $\iff R$ is commutative |
| $R/I$ is a field | $\iff I$ is maximal |
| $R/I$ is an integral domain | $\iff I$ is prime |
| $R/I = 0$ | $\iff I = R$ |
| $R/I \cong R$ | $\iff I = \{0\}$ |

These connections to prime and maximal ideals are explored in Chapter 6.4.

---

## 6. Computing in Quotient Rings

### Recipe for $R/I$

```
  To compute in R/I:
  
  1. Write elements as coset representatives
  2. Do arithmetic in R
  3. Reduce modulo I  (replace anything in I by 0)
  
  For R[x]/(f(x)):
  1. Write elements as polynomials of deg < deg(f)
  2. Multiply normally
  3. Divide by f(x), keep remainder
```

### Example: $\mathbb{Z}_3[x]/(x^2 + 1)$

Is $x^2 + 1$ irreducible over $\mathbb{F}_3$? Check: $0^2 + 1 = 1 \neq 0$, $1^2 + 1 = 2 \neq 0$, $2^2 + 1 = 5 = 2 \neq 0$. No roots, so yes (degree 2).

$\mathbb{F}_3[x]/(x^2 + 1) \cong \mathbb{F}_9$. Elements: $\{a + b\alpha : a, b \in \mathbb{F}_3\}$, $\alpha^2 = -1 = 2$.

$|\mathbb{F}_9| = 9$, $\mathbb{F}_9^\times$ is cyclic of order 8. Check: $|\alpha| = ?$

$\alpha^2 = 2$, $\alpha^4 = 4 = 1$. So $|\alpha| = 4$, not 8. Try $\alpha + 1$:

$(\alpha + 1)^2 = \alpha^2 + 2\alpha + 1 = 2 + 2\alpha + 1 = 2\alpha$
$(\alpha + 1)^4 = (2\alpha)^2 = 4\alpha^2 = 1 \cdot 2 = 2 \neq 1$
$(\alpha + 1)^8 = 2^2 = 4 = 1$. So $|\alpha + 1| = 8$. ✓ Primitive element!

---

## 7. Ideals of $R/I$

**Theorem 6.3 (Correspondence / Lattice Theorem).** The ideals of $R/I$ are in bijection with ideals of $R$ containing $I$:

$$\{J : I \subseteq J \trianglelefteq R\} \xrightarrow{\sim} \{\text{ideals of } R/I\}$$
$$J \mapsto J/I$$

This is the ring analogue of the correspondence theorem for groups.

```
  Ideals of R containing I  ←→  Ideals of R/I
  
  R        ←→  R/I
  |              |
  J        ←→  J/I
  |              |
  I        ←→  {0}
```

---

## 8. Chinese Remainder Theorem (Ring Version)

**Theorem 6.4 (CRT).** If $I, J$ are ideals of $R$ with $I + J = R$ (coprime), then:

$$R/(I \cap J) \cong R/I \times R/J$$

For $R = \mathbb{Z}$, $I = m\mathbb{Z}$, $J = n\mathbb{Z}$ with $\gcd(m,n) = 1$:

$$\mathbb{Z}/mn\mathbb{Z} \cong \mathbb{Z}/m\mathbb{Z} \times \mathbb{Z}/n\mathbb{Z}$$

i.e., $\mathbb{Z}_{mn} \cong \mathbb{Z}_m \times \mathbb{Z}_n$ when $\gcd(m,n) = 1$.

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| Quotient ring $R/I$ | Cosets $r + I$ with well-defined $+, \cdot$ |
| Well-defined | Requires $I$ to be an ideal (absorption) |
| Natural map | $\pi: R \to R/I$; surjective, $\ker\pi = I$ |
| $R/I$ is a field | $\iff I$ is maximal |
| $R/I$ is a domain | $\iff I$ is prime |
| Correspondence | Ideals of $R/I \leftrightarrow$ ideals of $R$ containing $I$ |
| CRT | $I + J = R \implies R/(I \cap J) \cong R/I \times R/J$ |

---

## Quick Revision Questions

1. **Construct** $\mathbb{F}_4 = \mathbb{F}_2[x]/(x^2 + x + 1)$ and verify it is a field.

2. **Prove** that the natural map $\pi: R \to R/I$ is a ring homomorphism.

3. **Show** why $R/I$ multiplication is not well-defined if $I$ is merely a subring (not an ideal).

4. **Compute** $(\mathbb{Z} \times \mathbb{Z})/\langle(2,3)\rangle$. What is its structure?

5. **Apply** CRT to show $\mathbb{Z}_{30} \cong \mathbb{Z}_2 \times \mathbb{Z}_3 \times \mathbb{Z}_5$.

6. **Find** all ideals of $\mathbb{Z}[x]/(x^2, 2)$.

---

[← Previous: Ideals](01-ideals.md) | [Back to README](../README.md) | [Next: Ring Homomorphisms →](03-ring-homomorphisms.md)
