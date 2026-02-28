# Chapter 4.7: Family of Circles

## 📚 Chapter Overview

Just as lines can form families, circles too can be grouped into families based on common properties. This chapter explores families of circles passing through intersection points, coaxial systems, and orthogonal circles—concepts crucial for advanced problem-solving.

---

## 📝 Family Through Intersection of Two Circles

### Definition

If $S_1 = 0$ and $S_2 = 0$ are two circles that intersect, then:

$$\boxed{S_1 + \lambda S_2 = 0}$$

represents a **family of circles** passing through the intersection points of $S_1$ and $S_2$.

Here, λ is a parameter (λ ≠ -1 when coefficients of $x^2$ are equal).

```
           S₁                 S₂
          ○                   ○
         /  \               /  \
        /    \             /    \
       │      ●───────────●      │
        \    / A         B \    /
         \  /               \  /
          ○                   ○
           
     Every circle through A and B
     can be written as S₁ + λS₂ = 0
     for some value of λ
```

---

## 📐 Derivation

Let the two circles be:
- $S_1: x^2 + y^2 + 2g_1x + 2f_1y + c_1 = 0$
- $S_2: x^2 + y^2 + 2g_2x + 2f_2y + c_2 = 0$

For any point $(x_1, y_1)$ on both circles:
- $S_1(x_1, y_1) = 0$
- $S_2(x_1, y_1) = 0$

Therefore: $S_1(x_1, y_1) + \lambda S_2(x_1, y_1) = 0 + \lambda \cdot 0 = 0$

So every member of $S_1 + \lambda S_2 = 0$ passes through the common points.

---

## 📝 Special Case: Radical Axis

When $\lambda = -1$:

$$S_1 - S_2 = 0$$

This is a **linear equation** (the $x^2$ and $y^2$ terms cancel), giving the **radical axis** of the two circles.

$$\boxed{2(g_1 - g_2)x + 2(f_1 - f_2)y + (c_1 - c_2) = 0}$$

The radical axis is the locus of points having equal tangent lengths to both circles.

---

## ⚡ Family Through Circle and Line

If $S = 0$ is a circle and $L = 0$ is a line that intersects it, then:

$$\boxed{S + \lambda L = 0}$$

represents the family of circles passing through the two intersection points.

```
        Line L
       ────────●──────●────────
              / A     B \
             /           \
            │     ●       │  Circle S
             \           /
              \         /
               ─────────
               
     S + λL = 0 gives circles through A and B
```

> **Note**: For $\lambda = 0$, we get the original circle $S = 0$.

---

## ✏️ Worked Examples

### Example 1: Basic Family of Circles

**Problem**: Find the family of circles through the intersection of:
- $S_1: x^2 + y^2 - 4x - 6y + 9 = 0$
- $S_2: x^2 + y^2 - 2x - 4y + 1 = 0$

**Solution**:

Family: $S_1 + \lambda S_2 = 0$

$(x^2 + y^2 - 4x - 6y + 9) + \lambda(x^2 + y^2 - 2x - 4y + 1) = 0$

$(1 + \lambda)x^2 + (1 + \lambda)y^2 - (4 + 2\lambda)x - (6 + 4\lambda)y + (9 + \lambda) = 0$

Dividing by $(1 + \lambda)$ when $\lambda \neq -1$:

$$x^2 + y^2 - \frac{4 + 2\lambda}{1 + \lambda}x - \frac{6 + 4\lambda}{1 + \lambda}y + \frac{9 + \lambda}{1 + \lambda} = 0$$

**Answer**: $(1 + \lambda)(x^2 + y^2) - (4 + 2\lambda)x - (6 + 4\lambda)y + (9 + \lambda) = 0$

---

### Example 2: Circle Through Intersection with Condition

**Problem**: Find the circle through the intersection of $x^2 + y^2 - 4 = 0$ and $x^2 + y^2 - 2x - 4y + 4 = 0$, passing through origin.

**Solution**:

Family: $S_1 + \lambda S_2 = 0$

$(x^2 + y^2 - 4) + \lambda(x^2 + y^2 - 2x - 4y + 4) = 0$

Since it passes through (0, 0):
$(-4) + \lambda(4) = 0$
$\lambda = 1$

Required circle:
$(x^2 + y^2 - 4) + 1 \cdot (x^2 + y^2 - 2x - 4y + 4) = 0$
$2x^2 + 2y^2 - 2x - 4y = 0$
$x^2 + y^2 - x - 2y = 0$

**Answer**: $x^2 + y^2 - x - 2y = 0$

---

### Example 3: Finding Radical Axis

**Problem**: Find the radical axis of:
- $C_1: x^2 + y^2 - 6x - 4y + 4 = 0$
- $C_2: x^2 + y^2 + 2x + 6y - 3 = 0$

**Solution**:

