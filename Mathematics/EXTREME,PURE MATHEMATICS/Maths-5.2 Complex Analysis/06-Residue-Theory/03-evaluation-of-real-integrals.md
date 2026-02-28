# 6.3 Evaluation of Real Integrals

[← Previous: The Residue Theorem](02-residue-theorem.md) | [Next: Principal Value Integrals →](04-principal-value-integrals.md)

---

## Chapter Overview

One of the most spectacular applications of complex analysis: evaluating **definite real integrals** that are difficult or impossible by real-variable methods. We convert real integrals into complex contour integrals, apply the Residue Theorem, and extract the real answer. Three main types are covered here.

---

## 1. Type I: Trigonometric Integrals $\int_0^{2\pi} R(\cos\theta, \sin\theta)\,d\theta$

### 1.1 Method

Substitute $z = e^{i\theta}$, $d\theta = dz/(iz)$:

$$\cos\theta = \frac{z + z^{-1}}{2}, \quad \sin\theta = \frac{z - z^{-1}}{2i}$$

The integral becomes a contour integral over $|z| = 1$.

### 1.2 Template

$$\int_0^{2\pi} R(\cos\theta, \sin\theta)\,d\theta = \oint_{|z|=1} R\left(\frac{z+z^{-1}}{2}, \frac{z-z^{-1}}{2i}\right)\frac{dz}{iz}$$

```
Substitution z = e^(iθ) converts:

  Real:  ∫₀²π R(cosθ, sinθ) dθ    →    Complex:  ∮_{|z|=1} f(z) dz

          θ: 0 → 2π                           z traces unit circle
          cosθ = (z+1/z)/2
          sinθ = (z-1/z)/2i
          dθ = dz/(iz)
```

### 1.3 Example: $\int_0^{2\pi} \frac{d\theta}{2 + \cos\theta}$

**Step 1.** Substitute:

$$\cos\theta = \frac{z + z^{-1}}{2}, \quad d\theta = \frac{dz}{iz}$$

$$\int = \oint_{|z|=1} \frac{1}{2 + (z+z^{-1})/2}\cdot\frac{dz}{iz} = \oint_{|z|=1} \frac{dz}{iz(2 + z/2 + 1/(2z))}$$

$$= \oint_{|z|=1} \frac{dz}{i(2z + z^2/2 + 1/2)} = \oint_{|z|=1} \frac{2\,dz}{i(z^2 + 4z + 1)}$$

**Step 2.** Roots of $z^2 + 4z + 1 = 0$: $z = -2 \pm \sqrt{3}$.

$z_1 = -2 + \sqrt{3} \approx -0.27$ (inside $|z| = 1$).
$z_2 = -2 - \sqrt{3} \approx -3.73$ (outside).

**Step 3.** $\text{Res}(f, z_1) = \frac{2/i}{2z_1 + 4} = \frac{2/i}{2\sqrt{3}} = \frac{1}{i\sqrt{3}}$

**Step 4.** Result:

$$\int_0^{2\pi} \frac{d\theta}{2+\cos\theta} = 2\pi i \cdot \frac{1}{i\sqrt{3}} = \frac{2\pi}{\sqrt{3}}$$

---

## 2. Type II: Integrals $\int_{-\infty}^{\infty} f(x)\,dx$ (Rational Functions)

### 2.1 Method

Close the contour with a semicircular arc in the upper (or lower) half-plane:

```
Semicircular contour for real-line integrals:

          ╭──── Γ_R (semicircle, radius R) ────╮
         ╱                                       ╲
        ╱     × z₁   × z₂   (poles in UHP)       ╲
       ╱                                           ╲
  ────•─────────────────────────────────────────────•────
     -R              real axis                       R

  ∮ = ∫_{-R}^{R} f(x)dx + ∫_{Γ_R} f(z)dz = 2πi Σ Res(UHP)

  If ∫_{Γ_R} → 0 as R → ∞, then ∫_{-∞}^{∞} f(x)dx = 2πi Σ Res(UHP)
```

### 2.2 When $\int_{\Gamma_R} \to 0$

If $f(z) = P(z)/Q(z)$ is rational with $\deg Q \geq \deg P + 2$, then:

$$|f(z)| = O(1/R^2) \text{ on } \Gamma_R, \quad L(\Gamma_R) = \pi R$$

$$\left|\int_{\Gamma_R}\right| \leq \frac{C}{R^2}\cdot\pi R = \frac{C\pi}{R} \to 0$$

### 2.3 Formula

$$\boxed{\int_{-\infty}^{\infty} f(x)\,dx = 2\pi i \sum_{\text{Im}(z_k)>0} \text{Res}(f, z_k)}$$

(Sum over poles in the **upper half-plane** only.)

### 2.4 Example: $\int_{-\infty}^{\infty} \frac{dx}{(x^2+1)(x^2+4)}$

**Step 1.** Poles: $z = \pm i$ (from $z^2+1$), $z = \pm 2i$ (from $z^2+4$). Upper half-plane poles: $z = i$ and $z = 2i$.

**Step 2.** Partial fractions give $f(z) = \frac{1}{3}\left(\frac{1}{z^2+1} - \frac{1}{z^2+4}\right)$.

**Step 3.** Residues:

