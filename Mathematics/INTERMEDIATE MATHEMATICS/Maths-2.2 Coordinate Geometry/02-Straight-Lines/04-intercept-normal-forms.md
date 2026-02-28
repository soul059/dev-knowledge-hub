# Chapter 2.4: Intercept and Normal Forms

## 📚 Chapter Overview

This chapter covers two important forms of line equations: the **intercept form** (useful when x and y intercepts are known) and the **normal form** (based on the perpendicular from the origin). These forms provide elegant ways to express lines with specific geometric properties.

---

## 📝 Intercept Form

### Definition

> **Intercept Form**: A line with x-intercept $a$ and y-intercept $b$ has the equation:
>
> $$\boxed{\frac{x}{a} + \frac{y}{b} = 1}$$

Where:
- **a** = x-intercept (where line crosses X-axis)
- **b** = y-intercept (where line crosses Y-axis)

```
        Y
        ↑
        │
      b ●
        │\
        │ \
        │  \
        │   \
        │    \
    ────O─────●──→ X
              a
        
    Line passes through (a, 0) and (0, b)
```

---

### Derivation

Using two-point form with points $(a, 0)$ and $(0, b)$:

$$\frac{y - 0}{b - 0} = \frac{x - a}{0 - a}$$

$$\frac{y}{b} = \frac{x - a}{-a}$$

$$\frac{y}{b} = -\frac{x}{a} + 1$$

$$\frac{x}{a} + \frac{y}{b} = 1$$

---

### Properties

| Property | Condition |
|----------|-----------|
| Both intercepts positive | Line in Q1-Q2-Q4 |
| Both intercepts negative | Line in Q2-Q3-Q4 |
| a > 0, b < 0 | Line in Q3-Q4 |
| a < 0, b > 0 | Line in Q1-Q2 |

---

## ✏️ Examples: Intercept Form

### Example 1: Basic Application

**Problem**: Write the equation of a line with x-intercept 4 and y-intercept 3.

**Solution**:

$$\frac{x}{4} + \frac{y}{3} = 1$$

Multiplying by 12:
$$3x + 4y = 12$$

**Answer**: $\frac{x}{4} + \frac{y}{3} = 1$ or $3x + 4y = 12$

---

### Example 2: Finding Intercepts

**Problem**: Express $2x + 5y = 10$ in intercept form and find the intercepts.

**Solution**:

Divide by 10:
$$\frac{2x}{10} + \frac{5y}{10} = 1$$

$$\frac{x}{5} + \frac{y}{2} = 1$$

Comparing with $\frac{x}{a} + \frac{y}{b} = 1$:
- x-intercept $a = 5$
- y-intercept $b = 2$

**Answer**: x-intercept = 5, y-intercept = 2

```
        Y
        ↑
    2   ●
        │\
        │ \
    ────O──●──→ X
           5
```

---

### Example 3: Negative Intercepts

**Problem**: Find the equation of the line with x-intercept −3 and y-intercept 6.

**Solution**:

$$\frac{x}{-3} + \frac{y}{6} = 1$$

$$-\frac{x}{3} + \frac{y}{6} = 1$$

Multiplying by 6:
$$-2x + y = 6$$

Or: $2x - y + 6 = 0$

---

### Example 4: Area of Triangle with Axes

**Problem**: A line makes intercepts $a$ and $b$ on the axes. Find the area of the triangle formed with the coordinate axes.

**Solution**:

The triangle has vertices at:
- Origin (0, 0)
- (a, 0) on X-axis
- (0, b) on Y-axis

```
        Y
        ↑
      b ●
        │\
        │ \ Triangle
        │  \
    ────O───●──→ X
            a
```

$$\text{Area} = \frac{1}{2} \times \text{base} \times \text{height} = \frac{1}{2}|a||b|$$

**Answer**: Area = $\frac{1}{2}|ab|$ square units

---

### Example 5: Sum of Intercepts Condition

**Problem**: A line passes through (2, 3) and the sum of its intercepts is 10. Find its equation.

**Solution**:

Let x-intercept = $a$, y-intercept = $b = 10 - a$

Line equation: $\frac{x}{a} + \frac{y}{10-a} = 1$

Since (2, 3) lies on the line:
$$\frac{2}{a} + \frac{3}{10-a} = 1$$

$$\frac{2(10-a) + 3a}{a(10-a)} = 1$$

$$20 - 2a + 3a = a(10 - a)$$

$$20 + a = 10a - a^2$$

$$a^2 - 9a + 20 = 0$$

$$(a - 4)(a - 5) = 0$$

$a = 4$ or $a = 5$

**Case 1**: $a = 4$, $b = 6$: $\frac{x}{4} + \frac{y}{6} = 1$ → $3x + 2y = 12$

**Case 2**: $a = 5$, $b = 5$: $\frac{x}{5} + \frac{y}{5} = 1$ → $x + y = 5$

