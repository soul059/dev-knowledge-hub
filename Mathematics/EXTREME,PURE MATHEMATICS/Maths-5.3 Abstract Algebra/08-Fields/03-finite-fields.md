# Chapter 8.3 — Finite Fields

[← Previous: Algebraic and Transcendental Extensions](02-algebraic-and-transcendental.md) | [Back to README](../README.md) | [Next: Splitting Fields →](04-splitting-fields.md)

---

## Overview

Finite fields are completely classified: for every prime power $q = p^n$, there exists a unique (up to isomorphism) field $\mathbb{F}_q$ of order $q$, and **no other finite fields exist**. This chapter proves existence and uniqueness, describes the structure, and studies subfield lattices.

---

## 1. Classification Theorem

**Theorem 8.8.** 
1. Every finite field has order $p^n$ for some prime $p$ and integer $n \geq 1$.
2. For every prime power $p^n$, there exists a field of that order.
3. Any two fields of order $p^n$ are isomorphic.

We denote this unique field $\mathbb{F}_{p^n}$ (or $\text{GF}(p^n)$).

```
  Finite fields exist iff order = prime power:
  
  Order:  2  3  4  5  7  8  9  11  13  16  25  27 ...
  Field?  ✓  ✓  ✓  ✓  ✓  ✓  ✓   ✓   ✓   ✓   ✓   ✓
  
  Order:  6  10  12  14  15  18  20  21  22 ...
  Field?  ✗   ✗   ✗   ✗   ✗   ✗   ✗   ✗   ✗
  
  No field of order 6, 10, 12, etc.
```

---

## 2. Existence Proof

**Theorem 8.9.** $\mathbb{F}_{p^n}$ exists as the splitting field of $x^{p^n} - x$ over $\mathbb{F}_p$.

*Proof.* Let $f(x) = x^{p^n} - x \in \mathbb{F}_p[x]$. Let $E$ be the splitting field of $f$ over $\mathbb{F}_p$. We show $S = \{\alpha \in E : \alpha^{p^n} = \alpha\}$ is a field of order $p^n$.

1. **$S$ is a subfield:** $0, 1 \in S$. If $\alpha, \beta \in S$: $(\alpha + \beta)^{p^n} = \alpha^{p^n} + \beta^{p^n} = \alpha + \beta$ (Frobenius!). $(\alpha\beta)^{p^n} = \alpha^{p^n}\beta^{p^n} = \alpha\beta$. $(\alpha^{-1})^{p^n} = (\alpha^{p^n})^{-1} = \alpha^{-1}$.

2. **$|S| = p^n$:** $f'(x) = p^n x^{p^n - 1} - 1 = -1$ (in char $p$), so $\gcd(f, f') = 1$, meaning $f$ has no repeated roots. Thus $f$ has exactly $p^n$ roots in $E$.

3. **$E = S$:** since $f$ splits completely in $S$.  $\blacksquare$

---

## 3. Uniqueness

**Theorem 8.10.** Any field of order $p^n$ is isomorphic to $\mathbb{F}_{p^n}$.

*Proof.* Let $F$ be a field with $|F| = p^n$. Then $F^\times$ is a group of order $p^n - 1$, so every $\alpha \in F^\times$ satisfies $\alpha^{p^n - 1} = 1$, hence $\alpha^{p^n} = \alpha$. Including $0$: every element of $F$ is a root of $x^{p^n} - x$. So $F$ is a splitting field of $x^{p^n} - x$ over $\mathbb{F}_p$. Splitting fields are unique up to isomorphism. $\blacksquare$

---

## 4. Multiplicative Group

**Theorem 8.11.** $\mathbb{F}_q^\times = \mathbb{F}_q \setminus \{0\}$ is a **cyclic group** of order $q - 1$.

A generator of $\mathbb{F}_q^\times$ is called a **primitive element** (or primitive root).

**Example:** $\mathbb{F}_7^\times = \langle 3 \rangle$:

| $k$ | 0 | 1 | 2 | 3 | 4 | 5 |
|-----|---|---|---|---|---|---|
| $3^k \bmod 7$ | 1 | 3 | 2 | 6 | 4 | 5 |

Every non-zero element is a power of 3. $\checkmark$

---

## 5. Constructing $\mathbb{F}_{p^n}$ Explicitly

$$\mathbb{F}_{p^n} \cong \mathbb{F}_p[x] / (f(x))$$

where $f$ is any irreducible polynomial of degree $n$ over $\mathbb{F}_p$.

### Example: $\mathbb{F}_4$

$\mathbb{F}_4 = \mathbb{F}_2[x]/(x^2 + x + 1)$. Let $\alpha$ be a root. Elements: $\{0, 1, \alpha, \alpha + 1\}$.

| $+$ | 0 | 1 | $\alpha$ | $\alpha+1$ |
|-----|---|---|---------|-----------|
| 0 | 0 | 1 | $\alpha$ | $\alpha+1$ |
| 1 | 1 | 0 | $\alpha+1$ | $\alpha$ |
| $\alpha$ | $\alpha$ | $\alpha+1$ | 0 | 1 |
| $\alpha+1$ | $\alpha+1$ | $\alpha$ | 1 | 0 |

| $\times$ | 0 | 1 | $\alpha$ | $\alpha+1$ |
|----------|---|---|---------|-----------|
| 0 | 0 | 0 | 0 | 0 |
| 1 | 0 | 1 | $\alpha$ | $\alpha+1$ |
| $\alpha$ | 0 | $\alpha$ | $\alpha+1$ | 1 |
| $\alpha+1$ | 0 | $\alpha+1$ | 1 | $\alpha$ |

