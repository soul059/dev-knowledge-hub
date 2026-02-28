# 3.2 Complex Trigonometric Functions

[← Previous: Exponential Function](01-exponential-function.md) | [Next: Hyperbolic Functions →](03-hyperbolic-functions.md)

---

## Chapter Overview

Complex trigonometric functions are defined through the exponential function via Euler's formula. While they reduce to the familiar real sine and cosine on the real axis, they behave very differently in $\mathbb{C}$: they are **unbounded**, have **complex zeros** for some related functions, and satisfy familiar identities in a more general setting.

---

## 1. Definitions

### 1.1 Complex Sine and Cosine

$$\boxed{\cos z = \frac{e^{iz} + e^{-iz}}{2}, \qquad \sin z = \frac{e^{iz} - e^{-iz}}{2i}}$$

### 1.2 Other Trigonometric Functions

$$\tan z = \frac{\sin z}{\cos z}, \qquad \cot z = \frac{\cos z}{\sin z}$$

$$\sec z = \frac{1}{\cos z}, \qquad \csc z = \frac{1}{\sin z}$$

### 1.3 Entire Functions

$\sin z$ and $\cos z$ are **entire** (analytic on all of $\mathbb{C}$).

$\tan z$ is analytic except at $z = (2n+1)\pi/2$.

---

## 2. Real and Imaginary Parts

For $z = x + iy$:

$$\sin z = \sin x \cosh y + i \cos x \sinh y$$

$$\cos z = \cos x \cosh y - i \sin x \sinh y$$

### 2.1 Modulus Formulas

$$|\sin z|^2 = \sin^2 x + \sinh^2 y$$

$$|\cos z|^2 = \cos^2 x + \sinh^2 y$$

> Since $\sinh^2 y \to \infty$ as $|y| \to \infty$, both $|\sin z|$ and $|\cos z|$ are **unbounded** in $\mathbb{C}$. This is dramatically different from the real case where $|\sin x| \le 1$.

---

## 3. Properties

### 3.1 Derivatives

$$(\sin z)' = \cos z, \qquad (\cos z)' = -\sin z$$

$$(\tan z)' = \sec^2 z, \qquad (\cot z)' = -\csc^2 z$$

### 3.2 Identities (Same as Real)

All real trigonometric identities extend to $\mathbb{C}$:

| Identity | Formula |
|----------|---------|
| Pythagorean | $\sin^2 z + \cos^2 z = 1$ |
| Addition | $\sin(z_1+z_2) = \sin z_1 \cos z_2 + \cos z_1 \sin z_2$ |
| Addition | $\cos(z_1+z_2) = \cos z_1 \cos z_2 - \sin z_1 \sin z_2$ |
| Double angle | $\sin(2z) = 2\sin z \cos z$ |
| Double angle | $\cos(2z) = \cos^2 z - \sin^2 z$ |
| Period | $\sin(z + 2\pi) = \sin z$ |
| Period | $\cos(z + 2\pi) = \cos z$ |
| Parity | $\sin(-z) = -\sin z$ (odd) |
| Parity | $\cos(-z) = \cos z$ (even) |

### 3.3 Zeros

$$\sin z = 0 \iff z = n\pi, \quad n \in \mathbb{Z}$$

$$\cos z = 0 \iff z = \frac{(2n+1)\pi}{2}, \quad n \in \mathbb{Z}$$

All zeros are **real** (same as in real analysis).

### 3.4 Key Differences from Real Trig

| Real | Complex |
|------|---------|
| $\|\sin x\| \le 1$ | $\|\sin z\|$ can be arbitrarily large |
| $\|\cos x\| \le 1$ | $\|\cos z\|$ can be arbitrarily large |
| $\sin x = 2$ has no solution | $\sin z = 2$ HAS solutions |

---

## 4. Solving Trigonometric Equations

### 4.1 Example: Solve $\sin z = 2$

Use the definition:
$$\frac{e^{iz} - e^{-iz}}{2i} = 2$$

Let $w = e^{iz}$:
$$w - w^{-1} = 4i \implies w^2 - 4iw - 1 = 0$$

$$w = \frac{4i \pm \sqrt{-16+4}}{2} = \frac{4i \pm \sqrt{-12}}{2} = \frac{4i \pm 2i\sqrt{3}}{2} = i(2 \pm \sqrt{3})$$

So $e^{iz} = i(2 + \sqrt{3})$ or $e^{iz} = i(2 - \sqrt{3})$.

Taking logarithms: $iz = \ln|w| + i\arg(w) + 2k\pi i$

For $w = i(2+\sqrt{3})$: $|w| = 2+\sqrt{3}$, $\arg(w) = \pi/2$

$$iz = \ln(2+\sqrt{3}) + i\pi/2 + 2k\pi i$$

$$z = \frac{\pi}{2} + 2k\pi - i\ln(2+\sqrt{3})$$

---

## 5. Design Example

### Example: Compute $|\sin(1+2i)|$.

**Step 1.** Use the modulus formula:
$$|\sin z|^2 = \sin^2 x + \sinh^2 y$$

**Step 2.** $x = 1$, $y = 2$:
$$|\sin(1+2i)|^2 = \sin^2 1 + \sinh^2 2$$

**Step 3.** $\sin 1 \approx 0.8415$, $\sinh 2 = \frac{e^2 - e^{-2}}{2} \approx 3.6269$

$$|\sin(1+2i)|^2 \approx 0.7081 + 13.1544 = 13.8625$$

$$|\sin(1+2i)| \approx 3.724$$

This is much larger than 1!

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| $\sin z$ | $(e^{iz} - e^{-iz})/(2i)$ |
| $\cos z$ | $(e^{iz} + e^{-iz})/2$ |
| Real/Im parts of $\sin z$ | $\sin x \cosh y + i\cos x \sinh y$ |
| Real/Im parts of $\cos z$ | $\cos x \cosh y - i\sin x \sinh y$ |
| $\|\sin z\|^2$ | $\sin^2 x + \sinh^2 y$ |
| Derivatives | $(\sin z)' = \cos z$, $(\cos z)' = -\sin z$ |
| Period | $2\pi$ (same as real) |
| Unboundedness | $\|\sin z\|$, $\|\cos z\|$ unbounded in $\mathbb{C}$ |
| Zeros of $\sin z$ | $z = n\pi$ only (all real) |

---

## Quick Revision Questions

1. **Derive** the expression $\sin z = \sin x \cosh y + i\cos x \sinh y$.

2. **Prove** that $\sin^2 z + \cos^2 z = 1$ using the exponential definitions.

3. **Show** that $|\sin z| \ge |\sinh y|$ and hence $\sin z$ is unbounded.

4. **Find** all solutions of $\cos z = 3$.

5. **Compute** $\cos(i)$ and verify it is real.

6. **Explain** why trigonometric identities valid for real numbers remain valid for complex numbers.

---

[← Previous: Exponential Function](01-exponential-function.md) | [Next: Hyperbolic Functions →](03-hyperbolic-functions.md)
