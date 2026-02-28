# Chapter 5.4: Equation of Tangent to Parabola

## 📚 Chapter Overview

A **tangent** to a parabola is a line that touches the curve at exactly one point. This chapter derives tangent equations in multiple forms: at a given point, in parametric form, and in slope form. Mastering these forms is essential for solving parabola problems.

---

## 📝 Tangent at a Point

### For Parabola $y^2 = 4ax$

> **Tangent at $(x_1, y_1)$**:
>
> $$\boxed{yy_1 = 2a(x + x_1)}$$

This is the **T = 0** form, obtained by replacing:
- $y^2$ → $yy_1$
- $x$ → $\frac{x + x_1}{2}$

```
                 │
                 │        
       P(x₁,y₁)●─────────  Tangent line
                \│
                 \
                  \
                   \
        ───────────●V─────────►
                   │
                   
    Tangent touches at P and doesn't cross the curve
```

### Derivation

For parabola $y^2 = 4ax$, differentiate implicitly:
$$2y \frac{dy}{dx} = 4a$$
$$\frac{dy}{dx} = \frac{2a}{y}$$

At $(x_1, y_1)$: slope = $\frac{2a}{y_1}$

Tangent equation:
$$y - y_1 = \frac{2a}{y_1}(x - x_1)$$

$$yy_1 - y_1^2 = 2ax - 2ax_1$$

Since $(x_1, y_1)$ is on parabola: $y_1^2 = 4ax_1$

$$yy_1 - 4ax_1 = 2ax - 2ax_1$$

$$yy_1 = 2ax + 2ax_1 = 2a(x + x_1)$$

---

## ⚡ Tangent Forms for All Standard Parabolas

| Parabola | Tangent at $(x_1, y_1)$ |
|----------|------------------------|
| $y^2 = 4ax$ | $yy_1 = 2a(x + x_1)$ |
| $y^2 = -4ax$ | $yy_1 = -2a(x + x_1)$ |
| $x^2 = 4ay$ | $xx_1 = 2a(y + y_1)$ |
| $x^2 = -4ay$ | $xx_1 = -2a(y + y_1)$ |

---

## 📐 Parametric Form of Tangent

For parabola $y^2 = 4ax$, a point on it can be written as $(at^2, 2at)$.

### Tangent at Parameter t

Substituting $(x_1, y_1) = (at^2, 2at)$ in tangent equation:

$$y(2at) = 2a(x + at^2)$$

$$\boxed{ty = x + at^2}$$

Or equivalently: $x - ty + at^2 = 0$

---

### Properties of Parametric Tangent

1. **Slope of tangent at t**: $\frac{1}{t}$

2. **Y-intercept**: $-at$

3. **X-intercept**: $-at^2$

4. **Tangent meets axis at**: $(-at^2, 0)$

---

## 📝 Slope Form of Tangent

A line with slope $m$ can be written as $y = mx + c$.

For this to be tangent to $y^2 = 4ax$:

Substituting in parabola equation:
$$(mx + c)^2 = 4ax$$
$$m^2x^2 + 2mcx + c^2 = 4ax$$
$$m^2x^2 + (2mc - 4a)x + c^2 = 0$$

For tangency (one point of contact), discriminant = 0:
$$(2mc - 4a)^2 = 4m^2c^2$$
$$4m^2c^2 - 16amc + 16a^2 = 4m^2c^2$$
$$-16amc + 16a^2 = 0$$
$$c = \frac{a}{m}$$

### Tangent with Slope m

$$\boxed{y = mx + \frac{a}{m}}$$

This tangent touches the parabola at point $\left(\frac{a}{m^2}, \frac{2a}{m}\right)$.

---

## 📐 Condition for Tangency

For line $y = mx + c$ to be tangent to $y^2 = 4ax$:

$$\boxed{c = \frac{a}{m}}$$

For general line $lx + my + n = 0$ to be tangent to $y^2 = 4ax$:

$$\boxed{ln = am^2}$$

---

## ✏️ Worked Examples

### Example 1: Tangent at a Point

**Problem**: Find the tangent to $y^2 = 8x$ at $(2, 4)$.

**Solution**:

Here $4a = 8$, so $a = 2$.

Tangent at $(x_1, y_1) = (2, 4)$:
$$yy_1 = 2a(x + x_1)$$
$$4y = 4(x + 2)$$
$$y = x + 2$$

