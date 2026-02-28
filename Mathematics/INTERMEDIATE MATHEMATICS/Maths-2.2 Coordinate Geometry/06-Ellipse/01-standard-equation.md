# Chapter 6.1: Standard Equation of Ellipse

## 📚 Chapter Overview

The **ellipse** is defined as the locus of points where the sum of distances from two fixed points (foci) is constant. This chapter derives the standard equations and explores both horizontal and vertical orientations.

---

## 📝 Definition of Ellipse

> An **ellipse** is the locus of a point P that moves such that the sum of its distances from two fixed points $F_1$ and $F_2$ (called **foci**) is constant.

$$\boxed{PF_1 + PF_2 = 2a}$$

where $2a$ is the length of the major axis.

```
                    P(x, y)
                      ●
                     /│\
                    / │ \
                   /  │  \
                  /   │   \
                 /    │    \
           ────●──────┼──────●────
              F₁(-c,0)│    F₂(c,0)
                      │
                      
    PF₁ + PF₂ = 2a (constant)
```

---

## 📐 Alternative Definition (Focus-Directrix)

An ellipse can also be defined as the locus of a point whose distance from a fixed point (focus) is in constant ratio (eccentricity $e$, where $0 < e < 1$) to its distance from a fixed line (directrix).

$$\frac{PF}{PM} = e < 1$$

---

## ⚡ Derivation: Standard Equation

### Setup

Let:
- Foci be at $F_1(-c, 0)$ and $F_2(c, 0)$
- Sum of distances = $2a$
- $P(x, y)$ be any point on ellipse

### Derivation

$$PF_1 + PF_2 = 2a$$

$$\sqrt{(x+c)^2 + y^2} + \sqrt{(x-c)^2 + y^2} = 2a$$

Let $\sqrt{(x+c)^2 + y^2} = d_1$ and $\sqrt{(x-c)^2 + y^2} = d_2$

$d_1 + d_2 = 2a$ ... (1)

Also: $d_1^2 - d_2^2 = (x+c)^2 - (x-c)^2 = 4cx$

$(d_1 + d_2)(d_1 - d_2) = 4cx$

$2a(d_1 - d_2) = 4cx$

$d_1 - d_2 = \frac{2cx}{a}$ ... (2)

From (1) and (2):
$$d_1 = a + \frac{cx}{a}, \quad d_2 = a - \frac{cx}{a}$$

Using $d_2^2 = (x-c)^2 + y^2$:
$$\left(a - \frac{cx}{a}\right)^2 = (x-c)^2 + y^2$$

$$a^2 - 2cx + \frac{c^2x^2}{a^2} = x^2 - 2cx + c^2 + y^2$$

$$a^2 + \frac{c^2x^2}{a^2} = x^2 + c^2 + y^2$$

$$a^2 - c^2 = x^2 - \frac{c^2x^2}{a^2} + y^2$$

$$a^2 - c^2 = x^2\left(1 - \frac{c^2}{a^2}\right) + y^2$$

$$a^2 - c^2 = \frac{x^2(a^2 - c^2)}{a^2} + y^2$$

Let $b^2 = a^2 - c^2$ (where $b > 0$):

$$b^2 = \frac{x^2 b^2}{a^2} + y^2$$

$$\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$$

$$\boxed{\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1}$$

---

## 📝 The Two Standard Forms

### Form 1: Major Axis along X-axis ($a > b$)

$$\boxed{\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1} \quad (a > b)$$

```
                    B(0, b)
                      ●
                    ╱   ╲
                  ╱       ╲
                ╱           ╲
    ──────────●───────●───────●──────────
           A'(-a,0)  O(0,0)  A(a,0)
                ╲           ╱
                  ╲       ╱
                    ╲   ╱
                      ●
                   B'(0,-b)
                   
    Major axis: AA' along x-axis (length = 2a)
    Minor axis: BB' along y-axis (length = 2b)
```

| Element | Value |
|---------|-------|
| Center | $(0, 0)$ |
| Vertices | $(\pm a, 0)$ |
| Co-vertices | $(0, \pm b)$ |
| Foci | $(\pm c, 0)$ where $c = \sqrt{a^2 - b^2}$ |
| Major axis | Along x-axis, length = $2a$ |
| Minor axis | Along y-axis, length = $2b$ |

---

