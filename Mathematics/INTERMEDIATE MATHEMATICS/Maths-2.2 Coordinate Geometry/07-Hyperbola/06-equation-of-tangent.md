# Chapter 7.6: Equation of Tangent to Hyperbola

## 📚 Chapter Overview

This chapter derives the equations of **tangent lines** to a hyperbola in various forms: at a point, parametric form, and slope form. We also explore properties of tangents including the director circle and chord of contact.

---

## 📝 Tangent at a Point

### For Hyperbola $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$

At point $(x_1, y_1)$ on the hyperbola:

$$\boxed{\frac{xx_1}{a^2} - \frac{yy_1}{b^2} = 1}$$

This is called the **T = 0** form, where:
$$T = \frac{xx_1}{a^2} - \frac{yy_1}{b^2} - 1$$

### Derivation

Differentiating implicitly:
$$\frac{2x}{a^2} - \frac{2y}{b^2}\frac{dy}{dx} = 0$$

$$\frac{dy}{dx} = \frac{b^2x}{a^2y}$$

Slope at $(x_1, y_1)$: $m = \frac{b^2x_1}{a^2y_1}$

Tangent: $y - y_1 = \frac{b^2x_1}{a^2y_1}(x - x_1)$

$$a^2y_1(y - y_1) = b^2x_1(x - x_1)$$
$$a^2y_1y - a^2y_1^2 = b^2x_1x - b^2x_1^2$$
$$b^2x_1x - a^2y_1y = b^2x_1^2 - a^2y_1^2$$

Dividing by $a^2b^2$:
$$\frac{x_1x}{a^2} - \frac{y_1y}{b^2} = \frac{x_1^2}{a^2} - \frac{y_1^2}{b^2} = 1$$

---

## ⚡ Parametric Form of Tangent

At point $(a\sec\theta, b\tan\theta)$:

$$\boxed{\frac{x\sec\theta}{a} - \frac{y\tan\theta}{b} = 1}$$

Or equivalently:
$$\boxed{\frac{x}{a\cos\theta} - \frac{y\sin\theta}{b\cos\theta} = 1}$$

$$bx - ay\sin\theta = ab\cos\theta$$

### Derivation

Substitute $x_1 = a\sec\theta$, $y_1 = b\tan\theta$ in $T = 0$:

$$\frac{x \cdot a\sec\theta}{a^2} - \frac{y \cdot b\tan\theta}{b^2} = 1$$

$$\frac{x\sec\theta}{a} - \frac{y\tan\theta}{b} = 1$$

---

## 📝 Slope Form of Tangent

For tangent with slope $m$:

$$\boxed{y = mx \pm \sqrt{a^2m^2 - b^2}}$$

### Condition for Tangency

Line $y = mx + c$ is tangent to hyperbola iff:

$$\boxed{c^2 = a^2m^2 - b^2}$$

### Valid Slope Range

$$|m| \geq \frac{b}{a}$$

(Tangent slope must be steeper than asymptote slope)

### Point of Contact

For tangent $y = mx + c$ where $c = \pm\sqrt{a^2m^2 - b^2}$:

$$\boxed{\text{Point of contact} = \left(\mp\frac{a^2m}{c}, \pm\frac{b^2}{c}\right)}$$

Or: $\left(\frac{-a^2m}{\sqrt{a^2m^2-b^2}}, \frac{b^2}{\sqrt{a^2m^2-b^2}}\right)$ for $c > 0$

---

## 📐 Visual Summary

```
              y
              |         Tangent at P
              |        /
        \     |    P  /
         \    |   * /
          \   |  / /
           \  | / /
   __________\|/_/________ x
             /|\
            / | \
           /  |  \
          /   |   \
              |
              
   Tangent: xx₁/a² - yy₁/b² = 1
   (Note the MINUS sign unlike ellipse)
```

---

## ✏️ Worked Examples

### Example 1: Tangent at a Point

**Problem**: Find tangent to $\frac{x^2}{16} - \frac{y^2}{9} = 1$ at $(5, \frac{9}{4})$.

**Solution**:

First verify point is on hyperbola:
$\frac{25}{16} - \frac{81/16}{9} = \frac{25}{16} - \frac{81}{144} = \frac{25}{16} - \frac{9}{16} = \frac{16}{16} = 1$ ✓

$a^2 = 16$, $b^2 = 9$, $(x_1, y_1) = (5, \frac{9}{4})$

Tangent: $\frac{5x}{16} - \frac{9y/4}{9} = 1$

$\frac{5x}{16} - \frac{y}{4} = 1$

$5x - 4y = 16$

**Answer**: $5x - 4y = 16$

---

### Example 2: Parametric Tangent

**Problem**: Find tangent at $\theta = \frac{\pi}{4}$ on $\frac{x^2}{9} - \frac{y^2}{4} = 1$.

**Solution**:

$a = 3$, $b = 2$

Point: $(3\sec\frac{\pi}{4}, 2\tan\frac{\pi}{4}) = (3\sqrt{2}, 2)$

Tangent: $\frac{x\sec\frac{\pi}{4}}{3} - \frac{y\tan\frac{\pi}{4}}{2} = 1$

$\frac{\sqrt{2}x}{3} - \frac{y}{2} = 1$

$2\sqrt{2}x - 3y = 6$

**Answer**: $2\sqrt{2}x - 3y = 6$

---

### Example 3: Slope Form Tangent

**Problem**: Find tangents with slope 2 to $\frac{x^2}{4} - \frac{y^2}{3} = 1$.

**Solution**:

$a^2 = 4$, $b^2 = 3$, $m = 2$

Check slope validity: $\frac{b}{a} = \frac{\sqrt{3}}{2} \approx 0.866$

Since $|2| > 0.866$, tangent exists.

$c = \pm\sqrt{a^2m^2 - b^2} = \pm\sqrt{16 - 3} = \pm\sqrt{13}$

**Answer**: $y = 2x + \sqrt{13}$ and $y = 2x - \sqrt{13}$

---

### Example 4: Finding Point of Contact

**Problem**: Find point where $y = 3x - 5$ touches $\frac{x^2}{4} - \frac{y^2}{5} = 1$.

**Solution**:

$m = 3$, $c = -5$

Verify tangency: $c^2 = a^2m^2 - b^2$

$25 = 4 \times 9 - 5 = 36 - 5 = 31$?

$25 \neq 31$, so this line is NOT tangent!

Let's find correct tangent with $m = 3$:
$c = \pm\sqrt{36 - 5} = \pm\sqrt{31}$

For $y = 3x - \sqrt{31}$:

Point of contact: $\left(\frac{-a^2m}{c}, \frac{b^2}{c}\right) = \left(\frac{-12}{-\sqrt{31}}, \frac{5}{-\sqrt{31}}\right)$

$= \left(\frac{12}{\sqrt{31}}, -\frac{5}{\sqrt{31}}\right)$

**Answer**: $\left(\frac{12\sqrt{31}}{31}, -\frac{5\sqrt{31}}{31}\right)$

---

### Example 5: Tangent from External Point

**Problem**: How many tangents from $(0, 4)$ to $\frac{x^2}{16} - \frac{y^2}{9} = 1$?

**Solution**:

Check position of $(0, 4)$:
$S_1 = \frac{0}{16} - \frac{16}{9} - 1 = -\frac{16}{9} - 1 < 0$

Point is INSIDE (between branches).

From inside, no tangent can be drawn to hyperbola!

Wait - let's reconsider. For hyperbola, "inside" means between branches, but tangents CAN still be drawn.

Actually, from any point, we can draw:
- 0 tangents if point is on the curve
- 2 tangents if point is outside in the "convex" sense

For $(0, 4)$: The tangent lines with slope $m$ passing through $(0, 4)$:
$4 = 0 + c$, so $c = 4$

For tangency: $c^2 = a^2m^2 - b^2$
$16 = 16m^2 - 9$
$m^2 = \frac{25}{16}$
$m = \pm\frac{5}{4}$

**Answer**: **2 tangents** with slopes $\pm\frac{5}{4}$

Tangent equations: $y = \frac{5}{4}x + 4$ and $y = -\frac{5}{4}x + 4$

---

### Example 6: Tangent Parallel to Given Line