$$\text{Res}(f, i) = \frac{1}{(2i)(i^2+4)} = \frac{1}{2i \cdot 3} = \frac{1}{6i}$$

$$\text{Res}(f, 2i) = \frac{1}{(2i\cdot 2)(4i^2+1)} = \frac{1}{4i(-3)} = -\frac{1}{12i}$$

**Step 4.** Result:

$$\int_{-\infty}^{\infty} \frac{dx}{(x^2+1)(x^2+4)} = 2\pi i\left(\frac{1}{6i} - \frac{1}{12i}\right) = 2\pi i \cdot \frac{1}{12i} = \frac{\pi}{6}$$

---

## 3. Type III: Integrals $\int_{-\infty}^{\infty} f(x)e^{iax}\,dx$ (Jordan's Lemma)

### 3.1 Jordan's Lemma

> If $f(z) \to 0$ uniformly as $|z| \to \infty$ in the upper half-plane and $a > 0$, then:
> $$\int_{\Gamma_R} f(z)e^{iaz}\,dz \to 0 \quad \text{as } R \to \infty$$

This is stronger than the ML-inequality — it only requires $f \to 0$ (not $f \cdot R \to 0$).

### 3.2 Why Jordan's Lemma Works

On $\Gamma_R$: $z = Re^{i\theta}$, so $e^{iaz} = e^{iaR\cos\theta - aR\sin\theta}$. Since $\sin\theta > 0$ for $\theta \in (0,\pi)$:

$$|e^{iaz}| = e^{-aR\sin\theta} \to 0 \text{ exponentially}$$

The exponential decay compensates for the $\pi R$ length.

### 3.3 Example: $\int_{-\infty}^{\infty} \frac{\cos x}{x^2+1}\,dx$

**Step 1.** Write $\cos x = \text{Re}(e^{ix})$, so compute:

$$I = \text{Re}\int_{-\infty}^{\infty}\frac{e^{ix}}{x^2+1}\,dx$$

**Step 2.** Close in UHP. $f(z) = \frac{1}{z^2+1}$. By Jordan's lemma, the arc integral vanishes.

**Step 3.** UHP pole: $z = i$ (simple). $\text{Res} = \frac{e^{i\cdot i}}{2i} = \frac{e^{-1}}{2i}$.

**Step 4.** $\int = 2\pi i \cdot \frac{e^{-1}}{2i} = \frac{\pi}{e}$.

**Step 5.** $\int_{-\infty}^{\infty}\frac{\cos x}{x^2+1}\,dx = \text{Re}\left(\frac{\pi}{e}\right) = \frac{\pi}{e}$.

---

## 4. Summary of Integral Types

| Type | Integral Form | Contour | Key Condition |
|------|---------------|---------|---------------|
| I | $\int_0^{2\pi} R(\cos\theta, \sin\theta)\,d\theta$ | Unit circle $|z| = 1$ | Substitution $z = e^{i\theta}$ |
| II | $\int_{-\infty}^{\infty} \frac{P(x)}{Q(x)}\,dx$ | Semicircle UHP | $\deg Q \geq \deg P + 2$ |
| III | $\int_{-\infty}^{\infty} f(x)e^{iax}\,dx$ | Semicircle UHP + Jordan | $a > 0$, $f \to 0$ in UHP |

---

## 5. Real-World Applications

| Application | Integral Type |
|-------------|--------------|
| **Quantum mechanics** | Fourier integrals of Green's functions (Type III) |
| **Probability** | Characteristic function inversion (Type III) |
| **Optics** | Diffraction integrals (Type III) |
| **Signal processing** | Frequency response computation (Type I) |
| **Electrostatics** | Potential integrals (Type II) |

---

## Summary Table

| Method | Formula |
|--------|---------|
| Type I substitution | $z = e^{i\theta}$, $\cos\theta = (z+1/z)/2$, $d\theta = dz/(iz)$ |
| Type II formula | $\int_{-\infty}^{\infty} f = 2\pi i \sum \text{Res}(\text{UHP})$ |
| ML-inequality condition | $\deg Q \geq \deg P + 2$ for $P/Q$ |
| Jordan's lemma | $f \to 0$ in UHP suffices for $fe^{iaz}$ |
| Extracting $\cos$/$\sin$ | $\int f(x)\cos(ax)\,dx = \text{Re}\int f(x)e^{iax}\,dx$ |

---

## Quick Revision Questions

1. **Evaluate** $\int_0^{2\pi} \frac{d\theta}{5 + 4\sin\theta}$ using the substitution $z = e^{i\theta}$.

2. **Evaluate** $\int_{-\infty}^{\infty} \frac{dx}{(x^2+1)^2}$.

3. **State** Jordan's lemma and explain when it is needed instead of the ML-inequality.

4. **Evaluate** $\int_{-\infty}^{\infty} \frac{x\sin x}{x^2+4}\,dx$.

5. **Explain** why we close in the upper half-plane for $e^{iax}$ with $a > 0$, and what we do when $a < 0$.

6. **Evaluate** $\int_0^{\infty} \frac{dx}{x^4+1}$ (Hint: the integrand is even).

---

[← Previous: The Residue Theorem](02-residue-theorem.md) | [Next: Principal Value Integrals →](04-principal-value-integrals.md)