**Answer**: $x - y + 2 = 0$ or $y = x + 2$

---

### Example 2: Tangent at Parameter

**Problem**: Find the tangent to $y^2 = 12x$ at $t = 2$.

**Solution**:

$a = 3$

Point at $t = 2$: $(at^2, 2at) = (12, 12)$

Tangent: $ty = x + at^2$
$$2y = x + 12$$

**Answer**: $x - 2y + 12 = 0$

---

### Example 3: Tangent with Given Slope

**Problem**: Find the tangent to $y^2 = 16x$ with slope 2.

**Solution**:

$a = 4$, $m = 2$

Tangent: $y = mx + \frac{a}{m} = 2x + \frac{4}{2} = 2x + 2$

Point of contact: $\left(\frac{a}{m^2}, \frac{2a}{m}\right) = \left(\frac{4}{4}, \frac{8}{2}\right) = (1, 4)$

**Verification**: $16 = 16 \times 1$ ✓

**Answer**: $y = 2x + 2$ touches at $(1, 4)$

---

### Example 4: Tangent Parallel to a Line

**Problem**: Find tangent to $y^2 = 4x$ parallel to $x - 2y + 5 = 0$.

**Solution**:

Slope of given line = $\frac{1}{2}$

$a = 1$, $m = \frac{1}{2}$

Tangent: $y = \frac{1}{2}x + \frac{1}{1/2} = \frac{1}{2}x + 2$

Or: $x - 2y + 4 = 0$

**Answer**: $x - 2y + 4 = 0$

---

### Example 5: Tangents from External Point

**Problem**: Find the tangents from $(4, 6)$ to $y^2 = 8x$.

**Solution**:

$a = 2$

Let tangent be $y = mx + \frac{2}{m}$

Since it passes through $(4, 6)$:
$$6 = 4m + \frac{2}{m}$$
$$6m = 4m^2 + 2$$
$$4m^2 - 6m + 2 = 0$$
$$2m^2 - 3m + 1 = 0$$
$$(2m - 1)(m - 1) = 0$$
$$m = \frac{1}{2} \text{ or } m = 1$$

Tangents:
- $m = \frac{1}{2}$: $y = \frac{x}{2} + 4$ → $x - 2y + 8 = 0$
- $m = 1$: $y = x + 2$ → $x - y + 2 = 0$

**Answer**: $x - 2y + 8 = 0$ and $x - y + 2 = 0$

---

### Example 6: Common Tangent

**Problem**: Find the common tangent to $y^2 = 4x$ and $x^2 = 32y$.

**Solution**:

Tangent to $y^2 = 4x$: $y = mx + \frac{1}{m}$ ... (1)

For this to be tangent to $x^2 = 32y$:
Substitute (1) in $x^2 = 32y$:
$$x^2 = 32\left(mx + \frac{1}{m}\right)$$
$$x^2 - 32mx - \frac{32}{m} = 0$$

For tangency, discriminant = 0:
$$(32m)^2 + 4 \cdot \frac{32}{m} = 0$$
$$1024m^2 + \frac{128}{m} = 0$$
$$1024m^3 + 128 = 0$$
$$m^3 = -\frac{1}{8}$$
$$m = -\frac{1}{2}$$

Tangent: $y = -\frac{1}{2}x + \frac{1}{-1/2} = -\frac{1}{2}x - 2$

**Answer**: $x + 2y + 4 = 0$

---

## 📝 Pair of Tangents from External Point

From an external point $(x_1, y_1)$, two tangents can be drawn to a parabola.

### Combined Equation

The pair of tangents from $(x_1, y_1)$ to $y^2 = 4ax$ is:

$$\boxed{SS_1 = T^2}$$

where:
- $S = y^2 - 4ax$
- $S_1 = y_1^2 - 4ax_1$
- $T = yy_1 - 2a(x + x_1)$

---

## 📝 Chord of Contact

If tangents from $(x_1, y_1)$ touch the parabola at P and Q, then PQ (chord of contact) has equation:

$$\boxed{T = 0}$$

i.e., $yy_1 = 2a(x + x_1)$

```
              External point
                  (x₁, y₁)
                     ●
                    /│\
                   / │ \
                  /  │  \
                 /   │   \
                /    │    \
           ───●──────┼──────●───
              P      │      Q
                     │
    PQ is the chord of contact
    Equation: yy₁ = 2a(x + x₁)
```

