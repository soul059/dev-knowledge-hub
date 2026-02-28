# Chapter 6.4: Position of Point and Line Relative to Ellipse

## 📚 Chapter Overview

Understanding whether a point lies inside, on, or outside an ellipse—and whether a line intersects, touches, or misses an ellipse—is fundamental for many applications. This chapter develops the algebraic conditions for these relationships.

---

## 📝 Position of a Point

For ellipse $S: \frac{x^2}{a^2} + \frac{y^2}{b^2} - 1 = 0$

Define:
$$S_1 = \frac{x_1^2}{a^2} + \frac{y_1^2}{b^2} - 1$$

### Position Criteria

| Condition | Position |
|-----------|----------|
| $S_1 < 0$ | Point $(x_1, y_1)$ is **inside** ellipse |
| $S_1 = 0$ | Point $(x_1, y_1)$ is **on** ellipse |
| $S_1 > 0$ | Point $(x_1, y_1)$ is **outside** ellipse |

```
                    ╭───────────────╮
                  ╱                   ╲
                ╱    S₁ < 0            ╲
               │     (Inside)           │
    ───────────●───────●────────────────●───────────
               │       O                │
                ╲                      ╱
                  ╲   S₁ = 0 (On)    ╱
                    ╲             ╱
                      ╰─────────╯
                      
                         S₁ > 0 (Outside)
```

---

## ✏️ Examples: Position of Point

### Example 1: Check Position

**Problem**: Determine position of $(3, 1)$ relative to $\frac{x^2}{25} + \frac{y^2}{9} = 1$.

**Solution**:

$S_1 = \frac{9}{25} + \frac{1}{9} - 1 = \frac{81 + 25 - 225}{225} = \frac{-119}{225} < 0$

**Answer**: Point is **inside** the ellipse.

---

### Example 2: Point on Ellipse

**Problem**: Check if $(4, \frac{9}{5})$ lies on $\frac{x^2}{25} + \frac{y^2}{9} = 1$.

**Solution**:

$S_1 = \frac{16}{25} + \frac{81/25}{9} - 1 = \frac{16}{25} + \frac{81}{225} - 1$

$= \frac{144 + 81 - 225}{225} = \frac{0}{225} = 0$

**Answer**: Point is **on** the ellipse.

---

## 📝 Position of a Line

### General Analysis

For ellipse $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$ and line $y = mx + c$:

Substituting line into ellipse:
$$\frac{x^2}{a^2} + \frac{(mx + c)^2}{b^2} = 1$$

$$b^2x^2 + a^2(mx + c)^2 = a^2b^2$$

$$(b^2 + a^2m^2)x^2 + 2a^2mcx + (a^2c^2 - a^2b^2) = 0$$

This is a quadratic in x. Let discriminant = $\Delta$.

---

### Line-Ellipse Relationship

| Condition | Relationship |
|-----------|--------------|
| $\Delta > 0$ | Line is **secant** (intersects at 2 points) |
| $\Delta = 0$ | Line is **tangent** (touches at 1 point) |
| $\Delta < 0$ | Line **doesn't intersect** ellipse |

---

## 📐 Condition for Tangency

For $\Delta = 0$:

$$(2a^2mc)^2 - 4(b^2 + a^2m^2)(a^2c^2 - a^2b^2) = 0$$

$$4a^4m^2c^2 - 4a^2(b^2 + a^2m^2)(c^2 - b^2) = 0$$

$$a^2m^2c^2 - (b^2 + a^2m^2)(c^2 - b^2) = 0$$

$$a^2m^2c^2 - b^2c^2 + b^4 - a^2m^2c^2 + a^2m^2b^2 = 0$$

$$-b^2c^2 + b^4 + a^2m^2b^2 = 0$$

$$c^2 = b^2 + a^2m^2$$

$$\boxed{c = \pm\sqrt{a^2m^2 + b^2}}$$

This is the **condition for tangency**.

---

## ⚡ Summary: Line Relationships

For line $y = mx + c$ and ellipse $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$:

| Condition | Line Type |
|-----------|-----------|
| $c^2 > a^2m^2 + b^2$ | No intersection |
| $c^2 = a^2m^2 + b^2$ | Tangent |
| $c^2 < a^2m^2 + b^2$ | Secant (2 points) |

---

