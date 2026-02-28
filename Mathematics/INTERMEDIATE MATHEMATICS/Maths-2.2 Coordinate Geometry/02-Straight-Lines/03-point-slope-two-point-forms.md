# Chapter 2.3: Point-Slope and Two-Point Forms

## 📚 Chapter Overview

When we know a point on a line and its slope, or when we know two points on a line, we can write the equation using the **point-slope form** or **two-point form**. These forms are particularly useful in deriving line equations from given geometric conditions.

---

## 📝 Point-Slope Form

### Definition

> **Point-Slope Form**: A line with slope $m$ passing through point $(x_1, y_1)$ has the equation:
>
> $$\boxed{y - y_1 = m(x - x_1)}$$

### Derivation

Let the line have slope $m$ and pass through fixed point $P(x_1, y_1)$.

For any other point $Q(x, y)$ on the line:
$$\text{Slope} = \frac{y - y_1}{x - x_1} = m$$

Rearranging:
$$y - y_1 = m(x - x_1)$$

```
        Y
        ↑
        │           Q(x, y)
        │           ●
        │          /
        │         /  slope = m
        │        /
        │   P(x₁,y₁)
        │       ●
        │      /
    ────O─────/──────→ X
```

---

## ✏️ Examples: Point-Slope Form

### Example 1: Basic Application

**Problem**: Find the equation of a line with slope 3 passing through (2, 5).

**Solution**:

Using point-slope form:
$$y - y_1 = m(x - x_1)$$
$$y - 5 = 3(x - 2)$$
$$y - 5 = 3x - 6$$
$$y = 3x - 1$$

**Answer**: $y = 3x - 1$ or $3x - y - 1 = 0$

---

### Example 2: With Negative Values

**Problem**: Find the equation of a line with slope $-\frac{2}{3}$ passing through (−3, 4).

**Solution**:

$$y - 4 = -\frac{2}{3}(x - (-3))$$
$$y - 4 = -\frac{2}{3}(x + 3)$$
$$y - 4 = -\frac{2}{3}x - 2$$
$$y = -\frac{2}{3}x + 2$$

**Or in general form**:
$$3y = -2x + 6$$
$$2x + 3y - 6 = 0$$

---

### Example 3: Line Through a Point with Given Inclination

**Problem**: Find the equation of a line through (1, 2) making an angle of 45° with the X-axis.

**Solution**:

Slope $m = \tan 45° = 1$

Using point-slope form:
$$y - 2 = 1(x - 1)$$
$$y = x + 1$$

**Answer**: $y = x + 1$ or $x - y + 1 = 0$

---

## 📝 Two-Point Form

### Definition

> **Two-Point Form**: A line passing through points $(x_1, y_1)$ and $(x_2, y_2)$ has the equation:
>
> $$\boxed{\frac{y - y_1}{y_2 - y_1} = \frac{x - x_1}{x_2 - x_1}}$$

### Alternative Form

$$\boxed{y - y_1 = \frac{y_2 - y_1}{x_2 - x_1}(x - x_1)}$$

### Derivation

For a line through $P(x_1, y_1)$ and $Q(x_2, y_2)$:

First, find the slope:
$$m = \frac{y_2 - y_1}{x_2 - x_1}$$

Then use point-slope form with point $(x_1, y_1)$:
$$y - y_1 = \frac{y_2 - y_1}{x_2 - x_1}(x - x_1)$$

```
        Y
        ↑
        │           Q(x₂, y₂)
        │           ●
        │          /
        │         /
        │        /
        │   P(x₁,y₁)
        │       ●
        │      /
    ────O─────/──────→ X
        │
```

---

## ✏️ Examples: Two-Point Form

### Example 4: Basic Two-Point Form

**Problem**: Find the equation of the line passing through (1, 3) and (4, 9).

**Solution**:

Using two-point form:
$$\frac{y - 3}{9 - 3} = \frac{x - 1}{4 - 1}$$

$$\frac{y - 3}{6} = \frac{x - 1}{3}$$

Cross-multiplying:
$$3(y - 3) = 6(x - 1)$$
$$3y - 9 = 6x - 6$$
$$6x - 3y + 3 = 0$$
$$2x - y + 1 = 0$$

**Or**: $y = 2x + 1$

---

### Example 5: Line Through Origin

