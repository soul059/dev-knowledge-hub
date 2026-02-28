# Chapter 5.3: Focal Chord

## 📚 Chapter Overview

A **focal chord** is any chord of a parabola that passes through its focus. Focal chords have remarkable properties that are frequently used in problems. The latus rectum is a special focal chord perpendicular to the axis.

---

## 📝 Definition

> A **focal chord** is a chord of the parabola passing through the focus.

```
                       y
                       │      
              P●───────┼───────●Q
               \       │      /
                \      │     /
                 \     │    /
                  \    ●F  /
                   \   │  /
                    \  │ /
                     \ │/
        ─────────────●V──────────► x
                     /│\
                    / │ \
                   /  │  \
                       
    PQ is a focal chord (passes through focus F)
    P and Q are the endpoints
```

---

## 📐 Parametric Representation

For parabola $y^2 = 4ax$, let:
- P be at parameter $t_1$: $P(at_1^2, 2at_1)$
- Q be at parameter $t_2$: $Q(at_2^2, 2at_2)$

### Condition for Focal Chord

If PQ passes through focus F$(a, 0)$, then:

$$\boxed{t_1 \cdot t_2 = -1}$$

### Derivation

Equation of chord PQ:
$$y - 2at_1 = \frac{2at_2 - 2at_1}{at_2^2 - at_1^2}(x - at_1^2)$$

$$y - 2at_1 = \frac{2a(t_2 - t_1)}{a(t_2^2 - t_1^2)}(x - at_1^2)$$

$$y - 2at_1 = \frac{2(t_2 - t_1)}{(t_2 + t_1)(t_2 - t_1)}(x - at_1^2)$$

$$y - 2at_1 = \frac{2}{t_1 + t_2}(x - at_1^2)$$

For this to pass through F$(a, 0)$:
$$0 - 2at_1 = \frac{2}{t_1 + t_2}(a - at_1^2)$$

$$-2at_1 = \frac{2a(1 - t_1^2)}{t_1 + t_2}$$

$$-t_1(t_1 + t_2) = 1 - t_1^2 = (1-t_1)(1+t_1)$$

$$-t_1 t_1 - t_1 t_2 = 1 - t_1^2$$

$$-t_1^2 - t_1 t_2 = 1 - t_1^2$$

$$-t_1 t_2 = 1$$

$$\boxed{t_1 t_2 = -1}$$

---

## ⚡ Key Properties of Focal Chords

### 1. Product of Parameters

$$t_1 t_2 = -1$$

If one endpoint has parameter $t$, the other has parameter $-\frac{1}{t}$.

---

### 2. Length of Focal Chord

For focal chord with endpoints at parameters $t$ and $-\frac{1}{t}$:

$$\boxed{\text{Length} = a\left(t + \frac{1}{t}\right)^2}$$

#### Derivation

$P = (at^2, 2at)$ and $Q = \left(\frac{a}{t^2}, -\frac{2a}{t}\right)$

$$PQ = \sqrt{(at^2 - \frac{a}{t^2})^2 + (2at + \frac{2a}{t})^2}$$

$$= \sqrt{a^2(t^2 - \frac{1}{t^2})^2 + 4a^2(t + \frac{1}{t})^2}$$

$$= a\sqrt{(t - \frac{1}{t})^2(t + \frac{1}{t})^2 + 4(t + \frac{1}{t})^2}$$

$$= a(t + \frac{1}{t})\sqrt{(t - \frac{1}{t})^2 + 4}$$

$$= a(t + \frac{1}{t})\sqrt{t^2 - 2 + \frac{1}{t^2} + 4}$$

$$= a(t + \frac{1}{t})\sqrt{t^2 + 2 + \frac{1}{t^2}}$$

$$= a(t + \frac{1}{t}) \cdot (t + \frac{1}{t})$$

$$= a\left(t + \frac{1}{t}\right)^2$$

---

### 3. Semi-Latus Rectum as Harmonic Mean

For focal chord with endpoints P and Q:

Let SP = l₁ and SQ = l₂ (focal distances)

Then:
$$\boxed{\frac{2}{l_1} + \frac{2}{l_2} = \frac{1}{a} \text{ i.e., Semi-LR is harmonic mean}}$$

