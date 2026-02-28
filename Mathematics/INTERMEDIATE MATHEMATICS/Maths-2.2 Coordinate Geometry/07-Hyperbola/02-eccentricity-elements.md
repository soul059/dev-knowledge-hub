# Chapter 7.2: Eccentricity and Elements of Hyperbola

## 📚 Chapter Overview

This chapter explores the **eccentricity** of a hyperbola and its various **elements** including foci, directrices, vertices, axes, and latus rectum. These elements completely define the geometry of a hyperbola.

---

## 📝 Eccentricity

### Definition

The **eccentricity (e)** of a hyperbola is defined as:

$$\boxed{e = \frac{c}{a}}$$

where:
- $c$ = distance from centre to focus
- $a$ = distance from centre to vertex

### Key Property

For a hyperbola: $c > a$, therefore:

$$\boxed{e > 1}$$

This distinguishes hyperbola from ellipse (where $0 < e < 1$).

### Relation with b

Since $c^2 = a^2 + b^2$:

$$e^2 = \frac{c^2}{a^2} = \frac{a^2 + b^2}{a^2} = 1 + \frac{b^2}{a^2}$$

$$\boxed{e = \sqrt{1 + \frac{b^2}{a^2}}}$$

Also:
$$\boxed{b^2 = a^2(e^2 - 1)}$$

---

## 📐 Elements of Hyperbola

For hyperbola $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$:

```
               y
               |
        \      |      /
         \     |     /
          \    |    /
    ........\..D₁../.........  x = -a/e (Directrix)
            \ .|. /
   F₁ ●------\-|-/------● F₂
 (-c,0)       \|/     (c,0)
        ___---V₁---___V₂___---___  x
              /|\
    ........./.|.\.........  x = a/e (Directrix)
           /  |  \
          /   |   \
         /    |    \
               |
               
   V₁(-a,0), V₂(a,0) = Vertices
   F₁(-c,0), F₂(c,0) = Foci
   D₁, D₂ = Directrices at x = ±a/e
```

### 1. Centre
The midpoint of the line segment joining foci: **(0, 0)**

### 2. Vertices
Points where hyperbola intersects transverse axis: **$(\pm a, 0)$**

### 3. Foci
Fixed points: **$(\pm c, 0)$** where $c = ae$

### 4. Transverse Axis
Line segment joining vertices, length = **2a**

### 5. Conjugate Axis  
Perpendicular to transverse axis through centre, length = **2b**

### 6. Principal Axes
Both the transverse and conjugate axes are called principal axes.

---

## 📝 Directrices

### Definition

The directrix is a line such that the ratio of distance from any point on hyperbola to focus and to directrix equals eccentricity.

For hyperbola $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$:

$$\boxed{\text{Directrices: } x = \pm\frac{a}{e}}$$

### Focus-Directrix Property

For any point $P$ on hyperbola:
$$\frac{PS}{PM} = e$$

where:
- $PS$ = distance to focus
- $PM$ = perpendicular distance to corresponding directrix

---

## 📝 Latus Rectum

### Definition

The **latus rectum** is a chord through focus perpendicular to transverse axis.

### Length of Latus Rectum

$$\boxed{\ell = \frac{2b^2}{a}}$$

### Derivation

At focus $F(c, 0)$, substitute $x = c$ in hyperbola equation:

$$\frac{c^2}{a^2} - \frac{y^2}{b^2} = 1$$

$$\frac{y^2}{b^2} = \frac{c^2}{a^2} - 1 = \frac{c^2 - a^2}{a^2} = \frac{b^2}{a^2}$$

$$y^2 = \frac{b^4}{a^2}$$

$$y = \pm\frac{b^2}{a}$$

Length = $2 \times \frac{b^2}{a} = \frac{2b^2}{a}$

### Ends of Latus Rectum

$$\boxed{\left(\pm ae, \pm\frac{b^2}{a}\right)}$$

The four endpoints are: $(ae, \frac{b^2}{a})$, $(ae, -\frac{b^2}{a})$, $(-ae, \frac{b^2}{a})$, $(-ae, -\frac{b^2}{a})$

---

## 📝 Focal Distances

For point $P(x_1, y_1)$ on hyperbola:

$$\boxed{SP = |ex_1 - a| \quad \text{and} \quad S'P = |ex_1 + a|}$$

For point on right branch ($x_1 > 0$):
- $SP = ex_1 - a$
- $S'P = ex_1 + a$

For point on left branch ($x_1 < 0$):
- $SP = -(ex_1 - a) = a - ex_1$
- $S'P = -(ex_1 + a)$ ... but this gives $|SP - S'P| = 2a$ ✓

### Key Property

$$\boxed{|SP - S'P| = 2a}$$

This is the defining property of hyperbola!

---

## 📐 Visual Summary

```
              y
              |
              |     Semi-conjugate axis = b
              |        ↕
        \     |       /
         \    |      /
          \   |     /
           \  |    /
            \ |   /
   -----------\---/------------- x
         ↔   V₁  V₂  ↔
        a/e       a/e
             ↔   ↔
             a   a
         ←─────→
            2a (Transverse axis)
             
   Directrix          Directrix
   x = -a/e           x = a/e
   
   Focus             Focus
   (-ae, 0)          (ae, 0)
```

---

## ✏️ Worked Examples

### Example 1: Find All Elements

**Problem**: For $\frac{x^2}{16} - \frac{y^2}{9} = 1$, find all elements.

**Solution**:

$a^2 = 16 \Rightarrow a = 4$

$b^2 = 9 \Rightarrow b = 3$

$c^2 = a^2 + b^2 = 16 + 9 = 25 \Rightarrow c = 5$

$e = \frac{c}{a} = \frac{5}{4}$

**Elements**:
- Centre: $(0, 0)$
- Vertices: $(\pm 4, 0)$
- Foci: $(\pm 5, 0)$
- Directrices: $x = \pm\frac{4}{5/4} = \pm\frac{16}{5}$
- Latus rectum: $\frac{2 \times 9}{4} = \frac{9}{2}$
- Transverse axis: $2a = 8$
- Conjugate axis: $2b = 6$

---

### Example 2: Eccentricity from Equation

**Problem**: Find eccentricity of $9x^2 - 16y^2 = 144$.

**Solution**:

Divide by 144:
$$\frac{x^2}{16} - \frac{y^2}{9} = 1$$

$a^2 = 16$, $b^2 = 9$

$e = \sqrt{1 + \frac{b^2}{a^2}} = \sqrt{1 + \frac{9}{16}} = \sqrt{\frac{25}{16}} = \frac{5}{4}$

**Answer**: $e = \frac{5}{4}$

---

### Example 3: Equation from Eccentricity

**Problem**: Find hyperbola with $e = 2$ and foci $(\pm 6, 0)$.

**Solution**:

$c = 6$ (from foci)

$e = \frac{c}{a} = 2$

$a = \frac{c}{2} = \frac{6}{2} = 3$

$b^2 = c^2 - a^2 = 36 - 9 = 27$

**Answer**: $\frac{x^2}{9} - \frac{y^2}{27} = 1$

---

### Example 4: Find Latus Rectum

**Problem**: Find ends of latus rectum for $\frac{x^2}{25} - \frac{y^2}{16} = 1$.

**Solution**:

$a = 5$, $b = 4$

$c = \sqrt{25 + 16} = \sqrt{41}$

$e = \frac{\sqrt{41}}{5}$

Ends of LR: $\left(\pm ae, \pm\frac{b^2}{a}\right) = \left(\pm\sqrt{41}, \pm\frac{16}{5}\right)$

**Answer**: $\left(\sqrt{41}, \frac{16}{5}\right)$, $\left(\sqrt{41}, -\frac{16}{5}\right)$, $\left(-\sqrt{41}, \frac{16}{5}\right)$, $\left(-\sqrt{41}, -\frac{16}{5}\right)$

---

### Example 5: Focal Distance

**Problem**: Find distances from $(5, \frac{16}{5})$ to the foci of $\frac{x^2}{16} - \frac{y^2}{9} = 1$.

**Solution**:

First verify point is on hyperbola:
$\frac{25}{16} - \frac{256/25}{9} = \frac{25}{16} - \frac{256}{225}$

Let's compute: $\frac{25 \times 225 - 256 \times 16}{16 \times 225} = \frac{5625 - 4096}{3600} = \frac{1529}{3600}$

Hmm, this doesn't equal 1. Let me recalculate for a point that IS on the hyperbola.

Take $x_1 = 5$: $\frac{25}{16} - \frac{y^2}{9} = 1$

$\frac{y^2}{9} = \frac{25}{16} - 1 = \frac{9}{16}$

$y^2 = \frac{81}{16}$, so $y = \pm\frac{9}{4}$

For point $(5, \frac{9}{4})$:

$e = \frac{5}{4}$, $a = 4$

$SP = ex_1 - a = \frac{5}{4} \times 5 - 4 = \frac{25}{4} - 4 = \frac{9}{4}$

$S'P = ex_1 + a = \frac{25}{4} + 4 = \frac{41}{4}$

Verify: $|S'P - SP| = \frac{41-9}{4} = \frac{32}{4} = 8 = 2a$ ✓

**Answer**: Focal distances are $\frac{9}{4}$ and $\frac{41}{4}$

---

### Example 6: Directrix Property

**Problem**: Verify focus-directrix property for vertex of $\frac{x^2}{9} - \frac{y^2}{7} = 1$.

**Solution**:

$a = 3$, $b^2 = 7$, $c = \sqrt{9+7} = 4$, $e = \frac{4}{3}$

Vertex: $(3, 0)$

Distance to focus $F(4, 0)$: $PF = |4-3| = 1$

Directrix: $x = \frac{a}{e} = \frac{3}{4/3} = \frac{9}{4}$

Distance to directrix: $PM = |3 - \frac{9}{4}| = |\frac{12-9}{4}| = \frac{3}{4}$

Check: $\frac{PF}{PM} = \frac{1}{3/4} = \frac{4}{3} = e$ ✓

**Verified!**

---

## 📊 Complete Elements Table

For $\frac{x^2}{a^2} - \frac{y^2}{b^2} = 1$ (Horizontal):

| Element | Formula | Location |
|---------|---------|----------|
| Centre | - | $(0, 0)$ |
| Vertices | - | $(\pm a, 0)$ |
| Foci | $c = ae$ | $(\pm ae, 0)$ |
| Directrices | $x = \pm\frac{a}{e}$ | Vertical lines |
| Eccentricity | $e = \frac{c}{a}$ | $e > 1$ |
| Latus Rectum | $\frac{2b^2}{a}$ | Chord through focus |
| Ends of LR | - | $(\pm ae, \pm\frac{b^2}{a})$ |
| Transverse axis | $2a$ | Along x-axis |
| Conjugate axis | $2b$ | Along y-axis |

---

## 🔄 Vertical Hyperbola Elements

For $\frac{y^2}{a^2} - \frac{x^2}{b^2} = 1$:

| Element | Location |
|---------|----------|
| Vertices | $(0, \pm a)$ |
| Foci | $(0, \pm ae)$ |
| Directrices | $y = \pm\frac{a}{e}$ |
| Ends of LR | $(\pm\frac{b^2}{a}, \pm ae)$ |

---

## 💡 Tips and Insights

> 💡 **Eccentricity Check**: For hyperbola, always $e > 1$. If you get $e \leq 1$, check your calculation!

> 💡 **Remember**: Directrix is INSIDE the hyperbola (between branches), closer to centre than vertices.

> ⚠️ **Sign Awareness**: Focal distance formula involves $|ex - a|$. Be careful with signs for left branch.

> 💡 **Comparison**: Latus rectum formula $\frac{2b^2}{a}$ is SAME for ellipse and hyperbola!

---

## 📋 Summary Table

| Property | Ellipse | Hyperbola |
|----------|---------|-----------|
| Relation | $c^2 = a^2 - b^2$ | $c^2 = a^2 + b^2$ |
| Eccentricity | $e = \frac{c}{a} < 1$ | $e = \frac{c}{a} > 1$ |
| $b^2$ formula | $b^2 = a^2(1-e^2)$ | $b^2 = a^2(e^2-1)$ |
| Directrix | $x = \pm\frac{a}{e}$ (outside) | $x = \pm\frac{a}{e}$ (inside) |
| Focal property | Sum = $2a$ | Difference = $2a$ |
| Latus Rectum | $\frac{2b^2}{a}$ | $\frac{2b^2}{a}$ |

---

## ❓ Quick Revision Questions

1. **What is the eccentricity of $\frac{x^2}{4} - \frac{y^2}{5} = 1$?**

2. **Find the directrices of $\frac{x^2}{25} - \frac{y^2}{11} = 1$.**

3. **For a hyperbola, if $a = 3$ and $e = 2$, find $b$.**

4. **Find the length of latus rectum for $\frac{x^2}{9} - \frac{y^2}{16} = 1$.**

5. **If the foci are $(0, \pm 5)$ and $e = \frac{5}{3}$, find the equation of hyperbola.**

6. **Find the focal distances of point $(4, \frac{7}{3})$ on $\frac{x^2}{9} - \frac{y^2}{7} = 1$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $e = \sqrt{1 + \frac{5}{4}} = \sqrt{\frac{9}{4}} = \frac{3}{2}$
   **$e = \frac{3}{2}$**

2. $a = 5$, $b^2 = 11$
   $c = \sqrt{25+11} = 6$
   $e = \frac{6}{5}$
   Directrices: $x = \pm\frac{5}{6/5} = \pm\frac{25}{6}$
   **$x = \pm\frac{25}{6}$**

3. $b^2 = a^2(e^2 - 1) = 9(4-1) = 27$
   **$b = 3\sqrt{3}$**

4. $a = 3$, $b^2 = 16$
   LR $= \frac{2 \times 16}{3} = \frac{32}{3}$
   **$\ell = \frac{32}{3}$**

5. Foci on y-axis → Vertical form
   $c = 5$, $e = \frac{5}{3}$
   $a = \frac{c}{e} = 3$
   $b^2 = c^2 - a^2 = 25 - 9 = 16$
   **$\frac{y^2}{9} - \frac{x^2}{16} = 1$**

6. First verify: $\frac{16}{9} - \frac{49/9}{7} = \frac{16}{9} - \frac{7}{9} = 1$ ✓
   $e = \frac{4}{3}$, $a = 3$
   $SP = ex_1 - a = \frac{4}{3} \times 4 - 3 = \frac{16}{3} - 3 = \frac{7}{3}$
   $S'P = ex_1 + a = \frac{16}{3} + 3 = \frac{25}{3}$
   **Focal distances: $\frac{7}{3}$ and $\frac{25}{3}$**

</details>

---

## ⏭️ Navigation

| [← Previous: Standard Equation](01-standard-equation.md) | [Unit 7 Home](README.md) | [Next: Asymptotes →](03-asymptotes.md) |
|:--------------------------------------------------------:|:------------------------:|---------------------------------------:|
