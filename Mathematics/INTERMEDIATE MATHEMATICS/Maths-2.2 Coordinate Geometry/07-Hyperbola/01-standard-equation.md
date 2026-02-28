# Chapter 7.1: Standard Equation of Hyperbola

## 📚 Chapter Overview

The **hyperbola** is a conic section defined by the property that the absolute difference of distances from any point on the curve to two fixed points (foci) is constant. This chapter derives the standard equation of hyperbola and explores its two forms.

---

## 📝 Definition of Hyperbola

> **Hyperbola**: The locus of a point that moves such that the **absolute difference** of its distances from two fixed points (foci) is constant.

```
           P(x,y)
            *
           /|\
          / | \
         /  |  \
        /   |   \
       /    |    \
  F₁ ●------+------● F₂
   (-c,0)   |    (c,0)
            |
            
   |PF₁ - PF₂| = 2a (constant)
```

### Key Terms

- **Foci (F₁, F₂)**: Fixed points at distance 2c apart
- **Transverse Axis**: Line segment between vertices (length 2a)
- **Conjugate Axis**: Perpendicular axis (length 2b)
- **Centre**: Midpoint of segment joining foci
- **Vertices**: Points where hyperbola crosses transverse axis

---

## 📐 Derivation of Standard Equation

### Setup
- Foci at $F_1(-c, 0)$ and $F_2(c, 0)$
- Centre at origin
- For point $P(x, y)$ on hyperbola: $|PF_1 - PF_2| = 2a$

### Step 1: Distance Expressions

$$PF_1 = \sqrt{(x+c)^2 + y^2}$$
$$PF_2 = \sqrt{(x-c)^2 + y^2}$$

### Step 2: Apply Definition

$$|PF_1 - PF_2| = 2a$$

Taking $PF_1 - PF_2 = 2a$ (for right branch, $x > 0$):

$$\sqrt{(x+c)^2 + y^2} - \sqrt{(x-c)^2 + y^2} = 2a$$

### Step 3: Rationalize

$$\sqrt{(x+c)^2 + y^2} = 2a + \sqrt{(x-c)^2 + y^2}$$

Squaring both sides:

$$(x+c)^2 + y^2 = 4a^2 + 4a\sqrt{(x-c)^2 + y^2} + (x-c)^2 + y^2$$

### Step 4: Simplify

$$x^2 + 2cx + c^2 + y^2 = 4a^2 + 4a\sqrt{(x-c)^2 + y^2} + x^2 - 2cx + c^2 + y^2$$

$$4cx = 4a^2 + 4a\sqrt{(x-c)^2 + y^2}$$

$$cx - a^2 = a\sqrt{(x-c)^2 + y^2}$$

### Step 5: Square Again

$$(cx - a^2)^2 = a^2[(x-c)^2 + y^2]$$

$$c^2x^2 - 2a^2cx + a^4 = a^2(x^2 - 2cx + c^2 + y^2)$$

$$c^2x^2 - 2a^2cx + a^4 = a^2x^2 - 2a^2cx + a^2c^2 + a^2y^2$$

$$c^2x^2 + a^4 = a^2x^2 + a^2c^2 + a^2y^2$$

$$c^2x^2 - a^2x^2 - a^2y^2 = a^2c^2 - a^4$$

$$(c^2 - a^2)x^2 - a^2y^2 = a^2(c^2 - a^2)$$

### Step 6: Define b²

Let $b^2 = c^2 - a^2$ (Note: $c > a$ for hyperbola, so $b^2 > 0$)

$$b^2x^2 - a^2y^2 = a^2b^2$$

Dividing by $a^2b^2$:

$$\boxed{\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1}$$

---

## ⚡ Standard Equations

### Form 1: Horizontal Transverse Axis

$$\boxed{\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1}$$

```
              y
              |
       \      |      /
        \     |     /
         \    |    /
          \   |   /
   ________\_-|-_/________ x
  (-a,0)    \ | /     (a,0)
             \|/
          ___/|\___
         /   | \   \
        /    |  \    \
       /     |   \     \
              |
              
   Transverse axis along x-axis
   Vertices: (±a, 0)
   Foci: (±c, 0) where c² = a² + b²
```

**Key Elements:**
- Centre: $(0, 0)$
- Vertices: $(\pm a, 0)$
- Foci: $(\pm c, 0)$ where $c^2 = a^2 + b^2$
- Transverse axis length: $2a$
- Conjugate axis length: $2b$

