# Chapter 1.2: Signal Classification

## Overview

Signals can be classified along multiple dimensions based on their mathematical properties and physical characteristics. Understanding these classifications is essential because each category has distinct analytical tools and techniques associated with it.

---

## 1.2.1 Classification Overview

```
                            SIGNALS
                              │
        ┌─────────┬──────────┼──────────┬──────────┐
        │         │          │          │          │
   Deterministic  Periodic   Energy    Causal    Power
   vs Random      vs         vs        vs        vs
                  Aperiodic  Power     Non-Causal Energy
```

---

## 1.2.2 Deterministic vs. Random Signals

### Deterministic Signals

A signal is **deterministic** if its value at any time can be precisely predicted using a mathematical expression.

$$x(t) = A \cos(\omega_0 t + \phi)$$

- Value at any $t$ is exactly known
- Can be expressed as a mathematical function
- Reproducible and predictable

```
Deterministic Signal: x(t) = cos(2πt)
    │
  1 ┤     ╭──╮         ╭──╮
    │    ╱    ╲       ╱    ╲
    │   ╱      ╲     ╱      ╲
  0 ┤──╱────────╲───╱────────╲──→ t
    │            ╲ ╱          ╲
    │             ╳            ╲
 -1 ┤             ╰──╯          ╰──
    │
    Exactly known for all t
```

### Random (Stochastic) Signals

A signal is **random** if its future values cannot be exactly predicted; only statistical properties (mean, variance, probability distributions) can be described.

- **Examples**: Noise, speech, stock market data
- Analyzed using probability and statistics
- Described by ensemble averages

```
Random Signal (noise):
    │
  1 ┤ ╱╲    ╱╲  ╱╲
    │╱  ╲╱╲╱  ╲╱  ╲   ╱╲
  0 ┤────────────────╲─╱──╲───→ t
    │                  ╲    ╲╱
 -1 ┤                   ╲╱
    │
    Cannot predict exact values
```

### Comparison

| Property | Deterministic | Random |
|----------|---------------|--------|
| Predictability | Fully predictable | Only statistical properties |
| Mathematical form | Exact function | Probability distribution |
| Analysis tool | Fourier/Laplace | Autocorrelation, PSD |
| Example | $\sin(\omega t)$ | Thermal noise |
| Reproducibility | Yes | No (each realization differs) |

---

## 1.2.3 Periodic vs. Aperiodic Signals

### Periodic Signals (Continuous-Time)

A CT signal $x(t)$ is **periodic** with period $T$ if:

$$x(t) = x(t + T), \quad \forall t$$

where $T > 0$ is the **fundamental period** (smallest such $T$).

- **Fundamental frequency**: $f_0 = 1/T$ Hz
- **Angular frequency**: $\omega_0 = 2\pi/T = 2\pi f_0$ rad/s

```
Periodic Signal: Period T
    │
    │    ╭──╮    ╭──╮    ╭──╮    ╭──╮
    │   ╱    ╲  ╱    ╲  ╱    ╲  ╱    ╲
  0 ┤──╱──────╲╱──────╲╱──────╲╱──────╲─→ t
    │
    │  |←─ T ─→|←─ T ─→|←─ T ─→|
    │
    x(t) = x(t + T) for all t
```

### Periodic Signals (Discrete-Time)

A DT signal $x[n]$ is **periodic** with period $N$ if:

$$x[n] = x[n + N], \quad \forall n$$

where $N$ is a **positive integer** (fundamental period).

> **Key difference**: For DT signals, $N$ must be an integer. A DT signal $x[n] = \cos(\omega_0 n)$ is periodic only if $\omega_0 / (2\pi)$ is a rational number.

### Periodicity Test for DT Signals

Given $x[n] = \cos(\omega_0 n + \phi)$:

1. Compute $\omega_0 / (2\pi)$
2. If the result is a **rational number** $p/q$ (in lowest terms), the signal is periodic with $N = q$
3. If irrational → signal is **not periodic**

**Example**: $x[n] = \cos(0.5\pi n)$
- $\omega_0/(2\pi) = 0.5\pi / (2\pi) = 1/4$ → Rational → Periodic with $N = 4$

