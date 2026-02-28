# Chapter 7.5: Position of Point and Line Relative to Hyperbola

## 📚 Chapter Overview

This chapter explores how to determine whether a point lies inside, on, or outside a hyperbola, and the conditions for a line to be a tangent, secant, or non-intersecting line to the hyperbola.

---

## 📝 Position of a Point

For hyperbola $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$, define:

$$S(x, y) = \frac{x^2}{a^2} - \frac{y^2}{b^2} - 1$$

For point $P(x_1, y_1)$:

$$S_1 = \frac{x_1^2}{a^2} - \frac{y_1^2}{b^2} - 1$$

### Position Criteria

| Condition | Position |
|-----------|----------|
| $S_1 = 0$ | Point is **ON** the hyperbola |
| $S_1 > 0$ | Point is **OUTSIDE** the hyperbola |
| $S_1 < 0$ | Point is **INSIDE** the hyperbola (between branches) |

```
              y
              |
     INSIDE   |   INSIDE
      S₁ < 0  |   S₁ < 0
        \     |     /
         \    |    /
          \   |   /
   _________\_|_/___________ x
   OUTSIDE   /|\   OUTSIDE
   S₁ > 0   / | \  S₁ > 0
           /  |  \
          /   |   \
              |
              
   "Inside" = region between the two branches
   "Outside" = region containing the branches
```

### Important Note

Unlike ellipse/circle, "inside" for hyperbola means the region BETWEEN the two branches (the region containing the centre).

---

## 📐 Visual Interpretation

```
   Region Analysis for x²/a² - y²/b² = 1
   
              y
              |
              |         ↖ INSIDE (S₁ < 0)
     OUTSIDE  |    /    
      (S₁>0)  |   /     
        *     |  /  INSIDE (S₁ < 0)
         *    | /    
          *   |/   
   _________*-|--*________ x
            * |\ *
           *  | \*    OUTSIDE
          *   |  \     (S₁ > 0)
              |   \
     INSIDE   |    ↘ OUTSIDE
              |
```

---

## ✏️ Examples: Position of Point

### Example 1

**Problem**: Determine position of $(3, 0)$ relative to $\frac{x^2}{4} - \frac{y^2}{5} = 1$.

**Solution**:

$S_1 = \frac{9}{4} - \frac{0}{5} - 1 = \frac{9}{4} - 1 = \frac{5}{4} > 0$

**Answer**: Point is **OUTSIDE** (on the branch side)

---

### Example 2

**Problem**: Check position of $(0, 2)$ for $\frac{x^2}{9} - \frac{y^2}{4} = 1$.

**Solution**:

$S_1 = \frac{0}{9} - \frac{4}{4} - 1 = -1 - 1 = -2 < 0$

**Answer**: Point is **INSIDE** (between the branches)

---

### Example 3

**Problem**: Find values of k if $(k, 3)$ is inside $\frac{x^2}{25} - \frac{y^2}{9} = 1$.

**Solution**:

For inside: $S_1 < 0$

$\frac{k^2}{25} - \frac{9}{9} - 1 < 0$

$\frac{k^2}{25} - 2 < 0$

$k^2 < 50$

$-\sqrt{50} < k < \sqrt{50}$

$-5\sqrt{2} < k < 5\sqrt{2}$

**Answer**: $k \in (-5\sqrt{2}, 5\sqrt{2})$

---

## 📝 Position of a Line

For line $y = mx + c$ and hyperbola $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$:

### Finding Intersection

Substitute $y = mx + c$ into hyperbola:

$$\frac{x^2}{a^2} - \frac{(mx + c)^2}{b^2} = 1$$

$$b^2x^2 - a^2(m^2x^2 + 2mcx + c^2) = a^2b^2$$

$$(b^2 - a^2m^2)x^2 - 2a^2mcx - a^2(c^2 + b^2) = 0$$

### Discriminant Analysis

Let $D$ = discriminant of above quadratic:

$$D = 4a^4m^2c^2 + 4(b^2 - a^2m^2) \cdot a^2(c^2 + b^2)$$

$$= 4a^2[a^2m^2c^2 + (b^2 - a^2m^2)(c^2 + b^2)]$$

$$= 4a^2[a^2m^2c^2 + b^2c^2 + b^4 - a^2m^2c^2 - a^2m^2b^2]$$

$$= 4a^2[b^2c^2 + b^4 - a^2m^2b^2]$$

$$= 4a^2b^2[c^2 + b^2 - a^2m^2]$$

$$= 4a^2b^2[c^2 - (a^2m^2 - b^2)]$$

### Position Criteria

