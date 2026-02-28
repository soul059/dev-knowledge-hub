# 7.2 Linear Fractional (Möbius) Transformations

[← Previous: Conformal Maps](01-conformal-maps.md) | [Next: Mapping of Standard Regions →](03-mapping-of-standard-regions.md)

---

## Chapter Overview

**Linear fractional transformations** (LFTs, also called **Möbius transformations**) are the most important class of conformal maps. They form a group, map circles/lines to circles/lines, and are completely determined by three points. They are the automorphisms of the Riemann sphere.

---

## 1. Definition

### 1.1 Formula

A **Möbius transformation** is:

$$\boxed{w = T(z) = \frac{az + b}{cz + d}, \quad ad - bc \neq 0}$$

where $a, b, c, d \in \mathbb{C}$.

The condition $ad - bc \neq 0$ ensures $T$ is not constant.

### 1.2 Matrix Representation

Associate $T \leftrightarrow \begin{pmatrix} a & b \\ c & d \end{pmatrix}$.

Composition of Möbius transformations corresponds to matrix multiplication!

$$T_2 \circ T_1 \leftrightarrow M_2 M_1$$

---

## 2. Properties

### 2.1 Key Properties Table

| Property | Statement |
|----------|-----------|
| **Conformal** | $T'(z) = \frac{ad-bc}{(cz+d)^2} \neq 0$ everywhere |
| **Bijective** | $T: \hat{\mathbb{C}} \to \hat{\mathbb{C}}$ is a bijection |
| **Inverse** | $T^{-1}(w) = \frac{dw - b}{-cw + a}$ (also Möbius) |
| **Composition** | $T_2 \circ T_1$ is Möbius |
| **Group** | The set of all Möbius transformations forms a group |
| **Circles ↔ circles** | Maps circles and lines (= circles through $\infty$) to circles and lines |
| **Three-point determination** | Uniquely determined by $T(z_1) = w_1$, $T(z_2) = w_2$, $T(z_3) = w_3$ |

### 2.2 Action on $\hat{\mathbb{C}}$

$$T(\infty) = \frac{a}{c}, \quad T\left(-\frac{d}{c}\right) = \infty$$

(If $c = 0$: $T(\infty) = \infty$, and $T$ is affine: $w = (a/d)z + b/d$.)

---

## 3. Special Types

| Type | Formula | Condition |
|------|---------|-----------|
| **Translation** | $w = z + b$ | $a = d = 1$, $c = 0$ |
| **Rotation** | $w = e^{i\theta}z$ | $|a| = 1$, $b = c = 0$, $d = 1$ |
| **Dilation** | $w = kz$, $k > 0$ | $b = c = 0$ |
| **Inversion** | $w = 1/z$ | $a = d = 0$, $b = c = 1$ |

**Decomposition:** Every Möbius transformation is a composition of translations, rotations, dilations, and inversions.

---

## 4. The Cross-Ratio

### 4.1 Definition

The **cross-ratio** of four points $z_1, z_2, z_3, z_4$:

$$\boxed{(z_1, z_2; z_3, z_4) = \frac{(z_1 - z_3)(z_2 - z_4)}{(z_1 - z_4)(z_2 - z_3)}}$$

### 4.2 Invariance

Cross-ratios are **invariant** under Möbius transformations:

$$\left(T(z_1), T(z_2); T(z_3), T(z_4)\right) = (z_1, z_2; z_3, z_4)$$

### 4.3 Finding the Map via Cross-Ratio

To find $T$ mapping $z_1 \mapsto w_1$, $z_2 \mapsto w_2$, $z_3 \mapsto w_3$, solve:

$$\frac{(w - w_1)(w_2 - w_3)}{(w - w_3)(w_2 - w_1)} = \frac{(z - z_1)(z_2 - z_3)}{(z - z_3)(z_2 - z_1)}$$

---

## 5. Circles and Lines

### 5.1 The Fundamental Theorem

> Every Möbius transformation maps **circles and lines** to **circles and lines** (viewing lines as circles through $\infty$).

### 5.2 Determining the Image

To find the image of a circle/line:
1. Pick 3 points on it.
2. Apply $T$ to each.
3. The image circle/line passes through these 3 image points.

```
Möbius maps take circles/lines to circles/lines:

  z-plane:                     w-plane:

    ╭───╮                       │
   │     │  circle    ──T──►    │  line (if circle
    ╰───╯                       │  passes through
                                │  the pole)

    ──────  line      ──T──►   ╭───╮  circle (if line
                               │     │  doesn't pass
                                ╰───╯  through pole)
```

---

## 6. Important Möbius Transformations

### 6.1 Cayley Transform: UHP → Disk

$$w = \frac{z - i}{z + i}$$