**Answer**: $3x + 2y = 12$ or $x + y = 5$

---

## 📝 Normal Form (Perpendicular Form)

### Definition

> **Normal Form**: A line at perpendicular distance $p$ from the origin, where the perpendicular makes angle $\alpha$ with the positive X-axis:
>
> $$\boxed{x\cos\alpha + y\sin\alpha = p}$$

Where:
- **p** = perpendicular distance from origin (always positive)
- **α** = angle the perpendicular makes with positive X-axis (0° ≤ α < 360°)

```
        Y
        ↑
        │        /
        │       / Line
        │      /
        │     /
        │    N●────── Perpendicular from O
        │   /│ p     
        │  / │       
        │ /α │       
    ────O────┴───────→ X
        │
        
    ON ⊥ Line, ON = p
    ∠NOX = α
```

---

### Derivation

Let N be the foot of perpendicular from O to the line, with ON = p.

Coordinates of N:
$$N = (p\cos\alpha, p\sin\alpha)$$

Slope of ON:
$$m_{ON} = \tan\alpha$$

Since the line is perpendicular to ON:
$$m_{\text{line}} = -\cot\alpha = -\frac{\cos\alpha}{\sin\alpha}$$

Using point-slope form through N:
$$y - p\sin\alpha = -\frac{\cos\alpha}{\sin\alpha}(x - p\cos\alpha)$$

Simplifying:
$$y\sin\alpha - p\sin^2\alpha = -x\cos\alpha + p\cos^2\alpha$$

$$x\cos\alpha + y\sin\alpha = p(\cos^2\alpha + \sin^2\alpha)$$

$$x\cos\alpha + y\sin\alpha = p$$

---

### Properties of Normal Form

| Property | Details |
|----------|---------|
| $p > 0$ | Always positive (distance) |
| $0° \leq α < 360°$ | Angle measured from positive X-axis |
| Unit coefficients | $\cos^2\alpha + \sin^2\alpha = 1$ |

---

## ✏️ Examples: Normal Form

### Example 6: Basic Normal Form

**Problem**: Write the equation of a line at distance 5 from origin, where the perpendicular makes angle 60° with X-axis.

**Solution**:

$$x\cos 60° + y\sin 60° = 5$$

$$x \cdot \frac{1}{2} + y \cdot \frac{\sqrt{3}}{2} = 5$$

$$\frac{x}{2} + \frac{\sqrt{3}y}{2} = 5$$

$$x + \sqrt{3}y = 10$$

**Answer**: $x + \sqrt{3}y = 10$

---

### Example 7: Converting to Normal Form

**Problem**: Convert $3x + 4y = 10$ to normal form.

**Solution**:

For normal form, we need: $\cos^2\alpha + \sin^2\alpha = 1$

First, find the normalizing factor:
$$\sqrt{3^2 + 4^2} = \sqrt{9 + 16} = 5$$

Divide the equation by 5:
$$\frac{3x}{5} + \frac{4y}{5} = 2$$

This gives:
- $\cos\alpha = \frac{3}{5}$
- $\sin\alpha = \frac{4}{5}$
- $p = 2$

**Answer**: $\frac{3}{5}x + \frac{4}{5}y = 2$ with $p = 2$, $\cos\alpha = \frac{3}{5}$, $\sin\alpha = \frac{4}{5}$

---

### Example 8: Finding Distance from Origin

**Problem**: Find the perpendicular distance from origin to the line $12x + 5y = 26$.

**Solution**:

Normalize by dividing by $\sqrt{12^2 + 5^2} = \sqrt{144 + 25} = 13$:

$$\frac{12x}{13} + \frac{5y}{13} = \frac{26}{13} = 2$$

**Answer**: Distance from origin = **2 units**

---

### Example 9: Line at Given Distance

**Problem**: Find the equations of lines parallel to $3x + 4y = 0$ at distance 5 from origin.

**Solution**:

Lines parallel to $3x + 4y = 0$ have the form: $3x + 4y = k$

Converting to normal form:
$$\frac{3x}{\sqrt{25}} + \frac{4y}{\sqrt{25}} = \frac{k}{5}$$

For distance 5 from origin: $\frac{|k|}{5} = 5$

$$|k| = 25$$
$$k = 25 \text{ or } k = -25$$

**Answer**: $3x + 4y = 25$ and $3x + 4y = -25$

---

## 📝 Converting General Form to Normal Form

For line $ax + by + c = 0$:

**Step 1**: Move constant to RHS: $ax + by = -c$

**Step 2**: Divide by $\sqrt{a^2 + b^2}$:
$$\frac{ax}{\sqrt{a^2 + b^2}} + \frac{by}{\sqrt{a^2 + b^2}} = \frac{-c}{\sqrt{a^2 + b^2}}$$

**Step 3**: Ensure RHS is positive (if not, multiply by −1)