---

## 💡 Tips and Insights

> 💡 **T = 0 Rule**: For tangent at $(x_1, y_1)$, use T = 0 with appropriate substitutions.

> 💡 **Slope Form Memory**: Tangent is $y = mx + \frac{a}{m}$; remember "a over m."

> ⚠️ **Slope ≠ 0**: For horizontal tangent, use the parametric form with $t \to \infty$.

> 💡 **Two Tangents**: From external point, always 2 tangents; from point on curve, 1 tangent; from inside, 0 tangents.

> 💡 **Relationship**: If tangent at $t$ is $ty = x + at^2$, then slope = $\frac{1}{t}$, so $m = \frac{1}{t}$, i.e., $t = \frac{1}{m}$.

---

## 📐 Geometrical Properties

### 1. Tangent at Ends of Focal Chord

If tangents at ends of a focal chord meet, they meet on the directrix.

### 2. Angle of Tangents at Ends of Focal Chord

Tangents at ends of a focal chord are perpendicular.

### 3. Foot of Perpendicular

The foot of perpendicular from focus to any tangent lies on the tangent at vertex.

---

## 📋 Summary Table

| Form | Tangent Equation | Point of Contact |
|------|------------------|------------------|
| At $(x_1, y_1)$ | $yy_1 = 2a(x + x_1)$ | $(x_1, y_1)$ |
| At parameter $t$ | $ty = x + at^2$ | $(at^2, 2at)$ |
| With slope $m$ | $y = mx + \frac{a}{m}$ | $\left(\frac{a}{m^2}, \frac{2a}{m}\right)$ |
| Condition for tangency | $c = \frac{a}{m}$ | - |
| Chord of contact | $yy_1 = 2a(x + x_1)$ | From $(x_1, y_1)$ |

---

## ❓ Quick Revision Questions

1. **Find tangent to $y^2 = 4x$ at $(1, 2)$.**

2. **Find tangent to $y^2 = 20x$ at parameter $t = 3$.**

3. **Find tangent to $y^2 = 12x$ with slope 3.**

4. **Find tangents from $(3, 4)$ to $y^2 = 4x$.**

5. **Find the chord of contact of tangents from $(2, 3)$ to $y^2 = 8x$.**

6. **Show that $y = 2x + 1$ touches $y^2 = 8x$ and find point of contact.**

<details>
<summary><b>Click to see answers</b></summary>

1. $a = 1$
   $yy_1 = 2a(x + x_1)$
   $2y = 2(x + 1)$
   **$y = x + 1$ or $x - y + 1 = 0$**

2. $a = 5$
   Point: $(45, 30)$
   Tangent: $ty = x + at^2$
   $3y = x + 45$
   **$x - 3y + 45 = 0$**

3. $a = 3$, $m = 3$
   $y = 3x + \frac{3}{3} = 3x + 1$
   **$3x - y + 1 = 0$**, point of contact: $\left(\frac{1}{3}, 2\right)$

4. $a = 1$
   Tangent: $y = mx + \frac{1}{m}$
   Through $(3, 4)$: $4 = 3m + \frac{1}{m}$
   $4m = 3m^2 + 1$
   $3m^2 - 4m + 1 = 0$
   $(3m - 1)(m - 1) = 0$
   $m = 1$ or $m = \frac{1}{3}$
   **$y = x + 1$ and $y = \frac{x}{3} + 3$ (or $x - 3y + 9 = 0$)**

5. $a = 2$
   Chord of contact: $yy_1 = 2a(x + x_1)$
   $3y = 4(x + 2)$
   **$4x - 3y + 8 = 0$**

6. $y^2 = 8x$ → $a = 2$
   For $y = 2x + c$ to be tangent: $c = \frac{a}{m} = \frac{2}{2} = 1$ ✓
   So $y = 2x + 1$ is indeed tangent.
   Point of contact: $\left(\frac{a}{m^2}, \frac{2a}{m}\right) = \left(\frac{2}{4}, \frac{4}{2}\right) = \left(\frac{1}{2}, 2\right)$
   **Point of contact: $\left(\frac{1}{2}, 2\right)$**

</details>

---

## ⏭️ Navigation

| [← Previous: Focal Chord](03-focal-chord.md) | [Next: Equation of Normal →](05-equation-of-normal.md) |
|:--------------------------------------------|--------------------------------------------------------:|
