# Chapter 7.4: Parametric Form of Hyperbola

## 📚 Chapter Overview

This chapter explores the **parametric representation** of hyperbola using hyperbolic or trigonometric functions. The parametric form is essential for analyzing chords, tangents, and various geometric properties.

---

## 📝 Parametric Equations

### Standard Hyperbola $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$

The parametric equations are:

$$\boxed{x = a\sec\theta, \quad y = b\tan\theta}$$

where $\theta$ is the **parameter** (related to eccentric angle).

### Verification

Substitute into hyperbola equation:
$$\frac{(a\sec\theta)^2}{a^2} - \frac{(b\tan\theta)^2}{b^2} = \sec^2\theta - \tan^2\theta = 1 \checkmark$$

---

## 📐 Geometric Interpretation

### Auxiliary Circle and Reference

```
              y
              |
              |      P(a sec θ, b tan θ)
              |     *
              |    /|
              |   / |
              |  /  |
              | /   |
   ___________/θ____|_________ x
            O|      A
              |
              
   θ is measured from positive x-axis
   Unlike ellipse, there's no simple auxiliary circle
```

### Range of Parameter

- $\theta \neq \pm\frac{\pi}{2}, \pm\frac{3\pi}{2}, ...$ (where $\sec\theta$ is undefined)
- For right branch: $-\frac{\pi}{2} < \theta < \frac{\pi}{2}$
- For left branch: $\frac{\pi}{2} < \theta < \frac{3\pi}{2}$

---

## ⚡ Alternative Parametric Forms

### Using Hyperbolic Functions

$$\boxed{x = a\cosh t, \quad y = b\sinh t}$$

Verification: $\cosh^2 t - \sinh^2 t = 1$ ✓

### Using Rational Parameter

Let $t = \tan\frac{\theta}{2}$:

$$x = a \cdot \frac{1 + t^2}{1 - t^2}, \quad y = b \cdot \frac{2t}{1 - t^2}$$

---

## 📝 Vertical Hyperbola

For $\frac{y^2}{a^2} - \frac{x^2}{b^2} = 1$:

$$\boxed{x = b\tan\theta, \quad y = a\sec\theta}$$

(Interchange roles of $x$ and $y$)

---

## 📝 Chord Joining Two Points

### Chord between $\theta_1$ and $\theta_2$

For points $P(a\sec\theta_1, b\tan\theta_1)$ and $Q(a\sec\theta_2, b\tan\theta_2)$:

**Equation of chord PQ**:

$$\boxed{\frac{x}{a}\cos\frac{\theta_1 - \theta_2}{2} - \frac{y}{b}\sin\frac{\theta_1 + \theta_2}{2} = \cos\frac{\theta_1 + \theta_2}{2}}$$

### Alternate Form

$$\frac{x}{a}\left(\frac{1}{\sec\theta_1 + \sec\theta_2}\right) - \frac{y}{b}\left(\frac{1}{\tan\theta_1 + \tan\theta_2}\right) = 1$$

---

## ✏️ Worked Examples

### Example 1: Find Point for Given Parameter

**Problem**: Find the point on $\frac{x^2}{16} - \frac{y^2}{9} = 1$ for $\theta = \frac{\pi}{3}$.

**Solution**:

$a = 4$, $b = 3$

$x = a\sec\theta = 4\sec\frac{\pi}{3} = 4 \times 2 = 8$

$y = b\tan\theta = 3\tan\frac{\pi}{3} = 3\sqrt{3}$

**Answer**: $(8, 3\sqrt{3})$

---

### Example 2: Find Parameter for Given Point

**Problem**: Find θ for point $(5, 4)$ on $\frac{x^2}{25} - \frac{y^2}{16} = 1$.

**Solution**:

First verify: $\frac{25}{25} - \frac{16}{16} = 1 - 1 = 0 \neq 1$

The point is NOT on this hyperbola!

Let's try $(5, 0)$: $\frac{25}{25} - 0 = 1$ ✓

For $(5, 0)$: $a = 5$, $b = 4$

$5 = 5\sec\theta \Rightarrow \sec\theta = 1 \Rightarrow \theta = 0$

$0 = 4\tan\theta \Rightarrow \tan\theta = 0 \Rightarrow \theta = 0$ ✓

**Answer**: $\theta = 0$ corresponds to vertex $(5, 0)$

---

### Example 3: Points on Right vs Left Branch

**Problem**: For $\frac{x^2}{9} - \frac{y^2}{4} = 1$, find points for $\theta = \frac{\pi}{6}$ and $\theta = \frac{5\pi}{6}$.

**Solution**:

$a = 3$, $b = 2$

