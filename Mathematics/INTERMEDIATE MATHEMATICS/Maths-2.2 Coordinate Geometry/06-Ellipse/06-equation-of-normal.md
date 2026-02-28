# Chapter 6.6: Equation of Normal to Ellipse

## 📚 Chapter Overview

The **normal** to an ellipse at any point is the line perpendicular to the tangent at that point. This chapter derives normal equations in various forms and explores their properties.

---

## 📝 Normal at a Point

### For Ellipse $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$

At point $(x_1, y_1)$:
- Slope of tangent = $-\frac{b^2x_1}{a^2y_1}$
- Slope of normal = $\frac{a^2y_1}{b^2x_1}$

> **Normal at $(x_1, y_1)$**:
>
> $$\boxed{\frac{a^2x}{x_1} - \frac{b^2y}{y_1} = a^2 - b^2}$$

Or equivalently:
$$\frac{x - x_1}{x_1/a^2} = \frac{y - y_1}{y_1/b^2}$$

---

## 📐 Derivation

Normal at $(x_1, y_1)$ with slope $\frac{a^2y_1}{b^2x_1}$:

$$y - y_1 = \frac{a^2y_1}{b^2x_1}(x - x_1)$$

$$b^2x_1(y - y_1) = a^2y_1(x - x_1)$$

$$b^2x_1y - b^2x_1y_1 = a^2y_1x - a^2x_1y_1$$

$$a^2y_1x - b^2x_1y = a^2x_1y_1 - b^2x_1y_1$$

$$a^2y_1x - b^2x_1y = x_1y_1(a^2 - b^2)$$

Dividing by $x_1y_1$:

$$\frac{a^2x}{x_1} - \frac{b^2y}{y_1} = a^2 - b^2$$

---

## ⚡ Parametric Form of Normal

For point at eccentric angle θ: $(a\cos\theta, b\sin\theta)$

### Normal at Eccentric Angle θ

$$\boxed{ax\sec\theta - by\csc\theta = a^2 - b^2}$$

Or equivalently:
$$\frac{ax}{\cos\theta} - \frac{by}{\sin\theta} = a^2 - b^2$$

### Derivation

Substitute $x_1 = a\cos\theta$, $y_1 = b\sin\theta$:

$$\frac{a^2x}{a\cos\theta} - \frac{b^2y}{b\sin\theta} = a^2 - b^2$$

$$\frac{ax}{\cos\theta} - \frac{by}{\sin\theta} = a^2 - b^2$$

---

## 📝 Slope Form of Normal

For normal with slope $m$:

From $\frac{a^2y_1}{b^2x_1} = m$, we get $y_1 = \frac{mb^2x_1}{a^2}$

Using $\frac{x_1^2}{a^2} + \frac{y_1^2}{b^2} = 1$:

$$\frac{x_1^2}{a^2} + \frac{m^2b^4x_1^2}{a^4b^2} = 1$$

$$x_1^2\left(\frac{1}{a^2} + \frac{m^2b^2}{a^4}\right) = 1$$

$$x_1^2 = \frac{a^4}{a^2 + m^2b^2}$$

### Normal with Slope m

$$\boxed{y = mx \mp \frac{m(a^2 - b^2)}{\sqrt{a^2 + m^2b^2}}}$$

The foot of normal (point on ellipse): 

$$\left(\pm\frac{a^2}{\sqrt{a^2 + m^2b^2}}, \pm\frac{mb^2}{\sqrt{a^2 + m^2b^2}}\right)$$

---

## ✏️ Worked Examples

### Example 1: Normal at a Point

**Problem**: Find normal to $\frac{x^2}{25} + \frac{y^2}{16} = 1$ at $(3, \frac{16}{5})$.

**Solution**:

$a^2 = 25$, $b^2 = 16$

Normal: $\frac{a^2x}{x_1} - \frac{b^2y}{y_1} = a^2 - b^2$

