# 4.2 Line Integrals in the Complex Plane

[← Previous: Contours and Paths](01-contours-and-paths.md) | [Next: Cauchy–Goursat Theorem →](03-cauchy-goursat-theorem.md)

---

## Chapter Overview

The **complex line integral** $\int_\gamma f(z)\,dz$ is the central object of complex analysis. Unlike real integrals, complex integrals generally depend on the *path* from $A$ to $B$. This chapter covers the definition, basic properties, the ML-inequality (the single most used estimation tool), and conditions under which the integral is path-independent.

---

## 1. Definition of the Complex Integral

### 1.1 Riemann Sum Approach

Given a contour $\gamma$ from $z_A$ to $z_B$ and a continuous function $f$, partition $\gamma$ into sub-arcs via points $z_0 = z_A, z_1, \ldots, z_n = z_B$:

$$\int_\gamma f(z)\,dz = \lim_{n \to \infty} \sum_{k=1}^{n} f(\zeta_k)(z_k - z_{k-1})$$

where $\zeta_k$ is any point on the sub-arc from $z_{k-1}$ to $z_k$.

### 1.2 Parametric Form

If $\gamma(t) = x(t) + iy(t)$, $t \in [a,b]$:

$$\boxed{\int_\gamma f(z)\,dz = \int_a^b f(\gamma(t))\,\gamma'(t)\,dt}$$

### 1.3 Component Form

Writing $f = u + iv$ and $dz = dx + i\,dy$:

$$\int_\gamma f\,dz = \int_\gamma (u\,dx - v\,dy) + i\int_\gamma (v\,dx + u\,dy)$$

This expresses the complex integral as two real line integrals.

---

## 2. Basic Properties

| Property | Formula |
|----------|---------|
| Linearity | $\int_\gamma [\alpha f + \beta g]\,dz = \alpha\int_\gamma f\,dz + \beta\int_\gamma g\,dz$ |
| Reversal | $\int_{-\gamma} f\,dz = -\int_\gamma f\,dz$ |
| Concatenation | $\int_{\gamma_1 + \gamma_2} f\,dz = \int_{\gamma_1} f\,dz + \int_{\gamma_2} f\,dz$ |
| Reparametrization | Value unchanged under orientation-preserving reparametrization |

---

## 3. Key Examples

### 3.1 $\int_\gamma z\,dz$ along any path from $0$ to $1+i$

$$\int_\gamma z\,dz = \frac{z^2}{2}\Big|_0^{1+i} = \frac{(1+i)^2}{2} = \frac{2i}{2} = i$$

Path-independent! (because $z$ has antiderivative $z^2/2$).

### 3.2 $\int_{|z|=1} \frac{1}{z}\,dz$ (counterclockwise)

Parametrize: $\gamma(t) = e^{it}$, $t \in [0, 2\pi]$, so $\gamma'(t) = ie^{it}$:

$$\int_{|z|=1} \frac{1}{z}\,dz = \int_0^{2\pi} \frac{1}{e^{it}} \cdot ie^{it}\,dt = \int_0^{2\pi} i\,dt = 2\pi i$$

This is the most important integral in complex analysis!

### 3.3 $\int_{|z|=1} z^n\,dz$ for integer $n$

$$\int_{|z|=1} z^n\,dz = \int_0^{2\pi} e^{int} \cdot ie^{it}\,dt = i\int_0^{2\pi} e^{i(n+1)t}\,dt = \begin{cases} 2\pi i & n = -1 \\ 0 & n \neq -1 \end{cases}$$

```
Fundamental integral formula:

  ┌─────────────────────────────────────────┐
  │                                         │
  │   ∮  z^n dz  =  { 2πi  if n = -1       │
  │  |z|=1          { 0    if n ≠ -1        │
  │                   (n integer)           │
  └─────────────────────────────────────────┘
```

### 3.4 $\int_{|z|=1} \bar{z}\,dz$

$\gamma(t) = e^{it}$, $\bar{\gamma(t)} = e^{-it}$:

$$\int_{|z|=1} \bar{z}\,dz = \int_0^{2\pi} e^{-it} \cdot ie^{it}\,dt = 2\pi i$$

Note: $\bar{z}$ is not analytic, so Cauchy's theorem does not apply.

---

## 4. The ML-Inequality

### 4.1 Statement

If $|f(z)| \leq M$ for all $z$ on $\gamma$ and $L(\gamma)$ is the arc length:

$$\boxed{\left|\int_\gamma f(z)\,dz\right| \leq ML}$$

### 4.2 Proof Sketch

$$\left|\int_a^b f(\gamma(t))\gamma'(t)\,dt\right| \leq \int_a^b |f(\gamma(t))|\,|\gamma'(t)|\,dt \leq M \int_a^b |\gamma'(t)|\,dt = ML$$

### 4.3 Application: Bounding Vanishing Integrals

To show $\int_{\gamma_R} f\,dz \to 0$ as $R \to \infty$ for a semicircular arc $\gamma_R$:

1. Find $M(R)$ such that $|f(z)| \leq M(R)$ on $\gamma_R$
2. Compute $L(\gamma_R) = \pi R$
3. Show $M(R) \cdot \pi R \to 0$ as $R \to \infty$

```
Typical application of ML-inequality:

           ╭──── γ_R (semicircle, radius R) ────╮
          ╱                                       ╲
         ╱     |f(z)| ≤ M(R) on γ_R               ╲
        ╱                                           ╲
   ────•───────────────────────────────────────────────•────
      -R          real axis integral                    R

  If M(R) · πR → 0  then ∫_{γ_R} f dz → 0
```

---

## 5. Antiderivatives and Path Independence

### 5.1 Existence of Antiderivative

If $f$ is continuous on a domain $D$ and has an **antiderivative** $F$ (i.e., $F'(z) = f(z)$ in $D$), then for any contour $\gamma$ in $D$ from $z_1$ to $z_2$:

$$\int_\gamma f(z)\,dz = F(z_2) - F(z_1)$$

The integral is **path-independent**.

### 5.2 Equivalent Conditions

For a continuous function $f$ on a domain $D$, the following are equivalent:

| Condition | Description |
|-----------|------------|
| (i) | $f$ has an antiderivative on $D$ |
| (ii) | $\int_\gamma f\,dz$ is path-independent in $D$ |
| (iii) | $\oint_\gamma f\,dz = 0$ for every closed contour $\gamma$ in $D$ |

### 5.3 When $1/z$ Does / Doesn't Have an Antiderivative

- On $\mathbb{C} \setminus \{0\}$: $1/z$ has **no** antiderivative (since $\oint_{|z|=1} (1/z)\,dz = 2\pi i \neq 0$).
- On $\mathbb{C} \setminus (-\infty, 0]$: $1/z$ **has** antiderivative $\text{Log}(z)$.

---

## 6. Design Example

### Example: Evaluate $\int_\gamma (3z^2 + 2z)\,dz$ where $\gamma$ is the parabola $y = x^2$ from $0$ to $1+i$.

**Method 1 (Antiderivative):** Since $f(z) = 3z^2 + 2z$ has antiderivative $F(z) = z^3 + z^2$:

$$\int_\gamma f\,dz = F(1+i) - F(0) = (1+i)^3 + (1+i)^2$$

$$(1+i)^2 = 2i, \quad (1+i)^3 = (1+i)(2i) = 2i + 2i^2 = -2 + 2i$$

$$= (-2+2i) + 2i = -2 + 4i$$

**Method 2 (Parametric — verification):** Let $\gamma(t) = t + it^2$, $t \in [0,1]$, so $\gamma'(t) = 1 + 2it$:

$$\int_0^1 [3(t+it^2)^2 + 2(t+it^2)](1+2it)\,dt$$

After expansion, this yields the same result: $-2 + 4i\;\checkmark$

---

## 7. Real-World Applications

| Application | Integration Role |
|-------------|-----------------|
| **Electrostatics** | Work done moving charge along a path |
| **Fluid dynamics** | Circulation $= \text{Re}\oint f\,dz$, Flux $= \text{Im}\oint f\,dz$ |
| **Control theory** | Nyquist integral counts encirclements |
| **Signal processing** | Inverse $z$-transform via contour integral |

---

## Summary Table

| Concept | Key Formula / Fact |
|---------|-------------------|
| Complex integral | $\int_\gamma f\,dz = \int_a^b f(\gamma(t))\gamma'(t)\,dt$ |
| Fundamental integral | $\oint_{|z|=1} z^n\,dz = 2\pi i\cdot\delta_{n,-1}$ |
| ML-inequality | $|\int_\gamma f\,dz| \leq ML$ |
| Antiderivative theorem | $\int_\gamma f\,dz = F(z_2) - F(z_1)$ if $F' = f$ |
| Path independence | $\Leftrightarrow$ existence of antiderivative $\Leftrightarrow$ $\oint = 0$ |
| Reversal | $\int_{-\gamma} = -\int_\gamma$ |
| Linearity | $\int (\alpha f + \beta g) = \alpha\int f + \beta\int g$ |

---

## Quick Revision Questions

1. **Evaluate** $\int_\gamma z^2\,dz$ from $0$ to $2+i$ along any path. Why is the path irrelevant?

2. **Compute** $\oint_{|z|=2} \frac{1}{z}\,dz$ and $\oint_{|z|=2} z^3\,dz$.

3. **Apply** the ML-inequality to show $\left|\int_{|z|=R} \frac{e^z}{z^3}\,dz\right| \leq \frac{2\pi e^R}{R^2}$.

4. **Explain** why $1/z$ has an antiderivative on $\{z : \text{Re}(z) > 0\}$ but not on $\mathbb{C}\setminus\{0\}$.

5. **Evaluate** $\int_\gamma |z|^2\,dz$ along: (a) the line from $0$ to $1+i$, (b) the segments $0 \to 1 \to 1+i$. Are they equal?

6. **State** the ML-inequality and outline its proof.

---

[← Previous: Contours and Paths](01-contours-and-paths.md) | [Next: Cauchy–Goursat Theorem →](03-cauchy-goursat-theorem.md)