| $z$-plane | $w$-plane |
|-----------|-----------|
| Upper half-plane $\mathbb{H}$ | Unit disk $\mathbb{D}$ |
| Real axis | Unit circle $|w| = 1$ |
| $z = i$ | $w = 0$ |
| $z = \infty$ | $w = 1$ |

### 6.2 Disk Automorphisms

Every biholomorphism $\mathbb{D} \to \mathbb{D}$ is:

$$T(z) = e^{i\theta}\frac{z - a}{1 - \bar{a}z}, \quad |a| < 1, \quad \theta \in \mathbb{R}$$

### 6.3 UHP Automorphisms

$$T(z) = \frac{az + b}{cz + d}, \quad a,b,c,d \in \mathbb{R}, \quad ad - bc > 0$$

---

## 7. Design Examples

### Example 1: Find the Möbius transformation mapping $0 \mapsto 1$, $1 \mapsto i$, $\infty \mapsto -1$.

Using the cross-ratio formula:

$$\frac{(w-1)(i-(-1))}{(w-(-1))(i-1)} = \frac{(z-0)(1-\infty)}{(z-\infty)(1-0)}$$

For $z_3 = \infty$: $\frac{z - z_1}{z - z_3}\cdot\frac{z_2 - z_3}{z_2 - z_1} \to \frac{z - 0}{1 - 0} = z$

$$\frac{(w-1)(i+1)}{(w+1)(i-1)} = z$$

Solve for $w$:

$$w = \frac{z(i-1) + (i+1)}{z(i+1) + (i-1)} \cdot \frac{(-1)}{(-1)} = \frac{(1-i)z + (1+i)}{(1+i)z + (1-i)} \cdot \frac{(?)}{(?)}$$

After simplification: $w = \frac{(i-1)z + (i+1)}{(i+1)z + (i-1)}$.

**Verify:** $T(0) = \frac{i+1}{i-1} = \frac{(i+1)(−i−1)}{(i-1)(−i−1)} = \frac{-i-1-1+i}{-i^2-i+i+1} = \frac{-2}{2} = -1$... 

Let me redo more carefully. Cross-ratio:

$$\frac{(w-1)(i+1)}{(w+1)(i-1)} = z$$

$$(w-1)(i+1) = z(w+1)(i-1)$$

$$w(i+1) - (i+1) = zw(i-1) + z(i-1)$$

$$w[(i+1) - z(i-1)] = (i+1) + z(i-1)$$

$$w = \frac{(i+1) + z(i-1)}{(i+1) - z(i-1)}$$

**Verify:** $T(0) = \frac{i+1}{i+1} = 1$ ✓, $T(1) = \frac{(i+1)+(i-1)}{(i+1)-(i-1)} = \frac{2i}{2} = i$ ✓, $T(\infty) = \frac{i-1}{-(i-1)} = -1$ ✓

### Example 2: Map the disk $|z| < 1$ to itself with $z = 1/2 \mapsto 0$.

Use the disk automorphism formula with $a = 1/2$:

$$w = \frac{z - 1/2}{1 - z/2} = \frac{2z - 1}{2 - z}$$

---

## 8. Real-World Applications

| Application | Möbius Role |
|-------------|------------|
| **Optics** | Optical transfer matrices (ABCD matrices, same algebra) |
| **Circuitry** | Smith chart = Möbius transform of impedance plane |
| **Hyperbolic geometry** | Isometries of hyperbolic plane = Möbius on UHP |
| **Special relativity** | Lorentz group acts as Möbius on celestial sphere |
| **Computer graphics** | Conformal texture mapping |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Möbius transformation | $w = (az+b)/(cz+d)$, $ad-bc \neq 0$ |
| Group structure | Composition = matrix multiplication |
| Three-point determination | Unique $T$ mapping $z_1, z_2, z_3 \mapsto w_1, w_2, w_3$ |
| Cross-ratio | $(z_1,z_2;z_3,z_4)$ invariant under Möbius |
| Circle/line preservation | Circles and lines map to circles and lines |
| Cayley transform | $(z-i)/(z+i)$: UHP → disk |
| Disk automorphisms | $e^{i\theta}(z-a)/(1-\bar{a}z)$ |

---

## Quick Revision Questions

1. **Define** Möbius transformation and state its key properties.

2. **Find** the Möbius transformation mapping $1 \mapsto 0$, $i \mapsto 1$, $-1 \mapsto \infty$.

3. **Prove** that Möbius transformations map circles/lines to circles/lines.

4. **Show** that the cross-ratio is invariant under Möbius transformations.

5. **Find** a Möbius transformation mapping the upper half-plane to the unit disk.

6. **Describe** all automorphisms (biholomorphisms) of the unit disk.

---

[← Previous: Conformal Maps](01-conformal-maps.md) | [Next: Mapping of Standard Regions →](03-mapping-of-standard-regions.md)
