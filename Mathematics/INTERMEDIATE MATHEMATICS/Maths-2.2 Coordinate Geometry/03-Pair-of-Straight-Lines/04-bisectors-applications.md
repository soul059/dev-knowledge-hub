# Chapter 3.4: Angle Bisectors and Applications

## 📚 Chapter Overview

The angle bisectors of a pair of lines have special properties and their own joint equation. This chapter explores bisector equations and various applications of pair of lines in geometry.

---

## 📝 Angle Bisectors of Pair of Lines

### Definition

The **angle bisectors** of two intersecting lines are two lines that divide the angles formed by the original lines into two equal parts.

```
        Y
        ↑
        │    L₁/    \L₂
        │   /    \   
        │  /  B₁  \   B₁ = Bisector of acute angle
        │ /    \   \
        │/______\___\
    ────●───────────→ X
        │\      /
        │ \B₂ /      B₂ = Bisector of obtuse angle
        │  \ /
        │   ×
```

**Key Property**: The two bisectors are always perpendicular to each other.

---

## ⚡ Equation of Bisectors (Lines Through Origin)

> **Bisector Equation for $ax^2 + 2hxy + by^2 = 0$**:
>
> $$\boxed{\frac{x^2 - y^2}{a - b} = \frac{xy}{h}}$$
>
> Or equivalently:
> $$\boxed{h(x^2 - y^2) = (a - b)xy}$$

---

## 📐 Derivation

Let the pair of lines be $y = m_1 x$ and $y = m_2 x$.

A point P(x, y) lies on a bisector if it is equidistant from both lines.

Distance from $(x, y)$ to $y = m_1 x$: $\frac{|y - m_1 x|}{\sqrt{1 + m_1^2}}$

Distance from $(x, y)$ to $y = m_2 x$: $\frac{|y - m_2 x|}{\sqrt{1 + m_2^2}}$

For bisector: These distances are equal.

After algebraic manipulation using:
- $m_1 + m_2 = -\frac{2h}{b}$
- $m_1 m_2 = \frac{a}{b}$

We get: $h(x^2 - y^2) = (a - b)xy$

---

## 📝 Properties of Bisectors

### 1. Perpendicularity

The two bisectors are always perpendicular.

**Proof**: The bisector equation $h(x^2 - y^2) - (a-b)xy = 0$ represents a pair of lines.

Comparing with $Ax^2 + 2Hxy + By^2 = 0$:
- $A = h$
- $2H = -(a-b)$
- $B = -h$

For perpendicular: $A + B = h + (-h) = 0$ ✓

### 2. Equal Division

Each bisector divides the angle into two equal parts.

### 3. Symmetry

The pair of original lines and the pair of bisectors are symmetric about each bisector.

---

## ✏️ Worked Examples

### Example 1: Finding Bisector Equation

**Problem**: Find the equation of angle bisectors of lines $x^2 - 7xy + 6y^2 = 0$.

**Solution**:

Here: $a = 1$, $2h = -7$ → $h = -\frac{7}{2}$, $b = 6$

Using $\frac{x^2 - y^2}{a - b} = \frac{xy}{h}$:

$$\frac{x^2 - y^2}{1 - 6} = \frac{xy}{-7/2}$$

$$\frac{x^2 - y^2}{-5} = \frac{-2xy}{7}$$

$$7(x^2 - y^2) = 10xy$$

$$7x^2 - 10xy - 7y^2 = 0$$

**Answer**: Bisectors are $7x^2 - 10xy - 7y^2 = 0$

---

### Example 2: Separate Bisector Equations

**Problem**: Find the separate equations of bisectors from Example 1.

**Solution**:

Factor $7x^2 - 10xy - 7y^2 = 0$

Divide by $y^2$: $7\left(\frac{x}{y}\right)^2 - 10\left(\frac{x}{y}\right) - 7 = 0$

Let $t = \frac{x}{y}$: $7t^2 - 10t - 7 = 0$

$$t = \frac{10 \pm \sqrt{100 + 196}}{14} = \frac{10 \pm \sqrt{296}}{14}$$

Hmm, not factoring nicely. Let's try direct factorization:

$7x^2 - 10xy - 7y^2 = 7x^2 - 14xy + 4xy - 7y^2$
$= 7x(x - 2y) + 7y \cdot ???$ - not working cleanly.

Using quadratic formula for $7m^2 - 10m - 7 = 0$ where $m = x/y$:
$m = \frac{10 \pm \sqrt{100 + 196}}{14} = \frac{10 \pm 2\sqrt{74}}{14} = \frac{5 \pm \sqrt{74}}{7}$

**Answer**: $7x - (5 + \sqrt{74})y = 0$ and $7x - (5 - \sqrt{74})y = 0$

---

### Example 3: Verifying Perpendicularity of Bisectors

