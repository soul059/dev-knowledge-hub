# Unit 8 — Applications of Derivatives: Rate of Change

[← Previous: Applications of Higher-Order Derivatives](../07-Higher-Order/03-applications.md) · [Next: Tangent & Normal Lines →](02-tangent-normal.md)

---

## Chapter Overview

The derivative $\frac{dy}{dx}$ is fundamentally the **instantaneous rate of change** of $y$ with respect to $x$. This chapter applies this concept to real-world problems: velocity, growth rates, related rates, and more.

---

## 8.1.1 Average vs. Instantaneous Rate of Change

**Average rate of change** over $[a, b]$:

$$\text{Average rate} = \frac{f(b) - f(a)}{b - a}$$

**Instantaneous rate of change** at $x = a$:

$$\text{Instantaneous rate} = f'(a) = \lim_{h \to 0}\frac{f(a+h) - f(a)}{h}$$

```
  y
  |        . B
  |       /.
  |      / .   Secant line = average rate
  |     /  .
  |    . A .
  |   /:   .
  |  / :   .
  | /  :   .   Tangent at A = instantaneous rate
  +----+---+----→ x
       a   b
```

---

## 8.1.2 Motion Along a Line

If $s(t)$ is the position of a particle at time $t$:

| Quantity | Formula | Meaning |
|----------|---------|---------|
| Velocity | $v(t) = s'(t) = \frac{ds}{dt}$ | Rate of change of position |
| Speed | $\|v(t)\|$ | Magnitude of velocity |
| Acceleration | $a(t) = v'(t) = s''(t)$ | Rate of change of velocity |
| Jerk | $j(t) = a'(t) = s'''(t)$ | Rate of change of acceleration |

### Example: $s(t) = t^3 - 6t^2 + 9t + 2$

$$v(t) = 3t^2 - 12t + 9 = 3(t-1)(t-3)$$

$$a(t) = 6t - 12 = 6(t-2)$$

**At rest:** $v(t) = 0 \Rightarrow t = 1, 3$

**Zero acceleration:** $a(t) = 0 \Rightarrow t = 2$

```
  v(t)
  9 +*
    | \
    |  \
  0 +---*-----*---→ t
    |   1  2  3
    |      \  /
 -3 +       *

  Particle moves right (v > 0) for t < 1,
  moves left (v < 0) for 1 < t < 3,
  moves right again for t > 3.
```

---

## 8.1.3 Related Rates

When two or more quantities change with time and are related by an equation, differentiating that equation gives a **related rates** equation.

**Method:**

```
  ┌──────────────────────────────────────┐
  │ 1. Draw a diagram                   │
  │ 2. Identify variables and rates     │
  │ 3. Write the relating equation      │
  │ 4. Differentiate both sides w.r.t. t│
  │ 5. Substitute known values          │
  │ 6. Solve for unknown rate           │
  └──────────────────────────────────────┘
```

### Example 1: Expanding Circle

A stone dropped into still water creates circular ripples. The radius increases at $2$ cm/s. Find the rate of increase of area when the radius is $10$ cm.

Given: $\frac{dr}{dt} = 2$ cm/s. Find: $\frac{dA}{dt}$ when $r = 10$.

$$A = \pi r^2$$

$$\frac{dA}{dt} = 2\pi r \cdot \frac{dr}{dt} = 2\pi(10)(2) = 40\pi \text{ cm}^2/\text{s}$$

### Example 2: Ladder Problem

A 10 m ladder leans against a wall. The foot slides away at 1 m/s. How fast is the top sliding down when the foot is 6 m from the wall?

```
  Wall
  |╲                x² + y² = 100
  | ╲  10 m
  |  ╲
  | y ╲
  |    ╲
  +─────╲────→
     x
```

$$x^2 + y^2 = 100$$

$$2x\frac{dx}{dt} + 2y\frac{dy}{dt} = 0$$

When $x = 6$: $y = \sqrt{100 - 36} = 8$

$$2(6)(1) + 2(8)\frac{dy}{dt} = 0 \Rightarrow \frac{dy}{dt} = -\frac{3}{4} \text{ m/s}$$

The top slides down at $\frac{3}{4}$ m/s.

### Example 3: Conical Tank

Water flows into a cone (semi-angle $30°$) at $5$ cm³/s. Find how fast the water level rises when the depth is $10$ cm.

At depth $h$: radius $r = h\tan 30° = \frac{h}{\sqrt{3}}$

$$V = \frac{1}{3}\pi r^2 h = \frac{1}{3}\pi \cdot \frac{h^2}{3} \cdot h = \frac{\pi h^3}{9}$$

$$\frac{dV}{dt} = \frac{\pi h^2}{3}\frac{dh}{dt}$$

$$5 = \frac{\pi(100)}{3}\frac{dh}{dt} \Rightarrow \frac{dh}{dt} = \frac{15}{100\pi} = \frac{3}{20\pi} \text{ cm/s}$$

---

## 8.1.4 Marginal Analysis (Economics)

| Term | Definition |
|------|-----------|
| Marginal cost | $MC = C'(x)$ = rate of change of cost |
| Marginal revenue | $MR = R'(x)$ = rate of change of revenue |
| Marginal profit | $MP = P'(x) = R'(x) - C'(x)$ |

**Profit is maximized** when $MR = MC$, i.e., $R'(x) = C'(x)$.

### Example

$C(x) = 0.01x^3 - 0.6x^2 + 13x + 100$

$MC = C'(x) = 0.03x^2 - 1.2x + 13$

$MC$ at $x = 10$: $0.03(100) - 1.2(10) + 13 = 3 - 12 + 13 = 4$

The 11th unit costs approximately $4 to produce.

---

## 8.1.5 Percentage Rate of Change

$$\text{Percentage rate} = \frac{f'(x)}{f(x)} \times 100\%$$

This is related to the **logarithmic derivative**: $\frac{d}{dx}[\ln f(x)] = \frac{f'(x)}{f(x)}$

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Instantaneous rate of change | $f'(a)$ |
| Velocity | $v = ds/dt$ |
| Acceleration | $a = dv/dt = d^2s/dt^2$ |
| Related rates | Differentiate relating equation w.r.t. $t$ |
| Marginal cost | $MC = C'(x)$ |
| Percentage rate | $\frac{f'}{f} \times 100\%$ |

---

## Quick Revision Questions

**Q1.** A particle moves with $s(t) = 2t^3 - 9t^2 + 12t$. Find when it is at rest and its acceleration at those times.

**Q2.** A spherical balloon is inflated at $100$ cm³/s. Find the rate of increase of the radius when $r = 5$ cm.

**Q3.** A point moves on $y = x^2$. If $dx/dt = 3$ units/s, find $dy/dt$ when $x = 2$.

**Q4.** Two ships leave a port: one north at 3 km/h, one east at 4 km/h. How fast is the distance between them changing after 2 hours?

**Q5.** If the revenue function is $R(x) = 100x - 0.5x^2$, find the marginal revenue at $x = 50$.

**Q6.** A 6 m ladder slides along the floor. When the base is 3 m from the wall and moving at 0.5 m/s, how fast is the angle between the ladder and floor decreasing?

---

[← Previous: Applications of Higher-Order Derivatives](../07-Higher-Order/03-applications.md) · [Next: Tangent & Normal Lines →](02-tangent-normal.md)
