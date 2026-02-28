# Chapter 5.2: Elements of Parabola

## 📚 Chapter Overview

A parabola has several important elements that define its geometry. Understanding these elements—vertex, focus, directrix, axis, latus rectum, and focal distance—is crucial for solving problems and applying parabola properties.

---

## 📝 Complete Terminology

### Visual Overview

```
                    Directrix
                       │
                       │     Latus Rectum (LR)
                       │        │
                       │     L●─┼─●L' ← Ends of LR
                       │      │ │ │
                       │       \│/
              ─────────┼────────●F───────●P──────────► Axis
                       │       /│\        (x,y)
                       │      │ │ │
                       │     L₁●─┼─●L₁'
                       │        │
                       │←── a ─→│←── a ──→│
                       │        V
                       │     Vertex
                       │
    Focal Distance (FP) = a + x₁ for point P(x₁, y₁)
```

---

## ⚡ Key Elements

### 1. Focus (F)

The **focus** is the fixed point from which all points on the parabola are equidistant to the directrix.

| Parabola | Focus |
|----------|-------|
| $y^2 = 4ax$ | $(a, 0)$ |
| $y^2 = -4ax$ | $(-a, 0)$ |
| $x^2 = 4ay$ | $(0, a)$ |
| $x^2 = -4ay$ | $(0, -a)$ |

---

### 2. Directrix

The **directrix** is the fixed line from which all points on the parabola are equidistant to the focus.

| Parabola | Directrix |
|----------|-----------|
| $y^2 = 4ax$ | $x = -a$ |
| $y^2 = -4ax$ | $x = a$ |
| $x^2 = 4ay$ | $y = -a$ |
| $x^2 = -4ay$ | $y = a$ |

---

### 3. Vertex (V)

The **vertex** is the point where the parabola meets its axis. It's the "turning point" and the point closest to the directrix.

- For standard parabolas: Vertex is at **origin (0, 0)**
- For shifted parabolas $(y-k)^2 = 4a(x-h)$: Vertex is at **(h, k)**

> **Property**: Vertex is the midpoint of focus and foot of directrix.

---

### 4. Axis of Symmetry

The **axis** (or axis of symmetry) is the line passing through focus and vertex, perpendicular to the directrix.

| Parabola | Axis |
|----------|------|
| $y^2 = 4ax$ or $y^2 = -4ax$ | $y = 0$ (x-axis) |
| $x^2 = 4ay$ or $x^2 = -4ay$ | $x = 0$ (y-axis) |

The parabola is symmetric about its axis.

---

### 5. Latus Rectum (LR)

The **latus rectum** is the chord passing through the focus and perpendicular to the axis.

```
                   L●─────────────●L'
                    │      │      │
                    │      ●F     │
                    │      │      │
        ────────────┴──────│──────┴──────
                           V
                           
    Latus Rectum = LL' through focus F
    Length = 4a
```

#### Length of Latus Rectum

For parabola $y^2 = 4ax$:

At focus $(a, 0)$: $y^2 = 4a \cdot a = 4a^2$

So $y = \pm 2a$

End points of LR: $(a, 2a)$ and $(a, -2a)$

$$\boxed{\text{Length of Latus Rectum} = 4a}$$

---

### 6. Semi-Latus Rectum

Half of the latus rectum = $\frac{4a}{2} = 2a$

This equals the distance from focus to either end of the latus rectum.

---

### 7. Focal Distance (Focal Length)

The **focal distance** of a point P$(x_1, y_1)$ on the parabola is the distance from P to the focus.

For $y^2 = 4ax$:

$$\boxed{\text{Focal Distance} = a + x_1}$$

#### Derivation

For point P$(x_1, y_1)$ on $y^2 = 4ax$:

By definition: PF = PM (distance to directrix)

PM = distance from $(x_1, y_1)$ to line $x = -a$
PM = $x_1 - (-a) = x_1 + a$

Therefore: $\boxed{PF = a + x_1}$

---

## 📋 Complete Element Table

For parabola $y^2 = 4ax$:

| Element | Symbol/Location | Value/Formula |
|---------|-----------------|---------------|
| Vertex | V | $(0, 0)$ |
| Focus | F | $(a, 0)$ |
| Directrix | | $x = -a$ |
| Axis | | $y = 0$ |
| Eccentricity | e | 1 |
| Latus rectum length | LR | $4a$ |
| Semi-latus rectum | | $2a$ |
| Ends of LR | L, L' | $(a, 2a)$, $(a, -2a)$ |
| Focal distance of $(x_1, y_1)$ | | $a + x_1$ |

