# Chapter 4.2: General Equation of Circle

## 📚 Chapter Overview

The **general equation** of a circle is the expanded form that allows easy identification of circles from second-degree equations. This chapter covers how to extract the center and radius from the general form and conditions for validity.

---

## ⚡ General Equation

> **General Form of Circle**:
>
> $$\boxed{x^2 + y^2 + 2gx + 2fy + c = 0}$$
>
> Where:
> - **Center**: $(-g, -f)$
> - **Radius**: $r = \sqrt{g^2 + f^2 - c}$

---

## 📐 Derivation

Starting from standard form:
$$(x - h)^2 + (y - k)^2 = r^2$$

Expanding:
$$x^2 - 2hx + h^2 + y^2 - 2ky + k^2 = r^2$$

$$x^2 + y^2 - 2hx - 2ky + (h^2 + k^2 - r^2) = 0$$

Comparing with $x^2 + y^2 + 2gx + 2fy + c = 0$:

| Standard | General |
|----------|---------|
| $-2h$ | $2g$ |
| $-2k$ | $2f$ |
| $h^2 + k^2 - r^2$ | $c$ |

Therefore:
- $h = -g$, $k = -f$
- $r^2 = h^2 + k^2 - c = g^2 + f^2 - c$

---

## 📝 Conditions for Circle

For the equation $x^2 + y^2 + 2gx + 2fy + c = 0$ to represent a **real circle**:

| Condition | Meaning |
|-----------|---------|
| $g^2 + f^2 - c > 0$ | Real circle with positive radius |
| $g^2 + f^2 - c = 0$ | **Point circle** (radius = 0) |
| $g^2 + f^2 - c < 0$ | **Imaginary circle** (no real locus) |

---

## 📝 Identifying Circle from General Second Degree

A general second-degree equation:
$$ax^2 + 2hxy + by^2 + 2gx + 2fy + c = 0$$

represents a circle if:

1. **Coefficient of $x^2$ = Coefficient of $y^2$**: $a = b$
2. **No $xy$ term**: $h = 0$

```
    ┌────────────────────────────────────────────────────────┐
    │     GENERAL SECOND DEGREE → CIRCLE CONDITIONS          │
    ├────────────────────────────────────────────────────────┤
    │                                                        │
    │     ax² + 2hxy + by² + 2gx + 2fy + c = 0               │
    │                                                        │
    │     For CIRCLE:                                        │
    │     ✓ a = b  (equal coefficients of x² and y²)         │
    │     ✓ h = 0  (no xy term)                              │
    │                                                        │
    │     Divide by a to get:                                │
    │     x² + y² + (2g/a)x + (2f/a)y + (c/a) = 0            │
    │                                                        │
    └────────────────────────────────────────────────────────┘
```

---

## ✏️ Worked Examples

### Example 1: Finding Center and Radius

**Problem**: Find the center and radius of $x^2 + y^2 - 6x + 4y - 12 = 0$.

**Solution**:

Comparing with $x^2 + y^2 + 2gx + 2fy + c = 0$:
- $2g = -6$ → $g = -3$
- $2f = 4$ → $f = 2$
- $c = -12$

**Center** = $(-g, -f) = (3, -2)$

**Radius** = $\sqrt{g^2 + f^2 - c} = \sqrt{9 + 4 + 12} = \sqrt{25} = 5$

---

### Example 2: Converting to Standard Form

**Problem**: Convert $x^2 + y^2 + 8x - 10y + 5 = 0$ to standard form.

**Solution**:

**Complete the square**:

$(x^2 + 8x) + (y^2 - 10y) + 5 = 0$

$(x^2 + 8x + 16) + (y^2 - 10y + 25) + 5 - 16 - 25 = 0$

$(x + 4)^2 + (y - 5)^2 = 36$

**Standard form**: $(x + 4)^2 + (y - 5)^2 = 36$

Center: $(-4, 5)$, Radius: $6$

---

### Example 3: Checking if Equation is a Circle