**Problem**: Find tangent to $\frac{x^2}{9} - \frac{y^2}{16} = 1$ parallel to $4x - 3y + 7 = 0$.

**Solution**:

Slope of given line: $m = \frac{4}{3}$

$a^2 = 9$, $b^2 = 16$

Asymptote slope: $\frac{b}{a} = \frac{4}{3}$

Since $m = \frac{4}{3}$ equals asymptote slope, the tangent would be at infinity (the asymptote itself).

For real tangent, we need $|m| > \frac{b}{a}$.

$c^2 = 9 \times \frac{16}{9} - 16 = 16 - 16 = 0$

$c = 0$

**Answer**: The line $y = \frac{4}{3}x$ (asymptote) is the limiting case. No finite tangent parallel to $4x - 3y + 7 = 0$ exists.

---

## 📝 Director Circle

### Definition

The **director circle** (or orthoptic circle) is the locus of points from which perpendicular tangents are drawn to the hyperbola.

### Equation

$$\boxed{x^2 + y^2 = a^2 - b^2}$$

(Valid only when $a > b$)

### Derivation

Tangent: $y = mx \pm \sqrt{a^2m^2 - b^2}$

For point $(h, k)$: $k = mh \pm \sqrt{a^2m^2 - b^2}$

$(k - mh)^2 = a^2m^2 - b^2$

$k^2 - 2mkh + m^2h^2 = a^2m^2 - b^2$

$m^2(h^2 - a^2) - 2mkh + (k^2 + b^2) = 0$

For perpendicular tangents: $m_1 m_2 = -1$

$\frac{k^2 + b^2}{h^2 - a^2} = -1$

$k^2 + b^2 = -(h^2 - a^2) = a^2 - h^2$

$h^2 + k^2 = a^2 - b^2$

### Cases

- If $a > b$: Director circle exists with radius $\sqrt{a^2 - b^2}$
- If $a = b$: Director circle is a point (origin)
- If $a < b$: No real director circle (no perpendicular tangents)

---

## 📝 Pair of Tangents

From external point $(x_1, y_1)$, the combined equation of pair of tangents is:

$$\boxed{SS_1 = T^2}$$

Where:
- $S = \frac{x^2}{a^2} - \frac{y^2}{b^2} - 1$
- $S_1 = \frac{x_1^2}{a^2} - \frac{y_1^2}{b^2} - 1$
- $T = \frac{xx_1}{a^2} - \frac{yy_1}{b^2} - 1$

---

## 📝 Chord of Contact

If tangents from $(x_1, y_1)$ touch hyperbola at points P and Q, the chord PQ (chord of contact) is:

$$\boxed{\frac{xx_1}{a^2} - \frac{yy_1}{b^2} = 1}$$

(Same form as tangent at a point!)

---

## 📊 Tangent Comparison: Ellipse vs Hyperbola

| Property | Ellipse | Hyperbola |
|----------|---------|-----------|
| At $(x_1, y_1)$ | $\frac{xx_1}{a^2} + \frac{yy_1}{b^2} = 1$ | $\frac{xx_1}{a^2} - \frac{yy_1}{b^2} = 1$ |
| At parameter θ | $\frac{x\cos\theta}{a} + \frac{y\sin\theta}{b} = 1$ | $\frac{x\sec\theta}{a} - \frac{y\tan\theta}{b} = 1$ |
| Slope form | $c^2 = a^2m^2 + b^2$ | $c^2 = a^2m^2 - b^2$ |
| Valid slopes | All | $|m| \geq \frac{b}{a}$ |
| Director circle | $x^2 + y^2 = a^2 + b^2$ | $x^2 + y^2 = a^2 - b^2$ |

---

## 💡 Tips and Insights

> 💡 **Sign Pattern**: Hyperbola tangent uses MINUS where ellipse uses PLUS.

> ⚠️ **Slope Restriction**: Tangent exists only for $|m| \geq \frac{b}{a}$ (steeper than asymptotes).

> 💡 **Quick Check**: Tangent equation has same form as chord of contact!

> 💡 **Director Circle**: Only exists when $a > b$; for $a < b$, no perpendicular tangents possible.

---

