# 2.2 Limits and Continuity

[← Previous: Functions of a Complex Variable](01-functions-of-complex-variable.md) | [Next: Differentiability →](03-differentiability.md)

---

## Chapter Overview

Limits and continuity in $\mathbb{C}$ generalize the corresponding real-variable concepts, but with a crucial difference: a complex limit requires convergence **from all directions simultaneously** — not just from left and right. This stronger requirement is what makes complex analysis so much more constrained (and powerful) than real analysis.

---

## 1. Limits of Complex Functions

### 1.1 Definition

$$\lim_{z \to z_0} f(z) = L$$

means: for every $\varepsilon > 0$, there exists $\delta > 0$ such that

$$0 < |z - z_0| < \delta \implies |f(z) - L| < \varepsilon$$

```
ε-δ Definition (geometric):

  z-plane:                     w-plane:
     Im                           Im
      ▲  ╭─ δ ─╮                  ▲  ╭─ ε ─╮
      │  │     │                  │  │     │
      │  │ z₀• │   f              │  │  L• │
      │  │     │  ──►             │  │     │
      │  ╰─────╯                  │  ╰─────╯
  ────O──────────► Re         ────O──────────► Re

  All z in δ-disk (minus z₀)    map into ε-disk around L
```

### 1.2 Key Difference from Real Limits

In $\mathbb{R}$: $z \to z_0$ means $z$ approaches from the **left** or the **right** (1D).

In $\mathbb{C}$: $z \to z_0$ means $z$ approaches from **every direction** in the plane (2D).

```
Approach directions in ℂ:

         ╲   │   ╱
          ╲  │  ╱
           ╲ │ ╱
            ╲│╱
    ─────────•─────────   z₀
            ╱│╲
           ╱ │ ╲
          ╱  │  ╲
         ╱   │   ╲

  Limit must be the same along EVERY path to z₀
```

### 1.3 Component-wise Limits

$$\lim_{z \to z_0} f(z) = L \iff \begin{cases} \lim_{(x,y) \to (x_0,y_0)} u(x,y) = U \\ \lim_{(x,y) \to (x_0,y_0)} v(x,y) = V \end{cases}$$

where $f = u + iv$, $L = U + iV$, $z_0 = x_0 + iy_0$.

### 1.4 Limit Laws

If $\lim_{z \to z_0} f(z) = A$ and $\lim_{z \to z_0} g(z) = B$, then:

| Rule | Formula |
|------|---------|
| Sum | $\lim (f + g) = A + B$ |
| Product | $\lim (fg) = AB$ |
| Quotient | $\lim (f/g) = A/B$ if $B \ne 0$ |
| Scalar | $\lim (\alpha f) = \alpha A$ for $\alpha \in \mathbb{C}$ |
| Composition | $\lim_{z \to z_0} h(f(z)) = h(A)$ if $h$ is continuous at $A$ |

---

## 2. Techniques for Proving Limits Don't Exist

### 2.1 The Path Test

If $f(z)$ approaches different values along two different paths to $z_0$, then $\lim_{z \to z_0} f(z)$ **does not exist**.

### 2.2 Classic Example: $f(z) = \frac{\bar{z}}{z}$

Study $\lim_{z \to 0} \frac{\bar{z}}{z}$:

**Path 1**: Along the real axis ($z = x$, $y = 0$):
$$\frac{\bar{z}}{z} = \frac{x}{x} = 1$$

**Path 2**: Along the imaginary axis ($z = iy$, $x = 0$):
$$\frac{\bar{z}}{z} = \frac{-iy}{iy} = -1$$

Since $1 \ne -1$, the limit **does not exist**.

**Path 3**: Along $z = re^{i\theta}$ (general direction):
$$\frac{\bar{z}}{z} = \frac{re^{-i\theta}}{re^{i\theta}} = e^{-2i\theta}$$

Different $\theta$ give different values — the "limit" depends on the direction.

---

## 3. Limits Involving Infinity

### 3.1 Definitions

| Statement | Meaning |
|-----------|---------|
| $\lim_{z \to z_0} f(z) = \infty$ | $\forall M > 0$, $\exists \delta > 0$: $0 < \|z - z_0\| < \delta \implies \|f(z)\| > M$ |
| $\lim_{z \to \infty} f(z) = L$ | $\forall \varepsilon > 0$, $\exists R > 0$: $\|z\| > R \implies \|f(z) - L\| < \varepsilon$ |
| $\lim_{z \to \infty} f(z) = \infty$ | $\forall M > 0$, $\exists R > 0$: $\|z\| > R \implies \|f(z)\| > M$ |

### 3.2 Useful Reduction

$$\lim_{z \to \infty} f(z) = L \iff \lim_{w \to 0} f(1/w) = L$$

$$\lim_{z \to z_0} f(z) = \infty \iff \lim_{z \to z_0} \frac{1}{f(z)} = 0$$

