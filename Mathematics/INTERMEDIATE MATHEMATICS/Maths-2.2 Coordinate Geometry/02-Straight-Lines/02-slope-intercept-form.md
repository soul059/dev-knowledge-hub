# Chapter 2.2: Slope-Intercept Form

## 📚 Chapter Overview

The **slope-intercept form** is the most commonly used form of a line equation. It explicitly shows the slope and the y-intercept, making it ideal for graphing and understanding linear relationships.

---

## 📝 The Slope-Intercept Form

### Definition

> **Slope-Intercept Form**: A line with slope $m$ and y-intercept $c$ has the equation:
>
> $$\boxed{y = mx + c}$$

Where:
- **m** = slope of the line
- **c** = y-intercept (the y-coordinate where the line crosses the Y-axis)

```
        Y
        ↑
        │        /
        │       /
        │      /
      c ●─────/───── ← y-intercept at (0, c)
        │    /
        │   / slope = m
        │  /
        │ /
    ────O──────────→ X
        │
```

---

## 📝 Understanding the Components

### The Y-Intercept (c)

The **y-intercept** is the point where the line crosses the Y-axis:
- Located at point (0, c)
- Found by setting x = 0 in the equation

```
    c > 0                    c = 0                    c < 0
    
        Y                        Y                        Y
        ↑                        ↑                        ↑
        │  /                     │  /                     │  /
      c ●                        │ /                      │ /
        │\                       │/                       │/
        │ \                  ────O────→ X            ────O────→ X
        │  \                     │                        │
    ────O────→ X                 │                      c ●
        │                        │                        │
    
    Line crosses Y-axis     Line passes             Line crosses Y-axis
    above origin            through origin          below origin
```

### The Slope (m)

The **slope** determines:
- **Steepness**: Larger |m| means steeper line
- **Direction**: Positive m → rising; Negative m → falling

---

## 📝 Derivation

Consider a line with slope $m$ passing through point $(0, c)$ on the Y-axis.

Using point-slope form:
$$y - c = m(x - 0)$$
$$y - c = mx$$
$$y = mx + c$$

---

## ✏️ Worked Examples

### Example 1: Identifying Slope and Intercept

**Problem**: For the line $y = 3x - 5$, identify the slope and y-intercept.

**Solution**:

Comparing with $y = mx + c$:
- Slope $m = 3$
- Y-intercept $c = -5$

The line crosses the Y-axis at point **(0, −5)**.

---

### Example 2: Writing Equation from Graph

**Problem**: A line passes through (0, 4) and has slope −2. Write its equation.

**Solution**:

Given: $m = -2$, $c = 4$

$$y = mx + c = -2x + 4$$

**Answer**: $y = -2x + 4$

---

### Example 3: Graphing a Line

**Problem**: Graph the line $y = \frac{2}{3}x + 1$.

**Solution**:

**Step 1**: Identify components
- Slope $m = \frac{2}{3}$
- Y-intercept $c = 1$ → Point (0, 1)

**Step 2**: Use slope to find another point
- From (0, 1), move right 3 units (run) and up 2 units (rise)
- New point: (3, 3)

**Step 3**: Plot and draw

```
        Y
        ↑
    4   │           /
        │          /
    3   │        ●(3, 3)
        │       /
    2   │      /
        │     /
    1   ●────/───── (0, 1) = y-intercept
        │   /
    ────O──────────→ X
        │ 1  2  3
```

---

### Example 4: Finding X-Intercept

**Problem**: Find the x-intercept of the line $y = 2x + 6$.

**Solution**:

The x-intercept is where the line crosses the X-axis (y = 0):
$$0 = 2x + 6$$
$$2x = -6$$
$$x = -3$$

**Answer**: X-intercept is at **(−3, 0)**

---

### Example 5: Converting to Slope-Intercept Form

**Problem**: Convert $3x - 4y + 12 = 0$ to slope-intercept form.

**Solution**:

Solve for y:
$$-4y = -3x - 12$$
$$y = \frac{-3x - 12}{-4}$$
$$y = \frac{3}{4}x + 3$$

**Answer**: $y = \frac{3}{4}x + 3$ (slope = $\frac{3}{4}$, y-intercept = 3)

---

### Example 6: Parallel Line

**Problem**: Find the equation of a line parallel to $y = 2x + 3$ and passing through (1, 5).

**Solution**:

Parallel lines have equal slopes, so $m = 2$.

Using point-slope form with point (1, 5):
$$y - 5 = 2(x - 1)$$
$$y = 2x - 2 + 5$$
$$y = 2x + 3$$

Wait — this gives the same line! Let's reconsider: checking if (1, 5) is on $y = 2x + 3$:
$$5 = 2(1) + 3 = 5$$ ✓

The point is on the original line, so the "parallel line through this point" **is** the original line.

**Alternative Problem**: Find a line parallel to $y = 2x + 3$ passing through (1, 4):
$$y - 4 = 2(x - 1)$$
$$y = 2x - 2 + 4$$
$$y = 2x + 2$$

---

## 📝 Special Cases

### Case 1: Line Through Origin

When $c = 0$: $y = mx$

```
        Y
        ↑      /
        │     /
        │    /
        │   /
        │  /
        │ /
    ────O──────────→ X
        │
        
    y = mx (passes through origin)
```

