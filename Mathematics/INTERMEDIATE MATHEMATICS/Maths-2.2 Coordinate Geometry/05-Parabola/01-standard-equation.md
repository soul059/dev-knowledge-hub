# Chapter 5.1: Standard Equation of Parabola

## 📚 Chapter Overview

The **parabola** is a conic section defined as the locus of points equidistant from a fixed point (focus) and a fixed line (directrix). This chapter derives the standard equations and explores the four orientations of parabolas.

---

## 📝 Definition of Parabola

> A **parabola** is the locus of a point P that moves such that its distance from a fixed point F (called **focus**) is equal to its distance from a fixed line (called **directrix**).

$$\boxed{PF = PM}$$

where PM is the perpendicular distance from P to the directrix.

```
         Directrix
            │
            │
            │      P●──────────── PM (perpendicular distance)
            │      /│
            │     / │
            │    /  │
            │   /   │
         ───┼──●F   │
            │   \   │
            │    \  │
            │     \ │
            │      \│
            │       ●──────────── 
            │
            
    For every point P on parabola: PF = PM
```

### Eccentricity

The ratio $\frac{PF}{PM} = e$ is called eccentricity.

For a parabola: $\boxed{e = 1}$

---

## 📐 Derivation: Standard Equation $y^2 = 4ax$

### Setup

Let:
- Focus F be at $(a, 0)$ where $a > 0$
- Directrix be the line $x = -a$
- P$(x, y)$ be any point on the parabola

```
      Directrix                    y
      x = -a                       │
         │                         │
         │                         │      P(x,y)
         │                         │     ●
    ─────│─────────────────────────●─────│─────► x
         │                       F(a,0)  │
         │                               │
         │                               │
         │←─────── a ──────→│←──── a ───→│
         
    Vertex V at origin (0,0)
    Focus at (a,0), Directrix at x = -a
```

### Derivation

Distance from P to focus F:
$$PF = \sqrt{(x-a)^2 + y^2}$$

Distance from P to directrix $x = -a$:
$$PM = |x - (-a)| = |x + a| = x + a$$ (since x ≥ 0 for points on this parabola)

By definition: $PF = PM$

$$\sqrt{(x-a)^2 + y^2} = x + a$$

Squaring both sides:
$$(x-a)^2 + y^2 = (x+a)^2$$

$$x^2 - 2ax + a^2 + y^2 = x^2 + 2ax + a^2$$

$$y^2 = 4ax$$

$$\boxed{y^2 = 4ax}$$

This is the **standard equation of parabola** opening to the right.

---

## ⚡ Four Standard Forms

Depending on the orientation, there are four standard forms:

### 1. Opening Right: $y^2 = 4ax$ (a > 0)

```
                 │
                 │        ═══════
                 │      /
                 │    /
       ──────────●───●F─────────►
                V│    \
                 │      \
                 │        ═══════
                 │
                 
    Focus: (a, 0)    Directrix: x = -a
```

| Element | Value |
|---------|-------|
| Vertex | (0, 0) |
| Focus | (a, 0) |
| Directrix | x = -a |
| Axis | y = 0 (x-axis) |

---

### 2. Opening Left: $y^2 = -4ax$ (a > 0)

```
                      │
           ═══════    │
                 \    │
                  \   │
        ◄──────F●────●──────────
                  /  V│
                 /    │
           ═══════    │
                      │
                      
    Focus: (-a, 0)    Directrix: x = a
```

| Element | Value |
|---------|-------|
| Vertex | (0, 0) |
| Focus | (-a, 0) |
| Directrix | x = a |
| Axis | y = 0 (x-axis) |

---

### 3. Opening Upward: $x^2 = 4ay$ (a > 0)

```
                 ▲
                 │
          ═══════│═══════
         /       │       \
        /        │        \
       /         ●F        \
      │          │          │
      ─────────────────────────
                V●
                 │
    Focus: (0, a)    Directrix: y = -a
```

| Element | Value |
|---------|-------|
| Vertex | (0, 0) |
| Focus | (0, a) |
| Directrix | y = -a |
| Axis | x = 0 (y-axis) |

