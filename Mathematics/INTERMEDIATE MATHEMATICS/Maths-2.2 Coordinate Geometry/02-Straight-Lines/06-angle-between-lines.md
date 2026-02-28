# Chapter 2.6: Angle Between Two Lines

## 📚 Chapter Overview

When two non-parallel lines intersect, they form angles. This chapter derives the formula for finding the angle between two lines and explores its applications in geometry and real-world problems.

---

## 📝 Concept of Angle Between Lines

When two lines intersect, they form two pairs of vertically opposite angles. We typically refer to the **acute angle** between the lines, unless specified otherwise.

```
             │\        /│
             │ \      / │
             │  \θ  θ/  │
             │   \  /   │
             │    \/    │
    ─────────┼────●─────┼───────
             │   /\     │
             │  /  \    │
             │ /    \   │
             │/      \  │
             │        \ │
             
    θ and (180° - θ) are the angles formed
    Acute angle = θ (if θ < 90°)
```

---

## 📝 Derivation of Formula

### Using Inclinations

Let two lines have inclinations $\theta_1$ and $\theta_2$ with the X-axis, with slopes:
- $m_1 = \tan\theta_1$
- $m_2 = \tan\theta_2$

```
        Y
        ↑           /L₂
        │          /
        │    L₁   /
        │    \   /
        │     \ /
        │      ●
        │     /│\
        │    / │ \
        │   /θ₁│θ₂\
    ────O──/───┼───\───→ X
```

The angle $\phi$ between the lines is:
$$\phi = \theta_2 - \theta_1 \quad \text{or} \quad \phi = \theta_1 - \theta_2$$

Using the tangent subtraction formula:
$$\tan\phi = \tan(\theta_2 - \theta_1) = \frac{\tan\theta_2 - \tan\theta_1}{1 + \tan\theta_1 \tan\theta_2}$$

$$\tan\phi = \frac{m_2 - m_1}{1 + m_1 m_2}$$

---

## ⚡ Angle Between Two Lines

> **Angle Formula**: If two lines have slopes $m_1$ and $m_2$, the acute angle $\theta$ between them is:
>
> $$\boxed{\tan\theta = \left|\frac{m_1 - m_2}{1 + m_1 m_2}\right|}$$

Where we take the absolute value to ensure we get the acute angle.

### Alternative Form (for General Equations)

For lines $a_1x + b_1y + c_1 = 0$ and $a_2x + b_2y + c_2 = 0$:

$$\tan\theta = \left|\frac{a_1b_2 - a_2b_1}{a_1a_2 + b_1b_2}\right|$$

---

## 📝 Special Cases

### Case 1: Parallel Lines

Lines are parallel when $m_1 = m_2$:
$$\tan\theta = \frac{m_1 - m_2}{1 + m_1 m_2} = \frac{0}{1 + m_1 m_2} = 0$$
$$\theta = 0°$$

```
        L₁          L₂
        /          /
       /          /     Parallel lines
      /          /      θ = 0°
     /          /
```

### Case 2: Perpendicular Lines

Lines are perpendicular when $1 + m_1 m_2 = 0$, i.e., $m_1 m_2 = -1$:
$$\tan\theta = \frac{m_1 - m_2}{0} = \text{undefined}$$
$$\theta = 90°$$

```
        │   L₂
        │  /
        │ /
    ────┼/───── L₁
        │
        │       Perpendicular lines
                θ = 90°
```

### Case 3: 45° Angle

$$\tan\theta = 1$$
$$\left|\frac{m_1 - m_2}{1 + m_1 m_2}\right| = 1$$
$$|m_1 - m_2| = |1 + m_1 m_2|$$

---

## ✏️ Worked Examples

### Example 1: Basic Angle Calculation

**Problem**: Find the angle between the lines $y = 2x + 3$ and $y = 3x - 1$.

**Solution**:

Slopes: $m_1 = 2$, $m_2 = 3$

$$\tan\theta = \left|\frac{2 - 3}{1 + 2 \cdot 3}\right| = \left|\frac{-1}{7}\right| = \frac{1}{7}$$

$$\theta = \tan^{-1}\left(\frac{1}{7}\right) \approx 8.13°$$

**Answer**: θ ≈ **8.13°**

---

### Example 2: Lines in General Form

**Problem**: Find the acute angle between $3x - 4y + 7 = 0$ and $4x + 3y - 5 = 0$.

**Solution**:

**Method 1: Using slopes**
- Line 1: $m_1 = -\frac{3}{-4} = \frac{3}{4}$
- Line 2: $m_2 = -\frac{4}{3}$

$$\tan\theta = \left|\frac{\frac{3}{4} - (-\frac{4}{3})}{1 + \frac{3}{4} \cdot (-\frac{4}{3})}\right|$$

$$= \left|\frac{\frac{3}{4} + \frac{4}{3}}{1 - 1}\right| = \left|\frac{\frac{25}{12}}{0}\right| = \text{undefined}$$

