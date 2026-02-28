# Chapter 6.2: Eccentricity and Elements of Ellipse

## 📚 Chapter Overview

**Eccentricity** measures how "stretched" an ellipse is compared to a circle. This chapter explores eccentricity, directrices, latus rectum, and the complete terminology of ellipse elements.

---

## 📝 Eccentricity

### Definition

The **eccentricity** $e$ of an ellipse is defined as:

$$\boxed{e = \frac{c}{a} = \frac{\text{distance from center to focus}}{\text{semi-major axis}}}$$

where $c = \sqrt{a^2 - b^2}$

### Properties

- For ellipse: $\boxed{0 < e < 1}$
- When $e \to 0$: ellipse → circle
- When $e \to 1$: ellipse → very elongated (approaches parabola)

```
     e = 0 (circle)     e = 0.5            e = 0.9
          ○              ⬭                  ⬬
      Nearly           Moderate          Highly
      circular         ellipse           elongated
```

---

## 📐 Relation Between e, a, and b

From $c^2 = a^2 - b^2$ and $e = \frac{c}{a}$:

$$c = ae$$

$$b^2 = a^2 - c^2 = a^2 - a^2e^2 = a^2(1 - e^2)$$

$$\boxed{b^2 = a^2(1 - e^2)}$$

Or equivalently:

$$\boxed{e = \sqrt{1 - \frac{b^2}{a^2}}}$$

---

## 📝 Directrices

### Definition

The **directrices** are fixed lines such that the ratio of distance from any point on ellipse to focus and to directrix equals eccentricity.

$$\frac{PF}{PM} = e$$

### For $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$ (a > b)

$$\boxed{\text{Directrices: } x = \pm \frac{a}{e}}$$

```
    Directrix       Directrix
    x = -a/e        x = a/e
        │              │
        │   ●F'  ●F    │
        │              │
    ────┼──────────────┼────
        │              │
        │              │
        
    Distance from center to directrix = a/e
    Distance from center to focus = ae
    Note: a/e > a > ae (since e < 1)
```

### For $\frac{x^2}{b^2} + \frac{y^2}{a^2} = 1$ (a > b)

$$\boxed{\text{Directrices: } y = \pm \frac{a}{e}}$$

---

## 📝 Verification of Focus-Directrix Property

For point P$(x, y)$ on ellipse $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$:

Distance to focus $F(ae, 0)$:
$$PF = \sqrt{(x - ae)^2 + y^2}$$

From earlier derivation: $PF = a - ex$ (using $d_2$ expression)

Distance to directrix $x = \frac{a}{e}$:
$$PM = \left|\frac{a}{e} - x\right| = \frac{a - ex}{e}$$ (since $x < \frac{a}{e}$ for points on ellipse)

$$\frac{PF}{PM} = \frac{a - ex}{\frac{a - ex}{e}} = e$$ ✓

---

## ⚡ Latus Rectum

### Definition

The **latus rectum** is the chord through a focus perpendicular to the major axis.

### Length of Latus Rectum

For $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$:

At focus $(c, 0)$ or $(ae, 0)$:
$$\frac{(ae)^2}{a^2} + \frac{y^2}{b^2} = 1$$
$$e^2 + \frac{y^2}{b^2} = 1$$
$$y^2 = b^2(1 - e^2) = b^2 \cdot \frac{b^2}{a^2} = \frac{b^4}{a^2}$$
$$y = \pm \frac{b^2}{a}$$

$$\boxed{\text{Length of Latus Rectum} = \frac{2b^2}{a}}$$

### Semi-Latus Rectum

$$\ell = \frac{b^2}{a}$$

---

## 📋 Complete Element Table

For $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$ with $a > b$:

| Element | Symbol | Value/Formula |
|---------|--------|---------------|
| Center | C | $(0, 0)$ |
| Semi-major axis | a | Given |
| Semi-minor axis | b | Given |
| Eccentricity | e | $\frac{\sqrt{a^2 - b^2}}{a}$ |
| Foci | F, F' | $(\pm ae, 0)$ |
| Vertices | A, A' | $(\pm a, 0)$ |
| Co-vertices | B, B' | $(0, \pm b)$ |
| Directrices | | $x = \pm \frac{a}{e}$ |
| Latus rectum | LR | $\frac{2b^2}{a}$ |
| Sum of focal distances | | $2a$ |
| Difference of focal distances | | 0 to $2ae$ |