### Form 2: Vertical Transverse Axis

$$\boxed{\frac{y^2}{a^2} - \frac{x^2}{b^2} = 1}$$

```
              y
              |
          *   |   *
           *  |  *
            * | *
   __________\|/___________ x
             /|\
            * | *
           *  |  *
          *   |   *
              |
              
   Transverse axis along y-axis
   Vertices: (0, ±a)
   Foci: (0, ±c)
```

**Key Elements:**
- Centre: $(0, 0)$
- Vertices: $(0, \pm a)$
- Foci: $(0, \pm c)$ where $c^2 = a^2 + b^2$

---

## 🔑 Key Relation

For ANY hyperbola:

$$\boxed{c^2 = a^2 + b^2}$$

This is OPPOSITE to ellipse where $c^2 = a^2 - b^2$!

```
   HYPERBOLA              ELLIPSE
   
   c² = a² + b²          c² = a² - b²
   c > a                 c < a
   e > 1                 0 < e < 1
```

---

## ✏️ Worked Examples

### Example 1: Identify Type

**Problem**: For $\frac{x^2}{16} - \frac{y^2}{9} = 1$, find a, b, c and identify orientation.

**Solution**:

$a^2 = 16 \Rightarrow a = 4$

$b^2 = 9 \Rightarrow b = 3$

$c^2 = a^2 + b^2 = 16 + 9 = 25 \Rightarrow c = 5$

Since $x^2$ has positive coefficient: **Horizontal transverse axis**

- Vertices: $(\pm 4, 0)$
- Foci: $(\pm 5, 0)$

---

### Example 2: Vertical Hyperbola

**Problem**: For $\frac{y^2}{25} - \frac{x^2}{16} = 1$, find all elements.

**Solution**:

$a^2 = 25 \Rightarrow a = 5$ (coefficient of positive term)

$b^2 = 16 \Rightarrow b = 4$

$c^2 = 25 + 16 = 41 \Rightarrow c = \sqrt{41}$

**Orientation**: Vertical (since $y^2$ is positive)

- Vertices: $(0, \pm 5)$
- Foci: $(0, \pm\sqrt{41})$
- Transverse axis: $2a = 10$
- Conjugate axis: $2b = 8$

---

### Example 3: Find Equation from Elements

**Problem**: Find hyperbola with foci $(\pm 5, 0)$ and transverse axis length 6.

**Solution**:

Foci on x-axis → Horizontal form: $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$

Transverse axis = $2a = 6 \Rightarrow a = 3$

$c = 5$ (foci at $(\pm c, 0)$)

$b^2 = c^2 - a^2 = 25 - 9 = 16$

**Answer**: $\frac{x^2}{9} - \frac{y^2}{16} = 1$

---

### Example 4: From Vertices and Foci

**Problem**: Find hyperbola with vertices $(0, \pm 4)$ and foci $(0, \pm 5)$.

**Solution**:

Vertices and foci on y-axis → Vertical form: $\frac{y^2}{a^2} - \frac{x^2}{b^2} = 1$

$a = 4$ (vertices at $(0, \pm a)$)

$c = 5$ (foci at $(0, \pm c)$)

$b^2 = c^2 - a^2 = 25 - 16 = 9$

**Answer**: $\frac{y^2}{16} - \frac{x^2}{9} = 1$

---

### Example 5: Verify Point on Hyperbola

**Problem**: Check if $(5, 3)$ lies on $\frac{x^2}{16} - \frac{y^2}{9} = 1$.

**Solution**:

Substitute:
$$\frac{25}{16} - \frac{9}{9} = \frac{25}{16} - 1 = \frac{25-16}{16} = \frac{9}{16}$$

Since $\frac{9}{16} \neq 1$, point $(5, 3)$ does **NOT** lie on the hyperbola.

---

### Example 6: Equation from Definition

**Problem**: Find hyperbola where difference of distances from $(3, 0)$ and $(-3, 0)$ is 4.

**Solution**:

Foci: $F_1(-3, 0)$ and $F_2(3, 0)$, so $c = 3$

$|PF_1 - PF_2| = 2a = 4 \Rightarrow a = 2$

$b^2 = c^2 - a^2 = 9 - 4 = 5$

**Answer**: $\frac{x^2}{4} - \frac{y^2}{5} = 1$

---

## 📊 Comparison Table

