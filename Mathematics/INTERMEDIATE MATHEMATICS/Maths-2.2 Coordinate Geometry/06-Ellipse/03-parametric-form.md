# Chapter 6.3: Parametric Form of Ellipse

## 📚 Chapter Overview

The **parametric form** provides an elegant way to represent points on an ellipse using a single parameter (eccentric angle). This approach simplifies many calculations and connects the ellipse to its **auxiliary circle**.

---

## 📝 Auxiliary Circle

### Definition

The **auxiliary circle** of an ellipse is the circle with center at the ellipse center and radius equal to the semi-major axis.

For ellipse $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$ (a > b):

$$\boxed{\text{Auxiliary Circle: } x^2 + y^2 = a^2}$$

```
                    Auxiliary Circle (radius a)
                          ╭─────╮
                      ╱           ╲
                    ╱               ╲
            Q●────/─────────────────\────
               ╱ │                   ╲
              │  │P                   │
              │  ●                    │
    ──────────●──┼────────────────────●──────────
              │  │                    │
              │  │                    │
               ╲ │                   ╱
                ╲│                 ╱
                  ╲             ╱
                    ╲─────────╱
                    
    Q is on auxiliary circle
    P is on ellipse, directly below Q
    Both have same x-coordinate
```

---

## 📐 Eccentric Angle

### Definition

For a point P on the ellipse, draw perpendicular to major axis meeting the auxiliary circle at Q. The angle θ that OQ makes with positive x-axis is the **eccentric angle** of P.

### Parametric Coordinates

$$\boxed{P = (a\cos\theta, b\sin\theta)}$$

where $\theta \in [0, 2\pi)$ is the eccentric angle.

### Derivation

Point Q on auxiliary circle: $(a\cos\theta, a\sin\theta)$

Point P has same x-coordinate as Q: $x = a\cos\theta$

From ellipse equation:
$$\frac{a^2\cos^2\theta}{a^2} + \frac{y^2}{b^2} = 1$$
$$\cos^2\theta + \frac{y^2}{b^2} = 1$$
$$y^2 = b^2\sin^2\theta$$
$$y = \pm b\sin\theta$$

Taking the same sign as Q: $y = b\sin\theta$

Therefore: $P = (a\cos\theta, b\sin\theta)$

---

## 📝 Verification

For $P = (a\cos\theta, b\sin\theta)$:

$$\frac{x^2}{a^2} + \frac{y^2}{b^2} = \frac{a^2\cos^2\theta}{a^2} + \frac{b^2\sin^2\theta}{b^2} = \cos^2\theta + \sin^2\theta = 1$$ ✓

---

## ⚡ Key Points at Special Angles

| Eccentric Angle θ | Point on Ellipse |
|-------------------|------------------|
| $0$ | $(a, 0)$ - right vertex |
| $\frac{\pi}{2}$ | $(0, b)$ - top co-vertex |
| $\pi$ | $(-a, 0)$ - left vertex |
| $\frac{3\pi}{2}$ | $(0, -b)$ - bottom co-vertex |

```
              θ = π/2
               (0, b)
                 ●
              ╱     ╲
            ╱         ╲
    θ = π ●─────●─────● θ = 0
   (-a,0)      O      (a,0)
            ╲         ╱
              ╲     ╱
                 ●
              (0,-b)
             θ = 3π/2
```

---

## 📐 Relationship with Coordinates

Given a point $(x_1, y_1)$ on ellipse, find eccentric angle:

$$\cos\theta = \frac{x_1}{a}, \quad \sin\theta = \frac{y_1}{b}$$

$$\theta = \arctan\left(\frac{ay_1}{bx_1}\right)$$

---

## ✏️ Worked Examples

### Example 1: Parametric Point

**Problem**: Find the parametric coordinates of points on $\frac{x^2}{25} + \frac{y^2}{9} = 1$.

**Solution**:

$a = 5$, $b = 3$

Any point: $(5\cos\theta, 3\sin\theta)$

At $\theta = \frac{\pi}{6}$: $\left(5 \cdot \frac{\sqrt{3}}{2}, 3 \cdot \frac{1}{2}\right) = \left(\frac{5\sqrt{3}}{2}, \frac{3}{2}\right)$

**Answer**: General point: $(5\cos\theta, 3\sin\theta)$

---

### Example 2: Finding Eccentric Angle

**Problem**: Find the eccentric angle of point $(2, \frac{3\sqrt{3}}{2})$ on $\frac{x^2}{16} + \frac{y^2}{9} = 1$.

**Solution**:

$a = 4$, $b = 3$

$\cos\theta = \frac{x}{a} = \frac{2}{4} = \frac{1}{2}$

$\sin\theta = \frac{y}{b} = \frac{3\sqrt{3}/2}{3} = \frac{\sqrt{3}}{2}$

