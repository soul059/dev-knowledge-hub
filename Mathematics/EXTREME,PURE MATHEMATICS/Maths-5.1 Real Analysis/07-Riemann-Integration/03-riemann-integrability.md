# Chapter 3: Riemann Integrability Criteria

[← Previous: Upper and Lower Integrals](02-upper-and-lower-integrals.md) | [Back to README](../README.md) | [Next: Properties of the Integral →](04-properties-of-integral.md)

---

## Overview

Which functions are Riemann integrable? Here we develop the main integrability criteria: the **ε-criterion** (already seen), **continuous functions**, **monotone functions**, **bounded functions with finitely many discontinuities**, and the complete characterization via **Lebesgue's criterion** (measure zero).

---

## 3.1 Recap: ε-Criterion

$f: [a,b] \to \mathbb{R}$ bounded is Riemann integrable iff:

$$\forall\, \varepsilon > 0,\; \exists\, P: \; U(f,P) - L(f,P) < \varepsilon$$

Equivalently, using the **oscillation** on $[x_{k-1}, x_k]$:

$$\omega_k(f) = M_k - m_k = \sup_{[x_{k-1},x_k]} f - \inf_{[x_{k-1},x_k]} f$$

The ε-criterion becomes: $\sum_{k=1}^n \omega_k(f)\, \Delta x_k < \varepsilon$.

---

## 3.2 Continuous Functions are Integrable

**Theorem 3.2.** If $f: [a,b] \to \mathbb{R}$ is continuous, then $f \in \mathcal{R}[a,b]$.

**Proof.** $f$ is continuous on compact $[a,b]$, hence **uniformly continuous**:

$$\forall\, \varepsilon > 0,\; \exists\, \delta > 0: |x - y| < \delta \Rightarrow |f(x) - f(y)| < \frac{\varepsilon}{b - a}$$

Choose $P$ with $\|P\| < \delta$. On each $[x_{k-1}, x_k]$, by EVT, $\exists$ points where $f$ attains $M_k$ and $m_k$, and their distance is $< \delta$. So:

$$\omega_k(f) = M_k - m_k < \frac{\varepsilon}{b - a}$$

$$U(f,P) - L(f,P) = \sum_{k=1}^n \omega_k \cdot \Delta x_k < \frac{\varepsilon}{b-a} \cdot \sum \Delta x_k = \varepsilon \quad \blacksquare$$

```
  Proof strategy (Continuous ⟹ Integrable):

  continuous on [a,b]
       │
       ▼  (Heine-Cantor)
  uniformly continuous
       │
       ▼  (choose mesh < δ)
  oscillation < ε/(b-a) on each subinterval
       │
       ▼  (sum)
  U(f,P) - L(f,P) < ε
       │
       ▼
  Integrable ✓
```

---

## 3.3 Monotone Functions are Integrable

**Theorem 3.3.** If $f: [a,b] \to \mathbb{R}$ is monotone, then $f \in \mathcal{R}[a,b]$.

**Proof.** Assume $f$ increasing (decreasing is similar). On $[x_{k-1}, x_k]$:

$$M_k = f(x_k), \quad m_k = f(x_{k-1}), \quad \omega_k = f(x_k) - f(x_{k-1})$$

For uniform partition ($\Delta x_k = (b-a)/n$):

$$U - L = \sum_{k=1}^n [f(x_k) - f(x_{k-1})] \cdot \frac{b-a}{n} = \frac{b-a}{n}[f(b) - f(a)]$$

This is a **telescoping sum**! For $n > (b-a)[f(b)-f(a)]/\varepsilon$, we get $U - L < \varepsilon$. $\blacksquare$

---

## 3.4 Finitely Many Discontinuities

**Theorem 3.4.** If $f: [a,b] \to \mathbb{R}$ is bounded and has finitely many discontinuities, then $f \in \mathcal{R}[a,b]$.

**Proof sketch.** Let $d_1, \ldots, d_m$ be the discontinuity points. Enclose each in an interval of length $\delta$. The oscillation on these intervals contributes at most $2M \cdot m\delta$ (where $M = \sup|f|$). On the remaining intervals, $f$ is continuous, hence uniformly continuous. Choose $\delta$ small enough and a fine partition on the continuous parts. $\blacksquare$

```
  Handling finitely many discontinuities:

  a───────●──────────●─────────────b
          d₁         d₂
       [  δ  ]    [  δ  ]    ← small intervals around discontinuities
       ◄──────────────────────────►
       continuous parts: fine partition

  Total U-L ≤ (oscillation at d's) + (oscillation on continuous parts)
            ≤  2M·m·δ  +  ε/2
             < ε   for δ small enough
```