**Example**: $x[n] = \cos(n)$
- $\omega_0/(2\pi) = 1/(2\pi)$ → Irrational → **Not periodic**

### Aperiodic Signals

A signal that does **not** satisfy the periodicity condition for any finite $T$ (or $N$) is **aperiodic**.

```
Aperiodic Signal (decaying exponential):
    │
  1 ┤╲
    │ ╲
    │  ╲
    │   ╲
    │    ╲╲
    │      ╲╲
  0 ┤────────╲╲────────────────→ t
    │
    No repeating pattern
```

### Summary

| Property | Periodic | Aperiodic |
|----------|----------|-----------|
| Repeats? | Yes, with period $T$ or $N$ | No |
| Duration | Infinite (extends both ways) | May be finite or infinite |
| CT condition | $x(t) = x(t+T)$ | No such $T$ exists |
| DT condition | $x[n] = x[n+N]$, $N \in \mathbb{Z}^+$ | No such integer $N$ |
| Fourier repr. | Fourier Series | Fourier Transform |
| Power/Energy | Power signal | Usually energy signal |

---

## 1.2.4 Energy and Power Signals

### Energy of a Signal

**Continuous-Time**:
$$E = \int_{-\infty}^{\infty} |x(t)|^2 \, dt$$

**Discrete-Time**:
$$E = \sum_{n=-\infty}^{\infty} |x[n]|^2$$

### Power of a Signal

**Continuous-Time**:
$$P = \lim_{T \to \infty} \frac{1}{2T} \int_{-T}^{T} |x(t)|^2 \, dt$$

**Discrete-Time**:
$$P = \lim_{N \to \infty} \frac{1}{2N+1} \sum_{n=-N}^{N} |x[n]|^2$$

### Classification Rules

| Condition | Classification |
|-----------|---------------|
| $0 < E < \infty$ and $P = 0$ | **Energy signal** |
| $E = \infty$ and $0 < P < \infty$ | **Power signal** |
| $E = \infty$ and $P = \infty$ | **Neither** |

> **Key Insight**: A signal cannot be both an energy signal and a power signal simultaneously.

### Decision Flowchart

```
         Compute E (total energy)
                │
         ┌──────┴──────┐
         │             │
    E < ∞          E = ∞
    P = 0             │
         │      ┌─────┴─────┐
    ENERGY      │           │
    SIGNAL   P < ∞       P = ∞
              P > 0          │
                │        NEITHER
            POWER
            SIGNAL
```

### Examples

| Signal | Energy | Power | Type |
|--------|--------|-------|------|
| $x(t) = e^{-at}u(t), a>0$ | $\frac{1}{2a}$ | $0$ | Energy |
| $x(t) = A\cos(\omega t)$ | $\infty$ | $\frac{A^2}{2}$ | Power |
| $x(t) = e^{at}u(t), a>0$ | $\infty$ | $\infty$ | Neither |
| $x(t) = c$ (constant) | $\infty$ | $c^2$ | Power |
| $x(t) = tu(t)$ (ramp) | $\infty$ | $\infty$ | Neither |

---

## 1.2.5 Causal vs. Non-Causal Signals

### Causal Signal

A signal $x(t)$ is **causal** if:

$$x(t) = 0, \quad \forall t < 0$$

The signal exists only for $t \geq 0$.

```
Causal Signal:
    │
  1 ┤     ╭──╮
    │    ╱    ╲
    │   ╱      ╲
  0 ┤──────────────╲───────→ t
    │  ↑           ╰──
    │  t=0
    │ x(t) = 0 for t < 0
```

### Anti-Causal Signal

A signal $x(t)$ is **anti-causal** if:

$$x(t) = 0, \quad \forall t > 0$$

The signal exists only for $t \leq 0$.

```
Anti-Causal Signal:
    │
  1 ┤  ╭──╮
    │ ╱    ╲
    │╱      ╲
  0 ┤────────╲─────────────→ t
    │         ╰──  ↑
    │              t=0
    │ x(t) = 0 for t > 0
```

### Non-Causal Signal