### Case 2: Horizontal Line

When $m = 0$: $y = c$

```
        Y
        ↑
        │
    ────●────────●────→  y = c
      c │
        │
    ────O────────────→ X
```

### Case 3: Line with Unit Slope

When $m = 1$: $y = x + c$ (45° line)

When $m = -1$: $y = -x + c$ (135° line)

---

## 📝 Comparing Lines

### Lines with Same Slope (Parallel)

```
        Y
        ↑      / y = 2x + 3
        │     / / y = 2x + 1
        │    / /
        │   / / y = 2x - 1
        │  / /
        │ / /
    ────O─/─/────→ X
        │/ /
         /
    
    All have slope m = 2
    Different y-intercepts make them parallel
```

### Lines with Same Y-Intercept

```
        Y
        ↑  |  /
        │  | /  Different slopes
        │  |/
      c ●──●─────
        │ /|
        │/ │
    ────O──│─────→ X
       /│  │
      / │  │
    
    All pass through (0, c)
    Different slopes make them intersect at y-intercept
```

---

## 📝 Applications of Slope-Intercept Form

### 1. Linear Functions

The equation $y = mx + c$ represents a linear function where:
- $m$ is the rate of change
- $c$ is the initial value (when x = 0)

### 2. Cost Functions

**Example**: A taxi charges ₹50 base fare plus ₹15 per kilometer.

Cost equation: $C = 15d + 50$

Where: $d$ = distance, $C$ = total cost
- Slope = 15 (cost per km)
- Y-intercept = 50 (base fare)

### 3. Temperature Conversion

Fahrenheit to Celsius: $F = \frac{9}{5}C + 32$

- Slope = $\frac{9}{5}$
- Intercept = 32 (freezing point of water in °F)

---

## 📝 Finding Line Equation Given Two Points

To find the slope-intercept form when given two points:

**Step 1**: Calculate slope using $m = \frac{y_2 - y_1}{x_2 - x_1}$

**Step 2**: Substitute m and one point into $y = mx + c$ to find c

**Step 3**: Write the equation

**Example**: Find the equation of line through (2, 3) and (6, 11).

$$m = \frac{11 - 3}{6 - 2} = \frac{8}{4} = 2$$

Using point (2, 3): $3 = 2(2) + c$
$$3 = 4 + c$$
$$c = -1$$

**Equation**: $y = 2x - 1$

---

## 💡 Tips and Insights

> 💡 **Quick Graphing**: Plot the y-intercept first, then use rise/run from the slope to find another point.

> 💡 **Sign of c**: If c > 0, line crosses Y-axis above origin; if c < 0, below origin.

> ⚠️ **Vertical Lines**: Cannot be written in slope-intercept form (undefined slope).

> 💡 **Parallel Lines**: Same m, different c.

> 💡 **Conversion**: Always isolate y on one side to get slope-intercept form.

---

## 🌍 Real-World Applications

1. **Economics**: Supply and demand curves, linear cost functions
2. **Physics**: Uniform motion (distance = velocity × time + initial position)
3. **Finance**: Simple interest calculations
4. **Engineering**: Linear approximations in control systems
5. **Medicine**: Drug dosage calculations
6. **Climate Science**: Temperature trends over time

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| Equation | $y = mx + c$ |
| Slope | m (coefficient of x) |
| Y-intercept | c (constant term), at point (0, c) |
| X-intercept | At $(-\frac{c}{m}, 0)$ when $m \neq 0$ |
| Horizontal line | m = 0, equation: y = c |
| Through origin | c = 0, equation: y = mx |
| Parallel lines | Same slope m |
| To graph | Plot (0, c), then use slope for next point |

---

## ❓ Quick Revision Questions

1. **Write the equation of a line with slope 4 and y-intercept −3.**

2. **Find the slope and y-intercept of $5x - 2y = 10$.**

3. **A line has equation $y = -3x + 7$. At what point does it cross the X-axis?**

4. **Find the equation of a line parallel to $y = \frac{1}{2}x + 4$ passing through origin.**

5. **Convert $2x + 3y - 6 = 0$ to slope-intercept form.**

6. **A car rental costs ₹500 plus ₹8 per km. Express total cost C in terms of distance d.**

<details>
<summary><b>Click to see answers</b></summary>

1. $y = 4x - 3$

2. $-2y = -5x + 10$
   $y = \frac{5}{2}x - 5$
   Slope = $\frac{5}{2}$, Y-intercept = −5

3. Set y = 0: $0 = -3x + 7$
   $x = \frac{7}{3}$
   X-intercept: $\left(\frac{7}{3}, 0\right)$

4. Same slope $\frac{1}{2}$, through origin means c = 0
   **Equation**: $y = \frac{1}{2}x$

5. $3y = -2x + 6$
   $y = -\frac{2}{3}x + 2$

6. **C = 8d + 500** (slope = 8, intercept = 500)

</details>

---

## ⏭️ Navigation

| [← Previous: Slope of a Line](01-slope-of-line.md) | [Next: Point-Slope Form →](03-point-slope-two-point-forms.md) |
|:--------------------------------------------------|-------------------------------------------------------------:|
