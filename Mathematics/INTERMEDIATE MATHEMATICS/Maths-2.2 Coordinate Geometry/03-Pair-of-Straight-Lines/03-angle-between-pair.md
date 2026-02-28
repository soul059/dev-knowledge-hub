# Chapter 3.3: Angle Between Pair of Lines

## 📚 Chapter Overview

Finding the angle between two lines represented by a single equation is a powerful technique. This chapter derives the angle formula and explores special cases like perpendicular and coincident lines.

---

## 📝 The Angle Formula

For the pair of lines represented by $ax^2 + 2hxy + by^2 = 0$ (or the general form with additional terms), the acute angle θ between them is given by:

> **Angle Between Pair of Lines**:
>
> $$\boxed{\tan\theta = \left|\frac{2\sqrt{h^2 - ab}}{a + b}\right|}$$

---

## 📐 Derivation

### From the Homogeneous Equation

Let the lines be $y = m_1 x$ and $y = m_2 x$ where $m_1$, $m_2$ are roots of:
$$bm^2 + 2hm + a = 0$$

**Relations**:
- $m_1 + m_2 = -\frac{2h}{b}$
- $m_1 m_2 = \frac{a}{b}$

**Angle between two lines**:
$$\tan\theta = \left|\frac{m_1 - m_2}{1 + m_1 m_2}\right|$$

**Finding $m_1 - m_2$**:
$$(m_1 - m_2)^2 = (m_1 + m_2)^2 - 4m_1 m_2$$
$$= \frac{4h^2}{b^2} - \frac{4a}{b} = \frac{4h^2 - 4ab}{b^2} = \frac{4(h^2 - ab)}{b^2}$$

$$|m_1 - m_2| = \frac{2\sqrt{h^2 - ab}}{|b|}$$

**Substituting**:
$$\tan\theta = \left|\frac{\frac{2\sqrt{h^2-ab}}{|b|}}{1 + \frac{a}{b}}\right| = \left|\frac{2\sqrt{h^2-ab}}{|b| \cdot \frac{b+a}{b}}\right| = \left|\frac{2\sqrt{h^2-ab}}{a+b}\right|$$

```
        Y
        ↑
        │     /
        │    /  m₁
        │   /
        │  / θ
        │ /____
        │/  θ  \  m₂
    ────O───────\────→ X
        │        \
        │         \
```

---

## ⚡ Special Cases

### 1. Perpendicular Lines

When $\theta = 90°$, $\tan\theta \to \infty$

This happens when the denominator = 0:

> **Condition for Perpendicular Lines**:
> $$\boxed{a + b = 0}$$
> 
> Or equivalently: **Coefficient of $x^2$ + Coefficient of $y^2$ = 0**

### 2. Coincident (Same) Lines

When the two lines are the same, $\theta = 0°$, so $\tan\theta = 0$

This happens when the numerator = 0:

> **Condition for Coincident Lines**:
> $$\boxed{h^2 = ab}$$
> 
> (The discriminant of the quadratic in m equals zero)

### 3. Parallel Lines (Non-Coincident)

For a pair of lines from the general equation (not through origin), if they're parallel:
$$h^2 = ab \text{ but the lines are distinct}$$

This only occurs when the equation doesn't represent a pair through a common point.

---

## 📝 Angle for General Equation

For $ax^2 + 2hxy + by^2 + 2gx + 2fy + c = 0$:

The angle formula remains the same since the linear and constant terms don't affect the slopes:

$$\tan\theta = \left|\frac{2\sqrt{h^2 - ab}}{a + b}\right|$$

---

## ✏️ Worked Examples

### Example 1: Finding the Angle

**Problem**: Find the angle between lines $x^2 - 5xy + 4y^2 = 0$.

**Solution**:

Here: $a = 1$, $2h = -5$ → $h = -\frac{5}{2}$, $b = 4$

$$\tan\theta = \left|\frac{2\sqrt{\frac{25}{4} - 4}}{1 + 4}\right|$$

$$= \left|\frac{2\sqrt{\frac{25-16}{4}}}{5}\right| = \left|\frac{2 \cdot \frac{3}{2}}{5}\right| = \frac{3}{5}$$

$$\theta = \tan^{-1}\left(\frac{3}{5}\right) \approx 30.96°$$

**Answer**: $\theta = \tan^{-1}\left(\frac{3}{5}\right)$

---

### Example 2: Proving Perpendicularity

**Problem**: Show that $3x^2 + 8xy - 3y^2 = 0$ represents perpendicular lines.

**Solution**:

Here: $a = 3$, $b = -3$

