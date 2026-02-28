# Chapter 6.5: Equation of Tangent to Ellipse

## 📚 Chapter Overview

A **tangent** to an ellipse is a line that touches the curve at exactly one point. This chapter derives tangent equations in multiple forms—point form, parametric form, and slope form—essential for solving ellipse problems.

---

## 📝 Tangent at a Point (T = 0)

### For Ellipse $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$

> **Tangent at $(x_1, y_1)$**:
>
> $$\boxed{\frac{xx_1}{a^2} + \frac{yy_1}{b^2} = 1}$$

This is the **T = 0** form, obtained by replacing:
- $x^2 → xx_1$
- $y^2 → yy_1$

```
                    ╭─────────────╮
                  ╱                 ╲
                ╱                     ╲
               ●P(x₁,y₁)───────────────  Tangent
              ╱                       ╲
    ─────────●───────────●─────────────●─────────
              ╲                       ╱
                ╲                   ╱
                  ╲               ╱
                    ╰───────────╯
```

---

## 📐 Derivation

For ellipse $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$, differentiate implicitly:

$$\frac{2x}{a^2} + \frac{2y}{b^2}\frac{dy}{dx} = 0$$

$$\frac{dy}{dx} = -\frac{b^2x}{a^2y}$$

At $(x_1, y_1)$: slope = $-\frac{b^2x_1}{a^2y_1}$

Tangent equation:
$$y - y_1 = -\frac{b^2x_1}{a^2y_1}(x - x_1)$$

$$a^2y_1(y - y_1) = -b^2x_1(x - x_1)$$

$$a^2y_1y - a^2y_1^2 = -b^2x_1x + b^2x_1^2$$

$$b^2x_1x + a^2y_1y = b^2x_1^2 + a^2y_1^2$$

Dividing by $a^2b^2$:
$$\frac{x_1x}{a^2} + \frac{y_1y}{b^2} = \frac{x_1^2}{a^2} + \frac{y_1^2}{b^2} = 1$$

---

## ⚡ Parametric Form of Tangent

For ellipse with parametric point $(a\cos\theta, b\sin\theta)$:

### Tangent at Eccentric Angle θ

$$\boxed{\frac{x\cos\theta}{a} + \frac{y\sin\theta}{b} = 1}$$

### Derivation

Substitute $(x_1, y_1) = (a\cos\theta, b\sin\theta)$:

$$\frac{x \cdot a\cos\theta}{a^2} + \frac{y \cdot b\sin\theta}{b^2} = 1$$

$$\frac{x\cos\theta}{a} + \frac{y\sin\theta}{b} = 1$$

---

## 📝 Slope Form of Tangent

A line with slope $m$: $y = mx + c$

For tangency: $c^2 = a^2m^2 + b^2$

### Tangent with Slope m

$$\boxed{y = mx \pm \sqrt{a^2m^2 + b^2}}$$

The two tangents (with + and -) touch at different points on the ellipse.

### Point of Contact

For tangent $y = mx + \sqrt{a^2m^2 + b^2}$:

Point of contact: $\left(\frac{-a^2m}{\sqrt{a^2m^2 + b^2}}, \frac{b^2}{\sqrt{a^2m^2 + b^2}}\right)$

For tangent $y = mx - \sqrt{a^2m^2 + b^2}$:

Point of contact: $\left(\frac{a^2m}{\sqrt{a^2m^2 + b^2}}, \frac{-b^2}{\sqrt{a^2m^2 + b^2}}\right)$

---

## ✏️ Worked Examples

### Example 1: Tangent at a Point

**Problem**: Find tangent to $\frac{x^2}{16} + \frac{y^2}{9} = 1$ at $(2, \frac{3\sqrt{3}}{2})$.

**Solution**:

$a^2 = 16$, $b^2 = 9$

Tangent: $\frac{xx_1}{a^2} + \frac{yy_1}{b^2} = 1$

$$\frac{2x}{16} + \frac{\frac{3\sqrt{3}}{2} \cdot y}{9} = 1$$

$$\frac{x}{8} + \frac{\sqrt{3}y}{6} = 1$$

$$\frac{3x + 4\sqrt{3}y}{24} = 1$$

$$3x + 4\sqrt{3}y = 24$$

**Answer**: $3x + 4\sqrt{3}y = 24$

---

### Example 2: Tangent at Eccentric Angle

**Problem**: Find tangent to $\frac{x^2}{25} + \frac{y^2}{16} = 1$ at $\theta = \frac{\pi}{3}$.

**Solution**:

$a = 5$, $b = 4$

Tangent: $\frac{x\cos\frac{\pi}{3}}{5} + \frac{y\sin\frac{\pi}{3}}{4} = 1$

$$\frac{x \cdot \frac{1}{2}}{5} + \frac{y \cdot \frac{\sqrt{3}}{2}}{4} = 1$$