A signal that is neither causal nor anti-causal — it has non-zero values for both $t < 0$ and $t > 0$.

```
Non-Causal Signal:
    │
  1 ┤     ╭──────╮
    │    ╱        ╲
    │   ╱          ╲
  0 ┤──╱────────────╲─────→ t
    │       ↑
    │       t=0
    │ Non-zero on BOTH sides of t=0
```

---

## 1.2.6 Bounded vs. Unbounded Signals

A signal is **bounded** if there exists a finite constant $M$ such that:

$$|x(t)| \leq M < \infty, \quad \forall t$$

| Signal | Bounded? |
|--------|----------|
| $\sin(t)$ | Yes ($M = 1$) |
| $e^{-t}u(t)$ | Yes ($M = 1$) |
| $e^{t}u(t)$ | No (grows to $\infty$) |
| $tu(t)$ (ramp) | No |
| $u(t)$ (step) | Yes ($M = 1$) |

---

## 1.2.7 Real vs. Complex Signals

| Type | Definition | Example |
|------|-----------|---------|
| **Real** | $x(t) \in \mathbb{R}$ for all $t$ | $\cos(\omega t)$ |
| **Complex** | $x(t) \in \mathbb{C}$ for some $t$ | $e^{j\omega t} = \cos(\omega t) + j\sin(\omega t)$ |

Complex exponentials are fundamental in Fourier analysis:

$$e^{j\omega t} = \cos(\omega t) + j\sin(\omega t) \quad \text{(Euler's formula)}$$

---

## 1.2.8 One-Dimensional vs. Multi-Dimensional Signals

| Dimension | Independent Variables | Example |
|-----------|----------------------|---------|
| 1-D | Single ($t$ or $n$) | Audio signal $x(t)$ |
| 2-D | Two ($x, y$) | Image $I(x, y)$ |
| 3-D | Three ($x, y, t$) | Video $V(x, y, t)$ |

---

## Summary Table

| Classification Basis | Categories | Key Criterion |
|---------------------|------------|---------------|
| Independent variable | CT / DT | $t \in \mathbb{R}$ vs. $n \in \mathbb{Z}$ |
| Predictability | Deterministic / Random | Exact formula vs. statistical |
| Repetition | Periodic / Aperiodic | $x(t) = x(t+T)$ |
| Energy/Power | Energy / Power / Neither | Finite $E$ vs. finite $P$ |
| Time existence | Causal / Anti-causal / Non-causal | Zero for $t<0$? |
| Amplitude range | Bounded / Unbounded | $|x(t)| \leq M$? |
| Value type | Real / Complex | $\mathbb{R}$ vs. $\mathbb{C}$ |
| Dimensions | 1-D / 2-D / Multi-D | Number of independent variables |

---

## Quick Revision Questions

1. **How do you test if a discrete-time sinusoidal signal $x[n] = \cos(\omega_0 n)$ is periodic?**
   - Check if $\omega_0/(2\pi)$ is rational. If $\omega_0/(2\pi) = p/q$, the signal is periodic with $N = q$.

2. **Is the signal $x(t) = 5\cos(3t)$ an energy signal or a power signal?**
   - Power signal with $P = 25/2 = 12.5$.

3. **What is the key difference between a causal and a non-causal signal?**
   - A causal signal is zero for all $t < 0$; a non-causal signal has non-zero values for $t < 0$.

4. **Is $x[n] = \cos(\sqrt{2}\pi n)$ periodic? Justify.**
   - $\omega_0/(2\pi) = \sqrt{2}\pi/(2\pi) = \sqrt{2}/2$, which is irrational, so the signal is **not periodic**.

5. **Can a signal be both an energy signal and a power signal?**
   - No. If $E$ is finite, $P = 0$. If $P$ is finite and nonzero, $E = \infty$.

6. **Classify the signal $x(t) = e^{-2t}u(t)$ as causal/non-causal and energy/power.**
   - Causal (zero for $t<0$) and energy signal ($E = 1/4$).

---

[← Previous: Continuous and Discrete Signals](01-continuous-and-discrete-signals.md) | [Next: Basic Operations on Signals →](03-basic-operations-on-signals.md)