| Property | Horizontal | Vertical |
|----------|------------|----------|
| Equation | $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$ | $\frac{y^2}{a^2} - \frac{x^2}{b^2} = 1$ |
| Transverse axis | x-axis | y-axis |
| Vertices | $(\pm a, 0)$ | $(0, \pm a)$ |
| Foci | $(\pm c, 0)$ | $(0, \pm c)$ |
| Relation | $c^2 = a^2 + b^2$ | $c^2 = a^2 + b^2$ |
| Opens towards | Left and right | Up and down |

---

## 📝 Conjugate Hyperbolas

Two hyperbolas are **conjugate** if the transverse axis of one is the conjugate axis of the other:

$$\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1 \quad \text{and} \quad \frac{y^2}{b^2} - \frac{x^2}{a^2} = 1$$

Or equivalently:

$$\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1 \quad \text{and} \quad \frac{x^2}{a^2} - \frac{y^2}{b^2} = -1$$

```
        y
        |      Hyperbola 1
     \  |  /   (x²/a² - y²/b² = 1)
      \ | /
   ____\|/____  x
       /|\
      / | \    Hyperbola 2
     /  |  \   (y²/b² - x²/a² = 1)
        |
        
   Conjugate hyperbolas share same asymptotes
```

---

## 💡 Tips and Insights

> 💡 **Identification**: If $x^2$ coefficient is positive → horizontal; if $y^2$ coefficient is positive → vertical.

> ⚠️ **Common Mistake**: Don't confuse with ellipse! For hyperbola: $c^2 = a^2 + b^2$ (NOT $a^2 - b^2$).

> 💡 **Always c > a**: In hyperbola, the distance to focus is always greater than distance to vertex.

> 💡 **The value 'a'**: Always corresponds to the POSITIVE term in the equation.

---

## 📋 Summary Table

| Element | Horizontal Hyperbola | Vertical Hyperbola |
|---------|---------------------|-------------------|
| Standard form | $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$ | $\frac{y^2}{a^2} - \frac{x^2}{b^2} = 1$ |
| Centre | $(0, 0)$ | $(0, 0)$ |
| Vertices | $(\pm a, 0)$ | $(0, \pm a)$ |
| Foci | $(\pm c, 0)$ | $(0, \pm c)$ |
| $c^2$ | $a^2 + b^2$ | $a^2 + b^2$ |
| Transverse axis | $2a$ along x | $2a$ along y |
| Conjugate axis | $2b$ along y | $2b$ along x |

---

## ❓ Quick Revision Questions

1. **What is the standard equation of hyperbola with transverse axis along x-axis?**

2. **For $\frac{x^2}{9} - \frac{y^2}{16} = 1$, find a, b, c and vertices.**

3. **Find the equation of hyperbola with foci $(0, \pm 6)$ and vertices $(0, \pm 4)$.**

4. **What is the relation between a, b, and c for a hyperbola?**

5. **Does the point $(4, 2\sqrt{3})$ lie on $\frac{x^2}{9} - \frac{y^2}{12} = 1$?**

6. **Write the conjugate of $\frac{x^2}{25} - \frac{y^2}{16} = 1$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$

2. $a^2 = 9 \Rightarrow a = 3$, $b^2 = 16 \Rightarrow b = 4$
   $c^2 = 9 + 16 = 25 \Rightarrow c = 5$
   Vertices: $(\pm 3, 0)$, Foci: $(\pm 5, 0)$

3. Foci on y-axis → Vertical form
   $a = 4$ (vertices), $c = 6$ (foci)
   $b^2 = 36 - 16 = 20$
   **$\frac{y^2}{16} - \frac{x^2}{20} = 1$**

4. $c^2 = a^2 + b^2$ (where $c$ = distance from center to focus)

5. Substitute: $\frac{16}{9} - \frac{12}{12} = \frac{16}{9} - 1 = \frac{7}{9} \neq 1$
   **No**, the point does NOT lie on the hyperbola.

6. Conjugate: **$\frac{y^2}{16} - \frac{x^2}{25} = 1$**
   Or equivalently: $\frac{x^2}{25} - \frac{y^2}{16} = -1$

</details>

---

## ⏭️ Navigation

| [← Unit 7 Home](README.md) | [Unit 7 Home](README.md) | [Next: Eccentricity and Elements →](02-eccentricity-elements.md) |
|:--------------------------:|:------------------------:|----------------------------------------------------------------:|
