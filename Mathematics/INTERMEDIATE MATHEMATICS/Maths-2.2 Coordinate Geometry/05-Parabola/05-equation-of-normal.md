# Chapter 5.5: Equation of Normal to Parabola

## 📚 Chapter Overview

The **normal** to a parabola at any point is the line perpendicular to the tangent at that point. Unlike circles, normals to a parabola don't pass through a fixed point. This chapter explores normal equations, their properties, and the concept of co-normal points.

---

## 📝 Normal at a Point

### For Parabola $y^2 = 4ax$

At point $(x_1, y_1)$, slope of tangent = $\frac{2a}{y_1}$

Slope of normal = $-\frac{y_1}{2a}$ (perpendicular)

> **Normal at $(x_1, y_1)$**:
>
> $$\boxed{y - y_1 = -\frac{y_1}{2a}(x - x_1)}$$

Or in standard form:
$$\boxed{y = -\frac{y_1}{2a}x + y_1 + \frac{y_1 x_1}{2a}}$$

```
                 │
           Normal│\
                 │ \     Tangent
                 │  ●P(x₁,y₁)────────
                 │ /
                 │/
        ─────────●V──────────────►
                 │
                 
    Normal ⊥ Tangent at point P
```

---

## ⚡ Parametric Form of Normal

For point at parameter $t$: $(at^2, 2at)$

### Normal at Parameter t

$$\boxed{y + tx = 2at + at^3}$$

Or: $tx + y = at(2 + t^2)$

### Derivation

At $(at^2, 2at)$:
- Slope of tangent = $\frac{1}{t}$
- Slope of normal = $-t$

Normal equation:
$$y - 2at = -t(x - at^2)$$
$$y - 2at = -tx + at^3$$
$$y + tx = 2at + at^3$$

---

## 📐 Slope Form of Normal

If normal has slope $m$:

Since slope of tangent = $\frac{1}{t}$ and slope of normal = $-t$:

$m = -t$, so $t = -m$

Substituting in parametric normal:
$$y + (-m)x = a(-m)(2 + m^2)$$
$$y - mx = -2am - am^3$$

$$\boxed{y = mx - 2am - am^3}$$

The normal with slope $m$ touches the parabola at $(am^2, -2am)$.

---

## 📝 Normal Forms for All Standard Parabolas

| Parabola | Normal at $(x_1, y_1)$ | Normal at parameter $t$ |
|----------|------------------------|-------------------------|
| $y^2 = 4ax$ | $y - y_1 = -\frac{y_1}{2a}(x - x_1)$ | $y + tx = 2at + at^3$ |
| $y^2 = -4ax$ | $y - y_1 = \frac{y_1}{2a}(x - x_1)$ | $y - tx = 2at - at^3$ |
| $x^2 = 4ay$ | $x - x_1 = -\frac{x_1}{2a}(y - y_1)$ | $x + ty = 2at + at^3$ |
| $x^2 = -4ay$ | $x - x_1 = \frac{x_1}{2a}(y - y_1)$ | $x - ty = 2at - at^3$ |

---

## ✏️ Worked Examples

### Example 1: Normal at a Point

**Problem**: Find the normal to $y^2 = 8x$ at $(2, 4)$.

**Solution**:

$4a = 8$, so $a = 2$

Slope of normal = $-\frac{y_1}{2a} = -\frac{4}{4} = -1$

Normal: $y - 4 = -1(x - 2)$
$$y = -x + 6$$

**Answer**: $x + y = 6$

---

### Example 2: Normal at Parameter

**Problem**: Find the normal to $y^2 = 12x$ at $t = 2$.

**Solution**:

$a = 3$

Normal: $y + tx = 2at + at^3$
$$y + 2x = 2(3)(2) + 3(8) = 12 + 24 = 36$$

**Answer**: $2x + y = 36$

---

### Example 3: Normal with Given Slope

**Problem**: Find the normal to $y^2 = 4x$ with slope 2.

**Solution**:

$a = 1$, $m = 2$

Normal: $y = mx - 2am - am^3$
$$y = 2x - 2(1)(2) - 1(8) = 2x - 4 - 8 = 2x - 12$$

Point of contact: $(am^2, -2am) = (4, -4)$

**Verification**: $(-4)^2 = 16 = 4 \times 4$ ✓

**Answer**: $y = 2x - 12$ at point $(4, -4)$

---

### Example 4: Normal Passing Through a Point

**Problem**: Find the normals to $y^2 = 8x$ that pass through $(14, -8)$.

**Solution**:

$a = 2$

Normal: $y = mx - 4m - 2m^3$

Through $(14, -8)$:
$$-8 = 14m - 4m - 2m^3$$
$$-8 = 10m - 2m^3$$
$$m^3 - 5m - 4 = 0$$

