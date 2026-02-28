# Chapter 7.3: Asymptotes of Hyperbola

## 📚 Chapter Overview

**Asymptotes** are unique to hyperbolas among conic sections. They are straight lines that the hyperbola approaches but never touches as it extends to infinity. This chapter explores asymptote equations and the special **rectangular hyperbola**.

---

## 📝 Definition of Asymptote

> **Asymptote**: A straight line that a curve approaches arbitrarily closely but never intersects (at finite points).

```
              y
              |     Asymptote: y = (b/a)x
              |    /
       \      |   /      /
        \     |  /     /
         \    | /    /
          \   |/   /
   _________\_|_/___________ x
            /\|/\
           /  |  \
          /   |   \
         /    |    \
              |      \
              |       Asymptote: y = -(b/a)x
              
   The hyperbola gets closer and closer 
   to the asymptotes but never touches them
```

---

## 📐 Deriving Asymptotes

For hyperbola $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$:

Solving for $y$:
$$y^2 = b^2\left(\frac{x^2}{a^2} - 1\right)$$

$$y = \pm\frac{b}{a}\sqrt{x^2 - a^2} = \pm\frac{bx}{a}\sqrt{1 - \frac{a^2}{x^2}}$$

As $x \to \pm\infty$:
$$\frac{a^2}{x^2} \to 0$$

$$y \to \pm\frac{bx}{a}$$

### Asymptotes of $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$

$$\boxed{y = \pm\frac{b}{a}x}$$

Or equivalently:
$$\boxed{\frac{x^2}{a^2} - \frac{y^2}{b^2} = 0}$$

This factors as:
$$\left(\frac{x}{a} - \frac{y}{b}\right)\left(\frac{x}{a} + \frac{y}{b}\right) = 0$$

---

## ⚡ Asymptote Equations

### Combined Form

For hyperbola $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$:

**Combined equation of asymptotes**:
$$\boxed{\frac{x^2}{a^2} - \frac{y^2}{b^2} = 0}$$

**Individual asymptotes**:
$$\boxed{y = \frac{b}{a}x \quad \text{and} \quad y = -\frac{b}{a}x}$$

Or:
$$\boxed{\frac{x}{a} = \pm\frac{y}{b}}$$

### For Vertical Hyperbola

For $\frac{y^2}{a^2} - \frac{x^2}{b^2} = 1$:

**Asymptotes**: $y = \pm\frac{a}{b}x$

---

## 📝 Properties of Asymptotes

### 1. Pass Through Centre
Both asymptotes pass through the centre of hyperbola (origin for standard form).

### 2. Symmetry
The asymptotes are symmetric about both axes.

### 3. Angle Between Asymptotes

The angle $\theta$ between asymptotes:
$$\tan\left(\frac{\theta}{2}\right) = \frac{b}{a}$$

### 4. Conjugate Hyperbolas Share Asymptotes

The hyperbolas:
- $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$ (Hyperbola)
- $\frac{x^2}{a^2} - \frac{y^2}{b^2} = -1$ (Conjugate)

have the **same asymptotes**: $y = \pm\frac{b}{a}x$

```
              y
              |
         *    |    *    Conjugate hyperbola
          *   |   *
           *  |  *     
    --------*-|-*-------- x
           *  |  *
          *   |   *
       --*----+----*--   Hyperbola
          *   |   *
           *  |  *
              |
   Both share asymptotes y = ±(b/a)x
```

---

## 📝 Rectangular Hyperbola

### Definition

A **rectangular (equilateral) hyperbola** is one where $a = b$.

### Equation

$$\frac{x^2}{a^2} - \frac{y^2}{a^2} = 1$$

$$\boxed{x^2 - y^2 = a^2}$$

### Properties

1. **Asymptotes**: $y = \pm x$ (perpendicular to each other)
2. **Eccentricity**: $e = \sqrt{1 + 1} = \sqrt{2}$
3. **Asymptotes at 45°** to axes

```
              y
              |    y = x
              |   /
       \      |  /      
        \     | /     
         \    |/    
   ________\_-|-_/________ x
            /\|/\
           /  |  \
          /   |   \
         /    |    y = -x
              |
              
   Rectangular hyperbola: x² - y² = a²
   Asymptotes perpendicular (at 90°)
```

---

## 📝 Rectangular Hyperbola: xy = c²

### Standard Form

When rectangular hyperbola has asymptotes as coordinate axes:

$$\boxed{xy = c^2}$$