**Problem**: Determine if $3x^2 + 3y^2 - 12x + 18y - 9 = 0$ represents a circle.

**Solution**:

**Step 1**: Check conditions
- Coefficient of $x^2$ = Coefficient of $y^2$ = 3 ✓
- No $xy$ term ✓

**Step 2**: Divide by 3
$$x^2 + y^2 - 4x + 6y - 3 = 0$$

**Step 3**: Find center and radius
$g = -2$, $f = 3$, $c = -3$

$r^2 = 4 + 9 + 3 = 16 > 0$ ✓

**Answer**: Yes, it's a circle with center $(2, -3)$ and radius $4$.

---

### Example 4: Point Circle

**Problem**: For what value of k does $x^2 + y^2 - 6x + 8y + k = 0$ represent a point circle?

**Solution**:

$g = -3$, $f = 4$, $c = k$

For point circle: $g^2 + f^2 - c = 0$

$$9 + 16 - k = 0$$
$$k = 25$$

**Verification**: When $k = 25$, the equation becomes $(x-3)^2 + (y+4)^2 = 0$, which is satisfied only by $(3, -4)$.

---

### Example 5: Circle Through Origin

**Problem**: Show that circle $x^2 + y^2 + 2gx + 2fy = 0$ always passes through origin.

**Solution**:

Substitute $(0, 0)$:
$$0 + 0 + 0 + 0 = 0$$ ✓

Since $(0, 0)$ satisfies the equation, the circle passes through origin.

**Note**: When $c = 0$, circle passes through origin.

---

### Example 6: Finding Equation Given Conditions

**Problem**: Find the equation of circle with center $(-1, 2)$ passing through $(2, 6)$.

**Solution**:

**Step 1**: Find radius
$$r = \sqrt{(2-(-1))^2 + (6-2)^2} = \sqrt{9 + 16} = 5$$

**Step 2**: Write general form
Center $(-1, 2)$ means $g = 1$, $f = -2$

From $r^2 = g^2 + f^2 - c$:
$$25 = 1 + 4 - c$$
$$c = -20$$

**Answer**: $x^2 + y^2 + 2x - 4y - 20 = 0$

---

### Example 7: Circle with Given Diameter

**Problem**: Find equation of circle having line segment from $(1, 2)$ to $(3, 4)$ as diameter.

**Solution**:

**Using diameter form**:
$$(x - 1)(x - 3) + (y - 2)(y - 4) = 0$$

$$x^2 - 4x + 3 + y^2 - 6y + 8 = 0$$

$$x^2 + y^2 - 4x - 6y + 11 = 0$$

**Verification**:
- $g = -2$, $f = -3$, $c = 11$
- Center = $(2, 3)$ ✓ (midpoint of diameter)
- $r^2 = 4 + 9 - 11 = 2$ ✓

---

## 📝 Finding Equation Through Three Points

Given three points, the circle equation can be found by:

### Method 1: Direct Substitution

Substitute each point in $x^2 + y^2 + 2gx + 2fy + c = 0$ to get three equations in $g$, $f$, $c$.

### Method 2: Determinant Method

$$\begin{vmatrix} x^2 + y^2 & x & y & 1 \\ x_1^2 + y_1^2 & x_1 & y_1 & 1 \\ x_2^2 + y_2^2 & x_2 & y_2 & 1 \\ x_3^2 + y_3^2 & x_3 & y_3 & 1 \end{vmatrix} = 0$$

---

## 📝 Intercepts Made by Circle

### X-axis Intercept

The circle $x^2 + y^2 + 2gx + 2fy + c = 0$ cuts x-axis where $y = 0$:

$$x^2 + 2gx + c = 0$$

**X-intercept length** = $|x_1 - x_2| = 2\sqrt{g^2 - c}$

Conditions:
- $g^2 > c$: Cuts x-axis at two points
- $g^2 = c$: Touches x-axis
- $g^2 < c$: Doesn't meet x-axis