Testing: $m = -1$: $-1 + 5 - 4 = 0$ ✓

Factoring: $(m + 1)(m^2 - m - 4) = 0$

$m = -1$ or $m = \frac{1 \pm \sqrt{17}}{2}$

For $m = -1$: $y = -x + 4 + 2 = -x + 6$, i.e., $x + y = 6$

The other two normals have slopes $\frac{1 + \sqrt{17}}{2}$ and $\frac{1 - \sqrt{17}}{2}$.

**Answer**: Three normals exist. One is $x + y = 6$.

---

## 📝 Co-Normal Points

### Definition

Three points $P$, $Q$, $R$ on a parabola are called **co-normal points** if the normals at these points are concurrent (pass through the same point).

```
                 ●R
                /│
               / │
              /  │
          ───●───┼───────────────
             Q\  │
               \ │
                \│
                 ●────────────●P
                              
    Normals at P, Q, R meet at a single point
    P, Q, R are co-normal points
```

---

### Properties of Co-Normal Points

If $t_1$, $t_2$, $t_3$ are parameters of three co-normal points:

1. **Sum of parameters**: $t_1 + t_2 + t_3 = 0$

2. **Centroid**: The centroid of triangle formed by co-normal points lies on the axis of the parabola.

3. **Maximum 3 Normals**: At most 3 normals can be drawn from any point to a parabola.

---

### Derivation of $t_1 + t_2 + t_3 = 0$

Normal at parameter $t$: $y + tx = 2at + at^3$

If this passes through $(h, k)$:
$$k + th = 2at + at^3$$
$$at^3 + (2a - h)t - k = 0$$

This is a cubic in $t$. If $t_1$, $t_2$, $t_3$ are roots:

By Vieta's formulas:
$$t_1 + t_2 + t_3 = -\frac{\text{coefficient of } t^2}{\text{coefficient of } t^3} = 0$$

$$\boxed{t_1 + t_2 + t_3 = 0}$$

---

### Example 5: Co-Normal Points

**Problem**: If normals at $t_1$, $t_2$, $t_3$ on $y^2 = 4ax$ are concurrent, prove their centroid lies on axis.

**Solution**:

Points: $(at_1^2, 2at_1)$, $(at_2^2, 2at_2)$, $(at_3^2, 2at_3)$

Centroid y-coordinate:
$$\bar{y} = \frac{2a(t_1 + t_2 + t_3)}{3} = \frac{2a \times 0}{3} = 0$$

So centroid lies on the x-axis (axis of parabola). ∎

---

## 📐 Relation Between Tangent and Normal

| Property | Tangent | Normal |
|----------|---------|--------|
| Slope at parameter $t$ | $\frac{1}{t}$ | $-t$ |
| Equation at $t$ | $ty = x + at^2$ | $y + tx = 2at + at^3$ |
| Slope form | $y = mx + \frac{a}{m}$ | $y = mx - 2am - am^3$ |
| X-intercept | $-at^2$ | $2a + at^2$ |

---

### Example 6: Tangent and Normal at Same Point

**Problem**: Find the tangent and normal at $t = 1$ on $y^2 = 16x$. Find where they meet the axis.

**Solution**:

$a = 4$, $t = 1$

Point: $(at^2, 2at) = (4, 8)$

**Tangent**: $ty = x + at^2$
$y = x + 4$
Meets axis ($y = 0$): $x = -4$
Point: $(-4, 0)$

**Normal**: $y + tx = 2at + at^3$
$y + x = 8 + 4 = 12$
Meets axis ($y = 0$): $x = 12$
Point: $(12, 0)$

**Answer**: Tangent meets axis at $(-4, 0)$, Normal meets axis at $(12, 0)$

---

## 📝 Length of Subnormal

The **subnormal** is the projection of the normal on the axis (from foot of ordinate to where normal meets axis).

For $y^2 = 4ax$ at $(at^2, 2at)$:

Normal meets axis at $(2a + at^2, 0)$

Foot of ordinate: $(at^2, 0)$

$$\text{Subnormal} = (2a + at^2) - at^2 = 2a$$

> **Key Result**: The subnormal is constant = $2a$ = semi-latus rectum.

---

## 💡 Tips and Insights

> 💡 **Cubic Equation**: Finding normals from a point leads to a cubic in $m$ or $t$, giving at most 3 solutions.

> 💡 **Co-Normal Sum**: For co-normal points, $t_1 + t_2 + t_3 = 0$ is very useful.

> ⚠️ **Sign of Slope**: Normal slope = $-t$, which is negative when $t > 0$.

> 💡 **Subnormal**: Always equals $2a$, regardless of the point on parabola.

> 💡 **Foot of Normal**: The normal at $(at^2, 2at)$ meets the parabola again at parameter $t' = -t - \frac{2}{t}$.