**Problem**: Show that the bisectors of $3x^2 + 4xy - 5y^2 = 0$ are perpendicular.

**Solution**:

Bisector equation: $\frac{x^2 - y^2}{a - b} = \frac{xy}{h}$

Here: $a = 3$, $h = 2$, $b = -5$

$$\frac{x^2 - y^2}{3 - (-5)} = \frac{xy}{2}$$

$$\frac{x^2 - y^2}{8} = \frac{xy}{2}$$

$$x^2 - y^2 = 4xy$$

$$x^2 - 4xy - y^2 = 0$$

Check perpendicularity: $a' + b' = 1 + (-1) = 0$ ✓

**Answer**: Bisectors are perpendicular.

---

### Example 4: Special Case - Equal Coefficients

**Problem**: Find the bisectors of $x^2 + 2xy - y^2 = 0$.

**Solution**:

Here: $a = 1$, $h = 1$, $b = -1$

$$\frac{x^2 - y^2}{1 - (-1)} = \frac{xy}{1}$$

$$\frac{x^2 - y^2}{2} = xy$$

$$x^2 - y^2 = 2xy$$

$$x^2 - 2xy - y^2 = 0$$

Factor: Using quadratic formula for $m^2 - 2m - 1 = 0$ where $m = x/y$:
$m = \frac{2 \pm \sqrt{4 + 4}}{2} = 1 \pm \sqrt{2}$

**Answer**: Bisectors are $x = (1 + \sqrt{2})y$ and $x = (1 - \sqrt{2})y$

---

## 📝 Identifying Acute and Obtuse Angle Bisectors

### Method 1: Using Slopes

1. Find the slopes of original lines: $m_1$, $m_2$
2. Find slopes of bisectors: $m_1'$, $m_2'$
3. The bisector with slope closer to $(m_1 + m_2)/2$ bisects the acute angle.

### Method 2: Coefficient Test

For original lines $l_1 x + m_1 y = 0$ and $l_2 x + m_2 y = 0$:

- If $l_1 l_2 + m_1 m_2 > 0$: Acute angle bisector is $\frac{l_1 x + m_1 y}{\sqrt{l_1^2 + m_1^2}} = \frac{l_2 x + m_2 y}{\sqrt{l_2^2 + m_2^2}}$

- If $l_1 l_2 + m_1 m_2 < 0$: The above gives obtuse angle bisector.

---

## 📝 Applications of Pair of Lines

### Application 1: Pair of Tangents from External Point

The pair of tangents from an external point to a conic forms a pair of lines.

```
                   ●───────────────●
                  /     Circle     \
                 /                  \
                /                    \
        P ●────●────────────────────●
                \                    /
                 \                  /
                  \                /
                   ●───────────────●
        
    P = External point, two tangent lines shown
```

### Application 2: Asymptotes of Hyperbola

The asymptotes of a hyperbola form a pair of lines through the center.

### Application 3: Homogenization Technique

Finding the joint equation of lines joining origin to intersection points.

---

### Example 5: Pair of Lines Joining Origin to Circle Intersection

**Problem**: Find the joint equation of lines joining origin to the points where line $x + y = 2$ meets circle $x^2 + y^2 = 4$.

**Solution**:

**Step 1**: Homogenize the circle using the line

From line: $\frac{x + y}{2} = 1$

**Step 2**: Substitute in circle equation
$$x^2 + y^2 = 4 \cdot 1^2 = 4\left(\frac{x+y}{2}\right)^2$$

$$x^2 + y^2 = \frac{4(x+y)^2}{4} = (x+y)^2$$

$$x^2 + y^2 = x^2 + 2xy + y^2$$

$$0 = 2xy$$

$$xy = 0$$

**Answer**: $xy = 0$ (i.e., $x = 0$ or $y = 0$, the coordinate axes)

---

### Example 6: Locus Problem

**Problem**: A line through origin meets lines $x + y = 1$ and $x - y = 3$ at P and Q respectively. Find the locus of the midpoint of PQ.

**Solution**:

Let the line through origin be $y = mx$.

**Intersection with $x + y = 1$**:
$x + mx = 1$ → $x = \frac{1}{1+m}$
$y = \frac{m}{1+m}$
So $P = \left(\frac{1}{1+m}, \frac{m}{1+m}\right)$

**Intersection with $x - y = 3$**:
$x - mx = 3$ → $x = \frac{3}{1-m}$
$y = \frac{3m}{1-m}$
So $Q = \left(\frac{3}{1-m}, \frac{3m}{1-m}\right)$

**Midpoint M(h, k)**:
$$h = \frac{1}{2}\left(\frac{1}{1+m} + \frac{3}{1-m}\right) = \frac{1}{2} \cdot \frac{1-m+3+3m}{(1+m)(1-m)} = \frac{2+m}{1-m^2}$$