$$\frac{25x}{3} - \frac{16y}{16/5} = 25 - 16$$

$$\frac{25x}{3} - 5y = 9$$

$$25x - 15y = 27$$

**Answer**: $25x - 15y = 27$

---

### Example 2: Normal at Eccentric Angle

**Problem**: Find normal to $\frac{x^2}{9} + \frac{y^2}{4} = 1$ at $\theta = \frac{\pi}{6}$.

**Solution**:

$a = 3$, $b = 2$

Normal: $\frac{ax}{\cos\theta} - \frac{by}{\sin\theta} = a^2 - b^2$

$$\frac{3x}{\cos\frac{\pi}{6}} - \frac{2y}{\sin\frac{\pi}{6}} = 9 - 4$$

$$\frac{3x}{\sqrt{3}/2} - \frac{2y}{1/2} = 5$$

$$\frac{6x}{\sqrt{3}} - 4y = 5$$

$$2\sqrt{3}x - 4y = 5$$

**Answer**: $2\sqrt{3}x - 4y = 5$

---

### Example 3: Normal with Given Slope

**Problem**: Find normal to $\frac{x^2}{16} + \frac{y^2}{9} = 1$ with slope 2.

**Solution**:

$a^2 = 16$, $b^2 = 9$, $m = 2$

$c = \mp\frac{m(a^2 - b^2)}{\sqrt{a^2 + m^2b^2}} = \mp\frac{2 \times 7}{\sqrt{16 + 36}} = \mp\frac{14}{\sqrt{52}} = \mp\frac{14}{2\sqrt{13}} = \mp\frac{7}{\sqrt{13}}$

Normals: $y = 2x \mp \frac{7}{\sqrt{13}}$

**Answer**: $y = 2x - \frac{7\sqrt{13}}{13}$ and $y = 2x + \frac{7\sqrt{13}}{13}$

---

### Example 4: Tangent and Normal at Same Point

**Problem**: Find tangent and normal to $\frac{x^2}{25} + \frac{y^2}{9} = 1$ at $\theta = \frac{\pi}{4}$.

**Solution**:

$a = 5$, $b = 3$

Point: $\left(\frac{5}{\sqrt{2}}, \frac{3}{\sqrt{2}}\right)$

**Tangent**: $\frac{x\cos\frac{\pi}{4}}{5} + \frac{y\sin\frac{\pi}{4}}{3} = 1$

$$\frac{x}{5\sqrt{2}} + \frac{y}{3\sqrt{2}} = 1$$

$$\frac{3x + 5y}{15\sqrt{2}} = 1$$

$$3x + 5y = 15\sqrt{2}$$

**Normal**: $\frac{5x}{\cos\frac{\pi}{4}} - \frac{3y}{\sin\frac{\pi}{4}} = 25 - 9$

$$5\sqrt{2}x - 3\sqrt{2}y = 16$$

$$5x - 3y = \frac{16}{\sqrt{2}} = 8\sqrt{2}$$

**Answer**: Tangent: $3x + 5y = 15\sqrt{2}$, Normal: $5x - 3y = 8\sqrt{2}$

---

### Example 5: Normal Passing Through a Point

**Problem**: How many normals can be drawn from $(0, 6)$ to $\frac{x^2}{25} + \frac{y^2}{9} = 1$?

**Solution**:

Normal at $(a\cos\theta, b\sin\theta)$ passes through $(0, 6)$:

$$\frac{5 \times 0}{\cos\theta} - \frac{3 \times 6}{\sin\theta} = 25 - 9$$

$$-\frac{18}{\sin\theta} = 16$$

$$\sin\theta = -\frac{18}{16} = -\frac{9}{8}$$

Since $|\sin\theta| \leq 1$, no solution exists.

**Answer**: **0 normals** can be drawn from $(0, 6)$.

---

### Example 6: Number of Normals from Point on Axis

**Problem**: How many normals from $(4, 0)$ to $\frac{x^2}{16} + \frac{y^2}{9} = 1$?