### Form 2: Major Axis along Y-axis ($a > b$)

$$\boxed{\frac{x^2}{b^2} + \frac{y^2}{a^2} = 1} \quad (a > b)$$

```
                   A(0, a)
                      ●
                    ╱   ╲
                  ╱       ╲
    ────────────●───────────●────────────
             B'(-b,0)     B(b,0)
                  ╲       ╱
                    ╲   ╱
                      ●
                  A'(0,-a)
                   
    Major axis: AA' along y-axis (length = 2a)
    Minor axis: BB' along x-axis (length = 2b)
```

| Element | Value |
|---------|-------|
| Center | $(0, 0)$ |
| Vertices | $(0, \pm a)$ |
| Co-vertices | $(\pm b, 0)$ |
| Foci | $(0, \pm c)$ where $c = \sqrt{a^2 - b^2}$ |
| Major axis | Along y-axis, length = $2a$ |
| Minor axis | Along x-axis, length = $2b$ |

---

## 📝 Key Relationship

$$\boxed{b^2 = a^2 - c^2}$$

Or equivalently:

$$\boxed{c^2 = a^2 - b^2}$$

where:
- $a$ = semi-major axis
- $b$ = semi-minor axis
- $c$ = distance from center to focus

> **Note**: Since $c^2 = a^2 - b^2 > 0$, we must have $a > b$ always.

---

## 📐 Identifying the Form

### Quick Rule

1. Write equation in form $\frac{x^2}{p^2} + \frac{y^2}{q^2} = 1$
2. **Larger denominator** corresponds to **major axis**
3. If $p > q$: major axis along x-axis
4. If $q > p$: major axis along y-axis

---

## ✏️ Worked Examples

### Example 1: Identifying Elements

**Problem**: Find the center, vertices, foci for $\frac{x^2}{25} + \frac{y^2}{9} = 1$.

**Solution**:

$a^2 = 25$, $b^2 = 9$ (since $25 > 9$)
$a = 5$, $b = 3$
$c = \sqrt{25 - 9} = \sqrt{16} = 4$

Major axis along x-axis.

| Element | Value |
|---------|-------|
| Center | $(0, 0)$ |
| Vertices | $(\pm 5, 0)$ |
| Co-vertices | $(0, \pm 3)$ |
| Foci | $(\pm 4, 0)$ |

---

### Example 2: Major Axis along Y-axis

**Problem**: Find elements for $\frac{x^2}{16} + \frac{y^2}{25} = 1$.

**Solution**:

Here $25 > 16$, so $a^2 = 25$, $b^2 = 16$
$a = 5$, $b = 4$
$c = \sqrt{25 - 16} = 3$

Major axis along y-axis.

| Element | Value |
|---------|-------|
| Center | $(0, 0)$ |
| Vertices | $(0, \pm 5)$ |
| Co-vertices | $(\pm 4, 0)$ |
| Foci | $(0, \pm 3)$ |

---

### Example 3: From Foci to Equation

**Problem**: Find ellipse equation with foci $(\pm 3, 0)$ and sum of focal distances = 10.

**Solution**:

Foci at $(\pm 3, 0)$: $c = 3$, major axis along x-axis

Sum of focal distances = $2a = 10$, so $a = 5$

$b^2 = a^2 - c^2 = 25 - 9 = 16$

**Answer**: $\frac{x^2}{25} + \frac{y^2}{16} = 1$

---

### Example 4: From Vertices to Equation

**Problem**: Find ellipse with vertices $(0, \pm 6)$ and co-vertices $(\pm 4, 0)$.

**Solution**:

Vertices at $(0, \pm 6)$: $a = 6$, major axis along y-axis
Co-vertices at $(\pm 4, 0)$: $b = 4$

**Answer**: $\frac{x^2}{16} + \frac{y^2}{36} = 1$

---

### Example 5: Point on Ellipse

**Problem**: If $(3, 2)$ lies on $\frac{x^2}{a^2} + \frac{y^2}{5} = 1$, find $a$.

**Solution**:

Substituting $(3, 2)$:
$$\frac{9}{a^2} + \frac{4}{5} = 1$$
$$\frac{9}{a^2} = \frac{1}{5}$$
$$a^2 = 45$$

**Answer**: $a^2 = 45$ (so $a = 3\sqrt{5}$)

---

### Example 6: Shifted Ellipse