| Condition | Relationship |
|-----------|--------------|
| $c^2 > a^2m^2 - b^2$ | Line is **SECANT** (2 points) |
| $c^2 = a^2m^2 - b^2$ | Line is **TANGENT** (1 point) |
| $c^2 < a^2m^2 - b^2$ | Line **DOES NOT INTERSECT** |

---

## ⚡ Condition for Tangency

For line $y = mx + c$ to be tangent to $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$:

$$\boxed{c^2 = a^2m^2 - b^2}$$

$$\boxed{c = \pm\sqrt{a^2m^2 - b^2}}$$

### Important Constraint

For real tangent: $a^2m^2 - b^2 \geq 0$

$$|m| \geq \frac{b}{a}$$

This means tangent lines exist only for slopes with $|m| \geq \frac{b}{a}$ (steeper than asymptotes).

---

## 📐 Comparison with Ellipse

| Property | Ellipse | Hyperbola |
|----------|---------|-----------|
| Tangent condition | $c^2 = a^2m^2 + b^2$ | $c^2 = a^2m^2 - b^2$ |
| Valid for all slopes | Yes | Only $|m| \geq \frac{b}{a}$ |
| Asymptote slope | N/A | $\pm\frac{b}{a}$ (tangent at ∞) |

```
   Sign in tangent condition:
   
   ELLIPSE:     c² = a²m² + b²   (PLUS)
   HYPERBOLA:   c² = a²m² - b²   (MINUS)
```

---

## ✏️ Examples: Line and Hyperbola

### Example 4: Condition for Tangency

**Problem**: For what values of c is $y = 2x + c$ tangent to $\frac{x^2}{9} - \frac{y^2}{4} = 1$?

**Solution**:

$a^2 = 9$, $b^2 = 4$, $m = 2$

Condition: $c^2 = a^2m^2 - b^2 = 9 \times 4 - 4 = 32$

$c = \pm\sqrt{32} = \pm 4\sqrt{2}$

**Answer**: $c = \pm 4\sqrt{2}$

---

### Example 5: Check Line Type

**Problem**: Determine if $y = x + 5$ is tangent, secant, or non-intersecting for $\frac{x^2}{16} - \frac{y^2}{9} = 1$.

**Solution**:

$a^2 = 16$, $b^2 = 9$, $m = 1$, $c = 5$

$a^2m^2 - b^2 = 16 - 9 = 7$

Compare $c^2$ with $a^2m^2 - b^2$:

$c^2 = 25$, $a^2m^2 - b^2 = 7$

$25 > 7$

**Answer**: Line is **SECANT** (intersects at 2 points)

---

### Example 6: Valid Slope Range

**Problem**: Find the range of slopes for tangent lines to $\frac{x^2}{25} - \frac{y^2}{16} = 1$.

**Solution**:

$a = 5$, $b = 4$

Asymptote slope: $\pm\frac{b}{a} = \pm\frac{4}{5}$

For tangent: $|m| \geq \frac{b}{a}$

$|m| \geq \frac{4}{5}$

**Answer**: $m \leq -\frac{4}{5}$ or $m \geq \frac{4}{5}$

---

### Example 7: Finding Tangent Equation

**Problem**: Find tangent to $\frac{x^2}{4} - \frac{y^2}{9} = 1$ with slope 2.

**Solution**:

$a^2 = 4$, $b^2 = 9$, $m = 2$

Check: $\frac{b}{a} = \frac{3}{2}$, and $|2| > \frac{3}{2}$ ✓

$c = \pm\sqrt{a^2m^2 - b^2} = \pm\sqrt{16 - 9} = \pm\sqrt{7}$

**Answer**: $y = 2x + \sqrt{7}$ and $y = 2x - \sqrt{7}$

---

### Example 8: Line Parallel to Asymptote

**Problem**: Does $y = \frac{3}{4}x + 1$ touch $\frac{x^2}{16} - \frac{y^2}{9} = 1$?

**Solution**:

$a = 4$, $b = 3$

Asymptote slope: $\frac{b}{a} = \frac{3}{4}$

Given line has slope $m = \frac{3}{4}$ = asymptote slope!

For this slope: $c^2 = a^2m^2 - b^2 = 16 \times \frac{9}{16} - 9 = 9 - 9 = 0$

So tangent at this slope would be $y = \frac{3}{4}x + 0$ (the asymptote itself).

Since $c = 1 \neq 0$, the line $y = \frac{3}{4}x + 1$ is parallel to asymptote but NOT a tangent.

**Answer**: The line does NOT touch the hyperbola (it has only ONE intersection point, but that's at infinity - it's parallel to asymptote!)

---

