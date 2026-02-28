# Unit 8 — Applications of Derivatives: Mean Value Theorems

[← Previous: Curve Sketching](06-curve-sketching.md) · [Next: Unit 9 — Taylor Series →](../09-Taylor-Maclaurin/01-taylor-series.md)

---

## Chapter Overview

The **Mean Value Theorems** are among the most important theoretical results in calculus. They connect the average behavior of a function over an interval to its instantaneous behavior at some point within that interval.

---

## 8.7.1 Rolle's Theorem

> **Rolle's Theorem:** If $f$ is:
> 1. Continuous on $[a, b]$
> 2. Differentiable on $(a, b)$
> 3. $f(a) = f(b)$
>
> Then there exists at least one $c \in (a, b)$ such that $f'(c) = 0$.

```
  y
  |         * (c, f(c))
  |        / \       f'(c) = 0 (horizontal tangent)
  |       /   \
  |      /     \
  f(a) *         * f(b)     f(a) = f(b)
  |    a    c    b
  +──────────────────→ x
```

**Geometric meaning:** If a curve starts and ends at the same height, somewhere in between there must be a horizontal tangent.

### Proof Sketch

By EVT, $f$ attains a maximum $M$ and minimum $m$ on $[a,b]$.

- If $M = m$: $f$ is constant, so $f'(c) = 0$ for all $c \in (a,b)$.
- If $M > m$: At least one of $M, m$ is attained at an interior point $c$. By Fermat's theorem, $f'(c) = 0$. $\blacksquare$

### Example: $f(x) = x^2 - 4x + 3$ on $[1, 3]$

Check: $f(1) = 1 - 4 + 3 = 0$, $f(3) = 9 - 12 + 3 = 0$ ✓

$f'(x) = 2x - 4 = 0 \Rightarrow x = 2 \in (1, 3)$ ✓

---

## 8.7.2 Lagrange's Mean Value Theorem (MVT)

> **MVT:** If $f$ is:
> 1. Continuous on $[a, b]$
> 2. Differentiable on $(a, b)$
>
> Then there exists at least one $c \in (a, b)$ such that:
>
> $$f'(c) = \frac{f(b) - f(a)}{b - a}$$

```
  y
  |              B * f(b)
  |             /.:
  |            / . :
  |        * /  .  :    ← Tangent at c is parallel
  |       / *  .   :       to the secant AB
  |      /  . .    :
  | A * /  .       :
  |  f(a) .        :
  +───+───+────+───+→ x
      a   c        b

  slope of secant AB = slope of tangent at c
```

**Geometric meaning:** There is a point where the instantaneous rate equals the average rate.

**Physical meaning:** If a car travels 100 km in 2 hours (average 50 km/h), at some moment its speedometer must read exactly 50 km/h.

### Proof Sketch

Define $g(x) = f(x) - f(a) - \frac{f(b)-f(a)}{b-a}(x-a)$.

Then $g(a) = 0$ and $g(b) = 0$. By Rolle's theorem, $\exists\, c$ with $g'(c) = 0$:

$$f'(c) - \frac{f(b)-f(a)}{b-a} = 0 \Rightarrow f'(c) = \frac{f(b)-f(a)}{b-a} \quad \blacksquare$$

---

## 8.7.3 Consequences of MVT

### 1. If $f'(x) = 0$ for all $x \in (a,b)$, then $f$ is constant

For any $x_1, x_2 \in (a,b)$: by MVT, $f(x_2) - f(x_1) = f'(c)(x_2 - x_1) = 0$.

### 2. If $f'(x) = g'(x)$ for all $x$, then $f(x) = g(x) + C$

Apply result 1 to $h = f - g$.

### 3. If $f'(x) > 0$ for all $x \in (a,b)$, then $f$ is strictly increasing

For $x_1 < x_2$: $f(x_2) - f(x_1) = f'(c)(x_2 - x_1) > 0$.

---

## 8.7.4 Worked Examples

### Example 1: Verify MVT for $f(x) = x^3$ on $[1, 3]$

$$\frac{f(3) - f(1)}{3 - 1} = \frac{27 - 1}{2} = 13$$

$$f'(c) = 3c^2 = 13 \Rightarrow c = \sqrt{13/3} \approx 2.08 \in (1, 3) \quad ✓$$