$$k = \frac{1}{2}\left(\frac{m}{1+m} + \frac{3m}{1-m}\right) = \frac{m(1-m+3+3m)}{2(1-m^2)} = \frac{m(2+m)}{1-m^2} \cdot \frac{1}{2}$$

Wait, let me recalculate:
$$k = \frac{1}{2}\left(\frac{m(1-m) + 3m(1+m)}{(1+m)(1-m)}\right) = \frac{m - m^2 + 3m + 3m^2}{2(1-m^2)} = \frac{4m + 2m^2}{2(1-m^2)}$$

From above: $k = \frac{2m(2+m)}{2(1-m^2)} = \frac{m(2+m)}{1-m^2}$

And: $h = \frac{2+m}{1-m^2}$ (after correction)

So: $\frac{k}{h} = m$, meaning the locus point lies on the line through origin!

Since $m = k/h$, substitute back to eliminate m:
$$h = \frac{2 + k/h}{1 - k^2/h^2} = \frac{(2h+k)/h}{(h^2-k^2)/h^2} = \frac{(2h+k)h}{h^2-k^2}$$

$$h(h^2 - k^2) = h(2h + k)$$

$$h^2 - k^2 = 2h + k$$ (assuming h ≠ 0)

**Answer**: Locus is $x^2 - y^2 = 2x + y$ or $x^2 - y^2 - 2x - y = 0$

---

### Example 7: Combined Equation Given Conditions

**Problem**: Find the combined equation of lines through origin, each making angle 30° with line $x + y = 0$.

**Solution**:

Slope of given line: $m = -1$

Lines making 30° angle with this line have slopes $m_1$ and $m_2$ where:

$$\tan 30° = \left|\frac{m_i - (-1)}{1 + m_i(-1)}\right| = \left|\frac{m_i + 1}{1 - m_i}\right|$$

$$\frac{1}{\sqrt{3}} = \left|\frac{m + 1}{1 - m}\right|$$

**Case 1**: $\frac{m + 1}{1 - m} = \frac{1}{\sqrt{3}}$
$\sqrt{3}(m + 1) = 1 - m$
$\sqrt{3}m + \sqrt{3} = 1 - m$
$m(\sqrt{3} + 1) = 1 - \sqrt{3}$
$m_1 = \frac{1 - \sqrt{3}}{1 + \sqrt{3}} = \frac{(1-\sqrt{3})^2}{1-3} = \frac{4 - 2\sqrt{3}}{-2} = \sqrt{3} - 2$

**Case 2**: $\frac{m + 1}{1 - m} = -\frac{1}{\sqrt{3}}$
$\sqrt{3}(m + 1) = -(1 - m) = m - 1$
$\sqrt{3}m + \sqrt{3} = m - 1$
$m(\sqrt{3} - 1) = -1 - \sqrt{3}$
$m_2 = \frac{-1 - \sqrt{3}}{\sqrt{3} - 1} = \frac{-(1+\sqrt{3})(\sqrt{3}+1)}{(\sqrt{3}-1)(\sqrt{3}+1)} = \frac{-(4+2\sqrt{3})}{2} = -2 - \sqrt{3}$

**Sum**: $m_1 + m_2 = (\sqrt{3} - 2) + (-2 - \sqrt{3}) = -4$
**Product**: $m_1 \cdot m_2 = (\sqrt{3} - 2)(-2 - \sqrt{3}) = -2\sqrt{3} - 3 + 4 + 2\sqrt{3} = 1$

The equation with these slopes: $y^2 - (m_1 + m_2)xy + m_1 m_2 x^2 = 0$

Wait, that's not quite right. For $y = mx$, we have $y^2 + 4xy + x^2 = 0$... let me verify.

Actually, for lines $y = m_1 x$ and $y = m_2 x$:
$(y - m_1 x)(y - m_2 x) = 0$
$y^2 - (m_1 + m_2)xy + m_1 m_2 x^2 = 0$
$y^2 - (-4)xy + (1)x^2 = 0$

**Answer**: $x^2 + 4xy + y^2 = 0$

---

## 💡 Tips and Insights

> 💡 **Bisector Perpendicularity**: Bisectors are always perpendicular - use this as a check.

> 💡 **When a = b**: If $a = b$, the bisector equation becomes $xy = 0$, i.e., the coordinate axes.

> ⚠️ **Division by Zero**: When $h = 0$, the bisector formula needs modification; the bisectors are $x^2 = y^2$, i.e., $x = \pm y$.

> 💡 **Homogenization**: To find lines through origin to intersection points, replace constants appropriately.

---

## 🌍 Real-World Applications