Radical axis: $S_1 - S_2 = 0$

$(x^2 + y^2 - 6x - 4y + 4) - (x^2 + y^2 + 2x + 6y - 3) = 0$

$-6x - 4y + 4 - 2x - 6y + 3 = 0$

$-8x - 10y + 7 = 0$

**Answer**: $8x + 10y - 7 = 0$

---

### Example 4: Circle Through Circle-Line Intersection

**Problem**: Find the circle through the intersection of $x^2 + y^2 = 16$ and line $x + y = 4$, passing through (6, 0).

**Solution**:

Family: $(x^2 + y^2 - 16) + \lambda(x + y - 4) = 0$

Passes through (6, 0):
$(36 - 16) + \lambda(6 + 0 - 4) = 0$
$20 + 2\lambda = 0$
$\lambda = -10$

Required circle:
$(x^2 + y^2 - 16) - 10(x + y - 4) = 0$
$x^2 + y^2 - 10x - 10y + 24 = 0$

**Verification**: Center (5, 5), radius = $\sqrt{25 + 25 - 24} = \sqrt{26}$

**Answer**: $x^2 + y^2 - 10x - 10y + 24 = 0$

---

### Example 5: Smallest Circle Through Intersection

**Problem**: Find the smallest circle through the intersection of $x^2 + y^2 - 2x - 4y - 4 = 0$ and $x + y = 2$.

**Solution**:

Family: $(x^2 + y^2 - 2x - 4y - 4) + \lambda(x + y - 2) = 0$

$x^2 + y^2 + (λ - 2)x + (λ - 4)y - (4 + 2λ) = 0$

Center: $\left(\frac{2-\lambda}{2}, \frac{4-\lambda}{2}\right)$

For smallest circle, the center lies on the chord (line $x + y = 2$):

$\frac{2-\lambda}{2} + \frac{4-\lambda}{2} = 2$

$\frac{6 - 2\lambda}{2} = 2$

$6 - 2\lambda = 4$

$\lambda = 1$

Required circle:
$x^2 + y^2 - x - 3y - 6 = 0$

**Answer**: $x^2 + y^2 - x - 3y - 6 = 0$ with center $\left(\frac{1}{2}, \frac{3}{2}\right)$

---

## 📝 Coaxial System of Circles

### Definition

A **coaxial system** (or coaxal system) is a family of circles such that every pair has the same radical axis.

### Standard Coaxial System

$$\boxed{x^2 + y^2 + 2gx + c = 0}$$

where $g$ is the parameter and $c$ is constant.

- All circles have centers on x-axis: $(-g, 0)$
- Common radical axis: $y$-axis (x = 0)

```
     Radical Axis (y-axis)
            │
            │    ○
            │   /│\     Each circle has center
     ○      │  / │ \    on x-axis
    /│\     │ │  │  │
   / │ \    │ │  ●  │   Limiting points (if exist)
  │  ●  │   │ │     │   are common to all
   \ │ /    │  \   /
    \│/     │   \ /
     ○      │    ○
            │
```

### Properties of Coaxial System

1. Centers are collinear (lie on a line)
2. Common radical axis is perpendicular to line of centers
3. May have **limiting points** (point circles in the system)

---

## 📐 Limiting Points

For coaxial system $x^2 + y^2 + 2gx + c = 0$:

Radius = $\sqrt{g^2 - c}$

When $g^2 = c$, radius = 0, giving **limiting points** at $(\pm\sqrt{c}, 0)$

> **Limiting points exist only when $c > 0$** (intersecting type coaxial system has no limiting points).

---

## 📝 Orthogonal Circles

### Definition

Two circles are **orthogonal** if they intersect at right angles (tangents at intersection point are perpendicular).

```
           ○
          /│\
         / │ \
        │  │  │
     ───●──┼──●───
        │  │  │
         \ │ /
          \│/
           ○
           
    At intersection point:
    Tangent to one circle = Radius of other
    (Radii are perpendicular)
```

### Condition for Orthogonality

For circles:
- $S_1: x^2 + y^2 + 2g_1x + 2f_1y + c_1 = 0$
- $S_2: x^2 + y^2 + 2g_2x + 2f_2y + c_2 = 0$

$$\boxed{2g_1g_2 + 2f_1f_2 = c_1 + c_2}$$

### Derivation

Let centers be $C_1(-g_1, -f_1)$ and $C_2(-g_2, -f_2)$.

At intersection point P:
- $C_1P \perp C_2P$ (tangent to one = radius of other)
- $C_1P^2 + C_2P^2 = C_1C_2^2$ (Pythagoras)

$r_1^2 + r_2^2 = d^2$ where $d = $ distance between centers

$(g_1^2 + f_1^2 - c_1) + (g_2^2 + f_2^2 - c_2) = (g_1 - g_2)^2 + (f_1 - f_2)^2$