---

## ✏️ Worked Examples

### Example 1: Finding All Elements

**Problem**: Find all elements of parabola $y^2 = 20x$.

**Solution**:

Comparing with $y^2 = 4ax$: $4a = 20$, so $a = 5$

| Element | Value |
|---------|-------|
| Vertex | $(0, 0)$ |
| Focus | $(5, 0)$ |
| Directrix | $x = -5$ |
| Axis | $y = 0$ |
| Latus rectum | 20 |
| Ends of LR | $(5, 10)$ and $(5, -10)$ |

---

### Example 2: Focal Distance

**Problem**: Find the focal distance of point $(4, 8)$ on parabola $y^2 = 16x$.

**Solution**:

$4a = 16$, so $a = 4$

Verify point is on parabola: $8^2 = 64 = 16 \times 4$ ✓

Focal distance = $a + x_1 = 4 + 4 = 8$

**Answer**: Focal distance = 8

---

### Example 3: From Latus Rectum

**Problem**: Find the equation of parabola with vertex at origin, axis along x-axis, and latus rectum = 6.

**Solution**:

Latus rectum = $4a = 6$, so $a = \frac{3}{2}$

Axis along x-axis → Form is $y^2 = 4ax$ or $y^2 = -4ax$

If opens right: $y^2 = 4 \times \frac{3}{2} \times x = 6x$

If opens left: $y^2 = -6x$

**Answer**: $y^2 = 6x$ or $y^2 = -6x$

---

### Example 4: Finding 'a' from Ends of LR

**Problem**: The ends of latus rectum of a parabola are $(3, 6)$ and $(3, -6)$. Find the equation.

**Solution**:

Ends of LR are symmetric about axis, so focus is midpoint:
Focus = $(3, 0)$

Distance from focus to end = $|6 - 0| = 6 = 2a$, so $a = 3$

Since focus is at $(3, 0)$ with $a = 3$:
- Vertex is at $(0, 0)$
- Opens right

Equation: $y^2 = 4(3)x = 12x$

**Answer**: $y^2 = 12x$

---

### Example 5: Point with Equal Focal Distances

**Problem**: Find the point on $y^2 = 8x$ whose focal distance is 6.

**Solution**:

$4a = 8$, so $a = 2$

Focal distance = $a + x_1 = 6$
$2 + x_1 = 6$
$x_1 = 4$

From parabola: $y^2 = 8 \times 4 = 32$
$y = \pm 4\sqrt{2}$

**Answer**: Points are $(4, 4\sqrt{2})$ and $(4, -4\sqrt{2})$

---

### Example 6: Shifted Parabola Elements

**Problem**: Find the vertex, focus, directrix, and latus rectum of $(y-3)^2 = -12(x-2)$.

**Solution**:

Form: $(y-k)^2 = -4a(x-h)$ → Opens left

Vertex: $(h, k) = (2, 3)$

$4a = 12$, so $a = 3$

Focus: $(h-a, k) = (2-3, 3) = (-1, 3)$

Directrix: $x = h + a = 2 + 3 = 5$

Latus rectum length = $4a = 12$

**Answer**: Vertex $(2, 3)$, Focus $(-1, 3)$, Directrix $x = 5$, LR = 12

---

## 📝 Double Ordinate

A **double ordinate** is any chord perpendicular to the axis.

```
              P●──────────────●P'
               │              │
               │              │
    ───────────┼──────────────┼───────►
               │              │
               │              │
              Q●──────────────●Q'
               
    PP' and QQ' are double ordinates
    Latus rectum is a special double ordinate (through focus)
```

For double ordinate at $x = x_1$:
- Points: $(x_1, 2\sqrt{ax_1})$ and $(x_1, -2\sqrt{ax_1})$
- Length = $4\sqrt{ax_1}$

---

## 📐 Important Relationships

### 1. Distance Relationships

```
    ┌────────────────────────────────────────────────────────┐
    │          KEY DISTANCE RELATIONSHIPS                    │
    ├────────────────────────────────────────────────────────┤
    │                                                        │
    │   Distance               Formula                       │
    │   ────────               ───────                       │
    │   Vertex to Focus        a                             │
    │   Vertex to Directrix    a                             │
    │   Focus to Directrix     2a                            │
    │   Focal distance of P    a + x₁ (for y² = 4ax)         │
    │   End of LR to Vertex    √(a² + 4a²) = a√5             │
    │                                                        │
    └────────────────────────────────────────────────────────┘
```

