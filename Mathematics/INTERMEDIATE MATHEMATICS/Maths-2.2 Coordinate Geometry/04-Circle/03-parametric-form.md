# Chapter 4.3: Parametric Form of Circle

## 📚 Chapter Overview

The **parametric form** represents a circle using a single parameter (usually an angle). This representation is powerful for problems involving motion on circles, trigonometric substitutions, and finding specific points.

---

## ⚡ Parametric Equations

> **Parametric Form**:
>
> For a circle with center $(h, k)$ and radius $r$:
>
> $$\boxed{x = h + r\cos\theta, \quad y = k + r\sin\theta}$$
>
> where $\theta \in [0, 2\pi)$ is the parameter (eccentric angle).

For circle centered at origin with radius $r$:
$$x = r\cos\theta, \quad y = r\sin\theta$$

---

## 📐 Geometric Interpretation

```
        Y
        ↑
        │         P(h+rcosθ, k+rsinθ)
        │        ●
        │       /│
        │      / │
        │     /  │ r·sinθ
        │    /   │
        │   / θ  │
        │  ●─────┼────
        │ C(h,k) r·cosθ
    ────┼────────────────→ X
        │
```

- $\theta$ is measured from the positive x-direction
- As $\theta$ varies from $0$ to $2\pi$, point P traces the entire circle
- $\theta = 0$ gives rightmost point: $(h + r, k)$
- $\theta = \frac{\pi}{2}$ gives topmost point: $(h, k + r)$
- $\theta = \pi$ gives leftmost point: $(h - r, k)$
- $\theta = \frac{3\pi}{2}$ gives bottommost point: $(h, k - r)$

---

## 📝 Special Angles on Circle

For circle $x^2 + y^2 = r^2$ (centered at origin):

| Angle θ | Point (x, y) | Position |
|---------|--------------|----------|
| 0 | $(r, 0)$ | Right |
| $\frac{\pi}{6}$ | $\left(\frac{r\sqrt{3}}{2}, \frac{r}{2}\right)$ | Upper right |
| $\frac{\pi}{4}$ | $\left(\frac{r}{\sqrt{2}}, \frac{r}{\sqrt{2}}\right)$ | Upper right (45°) |
| $\frac{\pi}{2}$ | $(0, r)$ | Top |
| $\pi$ | $(-r, 0)$ | Left |
| $\frac{3\pi}{2}$ | $(0, -r)$ | Bottom |

---

## ✏️ Worked Examples

### Example 1: Finding Parametric Equations

**Problem**: Write the parametric equations for circle $(x - 3)^2 + (y + 2)^2 = 25$.

**Solution**:

Center: $(3, -2)$, Radius: $5$

$$x = 3 + 5\cos\theta$$
$$y = -2 + 5\sin\theta$$

---

### Example 2: Converting to Cartesian Form

**Problem**: Find the Cartesian equation from $x = 4\cos\theta$, $y = 4\sin\theta$.

**Solution**:

From the equations:
$$\cos\theta = \frac{x}{4}, \quad \sin\theta = \frac{y}{4}$$

Using $\cos^2\theta + \sin^2\theta = 1$:
$$\frac{x^2}{16} + \frac{y^2}{16} = 1$$

$$x^2 + y^2 = 16$$

---

### Example 3: Finding a Point on Circle

**Problem**: Find the point on circle $x^2 + y^2 = 25$ at angle $\theta = \frac{\pi}{3}$.

**Solution**:

$$x = 5\cos\frac{\pi}{3} = 5 \times \frac{1}{2} = \frac{5}{2}$$

$$y = 5\sin\frac{\pi}{3} = 5 \times \frac{\sqrt{3}}{2} = \frac{5\sqrt{3}}{2}$$

**Answer**: $\left(\frac{5}{2}, \frac{5\sqrt{3}}{2}\right)$

---

### Example 4: Equation of Chord Joining Two Points

**Problem**: Find the equation of chord joining points $\theta_1$ and $\theta_2$ on circle $x^2 + y^2 = r^2$.

**Solution**:

Points: $P(r\cos\theta_1, r\sin\theta_1)$ and $Q(r\cos\theta_2, r\sin\theta_2)$

