# 8.3 Polynomial Spaces

[← Previous: Function Spaces](02-function-spaces.md) · [Back to README](../README.md) · [Next: Isomorphisms →](04-isomorphisms.md)

---

## Chapter Overview

Polynomial spaces are the most concrete infinite family of finite-dimensional vector spaces beyond $\mathbb{R}^n$. They serve as an ideal testing ground for abstract concepts — basis, dimension, linear transformations, eigenvalues — because every computation can be done explicitly.

---

## 1. $\mathcal{P}_n(\mathbb{R})$ — Polynomials of Degree $\leq n$

$$\mathcal{P}_n = \{a_0 + a_1 x + a_2 x^2 + \dots + a_n x^n \mid a_i \in \mathbb{R}\}$$

**Standard basis:** $\mathcal{B}_{\text{std}} = \{1, x, x^2, \dots, x^n\}$

**Dimension:** $n + 1$

**Zero vector:** $p(x) = 0$ (the zero polynomial)

| Space | Elements | Dimension | Standard Basis |
|-------|----------|:---------:|----------------|
| $\mathcal{P}_0$ | $\{a_0\}$ | 1 | $\{1\}$ |
| $\mathcal{P}_1$ | $\{a_0 + a_1 x\}$ | 2 | $\{1, x\}$ |
| $\mathcal{P}_2$ | $\{a_0 + a_1 x + a_2 x^2\}$ | 3 | $\{1, x, x^2\}$ |
| $\mathcal{P}_n$ | degree $\leq n$ | $n+1$ | $\{1, x, \dots, x^n\}$ |

> **Warning:** $\mathcal{P}_n$ includes the zero polynomial (which has no well-defined degree). The constraint is degree $\leq n$, **not** degree $= n$.

---

## 2. $\mathcal{P}(\mathbb{R})$ — All Polynomials

$$\mathcal{P} = \bigcup_{n=0}^{\infty} \mathcal{P}_n = \{a_0 + a_1 x + \dots + a_n x^n \mid n \in \mathbb{N},\, a_i \in \mathbb{R}\}$$

- Each polynomial has **finite** degree, but there is **no upper bound**.
- $\dim(\mathcal{P}) = \infty$ (countably infinite).
- Basis: $\{1, x, x^2, x^3, \dots\}$ (Hamel basis).

---

## 3. Alternative Bases for $\mathcal{P}_n$

The standard basis $\{1, x, x^2, \dots\}$ is not the only choice.

### 3.1 Shifted Monomials

**Basis for $\mathcal{P}_2$ centred at $a$:**

$$\{1,\; (x - a),\; (x - a)^2\}$$

This gives the **Taylor expansion** centred at $a$:

$$p(x) = p(a) + p'(a)(x-a) + \frac{p''(a)}{2}(x-a)^2$$

### 3.2 Lagrange Basis

Given distinct points $x_0, x_1, \dots, x_n$:

$$L_k(x) = \prod_{\substack{j=0 \\ j \neq k}}^{n} \frac{x - x_j}{x_k - x_j}$$

**Property:** $L_k(x_j) = \delta_{kj}$ (equals 1 at $x_k$, zero at all other nodes)

**Interpolation:** $p(x) = \sum_{k=0}^{n} y_k L_k(x)$ passes through $(x_k, y_k)$.

### 3.3 Bernstein Basis

$$b_{k,n}(x) = \binom{n}{k} x^k (1-x)^{n-k}, \quad k = 0, 1, \dots, n$$

Used in Bézier curves and computer graphics.

```
  Alternative bases for P₂:

  Standard         Shifted (a=1)      Lagrange (nodes 0,1,2)
  {1, x, x²}      {1, x-1, (x-1)²}  {L₀, L₁, L₂}
     │                  │                   │
     ▼                  ▼                   ▼
  [3, -2, 5]       [6, 8, 5]          [3, 6, 23]
  coefficients     Taylor coeffs      function values
  
  Same polynomial, different coordinate representations!
```

---

## 4. Differentiation as a Linear Map

### 4.1 $D: \mathcal{P}_n \to \mathcal{P}_{n-1}$

$$D(p) = p', \quad D(a_0 + a_1 x + \dots + a_n x^n) = a_1 + 2a_2 x + \dots + na_n x^{n-1}$$

**Linearity:** $D(\alpha p + \beta q) = \alpha D(p) + \beta D(q)$ ✓

**Matrix representation** (standard bases for $\mathcal{P}_3 \to \mathcal{P}_2$):

$$[D]_{\mathcal{B}_3 \to \mathcal{B}_2} = \begin{pmatrix} 0 & 1 & 0 & 0 \\ 0 & 0 & 2 & 0 \\ 0 & 0 & 0 & 3 \end{pmatrix}$$

**Properties of $D$ on $\mathcal{P}_n$:**

| Property | Value |
|----------|-------|
| $\ker(D)$ | $\mathcal{P}_0$ (constants) |
| $\text{nullity}(D)$ | 1 |
| $\text{Im}(D)$ | $\mathcal{P}_{n-1}$ |
| $\text{rank}(D)$ | $n$ |
| Surjective? | Yes (onto $\mathcal{P}_{n-1}$) |
| Injective? | No |

### 4.2 $D: \mathcal{P}_n \to \mathcal{P}_n$ (Same Domain and Codomain)

Matrix representation (standard basis for $\mathcal{P}_2$):

$$[D] = \begin{pmatrix} 0 & 1 & 0 \\ 0 & 0 & 2 \\ 0 & 0 & 0 \end{pmatrix}$$