```
              y
              |
              |  *
              | *
              |*
   ___________*____________ x
             *|
            * |
           *  |
          *   |
              |
              
   Hyperbola xy = c² 
   Asymptotes: x = 0 and y = 0 (the axes)
```

### Properties of xy = c²

| Property | Value |
|----------|-------|
| Asymptotes | x-axis and y-axis |
| Centre | $(0, 0)$ |
| Vertices | $(\pm c, \pm c)$ |
| Eccentricity | $e = \sqrt{2}$ |
| Transverse axis | $y = x$ or $y = -x$ |
| Parametric form | $(ct, \frac{c}{t})$ |

### Relation to Standard Form

$xy = c^2$ is obtained by rotating $x^2 - y^2 = 2c^2$ by 45°.

---

## ✏️ Worked Examples

### Example 1: Find Asymptotes

**Problem**: Find asymptotes of $\frac{x^2}{16} - \frac{y^2}{9} = 1$.

**Solution**:

$a^2 = 16 \Rightarrow a = 4$
$b^2 = 9 \Rightarrow b = 3$

Asymptotes: $y = \pm\frac{b}{a}x = \pm\frac{3}{4}x$

**Answer**: $y = \frac{3}{4}x$ and $y = -\frac{3}{4}x$

Or: $3x - 4y = 0$ and $3x + 4y = 0$

---

### Example 2: Combined Asymptote Equation

**Problem**: Write combined equation of asymptotes for $9x^2 - 16y^2 = 144$.

**Solution**:

Convert to standard form: $\frac{x^2}{16} - \frac{y^2}{9} = 1$

Combined asymptotes: $\frac{x^2}{16} - \frac{y^2}{9} = 0$

Multiply by 144: $9x^2 - 16y^2 = 0$

**Answer**: $9x^2 - 16y^2 = 0$

---

### Example 3: Angle Between Asymptotes

**Problem**: Find angle between asymptotes of $\frac{x^2}{9} - \frac{y^2}{16} = 1$.

**Solution**:

$a = 3$, $b = 4$

Slopes of asymptotes: $m_1 = \frac{4}{3}$, $m_2 = -\frac{4}{3}$

Angle between lines:
$$\tan\theta = \left|\frac{m_1 - m_2}{1 + m_1 m_2}\right| = \left|\frac{\frac{4}{3} - (-\frac{4}{3})}{1 + \frac{4}{3} \cdot (-\frac{4}{3})}\right|$$

$$= \left|\frac{\frac{8}{3}}{1 - \frac{16}{9}}\right| = \left|\frac{\frac{8}{3}}{-\frac{7}{9}}\right| = \frac{8}{3} \times \frac{9}{7} = \frac{24}{7}$$

$\theta = \tan^{-1}\left(\frac{24}{7}\right)$

**Answer**: $\theta = \tan^{-1}\left(\frac{24}{7}\right) \approx 73.7°$

---

### Example 4: Hyperbola from Asymptotes

**Problem**: Find hyperbola with asymptotes $y = \pm 2x$ passing through $(5, 3)$.

**Solution**:

From asymptotes $y = \pm 2x$: $\frac{b}{a} = 2$, so $b = 2a$

Hyperbola: $\frac{x^2}{a^2} - \frac{y^2}{4a^2} = 1$ or $\frac{x^2}{a^2} - \frac{y^2}{4a^2} = -1$

Substitute $(5, 3)$:

Case 1: $\frac{25}{a^2} - \frac{9}{4a^2} = 1$

$\frac{100 - 9}{4a^2} = 1$

$a^2 = \frac{91}{4}$

Case 2: $\frac{25}{a^2} - \frac{9}{4a^2} = -1$

$\frac{91}{4a^2} = -1$ (impossible)

So $a^2 = \frac{91}{4}$, $b^2 = 4a^2 = 91$

**Answer**: $\frac{4x^2}{91} - \frac{y^2}{91} = 1$ or $\frac{x^2}{91/4} - \frac{y^2}{91} = 1$

---

### Example 5: Rectangular Hyperbola

**Problem**: Find eccentricity and asymptotes of $x^2 - y^2 = 25$.

**Solution**:

$a^2 = b^2 = 25 \Rightarrow a = b = 5$

This is a rectangular hyperbola.

$e = \sqrt{1 + \frac{b^2}{a^2}} = \sqrt{1 + 1} = \sqrt{2}$

Asymptotes: $y = \pm\frac{b}{a}x = \pm x$

**Answer**: $e = \sqrt{2}$, Asymptotes: $y = x$ and $y = -x$

---

### Example 6: Point on xy = c²

