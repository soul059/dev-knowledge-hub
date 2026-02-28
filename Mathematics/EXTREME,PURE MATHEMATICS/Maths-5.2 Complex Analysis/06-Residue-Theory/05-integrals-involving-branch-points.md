# 6.5 Integrals Involving Branch Points

[← Previous: Principal Value Integrals](04-principal-value-integrals.md) | [Next: Unit 7 — Conformal Maps →](../07-Conformal-Mapping/01-conformal-maps.md)

---

## Chapter Overview

When the integrand involves multi-valued functions (like $x^\alpha$, $\ln x$, or fractional powers), the standard semicircular contour won't work because the function changes value as we cross the branch cut. Instead, we use **keyhole contours**, **dog-bone contours**, or **sector contours** that respect the branch cut geometry.

---

## 1. The Keyhole Contour

### 1.1 Setup

For integrals of the form $\int_0^\infty x^{\alpha-1} R(x)\,dx$ where $0 < \alpha < 1$ and $R$ is rational:

- Choose branch cut along $[0, \infty)$ (positive real axis)
- Use a keyhole contour that goes around the branch cut:

```
Keyhole contour for ∫₀^∞ x^(α-1) R(x) dx:

                    ╭────── Γ_R ──────╮
                   ╱                   ╲
                  ╱  × poles of R       ╲
                 ╱                       ╲
  ──────────────•                         •──────────── (above cut)
                │  ← γ_ε   ╭─╮           │
                │           │0│           │  branch cut
                │           ╰─╯           │  along [0,∞)
  ──────────────•                         •──────────── (below cut)
                 ╲                       ╱
                  ╲                     ╱
                   ╲                   ╱
                    ╰────── Γ_R ──────╯

  Traverse counterclockwise:
  Above cut (left to right) → Γ_R → Below cut (right to left) → γ_ε
```

### 1.2 Key Observation

On the **upper edge** of the branch cut: $z = x$, so $z^{\alpha-1} = x^{\alpha-1}$.

On the **lower edge**: $z = xe^{2\pi i}$, so $z^{\alpha-1} = x^{\alpha-1}e^{2\pi i(\alpha-1)}$.

The function has **different values** above and below the cut! This is what makes the contour useful.

### 1.3 Result

$$\oint = \int_0^\infty x^{\alpha-1}R(x)\,dx - e^{2\pi i(\alpha-1)}\int_0^\infty x^{\alpha-1}R(x)\,dx = (1 - e^{2\pi i(\alpha-1)})\int_0^\infty x^{\alpha-1}R(x)\,dx$$

Equating with the residue sum:

$$\int_0^\infty x^{\alpha-1}R(x)\,dx = \frac{2\pi i}{1 - e^{2\pi i(\alpha-1)}}\sum \text{Res}$$

$$= \frac{\pi}{\sin(\pi\alpha)} \cdot (-1) \sum \text{Res}(z^{\alpha-1}R(z), z_k) \cdot e^{-i\pi\alpha}$$

(The exact simplification depends on the phase convention.)

---

## 2. Design Example: $\int_0^\infty \frac{x^{-1/3}}{1+x}\,dx$

### Step 1. Setup

$\alpha = 2/3$ (since $x^{\alpha-1} = x^{-1/3}$ gives $\alpha - 1 = -1/3$).

Use keyhole contour with branch cut on $[0,\infty)$, $z^{-1/3} = |z|^{-1/3}e^{-i\theta/3}$ with $0 < \theta < 2\pi$.

### Step 2. Contour Integral

$$\oint = (1 - e^{-2\pi i/3})\int_0^\infty \frac{x^{-1/3}}{1+x}\,dx$$

(Large circle and small circle contributions vanish.)

### Step 3. Residues

Pole at $z = -1 = e^{i\pi}$:

$$\text{Res} = \lim_{z\to -1}(z+1)\frac{z^{-1/3}}{1+z} = (-1)^{-1/3} = e^{-i\pi/3}$$

### Step 4. Result

$$\int_0^\infty \frac{x^{-1/3}}{1+x}\,dx = \frac{2\pi i \cdot e^{-i\pi/3}}{1 - e^{-2i\pi/3}}$$

Simplify the denominator: $1 - e^{-2i\pi/3} = e^{-i\pi/3}(e^{i\pi/3} - e^{-i\pi/3}) = e^{-i\pi/3} \cdot 2i\sin(\pi/3)$

$$= \frac{2\pi i \cdot e^{-i\pi/3}}{e^{-i\pi/3}\cdot 2i\sin(\pi/3)} = \frac{2\pi i}{2i\cdot\sqrt{3}/2} = \frac{2\pi}{\sqrt{3}}$$

---

## 3. The General Formula

$$\boxed{\int_0^\infty \frac{x^{\alpha-1}}{1+x}\,dx = \frac{\pi}{\sin(\pi\alpha)}, \quad 0 < \alpha < 1}$$

This is equivalent to the **Beta function** identity $B(\alpha, 1-\alpha) = \frac{\pi}{\sin\pi\alpha}$, itself a consequence of the reflection formula for the Gamma function.