### Y-axis Intercept

Similarly, **Y-intercept length** = $2\sqrt{f^2 - c}$

---

## 💡 Tips and Insights

> 💡 **Memory Aid**: In $x^2 + y^2 + 2gx + 2fy + c = 0$, center is $(-g, -f)$ — just negate and halve the linear coefficients.

> 💡 **Quick Radius Check**: Before solving, ensure $g^2 + f^2 > c$ for a real circle.

> ⚠️ **Coefficient Unity**: The general form requires coefficient of $x^2$ and $y^2$ to be 1. Divide through if necessary.

> 💡 **Origin Test**: If $c = 0$, circle passes through origin.

---

## 🌍 Real-World Applications

1. **GPS Systems**: Circular regions for location services
2. **Radar**: Detection range circles
3. **Civil Engineering**: Roundabout design
4. **Astronomy**: Orbital calculations
5. **Manufacturing**: Quality control for circular parts

---

## 📋 Summary Table

| Property | Formula |
|----------|---------|
| General equation | $x^2 + y^2 + 2gx + 2fy + c = 0$ |
| Center | $(-g, -f)$ |
| Radius | $\sqrt{g^2 + f^2 - c}$ |
| Real circle | $g^2 + f^2 - c > 0$ |
| Point circle | $g^2 + f^2 - c = 0$ |
| Passes through origin | $c = 0$ |
| Touches x-axis | $g^2 = c$ |
| Touches y-axis | $f^2 = c$ |
| X-intercept length | $2\sqrt{g^2 - c}$ |
| Y-intercept length | $2\sqrt{f^2 - c}$ |

---

## ❓ Quick Revision Questions

1. **Find center and radius of $x^2 + y^2 + 4x - 6y - 3 = 0$.**

2. **Convert $2x^2 + 2y^2 - 8x + 12y - 6 = 0$ to standard form.**

3. **For what value of k does $x^2 + y^2 - 4x + 6y + k = 0$ represent a point circle?**

4. **Find the equation of circle with center $(2, -3)$ and radius $4$.**

5. **Find the length of intercepts on coordinate axes by $x^2 + y^2 - 6x - 8y = 0$.**

6. **Show that points $(0, 0)$, $(6, 0)$, $(0, 8)$ lie on a circle and find its equation.**

<details>
<summary><b>Click to see answers</b></summary>

1. $g = 2$, $f = -3$, $c = -3$
   Center = $(-2, 3)$
   Radius = $\sqrt{4 + 9 + 3} = 4$

2. Divide by 2: $x^2 + y^2 - 4x + 6y - 3 = 0$
   Complete squares: $(x-2)^2 + (y+3)^2 = 4 + 9 + 3 = 16$
   **$(x-2)^2 + (y+3)^2 = 16$**

3. $g = -2$, $f = 3$, $c = k$
   Point circle: $g^2 + f^2 - c = 0$
   $4 + 9 - k = 0$
   **$k = 13$**

4. General form: $g = -2$, $f = 3$, $r = 4$
   $c = g^2 + f^2 - r^2 = 4 + 9 - 16 = -3$
   **$x^2 + y^2 - 4x + 6y - 3 = 0$**

5. $g = -3$, $f = -4$, $c = 0$
   X-intercept = $2\sqrt{9 - 0} = 6$
   Y-intercept = $2\sqrt{16 - 0} = 8$

6. Using diameter form with $(0,0)$ and $(6,0)$, $(0,8)$:
   These form a right triangle with hypotenuse from $(6,0)$ to $(0,8)$ as diameter.
   $(x-6)(x-0) + (y-0)(y-8) = 0$
   **$x^2 + y^2 - 6x - 8y = 0$**
   Center = $(3, 4)$, Radius = $5$

</details>

---

## ⏭️ Navigation

| [← Previous: Standard Form](01-standard-form.md) | [Next: Parametric Form →](03-parametric-form.md) |
|:------------------------------------------------|------------------------------------------------:|
