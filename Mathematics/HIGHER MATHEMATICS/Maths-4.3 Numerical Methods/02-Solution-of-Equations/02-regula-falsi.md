# Chapter 2.2: Regula Falsi (False Position Method)

[← Previous: Bisection Method](01-bisection-method.md) | [Next: Newton-Raphson Method →](03-newton-raphson.md)

---

## Overview

The **Regula Falsi** (False Position) method improves upon bisection by replacing the midpoint with a **linear interpolation** between $(a, f(a))$ and $(b, f(b))$. Instead of blindly halving the interval, it uses the function values to make a smarter guess — the point where the line connecting the two endpoints crosses the x-axis.

---

## 1. Geometric Idea

```
  f(x)
    │      ● (a, f(a))
    │     ╱╲
    │    ╱  ╲     True curve
    │   ╱    ╲
    │  ╱      ╲
  ──┼─╱────────╳──────────── x
    │a       c  ╲       b
    │             ╲    ╱
    │              ╲  ╱
    │               ╲╱
    │                ● (b, f(b))
    │
    │  The line from (a,f(a)) to (b,f(b))
    │  crosses x-axis at c = Regula Falsi guess
```

---

## 2. Derivation of the Formula

The straight line through $(a, f(a))$ and $(b, f(b))$ is:

$$y - f(a) = \frac{f(b) - f(a)}{b - a}(x - a)$$

Setting $y = 0$ and solving for $x$:

$$c = a - f(a) \cdot \frac{b - a}{f(b) - f(a)}$$

Or equivalently:

$$\boxed{c = \frac{a \cdot f(b) - b \cdot f(a)}{f(b) - f(a)}}$$

---

## 3. Algorithm

1. **Choose** $[a, b]$ with $f(a) \cdot f(b) < 0$
2. **Compute**: $c = \frac{a \cdot f(b) - b \cdot f(a)}{f(b) - f(a)}$
3. **Evaluate** $f(c)$
4. **Decide**:
   - If $|f(c)| < \epsilon$ or $|c_{\text{new}} - c_{\text{old}}| < \epsilon$ → **Stop**
   - If $f(a) \cdot f(c) < 0$ → set $b = c$
   - If $f(c) \cdot f(b) < 0$ → set $a = c$
5. **Repeat** from Step 2

```
  ┌──────────────────────────┐
  │  Input: f, a, b, ε, Nmax │
  └────────────┬─────────────┘
               ▼
  ┌──────────────────────────┐
  │  c = [af(b)-bf(a)]      │◄──────────┐
  │      / [f(b)-f(a)]      │           │
  └────────────┬─────────────┘           │
               ▼                          │
  ┌──────────────────────────┐           │
  │  |f(c)| < ε ?           │──Yes──►STOP│
  └────────────┬─────────────┘           │
               │No                        │
               ▼                          │
  ┌──────────────────────────┐           │
  │  f(a)·f(c) < 0?         │           │
  │  Yes → b=c   No → a=c   │───────────┘
  └──────────────────────────┘
```

---

## 4. Worked Example

Find a root of $f(x) = x^3 - x - 2$ in $[1, 2]$.

$f(1) = -2$, $f(2) = 4$

| Iter | $a$ | $b$ | $f(a)$ | $f(b)$ | $c$ | $f(c)$ | New Interval |
|:----:|:---:|:---:|:------:|:------:|:---:|:------:|:------------:|
| 1 | 1.0 | 2.0 | $-2$ | $4$ | $1.3333$ | $-0.9630$ | $[1.3333, 2.0]$ |
| 2 | 1.3333 | 2.0 | $-0.963$ | $4$ | $1.4545$ | $-0.3788$ | $[1.4545, 2.0]$ |
| 3 | 1.4545 | 2.0 | $-0.379$ | $4$ | $1.4928$ | $-0.1629$ | $[1.4928, 2.0]$ |
| 4 | 1.4928 | 2.0 | $-0.163$ | $4$ | $1.5090$ | $-0.0680$ | $[1.5090, 2.0]$ |
| 5 | 1.5090 | 2.0 | $-0.068$ | $4$ | $1.5157$ | $-0.0281$ | $[1.5157, 2.0]$ |

