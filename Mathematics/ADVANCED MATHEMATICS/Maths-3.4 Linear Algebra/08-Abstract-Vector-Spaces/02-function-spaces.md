# 8.2 Function Spaces

[← Previous: General Vector Spaces](01-general-vector-spaces.md) · [Back to README](../README.md) · [Next: Polynomial Spaces →](03-polynomial-spaces.md)

---

## Chapter Overview

Functions can be added, scaled, and combined just like column vectors. The resulting **function spaces** are among the most important infinite-dimensional vector spaces. They underpin Fourier analysis, differential equations, quantum mechanics, and signal processing.

---

## 1. $\mathcal{F}(S, \mathbb{R})$ — All Functions from $S$ to $\mathbb{R}$

For any set $S$, define:

$$\mathcal{F}(S, \mathbb{R}) = \{ f : S \to \mathbb{R} \}$$

**Operations:**

$$(f + g)(x) = f(x) + g(x), \qquad (cf)(x) = c \cdot f(x)$$

**Zero vector:** $\mathbf{0}(x) = 0$ for all $x \in S$.

**This is always a vector space** (inheriting field properties pointwise).

**Dimension:**
- If $S$ is finite with $|S| = n$: $\dim = n$ (isomorphic to $\mathbb{R}^n$)
- If $S$ is infinite: $\dim = \infty$

---

## 2. Key Function Spaces — Hierarchy

```
  ┌──────────────────────────────────────────────┐
  │          F(ℝ, ℝ) — all functions              │
  │   ┌──────────────────────────────────────┐   │
  │   │    C(ℝ) — continuous functions        │   │
  │   │   ┌──────────────────────────────┐   │   │
  │   │   │  C¹(ℝ) — once differentiable │   │   │
  │   │   │  ┌──────────────────────┐    │   │   │
  │   │   │  │ C^∞(ℝ) — smooth      │    │   │   │
  │   │   │  │  ┌──────────────┐    │    │   │   │
  │   │   │  │  │ C^ω — analytic│    │    │   │   │
  │   │   │  │  │  ┌────────┐  │    │    │   │   │
  │   │   │  │  │  │ P(ℝ)   │  │    │    │   │   │
  │   │   │  │  │  │ polys  │  │    │    │   │   │
  │   │   │  │  │  └────────┘  │    │    │   │   │
  │   │   │  │  └──────────────┘    │    │   │   │
  │   │   │  └──────────────────────┘    │   │   │
  │   │   └──────────────────────────────┘   │   │
  │   └──────────────────────────────────────┘   │
  └──────────────────────────────────────────────┘
       Each layer is a subspace of the one above.
```

---

## 3. $C[a,b]$ — Continuous Functions on a Closed Interval

$$C[a,b] = \{ f : [a,b] \to \mathbb{R} \mid f \text{ is continuous} \}$$

**Why it's a subspace:** Sum and scalar multiple of continuous functions are continuous; the zero function is continuous.

**Dimension:** Infinite (contains $1, x, x^2, x^3, \dots$ — infinitely many linearly independent elements).

### Inner Product on $C[a,b]$

$$\langle f, g \rangle = \int_a^b f(x)\, g(x)\, dx$$

**Verification of axioms:**

| Axiom | Check |
|-------|-------|
| Symmetry | $\int f g = \int g f$ ✓ |
| Linearity | $\int (cf + g)h = c\int fh + \int gh$ ✓ |
| Positive definite | $\int f^2 \geq 0$; $= 0$ iff $f = 0$ (continuous) ✓ |

**Induced norm:**

$$\|f\| = \sqrt{\int_a^b [f(x)]^2\, dx}$$

**Distance:**

$$d(f,g) = \sqrt{\int_a^b [f(x) - g(x)]^2\, dx}$$

---

## 4. Orthogonality of Functions

Two functions are **orthogonal** on $[a,b]$ if:

$$\langle f, g \rangle = \int_a^b f(x)\, g(x)\, dx = 0$$

**Classic example:** $\sin(nx)$ and $\cos(mx)$ on $[0, 2\pi]$:

$$\int_0^{2\pi} \sin(nx)\cos(mx)\, dx = 0 \quad \text{for all } m, n$$

$$\int_0^{2\pi} \sin(nx)\sin(mx)\, dx = \begin{cases} 0 & \text{if } n \neq m \\ \pi & \text{if } n = m \end{cases}$$

$$\int_0^{2\pi} \cos(nx)\cos(mx)\, dx = \begin{cases} 0 & \text{if } n \neq m \\ \pi & \text{if } n = m \neq 0 \\ 2\pi & \text{if } m = n = 0 \end{cases}$$

This orthogonality is the **foundation of Fourier series**.

---

## 5. Fourier Series as Orthogonal Expansion

Given the orthogonal set $\{1, \cos x, \sin x, \cos 2x, \sin 2x, \dots\}$ in $C[0, 2\pi]$:

$$f(x) \sim \frac{a_0}{2} + \sum_{n=1}^{\infty} \left(a_n \cos(nx) + b_n \sin(nx)\right)$$

**Coefficients by projection (Gram–Schmidt principle):**

$$a_n = \frac{\langle f, \cos(nx) \rangle}{\langle \cos(nx), \cos(nx) \rangle} = \frac{1}{\pi}\int_0^{2\pi} f(x)\cos(nx)\, dx$$

$$b_n = \frac{1}{\pi}\int_0^{2\pi} f(x)\sin(nx)\, dx$$

