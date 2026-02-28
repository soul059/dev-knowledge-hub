# Chapter 2.1: Slope of a Line

## 📚 Chapter Overview

The **slope** (or **gradient**) of a line is a measure of its steepness and direction. Understanding slope is fundamental to analyzing linear relationships and forms the basis for all line equations.

---

## 📝 Inclination of a Line

### Definition

The **inclination** of a line is the angle $\theta$ that the line makes with the positive direction of the X-axis, measured counterclockwise.

```
        Y
        ↑
        │      /
        │     / Line
        │    /
        │   /
        │  /
        │ /θ
    ────O─────────→ X
        │    Positive X-axis
        
    θ = Angle of inclination (0° ≤ θ < 180°)
```

### Properties of Inclination

| Condition | Inclination θ |
|-----------|---------------|
| Horizontal line | θ = 0° |
| Line rising left to right | 0° < θ < 90° |
| Vertical line | θ = 90° |
| Line falling left to right | 90° < θ < 180° |

---

## 📝 Definition of Slope

> **Slope (m)**: The slope of a non-vertical line is defined as the tangent of its inclination angle:
>
> $$\boxed{m = \tan\theta}$$

### Slope from Two Points

For a line passing through points $P(x_1, y_1)$ and $Q(x_2, y_2)$:

> $$\boxed{m = \frac{y_2 - y_1}{x_2 - x_1} = \frac{\Delta y}{\Delta x} = \frac{\text{Rise}}{\text{Run}}}$$

```
        Y
        ↑
        │               Q(x₂, y₂)
        │               ●
        │              /│
        │             / │ Rise = y₂ - y₁
        │            /  │
        │   P(x₁,y₁)●───●
        │           Run = x₂ - x₁
    ────O──────────────────→ X
        │
```

---

## 📝 Understanding Slope Values

### Positive Slope (m > 0)

The line **rises** from left to right.

```
        Y                        
        ↑       /               
        │      /                 
        │     /                  
        │    /                   
        │   /    m > 0           
        │  /                     
    ────O─/──────→ X            
```

### Negative Slope (m < 0)

The line **falls** from left to right.

```
        Y                        
        ↑                        
        │\                       
        │ \                      
        │  \                     
        │   \    m < 0           
        │    \                   
    ────O─────\──→ X            
```

### Zero Slope (m = 0)

The line is **horizontal**.

```
        Y                        
        ↑                        
        │                        
    ────●────────●────→          
        │        m = 0           
        │                        
    ────O────────────→ X        
```

### Undefined Slope

The line is **vertical** (slope is undefined, as $x_2 - x_1 = 0$).

```
        Y                        
        ↑    │                   
        │    │                   
        │    │  m = undefined    
        │    │                   
        │    │                   
    ────O────│───→ X            
```

---

## 📝 Slope Summary Table

| Inclination θ | Slope m = tan θ | Line Type |
|---------------|-----------------|-----------|
| θ = 0° | m = 0 | Horizontal |
| 0° < θ < 45° | 0 < m < 1 | Gentle rise |
| θ = 45° | m = 1 | 45° line |
| 45° < θ < 90° | m > 1 | Steep rise |
| θ = 90° | m = undefined | Vertical |
| 90° < θ < 135° | m < −1 | Steep fall |
| θ = 135° | m = −1 | −45° line |
| 135° < θ < 180° | −1 < m < 0 | Gentle fall |

---

## 📝 Visual Comparison of Slopes

```
    m = 2        m = 1        m = 0.5       m = 0
       /           /             /            ─────
      /           /             /
     /           /             /
    /           /             /
   
    
    m = -0.5    m = -1        m = -2        m = undefined
         \          \             \               │
          \          \             \              │
           \          \             \             │
            \          \             \            │
```

---

## ✏️ Worked Examples

### Example 1: Slope from Two Points

**Problem**: Find the slope of the line passing through A(2, 3) and B(6, 11).

**Solution**:
$$m = \frac{y_2 - y_1}{x_2 - x_1} = \frac{11 - 3}{6 - 2} = \frac{8}{4} = 2$$

**Answer**: Slope m = **2**

---

### Example 2: Slope from Inclination

**Problem**: Find the slope of a line whose inclination is 60°.

**Solution**:
$$m = \tan 60° = \sqrt{3} \approx 1.732$$

**Answer**: Slope m = **√3**

---

### Example 3: Inclination from Slope

**Problem**: Find the inclination of a line with slope −1.

**Solution**:
$$\tan\theta = -1$$
$$\theta = 135°$$ (since slope is negative, θ is in second quadrant, i.e., 90° < θ < 180°)

**Answer**: Inclination θ = **135°**

---

### Example 4: Horizontal and Vertical Lines

**Problem**: Determine the slope of:
(a) Line joining (3, 5) and (8, 5)
(b) Line joining (4, 2) and (4, 9)

**Solution**:

**(a) Horizontal Line:**
$$m = \frac{5 - 5}{8 - 3} = \frac{0}{5} = 0$$

The line is horizontal with slope **m = 0**.

**(b) Vertical Line:**
$$m = \frac{9 - 2}{4 - 4} = \frac{7}{0} = \text{undefined}$$

The line is vertical with **undefined slope**.

---

### Example 5: Finding Unknown Coordinate

**Problem**: If the slope of the line joining (2, 5) and (x, 3) is 2, find x.