Or equivalently:
$$\frac{1}{SP} + \frac{1}{SQ} = \frac{1}{a}$$

---

### 4. Focal Distances

For endpoints of focal chord at parameters $t$ and $-\frac{1}{t}$:

| Endpoint | Focal Distance |
|----------|---------------|
| $P(at^2, 2at)$ | $a(1 + t^2)$ |
| $Q(\frac{a}{t^2}, -\frac{2a}{t})$ | $a(1 + \frac{1}{t^2})$ |

**Sum of focal distances**:
$$SP + SQ = a\left(t + \frac{1}{t}\right)^2 = \text{Length of focal chord}$$

---

### 5. Minimum Length

The minimum length of a focal chord equals the **latus rectum = 4a**.

This occurs when $t + \frac{1}{t}$ is minimum.

Since $\left(t + \frac{1}{t}\right)^2 \geq 4$ (by AM-GM), minimum value = 4 when $t = \pm 1$.

---

## 📝 Equation of Focal Chord

### Using Two Parameters

For focal chord through points with parameters $t_1$ and $t_2$ (where $t_1 t_2 = -1$):

$$\boxed{y(t_1 + t_2) = 2x + 2at_1 t_2}$$

Substituting $t_1 t_2 = -1$:
$$y(t_1 + t_2) = 2(x - a)$$

---

### Using Single Parameter t

If one endpoint has parameter $t$:

$$y = \frac{2(x - a)}{t - \frac{1}{t}} = \frac{2t(x - a)}{t^2 - 1}$$

---

## ✏️ Worked Examples

### Example 1: Finding Other Endpoint

**Problem**: One endpoint of a focal chord of $y^2 = 8x$ is $(8, 8)$. Find the other endpoint.

**Solution**:

Here $4a = 8$, so $a = 2$.

Point $(8, 8)$: Let parameter be $t$.
$(at^2, 2at) = (8, 8)$

From $2at = 8$: $4t = 8$, so $t = 2$

For focal chord: $t_1 t_2 = -1$
$2 \cdot t_2 = -1$
$t_2 = -\frac{1}{2}$

Other endpoint: $\left(a t_2^2, 2a t_2\right) = \left(2 \cdot \frac{1}{4}, 2 \cdot 2 \cdot (-\frac{1}{2})\right) = \left(\frac{1}{2}, -2\right)$

**Answer**: $\left(\frac{1}{2}, -2\right)$

---

### Example 2: Length of Focal Chord

**Problem**: Find the length of focal chord of $y^2 = 16x$ whose one end is at parameter $t = 2$.

**Solution**:

$a = 4$

Length = $a\left(t + \frac{1}{t}\right)^2 = 4\left(2 + \frac{1}{2}\right)^2 = 4 \times \frac{25}{4} = 25$

**Answer**: 25 units

---

### Example 3: Focal Chord with Given Slope

**Problem**: Find the length of focal chord of $y^2 = 4ax$ with slope 2.

**Solution**:

Slope of chord PQ = $\frac{2at_1 - 2at_2}{at_1^2 - at_2^2} = \frac{2}{t_1 + t_2} = 2$

So $t_1 + t_2 = 1$

Also, $t_1 t_2 = -1$

Length = $a(t_1 + t_2)^2 + 4at_1t_2 = a[(t_1 + t_2)^2 - 4t_1t_2] \cdot \frac{(t_1+t_2)^2}{(t_1-t_2)^2}$...

Actually, simpler approach:
$(t_1 - t_2)^2 = (t_1 + t_2)^2 - 4t_1t_2 = 1 + 4 = 5$

Length = $a(t_1 - t_2)^2(1 + \frac{4}{(t_1+t_2)^2})$... 

Let me use the direct formula:
Length = $a\left(t + \frac{1}{t}\right)^2$ where $t$ is one parameter.

Since $t_1 + t_2 = 1$ and $t_1 t_2 = -1$:
$t_1, t_2$ are roots of $x^2 - x - 1 = 0$
$t = \frac{1 + \sqrt{5}}{2}$ (taking positive root)