**Problem**: Find center, vertices, foci for $\frac{(x-2)^2}{9} + \frac{(y+1)^2}{25} = 1$.

**Solution**:

Center: $(2, -1)$

$a^2 = 25$, $b^2 = 9$ (since $25 > 9$, major axis is vertical)
$a = 5$, $b = 3$, $c = \sqrt{25-9} = 4$

| Element | Value |
|---------|-------|
| Center | $(2, -1)$ |
| Vertices | $(2, -1 \pm 5) = (2, 4)$ and $(2, -6)$ |
| Co-vertices | $(2 \pm 3, -1) = (5, -1)$ and $(-1, -1)$ |
| Foci | $(2, -1 \pm 4) = (2, 3)$ and $(2, -5)$ |

---

## 💡 Tips and Insights

> 💡 **Larger Denominator = Major Axis**: Always identify which denominator is larger to find major axis direction.

> 💡 **a > b Always**: By convention, $a$ is always the semi-major axis.

> ⚠️ **c < a**: Since $c^2 = a^2 - b^2$, foci are always inside the ellipse.

> 💡 **Circle Special Case**: When $a = b$, ellipse becomes a circle with radius $a$.

> 💡 **Sum = 2a**: For any point on ellipse, sum of focal distances equals $2a$.

---

## 📋 Summary Table

| Property | Horizontal Major Axis | Vertical Major Axis |
|----------|----------------------|---------------------|
| Equation | $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$ | $\frac{x^2}{b^2} + \frac{y^2}{a^2} = 1$ |
| Condition | $a > b$ | $a > b$ |
| Vertices | $(\pm a, 0)$ | $(0, \pm a)$ |
| Co-vertices | $(0, \pm b)$ | $(\pm b, 0)$ |
| Foci | $(\pm c, 0)$ | $(0, \pm c)$ |
| Relation | $c^2 = a^2 - b^2$ | $c^2 = a^2 - b^2$ |

---

## ❓ Quick Revision Questions

1. **Find vertices and foci for $\frac{x^2}{36} + \frac{y^2}{20} = 1$.**

2. **Find the equation of ellipse with foci $(0, \pm 4)$ and vertices $(0, \pm 5)$.**

3. **For $\frac{x^2}{49} + \frac{y^2}{64} = 1$, which axis is the major axis?**

4. **If $c = 5$ and $b = 12$, find $a$ and write the ellipse equation (major axis along x).**

5. **Find center and semi-axes for $\frac{(x+3)^2}{4} + \frac{(y-2)^2}{9} = 1$.**

6. **An ellipse passes through $(2, 3)$ and $(4, 0)$. Find its equation.**

<details>
<summary><b>Click to see answers</b></summary>

1. $a^2 = 36$, $b^2 = 20$
   $a = 6$, $b = 2\sqrt{5}$, $c = \sqrt{36-20} = 4$
   **Vertices: $(\pm 6, 0)$, Foci: $(\pm 4, 0)$**

2. Foci at $(0, \pm 4)$: $c = 4$, vertical major axis
   Vertices at $(0, \pm 5)$: $a = 5$
   $b^2 = 25 - 16 = 9$
   **$\frac{x^2}{9} + \frac{y^2}{25} = 1$**

3. $64 > 49$, so $a^2 = 64$ (under $y^2$)
   **Major axis along y-axis**

4. $a^2 = b^2 + c^2 = 144 + 25 = 169$
   $a = 13$
   **$\frac{x^2}{169} + \frac{y^2}{144} = 1$**

5. Center: $(-3, 2)$
   $a^2 = 9$ (under $y$), $b^2 = 4$ (under $x$)
   **Semi-major axis = 3 (along y), semi-minor = 2 (along x)**

6. Through $(4, 0)$: $\frac{16}{a^2} = 1$, so $a^2 = 16$
   Through $(2, 3)$: $\frac{4}{16} + \frac{9}{b^2} = 1$
   $\frac{9}{b^2} = \frac{3}{4}$, $b^2 = 12$
   **$\frac{x^2}{16} + \frac{y^2}{12} = 1$**

</details>

---

## ⏭️ Navigation

| [← Unit 6 Home](README.md) | [Next: Eccentricity and Elements →](02-eccentricity-elements.md) |
|:--------------------------|----------------------------------------------------------------:|