**Problem**: Find the equation of the line through (0, 0) and (3, 5).

**Solution**:

$$\frac{y - 0}{5 - 0} = \frac{x - 0}{3 - 0}$$

$$\frac{y}{5} = \frac{x}{3}$$

$$3y = 5x$$

**Answer**: $5x - 3y = 0$ or $y = \frac{5}{3}x$

---

### Example 6: With Negative Coordinates

**Problem**: Find the equation of the line through (−2, 5) and (3, −10).

**Solution**:

$$\frac{y - 5}{-10 - 5} = \frac{x - (-2)}{3 - (-2)}$$

$$\frac{y - 5}{-15} = \frac{x + 2}{5}$$

Cross-multiplying:
$$5(y - 5) = -15(x + 2)$$
$$5y - 25 = -15x - 30$$
$$15x + 5y + 5 = 0$$
$$3x + y + 1 = 0$$

**Answer**: $3x + y + 1 = 0$ or $y = -3x - 1$

---

## 📝 Special Cases

### Case 1: Horizontal Line (Same y-coordinates)

If $y_1 = y_2 = k$, the line is horizontal:

**Equation**: $y = k$

```
        Y
        ↑
        │
    ────●────────●────→  y = k
      k │P       Q
        │
    ────O────────────→ X
```

### Case 2: Vertical Line (Same x-coordinates)

If $x_1 = x_2 = k$, the line is vertical:

**Equation**: $x = k$

```
        Y
        ↑    │
        │    ●Q
        │    │
        │    ●P
        │    │  x = k
    ────O────│───→ X
             k
```

---

## 📝 Symmetric Form

The two-point form can be written in symmetric form:

$$\frac{x - x_1}{x_2 - x_1} = \frac{y - y_1}{y_2 - y_1}$$

Or with direction ratios $(a, b)$:

$$\frac{x - x_1}{a} = \frac{y - y_1}{b}$$

This form is useful for parametric representation.

---

## 📝 Parametric Form

From the symmetric form, we can write:

$$\frac{x - x_1}{x_2 - x_1} = \frac{y - y_1}{y_2 - y_1} = t$$

Where $t$ is a parameter. This gives:

$$x = x_1 + t(x_2 - x_1) = (1-t)x_1 + tx_2$$
$$y = y_1 + t(y_2 - y_1) = (1-t)y_1 + ty_2$$

| Value of t | Point Position |
|------------|----------------|
| t = 0 | At point P$(x_1, y_1)$ |
| t = 1 | At point Q$(x_2, y_2)$ |
| 0 < t < 1 | Between P and Q |
| t < 0 or t > 1 | Extended beyond P or Q |

---

## ✏️ More Examples

### Example 7: Median of a Triangle

**Problem**: Find the equation of the median from vertex A(2, 3) to side BC, where B = (4, 7) and C = (6, 1).

**Solution**:

**Step 1**: Find midpoint M of BC
$$M = \left(\frac{4+6}{2}, \frac{7+1}{2}\right) = (5, 4)$$

**Step 2**: Find equation of line AM using two-point form
$$\frac{y - 3}{4 - 3} = \frac{x - 2}{5 - 2}$$

$$\frac{y - 3}{1} = \frac{x - 2}{3}$$

$$3(y - 3) = x - 2$$
$$3y - 9 = x - 2$$
$$x - 3y + 7 = 0$$

**Answer**: $x - 3y + 7 = 0$

---

### Example 8: Altitude of a Triangle

**Problem**: Find the equation of the altitude from A(1, 4) to side BC, where B = (0, 0) and C = (6, 0).

**Solution**:

**Step 1**: Find slope of BC
$$m_{BC} = \frac{0 - 0}{6 - 0} = 0$$ (horizontal line)

**Step 2**: Altitude is perpendicular to BC
Since BC is horizontal, the altitude is vertical.
A vertical line through A(1, 4) has equation:

$$x = 1$$

**Answer**: $x = 1$

---

### Example 9: Line Through Intercepts

**Problem**: A line passes through the intercepts (a, 0) on X-axis and (0, b) on Y-axis. Find its equation.

**Solution**:

Using two-point form with $(a, 0)$ and $(0, b)$:

$$\frac{y - 0}{b - 0} = \frac{x - a}{0 - a}$$