## ✏️ Examples: Position of Line

### Example 3: Secant, Tangent, or Neither?

**Problem**: Classify line $y = x + 4$ relative to $\frac{x^2}{9} + \frac{y^2}{4} = 1$.

**Solution**:

$a^2 = 9$, $b^2 = 4$, $m = 1$, $c = 4$

$a^2m^2 + b^2 = 9(1) + 4 = 13$

$c^2 = 16$

Since $16 > 13$: $c^2 > a^2m^2 + b^2$

**Answer**: Line **does not intersect** the ellipse.

---

### Example 4: Finding Tangent Condition

**Problem**: For what value of $k$ is $y = 2x + k$ tangent to $\frac{x^2}{16} + \frac{y^2}{4} = 1$?

**Solution**:

$a^2 = 16$, $b^2 = 4$, $m = 2$

For tangency: $k^2 = a^2m^2 + b^2 = 16(4) + 4 = 68$

$k = \pm\sqrt{68} = \pm 2\sqrt{17}$

**Answer**: $k = \pm 2\sqrt{17}$

---

### Example 5: Intersection Points

**Problem**: Find where $y = x + 2$ intersects $\frac{x^2}{8} + \frac{y^2}{4} = 1$.

**Solution**:

Substituting $y = x + 2$:
$$\frac{x^2}{8} + \frac{(x+2)^2}{4} = 1$$

$$\frac{x^2}{8} + \frac{x^2 + 4x + 4}{4} = 1$$

$$x^2 + 2(x^2 + 4x + 4) = 8$$

$$3x^2 + 8x + 8 = 8$$

$$3x^2 + 8x = 0$$

$$x(3x + 8) = 0$$

$x = 0$ or $x = -\frac{8}{3}$

Points: $(0, 2)$ and $\left(-\frac{8}{3}, -\frac{2}{3}\right)$

**Answer**: $(0, 2)$ and $\left(-\frac{8}{3}, -\frac{2}{3}\right)$

---

## 📝 Length of Chord

If line $y = mx + c$ intersects ellipse at two points, the chord length is:

$$\text{Length} = \sqrt{1 + m^2} \cdot |x_1 - x_2|$$

where $x_1, x_2$ are roots of the quadratic.

Using $|x_1 - x_2| = \frac{\sqrt{\Delta}}{a^2m^2 + b^2}$ (adjusted coefficient):

$$\boxed{\text{Chord Length} = \frac{2ab\sqrt{(a^2m^2 + b^2) - c^2}}{a^2m^2 + b^2} \cdot \sqrt{1 + m^2}}$$

---

### Example 6: Chord Length

**Problem**: Find length of chord cut by $y = x$ from $\frac{x^2}{16} + \frac{y^2}{9} = 1$.

**Solution**:

$a^2 = 16$, $b^2 = 9$, $m = 1$, $c = 0$

Substituting $y = x$:
$$\frac{x^2}{16} + \frac{x^2}{9} = 1$$
$$x^2\left(\frac{9 + 16}{144}\right) = 1$$
$$x^2 = \frac{144}{25}$$
$$x = \pm\frac{12}{5}$$

Points: $\left(\frac{12}{5}, \frac{12}{5}\right)$ and $\left(-\frac{12}{5}, -\frac{12}{5}\right)$

Length = $\sqrt{\left(\frac{24}{5}\right)^2 + \left(\frac{24}{5}\right)^2} = \frac{24}{5}\sqrt{2}$

**Answer**: $\frac{24\sqrt{2}}{5}$

---

## 📝 Chord with Given Midpoint

If $(h, k)$ is the midpoint of a chord of ellipse $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$:

Equation of chord: 

$$\boxed{\frac{xh}{a^2} + \frac{yk}{b^2} = \frac{h^2}{a^2} + \frac{k^2}{b^2}}$$

Or using the **T = S₁** notation:

$$T = S_1$$

where $T = \frac{xh}{a^2} + \frac{yk}{b^2} - 1$ and $S_1 = \frac{h^2}{a^2} + \frac{k^2}{b^2} - 1$

---

### Example 7: Chord with Midpoint

**Problem**: Find chord of $\frac{x^2}{36} + \frac{y^2}{9} = 1$ with midpoint $(2, 1)$.

**Solution**:

Using $T = S_1$:

$\frac{2x}{36} + \frac{y}{9} = \frac{4}{36} + \frac{1}{9}$

$\frac{x}{18} + \frac{y}{9} = \frac{1}{9} + \frac{1}{9}$

$\frac{x}{18} + \frac{y}{9} = \frac{2}{9}$

$\frac{x + 2y}{18} = \frac{2}{9}$

$x + 2y = 4$

**Answer**: $x + 2y = 4$

---

## 💡 Tips and Insights

> 💡 **$S_1$ Test**: Quick way to check if point is inside/on/outside ellipse.

> 💡 **Tangent Condition**: $c^2 = a^2m^2 + b^2$ is the key criterion for tangency.

> ⚠️ **Sign Check**: $S_1 < 0$ means inside (ellipse equation gives values < 1 inside).

> 💡 **Chord Midpoint**: Use $T = S_1$ for chord with given midpoint.

---

## 📋 Summary Table

| Concept | Condition/Formula |
|---------|-------------------|
| Point inside | $S_1 < 0$ |
| Point on ellipse | $S_1 = 0$ |
| Point outside | $S_1 > 0$ |
| Line secant | $c^2 < a^2m^2 + b^2$ |
| Line tangent | $c^2 = a^2m^2 + b^2$ |
| Line no intersection | $c^2 > a^2m^2 + b^2$ |
| Chord with midpoint $(h,k)$ | $\frac{xh}{a^2} + \frac{yk}{b^2} = \frac{h^2}{a^2} + \frac{k^2}{b^2}$ |

---

## ❓ Quick Revision Questions

1. **Is $(1, 1)$ inside, on, or outside $\frac{x^2}{4} + \frac{y^2}{3} = 1$?**

2. **Classify $y = 2x + 5$ relative to $\frac{x^2}{9} + \frac{y^2}{4} = 1$.**

3. **Find $k$ if $y = 3x + k$ is tangent to $\frac{x^2}{4} + \frac{y^2}{9} = 1$.**

4. **Find intersection points of $x + y = 3$ and $\frac{x^2}{9} + \frac{y^2}{4} = 1$.**

5. **Find equation of chord of $\frac{x^2}{25} + \frac{y^2}{16} = 1$ with midpoint $(1, 2)$.**

6. **Find length of latus rectum chord for $\frac{x^2}{25} + \frac{y^2}{16} = 1$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $S_1 = \frac{1}{4} + \frac{1}{3} - 1 = \frac{3 + 4 - 12}{12} = \frac{-5}{12} < 0$
   **Inside**

2. $a^2m^2 + b^2 = 9(4) + 4 = 40$
   $c^2 = 25$
   $25 < 40$, so $c^2 < a^2m^2 + b^2$
   **Secant** (intersects at 2 points)

3. $k^2 = a^2m^2 + b^2 = 4(9) + 9 = 45$
   **$k = \pm 3\sqrt{5}$**

4. Substitute $y = 3 - x$:
   $\frac{x^2}{9} + \frac{(3-x)^2}{4} = 1$
   $4x^2 + 9(9 - 6x + x^2) = 36$
   $4x^2 + 81 - 54x + 9x^2 = 36$
   $13x^2 - 54x + 45 = 0$
   $x = \frac{54 \pm \sqrt{2916 - 2340}}{26} = \frac{54 \pm 24}{26}$
   $x = 3$ or $x = \frac{30}{26} = \frac{15}{13}$
   **$(3, 0)$ and $\left(\frac{15}{13}, \frac{24}{13}\right)$**

5. $\frac{x}{25} + \frac{2y}{16} = \frac{1}{25} + \frac{4}{16}$
   $\frac{x}{25} + \frac{y}{8} = \frac{1}{25} + \frac{1}{4} = \frac{4 + 25}{100} = \frac{29}{100}$
   $\frac{8x + 25y}{200} = \frac{29}{100}$
   $8x + 25y = 58$
   **$8x + 25y = 58$**

6. Latus rectum = $\frac{2b^2}{a} = \frac{32}{5}$
   **Length = $\frac{32}{5}$**

</details>

---

## ⏭️ Navigation

| [← Previous: Parametric Form](03-parametric-form.md) | [Next: Equation of Tangent →](05-equation-of-tangent.md) |
|:-----------------------------------------------------|----------------------------------------------------------:|