### 2. Parametric Representation

Any point on $y^2 = 4ax$ can be written as:

$$\boxed{(at^2, 2at)}$$

where t is the parameter.

Verification: $(2at)^2 = 4a^2t^2 = 4a \cdot at^2$ ✓

---

## 💡 Tips and Insights

> 💡 **Focal Distance Shortcut**: For $y^2 = 4ax$, focal distance = $a + x$, not $\sqrt{...}$.

> 💡 **LR Property**: Semi-latus rectum (2a) is the y-coordinate of ends of latus rectum for $y^2 = 4ax$.

> ⚠️ **Sign Matters**: For $y^2 = -4ax$, focus is at $(-a, 0)$, so focal distance = $a - x_1$.

> 💡 **Vertex = Midpoint**: Vertex is always the midpoint of the segment from focus to foot of perpendicular on directrix.

---

## 🌍 Real-World Applications

1. **Antenna Design**: Focus placement for satellite dishes
2. **Reflector Telescopes**: Positioning of detectors at focus
3. **Acoustic Mirrors**: Whispering galleries use parabolic shapes
4. **Solar Energy**: Focal length determines collector design
5. **Headlight Design**: Bulb positioning at focus

---

## 📋 Summary Table

| Element | $y^2 = 4ax$ | $y^2 = -4ax$ | $x^2 = 4ay$ | $x^2 = -4ay$ |
|---------|-------------|--------------|-------------|--------------|
| Opens | Right | Left | Up | Down |
| Vertex | $(0,0)$ | $(0,0)$ | $(0,0)$ | $(0,0)$ |
| Focus | $(a,0)$ | $(-a,0)$ | $(0,a)$ | $(0,-a)$ |
| Directrix | $x=-a$ | $x=a$ | $y=-a$ | $y=a$ |
| Axis | y=0 | y=0 | x=0 | x=0 |
| LR | $4a$ | $4a$ | $4a$ | $4a$ |
| Focal dist. | $a+x_1$ | $a-x_1$ | $a+y_1$ | $a-y_1$ |

---

## ❓ Quick Revision Questions

1. **Find the latus rectum of $y^2 = 24x$.**

2. **Find the ends of latus rectum of $x^2 = 8y$.**

3. **The focal distance of a point on $y^2 = 12x$ is 5. Find the point.**

4. **Find the double ordinate of $y^2 = 8x$ at $x = 2$.**

5. **For parabola $(x-1)^2 = -8(y+2)$, find vertex, focus, and directrix.**

6. **If focus is $(0, -4)$ and vertex is at origin, find the equation of parabola.**

<details>
<summary><b>Click to see answers</b></summary>

1. $4a = 24$ → $a = 6$
   **Latus rectum = $4a = 24$**

2. $x^2 = 8y$ → $4a = 8$ → $a = 2$
   Focus at $(0, 2)$
   At focus: $x^2 = 8(2) = 16$ → $x = \pm 4$
   **Ends of LR: $(4, 2)$ and $(-4, 2)$**

3. $4a = 12$ → $a = 3$
   Focal distance = $a + x_1 = 5$
   $x_1 = 2$
   $y^2 = 12(2) = 24$ → $y = \pm 2\sqrt{6}$
   **Points: $(2, 2\sqrt{6})$ and $(2, -2\sqrt{6})$**

4. At $x = 2$: $y^2 = 8(2) = 16$ → $y = \pm 4$
   Points: $(2, 4)$ and $(2, -4)$
   **Length of double ordinate = 8**

5. Form: $(x-h)^2 = -4a(y-k)$ → Opens down
   Vertex: $(1, -2)$
   $4a = 8$ → $a = 2$
   Focus: $(1, -2-2) = (1, -4)$
   **Vertex $(1, -2)$, Focus $(1, -4)$, Directrix $y = 0$**

6. Focus $(0, -4)$ is below vertex → Opens down
   $a = 4$
   Form: $x^2 = -4ay$
   **$x^2 = -16y$**

</details>

---

## ⏭️ Navigation

| [← Previous: Standard Equation](01-standard-equation.md) | [Next: Focal Chord →](03-focal-chord.md) |
|:---------------------------------------------------------|------------------------------------------:|