$\mathbb{F}_4^\times = \langle \alpha \rangle \cong \mathbb{Z}/3\mathbb{Z}$.

### Example: $\mathbb{F}_8$

$\mathbb{F}_8 = \mathbb{F}_2[x]/(x^3 + x + 1)$. Elements: $\{a + b\alpha + c\alpha^2 : a, b, c \in \mathbb{F}_2\}$, $|\mathbb{F}_8| = 8$.

$\mathbb{F}_8^\times \cong \mathbb{Z}/7\mathbb{Z}$ (cyclic of order 7).

### Example: $\mathbb{F}_9$

$\mathbb{F}_9 = \mathbb{F}_3[x]/(x^2 + 1)$ (since $x^2 + 1$ is irreducible over $\mathbb{F}_3$: $(\pm 1)^2 + 1 = 2 \neq 0$).

Elements: $\{a + bi : a, b \in \mathbb{F}_3\}$ where $i^2 = -1$. Like "Gaussian integers mod 3".

$\mathbb{F}_9^\times \cong \mathbb{Z}/8\mathbb{Z}$.

---

## 6. Subfield Lattice

**Theorem 8.12.** The subfields of $\mathbb{F}_{p^n}$ are exactly $\mathbb{F}_{p^d}$ for $d \mid n$.

Moreover, $\mathbb{F}_{p^d} \subseteq \mathbb{F}_{p^e} \iff d \mid e$.

```
  Subfield lattice of 𝔽_{p¹²}:
  
              𝔽_{p¹²}
            /    |    \
        𝔽_{p⁶}  𝔽_{p⁴}  (no 𝔽_{p⁸} — 8 ∤ 12)
        /   \    |
    𝔽_{p³}  𝔽_{p²}
        \   /
         𝔽_p
  
  Subfields correspond to divisors of 12: {1,2,3,4,6,12}
  
  
  Divisor lattice of 12:
  
         12
        / | \
       6  4  (not 8)
      / \  |
     3   2
      \ /
       1
```

---

## 7. Frobenius Automorphism

**Definition.** The **Frobenius automorphism** of $\mathbb{F}_{p^n}$ is:

$$\phi: \mathbb{F}_{p^n} \to \mathbb{F}_{p^n}, \quad \alpha \mapsto \alpha^p$$

**Theorem 8.13.** 
- $\phi$ is a field automorphism fixing $\mathbb{F}_p$.
- $\text{ord}(\phi) = n$ in $\text{Aut}(\mathbb{F}_{p^n}/\mathbb{F}_p)$.
- $\text{Aut}(\mathbb{F}_{p^n}/\mathbb{F}_p) = \langle \phi \rangle \cong \mathbb{Z}/n\mathbb{Z}$.

The fixed field of $\phi^k$ is $\mathbb{F}_{p^d}$ where $d = \gcd(k, n)$.

---

## 8. Counting Irreducible Polynomials

The number of monic irreducible polynomials of degree $n$ over $\mathbb{F}_p$ is:

$$N_p(n) = \frac{1}{n} \sum_{d \mid n} \mu(n/d) \, p^d$$

| $n$ | $N_2(n)$ | $N_3(n)$ | $N_5(n)$ |
|-----|---------|---------|---------|
| 1 | 2 | 3 | 5 |
| 2 | 1 | 3 | 10 |
| 3 | 2 | 8 | 40 |
| 4 | 3 | 18 | 150 |
| 5 | 6 | 48 | 624 |
| 6 | 9 | 116 | 2580 |

Since $N_p(n) \geq 1$ for all $p, n$, irreducible polynomials of every degree exist — another proof of the existence of $\mathbb{F}_{p^n}$.

---

## Summary Table

| Concept | Definition / Result |
|---------|-------------------|
| $\mathbb{F}_{p^n}$ | Unique field of order $p^n$ |
| Existence | Splitting field of $x^{p^n} - x$ over $\mathbb{F}_p$ |
| Uniqueness | Any two fields of same finite order are isomorphic |
| $\mathbb{F}_q^\times$ | Cyclic group of order $q - 1$ |
| Construction | $\mathbb{F}_p[x]/(f)$ for $f$ irreducible of degree $n$ |
| Subfields | $\mathbb{F}_{p^d} \subseteq \mathbb{F}_{p^n} \iff d \mid n$ |
| Frobenius | $\phi(\alpha) = \alpha^p$; $\text{Aut}(\mathbb{F}_{p^n}/\mathbb{F}_p) = \langle\phi\rangle \cong \mathbb{Z}/n\mathbb{Z}$ |

---

## Quick Revision Questions

1. **Prove** that every finite field has prime-power order.

2. **Construct** $\mathbb{F}_{16}$ explicitly as $\mathbb{F}_2[x]/(f)$ for an irreducible $f$ of degree 4, and list all elements.

3. **Find** a generator of $\mathbb{F}_{11}^\times$ and verify that it generates all non-zero elements.

4. **List** all subfields of $\mathbb{F}_{2^{12}}$ and draw the inclusion lattice.

5. **Determine** the minimal polynomial of a primitive element of $\mathbb{F}_8$ over $\mathbb{F}_2$.

6. **Compute** the number of irreducible monic polynomials of degree 6 over $\mathbb{F}_2$.

---

[← Previous: Algebraic and Transcendental Extensions](02-algebraic-and-transcendental.md) | [Back to README](../README.md) | [Next: Splitting Fields →](04-splitting-fields.md)