The result gives:
- $\cos\alpha = \frac{a}{\sqrt{a^2 + b^2}}$ (with appropriate sign)
- $\sin\alpha = \frac{b}{\sqrt{a^2 + b^2}}$ (with appropriate sign)
- $p = \frac{|c|}{\sqrt{a^2 + b^2}}$

---

## 📝 Relationship Between Forms

```
    ┌─────────────────────────────────────────────────┐
    │         CONVERSION BETWEEN LINE FORMS           │
    ├─────────────────────────────────────────────────┤
    │                                                 │
    │   Intercept Form        Normal Form             │
    │   x/a + y/b = 1    ↔   x·cosα + y·sinα = p    │
    │         ↑                     ↑                 │
    │         │                     │                 │
    │         ↓                     ↓                 │
    │         └──────→ General Form ←──────┘          │
    │                  ax + by + c = 0                │
    │                        ↑                        │
    │                        ↓                        │
    │              Slope-Intercept Form               │
    │                   y = mx + c                    │
    │                                                 │
    └─────────────────────────────────────────────────┘
```

---

## 📝 When to Use Each Form

| Form | Use When |
|------|----------|
| Intercept Form | x and y intercepts are given or needed |
| Normal Form | Perpendicular distance from origin is important |
| Slope-Intercept | Slope and y-intercept are known |
| Point-Slope | A point and slope are known |
| General Form | For algebraic manipulations |

---

## 💡 Tips and Insights

> 💡 **Intercept Form Limitation**: Cannot represent lines through the origin (both intercepts would be 0).

> 💡 **Normal Form Sign**: Ensure p is always positive by adjusting signs if necessary.

> ⚠️ **Common Mistake**: In intercept form, don't confuse intercept values with points (intercept 3 means point (3,0) or (0,3)).

> 💡 **Distance Formula**: The normal form directly gives the distance from origin as the constant term (when normalized).

---

## 🌍 Real-World Applications

1. **Optics**: Normal form describes light ray reflections
2. **Architecture**: Intercept form for designing slopes and ramps
3. **Navigation**: Distance calculations from reference points
4. **Computer Graphics**: Line clipping algorithms
5. **Physics**: Describing wavefronts perpendicular to direction of propagation
6. **Engineering**: Stress analysis in structures

---

## 📋 Summary Table

| Form | Equation | Key Components |
|------|----------|----------------|
| Intercept Form | $\frac{x}{a} + \frac{y}{b} = 1$ | a = x-intercept, b = y-intercept |
| Normal Form | $x\cos\alpha + y\sin\alpha = p$ | p = ⊥ distance from origin, α = angle |
| Area with axes | $\frac{1}{2}\|ab\|$ | For intercept form |
| To normalize | Divide by $\sqrt{a^2 + b^2}$ | For converting to normal form |

---

## ❓ Quick Revision Questions

1. **Write the intercept form of a line with x-intercept 6 and y-intercept −4.**

2. **Find the intercepts of the line $5x - 2y = 20$.**

3. **Convert $4x - 3y = 10$ to normal form.**

4. **A line makes equal intercepts on both axes. If it passes through (3, 5), find its equation.**

5. **Find the perpendicular distance from origin to the line $8x - 15y = 34$.**

6. **Write the equation of a line at distance 4 from origin with perpendicular inclined at 45°.**

<details>
<summary><b>Click to see answers</b></summary>

1. $\frac{x}{6} + \frac{y}{-4} = 1$ or $\frac{x}{6} - \frac{y}{4} = 1$
   Which gives: $2x - 3y = 12$

2. Setting y = 0: x = 4 (x-intercept)
   Setting x = 0: y = −10 (y-intercept)

3. $\sqrt{16 + 9} = 5$
   Dividing: $\frac{4x}{5} - \frac{3y}{5} = 2$
   Normal form: $\frac{4}{5}x - \frac{3}{5}y = 2$ with $p = 2$

4. Equal intercepts means $a = b$
   Line: $\frac{x}{a} + \frac{y}{a} = 1$ → $x + y = a$
   Passing through (3, 5): $3 + 5 = a$ → $a = 8$
   **Equation**: $x + y = 8$
   (Note: $a = -8$ also works giving $x + y = -8$, but (3,5) only satisfies $x + y = 8$)

5. $\sqrt{64 + 225} = \sqrt{289} = 17$
   Distance = $\frac{34}{17} = 2$ units

6. $x\cos 45° + y\sin 45° = 4$
   $\frac{x}{\sqrt{2}} + \frac{y}{\sqrt{2}} = 4$
   **Equation**: $x + y = 4\sqrt{2}$

</details>

---

## ⏭️ Navigation

| [← Previous: Point-Slope Forms](03-point-slope-two-point-forms.md) | [Next: General Equation →](05-general-equation.md) |
|:------------------------------------------------------------------|--------------------------------------------------:|