## 📝 Special Cases

### 1. Horizontal Tangent (m = 0)

For $m = 0$: $c^2 = -b^2 < 0$

No real solution! **No horizontal tangent** exists for horizontal hyperbola.

(The hyperbola never has a horizontal tangent line.)

### 2. Vertical Tangent

Vertical tangent: $x = k$

Substituting: $\frac{k^2}{a^2} - \frac{y^2}{b^2} = 1$

For tangent: must touch at one point → at vertices $x = \pm a$

**Vertical tangents**: $x = \pm a$

### 3. Tangent at Asymptote Slope

When $|m| = \frac{b}{a}$: $c = 0$

The line $y = \pm\frac{b}{a}x$ is the asymptote (touches at infinity).

---

## 💡 Tips and Insights

> 💡 **Memory Aid**: Hyperbola uses MINUS in tangent condition ($c^2 = a^2m^2 - b^2$), ellipse uses PLUS.

> ⚠️ **Slope Restriction**: Not all slopes give tangents! Only $|m| \geq \frac{b}{a}$.

> 💡 **Inside/Outside**: For hyperbola, "inside" means BETWEEN branches, unlike ellipse where inside means within the curve.

> 💡 **Asymptote Check**: If a line is parallel to asymptote but not the asymptote itself, it intersects hyperbola at exactly one finite point (secant at infinity).

---

## 📋 Summary Table

| Condition | Point Position | Line Position |
|-----------|---------------|---------------|
| $S_1 = 0$ | ON hyperbola | - |
| $S_1 > 0$ | OUTSIDE (branch region) | - |
| $S_1 < 0$ | INSIDE (between branches) | - |
| $c^2 > a^2m^2 - b^2$ | - | SECANT |
| $c^2 = a^2m^2 - b^2$ | - | TANGENT |
| $c^2 < a^2m^2 - b^2$ | - | NO INTERSECTION |

### Tangent Line Formulas

| Form | Equation | Condition |
|------|----------|-----------|
| Slope form | $y = mx \pm \sqrt{a^2m^2 - b^2}$ | $|m| \geq \frac{b}{a}$ |
| At $(x_1, y_1)$ | $\frac{xx_1}{a^2} - \frac{yy_1}{b^2} = 1$ | Point on hyperbola |

---

## ❓ Quick Revision Questions

1. **Is $(2, 3)$ inside or outside $\frac{x^2}{9} - \frac{y^2}{16} = 1$?**

2. **Find condition for $y = 3x + c$ to be tangent to $\frac{x^2}{4} - \frac{y^2}{5} = 1$.**

3. **Is $y = \frac{1}{2}x + 1$ a tangent to $\frac{x^2}{16} - \frac{y^2}{9} = 1$?**

4. **Find tangent lines to $\frac{x^2}{25} - \frac{y^2}{9} = 1$ with slope 1.**

5. **For what slopes can we NOT draw tangent to $\frac{x^2}{4} - \frac{y^2}{9} = 1$?**

6. **Is there a horizontal tangent to $\frac{x^2}{16} - \frac{y^2}{9} = 1$?**

<details>
<summary><b>Click to see answers</b></summary>

1. $S_1 = \frac{4}{9} - \frac{9}{16} - 1 = \frac{64 - 81}{144} - 1 = -\frac{17}{144} - 1 < 0$
   **INSIDE** (between branches)

2. $a^2 = 4$, $b^2 = 5$, $m = 3$
   $c^2 = 4 \times 9 - 5 = 31$
   **$c = \pm\sqrt{31}$**

3. $a^2 = 16$, $b^2 = 9$, $m = \frac{1}{2}$
   Asymptote slope: $\frac{3}{4} = 0.75$
   Check: $|m| = 0.5 < 0.75$
   **NO** - slope too small for tangent

4. $a^2 = 25$, $b^2 = 9$, $m = 1$
   $c = \pm\sqrt{25 - 9} = \pm 4$
   **$y = x + 4$ and $y = x - 4$**

5. Asymptote slope: $\frac{b}{a} = \frac{3}{2}$
   No tangent for: $|m| < \frac{3}{2}$
   **$-\frac{3}{2} < m < \frac{3}{2}$**

6. For horizontal tangent ($m = 0$):
   $c^2 = 0 - 9 = -9 < 0$
   **NO** - no horizontal tangent exists

</details>

---

## ⏭️ Navigation

| [← Previous: Parametric Form](04-parametric-form.md) | [Unit 7 Home](README.md) | [Next: Equation of Tangent →](06-equation-of-tangent.md) |
|:---------------------------------------------------:|:------------------------:|--------------------------------------------------------:|
