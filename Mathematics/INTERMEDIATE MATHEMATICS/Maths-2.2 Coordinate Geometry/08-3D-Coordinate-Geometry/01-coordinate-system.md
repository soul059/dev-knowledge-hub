# Chapter 8.1: Three-Dimensional Coordinate System

## 📚 Chapter Overview

This chapter introduces the **three-dimensional coordinate system**, extending the familiar 2D Cartesian plane to 3D space. We'll explore coordinate axes, octants, coordinates of points, and the fundamental distance formula in 3D.

---

## 📝 The 3D Coordinate System

### Coordinate Axes

Three mutually perpendicular lines intersecting at a point called the **origin (O)** form the coordinate axes:

- **x-axis**: Usually pointing towards the viewer (or right)
- **y-axis**: Usually pointing to the right (or into the page)
- **z-axis**: Usually pointing upward (vertical)

```
                z (vertical)
                |
                |
                |
                O_________ y (horizontal)
               /
              /
             /
            x (towards viewer)
```

### Right-Handed System

The standard convention follows the **right-hand rule**:
- Point fingers along positive x-axis
- Curl fingers towards positive y-axis
- Thumb points in positive z-direction

```
   Right-Hand Rule:
   
      z (thumb)
      |
      |_____ y (fingers curl this way)
     /
    x (fingers start here)
```

---

## 📐 Coordinate Planes

The three coordinate axes determine three **coordinate planes**:

| Plane | Contains Axes | Equation | Property |
|-------|--------------|----------|----------|
| **xy-plane** | x and y | $z = 0$ | Horizontal plane |
| **yz-plane** | y and z | $x = 0$ | Vertical plane |
| **xz-plane** | x and z | $y = 0$ | Vertical plane |

```
           z
           |   yz-plane (x=0)
           |  /
           | /_______ y
           |/|
   ________/________ xy-plane (z=0)
          /|
         / |
        x  xz-plane (y=0)
```

---

## 📝 The Eight Octants

The three coordinate planes divide 3D space into **eight regions** called **octants**:

| Octant | x | y | z | Position |
|--------|---|---|---|----------|
| I | + | + | + | Front-right-top |
| II | − | + | + | Back-right-top |
| III | − | − | + | Back-left-top |
| IV | + | − | + | Front-left-top |
| V | + | + | − | Front-right-bottom |
| VI | − | + | − | Back-right-bottom |
| VII | − | − | − | Back-left-bottom |
| VIII | + | − | − | Front-left-bottom |

```
           z+
           |
     II    |    I
           |
    -------O------- y+
           |
     III   |   IV
          /
         x+
         
   (Upper four octants shown; lower four are below xy-plane)
```

---

## 📝 Coordinates of a Point

A point P in 3D space is represented by an **ordered triple** $(x, y, z)$:

- **x-coordinate**: Perpendicular distance from yz-plane
- **y-coordinate**: Perpendicular distance from xz-plane  
- **z-coordinate**: Perpendicular distance from xy-plane

```
                z
                |
                |     P(x,y,z)
                |    /|
                |   / |
                |  /  |
                | /   |
                |/____|________ y
               /     Q(x,y,0)
              /
             x
             
   P is located by moving x units along x-axis,
   y units parallel to y-axis, z units parallel to z-axis
```

---

## ⚡ Distance Formula in 3D

### Distance from Origin

For point $P(x, y, z)$:

$$\boxed{OP = \sqrt{x^2 + y^2 + z^2}}$$

### Distance Between Two Points

For points $P_1(x_1, y_1, z_1)$ and $P_2(x_2, y_2, z_2)$:

$$\boxed{P_1P_2 = \sqrt{(x_2-x_1)^2 + (y_2-y_1)^2 + (z_2-z_1)^2}}$$

### Derivation

Consider points $P_1(x_1, y_1, z_1)$ and $P_2(x_2, y_2, z_2)$.

Construct a rectangular box with $P_1P_2$ as diagonal:

```
                 P₂(x₂,y₂,z₂)
                /|
               / |
              /  | (z₂-z₁)
             /   |
            /    |_____
           /          |
    P₁(x₁,y₁,z₁)______|
        (x₂-x₁)   (y₂-y₁)
```

By Pythagoras in 3D:
$$P_1P_2^2 = (x_2-x_1)^2 + (y_2-y_1)^2 + (z_2-z_1)^2$$

---

## 📝 Special Points and Distances

### Distance from Axes

Distance of $P(x, y, z)$ from:

| Axis | Distance | Formula |
|------|----------|---------|
| x-axis | $\sqrt{y^2 + z^2}$ | Project onto yz-plane |
| y-axis | $\sqrt{x^2 + z^2}$ | Project onto xz-plane |
| z-axis | $\sqrt{x^2 + y^2}$ | Project onto xy-plane |

### Distance from Planes

Distance of $P(x, y, z)$ from:

| Plane | Distance |
|-------|----------|
| xy-plane | $|z|$ |
| yz-plane | $|x|$ |
| xz-plane | $|y|$ |

---

## ✏️ Worked Examples

### Example 1: Plotting Points

**Problem**: In which octant do the following points lie?
(a) $(2, 3, 4)$ (b) $(-1, 2, 3)$ (c) $(1, -1, -1)$

**Solution**:

(a) $(2, 3, 4)$: $x > 0$, $y > 0$, $z > 0$ → **Octant I**

(b) $(-1, 2, 3)$: $x < 0$, $y > 0$, $z > 0$ → **Octant II**

(c) $(1, -1, -1)$: $x > 0$, $y < 0$, $z < 0$ → **Octant VIII**

---

### Example 2: Distance from Origin

**Problem**: Find distance of $(3, 4, 12)$ from origin.

**Solution**:

$OP = \sqrt{3^2 + 4^2 + 12^2} = \sqrt{9 + 16 + 144} = \sqrt{169} = 13$

**Answer**: 13 units

---

### Example 3: Distance Between Points

**Problem**: Find distance between $A(1, 2, 3)$ and $B(4, 6, 3)$.

**Solution**:

$AB = \sqrt{(4-1)^2 + (6-2)^2 + (3-3)^2}$

$= \sqrt{9 + 16 + 0} = \sqrt{25} = 5$

**Answer**: 5 units

---

### Example 4: Distance from Axes

**Problem**: Find distance of $(3, 4, 5)$ from (a) x-axis (b) z-axis.

**Solution**:

(a) Distance from x-axis $= \sqrt{y^2 + z^2} = \sqrt{16 + 25} = \sqrt{41}$

(b) Distance from z-axis $= \sqrt{x^2 + y^2} = \sqrt{9 + 16} = 5$

**Answer**: (a) $\sqrt{41}$ (b) 5

---

### Example 5: Equidistant Point

**Problem**: Find the point on y-axis equidistant from $A(3, 2, 1)$ and $B(1, 1, 2)$.

**Solution**:

Point on y-axis: $P(0, y, 0)$

$PA = PB$

$\sqrt{9 + (y-2)^2 + 1} = \sqrt{1 + (y-1)^2 + 4}$

$10 + (y-2)^2 = 5 + (y-1)^2$

$10 + y^2 - 4y + 4 = 5 + y^2 - 2y + 1$

$14 - 4y = 6 - 2y$

$8 = 2y$

$y = 4$

**Answer**: $(0, 4, 0)$

---

### Example 6: Type of Triangle

**Problem**: Determine the type of triangle with vertices $A(1, 2, 3)$, $B(2, 3, 1)$, $C(3, 1, 2)$.

**Solution**:

$AB = \sqrt{1 + 1 + 4} = \sqrt{6}$

$BC = \sqrt{1 + 4 + 1} = \sqrt{6}$

$CA = \sqrt{4 + 1 + 1} = \sqrt{6}$