---

## 📝 Normal Chord

If normal at parameter $t_1$ meets the parabola again at $t_2$:

$$\boxed{t_2 = -t_1 - \frac{2}{t_1}}$$

### Derivation

The normal at $t_1$ passes through $(at_2^2, 2at_2)$:
$$2at_2 + t_1 \cdot at_2^2 = 2at_1 + at_1^3$$
$$2t_2 + t_1 t_2^2 = 2t_1 + t_1^3$$
$$t_1 t_2^2 - 2t_1 = t_1^3 - 2t_2$$
$$t_1(t_2^2 - 2 - t_1^2) = -2t_2$$

Actually, let me derive correctly:
$$t_1 t_2^2 + 2t_2 = t_1^3 + 2t_1$$
$$t_1(t_2^2 - t_1^2) = 2(t_1 - t_2)$$
$$t_1(t_2 - t_1)(t_2 + t_1) = -2(t_2 - t_1)$$
$$t_1(t_2 + t_1) = -2$$ (since $t_2 \neq t_1$)
$$t_1 t_2 + t_1^2 = -2$$
$$t_2 = -t_1 - \frac{2}{t_1}$$

---

## 📋 Summary Table

| Form | Normal Equation | Contact Point/Condition |
|------|-----------------|------------------------|
| At $(x_1, y_1)$ | $y - y_1 = -\frac{y_1}{2a}(x - x_1)$ | $(x_1, y_1)$ on curve |
| At parameter $t$ | $y + tx = 2at + at^3$ | $(at^2, 2at)$ |
| Slope $m$ | $y = mx - 2am - am^3$ | $(am^2, -2am)$ |
| Co-normal condition | - | $t_1 + t_2 + t_3 = 0$ |
| Normal chord | - | $t_2 = -t_1 - \frac{2}{t_1}$ |
| Subnormal | - | Always $= 2a$ |

---

## ❓ Quick Revision Questions

1. **Find normal to $y^2 = 4x$ at $(4, 4)$.**

2. **Find normal to $y^2 = 8x$ at parameter $t = -1$.**

3. **Find normal to $y^2 = 12x$ with slope $-3$.**

4. **If normal at $t = 2$ on $y^2 = 4x$ meets the curve again, find the second point.**

5. **How many normals can be drawn from $(15, 0)$ to $y^2 = 4x$?**

6. **Prove that the subnormal to $y^2 = 4ax$ is constant.**

<details>
<summary><b>Click to see answers</b></summary>

1. $a = 1$, $(x_1, y_1) = (4, 4)$
   Slope = $-\frac{y_1}{2a} = -\frac{4}{2} = -2$
   $y - 4 = -2(x - 4)$
   **$2x + y = 12$**

2. $a = 2$, $t = -1$
   Point: $(2, -4)$
   $y + tx = 2at + at^3$
   $y - x = -4 - 2 = -6$
   **$x - y = 6$**

3. $a = 3$, $m = -3$
   $y = mx - 2am - am^3$
   $y = -3x - 2(3)(-3) - 3(-27) = -3x + 18 + 81 = -3x + 99$
   **$3x + y = 99$** at point $(27, 18)$

4. $a = 1$, $t_1 = 2$
   $t_2 = -t_1 - \frac{2}{t_1} = -2 - 1 = -3$
   Second point: $(a t_2^2, 2a t_2) = (9, -6)$
   **Second point: $(9, -6)$**

5. Normal from $(h, k) = (15, 0)$: $at^3 + (2a - h)t - k = 0$
   $t^3 + (2 - 15)t - 0 = 0$
   $t^3 - 13t = 0$
   $t(t^2 - 13) = 0$
   $t = 0, \pm\sqrt{13}$
   **3 normals** (corresponding to 3 real values of t)

6. Normal at $(at^2, 2at)$: $y + tx = 2at + at^3$
   At y = 0: $tx = 2at + at^3$, $x = 2a + at^2$
   Foot of ordinate: $x = at^2$
   Subnormal = $(2a + at^2) - at^2 = 2a$ = constant ∎

</details>

---

## 🎉 Unit 5 Complete!

Congratulations! You've completed the study of **Parabola**. Key concepts covered:

✅ Standard equations and four orientations  
✅ Focus, directrix, vertex, axis, latus rectum  
✅ Focal chord properties  
✅ Tangent equations (point, parametric, slope forms)  
✅ Normal equations and co-normal points  

---

## ⏭️ Navigation

| [← Previous: Equation of Tangent](04-equation-of-tangent.md) | [Unit 5 Home](README.md) | [Next Unit: Ellipse →](../06-Ellipse/README.md) |
|:-------------------------------------------------------------|:------------------------:|------------------------------------------------:|