Check: $a + b = 3 + (-3) = 0$ ✓

**Answer**: Since $a + b = 0$, the lines are perpendicular.

---

### Example 3: Finding k for Perpendicular Lines

**Problem**: Find k if $x^2 + 2kxy - 5y^2 = 0$ represents perpendicular lines.

**Solution**:

For perpendicular: $a + b = 0$
$$1 + (-5) = -4 \neq 0$$

Wait - the perpendicular condition doesn't depend on h (or k here).

Since $a + b = 1 - 5 = -4 \neq 0$, these lines can never be perpendicular for any value of k.

**Answer**: No value of k makes these lines perpendicular.

---

### Example 4: Finding k for Coincident Lines

**Problem**: Find k if $4x^2 + kxy + y^2 = 0$ represents coincident lines.

**Solution**:

For coincident lines: $h^2 = ab$

Here: $a = 4$, $b = 1$, $h = \frac{k}{2}$

$$\left(\frac{k}{2}\right)^2 = 4 \times 1$$
$$\frac{k^2}{4} = 4$$
$$k^2 = 16$$
$$k = \pm 4$$

**Answer**: $k = 4$ or $k = -4$

---

### Example 5: Specific Angle

**Problem**: Find the equation of pair of lines through origin such that the angle between them is 60°.

**Solution**:

For angle = 60°: $\tan 60° = \sqrt{3}$

$$\sqrt{3} = \left|\frac{2\sqrt{h^2 - ab}}{a + b}\right|$$

This gives us a relation between a, b, and h, but we need more constraints.

If we want symmetric lines (equally inclined to axes), let's take $h = 0$:
$$\sqrt{3} = \left|\frac{2\sqrt{-ab}}{a + b}\right|$$

For real lines with h = 0, we need ab < 0. Let a = 1, b = -k (k > 0):
$$\sqrt{3} = \frac{2\sqrt{k}}{1 - k}$$

This gets complex. Let's try differently.

**Alternative**: Let the lines make angles α and (α + 60°) with x-axis.

If slopes are $m_1 = \tan\alpha$ and $m_2 = \tan(\alpha + 60°)$

For simplicity, let α = 0°, then $m_1 = 0$, $m_2 = \tan 60° = \sqrt{3}$

Lines: $y = 0$ and $y = \sqrt{3}x$

Joint equation: $y(y - \sqrt{3}x) = 0$ → $y^2 - \sqrt{3}xy = 0$

**Answer**: One such pair is $\sqrt{3}xy - y^2 = 0$ or $y^2 - \sqrt{3}xy = 0$

---

### Example 6: Angle in General Form

**Problem**: Find the angle between lines $x^2 - 4xy + y^2 + x - 2y - 2 = 0$.

**Solution**:

Extract coefficients: $a = 1$, $h = -2$, $b = 1$

$$\tan\theta = \left|\frac{2\sqrt{4 - 1}}{1 + 1}\right| = \left|\frac{2\sqrt{3}}{2}\right| = \sqrt{3}$$

$$\theta = 60°$$

**Answer**: Angle = **60°**

---

### Example 7: Condition for Given Angle

**Problem**: If the angle between lines $ax^2 + 2hxy + by^2 = 0$ is 45°, prove that $h^2 = ab + (a+b)^2/4$.

**Solution**:

Given: $\tan 45° = 1$

$$1 = \left|\frac{2\sqrt{h^2 - ab}}{a + b}\right|$$

$$|a + b| = 2\sqrt{h^2 - ab}$$

$$(a + b)^2 = 4(h^2 - ab)$$

$$a^2 + 2ab + b^2 = 4h^2 - 4ab$$

$$4h^2 = a^2 + 6ab + b^2 = (a+b)^2 + 4ab$$

$$h^2 = \frac{(a+b)^2}{4} + ab$$

**Proved**: $h^2 = ab + \frac{(a+b)^2}{4}$

---

## 📝 Summary of Special Angles

| Angle θ | Condition |
|---------|-----------|
| 0° (coincident) | $h^2 = ab$ |
| 45° | $h^2 - ab = \frac{(a+b)^2}{4}$ |
| 60° | $h^2 - ab = \frac{3(a+b)^2}{4}$ |
| 90° (perpendicular) | $a + b = 0$ |

---

## 📝 Visual Understanding