---

## 📝 Focal Distances

For a point P$(x_1, y_1)$ on the ellipse:

### Focal Distances (Focal Radii)

$$\boxed{PF_1 = a + ex_1}$$
$$\boxed{PF_2 = a - ex_1}$$

where $F_1(-ae, 0)$ and $F_2(ae, 0)$.

**Sum**: $PF_1 + PF_2 = 2a$ (constant)

**Difference**: $|PF_1 - PF_2| = 2|ex_1|$ (depends on position)

---

## ✏️ Worked Examples

### Example 1: Finding Eccentricity

**Problem**: Find eccentricity of $\frac{x^2}{25} + \frac{y^2}{16} = 1$.

**Solution**:

$a^2 = 25$, $b^2 = 16$, so $a = 5$, $b = 4$

$c = \sqrt{25 - 16} = 3$

$e = \frac{c}{a} = \frac{3}{5} = 0.6$

**Answer**: $e = \frac{3}{5}$

---

### Example 2: From Eccentricity to Equation

**Problem**: Find the ellipse with center at origin, major axis along x-axis, $a = 6$, and $e = \frac{2}{3}$.

**Solution**:

$e = \frac{2}{3}$, $a = 6$

$b^2 = a^2(1 - e^2) = 36\left(1 - \frac{4}{9}\right) = 36 \times \frac{5}{9} = 20$

**Answer**: $\frac{x^2}{36} + \frac{y^2}{20} = 1$

---

### Example 3: Finding Directrices

**Problem**: Find the directrices of $\frac{x^2}{49} + \frac{y^2}{24} = 1$.

**Solution**:

$a^2 = 49$, $b^2 = 24$
$a = 7$, $c = \sqrt{49 - 24} = 5$
$e = \frac{5}{7}$

Directrices: $x = \pm \frac{a}{e} = \pm \frac{7}{5/7} = \pm \frac{49}{5}$

**Answer**: $x = \pm \frac{49}{5}$ or $x = \pm 9.8$

---

### Example 4: Latus Rectum

**Problem**: Find the latus rectum of $\frac{x^2}{9} + \frac{y^2}{5} = 1$.

**Solution**:

$a^2 = 9$, $b^2 = 5$
$a = 3$, $b = \sqrt{5}$

Latus rectum = $\frac{2b^2}{a} = \frac{2 \times 5}{3} = \frac{10}{3}$

**Answer**: $\frac{10}{3}$

---

### Example 5: Focal Distance

**Problem**: Find the focal distances of point $(3, \frac{8}{3})$ on ellipse $\frac{x^2}{25} + \frac{y^2}{9} = 1$.

**Solution**:

$a = 5$, $b = 3$, $c = 4$, $e = \frac{4}{5}$

First verify point is on ellipse:
$\frac{9}{25} + \frac{64/9}{9} = \frac{9}{25} + \frac{64}{81}$
$= \frac{729 + 1600}{2025} = \frac{2329}{2025}$... 

Let me recalculate: $\frac{9}{25} + \frac{64}{81}$
$= \frac{729 + 1600}{2025} \neq 1$

The point may not be exactly on ellipse. Let me adjust the problem.

**Alternative**: For point $(3, \frac{12}{5})$ on $\frac{x^2}{25} + \frac{y^2}{9} = 1$:

$\frac{9}{25} + \frac{144/25}{9} = \frac{9}{25} + \frac{144}{225} = \frac{81}{225} + \frac{144}{225} = \frac{225}{225} = 1$ ✓

$x_1 = 3$

$PF_1 = a + ex_1 = 5 + \frac{4}{5}(3) = 5 + \frac{12}{5} = \frac{37}{5}$

$PF_2 = a - ex_1 = 5 - \frac{12}{5} = \frac{13}{5}$

**Answer**: $PF_1 = \frac{37}{5}$, $PF_2 = \frac{13}{5}$

---

### Example 6: Ellipse from Foci and Directrix

**Problem**: Find ellipse with focus $(3, 0)$ and directrix $x = \frac{25}{3}$.

**Solution**:

Focus at $(ae, 0) = (3, 0)$, so $ae = 3$ ... (1)

Directrix at $x = \frac{a}{e} = \frac{25}{3}$ ... (2)