$$\frac{x}{10} + \frac{\sqrt{3}y}{8} = 1$$

$$\frac{4x + 5\sqrt{3}y}{40} = 1$$

$$4x + 5\sqrt{3}y = 40$$

**Answer**: $4x + 5\sqrt{3}y = 40$

---

### Example 3: Tangent with Given Slope

**Problem**: Find tangents to $\frac{x^2}{9} + \frac{y^2}{4} = 1$ with slope 2.

**Solution**:

$a^2 = 9$, $b^2 = 4$, $m = 2$

$c = \pm\sqrt{a^2m^2 + b^2} = \pm\sqrt{36 + 4} = \pm\sqrt{40} = \pm 2\sqrt{10}$

Tangents: $y = 2x \pm 2\sqrt{10}$

**Answer**: $y = 2x + 2\sqrt{10}$ and $y = 2x - 2\sqrt{10}$

---

### Example 4: Tangent Parallel to Line

**Problem**: Find tangent to $\frac{x^2}{25} + \frac{y^2}{9} = 1$ parallel to $x + y = 7$.

**Solution**:

Slope of $x + y = 7$ is $m = -1$

$c = \pm\sqrt{25(1) + 9} = \pm\sqrt{34}$

Tangent: $y = -x \pm \sqrt{34}$

Or: $x + y = \pm\sqrt{34}$

**Answer**: $x + y = \sqrt{34}$ and $x + y = -\sqrt{34}$

---

### Example 5: Tangent Perpendicular to Line

**Problem**: Find tangent to $\frac{x^2}{16} + \frac{y^2}{9} = 1$ perpendicular to $2x - 3y = 5$.

**Solution**:

Slope of given line = $\frac{2}{3}$

Perpendicular slope: $m = -\frac{3}{2}$

$c = \pm\sqrt{16 \times \frac{9}{4} + 9} = \pm\sqrt{36 + 9} = \pm\sqrt{45} = \pm 3\sqrt{5}$

Tangent: $y = -\frac{3}{2}x \pm 3\sqrt{5}$

Or: $3x + 2y = \pm 6\sqrt{5}$

**Answer**: $3x + 2y = \pm 6\sqrt{5}$

---

### Example 6: Tangents from External Point

**Problem**: Find tangents from $(4, 3)$ to $\frac{x^2}{9} + \frac{y^2}{4} = 1$.

**Solution**:

Let tangent be $y = mx + \sqrt{9m^2 + 4}$

Passes through $(4, 3)$:
$$3 = 4m + \sqrt{9m^2 + 4}$$
$$3 - 4m = \sqrt{9m^2 + 4}$$

Squaring (valid when $3 - 4m \geq 0$, i.e., $m \leq \frac{3}{4}$):
$$9 - 24m + 16m^2 = 9m^2 + 4$$
$$7m^2 - 24m + 5 = 0$$
$$(7m - 1)(m - 5) = 0$$

$m = \frac{1}{7}$ or $m = 5$

Check $m \leq \frac{3}{4}$: Only $m = \frac{1}{7}$ is valid.

For $m = 5$, try $y = mx - \sqrt{9m^2 + 4}$:
$3 = 20 - \sqrt{229}$... doesn't work.

Actually, let me use both signs. For $m = 5$:
$3 = 4(5) + c$ gives $c = -17$
Check: $17^2 = 289$ vs $9(25) + 4 = 229$. Not equal, so $m = 5$ doesn't give tangent.

For $m = \frac{1}{7}$:
$c = 3 - \frac{4}{7} = \frac{17}{7}$
Check: $\left(\frac{17}{7}\right)^2 = \frac{289}{49}$ vs $\frac{9}{49} + 4 = \frac{205}{49}$. Not equal!

Let me recalculate. The tangent from external point requires:

$y - 3 = m(x - 4)$, i.e., $y = mx - 4m + 3$

For this to be tangent: $(3 - 4m)^2 = 9m^2 + 4$

$9 - 24m + 16m^2 = 9m^2 + 4$
$7m^2 - 24m + 5 = 0$
$m = \frac{24 \pm \sqrt{576 - 140}}{14} = \frac{24 \pm \sqrt{436}}{14} = \frac{24 \pm 2\sqrt{109}}{14} = \frac{12 \pm \sqrt{109}}{7}$

**Answer**: Tangents with slopes $m = \frac{12 + \sqrt{109}}{7}$ and $m = \frac{12 - \sqrt{109}}{7}$

---

## 📝 Pair of Tangents from External Point

From external point $(x_1, y_1)$, the combined equation of pair of tangents is:

$$\boxed{SS_1 = T^2}$$