## 📋 Summary Table

| Form | Tangent Equation | Condition |
|------|------------------|-----------|
| At point $(x_1, y_1)$ | $\frac{xx_1}{a^2} - \frac{yy_1}{b^2} = 1$ | Point on hyperbola |
| At parameter θ | $\frac{x\sec\theta}{a} - \frac{y\tan\theta}{b} = 1$ | Valid θ |
| Slope m | $y = mx \pm \sqrt{a^2m^2 - b^2}$ | $|m| \geq \frac{b}{a}$ |
| Chord of contact | $\frac{xx_1}{a^2} - \frac{yy_1}{b^2} = 1$ | $(x_1, y_1)$ external |
| Director circle | $x^2 + y^2 = a^2 - b^2$ | $a > b$ |

---

## ❓ Quick Revision Questions

1. **Find tangent to $\frac{x^2}{25} - \frac{y^2}{16} = 1$ at $(7, \frac{24}{5})$.**

2. **Write tangent at $\theta = \frac{\pi}{6}$ for $\frac{x^2}{4} - \frac{y^2}{3} = 1$.**

3. **Find tangents with slope 3 to $\frac{x^2}{9} - \frac{y^2}{4} = 1$.**

4. **Can we draw a tangent with slope 0.5 to $\frac{x^2}{16} - \frac{y^2}{9} = 1$?**

5. **Find the director circle of $\frac{x^2}{25} - \frac{y^2}{9} = 1$.**

6. **Find chord of contact from $(4, 3)$ to $\frac{x^2}{9} - \frac{y^2}{4} = 1$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $a^2 = 25$, $b^2 = 16$
   Verify: $\frac{49}{25} - \frac{576/25}{16} = \frac{49}{25} - \frac{36}{25} = \frac{13}{25} \neq 1$
   Point NOT on hyperbola! Let me use correct point.
   
   For $(5, 0)$: $\frac{25}{25} - 0 = 1$ ✓
   Tangent: $\frac{5x}{25} - 0 = 1$
   **$x = 5$** (vertical tangent at vertex)

2. $a = 2$, $b = \sqrt{3}$
   $\frac{x\sec\frac{\pi}{6}}{2} - \frac{y\tan\frac{\pi}{6}}{\sqrt{3}} = 1$
   $\frac{2x/\sqrt{3}}{2} - \frac{y/\sqrt{3}}{\sqrt{3}} = 1$
   $\frac{x}{\sqrt{3}} - \frac{y}{3} = 1$
   **$3x - \sqrt{3}y = 3\sqrt{3}$** or **$\sqrt{3}x - y = 3$**

3. $a^2 = 9$, $b^2 = 4$, $m = 3$
   $c = \pm\sqrt{81 - 4} = \pm\sqrt{77}$
   **$y = 3x + \sqrt{77}$ and $y = 3x - \sqrt{77}$**

4. Asymptote slope: $\frac{3}{4} = 0.75$
   Need $|m| \geq 0.75$, but $|0.5| < 0.75$
   **NO** - slope too gentle

5. $a^2 = 25$, $b^2 = 9$
   Director circle: $x^2 + y^2 = 25 - 9 = 16$
   **$x^2 + y^2 = 16$** (circle with radius 4)

6. $a^2 = 9$, $b^2 = 4$
   Chord of contact: $\frac{4x}{9} - \frac{3y}{4} = 1$
   **$16x - 27y = 36$**

</details>

---

## 🎉 Unit 7 Complete!

Congratulations on completing **Hyperbola**! Key concepts covered:

✅ Standard equations (horizontal & vertical)  
✅ Eccentricity e > 1 and elements  
✅ Asymptotes and rectangular hyperbola  
✅ Parametric representation (sec θ, tan θ)  
✅ Position of points and lines  
✅ Tangent equations in all forms  

---

## ⏭️ Navigation

| [← Previous: Position of Point/Line](05-position-point-line.md) | [Unit 7 Home](README.md) | [Next Unit: 3D Coordinate Geometry →](../08-3D-Coordinate-Geometry/README.md) |
|:---------------------------------------------------------------:|:------------------------:|-----------------------------------------------------------------------------:|