Multiplying (1) and (2): $ae \cdot \frac{a}{e} = 3 \times \frac{25}{3}$
$a^2 = 25$, so $a = 5$

From (1): $e = \frac{3}{5}$

$b^2 = a^2(1 - e^2) = 25(1 - \frac{9}{25}) = 16$

**Answer**: $\frac{x^2}{25} + \frac{y^2}{16} = 1$

---

## 📐 Special Values

### End Points of Latus Rectum

For $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$:

At right focus $(ae, 0)$:
- Endpoints: $\left(ae, \pm\frac{b^2}{a}\right)$

At left focus $(-ae, 0)$:
- Endpoints: $\left(-ae, \pm\frac{b^2}{a}\right)$

---

## 💡 Tips and Insights

> 💡 **Memory Aid**: $e = \frac{c}{a}$, where c = focus distance, a = vertex distance.

> 💡 **Directrix > a**: Distance to directrix ($\frac{a}{e}$) > $a$ since $e < 1$.

> ⚠️ **Common Error**: Don't confuse $\frac{a}{e}$ (directrix) with $ae$ (focus).

> 💡 **Product**: Focus distance × directrix distance = $ae \cdot \frac{a}{e} = a^2$.

> 💡 **LR Formula**: Latus rectum = $\frac{2b^2}{a}$, similar to parabola formula $\frac{2 \times \text{(minor)}^2}{\text{major}}$.

---

## 📋 Summary Formulas

| Quantity | Formula |
|----------|---------|
| Eccentricity | $e = \frac{c}{a} = \sqrt{1 - \frac{b^2}{a^2}}$ |
| Semi-minor axis | $b = a\sqrt{1 - e^2}$ |
| Focus distance | $c = ae = \sqrt{a^2 - b^2}$ |
| Directrix distance | $\frac{a}{e} = \frac{a^2}{c}$ |
| Latus rectum | $\frac{2b^2}{a}$ |
| Focal distance | $a \pm ex$ |

---

## ❓ Quick Revision Questions

1. **Find eccentricity of $\frac{x^2}{16} + \frac{y^2}{9} = 1$.**

2. **If $e = \frac{1}{2}$ and $b = 3$, find $a$ for an ellipse.**

3. **Find the directrices of ellipse with $a = 5$, $e = \frac{3}{5}$, major axis along x-axis.**

4. **Find latus rectum of $\frac{x^2}{100} + \frac{y^2}{64} = 1$.**

5. **For ellipse $9x^2 + 25y^2 = 225$, find eccentricity.**

6. **Find the ends of latus rectum for $\frac{x^2}{25} + \frac{y^2}{16} = 1$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $a^2 = 16$, $b^2 = 9$
   $c = \sqrt{16-9} = \sqrt{7}$
   **$e = \frac{\sqrt{7}}{4}$**

2. $b^2 = a^2(1 - e^2)$
   $9 = a^2(1 - \frac{1}{4}) = \frac{3a^2}{4}$
   $a^2 = 12$
   **$a = 2\sqrt{3}$**

3. $\frac{a}{e} = \frac{5}{3/5} = \frac{25}{3}$
   **Directrices: $x = \pm\frac{25}{3}$**

4. $a^2 = 100$, $b^2 = 64$, $a = 10$
   LR = $\frac{2 \times 64}{10} = \frac{128}{10} = \frac{64}{5}$
   **Latus rectum = $\frac{64}{5} = 12.8$**

5. $\frac{x^2}{25} + \frac{y^2}{9} = 1$
   $a^2 = 25$, $b^2 = 9$, $a = 5$
   $c = \sqrt{25-9} = 4$
   **$e = \frac{4}{5}$**

6. $a = 5$, $b = 4$, $c = 3$, $e = \frac{3}{5}$
   $\frac{b^2}{a} = \frac{16}{5}$
   At right focus $(3, 0)$: $\left(3, \pm\frac{16}{5}\right)$
   At left focus $(-3, 0)$: $\left(-3, \pm\frac{16}{5}\right)$
   **Ends: $\left(\pm 3, \pm\frac{16}{5}\right)$** (4 points total)

</details>

---

## ⏭️ Navigation

| [← Previous: Standard Equation](01-standard-equation.md) | [Next: Parametric Form →](03-parametric-form.md) |
|:---------------------------------------------------------|------------------------------------------------:|