Since $AB = BC = CA$, triangle is **equilateral**.

**Answer**: Equilateral triangle with side $\sqrt{6}$

---

## 📝 Projections

### Foot of Perpendicular

The **foot of perpendicular** from $P(a, b, c)$ to:

| Surface | Foot |
|---------|------|
| xy-plane | $(a, b, 0)$ |
| yz-plane | $(0, b, c)$ |
| xz-plane | $(a, 0, c)$ |
| x-axis | $(a, 0, 0)$ |
| y-axis | $(0, b, 0)$ |
| z-axis | $(0, 0, c)$ |

```
           z
           |
           |  P(a,b,c)
           | /|
           |/ |
     ______|__|_______ y
          /|  |
         / |  |(0,b,c)
        x  (a,0,0)
           
   Projections onto axes and planes
```

---

## 💡 Tips and Insights

> 💡 **3D Extension**: The 3D distance formula is a natural extension of 2D: just add $(z_2 - z_1)^2$ under the square root.

> 💡 **Right-Hand Rule**: Always use right-handed coordinate system for consistency.

> ⚠️ **Sign Convention**: Pay attention to signs when determining octants.

> 💡 **Visualization**: Sketch the scenario whenever possible - 3D problems become easier with diagrams.

---

## 📋 Summary Table

| Concept | 2D | 3D |
|---------|----|----|
| Point | $(x, y)$ | $(x, y, z)$ |
| Origin | $(0, 0)$ | $(0, 0, 0)$ |
| Distance from origin | $\sqrt{x^2 + y^2}$ | $\sqrt{x^2 + y^2 + z^2}$ |
| Distance formula | $\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2}$ | $\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2 + (z_2-z_1)^2}$ |
| Regions | 4 quadrants | 8 octants |
| Coordinate lines | 2 axes | 3 axes |
| Coordinate surfaces | N/A | 3 planes |

### Distance from Axes and Planes

| From | Distance |
|------|----------|
| x-axis | $\sqrt{y^2 + z^2}$ |
| y-axis | $\sqrt{x^2 + z^2}$ |
| z-axis | $\sqrt{x^2 + y^2}$ |
| xy-plane | $|z|$ |
| yz-plane | $|x|$ |
| xz-plane | $|y|$ |

---

## ❓ Quick Revision Questions

1. **In which octant does $(-2, 3, -4)$ lie?**

2. **Find distance between $(1, 1, 1)$ and $(2, 2, 2)$.**

3. **Find distance of $(2, 3, 6)$ from origin.**

4. **What is the distance of $(4, 5, 6)$ from the xy-plane?**

5. **Find distance of $(1, 2, 2)$ from z-axis.**

6. **If $P(x, y, z)$ is equidistant from $(1, 0, 0)$ and $(0, 2, 0)$, find relation between x, y, z.**

<details>
<summary><b>Click to see answers</b></summary>

1. $x < 0$, $y > 0$, $z < 0$ → **Octant VI**

2. $d = \sqrt{1 + 1 + 1} = \sqrt{3}$

3. $d = \sqrt{4 + 9 + 36} = \sqrt{49} = 7$

4. Distance from xy-plane $= |z| = |6| = 6$

5. Distance from z-axis $= \sqrt{x^2 + y^2} = \sqrt{1 + 4} = \sqrt{5}$

6. Equidistant: $(x-1)^2 + y^2 + z^2 = x^2 + (y-2)^2 + z^2$
   $x^2 - 2x + 1 + y^2 = x^2 + y^2 - 4y + 4$
   $-2x + 1 = -4y + 4$
   $-2x + 4y = 3$
   **$2x - 4y + 3 = 0$** or **$2x - 4y = -3$**

</details>

---

## ⏭️ Navigation

| [← Unit 8 Home](README.md) | [Unit 8 Home](README.md) | [Next: Section Formula →](02-section-formula.md) |
|:--------------------------:|:------------------------:|------------------------------------------------:|