$g_1^2 + f_1^2 - c_1 + g_2^2 + f_2^2 - c_2 = g_1^2 - 2g_1g_2 + g_2^2 + f_1^2 - 2f_1f_2 + f_2^2$

$-c_1 - c_2 = -2g_1g_2 - 2f_1f_2$

$$2g_1g_2 + 2f_1f_2 = c_1 + c_2$$

---

### Example 6: Orthogonality Check

**Problem**: Are circles $x^2 + y^2 - 2x - 6y + 6 = 0$ and $x^2 + y^2 + 4x + 2y - 5 = 0$ orthogonal?

**Solution**:

Circle 1: $g_1 = -1$, $f_1 = -3$, $c_1 = 6$
Circle 2: $g_2 = 2$, $f_2 = 1$, $c_2 = -5$

LHS: $2g_1g_2 + 2f_1f_2 = 2(-1)(2) + 2(-3)(1) = -4 - 6 = -10$

RHS: $c_1 + c_2 = 6 + (-5) = 1$

LHS ≠ RHS

**Answer**: **Not orthogonal**

---

### Example 7: Finding Orthogonal Circle

**Problem**: Find the circle passing through (1, 1) orthogonal to $x^2 + y^2 - 4x + 2y - 11 = 0$ with center on x-axis.

**Solution**:

Let required circle: $x^2 + y^2 + 2gx + c = 0$ (center on x-axis means $f = 0$)

Given circle: $g_2 = -2$, $f_2 = 1$, $c_2 = -11$

Orthogonality condition:
$2g(-2) + 2(0)(1) = c + (-11)$
$-4g = c - 11$
$c = 11 - 4g$ ... (1)

Passes through (1, 1):
$1 + 1 + 2g + c = 0$
$c = -2 - 2g$ ... (2)

From (1) and (2):
$11 - 4g = -2 - 2g$
$13 = 2g$
$g = \frac{13}{2}$

$c = -2 - 2(\frac{13}{2}) = -2 - 13 = -15$

**Answer**: $x^2 + y^2 + 13x - 15 = 0$

---

## 📝 Orthogonal Family

A circle orthogonal to all circles of a coaxial system also forms a coaxial system.

```
    ┌────────────────────────────────────────────────────────┐
    │            COAXIAL SYSTEMS RELATIONSHIP                │
    ├────────────────────────────────────────────────────────┤
    │                                                        │
    │   Original System         Orthogonal System            │
    │   ───────────────         ─────────────────            │
    │   x² + y² + 2gx + c = 0   x² + y² + 2fy - c = 0        │
    │   Centers on x-axis       Centers on y-axis            │
    │   Radical axis: y-axis    Radical axis: x-axis         │
    │                                                        │
    │   Every circle of one system is orthogonal to          │
    │   every circle of the other system.                    │
    │                                                        │
    └────────────────────────────────────────────────────────┘
```

---

## 💡 Tips and Insights

> 💡 **Quick Check**: For orthogonality, just verify $2g_1g_2 + 2f_1f_2 = c_1 + c_2$.

> 💡 **Radical Axis**: Always found by $S_1 - S_2 = 0$ regardless of whether circles intersect.

> 💡 **Smallest Circle**: The smallest circle through two points has those points as diameter endpoints.

> ⚠️ **λ ≠ -1**: When writing $S_1 + \lambda S_2 = 0$, ensure λ ≠ -1 (otherwise you get a line, not a circle).

> 💡 **Three Circles**: The radical axes of three circles taken in pairs are concurrent (radical center).

---

## 📝 Radical Center

The **radical center** of three circles is the point where the three radical axes (taken pairwise) meet.

$$\text{Radical Center} = \text{Intersection of } (S_1 - S_2) \text{ and } (S_2 - S_3)$$

```
         ○ C₁
        / \
       /   \──────○ C₂
      /   ●         \
     /    Radical    \
    /     Center      \
   ○──────────────────○
   C₃
   
   Three radical axes are concurrent
```

---

## 🌍 Real-World Applications

1. **Gear Systems**: Meshing circular gears
2. **Venn Diagrams**: Set theory representations
3. **Lens Design**: Overlapping circular lenses
4. **Surveying**: Triangulation using circular arcs
5. **Astronomy**: Orbital intersections

---

## 📋 Summary Table

| Concept | Formula/Condition |
|---------|-------------------|
| Family through intersection of $S_1$ & $S_2$ | $S_1 + \lambda S_2 = 0$ |
| Family through circle-line intersection | $S + \lambda L = 0$ |
| Radical axis | $S_1 - S_2 = 0$ |
| Standard coaxial system | $x^2 + y^2 + 2gx + c = 0$ |
| Limiting points | $(\pm\sqrt{c}, 0)$ when $c > 0$ |
| Orthogonality condition | $2g_1g_2 + 2f_1f_2 = c_1 + c_2$ |
| Orthogonal coaxial system | $x^2 + y^2 + 2fy - c = 0$ |