where:
- $S = \frac{x^2}{a^2} + \frac{y^2}{b^2} - 1$
- $S_1 = \frac{x_1^2}{a^2} + \frac{y_1^2}{b^2} - 1$
- $T = \frac{xx_1}{a^2} + \frac{yy_1}{b^2} - 1$

---

## 📝 Chord of Contact

If tangents from $(x_1, y_1)$ touch ellipse at P and Q, then PQ (chord of contact) has equation:

$$\boxed{T = 0}$$

i.e., $\frac{xx_1}{a^2} + \frac{yy_1}{b^2} = 1$

---

## 📐 Director Circle

The **director circle** is the locus of point from which perpendicular tangents can be drawn to the ellipse.

For ellipse $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$:

$$\boxed{\text{Director Circle: } x^2 + y^2 = a^2 + b^2}$$

### Derivation

Tangents with slopes $m_1$ and $m_2$ from $(h, k)$:
$k = m \cdot h \pm \sqrt{a^2m^2 + b^2}$

$(k - mh)^2 = a^2m^2 + b^2$

$m^2(h^2 - a^2) - 2khm + (k^2 - b^2) = 0$

$m_1 m_2 = \frac{k^2 - b^2}{h^2 - a^2}$

For perpendicular tangents: $m_1 m_2 = -1$

$\frac{k^2 - b^2}{h^2 - a^2} = -1$

$k^2 - b^2 = -h^2 + a^2$

$h^2 + k^2 = a^2 + b^2$

---

## 💡 Tips and Insights

> 💡 **T = 0 Rule**: For tangent at $(x_1, y_1)$, replace $x^2 → xx_1$, $y^2 → yy_1$.

> 💡 **Slope Form**: $y = mx \pm \sqrt{a^2m^2 + b^2}$ gives both tangents with slope $m$.

> 💡 **Director Circle**: Points on director circle see the ellipse at right angles.

> ⚠️ **Two Tangents**: From external point, exactly 2 tangents; from point on ellipse, 1 tangent; from inside, 0 tangents.

---

## 📋 Summary Table

| Form | Tangent Equation | Contact Point/Condition |
|------|------------------|------------------------|
| At $(x_1, y_1)$ | $\frac{xx_1}{a^2} + \frac{yy_1}{b^2} = 1$ | $(x_1, y_1)$ on ellipse |
| At eccentric angle θ | $\frac{x\cos\theta}{a} + \frac{y\sin\theta}{b} = 1$ | $(a\cos\theta, b\sin\theta)$ |
| With slope $m$ | $y = mx \pm \sqrt{a^2m^2 + b^2}$ | Two tangents |
| Condition for tangency | $c^2 = a^2m^2 + b^2$ | For $y = mx + c$ |
| Director circle | $x^2 + y^2 = a^2 + b^2$ | Perpendicular tangents |

---

## ❓ Quick Revision Questions

1. **Find tangent to $\frac{x^2}{25} + \frac{y^2}{9} = 1$ at $(4, \frac{9}{5})$.**

2. **Find tangent at $\theta = \frac{\pi}{4}$ for $\frac{x^2}{16} + \frac{y^2}{9} = 1$.**

3. **Find tangent to $\frac{x^2}{4} + \frac{y^2}{3} = 1$ with slope 1.**

4. **Find director circle of $\frac{x^2}{25} + \frac{y^2}{16} = 1$.**

5. **Find chord of contact from $(6, 4)$ to $\frac{x^2}{9} + \frac{y^2}{4} = 1$.**

6. **If $y = 3x + c$ is tangent to $\frac{x^2}{16} + \frac{y^2}{9} = 1$, find $c$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $\frac{4x}{25} + \frac{9y/5}{9} = 1$
   $\frac{4x}{25} + \frac{y}{5} = 1$
   **$4x + 5y = 25$**

2. $\frac{x \cdot \frac{1}{\sqrt{2}}}{4} + \frac{y \cdot \frac{1}{\sqrt{2}}}{3} = 1$
   $\frac{x}{4\sqrt{2}} + \frac{y}{3\sqrt{2}} = 1$
   **$3x + 4y = 12\sqrt{2}$**

3. $c = \pm\sqrt{4(1) + 3} = \pm\sqrt{7}$
   **$y = x \pm \sqrt{7}$**

4. $a^2 + b^2 = 25 + 16 = 41$
   **$x^2 + y^2 = 41$**

5. $\frac{6x}{9} + \frac{4y}{4} = 1$
   $\frac{2x}{3} + y = 1$
   **$2x + 3y = 3$**

6. $c^2 = 16(9) + 9 = 153$
   **$c = \pm\sqrt{153} = \pm 3\sqrt{17}$**

</details>

---

## ⏭️ Navigation

| [← Previous: Position of Point and Line](04-position-point-line.md) | [Next: Equation of Normal →](06-equation-of-normal.md) |
|:--------------------------------------------------------------------|--------------------------------------------------------:|