$$\frac{y}{b} = \frac{x - a}{-a}$$

$$\frac{y}{b} = -\frac{x - a}{a}$$

$$\frac{y}{b} = -\frac{x}{a} + 1$$

$$\frac{x}{a} + \frac{y}{b} = 1$$

This is the **intercept form** (covered in the next chapter).

---

## 📝 Comparison of Forms

| Form | Equation | Best Used When |
|------|----------|----------------|
| Point-Slope | $y - y_1 = m(x - x_1)$ | Slope and one point known |
| Two-Point | $\frac{y - y_1}{y_2 - y_1} = \frac{x - x_1}{x_2 - x_1}$ | Two points known |
| Slope-Intercept | $y = mx + c$ | Slope and y-intercept known |

---

## 💡 Tips and Insights

> 💡 **Choice of Point**: In two-point form, you can use either point as $(x_1, y_1)$ — the result is the same.

> 💡 **Quick Check**: Substitute both given points into your final equation to verify.

> ⚠️ **Division by Zero**: Two-point form doesn't work directly for vertical lines (use $x = k$ instead).

> 💡 **Conversion**: You can always convert any form to slope-intercept form by solving for y.

> 💡 **Same Line**: Equations that look different may represent the same line (e.g., $2x - y + 1 = 0$ and $y = 2x + 1$).

---

## 🌍 Real-World Applications

1. **Data Analysis**: Finding trend lines through data points
2. **Computer Graphics**: Drawing lines between vertices
3. **Navigation**: Plotting course between two waypoints
4. **Physics**: Trajectory of objects moving in straight lines
5. **Economics**: Linear interpolation between known values
6. **Engineering**: Designing straight structural members

---

## 📋 Summary Table

| Concept | Formula |
|---------|---------|
| Point-Slope Form | $y - y_1 = m(x - x_1)$ |
| Two-Point Form | $\frac{y - y_1}{y_2 - y_1} = \frac{x - x_1}{x_2 - x_1}$ |
| Symmetric Form | $\frac{x - x_1}{x_2 - x_1} = \frac{y - y_1}{y_2 - y_1}$ |
| Parametric Form | $x = x_1 + t(x_2 - x_1)$, $y = y_1 + t(y_2 - y_1)$ |
| Horizontal line | $y = k$ (same y-coordinates) |
| Vertical line | $x = k$ (same x-coordinates) |

---

## ❓ Quick Revision Questions

1. **Find the equation of a line with slope −2 passing through (3, 1).**

2. **Write the equation of the line through (2, 5) and (6, 13).**

3. **Find the equation of the line through (−1, 3) and (2, 3).**

4. **A line passes through (4, −2) and is parallel to the line joining (1, 1) and (3, 5). Find its equation.**

5. **Find the equation of the perpendicular bisector of the segment joining (2, 4) and (6, 8).**

6. **Write the parametric equations of the line through (1, 2) and (5, 10).**

<details>
<summary><b>Click to see answers</b></summary>

1. $y - 1 = -2(x - 3)$
   $y = -2x + 7$ or $2x + y - 7 = 0$

2. Slope = $\frac{13-5}{6-2} = \frac{8}{4} = 2$
   $y - 5 = 2(x - 2)$
   $y = 2x + 1$ or $2x - y + 1 = 0$

3. Same y-coordinates (both 3), so it's a horizontal line.
   **Equation**: $y = 3$

4. Slope of given line = $\frac{5-1}{3-1} = 2$
   Parallel line through (4, −2):
   $y - (-2) = 2(x - 4)$
   $y = 2x - 10$ or $2x - y - 10 = 0$

5. Midpoint = $(4, 6)$
   Slope of segment = $\frac{8-4}{6-2} = 1$
   Slope of perpendicular = −1
   $y - 6 = -1(x - 4)$
   $y = -x + 10$ or $x + y - 10 = 0$

6. $x = 1 + 4t$, $y = 2 + 8t$
   (or equivalently: $x = 1 + t$, $y = 2 + 2t$ with different parameterization)

</details>

---

## ⏭️ Navigation

| [← Previous: Slope-Intercept Form](02-slope-intercept-form.md) | [Next: Intercept & Normal Forms →](04-intercept-normal-forms.md) |
|:--------------------------------------------------------------|----------------------------------------------------------------:|