---

## ❓ Quick Revision Questions

1. **Write the equation of family of circles through intersection of $x^2 + y^2 = 4$ and $x^2 + y^2 - 6x + 2y + 5 = 0$.**

2. **Find the radical axis of $x^2 + y^2 - 2x - 4y = 0$ and $x^2 + y^2 + 4x + 6y - 3 = 0$.**

3. **Find the circle through intersection of $x^2 + y^2 - 4 = 0$ and $y = x$, having smallest radius.**

4. **Check if $x^2 + y^2 - 6x - 4y + 9 = 0$ and $x^2 + y^2 + 2x + 4y - 11 = 0$ are orthogonal.**

5. **Find the limiting points of the coaxial system $x^2 + y^2 + 2gx + 4 = 0$.**

6. **Find the circle orthogonal to both $x^2 + y^2 = 4$ and $x^2 + y^2 - 6x - 8y + 21 = 0$ with center on line $x = 3$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $(x^2 + y^2 - 4) + \lambda(x^2 + y^2 - 6x + 2y + 5) = 0$
   **$(1+\lambda)(x^2 + y^2) - 6\lambda x + 2\lambda y + (5\lambda - 4) = 0$**

2. $S_1 - S_2 = 0$:
   $(x^2 + y^2 - 2x - 4y) - (x^2 + y^2 + 4x + 6y - 3) = 0$
   $-6x - 10y + 3 = 0$
   **$6x + 10y - 3 = 0$**

3. Family: $(x^2 + y^2 - 4) + \lambda(x - y) = 0$
   Center: $(-\frac{\lambda}{2}, \frac{\lambda}{2})$
   For smallest circle, center on chord $y = x$:
   $\frac{\lambda}{2} = -\frac{\lambda}{2}$ → $\lambda = 0$
   Wait, this gives original circle. Let me reconsider.
   For $y = x$: $-\frac{\lambda}{2} = \frac{\lambda}{2}$ is impossible unless $\lambda = 0$.
   Actually, center $(-\lambda/2, \lambda/2)$ lies on $y = x$ when $\lambda/2 = -\lambda/2$, i.e., $\lambda = 0$.
   Better approach: Intersection points: $2x^2 = 4$, $x = \pm\sqrt{2}$
   Points: $(\sqrt{2}, \sqrt{2})$, $(-\sqrt{2}, -\sqrt{2})$
   Smallest circle has these as diameter:
   Center: (0, 0), radius = $\sqrt{2}$
   **$x^2 + y^2 = 2$**

4. Circle 1: $g_1 = -3$, $f_1 = -2$, $c_1 = 9$
   Circle 2: $g_2 = 1$, $f_2 = 2$, $c_2 = -11$
   $2g_1g_2 + 2f_1f_2 = 2(-3)(1) + 2(-2)(2) = -6 - 8 = -14$
   $c_1 + c_2 = 9 - 11 = -2$
   $-14 \neq -2$
   **Not orthogonal**

5. Radius = $\sqrt{g^2 - 4} = 0$ when $g^2 = 4$, $g = \pm 2$
   Limiting points: **$(\pm 2, 0)$** i.e., **(2, 0)** and **(-2, 0)**

6. Let circle: $x^2 + y^2 - 6x + 2fy + c = 0$ (center at $(3, -f)$, on $x = 3$)
   Orthogonal to $x^2 + y^2 = 4$ ($g_2 = 0, f_2 = 0, c_2 = -4$):
   $2(-3)(0) + 2f(0) = c + (-4)$
   $c = 4$ ... (1)
   
   Orthogonal to $x^2 + y^2 - 6x - 8y + 21 = 0$ ($g_2 = -3, f_2 = -4, c_2 = 21$):
   $2(-3)(-3) + 2f(-4) = 4 + 21$
   $18 - 8f = 25$
   $f = -\frac{7}{8}$
   
   **$x^2 + y^2 - 6x - \frac{7}{4}y + 4 = 0$** or **$4x^2 + 4y^2 - 24x - 7y + 16 = 0$**

</details>

---

## 🎉 Unit 4 Complete!

Congratulations! You've completed the study of **Circles** in Coordinate Geometry. You've learned:

✅ Standard and general forms of circle equations  
✅ Parametric representation  
✅ Position of points and lines relative to circles  
✅ Tangent and normal equations  
✅ Family of circles and coaxial systems  
✅ Orthogonal circles  

---

## ⏭️ Navigation

| [← Previous: Equation of Normal](06-equation-of-normal.md) | [Unit 4 Home](README.md) | [Next Unit: Parabola →](../05-Parabola/README.md) |
|:----------------------------------------------------------|:------------------------:|--------------------------------------------------:|