$t + \frac{1}{t} = t + (-t_2) = t - t_2$... Actually $\frac{1}{t} = -t_2$, so:
$t + \frac{1}{t} = t_1 - t_2 = \pm\sqrt{5}$

Length = $a \times 5 = 5a$

**Answer**: $5a$ units

---

### Example 4: Focal Chord as Diameter

**Problem**: A focal chord of $y^2 = 4ax$ is taken as diameter of a circle. Show that the circle touches the directrix.

**Solution**:

Let endpoints be $P(at^2, 2at)$ and $Q(\frac{a}{t^2}, -\frac{2a}{t})$.

Center of circle = Midpoint of PQ:
$$C = \left(\frac{a(t^2 + \frac{1}{t^2})}{2}, \frac{a(t - \frac{1}{t})}{2} \cdot 2\right) = \left(\frac{a(t^4 + 1)}{2t^2}, a\left(t - \frac{1}{t}\right)\right)$$

Wait, let me recalculate:
$$C_x = \frac{at^2 + \frac{a}{t^2}}{2} = \frac{a(t^4 + 1)}{2t^2}$$

$$C_y = \frac{2at - \frac{2a}{t}}{2} = a\left(t - \frac{1}{t}\right)$$

Diameter = Length of focal chord = $a\left(t + \frac{1}{t}\right)^2$

Radius = $\frac{a}{2}\left(t + \frac{1}{t}\right)^2$

Distance from center to directrix $x = -a$:
$$C_x - (-a) = \frac{a(t^4 + 1)}{2t^2} + a = \frac{a(t^4 + 1) + 2at^2}{2t^2} = \frac{a(t^4 + 2t^2 + 1)}{2t^2} = \frac{a(t^2 + 1)^2}{2t^2}$$

$$= \frac{a}{2}\left(t + \frac{1}{t}\right)^2 = \text{Radius}$$

Since distance to directrix = radius, **the circle touches the directrix**. ∎

---

### Example 5: Locus Problem

**Problem**: Find the locus of the midpoint of a focal chord of $y^2 = 4ax$.

**Solution**:

Let endpoints be $(at_1^2, 2at_1)$ and $(at_2^2, 2at_2)$ where $t_1 t_2 = -1$.

Midpoint $(h, k)$:
$$h = \frac{a(t_1^2 + t_2^2)}{2}$$
$$k = \frac{2a(t_1 + t_2)}{2} = a(t_1 + t_2)$$

So $t_1 + t_2 = \frac{k}{a}$

Also: $t_1^2 + t_2^2 = (t_1 + t_2)^2 - 2t_1 t_2 = \frac{k^2}{a^2} + 2$

Thus: $h = \frac{a}{2}\left(\frac{k^2}{a^2} + 2\right) = \frac{k^2}{2a} + a$

$$h - a = \frac{k^2}{2a}$$

$$k^2 = 2a(h - a)$$

Replacing $(h, k)$ with $(x, y)$:

**Locus**: $y^2 = 2a(x - a)$

This is a parabola with vertex at $(a, 0)$ (the focus of original parabola)!

---

### Example 6: Product Property

**Problem**: If PSQ is a focal chord of $y^2 = 4ax$ (S is focus), prove that $\frac{1}{PS} + \frac{1}{SQ} = \frac{1}{a}$.

**Solution**:

Let $P = (at^2, 2at)$ and $Q = (\frac{a}{t^2}, -\frac{2a}{t})$

$PS = a + at^2 = a(1 + t^2)$ (focal distance)

$SQ = a + \frac{a}{t^2} = \frac{a(t^2 + 1)}{t^2}$

$$\frac{1}{PS} + \frac{1}{SQ} = \frac{1}{a(1+t^2)} + \frac{t^2}{a(1+t^2)}$$

$$= \frac{1 + t^2}{a(1+t^2)} = \frac{1}{a}$$

**Proved.** ∎

---

## 💡 Tips and Insights

> 💡 **Quick Check**: For focal chord, endpoints have parameters $t$ and $-\frac{1}{t}$.

> 💡 **Minimum Focal Chord**: The latus rectum (length = 4a) is the shortest focal chord.

> ⚠️ **Sign**: If $t > 0$, the other endpoint is in the opposite quadrant (negative y).