**Solution**:

Normal equation: $\frac{ax}{\cos\theta} - \frac{by}{\sin\theta} = a^2 - b^2$

$$\frac{4 \times 4}{\cos\theta} - 0 = 16 - 9 = 7$$

$$\frac{16}{\cos\theta} = 7$$

$$\cos\theta = \frac{16}{7} > 1$$

This has no solution for θ.

But wait, the normal from a point on the major axis can also be the major axis itself!

At vertices $(\pm 4, 0)$, the normal IS the x-axis (major axis), which passes through $(4, 0)$.

Actually, $(4, 0)$ is ON the ellipse (vertex), so only 1 normal (the major axis).

**Answer**: **1 normal** (the major axis itself)

---

## 📝 Co-Normal Points

### Definition

If normals at three points on an ellipse pass through a common point, those points are called **co-normal points**.

### Property

For co-normal points at eccentric angles $\alpha$, $\beta$, $\gamma$:

$$\sin(\alpha + \beta) + \sin(\beta + \gamma) + \sin(\gamma + \alpha) = 0$$

---

## 📐 Relation Between Tangent and Normal

| Property | Tangent | Normal |
|----------|---------|--------|
| At $(x_1, y_1)$ | $\frac{xx_1}{a^2} + \frac{yy_1}{b^2} = 1$ | $\frac{a^2x}{x_1} - \frac{b^2y}{y_1} = a^2 - b^2$ |
| At θ | $\frac{x\cos\theta}{a} + \frac{y\sin\theta}{b} = 1$ | $ax\sec\theta - by\csc\theta = a^2 - b^2$ |
| Product of slopes | $-\frac{b^2x_1}{a^2y_1} \times \frac{a^2y_1}{b^2x_1} = -1$ | (Perpendicular) |

---

## 📝 Where Normal Meets Axes

At point $(a\cos\theta, b\sin\theta)$:

**Normal meets x-axis at**:
$$\left(\frac{(a^2-b^2)\cos\theta}{a}, 0\right) = (e^2 a\cos\theta, 0)$$

**Normal meets y-axis at**:
$$\left(0, -\frac{(a^2-b^2)\sin\theta}{b}\right)$$

---

## 💡 Tips and Insights

> 💡 **Quick Formula**: Normal at $(x_1, y_1)$ has form $\frac{a^2x}{x_1} - \frac{b^2y}{y_1} = a^2 - b^2$.

> 💡 **At Vertices**: Normal at vertices $(\pm a, 0)$ is the major axis (x-axis for horizontal ellipse).

> ⚠️ **At Co-vertices**: Normal at co-vertices $(0, \pm b)$ is the minor axis.

> 💡 **Maximum Normals**: At most 4 normals can be drawn from any point to an ellipse.

---

## 📋 Summary Table

| Form | Normal Equation |
|------|-----------------|
| At $(x_1, y_1)$ | $\frac{a^2x}{x_1} - \frac{b^2y}{y_1} = a^2 - b^2$ |
| At eccentric angle θ | $ax\sec\theta - by\csc\theta = a^2 - b^2$ |
| With slope $m$ | $y = mx \mp \frac{m(a^2-b^2)}{\sqrt{a^2+m^2b^2}}$ |
| Meets x-axis | $\left(\frac{(a^2-b^2)\cos\theta}{a}, 0\right)$ |
| Meets y-axis | $\left(0, -\frac{(a^2-b^2)\sin\theta}{b}\right)$ |

---

## ❓ Quick Revision Questions

1. **Find normal to $\frac{x^2}{16} + \frac{y^2}{9} = 1$ at $(2, \frac{3\sqrt{3}}{2})$.**

2. **Find normal at $\theta = \frac{\pi}{2}$ for $\frac{x^2}{25} + \frac{y^2}{16} = 1$.**