**Eigenvalues:** $\lambda = 0$ (only eigenvalue, algebraic multiplicity 3)

**$D$ is nilpotent:** $D^{n+1} = 0$ on $\mathcal{P}_n$.

$$D^2(a + bx + cx^2) = D(b + 2cx) = 2c, \quad D^3 = 0$$

---

## 5. Integration as a Linear Map

$$J: \mathcal{P}_n \to \mathcal{P}_{n+1}, \quad J(p)(x) = \int_0^x p(t)\, dt$$

Matrix for $J: \mathcal{P}_2 \to \mathcal{P}_3$ (standard bases):

$$[J] = \begin{pmatrix} 0 & 0 & 0 \\ 1 & 0 & 0 \\ 0 & \frac{1}{2} & 0 \\ 0 & 0 & \frac{1}{3} \end{pmatrix}$$

**Properties:**

| Property | Value |
|----------|-------|
| $\ker(J)$ | $\{0\}$ |
| Injective? | Yes |
| Surjective? | No (range is polynomials with $p(0) = 0$) |

**Relationship:** $D \circ J = I$ (identity on $\mathcal{P}_n$), but $J \circ D \neq I$ (loses the constant).

---

## 6. Evaluation Map

$$\text{ev}_a : \mathcal{P}_n \to \mathbb{R}, \quad \text{ev}_a(p) = p(a)$$

**This is a linear transformation** (into $\mathbb{R}$ viewed as a 1-D vector space).

**Matrix (row vector):** For $\mathcal{P}_2$ with standard basis:

$$[\text{ev}_a] = \begin{pmatrix} 1 & a & a^2 \end{pmatrix}$$

**Kernel:** $\ker(\text{ev}_a) = \{p \in \mathcal{P}_n : p(a) = 0\}$ = polynomials with root $a$.

$\dim(\ker(\text{ev}_a)) = n$ (codimension 1 by Rank-Nullity).

---

## 7. Inner Products on Polynomial Spaces

### 7.1 Standard $L^2$ Inner Product

$$\langle p, q \rangle = \int_{-1}^{1} p(x) q(x)\, dx$$

Applying Gram–Schmidt to $\{1, x, x^2, \dots\}$ yields the **Legendre polynomials:**

$$P_0 = 1, \quad P_1 = x, \quad P_2 = \frac{1}{2}(3x^2 - 1), \quad P_3 = \frac{1}{2}(5x^3 - 3x)$$

### 7.2 Coefficient Inner Product

$$\langle p, q \rangle = \sum_{k=0}^{n} a_k b_k \quad \text{where } p = \sum a_k x^k,\; q = \sum b_k x^k$$

This makes $\mathcal{P}_n$ isometric to $\mathbb{R}^{n+1}$ with the dot product.

---

## 8. Worked Example — Complete Analysis

**Problem:** Analyse $T: \mathcal{P}_2 \to \mathcal{P}_2$ defined by $T(p)(x) = p(x) + xp'(x)$.

**Step 1: Apply to basis.**

$$T(1) = 1, \quad T(x) = x + x \cdot 1 = 2x, \quad T(x^2) = x^2 + x \cdot 2x = 3x^2$$

**Step 2: Matrix.**

$$[T] = \begin{pmatrix} 1 & 0 & 0 \\ 0 & 2 & 0 \\ 0 & 0 & 3 \end{pmatrix}$$

**Step 3: Eigenvalues.** $\lambda_1 = 1, \lambda_2 = 2, \lambda_3 = 3$ (read from diagonal).

**Step 4: Eigenvectors.** $1 \mapsto \{1\}$, $x \mapsto \{x\}$, $x^2 \mapsto \{x^2\}$.

**Step 5: $T$ is diagonalisable** (already diagonal in the standard basis).

**Step 6: $T^k$.**

$$T^k(p) = a_0 \cdot 1^k + a_1 \cdot 2^k x + a_2 \cdot 3^k x^2$$

---

## Summary Table

| Concept | Key Result |
|---------|-----------|
| $\mathcal{P}_n$ | $\dim = n+1$; basis $\{1, x, \dots, x^n\}$ |
| Alternative bases | Lagrange, Bernstein, shifted monomials |
| Differentiation $D$ | Nilpotent, $\ker = $ constants, rank $= n$ |
| Integration $J$ | Injective, $D \circ J = I$ |
| Evaluation $\text{ev}_a$ | Linear functional (rank 1) |
| $L^2$ inner product | Gram–Schmidt → Legendre polynomials |
| $T(p) = p + xp'$ | Diagonal with eigenvalues $1, 2, \dots, n+1$ |

---

## Quick Revision Questions

1. Show that $\{1, 1+x, 1+x+x^2\}$ is a basis for $\mathcal{P}_2$.

2. Write the matrix of $D: \mathcal{P}_2 \to \mathcal{P}_2$ with respect to the basis $\{1, 1+x, 1+x+x^2\}$.

3. Find the eigenvalues and eigenvectors of the map $T: \mathcal{P}_1 \to \mathcal{P}_1$ given by $T(p)(x) = p(1-x)$.

4. Verify that $P_0 = 1$ and $P_1 = x$ are orthogonal on $[-1, 1]$ with $\langle f, g \rangle = \int_{-1}^{1} fg\, dx$.

5. Find $\ker(\text{ev}_2)$ in $\mathcal{P}_2$ and state its dimension.

6. Prove that $D^3 = 0$ on $\mathcal{P}_2$ (i.e., differentiation is nilpotent).

---

[← Previous: Function Spaces](02-function-spaces.md) · [Back to README](../README.md) · [Next: Isomorphisms →](04-isomorphisms.md)