**Chord equation**:
$$\boxed{x\cos\left(\frac{\theta_1 + \theta_2}{2}\right) + y\sin\left(\frac{\theta_1 + \theta_2}{2}\right) = r\cos\left(\frac{\theta_1 - \theta_2}{2}\right)}$$

**Derivation**: Using two-point form and trigonometric identities.

---

### Example 5: Tangent at a Point (Parametric Form)

**Problem**: Find the equation of tangent at point $\theta$ on circle $x^2 + y^2 = r^2$.

**Solution**:

Point of tangency: $(r\cos\theta, r\sin\theta)$

The tangent at $(x_1, y_1)$ on $x^2 + y^2 = r^2$ is $xx_1 + yy_1 = r^2$.

Substituting:
$$x(r\cos\theta) + y(r\sin\theta) = r^2$$

$$\boxed{x\cos\theta + y\sin\theta = r}$$

---

### Example 6: Motion on a Circle

**Problem**: A particle moves on circle $x^2 + y^2 = 16$ such that $\theta = 2t$ (t in seconds). Find its position at $t = \frac{\pi}{4}$ seconds.

**Solution**:

At $t = \frac{\pi}{4}$: $\theta = 2 \times \frac{\pi}{4} = \frac{\pi}{2}$

Position:
$$x = 4\cos\frac{\pi}{2} = 0$$
$$y = 4\sin\frac{\pi}{2} = 4$$

**Answer**: $(0, 4)$

---

### Example 7: Chord Subtending Given Angle at Center

**Problem**: Find the length of chord that subtends angle $\frac{\pi}{3}$ at the center of circle with radius 6.

**Solution**:

```
          ●P
         /│\
        / │ \
       /  │  \
      /   │   \
     / θ/2│    \
    ●────┼────●
   C     M     Q
```

If the chord subtends angle $\theta$ at center:

Chord length = $2r\sin\frac{\theta}{2}$

$$= 2 \times 6 \times \sin\frac{\pi}{6} = 12 \times \frac{1}{2} = 6$$

**Answer**: 6 units

---

## 📝 Parametric Form for General Circle

For circle $x^2 + y^2 + 2gx + 2fy + c = 0$:

Center: $(-g, -f)$, Radius: $R = \sqrt{g^2 + f^2 - c}$

**Parametric equations**:
$$x = -g + R\cos\theta$$
$$y = -f + R\sin\theta$$

---

## 📝 Applications of Parametric Form

### 1. Finding Maximum/Minimum Values

If $(x, y) = (r\cos\theta, r\sin\theta)$ lies on $x^2 + y^2 = r^2$, then:

For expression $ax + by$:
$$ax + by = ar\cos\theta + br\sin\theta = r\sqrt{a^2 + b^2}\cos(\theta - \alpha)$$

Maximum = $r\sqrt{a^2 + b^2}$, Minimum = $-r\sqrt{a^2 + b^2}$

### 2. Trigonometric Substitution

For integrals/expressions involving $\sqrt{r^2 - x^2}$, substitute $x = r\sin\theta$.

### 3. Inscribed Polygons

For regular n-gon inscribed in circle $x^2 + y^2 = r^2$:

Vertices: $\left(r\cos\frac{2k\pi}{n}, r\sin\frac{2k\pi}{n}\right)$ for $k = 0, 1, 2, ..., n-1$

---

### Example 8: Maximum Value Problem

**Problem**: Find maximum value of $3x + 4y$ if $(x, y)$ lies on $x^2 + y^2 = 25$.

**Solution**:

Let $x = 5\cos\theta$, $y = 5\sin\theta$

$$3x + 4y = 15\cos\theta + 20\sin\theta$$

Maximum = $\sqrt{15^2 + 20^2} = \sqrt{225 + 400} = \sqrt{625} = 25$

**Answer**: Maximum = 25 (at $\theta = \tan^{-1}\frac{4}{3}$)

---

## 📝 Eccentric Angle Concept

The parameter $\theta$ is called the **eccentric angle** (though this term is more commonly used for ellipses).

```
    ┌──────────────────────────────────────────────────────┐
    │                                                      │
    │   Point P on circle: (r·cosθ, r·sinθ)                │
    │                                                      │
    │   θ = Angle made by OP with positive x-axis          │
    │       (O being the center)                           │
    │                                                      │
    │   NOT the same as the angle in polar coordinates     │
    │   unless the center is at origin                     │
    │                                                      │
    └──────────────────────────────────────────────────────┘
```

