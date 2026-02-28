# Chapter 1.1: Coordinate Axes and Quadrants

## 📚 Chapter Overview

The Cartesian coordinate system provides a framework for locating points in a plane using two perpendicular number lines. This chapter introduces the fundamental concepts of coordinate axes, quadrants, and how to represent points using ordered pairs.

---

## 📝 Historical Background

The Cartesian coordinate system was independently developed by two French mathematicians in the 17th century:

- **René Descartes (1596-1650)** - Published "La Géométrie" in 1637
- **Pierre de Fermat (1607-1665)** - Developed similar ideas around the same time

The system is named "Cartesian" after the Latinized name of Descartes: *Cartesius*.

---

## 📝 The Coordinate Plane

### Definition

The **Cartesian coordinate plane** (or **rectangular coordinate system**) consists of:

1. **X-axis**: A horizontal number line
2. **Y-axis**: A vertical number line
3. **Origin (O)**: The point where both axes intersect, denoted as (0, 0)

```
                          Y-axis
                            ↑
                            │
                        4   ┼
                            │
                        3   ┼
                            │
                        2   ┼           • P(3, 2)
                            │
                        1   ┼
                            │
    ────┼───┼───┼───┼───┼───O───┼───┼───┼───┼───┼────→ X-axis
       -5  -4  -3  -2  -1   │   1   2   3   4   5
                       -1   ┼
                            │
                       -2   ┼
                            │
                       -3   ┼
                            │
                       -4   ┼
                            │
```

---

## 📝 Ordered Pairs

### Definition

An **ordered pair** $(x, y)$ represents the coordinates of a point in the plane:

- **x-coordinate (abscissa)**: The horizontal distance from the y-axis
- **y-coordinate (ordinate)**: The vertical distance from the x-axis

### Key Properties

| Property | Description |
|----------|-------------|
| Order matters | $(3, 5) \neq (5, 3)$ |
| Unique representation | Each point has exactly one ordered pair |
| Origin | Represented as $(0, 0)$ |

### Plotting a Point

To plot point $P(a, b)$:
1. Start at the origin
2. Move $|a|$ units right (if $a > 0$) or left (if $a < 0$)
3. Move $|b|$ units up (if $b > 0$) or down (if $b < 0$)

```
        Y
        ↑
        │           Step 2: Move up b units
        │                 ↑
        │                 │
        │    ─────────────┼─── • P(a, b)
        │    :            b
        │    :            :
        │    :            :
    ────O────:────────────:────→ X
             ←─────a─────→
             Step 1: Move right a units
```

---

## 📝 The Four Quadrants

The coordinate axes divide the plane into four regions called **quadrants**, numbered counterclockwise from the positive x-axis.

```
                            Y
                            ↑
                            │
         QUADRANT II        │        QUADRANT I
                            │
         (−, +)             │         (+, +)
       x < 0, y > 0         │       x > 0, y > 0
                            │
    ────────────────────────O────────────────────────→ X
                            │
         QUADRANT III       │        QUADRANT IV
                            │
         (−, −)             │         (+, −)
       x < 0, y < 0         │       x > 0, y < 0
                            │
```

### Quadrant Sign Rules

| Quadrant | x-coordinate | y-coordinate | Example Point |
|----------|--------------|--------------|---------------|
| **I** | Positive (+) | Positive (+) | (3, 4) |
| **II** | Negative (−) | Positive (+) | (−2, 5) |
| **III** | Negative (−) | Negative (−) | (−3, −2) |
| **IV** | Positive (+) | Negative (−) | (4, −3) |

---

## 📝 Points on the Axes

### Points on the X-axis

- All points have **y-coordinate = 0**
- General form: $(x, 0)$
- Examples: $(3, 0)$, $(−5, 0)$, $(0, 0)$

```
        Y
        ↑
        │
    ────●───●───●───O───●───●───●────→ X
      (-3,0)(-2,0)(-1,0)(0,0)(1,0)(2,0)(3,0)
        │
```

### Points on the Y-axis

- All points have **x-coordinate = 0**
- General form: $(0, y)$
- Examples: $(0, 4)$, $(0, −2)$, $(0, 0)$

```
        Y
        ↑
        ●  (0, 3)
        │
        ●  (0, 2)
        │
        ●  (0, 1)
        │
    ────O────────────→ X
        │
        ●  (0, -1)
        │
        ●  (0, -2)
        │
```

### Special Observations

| Location | Condition | Example |
|----------|-----------|---------|
| On X-axis | $y = 0$ | $(5, 0)$ |
| On Y-axis | $x = 0$ | $(0, 7)$ |
| At Origin | $x = 0$ and $y = 0$ | $(0, 0)$ |
| Not on any axis | $x \neq 0$ and $y \neq 0$ | $(3, 4)$ |

---

## 📝 Reflection of Points

### Reflection Across X-axis

The reflection of point $P(x, y)$ across the X-axis is $P'(x, -y)$

```
        Y
        ↑
        │     • P(2, 3)
        │
        │
    ────O────────────→ X  ← X-axis (mirror)
        │
        │
        │     • P'(2, -3)
        │
```

### Reflection Across Y-axis

The reflection of point $P(x, y)$ across the Y-axis is $P'(-x, y)$

```
        Y
        ↑
        │
  P'(-2,3) •         • P(2, 3)
        │
        │
    ────O────────────→ X
        │
            ↑
       Y-axis (mirror)
```