---

## 3.5 Lebesgue's Criterion (Statement)

**Theorem 3.5 (Lebesgue, 1902).** A bounded function $f: [a,b] \to \mathbb{R}$ is Riemann integrable if and only if:

$$\text{The set of discontinuities of } f \text{ has **measure zero**.}$$

A set $S \subseteq \mathbb{R}$ has **measure zero** if $\forall \varepsilon > 0$, $S$ can be covered by countably many intervals whose total length is $< \varepsilon$.

**Examples of measure zero sets:** finite sets, countable sets (like $\mathbb{Q}$), the Cantor set.

| Function | Discontinuities | Measure zero? | Integrable? |
|----------|----------------|---------------|-------------|
| Continuous $f$ | $\emptyset$ | Yes (trivially) | **Yes** |
| Monotone $f$ | At most countable | Yes | **Yes** |
| Thomae's function | $\mathbb{Q} \cap [0,1]$ | Yes (countable) | **Yes** |
| Dirichlet function | $[0,1]$ | No | **No** |
| $f$ with Cantor set discontinuities | Cantor set | Yes | **Yes** |

---

## 3.6 Classes of Integrable Functions

```
  Hierarchy of Integrable Function Classes:

  ┌─────────────────────────────────────────────────┐
  │  Riemann Integrable = Bounded + Disc(f) meas. 0 │
  │  ┌───────────────────────────────────────────┐  │
  │  │  Bounded with finitely many disc.         │  │
  │  │  ┌─────────────────────────────────────┐  │  │
  │  │  │ Piecewise Continuous                │  │  │
  │  │  │  ┌───────────────────────────────┐  │  │  │
  │  │  │  │ Monotone                      │  │  │  │
  │  │  │  │  ┌─────────────────────────┐  │  │  │  │
  │  │  │  │  │ Continuous              │  │  │  │  │
  │  │  │  │  │  ┌───────────────────┐  │  │  │  │  │
  │  │  │  │  │  │ C¹ (smooth)       │  │  │  │  │  │
  │  │  │  │  │  └───────────────────┘  │  │  │  │  │
  │  │  │  │  └─────────────────────────┘  │  │  │  │
  │  │  │  └───────────────────────────────┘  │  │  │
  │  │  └─────────────────────────────────────┘  │  │
  │  └───────────────────────────────────────────┘  │
  └─────────────────────────────────────────────────┘
```

---

## 3.7 Worked Examples

**Example 1.** $f(x) = \begin{cases} 1 & x = 1/n \text{ for some } n \in \mathbb{N} \\ 0 & \text{otherwise} \end{cases}$ on $[0, 1]$.

Discontinuities: $\{1/n : n \in \mathbb{N}\} \cup \{0\}$ — countable, hence measure zero. By Lebesgue's criterion, $f \in \mathcal{R}[0,1]$, and $\int_0^1 f = 0$ (since $L(f, P) = 0$ for all $P$).

**Example 2.** Thomae's function on $[0,1]$: $f(p/q) = 1/q$ (reduced), $f(\text{irrational}) = 0$.

Continuous at every irrational, discontinuous at every rational. $\text{Disc}(f) = \mathbb{Q} \cap [0,1]$ is countable $\Rightarrow$ measure zero $\Rightarrow$ integrable with $\int_0^1 f = 0$.

---

## Summary Table

| Criterion | Condition | Integrable? |
|-----------|-----------|-------------|
| Continuous | $f \in C[a,b]$ | Always |
| Monotone | $f$ monotone on $[a,b]$ | Always |
| Finite discontinuities | Bounded, finitely many disc. | Always |
| Lebesgue | Bounded + $\text{Disc}(f)$ measure zero | ⟺ |
| Unbounded | $f$ unbounded | Never (Riemann) |

---

## Quick Revision Questions

1. **Prove** that every continuous function on $[a,b]$ is Riemann integrable.

2. **Prove** that every monotone function on $[a,b]$ is Riemann integrable.

3. **Is** $f(x) = \sin(1/x)$ for $x \neq 0$, $f(0) = 0$ integrable on $[0,1]$? Justify.

4. **State** Lebesgue's criterion, and use it to determine integrability of Thomae's function.

5. **Give** an example of a bounded function on $[0,1]$ that is NOT Riemann integrable.

6. **Explain** why the oscillation sum $\sum \omega_k \Delta x_k$ characterizes integrability.

---

[← Previous: Upper and Lower Integrals](02-upper-and-lower-integrals.md) | [Back to README](../README.md) | [Next: Properties of the Integral →](04-properties-of-integral.md)