$$\theta = 90°$$

**Method 2: Using general form formula**
$$\tan\theta = \left|\frac{(3)(3) - (4)(-4)}{(3)(4) + (-4)(3)}\right| = \left|\frac{9 + 16}{12 - 12}\right| = \left|\frac{25}{0}\right| = \text{undefined}$$

**Answer**: θ = **90°** (lines are perpendicular)

---

### Example 3: Finding Line at Given Angle

**Problem**: Find the equation of a line through (2, 1) making an angle of 45° with the line $x - y + 1 = 0$.

**Solution**:

Given line has slope $m_1 = 1$

Let the required line have slope $m$.

$$\tan 45° = \left|\frac{m - 1}{1 + m \cdot 1}\right| = 1$$

$$\left|\frac{m - 1}{1 + m}\right| = 1$$

**Case 1**: $\frac{m - 1}{1 + m} = 1$
$$m - 1 = 1 + m$$
$$-1 = 1$$ (No solution)

**Case 2**: $\frac{m - 1}{1 + m} = -1$
$$m - 1 = -(1 + m)$$
$$m - 1 = -1 - m$$
$$2m = 0$$
$$m = 0$$

So the required line has slope 0 (horizontal).

Using point-slope form through (2, 1):
$$y - 1 = 0(x - 2)$$
$$y = 1$$

**Wait** — Let's reconsider. There should be two lines at 45°.

Actually, Case 1 gives no solution because when $m = -1$, the denominator becomes zero.

Let's check $m = -1$:
The angle between slopes 1 and -1:
$$\tan\theta = \left|\frac{1 - (-1)}{1 + (1)(-1)}\right| = \left|\frac{2}{0}\right| = \text{undefined}$$

This means $m = -1$ gives perpendicular lines (90°), not 45°.

So the only line at 45° to the given line through (2, 1) is **y = 1**.

Actually, there's another approach. The given line $x - y + 1 = 0$ has inclination 45°. A line making 45° with it could have inclination 45° + 45° = 90° (vertical, $x = 2$) or 45° - 45° = 0° (horizontal, $y = 1$).

**Answer**: $y = 1$ (horizontal line) or $x = 2$ (vertical line)

---

### Example 4: Angle in Triangle

**Problem**: Find the interior angles of the triangle formed by lines $y = 0$, $y = x$, and $x + y = 6$.

**Solution**:

**Line 1**: $y = 0$ (X-axis), slope $m_1 = 0$
**Line 2**: $y = x$, slope $m_2 = 1$
**Line 3**: $x + y = 6$, slope $m_3 = -1$

**Angle between Lines 1 and 2:**
$$\tan A = \left|\frac{0 - 1}{1 + 0}\right| = 1$$
$$A = 45°$$

**Angle between Lines 2 and 3:**
$$\tan B = \left|\frac{1 - (-1)}{1 + (1)(-1)}\right| = \left|\frac{2}{0}\right| = \text{undefined}$$
$$B = 90°$$

**Angle between Lines 1 and 3:**
$$\tan C = \left|\frac{0 - (-1)}{1 + 0}\right| = 1$$
$$C = 45°$$

**Verification**: $A + B + C = 45° + 90° + 45° = 180°$ ✓

**Answer**: The triangle has angles **45°, 90°, and 45°**

```
        Y
        ↑
        │      /\
        │     /  \  y = x
        │    / 90°\
        │   /      \
        │  /   45°  \ x + y = 6
        │ /──────────\
    ────O──────45°────●──→ X
                      (6,0)
```

---

### Example 5: Angle Bisector Problem

**Problem**: Find the slopes of lines that make equal angles with the lines having slopes 3 and $\frac{1}{3}$.

**Solution**:

Let the required line have slope $m$.

Angle with line of slope 3:
$$\tan\alpha = \left|\frac{m - 3}{1 + 3m}\right|$$

Angle with line of slope $\frac{1}{3}$:
$$\tan\beta = \left|\frac{m - \frac{1}{3}}{1 + \frac{m}{3}}\right| = \left|\frac{3m - 1}{3 + m}\right|$$

For equal angles: $\tan\alpha = \tan\beta$

$$\left|\frac{m - 3}{1 + 3m}\right| = \left|\frac{3m - 1}{3 + m}\right|$$

**Case 1**: $\frac{m - 3}{1 + 3m} = \frac{3m - 1}{3 + m}$

Cross-multiply:
$(m - 3)(3 + m) = (3m - 1)(1 + 3m)$
$3m + m^2 - 9 - 3m = 3m + 9m^2 - 1 - 3m$
$m^2 - 9 = 9m^2 - 1$
$-8m^2 = 8$
$m^2 = -1$ (No real solution)

**Case 2**: $\frac{m - 3}{1 + 3m} = -\frac{3m - 1}{3 + m}$

