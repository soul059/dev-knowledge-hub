# 1.3 De Moivre's Theorem

[← Previous: Polar Form and Euler's Formula](02-polar-form-eulers-formula.md) | [Next: Roots of Complex Numbers →](04-roots-of-complex-numbers.md)

---

## Chapter Overview

De Moivre's theorem provides a powerful shortcut for computing powers of complex numbers and is the key tool for finding $n$th roots of complex numbers. It also connects naturally to multiple-angle trigonometric identities.

---

## 1. Statement of the Theorem

### 1.1 De Moivre's Theorem

For any integer $n$ and any $\theta \in \mathbb{R}$:

$$\boxed{(\cos\theta + i\sin\theta)^n = \cos(n\theta) + i\sin(n\theta)}$$

Equivalently, using exponential notation:

$$(e^{i\theta})^n = e^{in\theta}$$

### 1.2 General Power Form

For $z = r(\cos\theta + i\sin\theta)$:

$$z^n = r^n (\cos(n\theta) + i\sin(n\theta))$$

```
Powers on the Complex Plane — z = e^(iπ/4):

          Im
           ▲
           │  z² = e^(iπ/2) = i
     z³ •  │  • z²
        ╲  │ ╱
         ╲ │╱        z = e^(iπ/4)
  ────────O──────•──► Re
         ╱ │         (on unit circle)
        ╱  │
     z⁷ •  │  • z⁶
           │
           │  z⁵ = e^(i·5π/4)

  Each power rotates by π/4 around the unit circle
```

---

## 2. Proof

### 2.1 Proof for Positive Integers (by Induction)

**Base case** ($n = 1$): Trivially true.

**Inductive step**: Assume $(\cos\theta + i\sin\theta)^k = \cos(k\theta) + i\sin(k\theta)$.

Then:
$$(\cos\theta + i\sin\theta)^{k+1} = (\cos\theta + i\sin\theta)^k \cdot (\cos\theta + i\sin\theta)$$

$$= (\cos(k\theta) + i\sin(k\theta))(\cos\theta + i\sin\theta)$$

Expanding via the multiplication rule for polar forms:

$$= \cos(k\theta)\cos\theta - \sin(k\theta)\sin\theta + i[\cos(k\theta)\sin\theta + \sin(k\theta)\cos\theta]$$

$$= \cos((k+1)\theta) + i\sin((k+1)\theta) \qquad \checkmark$$

### 2.2 Extension to Negative Integers

For $n = -m$ where $m > 0$:

$$(\cos\theta + i\sin\theta)^{-m} = \frac{1}{(\cos\theta + i\sin\theta)^m} = \frac{1}{\cos(m\theta) + i\sin(m\theta)}$$

$$= \cos(m\theta) - i\sin(m\theta) = \cos(-m\theta) + i\sin(-m\theta) \qquad \checkmark$$

### 2.3 Proof via Euler's Formula (Immediate)

$$(\cos\theta + i\sin\theta)^n = (e^{i\theta})^n = e^{in\theta} = \cos(n\theta) + i\sin(n\theta)$$

---

## 3. Applications to Trigonometric Identities

### 3.1 Method

By expanding $(\cos\theta + i\sin\theta)^n$ using the binomial theorem and comparing real and imaginary parts with $\cos(n\theta) + i\sin(n\theta)$, we obtain multiple-angle formulas.

### 3.2 Double Angle Formulas ($n = 2$)

$$(\cos\theta + i\sin\theta)^2 = \cos^2\theta - \sin^2\theta + 2i\sin\theta\cos\theta$$

Comparing with $\cos(2\theta) + i\sin(2\theta)$:

$$\cos(2\theta) = \cos^2\theta - \sin^2\theta$$
$$\sin(2\theta) = 2\sin\theta\cos\theta$$

### 3.3 Triple Angle Formulas ($n = 3$)

$$(\cos\theta + i\sin\theta)^3 = \cos^3\theta + 3i\cos^2\theta\sin\theta - 3\cos\theta\sin^2\theta - i\sin^3\theta$$

Real part:
$$\cos(3\theta) = \cos^3\theta - 3\cos\theta\sin^2\theta = 4\cos^3\theta - 3\cos\theta$$

Imaginary part:
$$\sin(3\theta) = 3\cos^2\theta\sin\theta - \sin^3\theta = 3\sin\theta - 4\sin^3\theta$$

### 3.4 General Pattern

| $n$ | $\cos(n\theta)$ | $\sin(n\theta)$ |
|-----|-----------------|-----------------|
| 1 | $\cos\theta$ | $\sin\theta$ |
| 2 | $\cos^2\theta - \sin^2\theta$ | $2\sin\theta\cos\theta$ |
| 3 | $4\cos^3\theta - 3\cos\theta$ | $3\sin\theta - 4\sin^3\theta$ |
| 4 | $8\cos^4\theta - 8\cos^2\theta + 1$ | $4\cos^3\theta\sin\theta - 4\cos\theta\sin^3\theta$ |

---

## 4. Powers of Complex Numbers

### 4.1 Computing $z^n$ Efficiently

Given $z = a + bi$, computing $z^n$ directly requires expanding $(a+bi)^n$. Using De Moivre:

1. Convert to polar: $z = re^{i\theta}$
2. Apply: $z^n = r^n e^{in\theta}$
3. Convert back if needed

### 4.2 Worked Example

**Compute** $(1 + i)^{10}$.

**Step 1.** Polar form: $|1+i| = \sqrt{2}$, $\arg(1+i) = \pi/4$
$$1 + i = \sqrt{2}\,e^{i\pi/4}$$

**Step 2.** Apply De Moivre:
$$(1+i)^{10} = (\sqrt{2})^{10}\, e^{i \cdot 10\pi/4} = 2^5\, e^{i \cdot 5\pi/2}$$

**Step 3.** Simplify the argument: $5\pi/2 = 2\pi + \pi/2$, so $e^{i5\pi/2} = e^{i\pi/2} = i$

$$(1+i)^{10} = 32i$$

**Verification**: $(1+i)^2 = 2i$, so $(1+i)^{10} = (2i)^5 = 32i^5 = 32i$ ✓

---

## 5. Expressing $\cos^n\theta$ and $\sin^n\theta$ in Terms of Multiple Angles

### 5.1 The Reverse Problem

While De Moivre gives $\cos(n\theta)$ in terms of powers of $\cos\theta$, we often need the reverse: express $\cos^n\theta$ or $\sin^n\theta$ as a sum of cosines/sines of multiple angles.

### 5.2 Method Using $z = e^{i\theta}$

Let $z = e^{i\theta}$. Then:
- $z + z^{-1} = 2\cos\theta \implies \cos\theta = \frac{z + z^{-1}}{2}$
- $z - z^{-1} = 2i\sin\theta \implies \sin\theta = \frac{z - z^{-1}}{2i}$
- $z^n + z^{-n} = 2\cos(n\theta)$
- $z^n - z^{-n} = 2i\sin(n\theta)$

### 5.3 Example: Express $\cos^4\theta$ in terms of multiple angles

$$\cos^4\theta = \left(\frac{z + z^{-1}}{2}\right)^4 = \frac{1}{16}(z + z^{-1})^4$$

Expanding using the binomial theorem:

$$(z + z^{-1})^4 = z^4 + 4z^2 + 6 + 4z^{-2} + z^{-4}$$

$$= (z^4 + z^{-4}) + 4(z^2 + z^{-2}) + 6 = 2\cos(4\theta) + 8\cos(2\theta) + 6$$

Therefore:

$$\cos^4\theta = \frac{1}{16}[2\cos(4\theta) + 8\cos(2\theta) + 6] = \frac{1}{8}\cos(4\theta) + \frac{1}{2}\cos(2\theta) + \frac{3}{8}$$

---

## 6. Design Example — Identity Derivation

### Example: Express $\sin^3\theta$ as a sum of sines of multiple angles.

**Step 1.** Write $\sin\theta = \frac{z - z^{-1}}{2i}$ where $z = e^{i\theta}$

**Step 2.** Cube it:
$$\sin^3\theta = \left(\frac{z - z^{-1}}{2i}\right)^3 = \frac{(z - z^{-1})^3}{(2i)^3} = \frac{(z - z^{-1})^3}{-8i}$$

**Step 3.** Expand $(z - z^{-1})^3 = z^3 - 3z + 3z^{-1} - z^{-3}$:

$$= (z^3 - z^{-3}) - 3(z - z^{-1}) = 2i\sin(3\theta) - 3 \cdot 2i\sin\theta$$

**Step 4.** Substitute:
$$\sin^3\theta = \frac{2i\sin(3\theta) - 6i\sin\theta}{-8i} = \frac{-\sin(3\theta) + 3\sin\theta}{4}$$

$$\boxed{\sin^3\theta = \frac{3\sin\theta - \sin(3\theta)}{4}}$$

---

## 7. Real-World Applications

| Application | How De Moivre's Theorem Helps |
|-------------|-------------------------------|
| **Signal Processing** | Decomposing products of sinusoids into sums (linearization) |
| **Antenna Arrays** | Computing array factors which involve sums of $e^{in\theta}$ |
| **Vibration Analysis** | Expressing powers of oscillatory terms as Fourier-like sums |
| **Chebyshev Polynomials** | $T_n(\cos\theta) = \cos(n\theta)$ — directly from De Moivre |

### Chebyshev Connection

The Chebyshev polynomial of the first kind $T_n(x)$ satisfies:

$$T_n(\cos\theta) = \cos(n\theta)$$

From De Moivre's theorem, $\cos(n\theta) = \text{Re}[(\cos\theta + i\sin\theta)^n]$, which is a polynomial of degree $n$ in $\cos\theta$.

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| De Moivre's Theorem | $(\cos\theta + i\sin\theta)^n = \cos(n\theta) + i\sin(n\theta)$ |
| Exponential form | $(e^{i\theta})^n = e^{in\theta}$ |
| Power of $z$ | $z^n = r^n e^{in\theta}$ |
| Double angle (cos) | $\cos(2\theta) = \cos^2\theta - \sin^2\theta$ |
| Double angle (sin) | $\sin(2\theta) = 2\sin\theta\cos\theta$ |
| Triple angle (cos) | $\cos(3\theta) = 4\cos^3\theta - 3\cos\theta$ |
| $\cos^n\theta$ method | Use $\cos\theta = \frac{z + z^{-1}}{2}$, expand, regroup |
| $\sin^n\theta$ method | Use $\sin\theta = \frac{z - z^{-1}}{2i}$, expand, regroup |
| Chebyshev connection | $T_n(\cos\theta) = \cos(n\theta)$ |

---

## Quick Revision Questions

1. **State** De Moivre's theorem and prove it for positive integers by induction.

2. **Compute** $(1 - i)^8$ using De Moivre's theorem.

3. **Derive** the formula for $\cos(4\theta)$ in terms of $\cos\theta$ using De Moivre.

4. **Express** $\sin^4\theta$ as a sum of cosines of multiple angles.

5. **Explain** why De Moivre's theorem does not directly apply for non-integer exponents and what complications arise.

6. **Show** that $\cos(5\theta) = 16\cos^5\theta - 20\cos^3\theta + 5\cos\theta$ using De Moivre's theorem.

---

[← Previous: Polar Form and Euler's Formula](02-polar-form-eulers-formula.md) | [Next: Roots of Complex Numbers →](04-roots-of-complex-numbers.md)