**Problem**: Find the equation of tangent to $xy = 8$ at $(4, 2)$.

**Solution**:

For $xy = c^2$, tangent at $(x_1, y_1)$:
$$xy_1 + x_1y = 2c^2$$

At $(4, 2)$ with $c^2 = 8$:
$$2x + 4y = 16$$
$$x + 2y = 8$$

**Answer**: $x + 2y = 8$

---

## 📐 Tangent and Normal to xy = c²

### Parametric Point
$$P = \left(ct, \frac{c}{t}\right)$$

### Tangent at Parameter t

$$\boxed{\frac{x}{t} + ty = 2c}$$

### Normal at Parameter t

$$\boxed{xt^3 - yt = c(t^4 - 1)}$$

### Tangent at $(x_1, y_1)$

$$\boxed{xy_1 + x_1y = 2c^2}$$

---

## 📝 Chord of Contact

For $xy = c^2$, chord of contact from $(x_1, y_1)$:

$$\boxed{xy_1 + x_1y = 2c^2}$$

(Same form as tangent equation)

---

## 💡 Tips and Insights

> 💡 **Quick Check**: Asymptotes equation is hyperbola equation with 1 replaced by 0.

> 💡 **Rectangular Hyperbola**: $a = b$ means $e = \sqrt{2}$ and asymptotes are perpendicular.

> ⚠️ **xy = c²**: This is a rotated rectangular hyperbola with axes as asymptotes.

> 💡 **Memory Aid**: The hyperbola and its conjugate share the SAME asymptotes.

---

## 📋 Summary Table

| Property | General Hyperbola | Rectangular Hyperbola |
|----------|-------------------|----------------------|
| Equation | $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$ | $x^2 - y^2 = a^2$ |
| Asymptotes | $y = \pm\frac{b}{a}x$ | $y = \pm x$ |
| Combined | $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 0$ | $x^2 - y^2 = 0$ |
| Angle | $2\tan^{-1}\frac{b}{a}$ | $90°$ |
| Eccentricity | $\sqrt{1 + \frac{b^2}{a^2}}$ | $\sqrt{2}$ |

### xy = c² Properties

| Property | Value |
|----------|-------|
| Parametric | $(ct, c/t)$ |
| Tangent at $(x_1, y_1)$ | $xy_1 + x_1y = 2c^2$ |
| Tangent at $t$ | $x + t^2y = 2ct$ |
| Normal at $t$ | $xt^3 - yt = c(t^4 - 1)$ |

---

## ❓ Quick Revision Questions

1. **Find asymptotes of $\frac{x^2}{25} - \frac{y^2}{16} = 1$.**

2. **What are the asymptotes of the conjugate of $\frac{x^2}{9} - \frac{y^2}{4} = 1$?**

3. **Find eccentricity of rectangular hyperbola $x^2 - y^2 = 16$.**

4. **Write combined equation of asymptotes for $4x^2 - 9y^2 = 36$.**

5. **Find tangent to $xy = 4$ at $t = 2$.**

6. **If asymptotes of a hyperbola are $y = \pm 3x$, what is the relation between a and b?**

<details>
<summary><b>Click to see answers</b></summary>

1. $a = 5$, $b = 4$
   Asymptotes: $y = \pm\frac{4}{5}x$
   Or: **$4x - 5y = 0$ and $4x + 5y = 0$**

2. Conjugate shares same asymptotes!
   From original: $a = 3$, $b = 2$
   **Asymptotes: $y = \pm\frac{2}{3}x$**

3. For rectangular hyperbola (a = b):
   **$e = \sqrt{2}$**

4. Standard form: $\frac{x^2}{9} - \frac{y^2}{4} = 1$
   Combined asymptotes: $\frac{x^2}{9} - \frac{y^2}{4} = 0$
   **$4x^2 - 9y^2 = 0$**

5. Point: $(ct, c/t) = (2 \times 2, 2/2) = (4, 1)$ where $c^2 = 4$, $c = 2$
   Tangent: $\frac{x}{t} + ty = 2c$
   $\frac{x}{2} + 2y = 4$
   **$x + 4y = 8$**

6. From $y = \pm 3x$: $\frac{b}{a} = 3$
   **$b = 3a$**

</details>

---

## ⏭️ Navigation

| [← Previous: Eccentricity and Elements](02-eccentricity-elements.md) | [Unit 7 Home](README.md) | [Next: Parametric Form →](04-parametric-form.md) |
|:-------------------------------------------------------------------:|:------------------------:|------------------------------------------------:|