---

### 4. Opening Downward: $x^2 = -4ay$ (a > 0)

```
                 │
                V●
      ─────────────────────────
      │          │          │
       \         ●F        /
        \        │        /
         \       │       /
          ═══════│═══════
                 │
                 ▼
                 
    Focus: (0, -a)    Directrix: y = a
```

| Element | Value |
|---------|-------|
| Vertex | (0, 0) |
| Focus | (0, -a) |
| Directrix | y = a |
| Axis | x = 0 (y-axis) |

---

## 📋 Complete Reference Table

| Form | Equation | Opens | Focus | Directrix | Axis |
|------|----------|-------|-------|-----------|------|
| 1 | $y^2 = 4ax$ | Right | $(a, 0)$ | $x = -a$ | $y = 0$ |
| 2 | $y^2 = -4ax$ | Left | $(-a, 0)$ | $x = a$ | $y = 0$ |
| 3 | $x^2 = 4ay$ | Up | $(0, a)$ | $y = -a$ | $x = 0$ |
| 4 | $x^2 = -4ay$ | Down | $(0, -a)$ | $y = a$ | $x = 0$ |

---

## 📝 Identifying Form from Equation

> **Quick Rules**:
> - If $y^2$ term: Horizontal axis (opens right or left)
> - If $x^2$ term: Vertical axis (opens up or down)
> - Positive coefficient: Opens positive direction (right/up)
> - Negative coefficient: Opens negative direction (left/down)

---

## ✏️ Worked Examples

### Example 1: Standard to Properties

**Problem**: Find the focus and directrix of $y^2 = 12x$.

**Solution**:

Comparing with $y^2 = 4ax$:
$4a = 12$
$a = 3$

Focus: $(a, 0) = (3, 0)$

Directrix: $x = -a$ → $x = -3$

**Answer**: Focus $(3, 0)$, Directrix $x = -3$

---

### Example 2: Negative Coefficient

**Problem**: Find the focus and directrix of $y^2 = -8x$.

**Solution**:

This is of form $y^2 = -4ax$ (opens left).
$4a = 8$
$a = 2$

Focus: $(-a, 0) = (-2, 0)$

Directrix: $x = a$ → $x = 2$

**Answer**: Focus $(-2, 0)$, Directrix $x = 2$

---

### Example 3: x² Form

**Problem**: Find the focus and directrix of $x^2 = 20y$.

**Solution**:

Comparing with $x^2 = 4ay$ (opens up):
$4a = 20$
$a = 5$

Focus: $(0, a) = (0, 5)$

Directrix: $y = -a$ → $y = -5$

**Answer**: Focus $(0, 5)$, Directrix $y = -5$

---

### Example 4: From Focus to Equation

**Problem**: Find the equation of parabola with vertex at origin and focus at $(-4, 0)$.

**Solution**:

Focus is at $(-4, 0)$, which is on the negative x-axis.
So parabola opens left.

Form: $y^2 = -4ax$

Here $a = 4$ (distance from vertex to focus).

Equation: $y^2 = -4(4)x = -16x$

**Answer**: $y^2 = -16x$

---

### Example 5: From Directrix to Equation

**Problem**: Find the equation of parabola with vertex at origin and directrix $y = 6$.

**Solution**:

Directrix $y = 6$ is above the origin.
So parabola opens downward (focus below vertex).

Form: $x^2 = -4ay$

Distance from vertex to directrix = 6, so $a = 6$.

Equation: $x^2 = -4(6)y = -24y$

**Answer**: $x^2 = -24y$

---

### Example 6: Finding 'a' from a Point

**Problem**: The parabola $y^2 = 4ax$ passes through $(2, 4)$. Find $a$.

**Solution**:

Substituting $(2, 4)$ into $y^2 = 4ax$:
$16 = 4a \cdot 2$
$16 = 8a$
$a = 2$

**Answer**: $a = 2$, so parabola is $y^2 = 8x$

---

## 📝 General Vertex Form

When vertex is at $(h, k)$ instead of origin:

| Original | Shifted (Vertex at (h,k)) |
|----------|---------------------------|
| $y^2 = 4ax$ | $(y-k)^2 = 4a(x-h)$ |
| $y^2 = -4ax$ | $(y-k)^2 = -4a(x-h)$ |
| $x^2 = 4ay$ | $(x-h)^2 = 4a(y-k)$ |
| $x^2 = -4ay$ | $(x-h)^2 = -4a(y-k)$ |

### Example 7: Shifted Vertex

**Problem**: Find the vertex, focus, and directrix of $(y-2)^2 = 8(x+3)$.

**Solution**:

Comparing with $(y-k)^2 = 4a(x-h)$:
- Vertex $(h, k) = (-3, 2)$
- $4a = 8$, so $a = 2$

Focus: $(h+a, k) = (-3+2, 2) = (-1, 2)$

Directrix: $x = h - a = -3 - 2 = -5$

**Answer**: Vertex $(-3, 2)$, Focus $(-1, 2)$, Directrix $x = -5$

---

## 💡 Tips and Insights

> 💡 **Memory Aid**: "4a" always appears with the unsquared variable.

> 💡 **Sign Check**: Positive 4a → opens positive direction; negative → opens negative direction.

> ⚠️ **Common Error**: Don't confuse which term is squared. $y^2$ means horizontal axis, $x^2$ means vertical axis.

> 💡 **Eccentricity**: For all parabolas, $e = 1$. This distinguishes it from ellipse $(e < 1)$ and hyperbola $(e > 1)$.

---

## 🌍 Real-World Applications

1. **Satellite Dishes**: Parabolic shape focuses signals at focus
2. **Car Headlights**: Bulb at focus, rays emerge parallel
3. **Solar Cookers**: Concentrate sunlight at focus
4. **Telescope Mirrors**: Parabolic mirrors in reflectors
5. **Bridge Cables**: Main cables approximate parabolic shape
6. **Projectile Motion**: Path of thrown objects (under gravity)

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Definition | Locus equidistant from focus and directrix |
| Eccentricity | e = 1 for all parabolas |
| Standard form (right) | $y^2 = 4ax$ |
| Standard form (up) | $x^2 = 4ay$ |
| Vertex to focus distance | a |
| Vertex to directrix distance | a |
| Axis | Perpendicular to directrix through focus |

---

## ❓ Quick Revision Questions

1. **Find focus and directrix of $y^2 = 16x$.**

2. **Find focus and directrix of $x^2 = -12y$.**

3. **Write the equation of parabola with focus $(0, 3)$ and vertex at origin.**

4. **Find 'a' if parabola $y^2 = 4ax$ passes through $(3, 6)$.**

5. **Find vertex, focus, and directrix of $(x+1)^2 = 16(y-2)$.**

6. **Which way does parabola $y^2 + 8x = 0$ open?**

<details>
<summary><b>Click to see answers</b></summary>

1. $y^2 = 16x$ → $4a = 16$ → $a = 4$
   **Focus: $(4, 0)$, Directrix: $x = -4$**

2. $x^2 = -12y$ → $4a = 12$ → $a = 3$, opens down
   **Focus: $(0, -3)$, Directrix: $y = 3$**

3. Focus at $(0, 3)$ means parabola opens upward
   $a = 3$
   **$x^2 = 12y$**

4. $36 = 4a \cdot 3$
   $36 = 12a$
   **$a = 3$**

5. Form: $(x-h)^2 = 4a(y-k)$
   Vertex: $(-1, 2)$
   $4a = 16$ → $a = 4$
   Focus: $(-1, 2+4) = (-1, 6)$
   Directrix: $y = 2 - 4 = -2$
   **Vertex $(-1, 2)$, Focus $(-1, 6)$, Directrix $y = -2$**

6. $y^2 = -8x$
   Negative coefficient of x → **Opens left**

</details>

---

## ⏭️ Navigation

| [← Unit 5 Home](README.md) | [Next: Elements of Parabola →](02-elements.md) |
|:--------------------------|-----------------------------------------------:|