$(m - 3)(3 + m) = -(3m - 1)(1 + 3m)$
$m^2 - 9 = -(9m^2 - 1)$
$m^2 - 9 = -9m^2 + 1$
$10m^2 = 10$
$m^2 = 1$
$m = \pm 1$

**Answer**: The required slopes are $m = 1$ and $m = -1$

---

## 📝 Angle Bisectors

The angle bisectors of two lines $a_1x + b_1y + c_1 = 0$ and $a_2x + b_2y + c_2 = 0$ are:

$$\frac{a_1x + b_1y + c_1}{\sqrt{a_1^2 + b_1^2}} = \pm\frac{a_2x + b_2y + c_2}{\sqrt{a_2^2 + b_2^2}}$$

- The **+** sign gives one bisector
- The **−** sign gives the other bisector
- The two bisectors are perpendicular to each other

---

## 💡 Tips and Insights

> 💡 **Acute vs Obtuse**: Taking absolute value gives the acute angle. Without absolute value, you get the actual directed angle.

> 💡 **Parallel Check**: If $m_1 = m_2$, lines are parallel (angle = 0°).

> 💡 **Perpendicular Check**: If $m_1 m_2 = -1$, lines are perpendicular (angle = 90°).

> ⚠️ **Vertical Lines**: Be careful when one line is vertical (undefined slope). Use the inclination directly.

> 💡 **Two Angles**: Any two intersecting lines form two angles that sum to 180°.

---

## 🌍 Real-World Applications

1. **Architecture**: Calculating roof angles and intersections
2. **Engineering**: Structural analysis of trusses
3. **Navigation**: Computing heading changes
4. **Optics**: Angle of incidence and reflection
5. **Surveying**: Calculating turning angles
6. **Computer Graphics**: Rotation and transformation calculations

---

## 📋 Summary Table

| Scenario | Condition | Angle |
|----------|-----------|-------|
| General case | $\tan\theta = \left\|\frac{m_1 - m_2}{1 + m_1 m_2}\right\|$ | $\theta = \tan^{-1}(\ldots)$ |
| Parallel lines | $m_1 = m_2$ | θ = 0° |
| Perpendicular lines | $m_1 m_2 = -1$ | θ = 90° |
| 45° angle | $\|m_1 - m_2\| = \|1 + m_1 m_2\|$ | θ = 45° |
| General form | $\tan\theta = \left\|\frac{a_1b_2 - a_2b_1}{a_1a_2 + b_1b_2}\right\|$ | — |

---

## ❓ Quick Revision Questions

1. **Find the acute angle between lines with slopes 2 and $\frac{1}{2}$.**

2. **Show that the lines $3x + 4y = 5$ and $4x - 3y = 7$ are perpendicular.**

3. **Find the angle between the X-axis and the line $\sqrt{3}x + y = 5$.**

4. **A line makes an angle of 60° with the positive X-axis. What is its slope?**

5. **Find the angles of the triangle formed by $x = 0$, $y = 0$, and $x + y = 1$.**

6. **Find the equation of a line through origin making 45° angle with the line $y = 2x$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $\tan\theta = \left|\frac{2 - \frac{1}{2}}{1 + 2 \cdot \frac{1}{2}}\right| = \left|\frac{\frac{3}{2}}{2}\right| = \frac{3}{4}$
   $\theta = \tan^{-1}\frac{3}{4} \approx 36.87°$

2. $m_1 = -\frac{3}{4}$, $m_2 = \frac{4}{3}$
   $m_1 \cdot m_2 = -\frac{3}{4} \cdot \frac{4}{3} = -1$ ✓
   Lines are perpendicular.

3. Slope = $-\sqrt{3}$, so $\tan\theta = -\sqrt{3}$
   Inclination = 120° (since slope is negative)
   Angle with X-axis = 120° (or acute angle 60°)

4. Slope $m = \tan 60° = \sqrt{3}$

5. Lines: X-axis ($y = 0$, m = 0), Y-axis ($x = 0$, vertical), and $x + y = 1$ (m = -1)
   - At origin: angle between axes = 90°
   - At (1, 0): angle = $\tan^{-1}|(-1-0)/(1+0)| = 45°$
   - At (0, 1): angle = 90° - 45° = 45°
   Triangle angles: **45°, 45°, 90°**

6. Given line has slope 2. Let required slope be m.
   $1 = \left|\frac{m - 2}{1 + 2m}\right|$
   
   Case 1: $m - 2 = 1 + 2m$ → $m = -3$
   Case 2: $m - 2 = -(1 + 2m)$ → $3m = 1$ → $m = \frac{1}{3}$
   
   Lines: $y = -3x$ or $y = \frac{1}{3}x$

</details>

---

## ⏭️ Navigation

| [← Previous: General Equation](05-general-equation.md) | [Next: Parallel & Perpendicular →](07-parallel-perpendicular.md) |
|:------------------------------------------------------|----------------------------------------------------------------:|
