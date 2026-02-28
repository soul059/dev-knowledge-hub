# Chapter 4.4: Position of Point and Line with Respect to Circle

## 📚 Chapter Overview

Understanding whether a point lies inside, on, or outside a circle, and how a line interacts with a circle (tangent, secant, or non-intersecting), is crucial for solving many geometry problems. This chapter develops these concepts systematically.

---

## 📝 Position of a Point

### Definition

For a circle $S: x^2 + y^2 + 2gx + 2fy + c = 0$ and a point $P(x_1, y_1)$:

Define: $S_1 = x_1^2 + y_1^2 + 2gx_1 + 2fy_1 + c$

> **Position of Point**:
>
> | Condition | Position |
> |-----------|----------|
> | $S_1 < 0$ | Point is **inside** the circle |
> | $S_1 = 0$ | Point is **on** the circle |
> | $S_1 > 0$ | Point is **outside** the circle |

```
        ┌─────────────────────┐
       /         ●            \
      /      (inside)          \
     │     S₁ < 0               │
     │                          │
     │          ●               │
     │       (center)           │
     │                          │
      \                        /
       \          ●           /
        \      (on S₁=0)     /
         └───────●──────────┘
              (outside)
               S₁ > 0
```

---

## 📐 Understanding the Condition

For circle with center C$(−g, −f)$ and radius $r = \sqrt{g^2 + f^2 - c}$:

Distance from C to P: $d = \sqrt{(x_1 + g)^2 + (y_1 + f)^2}$

$$d^2 = (x_1 + g)^2 + (y_1 + f)^2$$
$$= x_1^2 + 2gx_1 + g^2 + y_1^2 + 2fy_1 + f^2$$
$$= (x_1^2 + y_1^2 + 2gx_1 + 2fy_1 + c) + (g^2 + f^2 - c)$$
$$= S_1 + r^2$$

So: $S_1 = d^2 - r^2$

- $S_1 < 0$ means $d < r$ (point inside)
- $S_1 = 0$ means $d = r$ (point on circle)
- $S_1 > 0$ means $d > r$ (point outside)

---

## ✏️ Examples: Position of Point

### Example 1: Classifying Point Position

**Problem**: Determine the position of points (1, 1), (2, 3), and (4, 5) with respect to circle $x^2 + y^2 - 4x - 6y + 9 = 0$.

**Solution**:

For point (1, 1):
$S_1 = 1 + 1 - 4 - 6 + 9 = 1$ > 0 → **Outside**

For point (2, 3):
$S_1 = 4 + 9 - 8 - 18 + 9 = -4$ < 0 → **Inside**

For point (4, 5):
$S_1 = 16 + 25 - 16 - 30 + 9 = 4$ > 0 → **Outside**

---

### Example 2: Finding Region

**Problem**: For what values of k is point (k, k+1) inside circle $x^2 + y^2 = 25$?

**Solution**:

$S_1 = k^2 + (k+1)^2 - 25 < 0$
$k^2 + k^2 + 2k + 1 - 25 < 0$
$2k^2 + 2k - 24 < 0$
$k^2 + k - 12 < 0$
$(k + 4)(k - 3) < 0$

**Answer**: $-4 < k < 3$

---

## 📝 Position of a Line

### Line and Circle Intersection

For circle $S: x^2 + y^2 + 2gx + 2fy + c = 0$ and line $L: ax + by + c' = 0$:

The perpendicular distance from center $(-g, -f)$ to line is:
$$d = \frac{|-ag - bf + c'|}{\sqrt{a^2 + b^2}}$$

> **Position of Line**:
>
> | Condition | Relationship | Number of Intersection Points |
> |-----------|--------------|-------------------------------|
> | $d > r$ | Line is **outside** | 0 (no intersection) |
> | $d = r$ | Line is **tangent** | 1 (touches) |
> | $d < r$ | Line is **secant** | 2 (cuts) |

```
    ┌─────────────────────────────────────────────────────────┐
    │                                                         │
    │       d > r              d = r              d < r       │
    │                                                         │
    │    ──────────        ────────●         ───●───●───      │
    │                           /    \        /       \       │
    │      ●                   ●      ●      ●    ●    ●      │
    │     / \                  │      │      │    │    │      │
    │    │   │                 │   ●  │      │    ●    │      │
    │     \ /                  │      │      │         │      │
    │      ●                   ●      ●      ●         ●      │
    │                           \    /        \       /       │
    │   No intersection        ────●───        ───────        │
    │                         Tangent           Secant        │
    │                                                         │
    └─────────────────────────────────────────────────────────┘
```

---

## 📐 Finding Intersection Points

To find intersection of circle $x^2 + y^2 + 2gx + 2fy + c = 0$ and line $ax + by + c' = 0$:

**Method 1**: Substitution
- Express one variable from line equation
- Substitute in circle equation
- Solve the resulting quadratic

**Method 2**: Discriminant Analysis
The resulting quadratic has:
- 2 distinct roots → 2 intersection points
- 1 repeated root → 1 point (tangent)
- No real roots → no intersection

---

## ✏️ Examples: Line and Circle

### Example 3: Line-Circle Relationship

**Problem**: Determine if line $3x + 4y = 25$ is tangent, secant, or outside circle $x^2 + y^2 = 25$.

**Solution**:

Circle: Center (0, 0), Radius = 5

Distance from center to line:
$$d = \frac{|3(0) + 4(0) - 25|}{\sqrt{9 + 16}} = \frac{25}{5} = 5$$

Since $d = r = 5$, the line is a **tangent**.

---

### Example 4: Finding Intersection Points

**Problem**: Find the points of intersection of line $y = x + 1$ with circle $x^2 + y^2 = 5$.

**Solution**:

Substitute $y = x + 1$ in circle:
$$x^2 + (x + 1)^2 = 5$$
$$x^2 + x^2 + 2x + 1 = 5$$
$$2x^2 + 2x - 4 = 0$$
$$x^2 + x - 2 = 0$$
$$(x + 2)(x - 1) = 0$$

$x = -2$ → $y = -1$
$x = 1$ → $y = 2$

**Answer**: $(-2, -1)$ and $(1, 2)$

---

### Example 5: Condition for Tangency

**Problem**: Find k if line $y = kx + 3$ is tangent to circle $x^2 + y^2 = 9$.

**Solution**:

Circle: Center (0, 0), Radius = 3
Line: $kx - y + 3 = 0$

For tangent: $d = r$
$$\frac{|0 - 0 + 3|}{\sqrt{k^2 + 1}} = 3$$

$$\frac{3}{\sqrt{k^2 + 1}} = 3$$

$$\sqrt{k^2 + 1} = 1$$

$$k^2 + 1 = 1$$

$$k = 0$$

**Answer**: $k = 0$ (line is $y = 3$)

---

## 📝 Length of Chord

When a line intersects a circle at two points, the **chord length** can be found using:

> **Chord Length Formula**:
>
> $$\boxed{\text{Chord length} = 2\sqrt{r^2 - d^2}}$$
>
> where $d$ = perpendicular distance from center to line

```
           ●A
          /│\
         / │ \
        /  │d \ r
       /   │   \
    ──●────●────●──  Line
      B    M    
          (foot of ⊥)
          
    Chord AB = 2·AM = 2√(r² - d²)
```

---

### Example 6: Finding Chord Length

**Problem**: Find the length of chord cut by circle $x^2 + y^2 = 25$ on line $3x + 4y = 15$.

**Solution**:

Circle: Center (0, 0), Radius = 5

Distance from center to line:
$$d = \frac{|15|}{\sqrt{9 + 16}} = \frac{15}{5} = 3$$

Chord length = $2\sqrt{25 - 9} = 2\sqrt{16} = 8$

**Answer**: 8 units

---

## 📝 Length of Tangent from External Point

For a point P$(x_1, y_1)$ outside circle $x^2 + y^2 + 2gx + 2fy + c = 0$:

> **Length of Tangent**:
>
> $$\boxed{L = \sqrt{x_1^2 + y_1^2 + 2gx_1 + 2fy_1 + c} = \sqrt{S_1}}$$

```
           P(x₁,y₁)
          /│
         / │
        /  │ L = length of tangent
       /   │
      /    │
     ●─────●
     T  90°│  Circle
           C
```

**Derivation**: In right triangle PTC:
$PT^2 = PC^2 - TC^2 = d^2 - r^2 = S_1$

---

### Example 7: Length of Tangent

**Problem**: Find the length of tangent from point (5, 4) to circle $x^2 + y^2 - 2x - 4y = 0$.

**Solution**:

$S_1 = 25 + 16 - 10 - 16 = 15$

Length = $\sqrt{15}$

**Answer**: $\sqrt{15}$ units

---

## 📝 Power of a Point

The expression $S_1 = x_1^2 + y_1^2 + 2gx_1 + 2fy_1 + c$ is called the **power of point** P$(x_1, y_1)$ with respect to the circle.

**Properties**:
- Power = (Length of tangent)² for external points
- Power = $-$(Product of distances to intersection points) for internal points
- Power is negative inside, zero on, and positive outside the circle

---

## 💡 Tips and Insights

> 💡 **Quick Check**: Substitute point in circle equation. Sign tells position.

> 💡 **Tangent Condition**: Distance from center to line equals radius.

> ⚠️ **Direction Matters**: When finding distance, use absolute value in numerator.

> 💡 **Chord Formula**: Works only when $d < r$ (line actually cuts circle).

---

## 🌍 Real-World Applications

1. **Collision Detection**: Determining if objects intersect
2. **GPS**: Checking if a point is within a circular boundary
3. **Engineering**: Clearance calculations
4. **Physics**: Interference patterns
5. **Navigation**: Entry/exit points on circular paths

---

## 📋 Summary Table

| Concept | Formula/Condition |
|---------|-------------------|
| Point inside | $S_1 < 0$ |
| Point on circle | $S_1 = 0$ |
| Point outside | $S_1 > 0$ |
| Line doesn't meet circle | $d > r$ |
| Line tangent to circle | $d = r$ |
| Line secant to circle | $d < r$ |
| Chord length | $2\sqrt{r^2 - d^2}$ |
| Tangent length | $\sqrt{S_1}$ |
| Power of point | $S_1 = d^2 - r^2$ |

---

## ❓ Quick Revision Questions

1. **Is point (3, 4) inside, on, or outside circle $x^2 + y^2 = 20$?**

2. **Find the position of origin with respect to $x^2 + y^2 - 6x + 8y + 9 = 0$.**

3. **Determine if line $x + y = 10$ is tangent, secant, or outside $x^2 + y^2 = 50$.**

4. **Find chord length cut by $x^2 + y^2 = 100$ on line $4x + 3y = 30$.**

5. **Find length of tangent from (7, 1) to circle $x^2 + y^2 = 25$.**

6. **For what value of k is line $3x + 4y = k$ tangent to circle $x^2 + y^2 = 4$?**

<details>
<summary><b>Click to see answers</b></summary>

1. $S_1 = 9 + 16 - 20 = 5 > 0$ → **Outside**

2. $S_1 = 0 + 0 - 0 + 0 + 9 = 9 > 0$ → **Outside**

3. Center (0,0), $r = 5\sqrt{2}$
   $d = \frac{10}{\sqrt{2}} = 5\sqrt{2} = r$ → **Tangent**

4. $d = \frac{30}{5} = 6$, $r = 10$
   Chord = $2\sqrt{100 - 36} = 2 \times 8 = 16$ units

5. $S_1 = 49 + 1 - 25 = 25$
   Length = $\sqrt{25} = 5$ units

6. For tangent: $\frac{|k|}{5} = 2$
   $|k| = 10$
   **$k = \pm 10$**

</details>

---

## ⏭️ Navigation

| [← Previous: Parametric Form](03-parametric-form.md) | [Next: Equation of Tangent →](05-equation-of-tangent.md) |
|:----------------------------------------------------|--------------------------------------------------------:|
