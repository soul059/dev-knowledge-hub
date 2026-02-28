# Chapter 3.5: Stable and Unstable Systems

## Overview

Stability ensures that a system produces a **finite, well-behaved output** for any reasonable input. An unstable system can produce outputs that grow without bound, potentially causing physical damage or meaningless results. BIBO stability is the primary criterion.

---

## 3.5.1 BIBO Stability Definition

A system is **Bounded-Input Bounded-Output (BIBO) stable** if:

$$|x(t)| \leq M_x < \infty \quad \forall t \implies |y(t)| \leq M_y < \infty \quad \forall t$$

> Every bounded input produces a bounded output.

A system is **BIBO unstable** if there exists even **one** bounded input that produces an unbounded output.

```
STABLE SYSTEM:                    UNSTABLE SYSTEM:
    │                                 │
    │  Input (bounded):               │  Input (bounded):
  M ┤──┬──┬──                       M ┤──┬──┬──
    │  │  │  │                        │  │  │  │
  0 ┤──┴──┴──┴──→ t               0 ┤──┴──┴──┴──→ t
    │                                 │
    │  Output (bounded):              │  Output (UNBOUNDED!):
  N ┤──╲──╲──                         │              ╱
    │   ╲  ╲  ╲                       │            ╱
  0 ┤────╲──╲──╲→ t                0 ┤──────────╱──→ t
    │                                 │       ╱
    │  |y| ≤ N ✓                     │     ╱ → ∞  ✗
```

---

## 3.5.2 Testing for BIBO Stability

### Direct Test

For each system, try to find:
1. A bounded input $x$ with $|x| \leq M$
2. Check if $|y|$ is always bounded

If **any** bounded input causes unbounded output → **Unstable**.

### LTI System Test (Most Important!)

For an LTI system with impulse response $h(t)$:

> **An LTI system is BIBO stable if and only if the impulse response is absolutely integrable:**

$$\int_{-\infty}^{\infty} |h(t)| \, dt < \infty \quad \text{(CT)}$$

$$\sum_{n=-\infty}^{\infty} |h[n]| < \infty \quad \text{(DT)}$$

### Proof Sketch (CT)

$$|y(t)| = \left|\int_{-\infty}^{\infty} h(\tau)x(t-\tau)d\tau\right| \leq \int_{-\infty}^{\infty} |h(\tau)| \cdot |x(t-\tau)| d\tau \leq M_x \int_{-\infty}^{\infty} |h(\tau)| d\tau$$

If $\int|h| < \infty$, then $|y| \leq M_x \cdot \int|h| = M_y < \infty$. ✓

---

## 3.5.3 Examples — Stable Systems

### Example 1: $y(t) = 3x(t)$

$|y| = 3|x| \leq 3M$. Bounded. **Stable** ✓

### Example 2: $h(t) = e^{-2t}u(t)$

$$\int_0^{\infty} |e^{-2t}| dt = \int_0^{\infty} e^{-2t} dt = \frac{1}{2} < \infty$$

**Stable** ✓

### Example 3: $h[n] = (0.5)^n u[n]$

$$\sum_{n=0}^{\infty} |0.5|^n = \frac{1}{1 - 0.5} = 2 < \infty$$

**Stable** ✓

### Example 4: $h[n] = \delta[n] - 0.8\delta[n-1]$ (FIR filter)

$$\sum |h[n]| = |1| + |-0.8| = 1.8 < \infty$$

**Stable** ✓ (all FIR filters are BIBO stable)

```
Stable systems — impulse responses that decay:

h(t) = e⁻²ᵗu(t):           h[n] = (0.5)ⁿu[n]:
    │                           │
  1 ┤╲                       1 ┤  ↑
    │ ╲                         │     ↑
    │  ╲╲                       │        ↑
    │    ╲╲                     │           ↑
  0 ┤──────╲╲──→ t           0 ┼──┼──┼──┼──┼──→ n
    0  1  2  3                   0  1  2  3  4
    Decays → ∫|h|<∞               Decays → Σ|h|<∞
```

---

## 3.5.4 Examples — Unstable Systems

### Example 5: $y(t) = \int_{-\infty}^{t} x(\tau)d\tau$ (Integrator)

$h(t) = u(t)$: $\int_0^{\infty} |u(t)| dt = \infty$ → **Unstable** ✗

**Counterexample**: Input $x(t) = u(t)$ (bounded, $|x| \leq 1$) → $y(t) = tu(t)$ (unbounded ramp)

### Example 6: $h(t) = e^{2t}u(t)$ (Growing exponential)

$$\int_0^{\infty} e^{2t} dt = \infty$$

**Unstable** ✗

### Example 7: $h[n] = 2^n u[n]$

$$\sum_{n=0}^{\infty} |2^n| = \infty$$

**Unstable** ✗

### Example 8: $y[n] = \sum_{k=-\infty}^{n} x[k]$ (Accumulator)

$h[n] = u[n]$: $\sum_{n=0}^{\infty} 1 = \infty$ → **Unstable** ✗