$\cos\theta = \frac{1}{2}$ and $\sin\theta = \frac{\sqrt{3}}{2}$

**Answer**: $\theta = \frac{\pi}{3}$

---

### Example 3: End Points of Latus Rectum

**Problem**: Find the eccentric angles at the ends of latus rectum for $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$.

**Solution**:

Ends of latus rectum: $\left(ae, \pm\frac{b^2}{a}\right)$

For the point $\left(ae, \frac{b^2}{a}\right)$:

$\cos\theta = \frac{ae}{a} = e$

$\sin\theta = \frac{b^2/a}{b} = \frac{b}{a}$

**Answer**: $\theta = \arccos(e) = \arctan\left(\frac{b}{ae}\right)$

For four endpoints: $\theta = \pm\arccos(e)$ and $\theta = \pi \pm \arccos(e)$

---

### Example 4: Distance Between Two Points

**Problem**: Find distance between points at $\theta_1 = \frac{\pi}{4}$ and $\theta_2 = \frac{3\pi}{4}$ on $\frac{x^2}{16} + \frac{y^2}{4} = 1$.

**Solution**:

$a = 4$, $b = 2$

$P_1 = \left(4\cos\frac{\pi}{4}, 2\sin\frac{\pi}{4}\right) = (2\sqrt{2}, \sqrt{2})$

$P_2 = \left(4\cos\frac{3\pi}{4}, 2\sin\frac{3\pi}{4}\right) = (-2\sqrt{2}, \sqrt{2})$

Distance = $\sqrt{(2\sqrt{2} - (-2\sqrt{2}))^2 + (\sqrt{2} - \sqrt{2})^2}$
$= \sqrt{(4\sqrt{2})^2} = 4\sqrt{2}$

**Answer**: $4\sqrt{2}$

---

### Example 5: Chord Joining Two Points

**Problem**: Find the equation of chord joining points with eccentric angles $\alpha$ and $\beta$ on $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$.

**Solution**:

Points: $P_1 = (a\cos\alpha, b\sin\alpha)$ and $P_2 = (a\cos\beta, b\sin\beta)$

Slope: $m = \frac{b\sin\beta - b\sin\alpha}{a\cos\beta - a\cos\alpha} = \frac{b(\sin\beta - \sin\alpha)}{a(\cos\beta - \cos\alpha)}$

Using sum-to-product formulas:
$\sin\beta - \sin\alpha = 2\cos\frac{\alpha+\beta}{2}\sin\frac{\beta-\alpha}{2}$
$\cos\beta - \cos\alpha = -2\sin\frac{\alpha+\beta}{2}\sin\frac{\beta-\alpha}{2}$

$m = \frac{b \cdot 2\cos\frac{\alpha+\beta}{2}\sin\frac{\beta-\alpha}{2}}{a \cdot (-2\sin\frac{\alpha+\beta}{2}\sin\frac{\beta-\alpha}{2})} = -\frac{b}{a}\cot\frac{\alpha+\beta}{2}$

Using point-slope form:

$$\boxed{\frac{x}{a}\cos\frac{\alpha+\beta}{2} + \frac{y}{b}\sin\frac{\alpha+\beta}{2} = \cos\frac{\alpha-\beta}{2}}$$

---

### Example 6: Focal Chord Property

**Problem**: If the eccentric angles of ends of a focal chord are $\alpha$ and $\beta$, show that $\tan\frac{\alpha}{2}\tan\frac{\beta}{2} = \frac{e-1}{e+1}$ or $\frac{e+1}{e-1}$.

**Solution**:

The focal chord passes through focus $(ae, 0)$.

Using the chord equation:
$\frac{ae}{a}\cos\frac{\alpha+\beta}{2} + 0 = \cos\frac{\alpha-\beta}{2}$

$e\cos\frac{\alpha+\beta}{2} = \cos\frac{\alpha-\beta}{2}$

Using $\cos A = \frac{1-\tan^2(A/2)}{1+\tan^2(A/2)}$:

Let $t_1 = \tan\frac{\alpha}{2}$, $t_2 = \tan\frac{\beta}{2}$

After algebraic manipulation:

$$\tan\frac{\alpha}{2}\tan\frac{\beta}{2} = \frac{e-1}{e+1}$$ (for right focus)

$$\tan\frac{\alpha}{2}\tan\frac{\beta}{2} = \frac{e+1}{e-1}$$ (for left focus)

---

## 📝 Auxiliary Circle Properties

### 1. Corresponding Points

Every point P on ellipse corresponds to point Q on auxiliary circle with same x-coordinate.

### 2. Ratio of y-coordinates

If P$(a\cos\theta, b\sin\theta)$ on ellipse and Q$(a\cos\theta, a\sin\theta)$ on auxiliary circle:

$$\frac{y_P}{y_Q} = \frac{b\sin\theta}{a\sin\theta} = \frac{b}{a} = \sqrt{1-e^2}$$