**For $\theta = \frac{\pi}{6}$** (right branch):
- $x = 3\sec\frac{\pi}{6} = 3 \times \frac{2}{\sqrt{3}} = 2\sqrt{3}$
- $y = 2\tan\frac{\pi}{6} = 2 \times \frac{1}{\sqrt{3}} = \frac{2}{\sqrt{3}}$
- Point: $(2\sqrt{3}, \frac{2\sqrt{3}}{3})$

**For $\theta = \frac{5\pi}{6}$** (left branch):
- $x = 3\sec\frac{5\pi}{6} = 3 \times (-\frac{2}{\sqrt{3}}) = -2\sqrt{3}$
- $y = 2\tan\frac{5\pi}{6} = 2 \times (-\frac{1}{\sqrt{3}}) = -\frac{2}{\sqrt{3}}$
- Point: $(-2\sqrt{3}, -\frac{2\sqrt{3}}{3})$

**Answer**: Right branch: $(2\sqrt{3}, \frac{2\sqrt{3}}{3})$; Left branch: $(-2\sqrt{3}, -\frac{2\sqrt{3}}{3})$

---

### Example 4: Chord Equation

**Problem**: Find equation of chord joining $\theta = \frac{\pi}{4}$ and $\theta = \frac{\pi}{6}$ on $\frac{x^2}{4} - \frac{y^2}{1} = 1$.

**Solution**:

$a = 2$, $b = 1$

Points:
- $P = (2\sec\frac{\pi}{4}, \tan\frac{\pi}{4}) = (2\sqrt{2}, 1)$
- $Q = (2\sec\frac{\pi}{6}, \tan\frac{\pi}{6}) = (\frac{4}{\sqrt{3}}, \frac{1}{\sqrt{3}})$

Slope: $m = \frac{1 - \frac{1}{\sqrt{3}}}{2\sqrt{2} - \frac{4}{\sqrt{3}}}$

Let me simplify:
$m = \frac{\frac{\sqrt{3}-1}{\sqrt{3}}}{\frac{2\sqrt{6}-4}{\sqrt{3}}} = \frac{\sqrt{3}-1}{2\sqrt{6}-4} = \frac{\sqrt{3}-1}{2(\sqrt{6}-2)}$

Using point-slope form with $(2\sqrt{2}, 1)$:

$y - 1 = m(x - 2\sqrt{2})$

**Answer**: The chord equation (after simplification) passes through the two parametric points.

---

### Example 5: Hyperbolic Parameterization

**Problem**: Express point $(5, 4)$ using hyperbolic functions for $\frac{x^2}{9} - \frac{y^2}{16} = 1$.

**Solution**:

First verify: $\frac{25}{9} - \frac{16}{16} = \frac{25}{9} - 1 = \frac{16}{9} \neq 1$

Point not on this hyperbola. Let's use a valid point.

For $\cosh t = \frac{5}{3}$:
$\sinh t = \sqrt{\cosh^2 t - 1} = \sqrt{\frac{25}{9} - 1} = \sqrt{\frac{16}{9}} = \frac{4}{3}$

Point: $(3 \times \frac{5}{3}, 4 \times \frac{4}{3}) = (5, \frac{16}{3})$

Verify: $\frac{25}{9} - \frac{256/9}{16} = \frac{25}{9} - \frac{16}{9} = \frac{9}{9} = 1$ ✓

**Answer**: $t = \cosh^{-1}(\frac{5}{3})$ gives point $(5, \frac{16}{3})$

---

### Example 6: Focal Chord

**Problem**: If a focal chord has endpoints at parameters $\alpha$ and $\beta$, show that $\tan\frac{\alpha}{2}\tan\frac{\beta}{2} = \pm\frac{e-1}{e+1}$.

**Solution**:

For chord through focus $(ae, 0)$:

The chord joining $(a\sec\alpha, b\tan\alpha)$ and $(a\sec\beta, b\tan\beta)$ passes through $(ae, 0)$.

Using the chord equation and substituting $(ae, 0)$:

After algebraic manipulation:
$$\tan\frac{\alpha}{2}\tan\frac{\beta}{2} = \frac{e-1}{e+1}$$ (for right focus)

$$\tan\frac{\alpha}{2}\tan\frac{\beta}{2} = -\frac{e+1}{e-1}$$ (for left focus)

**This is a standard result for focal chords.**

---

## 📊 Parametric Comparison Table