```
Unstable systems — impulse responses that DON'T decay:

h(t) = u(t):                h(t) = e²ᵗu(t):
    │                           │         ╱
  1 ┤  ┌──────────            │        ╱
    │  │                        │       ╱
    │  │                        │     ╱
  0 ┤──┘──────────→ t        0 ┤────╱─────→ t
    0                           0
    ∫|h| = ∞ (Unstable)        ∫|h| = ∞ (Unstable)
```

---

## 3.5.5 Stability in Transform Domain

### Laplace Transform (CT)

An LTI system with transfer function $H(s)$ is BIBO stable iff **all poles** have **negative real parts** (lie in the left half of the $s$-plane):

$$\text{Re}\{p_i\} < 0 \quad \forall \text{ poles } p_i$$

```
s-plane:
    jω ↑
       │
   ×   │        × = pole
   ────┼────→ σ
   ×   │
       │
  LEFT │ RIGHT
  HALF │ HALF
       
  All poles in LHP → STABLE
  Any pole in RHP → UNSTABLE
  Pole on jω axis → MARGINALLY STABLE
```

### Z-Transform (DT)

An LTI system with transfer function $H(z)$ is BIBO stable iff **all poles** lie **inside the unit circle**:

$$|p_i| < 1 \quad \forall \text{ poles } p_i$$

```
z-plane:
    Im ↑
       │  ╭──╮
   ×   │ ╱    ╲   Unit circle: |z| = 1
   ────┼╱──────╲──→ Re
       │╲      ╱
       │  ╰──╯
       │  × 
       
  All poles inside unit circle → STABLE
  Any pole outside → UNSTABLE
  Pole on unit circle → MARGINALLY STABLE
```

---

## 3.5.6 Marginal Stability

A system is **marginally stable** if bounded inputs produce bounded outputs for most (but not all) inputs. Examples:

- $h(t) = u(t)$ — integrator (marginally unstable in general, but bounded output for some specific bounded inputs)
- $h(t) = \cos(\omega_0 t)$ — pure oscillator (bounded $h$ but $\int|h| = \infty$, the response to a sinusoidal input at the same frequency grows without bound → resonance)

---

## 3.5.7 Stability of Common Systems

| System | $h(t)$ or $h[n]$ | $\int\|h\|$ or $\sum\|h\|$ | Stable? |
|--------|-------------------|---------------------------|---------|
| $y = ax$ | $a\delta(t)$ | $\|a\|$ | ✓ |
| $y = x(t-d)$ | $\delta(t-d)$ | $1$ | ✓ |
| FIR filter | Finite-length $h$ | Always finite | ✓ |
| $e^{-at}u(t)$, $a>0$ | Decaying exponential | $1/a$ | ✓ |
| $e^{at}u(t)$, $a>0$ | Growing exponential | $\infty$ | ✗ |
| $u(t)$ | Step | $\infty$ | ✗ |
| $r^n u[n]$, $\|r\|<1$ | Decaying geometric | $1/(1-\|r\|)$ | ✓ |
| $r^n u[n]$, $\|r\| \geq 1$ | Growing geometric | $\infty$ | ✗ |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| BIBO Stable | Bounded input → bounded output, always |
| Unstable | ∃ bounded input → unbounded output |
| LTI stability test | $\int_{-\infty}^{\infty}\|h(t)\|dt < \infty$ |
| DT LTI test | $\sum_{n=-\infty}^{\infty}\|h[n]\| < \infty$ |
| Pole location (CT) | All poles in LHP ($\text{Re}(p) < 0$) |
| Pole location (DT) | All poles inside unit circle ($\|p\| < 1$) |
| FIR filters | Always BIBO stable |
| Integrator | Unstable (impulse response not absolutely integrable) |

---

## Quick Revision Questions

1. **Is the system $y(t) = e^{x(t)}$ BIBO stable?**
   - Yes. If $|x(t)| \leq M$, then $|y(t)| = e^{x(t)} \leq e^M < \infty$.

2. **What is the BIBO stability condition for an LTI system?**
   - The impulse response must be absolutely integrable (CT) or absolutely summable (DT).

3. **Is a system with $h[n] = (1.01)^n u[n]$ stable?**
   - No. $|r| = 1.01 > 1$, so $\sum|h[n]| = \infty$.

4. **Are all FIR filters stable?**
   - Yes. Finite-length impulse response → finite sum of absolute values.

5. **In the $s$-plane, where must all poles be for a stable CT-LTI system?**
   - In the left half-plane: $\text{Re}(s) < 0$ for all poles.

6. **Give a bounded input that causes the integrator to produce unbounded output.**
   - $x(t) = u(t)$ (unit step, bounded by 1) → $y(t) = tu(t)$ (ramp, unbounded).

---

[← Previous: Causal and Non-Causal Systems](04-causal-and-noncausal-systems.md) | [Next: Memory and Memoryless Systems →](06-memory-and-memoryless-systems.md)