### Example 2: Prove $|\sin a - \sin b| \leq |a - b|$

By MVT: $\sin a - \sin b = \cos c \cdot (a - b)$ for some $c$.

$$|\sin a - \sin b| = |\cos c| \cdot |a - b| \leq |a - b|$$

since $|\cos c| \leq 1$. $\blacksquare$

### Example 3: Show $\frac{a-b}{a} < \ln\frac{a}{b} < \frac{a-b}{b}$ for $0 < b < a$

Apply MVT to $f(x) = \ln x$ on $[b, a]$:

$$\frac{\ln a - \ln b}{a - b} = \frac{1}{c} \quad \text{for some } c \in (b, a)$$

Since $b < c < a$: $\frac{1}{a} < \frac{1}{c} < \frac{1}{b}$

$$\frac{1}{a} < \frac{\ln a - \ln b}{a - b} < \frac{1}{b}$$

$$\frac{a-b}{a} < \ln\frac{a}{b} < \frac{a-b}{b} \quad \blacksquare$$

---

## 8.7.5 Cauchy's Mean Value Theorem (Generalized MVT)

> If $f$ and $g$ are continuous on $[a,b]$, differentiable on $(a,b)$, and $g'(x) \neq 0$ in $(a,b)$, then:
>
> $$\frac{f'(c)}{g'(c)} = \frac{f(b) - f(a)}{g(b) - g(a)}$$
>
> for some $c \in (a,b)$.

- Setting $g(x) = x$ recovers the standard MVT.
- Cauchy's MVT is the key tool used to **prove L'Hôpital's Rule**.

---

## 8.7.6 Relationship Between the Theorems

```
  ┌──────────────────────────────────────────────┐
  │                                              │
  │   Rolle's Theorem                            │
  │   (f(a) = f(b), conclude f'(c) = 0)        │
  │         ↕  (special case)                    │
  │   Mean Value Theorem (Lagrange)              │
  │   (conclude f'(c) = [f(b)-f(a)]/(b-a))     │
  │         ↕  (special case: g(x) = x)         │
  │   Cauchy's Mean Value Theorem               │
  │   (conclude f'(c)/g'(c) = Δf/Δg)           │
  │         ↓  (used to prove)                   │
  │   L'Hôpital's Rule                          │
  │                                              │
  └──────────────────────────────────────────────┘
```

---

## 8.7.7 Applications of MVT

| Application | Method |
|-------------|--------|
| Proving inequalities | Apply MVT, bound $f'(c)$ |
| Showing uniqueness of roots | MVT + Rolle's (assume two roots → contradiction) |
| Estimating function values | $f(b) \approx f(a) + f'(c)(b-a)$ |
| Proving derivative properties | $f' = 0$ everywhere → $f$ constant |

---

## Summary Table

| Theorem | Hypotheses | Conclusion |
|---------|-----------|------------|
| **Rolle** | Cont. on $[a,b]$, diff. on $(a,b)$, $f(a)=f(b)$ | $\exists\,c: f'(c)=0$ |
| **Lagrange MVT** | Cont. on $[a,b]$, diff. on $(a,b)$ | $\exists\,c: f'(c)=\frac{f(b)-f(a)}{b-a}$ |
| **Cauchy MVT** | Both cont. on $[a,b]$, diff. on $(a,b)$, $g'\neq 0$ | $\exists\,c: \frac{f'(c)}{g'(c)}=\frac{f(b)-f(a)}{g(b)-g(a)}$ |

---

## Quick Revision Questions

**Q1.** Verify Rolle's theorem for $f(x) = x(x-1)(x-2)$ on $[0, 2]$.

**Q2.** Find the value of $c$ in MVT for $f(x) = \sqrt{x}$ on $[4, 9]$.

**Q3.** Use MVT to prove that $|\cos a - \cos b| \leq |a - b|$.

**Q4.** Prove that $e^x \geq 1 + x$ for all $x \geq 0$ using MVT.

**Q5.** State Cauchy's MVT and show how Lagrange's MVT is a special case.

**Q6.** Using Rolle's theorem, show that $f(x) = x^3 + x - 1$ has exactly one real root.

---

[← Previous: Curve Sketching](06-curve-sketching.md) · [Next: Unit 9 — Taylor Series →](../09-Taylor-Maclaurin/01-taylor-series.md)