**Solution**:
$$m = \frac{3 - 5}{x - 2} = 2$$

$$\frac{-2}{x - 2} = 2$$

$$-2 = 2(x - 2)$$

$$-2 = 2x - 4$$

$$2x = 2$$

$$x = 1$$

**Answer**: x = **1**

---

### Example 6: Triangle Type Using Slopes

**Problem**: Show that the triangle with vertices A(0, 0), B(4, 0), and C(4, 3) is right-angled.

**Solution**:

**Slope of AB:**
$$m_{AB} = \frac{0 - 0}{4 - 0} = 0$$ (horizontal)

**Slope of BC:**
$$m_{BC} = \frac{3 - 0}{4 - 4} = \frac{3}{0} = \text{undefined}$$ (vertical)

Since AB is horizontal and BC is vertical, they are **perpendicular** to each other.

Therefore, ∠B = 90°, and the triangle is **right-angled at B**.

```
        Y
        ↑
    3   │           ●C(4, 3)
        │           │
        │           │ BC (vertical)
        │           │
    ────O───────────●────→ X
        A(0,0)      B(4,0)
            ────────
            AB (horizontal)
```

---

## 📝 Properties of Slope

### Property 1: Order Independence

The slope is the same regardless of which point is considered first:
$$\frac{y_2 - y_1}{x_2 - x_1} = \frac{y_1 - y_2}{x_1 - x_2}$$

### Property 2: Parallel Lines

Two non-vertical lines are parallel if and only if they have equal slopes:
$$\text{Parallel: } m_1 = m_2$$

### Property 3: Perpendicular Lines

Two non-vertical lines are perpendicular if and only if the product of their slopes is −1:
$$\text{Perpendicular: } m_1 \cdot m_2 = -1$$

### Property 4: Collinear Points

Three points are collinear if the slope between any two pairs is equal.

---

## 📝 Slope in Different Contexts

### Slope as Rate of Change

In many applications, slope represents the rate of change:

| Context | What Slope Represents |
|---------|----------------------|
| Distance-Time graph | Velocity (speed) |
| Velocity-Time graph | Acceleration |
| Cost-Quantity graph | Price per unit |
| Population-Time graph | Growth rate |

### Gradient in Real Life

```
    Road Gradient
    
         _________
        /    20%  \
       /           \
      /             \
     /_______________ \
     
    20% gradient means 20m rise for every 100m horizontal distance
    = slope of 0.2 or 1:5
```

---

## 💡 Tips and Insights

> 💡 **Memory Aid**: "Rise over Run" — vertical change divided by horizontal change.

> 💡 **Sign Check**: If the line goes "uphill" left to right, slope is positive; "downhill" means negative.

> ⚠️ **Vertical Lines**: Never calculate slope for vertical lines — it's undefined!

> 💡 **45° Reference**: A 45° line has slope = 1 (or −1 for 135°).

> ⚠️ **Common Mistake**: Swapping the order of subtraction in numerator but not denominator (or vice versa).

---

## 🌍 Real-World Applications

1. **Road Engineering**: Designing roads with appropriate gradients
2. **Ramps**: ADA-compliant wheelchair ramps (max 1:12 slope)
3. **Roof Design**: Pitch of roofs for water drainage
4. **Ski Slopes**: Classification by steepness
5. **Economics**: Marginal cost and marginal revenue analysis
6. **Physics**: Velocity and acceleration from graphs

---

## 📋 Summary Table

| Concept | Formula/Condition |
|---------|-------------------|
| Slope from angle | $m = \tan\theta$ |
| Slope from two points | $m = \frac{y_2 - y_1}{x_2 - x_1}$ |
| Horizontal line | m = 0, equation: y = k |
| Vertical line | m = undefined, equation: x = k |
| Positive slope | Line rises left to right |
| Negative slope | Line falls left to right |
| 45° line | m = 1 |
| 135° line | m = −1 |

---

## ❓ Quick Revision Questions

1. **Find the slope of the line joining (−3, 2) and (5, −4).**

2. **A line has inclination 120°. What is its slope?**

3. **If the slope of a line is 0, what type of line is it?**

4. **Find the value of y if the slope of line joining (3, y) and (2, 7) is 5.**

5. **What is the inclination of a line with slope √3?**

6. **Points A(1, 2), B(3, 6), and C(4, k) are collinear. Find k.**

<details>
<summary><b>Click to see answers</b></summary>

1. $m = \frac{-4 - 2}{5 - (-3)} = \frac{-6}{8} = -\frac{3}{4}$

2. $m = \tan 120° = \tan(180° - 60°) = -\tan 60° = -\sqrt{3}$

3. **Horizontal line** (parallel to X-axis)

4. $\frac{7 - y}{2 - 3} = 5$
   $\frac{7 - y}{-1} = 5$
   $7 - y = -5$
   $y = 12$

5. $\tan\theta = \sqrt{3}$
   $\theta = 60°$

6. Slope of AB = $\frac{6-2}{3-1} = \frac{4}{2} = 2$
   Slope of BC = $\frac{k-6}{4-3} = k - 6$
   For collinearity: $k - 6 = 2$
   $k = 8$

</details>

---

## ⏭️ Navigation

| [← Back to Unit 2](README.md) | [Next: Slope-Intercept Form →](02-slope-intercept-form.md) |
|:-----------------------------|-----------------------------------------------------------:|