```
  Fourier series = projection onto orthogonal basis

  f(x)                     Projection onto each basis function:
   |                       ┌──────────┬──────────┬───────┐
   |   /\      /\          │ a₀/2 = ? │ a₁ = ?   │ b₁ = ?│
   |  /  \    /  \         │ constant  │ cos(x)   │sin(x) │
   | /    \  /    \        ├──────────┼──────────┼───────┤
   |/      \/      \       │ a₂ = ?   │ b₂ = ?   │  ...  │
   └────────────────→ x    │ cos(2x)  │ sin(2x)  │       │
                           └──────────┴──────────┴───────┘
```

---

## 6. $L^2[a,b]$ — Square-Integrable Functions

$$L^2[a,b] = \left\{ f : [a,b] \to \mathbb{R} \ \middle|\ \int_a^b |f(x)|^2\, dx < \infty \right\}$$

This is a **complete** inner product space (Hilbert space) — sequences that "should converge" actually do converge (in the $L^2$ norm).

**Key properties:**

| Property | Statement |
|----------|-----------|
| Inner product | $\langle f, g \rangle = \int_a^b f(x)g(x)\, dx$ |
| Completeness | Every Cauchy sequence converges |
| Separable | Has a countable orthonormal basis |
| Parseval's identity | $\|f\|^2 = \sum_{n} |\langle f, e_n \rangle|^2$ |
| Bessel's inequality | $\sum_{n=1}^{N} |\langle f, e_n \rangle|^2 \leq \|f\|^2$ |

---

## 7. Solution Spaces of Differential Equations

The set of solutions to a **homogeneous linear ODE** is a vector space.

**Example:** $y'' + y = 0$

Solutions: $y(x) = c_1 \cos x + c_2 \sin x$

- **Dimension:** 2
- **Basis:** $\{\cos x, \sin x\}$
- **Zero vector:** $y(x) = 0$

**General principle:** For an $n$-th order homogeneous linear ODE, the solution space has dimension $n$.

> **Warning:** Non-homogeneous equations ($y'' + y = 1$) do **not** give a subspace — the zero function is NOT a solution.

```
  Solution space of y'' + y = 0:

     y
     |     * cos(x)
   1 +   *   *
     | *       *
     *───────────*───→ x       All solutions = c₁cos(x) + c₂sin(x)
     |             *           2-dimensional subspace of C^∞(ℝ)
  -1 +               *
     |
```

---

## 8. Sequence Spaces

**$\ell^2$** — the space of square-summable sequences:

$$\ell^2 = \left\{ (a_1, a_2, a_3, \dots) \ \middle|\ \sum_{n=1}^{\infty} a_n^2 < \infty \right\}$$

**Inner product:** $\langle \mathbf{a}, \mathbf{b} \rangle = \sum_{n=1}^{\infty} a_n b_n$

**Standard basis (Schauder basis):**

$$\mathbf{e}_1 = (1, 0, 0, \dots), \quad \mathbf{e}_2 = (0, 1, 0, \dots), \quad \mathbf{e}_3 = (0, 0, 1, \dots), \;\dots$$

Every $\mathbf{a} \in \ell^2$ can be written $\mathbf{a} = \sum_{n=1}^{\infty} a_n \mathbf{e}_n$ (convergent in $\ell^2$ norm).

**Other sequence spaces:**

| Space | Condition | Subset relations |
|-------|-----------|--------------|
| $\ell^1$ | $\sum |a_n| < \infty$ | $\ell^1 \subset \ell^2$ |
| $\ell^2$ | $\sum a_n^2 < \infty$ | Hilbert space |
| $\ell^\infty$ | $\sup |a_n| < \infty$ | $\ell^2 \subset \ell^\infty$ |
| $c_0$ | $\lim a_n = 0$ | Subspace of $\ell^\infty$ |

---

## Summary Table

| Space | Description | Dimension | Inner Product |
|-------|-------------|:---------:|---------------|
| $\mathcal{F}(S, \mathbb{R})$ | All functions $S \to \mathbb{R}$ | $|S|$ or $\infty$ | Varies |
| $C[a,b]$ | Continuous functions | $\infty$ | $\int fg$ |
| $L^2[a,b]$ | Square-integrable | $\infty$ | $\int fg$ |
| ODE solutions | $n$-th order homogeneous | $n$ | Inherited |
| $\ell^2$ | Square-summable sequences | $\infty$ | $\sum a_n b_n$ |
| Fourier basis | Trig orthogonal set | $\infty$ | $\int fg$ on $[0,2\pi]$ |

---

## Quick Revision Questions

1. Verify that $C[0,1]$ is a subspace of $\mathcal{F}([0,1], \mathbb{R})$.

2. Show that $f(x) = x$ and $g(x) = x^2 - \frac{1}{2}$ are orthogonal on $[-1, 1]$ with the standard inner product.

3. Find the dimension and a basis for the solution space of $y'' - 3y' + 2y = 0$.

4. Write the Fourier coefficient formula for $b_3$ when expanding $f(x)$ in a Fourier series on $[0, 2\pi]$.

5. Explain why the set of solutions to $y'' + y = x$ is **not** a vector space.

6. Show that $(1, \frac{1}{2}, \frac{1}{3}, \frac{1}{4}, \dots) \in \ell^2$ but $(1, \frac{1}{2}, \frac{1}{3}, \frac{1}{4}, \dots) \notin \ell^1$.

---

[← Previous: General Vector Spaces](01-general-vector-spaces.md) · [Back to README](../README.md) · [Next: Polynomial Spaces →](03-polynomial-spaces.md)