### 3. Area Relationship

$$\frac{\text{Area of ellipse}}{\text{Area of auxiliary circle}} = \frac{\pi ab}{\pi a^2} = \frac{b}{a}$$

---

## 💡 Tips and Insights

> 💡 **Not the Angle at Origin**: Eccentric angle θ is NOT the angle OP makes with x-axis. It's the angle OQ makes.

> 💡 **Parameter Range**: θ ranges from 0 to 2π, covering the entire ellipse once.

> ⚠️ **Chord Formula**: The beautiful chord formula uses sum and difference of eccentric angles.

> 💡 **Compression**: The ellipse is the auxiliary circle "compressed" by factor $\frac{b}{a}$ in y-direction.

---

## 📋 Summary Table

| Concept | Formula |
|---------|---------|
| Auxiliary circle | $x^2 + y^2 = a^2$ |
| Parametric point | $(a\cos\theta, b\sin\theta)$ |
| Eccentric angle | $\theta = \arctan\left(\frac{ay}{bx}\right)$ |
| Chord equation | $\frac{x}{a}\cos\frac{\alpha+\beta}{2} + \frac{y}{b}\sin\frac{\alpha+\beta}{2} = \cos\frac{\alpha-\beta}{2}$ |
| Focal chord | $\tan\frac{\alpha}{2}\tan\frac{\beta}{2} = \frac{e-1}{e+1}$ |
| y-ratio (ellipse/circle) | $\frac{b}{a}$ |

---

## ❓ Quick Revision Questions

1. **Write the auxiliary circle for $\frac{x^2}{49} + \frac{y^2}{25} = 1$.**

2. **Find the parametric coordinates for $\frac{x^2}{9} + \frac{y^2}{4} = 1$ at $\theta = \frac{\pi}{3}$.**

3. **Find the eccentric angle of $(3, 2)$ on $\frac{x^2}{9} + \frac{y^2}{4} = 1$.**

4. **Find the equation of chord joining $\theta = 0$ and $\theta = \frac{\pi}{2}$ on $\frac{x^2}{16} + \frac{y^2}{9} = 1$.**

5. **If $P(a\cos\theta, b\sin\theta)$ is at eccentric angle $\frac{\pi}{4}$, find the eccentric angle of its reflection in the major axis.**

6. **For ellipse $\frac{x^2}{25} + \frac{y^2}{9} = 1$, find the eccentric angle at vertex.**

<details>
<summary><b>Click to see answers</b></summary>

1. $a^2 = 49$, so $a = 7$
   **$x^2 + y^2 = 49$**

2. $a = 3$, $b = 2$
   Point: $\left(3\cos\frac{\pi}{3}, 2\sin\frac{\pi}{3}\right) = \left(\frac{3}{2}, \sqrt{3}\right)$
   **$\left(\frac{3}{2}, \sqrt{3}\right)$**

3. $\cos\theta = \frac{3}{3} = 1$, $\sin\theta = \frac{2}{2} = 1$
   But $\cos\theta = 1$ means $\theta = 0$, and $\sin 0 = 0 \neq 1$
   Let me verify: $(3, 2)$ on $\frac{9}{9} + \frac{4}{4} = 1 + 1 = 2 \neq 1$
   Point $(3, 2)$ is NOT on this ellipse.
   **Point not on ellipse** (for $(3, 0)$, θ = 0; for $(0, 2)$, θ = π/2)

4. $\alpha = 0$, $\beta = \frac{\pi}{2}$, $a = 4$, $b = 3$
   $\frac{x}{4}\cos\frac{\pi}{4} + \frac{y}{3}\sin\frac{\pi}{4} = \cos\frac{\pi}{4}$
   $\frac{x}{4} \cdot \frac{1}{\sqrt{2}} + \frac{y}{3} \cdot \frac{1}{\sqrt{2}} = \frac{1}{\sqrt{2}}$
   $\frac{x}{4} + \frac{y}{3} = 1$
   **$\frac{x}{4} + \frac{y}{3} = 1$** or **$3x + 4y = 12$**

5. Reflection in major axis (x-axis) changes sign of y-coordinate.
   New point: $(a\cos\theta, -b\sin\theta) = (a\cos(-\theta), b\sin(-\theta))$
   **Eccentric angle: $-\frac{\pi}{4}$ or $\frac{7\pi}{4}$**

6. Vertices: $(\pm 5, 0)$
   At $(5, 0)$: $\cos\theta = 1$, $\sin\theta = 0$, so **θ = 0**
   At $(-5, 0)$: $\cos\theta = -1$, $\sin\theta = 0$, so **θ = π**

</details>

---

## ⏭️ Navigation

| [← Previous: Eccentricity and Elements](02-eccentricity-elements.md) | [Next: Position of Point and Line →](04-position-point-line.md) |
|:--------------------------------------------------------------------|----------------------------------------------------------------:|