---

## 4. Continuity

### 4.1 Definition

$f$ is **continuous at $z_0$** if:

1. $f(z_0)$ is defined
2. $\lim_{z \to z_0} f(z)$ exists
3. $\lim_{z \to z_0} f(z) = f(z_0)$

In short: $\lim_{z \to z_0} f(z) = f(z_0)$.

### 4.2 Continuity via Components

$f = u + iv$ is continuous at $z_0$ **if and only if** both $u$ and $v$ are continuous at $(x_0, y_0)$.

### 4.3 Properties

| Statement | Truth |
|-----------|-------|
| Sum of continuous functions is continuous | ✓ |
| Product of continuous functions is continuous | ✓ |
| Quotient is continuous where denominator ≠ 0 | ✓ |
| Composition of continuous functions is continuous | ✓ |
| Polynomials are continuous on $\mathbb{C}$ | ✓ |
| Rational functions are continuous on their domain | ✓ |
| $f(z) = \bar{z}$ is continuous | ✓ (but not analytic!) |
| $f(z) = \|z\|$ is continuous | ✓ (but not analytic!) |

### 4.4 Continuity Does Not Imply Analyticity

> **Critical insight**: Many continuous functions are NOT analytic. The functions $\bar{z}$, $|z|$, $\text{Re}(z)$, and $\text{Im}(z)$ are all continuous everywhere but analytic **nowhere**. Analyticity requires much more (see Ch. 2.3–2.5).

---

## 5. Uniform Continuity and Compactness

### 5.1 Uniform Continuity

$f$ is **uniformly continuous** on $S$ if the $\delta$ in the $\varepsilon$-$\delta$ definition can be chosen independently of $z_0$:

$$\forall \varepsilon > 0, \exists \delta > 0: \forall z_1, z_2 \in S, |z_1 - z_2| < \delta \implies |f(z_1) - f(z_2)| < \varepsilon$$

### 5.2 Heine–Cantor Theorem

> A continuous function on a **compact** (closed and bounded) set is uniformly continuous.

---

## 6. Design Example

### Example: Show that $\lim_{z \to i} (z^2 + 1) = 0$ using the $\varepsilon$-$\delta$ definition.

**Step 1.** We need $|z^2 + 1 - 0| < \varepsilon$ whenever $0 < |z - i| < \delta$.

**Step 2.** Factor: $z^2 + 1 = (z-i)(z+i)$

**Step 3.** So $|z^2 + 1| = |z - i| \cdot |z + i|$

**Step 4.** If $|z - i| < 1$, then $|z + i| = |z - i + 2i| \le |z - i| + 2 < 3$.

**Step 5.** Choose $\delta = \min(1, \varepsilon/3)$. Then:
$$|z^2 + 1| = |z-i||z+i| < \delta \cdot 3 \le \frac{\varepsilon}{3} \cdot 3 = \varepsilon \qquad \checkmark$$

---

## 7. Real-World Applications

| Application | Relevance of Limits/Continuity |
|-------------|-------------------------------|
| **Signal Processing** | Continuous signals can be approximated by polynomials (Weierstrass) |
| **Numerical Analysis** | Stability of algorithms requires continuous dependence on data |
| **Physics** | Physical fields are typically continuous; discontinuities = phase transitions |
| **Complex Dynamics** | Julia sets are boundaries of convergence regions — limit behavior |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Complex limit | Must converge from **all** directions in the plane |
| Non-existence test | Different limits along two paths ⟹ limit DNE |
| Component-wise | $\lim f = L \iff \lim u = U$ and $\lim v = V$ |
| Continuity | $\lim_{z \to z_0} f(z) = f(z_0)$ |
| Continuous ≠ analytic | $\bar{z}$, $\|z\|$ are continuous but not analytic |
| Polynomials | Continuous on all of $\mathbb{C}$ |
| Compact + continuous | ⟹ uniformly continuous (Heine–Cantor) |
| Limit at $\infty$ | Study $f(1/w)$ as $w \to 0$ |

---

## Quick Revision Questions

1. **State** the $\varepsilon$-$\delta$ definition of $\lim_{z \to z_0} f(z) = L$.

2. **Show** that $\lim_{z \to 0} \frac{z}{\bar{z}}$ does not exist by considering two different paths.

3. **Prove** that $f(z) = z^2$ is continuous at every $z_0 \in \mathbb{C}$.

4. **Why** is the requirement of convergence from all directions more restrictive in $\mathbb{C}$ than in $\mathbb{R}$?

5. **Give** an example of a function that is continuous everywhere on $\mathbb{C}$ but analytic nowhere.

6. **Evaluate** $\lim_{z \to \infty} \frac{z^2 + 1}{z^2 - 1}$.

---

[← Previous: Functions of a Complex Variable](01-functions-of-complex-variable.md) | [Next: Differentiability →](03-differentiability.md)