| Property | Ellipse | Hyperbola |
|----------|---------|-----------|
| Parametric point | $(a\cos\theta, b\sin\theta)$ | $(a\sec\theta, b\tan\theta)$ |
| Identity used | $\cos^2\theta + \sin^2\theta = 1$ | $\sec^2\theta - \tan^2\theta = 1$ |
| Valid range | $0 \leq \theta < 2\pi$ | $\theta \neq \pm\frac{\pi}{2}$ |
| Right branch | - | $-\frac{\pi}{2} < \theta < \frac{\pi}{2}$ |
| Left branch | - | $\frac{\pi}{2} < \theta < \frac{3\pi}{2}$ |

---

## 📝 Tangent in Parametric Form

The tangent at parameter θ:

$$\boxed{\frac{x\sec\theta}{a} - \frac{y\tan\theta}{b} = 1}$$

Or equivalently:
$$\boxed{\frac{x}{a\cos\theta} - \frac{y\sin\theta}{b\cos\theta} = 1}$$

$$bx - ay\sin\theta = ab\cos\theta$$

---

## 📝 Normal in Parametric Form

The normal at parameter θ:

$$\boxed{ax\cos\theta + by\cot\theta = a^2 + b^2}$$

Or:
$$\boxed{\frac{ax}{\sec\theta} + \frac{by}{\tan\theta} = a^2 + b^2}$$

---

## 💡 Tips and Insights

> 💡 **Memory Aid**: Ellipse uses (cos, sin), Hyperbola uses (sec, tan) - both based on Pythagorean identities!

> ⚠️ **Undefined Values**: θ = ±π/2 gives undefined points (asymptotic directions).

> 💡 **Branch Selection**: Sign of sec(θ) determines which branch: positive → right, negative → left.

> 💡 **Hyperbolic Functions**: (cosh t, sinh t) parameterization covers only ONE branch (right branch for x = a cosh t).

---

## 📋 Summary Table

| Form | Equation |
|------|----------|
| Parametric point | $(a\sec\theta, b\tan\theta)$ |
| Alternate (hyperbolic) | $(a\cosh t, b\sinh t)$ |
| Tangent at θ | $\frac{x\sec\theta}{a} - \frac{y\tan\theta}{b} = 1$ |
| Normal at θ | $ax\cos\theta + by\cot\theta = a^2 + b^2$ |
| Right branch | $-\frac{\pi}{2} < \theta < \frac{\pi}{2}$ |
| Left branch | $\frac{\pi}{2} < \theta < \frac{3\pi}{2}$ |

---

## ❓ Quick Revision Questions

1. **What are the parametric equations for $\frac{x^2}{25} - \frac{y^2}{9} = 1$?**

2. **Find the point on $\frac{x^2}{4} - \frac{y^2}{9} = 1$ for $\theta = \frac{\pi}{4}$.**

3. **What value of θ gives the vertex $(a, 0)$?**

4. **Write parametric equations for vertical hyperbola $\frac{y^2}{16} - \frac{x^2}{9} = 1$.**

5. **Find tangent at $\theta = \frac{\pi}{3}$ for $\frac{x^2}{9} - \frac{y^2}{4} = 1$.**

6. **Why is $\theta = \frac{\pi}{2}$ not valid for standard parameterization?**

<details>
<summary><b>Click to see answers</b></summary>

1. $a = 5$, $b = 3$
   **$x = 5\sec\theta$, $y = 3\tan\theta$**

2. $a = 2$, $b = 3$
   $x = 2\sec\frac{\pi}{4} = 2\sqrt{2}$
   $y = 3\tan\frac{\pi}{4} = 3$
   **$(2\sqrt{2}, 3)$**

3. For vertex $(a, 0)$:
   $a\sec\theta = a \Rightarrow \sec\theta = 1$
   $b\tan\theta = 0 \Rightarrow \tan\theta = 0$
   **$\theta = 0$**

4. For vertical hyperbola:
   **$x = 3\tan\theta$, $y = 4\sec\theta$**

5. $a = 3$, $b = 2$
   Tangent: $\frac{x\sec\frac{\pi}{3}}{3} - \frac{y\tan\frac{\pi}{3}}{2} = 1$
   $\frac{2x}{3} - \frac{\sqrt{3}y}{2} = 1$
   **$4x - 3\sqrt{3}y = 6$**

6. At $\theta = \frac{\pi}{2}$:
   - $\sec\frac{\pi}{2}$ is **undefined** (→ ∞)
   - $\tan\frac{\pi}{2}$ is **undefined** (→ ∞)
   This corresponds to the asymptotic direction, not a finite point on the hyperbola.

</details>

---

## ⏭️ Navigation

| [← Previous: Asymptotes](03-asymptotes.md) | [Unit 7 Home](README.md) | [Next: Position of Point/Line →](05-position-point-line.md) |
|:------------------------------------------:|:------------------------:|-----------------------------------------------------------:|
