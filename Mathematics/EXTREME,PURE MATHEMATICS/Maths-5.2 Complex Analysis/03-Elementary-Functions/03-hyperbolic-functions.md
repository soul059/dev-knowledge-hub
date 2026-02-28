# 3.3 Complex Hyperbolic Functions

[← Previous: Trigonometric Functions](02-trigonometric-functions.md) | [Next: Logarithmic Function →](04-logarithmic-function.md)

---

## Chapter Overview

Hyperbolic functions in the complex plane are intimately connected to trigonometric functions. In $\mathbb{R}$, $\cosh$ and $\sinh$ are fundamentally different from $\cos$ and $\sin$, but in $\mathbb{C}$ they are related by a simple rotation: $\cosh z = \cos(iz)$. This unification is one of the elegant simplifications that complex analysis provides.

---

## 1. Definitions

$$\boxed{\cosh z = \frac{e^z + e^{-z}}{2}, \qquad \sinh z = \frac{e^z - e^{-z}}{2}}$$

$$\tanh z = \frac{\sinh z}{\cosh z}, \qquad \text{etc.}$$

Both $\cosh z$ and $\sinh z$ are **entire** functions.

---

## 2. Connection to Trigonometric Functions

$$\boxed{\cosh(iz) = \cos z, \qquad \sinh(iz) = i\sin z}$$

$$\cos(iz) = \cosh z, \qquad \sin(iz) = i\sinh z$$

This means: rotating the argument by $\pi/2$ converts hyperbolic to trigonometric (and vice versa).

| Trig Identity | ↔ | Hyperbolic Identity |
|---------------|---|---------------------|
| $\cos^2 z + \sin^2 z = 1$ | | $\cosh^2 z - \sinh^2 z = 1$ |
| $\cos(z_1+z_2) = \cdots$ | | $\cosh(z_1+z_2) = \cdots$ |
| $(\sin z)' = \cos z$ | | $(\sinh z)' = \cosh z$ |
| $(\cos z)' = -\sin z$ | | $(\cosh z)' = \sinh z$ |

---

## 3. Real and Imaginary Parts

For $z = x + iy$:

$$\cosh z = \cosh x \cos y + i\sinh x \sin y$$

$$\sinh z = \sinh x \cos y + i\cosh x \sin y$$

### 3.1 Modulus

$$|\cosh z|^2 = \sinh^2 x + \cos^2 y = \cosh^2 x - \sin^2 y$$

$$|\sinh z|^2 = \sinh^2 x + \sin^2 y$$

---

## 4. Properties

| Property | Formula |
|----------|---------|
| Derivatives | $(\cosh z)' = \sinh z$, $(\sinh z)' = \cosh z$ |
| Fundamental identity | $\cosh^2 z - \sinh^2 z = 1$ |
| Period | $2\pi i$ |
| Zeros of $\sinh z$ | $z = n\pi i$, $n \in \mathbb{Z}$ |
| Zeros of $\cosh z$ | $z = (n + 1/2)\pi i$, $n \in \mathbb{Z}$ |
| $\cosh$ parity | $\cosh(-z) = \cosh z$ (even) |
| $\sinh$ parity | $\sinh(-z) = -\sinh z$ (odd) |

---

## 5. Design Example

### Example: Show that $\cosh^2 z - \sinh^2 z = 1$.

$$\cosh^2 z - \sinh^2 z = \left(\frac{e^z+e^{-z}}{2}\right)^2 - \left(\frac{e^z-e^{-z}}{2}\right)^2$$

$$= \frac{e^{2z}+2+e^{-2z}}{4} - \frac{e^{2z}-2+e^{-2z}}{4} = \frac{4}{4} = 1 \qquad \checkmark$$

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| $\cosh z$ | $(e^z + e^{-z})/2$ |
| $\sinh z$ | $(e^z - e^{-z})/2$ |
| Trig connection | $\cosh(iz) = \cos z$, $\sinh(iz) = i\sin z$ |
| Identity | $\cosh^2 z - \sinh^2 z = 1$ |
| Period | $2\pi i$ |
| Zeros of $\sinh$ | $z = n\pi i$ |
| Zeros of $\cosh$ | $z = (n+\frac{1}{2})\pi i$ |

---

## Quick Revision Questions

1. **Derive** the relation $\cos(iz) = \cosh z$ from the definitions.

2. **Find** all zeros of $\cosh z$.

3. **Prove** $\cosh^2 z - \sinh^2 z = 1$.

4. **Compute** $\sinh(i\pi/4)$.

5. **Show** that $\cosh z$ and $\sinh z$ have period $2\pi i$.

6. **Decompose** $\tanh(x + iy)$ into real and imaginary parts.

---

[← Previous: Trigonometric Functions](02-trigonometric-functions.md) | [Next: Logarithmic Function →](04-logarithmic-function.md)