1. **Optics**: Angle bisectors in reflection problems
2. **Engineering**: Stress analysis at material joints
3. **Architecture**: Symmetric design elements
4. **Navigation**: Equal-angle routes
5. **Surveying**: Land division into equal angular sections

---

## 📋 Summary Table

| Concept | Formula/Property |
|---------|------------------|
| Bisector equation | $\frac{x^2 - y^2}{a-b} = \frac{xy}{h}$ |
| Alternative form | $h(x^2 - y^2) = (a-b)xy$ |
| Bisectors are... | Always perpendicular |
| If $a = b$ | Bisectors: $xy = 0$ (axes) |
| If $h = 0$ | Bisectors: $x^2 - y^2 = 0$ ($x = \pm y$) |

---

## 🎉 Unit 3 Complete!

Congratulations on completing Unit 3: Pair of Straight Lines! Key takeaways:

✅ Homogeneous equations represent lines through origin  
✅ Determinant condition for general equation to represent pair  
✅ Angle formula using coefficients  
✅ Perpendicular condition: $a + b = 0$  
✅ Bisector equation and its properties  

---

## ❓ Quick Revision Questions

1. **Find the equation of bisectors of $3x^2 - 5xy - 2y^2 = 0$.**

2. **Show that bisectors of any pair of lines through origin are perpendicular.**

3. **Find the pair of lines through origin making angle 45° with line $y = x$.**

4. **Find joint equation of lines joining origin to intersection of $y = x + 3$ and circle $x^2 + y^2 = 9$.**

5. **If bisectors of $ax^2 + 2hxy + by^2 = 0$ are the coordinate axes, find the relation between a, b, h.**

6. **Find the acute angle bisector of lines $x + y = 0$ and $x - 7y = 0$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $a = 3$, $h = -\frac{5}{2}$, $b = -2$
   $\frac{x^2 - y^2}{3-(-2)} = \frac{xy}{-5/2}$
   $\frac{x^2 - y^2}{5} = \frac{-2xy}{5}$
   **$x^2 + 2xy - y^2 = 0$** or $x^2 - y^2 = -2xy$

2. Bisector equation: $h(x^2 - y^2) - (a-b)xy = 0$
   Coefficients: $A = h$, $B = -h$
   $A + B = h - h = 0$ ✓
   **Bisectors are always perpendicular**

3. Line $y = x$ has slope 1
   For 45° angle: $\tan 45° = 1 = \left|\frac{m-1}{1+m}\right|$
   Case 1: $m - 1 = 1 + m$ → No solution
   Case 2: $m - 1 = -(1 + m)$ → $2m = 0$ → $m = 0$
   Case 3: $1 - m = 1 + m$ → $m = 0$
   Case 4: $-(m-1) = 1+m$ → $m = 0$
   Also checking: $|m-1| = |1+m|$ gives $m = 0$ or undefined
   Lines: $y = 0$ (x-axis) and... need to reconsider.
   Actually for 45°: slopes are $m = 0$ and $m = \infty$ (y-axis)
   **$xy = 0$**

4. From $y = x + 3$: $\frac{y-x}{3} = 1$
   $x^2 + y^2 = 9 \cdot 1^2 = 9 \cdot \frac{(y-x)^2}{9}$
   $x^2 + y^2 = (y-x)^2 = y^2 - 2xy + x^2$
   $0 = -2xy$
   **$xy = 0$**

5. If bisectors are $xy = 0$, the bisector equation must be $xy = 0$.
   From $h(x^2 - y^2) = (a-b)xy$:
   For this to be $xy = 0$, we need $h = 0$ and $a - b = 0$... but that gives $0 = 0$.
   
   Actually, if $h = 0$: bisectors are $x^2 - y^2 = 0$ → $x = \pm y$
   For bisectors to be $xy = 0$ (axes): need $a = b$ (then any h gives axes bisectors)
   **$a = b$** (coefficient of $x^2$ = coefficient of $y^2$)

6. Lines: $x + y = 0$ ($m_1 = -1$) and $x - 7y = 0$ ($m_2 = 1/7$)
   Joint equation: $(x+y)(x-7y) = x^2 - 7xy + xy - 7y^2 = x^2 - 6xy - 7y^2$
   $a = 1$, $h = -3$, $b = -7$
   Bisector: $\frac{x^2 - y^2}{1-(-7)} = \frac{xy}{-3}$
   $\frac{x^2 - y^2}{8} = \frac{-xy}{3}$
   $3(x^2 - y^2) = -8xy$
   **$3x^2 + 8xy - 3y^2 = 0$**

</details>

---

## ⏭️ Navigation

| [← Previous: Angle Between Pair](03-angle-between-pair.md) | [Back to Unit 3 Overview](README.md) |
|:----------------------------------------------------------|-------------------------------------:|
| | [Next Unit: Circle →](../04-Circle/README.md) |