The method converges toward $r \approx 1.5214$.

**Notice**: One endpoint ($b = 2$) remains **fixed** — this is a known drawback.

---

## 5. Bisection vs. Regula Falsi

```
  Bisection:                    Regula Falsi:
  
  f(x)                          f(x)
    │   ●                         │   ●
    │  ╱╲                        │  ╱╲
    │ ╱  ╲                       │ ╱  ╲
  ──┼╱──●─╲──── x              ──┼╱────╳──── x
    │a  c  ╲  b                  │a    c ╲  b
    │       ╲╱                   │        ╲╱
    │        ●                   │         ●
    │                            │
    │  c = midpoint              │  c = x-intercept of line
    │  (ignores f values)        │  (uses f values)
```

| Feature | Bisection | Regula Falsi |
|---------|-----------|--------------|
| Formula | $c = (a+b)/2$ | $c = \frac{af(b)-bf(a)}{f(b)-f(a)}$ |
| Uses function values? | Only signs | Yes, magnitudes too |
| Convergence | Always linear | Usually faster than bisection |
| Guaranteed? | Yes | Yes (brackets root) |
| Problem | None (slow but steady) | One endpoint may get "stuck" |

---

## 6. The Stagnation Problem

In Regula Falsi, one endpoint often remains fixed across many iterations, causing slow convergence:

```
  Iteration 1:  [a₀ ─────────────────── b₀]     c₁ = ──●
  Iteration 2:  [a₀ ─────────────── b₁=c₁]      c₂ = ──●
  Iteration 3:  [a₀ ─────────── b₂=c₂]          c₃ = ──●
  Iteration 4:  [a₀ ─────── b₃=c₃]              c₄ = ──●
     ...
  
  The left endpoint a₀ stays FIXED!
  The method "creeps" from one side.
```

### Modified Regula Falsi (Illinois Method)

If the same endpoint is retained, halve its function value:

- If $a$ is retained: use $f(a)/2$ in the next iteration
- This forces the stagnant endpoint to move eventually

---

## 7. Convergence

- **Order of convergence**: Superlinear ($\approx 1.618$ for simple roots under ideal conditions)
- In practice, due to stagnation, convergence can be **slower than bisection** in some cases
- The method is guaranteed to converge (it's a bracketing method)

---

## 8. Real-World Applications

| Application | Why Regula Falsi |
|-------------|-----------------|
| Initial guess refinement | Use before switching to Newton-Raphson |
| Embedded systems | Simple to implement, no derivatives needed |
| Safety-critical calculations | Guaranteed convergence with bracket |
| Pipe flow calculations | Solving Colebrook equation for friction factor |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Bracketing method |
| **Formula** | $c = \frac{af(b) - bf(a)}{f(b) - f(a)}$ |
| **Convergence** | Guaranteed, usually faster than bisection |
| **Order** | Superlinear (~1.618) in ideal case |
| **Derivatives** | Not needed |
| **Drawback** | Stagnation (one fixed endpoint) |
| **Fix** | Modified Regula Falsi / Illinois method |

---

## Quick Revision Questions

1. **Derive the Regula Falsi formula from the equation of the secant line.**

2. **Apply Regula Falsi to find a root of $f(x) = x \cdot e^x - 1$ in $[0, 1]$. Do 3 iterations.**

3. **What is the stagnation problem? How does the Illinois modification address it?**

4. **Compare bisection and Regula Falsi for $f(x) = x^2 - 3$ on $[1, 2]$. Which converges faster in the first 5 iterations?**

5. **Can Regula Falsi diverge? Justify your answer.**

6. **When would you prefer Regula Falsi over Newton-Raphson?**

---

[← Previous: Bisection Method](01-bisection-method.md) | [Next: Newton-Raphson Method →](03-newton-raphson.md)