```
    ┌────────────────────────────────────────────────────────────┐
    │                  ANGLE BETWEEN PAIR OF LINES               │
    ├────────────────────────────────────────────────────────────┤
    │                                                            │
    │   θ = 0° (Coincident)     θ = 90° (Perpendicular)          │
    │                                                            │
    │        ──────────               │                          │
    │        ──────────           ────┼────                      │
    │        (same line)              │                          │
    │                                                            │
    │   Condition: h² = ab      Condition: a + b = 0             │
    │                                                            │
    │                                                            │
    │   Acute Angle θ:                                           │
    │                                                            │
    │              2√(h² - ab)                                   │
    │   tan θ = ─────────────                                    │
    │               |a + b|                                      │
    │                                                            │
    └────────────────────────────────────────────────────────────┘
```

---

## 📝 Angle in Terms of Slopes

If you know the individual slopes $m_1$ and $m_2$:

$$\tan\theta = \left|\frac{m_1 - m_2}{1 + m_1 m_2}\right|$$

**Relationship**:
- $m_1 + m_2 = -\frac{2h}{b}$
- $m_1 \cdot m_2 = \frac{a}{b}$

---

## 💡 Tips and Insights

> 💡 **Quick Perpendicular Check**: Just add coefficients of $x^2$ and $y^2$. If sum is zero, lines are perpendicular.

> 💡 **Formula Independence**: The angle formula works for both homogeneous and general forms - linear terms don't affect the angle.

> ⚠️ **Always Take Absolute Value**: The formula gives the acute angle; use absolute value for tan θ.

> 💡 **45° Angle**: When $|a+b| = 2\sqrt{h^2-ab}$, the angle is 45°.

---

## 🌍 Real-World Applications

1. **Surveying**: Angle between boundary lines
2. **Optics**: Angle of divergence/convergence
3. **Architecture**: Roof pitch angles
4. **Engineering**: V-belt angle calculations
5. **Navigation**: Course intersection angles

---

## 📋 Summary Table

| Property | Formula/Condition |
|----------|-------------------|
| Angle formula | $\tan\theta = \left\|\frac{2\sqrt{h^2-ab}}{a+b}\right\|$ |
| Perpendicular | $a + b = 0$ |
| Coincident | $h^2 = ab$ |
| Real distinct lines | $h^2 > ab$ |
| 45° angle | $(a+b)^2 = 4(h^2 - ab)$ |

---

## ❓ Quick Revision Questions

1. **Find the angle between lines $x^2 - 7xy + 12y^2 = 0$.**

2. **Show that $5x^2 + 2xy - 5y^2 = 0$ represents perpendicular lines.**

3. **Find k if lines $x^2 + 4kxy + 4y^2 = 0$ are coincident.**

4. **The angle between lines $x^2 + 2hxy - 3y^2 = 0$ is 60°. Find h.**

5. **If lines $kx^2 + 5xy - 6y^2 = 0$ are perpendicular, find k.**

6. **Find the acute angle between lines $2x^2 + 7xy + 3y^2 = 0$.**

<details>
<summary><b>Click to see answers</b></summary>

1. $a = 1$, $h = -\frac{7}{2}$, $b = 12$
   $\tan\theta = \left|\frac{2\sqrt{\frac{49}{4} - 12}}{13}\right| = \left|\frac{2 \cdot \frac{1}{2}}{13}\right| = \frac{1}{13}$
   **$\theta = \tan^{-1}\left(\frac{1}{13}\right)$**

2. $a = 5$, $b = -5$
   $a + b = 5 - 5 = 0$ ✓
   **Lines are perpendicular**

3. For coincident: $h^2 = ab$
   $(2k)^2 = 1 \times 4$
   $4k^2 = 4$
   $k^2 = 1$
   **$k = \pm 1$**

4. $\tan 60° = \sqrt{3}$
   $\sqrt{3} = \left|\frac{2\sqrt{h^2 + 3}}{1 - 3}\right| = \left|\frac{2\sqrt{h^2 + 3}}{-2}\right| = \sqrt{h^2 + 3}$
   $3 = h^2 + 3$
   $h^2 = 0$
   **$h = 0$**

5. For perpendicular: $a + b = 0$
   $k + (-6) = 0$
   **$k = 6$**

6. $a = 2$, $h = \frac{7}{2}$, $b = 3$
   $\tan\theta = \left|\frac{2\sqrt{\frac{49}{4} - 6}}{5}\right| = \left|\frac{2\sqrt{\frac{25}{4}}}{5}\right| = \left|\frac{2 \times \frac{5}{2}}{5}\right| = 1$
   **$\theta = 45°$**

</details>

---

## ⏭️ Navigation

| [← Previous: General Second Degree](02-general-second-degree.md) | [Next: Bisectors and Applications →](04-bisectors-applications.md) |
|:----------------------------------------------------------------|------------------------------------------------------------------:|