> 💡 **Circle Property**: Circle with focal chord as diameter always touches the directrix.

---

## 📋 Summary Table

| Property | Formula |
|----------|---------|
| Focal chord condition | $t_1 t_2 = -1$ |
| If one parameter is $t$ | Other is $-\frac{1}{t}$ |
| Length of focal chord | $a\left(t + \frac{1}{t}\right)^2$ |
| Minimum length | $4a$ (latus rectum) |
| Harmonic mean property | $\frac{1}{PS} + \frac{1}{SQ} = \frac{1}{a}$ |
| Midpoint locus | $y^2 = 2a(x - a)$ |
| Focal chord slope | $\frac{2}{t_1 + t_2}$ |

---

## ❓ Quick Revision Questions

1. **If one end of a focal chord of $y^2 = 12x$ is $(3, 6)$, find the other end.**

2. **Find the length of focal chord of $y^2 = 8x$ at parameter $t = 3$.**

3. **Prove that the minimum length of a focal chord is the latus rectum.**

4. **For $y^2 = 16x$, find the focal chord with slope $\frac{1}{2}$.**

5. **A focal chord makes angle 60° with axis. Find its length for $y^2 = 4ax$.**

6. **Show that for focal chord PSQ, $PS \cdot SQ = (\text{semi-latus rectum})^2$ is FALSE and find correct relation.**

<details>
<summary><b>Click to see answers</b></summary>

1. $4a = 12$ → $a = 3$
   Point $(3, 6) = (at^2, 2at) = (3t^2, 6t)$
   $6t = 6$ → $t = 1$
   Other parameter: $t' = -1$
   Other end: $(3 \times 1, 6 \times (-1)) = (3, -6)$
   **Other endpoint: $(3, -6)$**

2. $a = 2$
   Length = $a(t + \frac{1}{t})^2 = 2(3 + \frac{1}{3})^2 = 2 \times \frac{100}{9} = \frac{200}{9}$
   **Length = $\frac{200}{9}$ units**

3. Length = $a(t + \frac{1}{t})^2$
   By AM-GM: $t + \frac{1}{t} \geq 2$ when $t > 0$
   (or $t + \frac{1}{t} \leq -2$ when $t < 0$)
   Minimum $(t + \frac{1}{t})^2 = 4$, when $t = \pm 1$
   **Minimum length = $4a$ = Latus rectum** ∎

4. Slope = $\frac{2}{t_1 + t_2} = \frac{1}{2}$
   $t_1 + t_2 = 4$, $t_1 t_2 = -1$
   From $t^2 - 4t - 1 = 0$: $t = \frac{4 \pm \sqrt{20}}{2} = 2 \pm \sqrt{5}$
   Focal chord passes through focus $(4, 0)$ with slope $\frac{1}{2}$:
   **$y = \frac{1}{2}(x - 4)$ or $x - 2y - 4 = 0$**

5. Slope = $\tan 60° = \sqrt{3}$
   $\frac{2}{t_1 + t_2} = \sqrt{3}$
   $t_1 + t_2 = \frac{2}{\sqrt{3}}$
   $(t_1 - t_2)^2 = (t_1 + t_2)^2 - 4t_1t_2 = \frac{4}{3} + 4 = \frac{16}{3}$
   $(t + \frac{1}{t})^2 = (t_1 - t_2)^2 = \frac{16}{3}$
   **Length = $\frac{16a}{3}$**

6. $PS = a(1 + t^2)$, $SQ = a(1 + \frac{1}{t^2}) = \frac{a(t^2+1)}{t^2}$
   $PS \cdot SQ = \frac{a^2(1+t^2)^2}{t^2}$
   This is NOT constant (depends on t).
   **Correct relation**: $\frac{1}{PS} + \frac{1}{SQ} = \frac{1}{a}$
   (Semi-latus rectum $2a$ is the harmonic mean of $PS$ and $SQ$)

</details>

---

## ⏭️ Navigation

| [← Previous: Elements of Parabola](02-elements.md) | [Next: Equation of Tangent →](04-equation-of-tangent.md) |
|:--------------------------------------------------|----------------------------------------------------------:|
