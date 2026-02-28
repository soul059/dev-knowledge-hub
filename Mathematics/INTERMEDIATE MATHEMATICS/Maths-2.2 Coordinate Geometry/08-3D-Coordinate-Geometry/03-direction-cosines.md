# Chapter 8.3: Direction Cosines and Direction Ratios

## 📚 Chapter Overview

**Direction cosines** and **direction ratios** are fundamental concepts for describing the direction of lines in 3D space. This chapter introduces these concepts, derives their properties, and shows how to find angles between lines.

---

## 📝 Direction Cosines

### Definition

The **direction cosines** of a line are the cosines of the angles that the line makes with the positive directions of the x, y, and z axes.

If a line makes angles $\alpha$, $\beta$, $\gamma$ with the positive x, y, z axes respectively, then:

$$l = \cos\alpha, \quad m = \cos\beta, \quad n = \cos\gamma$$

are called the **direction cosines (DCs)** of the line.

```
                z
                |     /Line
                |    /
                |   / γ (angle with z-axis)
                |  /
                | /
                |/_________ y
               /\ β
              /  \
             / α  \
            x     (angle with y-axis)
            
   (angle with x-axis)
```

### Fundamental Relation

$$\boxed{l^2 + m^2 + n^2 = 1}$$

or equivalently:

$$\boxed{\cos^2\alpha + \cos^2\beta + \cos^2\gamma = 1}$$

### Derivation

Consider a line through origin with direction making angles $\alpha$, $\beta$, $\gamma$ with axes.

Take a point $P(x, y, z)$ on this line at distance $r$ from origin.

Then:
- $x = r\cos\alpha$
- $y = r\cos\beta$  
- $z = r\cos\gamma$

Since $r^2 = x^2 + y^2 + z^2$:

$r^2 = r^2\cos^2\alpha + r^2\cos^2\beta + r^2\cos^2\gamma$

$1 = \cos^2\alpha + \cos^2\beta + \cos^2\gamma$

$1 = l^2 + m^2 + n^2$

---

## 📝 Direction Cosines of Line Joining Two Points

For line joining $A(x_1, y_1, z_1)$ and $B(x_2, y_2, z_2)$:

Let $d = AB = \sqrt{(x_2-x_1)^2 + (y_2-y_1)^2 + (z_2-z_1)^2}$

Direction cosines from A to B:

$$\boxed{l = \frac{x_2-x_1}{d}, \quad m = \frac{y_2-y_1}{d}, \quad n = \frac{z_2-z_1}{d}}$$

---

## 📝 Direction Ratios

### Definition

**Direction ratios (DRs)** are any three numbers proportional to the direction cosines.

If $(l, m, n)$ are DCs, then $(a, b, c)$ are DRs if:

$$\frac{l}{a} = \frac{m}{b} = \frac{n}{c}$$

### Key Properties

1. DCs are **unique** (up to sign), DRs are **not unique**
2. If $(a, b, c)$ are DRs, so are $(ka, kb, kc)$ for any $k \neq 0$
3. $(x_2-x_1, y_2-y_1, z_2-z_1)$ are DRs of line AB

### Converting DRs to DCs

If $(a, b, c)$ are direction ratios:

$$\boxed{l = \frac{a}{\sqrt{a^2+b^2+c^2}}, \quad m = \frac{b}{\sqrt{a^2+b^2+c^2}}, \quad n = \frac{c}{\sqrt{a^2+b^2+c^2}}}$$

Or with $\pm$ to account for opposite directions:

$$l = \pm\frac{a}{\sqrt{a^2+b^2+c^2}}, \quad m = \pm\frac{b}{\sqrt{a^2+b^2+c^2}}, \quad n = \pm\frac{c}{\sqrt{a^2+b^2+c^2}}$$

---

## ⚡ DCs of Coordinate Axes

| Axis | Direction Cosines $(l, m, n)$ |
|------|------------------------------|
| x-axis | $(1, 0, 0)$ |
| y-axis | $(0, 1, 0)$ |
| z-axis | $(0, 0, 1)$ |

---

## 📝 Angle Between Two Lines

### Using Direction Cosines

If two lines have DCs $(l_1, m_1, n_1)$ and $(l_2, m_2, n_2)$:

$$\boxed{\cos\theta = l_1l_2 + m_1m_2 + n_1n_2}$$

### Using Direction Ratios

If two lines have DRs $(a_1, b_1, c_1)$ and $(a_2, b_2, c_2)$:

$$\boxed{\cos\theta = \frac{a_1a_2 + b_1b_2 + c_1c_2}{\sqrt{a_1^2+b_1^2+c_1^2} \cdot \sqrt{a_2^2+b_2^2+c_2^2}}}$$

---

## 📝 Conditions for Parallel and Perpendicular

### Parallel Lines

Lines are parallel if their DCs (or DRs) are proportional:

$$\boxed{\frac{a_1}{a_2} = \frac{b_1}{b_2} = \frac{c_1}{c_2}}$$

Or: $(l_1, m_1, n_1) = \pm(l_2, m_2, n_2)$

### Perpendicular Lines

Lines are perpendicular if:

$$\boxed{l_1l_2 + m_1m_2 + n_1n_2 = 0}$$

Or with DRs:

$$\boxed{a_1a_2 + b_1b_2 + c_1c_2 = 0}$$

---

## ✏️ Worked Examples

### Example 1: DCs from Angles

**Problem**: A line makes angles 60°, 45°, and θ with x, y, z axes. Find θ.

**Solution**:

$l = \cos 60° = \frac{1}{2}$

$m = \cos 45° = \frac{1}{\sqrt{2}}$

$n = \cos\theta$

Using $l^2 + m^2 + n^2 = 1$:

$\frac{1}{4} + \frac{1}{2} + \cos^2\theta = 1$

$\cos^2\theta = 1 - \frac{3}{4} = \frac{1}{4}$

$\cos\theta = \pm\frac{1}{2}$

$\theta = 60°$ or $\theta = 120°$

**Answer**: θ = 60° or 120°

---

### Example 2: DRs to DCs

**Problem**: Find DCs if DRs are $(2, 3, 6)$.

**Solution**:

$\sqrt{a^2+b^2+c^2} = \sqrt{4+9+36} = \sqrt{49} = 7$

$l = \frac{2}{7}$, $m = \frac{3}{7}$, $n = \frac{6}{7}$

**Answer**: $\left(\frac{2}{7}, \frac{3}{7}, \frac{6}{7}\right)$

---

### Example 3: DCs of Line Through Two Points

**Problem**: Find DCs of line joining $(1, 2, 3)$ and $(4, 6, 3)$.

**Solution**:

DRs: $(4-1, 6-2, 3-3) = (3, 4, 0)$

$d = \sqrt{9+16+0} = 5$

DCs: $\left(\frac{3}{5}, \frac{4}{5}, 0\right)$

**Answer**: $\left(\frac{3}{5}, \frac{4}{5}, 0\right)$

---

### Example 4: Angle Between Lines

**Problem**: Find angle between lines with DRs $(1, 2, 2)$ and $(2, 2, 1)$.

**Solution**:

$\cos\theta = \frac{1(2)+2(2)+2(1)}{\sqrt{1+4+4} \cdot \sqrt{4+4+1}}$

$= \frac{2+4+2}{\sqrt{9} \cdot \sqrt{9}} = \frac{8}{9}$

$\theta = \cos^{-1}\left(\frac{8}{9}\right)$

**Answer**: $\theta = \cos^{-1}\left(\frac{8}{9}\right) \approx 27.3°$

---

### Example 5: Parallel Lines Check

**Problem**: Are lines with DRs $(2, 3, 4)$ and $(4, 6, 8)$ parallel?

**Solution**:

Check proportionality:
$\frac{2}{4} = \frac{1}{2}$
$\frac{3}{6} = \frac{1}{2}$
$\frac{4}{8} = \frac{1}{2}$

All ratios equal!

**Answer**: **Yes, lines are parallel**

---

### Example 6: Perpendicular Lines Check

**Problem**: Check if lines with DRs $(1, -2, 1)$ and $(2, 1, 0)$ are perpendicular.

**Solution**:

$a_1a_2 + b_1b_2 + c_1c_2 = 1(2) + (-2)(1) + 1(0)$
$= 2 - 2 + 0 = 0$

**Answer**: **Yes, lines are perpendicular**

---

### Example 7: Equal Angles with Axes

**Problem**: Find DCs of a line equally inclined to all axes.

**Solution**:

Let $\alpha = \beta = \gamma$

Then $l = m = n = \cos\alpha$

Using $l^2 + m^2 + n^2 = 1$:

$3\cos^2\alpha = 1$

$\cos\alpha = \pm\frac{1}{\sqrt{3}}$

DCs: $\left(\pm\frac{1}{\sqrt{3}}, \pm\frac{1}{\sqrt{3}}, \pm\frac{1}{\sqrt{3}}\right)$

**Answer**: $\left(\pm\frac{1}{\sqrt{3}}, \pm\frac{1}{\sqrt{3}}, \pm\frac{1}{\sqrt{3}}\right)$

(4 possible directions based on sign combinations where all signs are same)

---

### Example 8: Find Missing DR

**Problem**: If lines with DRs $(3, -2, k)$ and $(2, 1, -3)$ are perpendicular, find k.

**Solution**:

For perpendicular lines:
$a_1a_2 + b_1b_2 + c_1c_2 = 0$

$3(2) + (-2)(1) + k(-3) = 0$

$6 - 2 - 3k = 0$

$k = \frac{4}{3}$

**Answer**: $k = \frac{4}{3}$

---

## 📐 Projection of Line Segment

The **projection** of line segment AB on a line with DCs $(l, m, n)$:

$$\boxed{\text{Projection} = l(x_2-x_1) + m(y_2-y_1) + n(z_2-z_1)}$$

```
    A ●────────────● B
       \          /
        \        /
         \      /
          ●────●  Projection on line
```

---

## 📊 Summary Table

| Concept | Formula |
|---------|---------|
| Direction cosines | $l = \cos\alpha$, $m = \cos\beta$, $n = \cos\gamma$ |
| Fundamental relation | $l^2 + m^2 + n^2 = 1$ |
| DCs from DRs $(a,b,c)$ | $l = \frac{a}{\sqrt{a^2+b^2+c^2}}$, etc. |
| DRs of line AB | $(x_2-x_1, y_2-y_1, z_2-z_1)$ |
| Angle between lines | $\cos\theta = l_1l_2 + m_1m_2 + n_1n_2$ |
| Parallel condition | $\frac{a_1}{a_2} = \frac{b_1}{b_2} = \frac{c_1}{c_2}$ |
| Perpendicular condition | $a_1a_2 + b_1b_2 + c_1c_2 = 0$ |

---

## 💡 Tips and Insights

> 💡 **DCs are normalized DRs**: DCs have magnitude 1; DRs can have any magnitude.

> 💡 **Sign matters**: DCs can be positive or negative depending on direction.

> ⚠️ **Uniqueness**: A line has two sets of DCs (opposite signs for opposite directions), but infinitely many DRs.

> 💡 **Quick check**: If $l^2 + m^2 + n^2 \neq 1$, you have DRs, not DCs!

---

## 📋 Direction Cosines of Axes and Bisectors

| Line | DCs |
|------|-----|
| x-axis | $(1, 0, 0)$ |
| y-axis | $(0, 1, 0)$ |
| z-axis | $(0, 0, 1)$ |
| Equal inclination | $\left(\pm\frac{1}{\sqrt{3}}, \pm\frac{1}{\sqrt{3}}, \pm\frac{1}{\sqrt{3}}\right)$ |
| In xy-plane, 45° to axes | $\left(\frac{1}{\sqrt{2}}, \frac{1}{\sqrt{2}}, 0\right)$ |

---

## ❓ Quick Revision Questions

1. **If a line makes equal angles with all axes, find its direction cosines.**

2. **Find DCs of line with DRs $(1, 2, 2)$.**

3. **Find angle between lines with DCs $\left(\frac{1}{2}, \frac{1}{2}, \frac{1}{\sqrt{2}}\right)$ and $\left(\frac{1}{\sqrt{2}}, \frac{1}{\sqrt{2}}, 0\right)$.**

4. **If DRs are $(3, 4, k)$ and line is perpendicular to line with DRs $(2, -1, 1)$, find k.**

5. **Find DCs of line joining $(1, 1, 1)$ and $(3, 5, 7)$.**

6. **A line makes angles 45° and 60° with x and y axes. Find angle with z-axis.**

<details>
<summary><b>Click to see answers</b></summary>

1. Equal angles means $\cos\alpha = \cos\beta = \cos\gamma$
   $3\cos^2\alpha = 1$
   **$\left(\pm\frac{1}{\sqrt{3}}, \pm\frac{1}{\sqrt{3}}, \pm\frac{1}{\sqrt{3}}\right)$**

2. $\sqrt{1+4+4} = 3$
   **$\left(\frac{1}{3}, \frac{2}{3}, \frac{2}{3}\right)$**

3. $\cos\theta = \frac{1}{2} \cdot \frac{1}{\sqrt{2}} + \frac{1}{2} \cdot \frac{1}{\sqrt{2}} + \frac{1}{\sqrt{2}} \cdot 0$
   $= \frac{1}{2\sqrt{2}} + \frac{1}{2\sqrt{2}} = \frac{1}{\sqrt{2}}$
   **$\theta = 45°$**

4. $3(2) + 4(-1) + k(1) = 0$
   $6 - 4 + k = 0$
   **$k = -2$**

5. DRs: $(2, 4, 6)$
   $d = \sqrt{4+16+36} = \sqrt{56} = 2\sqrt{14}$
   **$\left(\frac{1}{\sqrt{14}}, \frac{2}{\sqrt{14}}, \frac{3}{\sqrt{14}}\right)$**

6. $\cos^2 45° + \cos^2 60° + \cos^2\gamma = 1$
   $\frac{1}{2} + \frac{1}{4} + \cos^2\gamma = 1$
   $\cos^2\gamma = \frac{1}{4}$
   $\gamma = 60°$ or $120°$
   **$\gamma = 60°$ or $120°$**

</details>

---

## ⏭️ Navigation

| [← Previous: Section Formula](02-section-formula.md) | [Unit 8 Home](README.md) | [Next: Straight Line in 3D →](04-straight-line.md) |
|:---------------------------------------------------:|:------------------------:|--------------------------------------------------:|
