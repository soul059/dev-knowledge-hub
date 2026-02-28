# 5.4 Zeros of Analytic Functions

[← Previous: Classification of Singularities](03-classification-of-singularities.md) | [Next: Unit 6 — Residues at Poles →](../06-Residue-Theory/01-residues-at-poles.md)

---

## Chapter Overview

Zeros and poles are dual concepts. Understanding zeros — their orders, their isolation, and the profound constraints they impose — gives us the **Identity Theorem**, **counting zeros via integrals**, and a deep appreciation of how rigid analytic functions really are.

---

## 1. Order of a Zero

### 1.1 Definition

$z_0$ is a **zero of order $m$** (or **multiplicity $m$**) of $f$ if:

$$f(z_0) = f'(z_0) = \cdots = f^{(m-1)}(z_0) = 0, \quad f^{(m)}(z_0) \neq 0$$

Equivalently, the Taylor series about $z_0$ begins with the $(z-z_0)^m$ term:

$$f(z) = (z - z_0)^m g(z)$$

where $g$ is analytic and $g(z_0) \neq 0$.

### 1.2 Terminology

| Order | Name |
|-------|------|
| $m = 1$ | Simple zero |
| $m = 2$ | Double zero |
| $m = 3$ | Triple zero |
| $m \geq 1$ | Zero of order $m$ |

### 1.3 Zero–Pole Duality

If $f$ has a zero of order $m$ at $z_0$, then $1/f$ has a pole of order $m$ at $z_0$, and vice versa.

```
Zero-pole duality:

  f(z) = (z - z₀)^m · g(z)     ←→     1/f(z) = 1/[(z-z₀)^m · g(z)]
       zero of order m                     pole of order m
       g(z₀) ≠ 0                          1/g(z₀) ≠ 0
```

---

## 2. Isolation of Zeros

### 2.1 Theorem

> If $f$ is analytic and not identically zero in a domain $D$, then the zeros of $f$ are **isolated**: each zero $z_0$ has a neighborhood containing no other zeros.

**Proof:** Write $f(z) = (z-z_0)^m g(z)$ with $g(z_0) \neq 0$. By continuity of $g$, there exists $\delta > 0$ such that $g(z) \neq 0$ for $|z - z_0| < \delta$. So the only zero in this disk is $z_0$ itself. $\square$

### 2.2 Consequence

The zero set of a non-constant analytic function is **discrete** (no accumulation point in $D$).

Zeros can only accumulate at the boundary of the domain.

---

## 3. The Identity Theorem

### 3.1 Statement

> If $f$ and $g$ are analytic on a connected domain $D$ and $f(z_n) = g(z_n)$ for a sequence $\{z_n\}$ with an accumulation point in $D$, then $f \equiv g$ on $D$.

### 3.2 Equivalent Formulations

| Version | Statement |
|---------|-----------|
| **Zero version** | If $f$ is analytic on connected $D$ and $f$ vanishes on a set with a limit point in $D$, then $f \equiv 0$ |
| **Agreement version** | If $f = g$ on a set with a limit point in $D$, then $f \equiv g$ |
| **Open set version** | If $f \equiv 0$ on a nonempty open subset of connected $D$, then $f \equiv 0$ on $D$ |
| **Segment version** | If $f \equiv 0$ on a line segment in $D$, then $f \equiv 0$ |

### 3.3 Applications

1. **Analytic continuation is unique:** If $f_1$ on $D_1$ and $f_2$ on $D_2$ are both analytic continuations of $f$ from $D_1 \cap D_2$, and $D_1 \cup D_2$ is connected, then $f_1 = f_2$ on $D_1 \cap D_2$.

2. **Real identities extend to $\mathbb{C}$:** If $\sin^2 x + \cos^2 x = 1$ for $x \in \mathbb{R}$, then $\sin^2 z + \cos^2 z = 1$ for all $z \in \mathbb{C}$.

---

## 4. Counting Zeros: The Argument Principle (Preview)

The number of zeros (counted with multiplicity) of $f$ inside a contour $C$ is:

$$N = \frac{1}{2\pi i}\oint_C \frac{f'(z)}{f(z)}\,dz$$

(Full treatment in Unit 8.)

---

## 5. Zeros and Factorization

### 5.1 Polynomials

$$p(z) = a_n(z - z_1)^{m_1}(z - z_2)^{m_2}\cdots(z - z_k)^{m_k}$$

where $m_1 + m_2 + \cdots + m_k = n = \deg p$.

### 5.2 Finite Products (Rational Functions)

If $f = p/q$ is rational, zeros and poles completely determine $f$ up to a constant.

### 5.3 Weierstrass Factorization (Preview)

Any entire function with prescribed zeros $\{z_n\}$ can be written as:

$$f(z) = e^{g(z)} \prod_{n} E_n\left(\frac{z}{z_n}\right)$$

where $E_n$ are elementary factors. (Covered in advanced courses.)

---

## 6. Design Examples

### Example 1: Find the order of the zero of $f(z) = z^2 \sin z$ at $z = 0$.

$z^2$ has a zero of order 2 at $z = 0$.
$\sin z$ has a simple zero at $z = 0$.

$f(z) = z^2 \sin z = z^2 \cdot z \cdot g(z) = z^3 g(z)$, where $g(z) = \frac{\sin z}{z} \to 1 \neq 0$.

**Order: 3** (triple zero).

### Example 2: Find the order of the zero of $f(z) = e^z - 1 - z$ at $z = 0$.

$$e^z - 1 - z = \frac{z^2}{2} + \frac{z^3}{6} + \cdots = z^2\left(\frac{1}{2} + \frac{z}{6} + \cdots\right)$$

The leading term is $z^2/2$, and $g(0) = 1/2 \neq 0$.

**Order: 2** (double zero).

### Example 3: Use the Identity Theorem to prove $\sin(2z) = 2\sin z\cos z$ for all $z \in \mathbb{C}$.

Both sides are entire functions that agree on $\mathbb{R}$ (known trigonometric identity).

$\mathbb{R}$ has accumulation points in $\mathbb{C}$.

By the Identity Theorem, they agree everywhere in $\mathbb{C}$. $\square$

---

## 7. Real-World Applications

| Application | Role of Zeros |
|-------------|--------------|
| **Control theory** | Zeros of transfer function → frequency nulls |
| **Signal processing** | Zeros of $z$-transform → filter notches |
| **Quantum mechanics** | Nodes of wavefunctions = zeros |
| **Numerical analysis** | Root-finding algorithms (Newton's method in $\mathbb{C}$) |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Zero of order $m$ | $f(z) = (z-z_0)^m g(z)$, $g(z_0) \neq 0$ |
| Zero–pole duality | Zero of $f$ ↔ pole of $1/f$ |
| Isolation of zeros | Non-constant analytic $f$: zeros are isolated |
| Identity Theorem | $f = g$ on limit-set → $f = g$ everywhere |
| Uniqueness of Taylor series | Follows from Identity Theorem |
| Analytic continuation | Unique when it exists |
| Counting zeros | $N = \frac{1}{2\pi i}\oint \frac{f'}{f}\,dz$ |

---

## Quick Revision Questions

1. **Define** zero of order $m$ and give the factorization $f(z) = (z-z_0)^m g(z)$.

2. **Find** the order of the zero of $f(z) = 1 - \cos z$ at $z = 0$.

3. **State** the Identity Theorem and explain its significance.

4. **Prove** that a non-constant polynomial of degree $n$ has exactly $n$ zeros in $\mathbb{C}$ (counted with multiplicity).

5. **Why** does the Identity Theorem fail for real analysis? Give a counterexample.

6. **Apply** the Identity Theorem to show that $e^{z_1+z_2} = e^{z_1}e^{z_2}$ for all $z_1, z_2 \in \mathbb{C}$.

---

[← Previous: Classification of Singularities](03-classification-of-singularities.md) | [Next: Unit 6 — Residues at Poles →](../06-Residue-Theory/01-residues-at-poles.md)