---

## 4. Integrals Involving Logarithms

### 4.1 Method

For $\int_0^\infty R(x)\ln x\,dx$, use:

$$\int_0^\infty R(x)\ln x\,dx = \frac{\partial}{\partial\alpha}\Big|_{\alpha=1}\int_0^\infty x^{\alpha-1}R(x)\,dx$$

Or apply the keyhole contour directly with $f(z) = R(z)\text{Log}(z)$.

### 4.2 Example: $\int_0^\infty \frac{\ln x}{1+x^2}\,dx$

Use keyhole contour with $f(z) = \frac{(\text{Log}\,z)^2}{1+z^2}$ (squaring the log is a common trick):

Above cut: $\text{Log}\,z = \ln x$.
Below cut: $\text{Log}\,z = \ln x + 2\pi i$.

After careful calculation:

$$\int_0^\infty \frac{\ln x}{1+x^2}\,dx = 0$$

(The integrand is related to $\frac{d}{d\alpha}\frac{\pi}{\sin\pi\alpha}\Big|_{\alpha=1/2}$.)

---

## 5. Dog-Bone Contour

For integrals like $\int_{-1}^{1}\frac{f(x)}{\sqrt{1-x^2}}\,dx$ where branch points are at $x = \pm 1$:

```
Dog-bone contour for branch points at ±1:

              ╭────────────────────╮
             ╱                      ╲
  ──────────•──────────────────────── •──────
           -1    ← branch cut →      1
  ──────────•────────────────────────•──────
             ╲                      ╱
              ╰────────────────────╯

  Contour wraps around the segment [-1, 1]
  Contributions above and below the cut differ
```

---

## 6. Sector Contour

For integrals like $\int_0^\infty R(x^n)\,dx$, use a sector of angle $2\pi/n$:

```
Sector contour (angle 2π/n):

                    ╱ z = re^(2πi/n)
                   ╱
                  ╱
                 ╱  θ = 2π/n
                ╱
  ─────────────•────────────────► x
               0

  Along the real axis: z = x
  Along the ray: z = xe^(2πi/n)
  Curved arc at radius R
```

**Example:** $\int_0^\infty \frac{dx}{1+x^n}$ uses a sector of angle $2\pi/n$.

$$\int_0^\infty \frac{dx}{1+x^n} = \frac{\pi/n}{\sin(\pi/n)}, \quad n \geq 2$$

---

## 7. Summary of Contour Types

| Integral Type | Contour | Branch Cut |
|---------------|---------|------------|
| $\int_0^\infty x^{\alpha-1}R(x)\,dx$ | Keyhole | $[0, \infty)$ |
| $\int_0^\infty R(x)\ln x\,dx$ | Keyhole (with $\log^2$) | $[0, \infty)$ |
| $\int_{-1}^1 f(x)/\sqrt{1-x^2}\,dx$ | Dog-bone | $[-1, 1]$ |
| $\int_0^\infty R(x^n)\,dx$ | Sector (angle $2\pi/n$) | $[0, \infty)$ |

---

## 8. Real-World Applications

| Application | Integral Type |
|-------------|--------------|
| **Diffraction theory** | Fresnel integrals via sector contours |
| **Statistical mechanics** | Bose–Einstein integrals $\int_0^\infty \frac{x^{s-1}}{e^x - 1}\,dx$ |
| **Quantum field theory** | Dimensional regularization uses $x^\alpha$ integrals |
| **Heat conduction** | Inverse Laplace transforms with branch points |

---

## Summary Table

| Result | Formula |
|--------|---------|
| Euler's integral | $\int_0^\infty \frac{x^{\alpha-1}}{1+x}\,dx = \frac{\pi}{\sin\pi\alpha}$ |
| Keyhole principle | Above/below cut differ by $e^{2\pi i\alpha}$ factor |
| Dog-bone | For finite branch cuts $[a, b]$ |
| Sector | For $\int_0^\infty R(x^n)\,dx$ with angle $2\pi/n$ |
| Log integral trick | Use $(\log z)^2$ or differentiate w.r.t. $\alpha$ |

---

## Quick Revision Questions

1. **Explain** why the keyhole contour is needed for integrals involving $x^\alpha$.

2. **Evaluate** $\int_0^\infty \frac{x^{1/2}}{1+x^2}\,dx$ using a keyhole contour.

3. **Derive** the formula $\int_0^\infty \frac{x^{\alpha-1}}{1+x}\,dx = \frac{\pi}{\sin\pi\alpha}$.

4. **Explain** the difference above and below the branch cut for $z^{\alpha-1}$ with cut along $[0,\infty)$.

5. **Evaluate** $\int_0^\infty \frac{\ln x}{(1+x)^2}\,dx$ using appropriate contour techniques.

6. **Describe** when to use a sector contour instead of a keyhole contour.

---

[← Previous: Principal Value Integrals](04-principal-value-integrals.md) | [Next: Unit 7 — Conformal Maps →](../07-Conformal-Mapping/01-conformal-maps.md)