---

## 💡 Tips and Insights

> 💡 **Simplification**: Parametric form often simplifies problems involving specific points on circles.

> 💡 **Periodic Nature**: Since $\cos$ and $\sin$ are periodic with period $2\pi$, adding $2n\pi$ gives the same point.

> ⚠️ **Direction**: Counter-clockwise motion increases $\theta$; clockwise motion decreases $\theta$.

> 💡 **Chord Formula**: $x\cos\frac{\theta_1+\theta_2}{2} + y\sin\frac{\theta_1+\theta_2}{2} = r\cos\frac{\theta_1-\theta_2}{2}$

---

## 🌍 Real-World Applications

1. **Circular Motion**: Position of objects moving in circles
2. **Animation**: Smooth circular paths in graphics
3. **Engineering**: Cam and gear profiles
4. **Physics**: Wave motion, oscillations
5. **Robotics**: Circular arm movements
6. **Navigation**: Satellite positions

---

## 📋 Summary Table

| Circle | Parametric Form |
|--------|-----------------|
| $x^2 + y^2 = r^2$ | $x = r\cos\theta$, $y = r\sin\theta$ |
| $(x-h)^2 + (y-k)^2 = r^2$ | $x = h + r\cos\theta$, $y = k + r\sin\theta$ |
| Tangent at θ | $x\cos\theta + y\sin\theta = r$ |
| Chord joining $\theta_1$, $\theta_2$ | $x\cos\frac{\theta_1+\theta_2}{2} + y\sin\frac{\theta_1+\theta_2}{2} = r\cos\frac{\theta_1-\theta_2}{2}$ |
| Chord length for angle θ | $2r\sin\frac{\theta}{2}$ |

---

## ❓ Quick Revision Questions

1. **Write parametric equations for $(x + 2)^2 + (y - 3)^2 = 16$.**

2. **Find the Cartesian equation from $x = 3 + 2\cos\theta$, $y = 1 + 2\sin\theta$.**

3. **Find point on $x^2 + y^2 = 36$ at $\theta = \frac{2\pi}{3}$.**

4. **Find tangent at $\theta = \frac{\pi}{4}$ on circle $x^2 + y^2 = 8$.**

5. **Find maximum value of $x + y$ where $(x, y)$ lies on $x^2 + y^2 = 2$.**

6. **Find the length of chord on $x^2 + y^2 = 100$ subtending angle $\frac{\pi}{2}$ at center.**

<details>
<summary><b>Click to see answers</b></summary>

1. $x = -2 + 4\cos\theta$, $y = 3 + 4\sin\theta$

2. $\cos\theta = \frac{x-3}{2}$, $\sin\theta = \frac{y-1}{2}$
   $(x-3)^2 + (y-1)^2 = 4$

3. $x = 6\cos\frac{2\pi}{3} = 6 \times (-\frac{1}{2}) = -3$
   $y = 6\sin\frac{2\pi}{3} = 6 \times \frac{\sqrt{3}}{2} = 3\sqrt{3}$
   **$(-3, 3\sqrt{3})$**

4. $x\cos\frac{\pi}{4} + y\sin\frac{\pi}{4} = \sqrt{8}$
   $\frac{x}{\sqrt{2}} + \frac{y}{\sqrt{2}} = 2\sqrt{2}$
   **$x + y = 4$**

5. Let $x = \sqrt{2}\cos\theta$, $y = \sqrt{2}\sin\theta$
   $x + y = \sqrt{2}(\cos\theta + \sin\theta) = 2\sin(\theta + \frac{\pi}{4})$
   Maximum = **2**

6. $r = 10$, $\theta = \frac{\pi}{2}$
   Chord = $2 \times 10 \times \sin\frac{\pi}{4} = 20 \times \frac{1}{\sqrt{2}} = 10\sqrt{2}$

</details>

---

## ⏭️ Navigation

| [← Previous: General Equation](02-general-equation.md) | [Next: Position of Point and Line →](04-point-line-position.md) |
|:-------------------------------------------------------|----------------------------------------------------------------:|