### Reflection Across Origin

The reflection of point $P(x, y)$ across the origin is $P'(-x, -y)$

```
        Y
        ↑
        │           • P(2, 3)
        │
        │
    ────O────────────→ X
        │
        │
  P'(-2,-3) •
        │
```

### Reflection Summary Table

| Reflection Across | Original Point | Reflected Point |
|-------------------|----------------|-----------------|
| X-axis | $(x, y)$ | $(x, -y)$ |
| Y-axis | $(x, y)$ | $(-x, y)$ |
| Origin | $(x, y)$ | $(-x, -y)$ |
| Line $y = x$ | $(x, y)$ | $(y, x)$ |
| Line $y = -x$ | $(x, y)$ | $(-y, -x)$ |

---

## ✏️ Worked Examples

### Example 1: Identifying Quadrants

**Problem**: In which quadrant or on which axis do the following points lie?
- A(3, 7)
- B(−4, 2)
- C(−5, −3)
- D(6, −8)
- E(0, 5)
- F(−3, 0)

**Solution**:

| Point | x-sign | y-sign | Location |
|-------|--------|--------|----------|
| A(3, 7) | + | + | Quadrant I |
| B(−4, 2) | − | + | Quadrant II |
| C(−5, −3) | − | − | Quadrant III |
| D(6, −8) | + | − | Quadrant IV |
| E(0, 5) | 0 | + | Y-axis |
| F(−3, 0) | − | 0 | X-axis |

---

### Example 2: Finding Reflections

**Problem**: Find the reflection of point P(4, −3) across:
(a) X-axis
(b) Y-axis  
(c) Origin

**Solution**:

```
            Y
            ↑
   P''(-4,3)●         ● P'(4, 3)   ← Reflection across X-axis
            │
            │
    ────────O────────────→ X
            │
  P'''(-4,-3)●         ● P(4, -3)  ← Original Point
            │
```

(a) Reflection across X-axis: $P'(4, 3)$ — Change sign of y
(b) Reflection across Y-axis: $P''(-4, -3)$ — Change sign of x  
(c) Reflection across Origin: $P'''(-4, 3)$ — Change sign of both

---

### Example 3: Quadrant Determination

**Problem**: If point P(a, b) lies in Quadrant III, in which quadrant do the following points lie?
(a) (−a, b)
(b) (a, −b)
(c) (−a, −b)
(d) (b, a)

**Solution**:

Since P(a, b) is in Quadrant III: **a < 0** and **b < 0**

| Point | Analysis | Quadrant |
|-------|----------|----------|
| (−a, b) | −a > 0, b < 0 | **IV** |
| (a, −b) | a < 0, −b > 0 | **II** |
| (−a, −b) | −a > 0, −b > 0 | **I** |
| (b, a) | b < 0, a < 0 | **III** |

---

## 💡 Tips and Insights

> 💡 **Memory Tip**: Quadrants are numbered counterclockwise, starting from the positive direction of both axes (Quadrant I).

> 💡 **Quick Check**: The product $xy > 0$ in Quadrants I and III (same signs), and $xy < 0$ in Quadrants II and IV (opposite signs).

> ⚠️ **Common Mistake**: Don't confuse $(3, 5)$ with $(5, 3)$ — the order matters! The first number is always the x-coordinate.

---

## 🌍 Real-World Applications

1. **GPS Coordinates**: Latitude and longitude work similarly to x and y coordinates
2. **Computer Graphics**: Screen pixels are addressed using coordinate systems
3. **Maps**: Grid references use ordered pairs to locate places
4. **Architecture**: Floor plans use coordinate systems for precise measurements
5. **Video Games**: Character positions are tracked using coordinate pairs

---

## 📋 Summary Table

| Concept | Key Point |
|---------|-----------|
| Coordinate Plane | Formed by two perpendicular number lines |
| Origin | Intersection point (0, 0) |
| Ordered Pair | (x, y) where x is abscissa, y is ordinate |
| Quadrant I | x > 0, y > 0 |
| Quadrant II | x < 0, y > 0 |
| Quadrant III | x < 0, y < 0 |
| Quadrant IV | x > 0, y < 0 |
| X-axis points | y = 0 |
| Y-axis points | x = 0 |

---

## ❓ Quick Revision Questions

1. **In which quadrant does the point (−7, −4) lie?**

2. **If point A lies on the positive X-axis, what can you say about its y-coordinate?**

3. **What is the reflection of point (5, −2) across the Y-axis?**

4. **If point P(a, b) is in Quadrant II, in which quadrant is (b, a)?**

5. **A point is equidistant from both axes. What is the relationship between its coordinates?**

6. **What are the coordinates of a point that lies on both axes simultaneously?**

<details>
<summary><b>Click to see answers</b></summary>

1. Quadrant III (both coordinates are negative)

2. The y-coordinate is 0 (all points on X-axis have y = 0)

3. (−5, −2) — only the x-coordinate changes sign

4. Quadrant IV. Since a < 0 and b > 0 in Quadrant II, point (b, a) has b > 0 and a < 0, meaning x > 0 and y < 0 → Quadrant IV

5. Either x = y or x = −y (the point lies on the line y = x or y = −x)

6. (0, 0) — the origin is the only point common to both axes

</details>

---

## ⏭️ Navigation

| [← Back to Unit 1](README.md) | [Next: Distance Formula →](02-distance-formula.md) |
|:-----------------------------|---------------------------------------------------:|