3. **Where does the normal at $\theta = \frac{\pi}{4}$ on $\frac{x^2}{9} + \frac{y^2}{4} = 1$ meet the x-axis?**

4. **Find normal to $\frac{x^2}{9} + \frac{y^2}{5} = 1$ with slope 1.**

5. **Find the feet of normals from origin to $\frac{x^2}{4} + \frac{y^2}{3} = 1$.**

6. **Show that the normal at an end of latus rectum passes through an end of the minor axis for special ellipses.**

<details>
<summary><b>Click to see answers</b></summary>

1. $a^2 = 16$, $b^2 = 9$
   $\frac{16x}{2} - \frac{9y}{3\sqrt{3}/2} = 16 - 9$
   $8x - \frac{18y}{3\sqrt{3}} = 7$
   $8x - \frac{6y}{\sqrt{3}} = 7$
   $8x - 2\sqrt{3}y = 7$
   **$8x - 2\sqrt{3}y = 7$**

2. At $\theta = \frac{\pi}{2}$: Point $(0, 4)$
   $\cos\frac{\pi}{2} = 0$, so $\sec\theta$ is undefined.
   This means the normal is vertical: **$x = 0$** (the y-axis, which is the minor axis)

3. $a = 3$, $b = 2$
   X-intercept: $\left(\frac{(9-4)\cos\frac{\pi}{4}}{3}, 0\right) = \left(\frac{5 \cdot \frac{1}{\sqrt{2}}}{3}, 0\right) = \left(\frac{5}{3\sqrt{2}}, 0\right)$
   **$\left(\frac{5\sqrt{2}}{6}, 0\right)$**

4. $a^2 = 9$, $b^2 = 5$, $m = 1$
   $c = \mp\frac{1 \times 4}{\sqrt{9 + 5}} = \mp\frac{4}{\sqrt{14}}$
   **$y = x \mp \frac{4}{\sqrt{14}}$** or **$y = x \mp \frac{2\sqrt{14}}{7}$**

5. Normal at $(2\cos\theta, \sqrt{3}\sin\theta)$ passes through origin:
   $\frac{2 \times 0}{\cos\theta} - \frac{\sqrt{3} \times 0}{\sin\theta} = 4 - 3 = 1$
   $0 = 1$ (contradiction)
   Only exception: at vertices where $\theta = 0, \pi$ (normal is x-axis through origin)
   and at co-vertices where $\theta = \frac{\pi}{2}, \frac{3\pi}{2}$ (normal is y-axis through origin)
   **Feet: $(\pm 2, 0)$ and $(0, \pm\sqrt{3})$** (vertices and co-vertices)

6. End of LR: $(ae, \frac{b^2}{a})$
   Normal at this point should pass through $(0, \pm b)$.
   This is true when $\frac{a^2 \times 0}{ae} - \frac{b^2 \times (\pm b)}{b^2/a} = a^2 - b^2$
   $\mp ab = a^2 - b^2$
   This requires $a^2 - b^2 = ab$, i.e., $a^2 - ab - b^2 = 0$
   $a = \frac{b(1 + \sqrt{5})}{2} = \phi b$ where $\phi$ is golden ratio!
   **True only when $\frac{a}{b} = \phi$ (golden ratio ellipse)**

</details>

---

## 🎉 Unit 6 Complete!

Congratulations! You've completed the study of **Ellipse**. Key concepts covered:

✅ Standard equations and two orientations  
✅ Eccentricity, foci, directrices, latus rectum  
✅ Parametric form and auxiliary circle  
✅ Position of points and lines  
✅ Tangent equations (point, parametric, slope forms)  
✅ Normal equations and properties  

---

## ⏭️ Navigation

| [← Previous: Equation of Tangent](05-equation-of-tangent.md) | [Unit 6 Home](README.md) | [Next Unit: Hyperbola →](../07-Hyperbola/README.md) |
|:-------------------------------------------------------------|:------------------------:|----------------------------------------------------:|
