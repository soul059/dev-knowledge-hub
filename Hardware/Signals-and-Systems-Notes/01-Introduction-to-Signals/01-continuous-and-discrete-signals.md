# Chapter 1.1: Continuous and Discrete Signals

## Overview

A **signal** is a function of one or more independent variables that carries information. In signals and systems, the most common independent variable is **time**. Signals can be broadly categorized based on the nature of their independent variable (time axis) and their amplitude values.

---

## 1.1.1 What is a Signal?

A signal is a mathematical function that maps an independent variable (usually time) to a dependent variable (usually amplitude/voltage/current).

$$x : \text{Domain (time)} \rightarrow \text{Range (amplitude)}$$

### Physical Examples of Signals

| Signal Type | Example | Nature |
|-------------|---------|--------|
| Speech | Microphone voltage vs. time | Continuous |
| Temperature | Room temperature over a day | Continuous |
| Stock Price | Daily closing price | Discrete |
| Digital Audio | CD samples (44,100/sec) | Discrete |
| ECG | Heart electrical activity | Continuous |

---

## 1.1.2 Continuous-Time (CT) Signals

A **continuous-time signal** is defined for every instant of time within a given interval. The independent variable (time) takes on continuous values.

**Notation**: $x(t)$, where $t \in \mathbb{R}$ (real numbers)

### Mathematical Definition

A signal $x(t)$ is continuous-time if it is defined for all $t$ in some continuous interval:

$$x(t), \quad t \in (-\infty, +\infty) \text{ or } t \in [t_1, t_2]$$

### ASCII Representation of a Continuous-Time Signal

```
x(t) — Continuous Sinusoidal Signal
    │
  1 ┤        ╭──╮              ╭──╮
    │       ╱    ╲            ╱    ╲
    │      ╱      ╲          ╱      ╲
  0 ┤─────╱────────╲────────╱────────╲───── t
    │                ╲      ╱          ╲
    │                 ╲    ╱            ╲
 -1 ┤                  ╰──╯              ╰──╯
    │
    └──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──→ t
       0  1  2  3  4  5  6  7  8  9 10 11
    
    Signal is defined at EVERY value of t
    (smooth, unbroken curve)
```

### Properties of CT Signals

1. **Domain**: Time is continuous ($t \in \mathbb{R}$)
2. **Range**: Amplitude can be any real value
3. **Representation**: Smooth curves
4. **Processing**: Analog circuits (op-amps, resistors, capacitors)
5. **Examples**: Voltage, current, temperature, pressure

### CT Signal Processing System (Block Diagram)

```
    ┌─────────────────┐       ┌─────────────────┐
    │   Continuous     │       │   Continuous     │
    │   Input Signal   │       │   Output Signal  │
    │    x(t)          ├──────►│    y(t)          │
    │                  │       │                  │
    │  (e.g., voice)   │  CT   │  (e.g., filtered │
    │                  │ System│   audio)         │
    └─────────────────┘       └─────────────────┘
```

---

## 1.1.3 Discrete-Time (DT) Signals

A **discrete-time signal** is defined only at discrete instants of time, typically at integer values of the independent variable. The signal is represented as a **sequence** of numbers.

**Notation**: $x[n]$, where $n \in \mathbb{Z}$ (integers)

### Mathematical Definition

A signal $x[n]$ is discrete-time if it is defined only for integer values of $n$:

$$x[n], \quad n \in \{\ldots, -2, -1, 0, 1, 2, \ldots\}$$

### ASCII Representation of a Discrete-Time Signal

```
x[n] — Discrete-Time Signal
    │
  4 ┤              ↑
    │              │
  3 ┤     ↑       │        ↑
    │     │       │        │
  2 ┤     │  ↑   │   ↑    │
    │     │  │   │   │    │
  1 ┤  ↑  │  │   │   │    │  ↑
    │  │  │  │   │   │    │  │
  0 ┼──┼──┼──┼───┼───┼────┼──┼──→ n
    │ -3 -2 -1   0   1    2  3
    
    Signal is defined ONLY at integer values of n
    (individual "stem" values, not connected)
```

### Properties of DT Signals

1. **Domain**: Time is discrete ($n \in \mathbb{Z}$)
2. **Range**: Amplitude can be any real value (unless quantized)
3. **Representation**: Stem plots (individual points)
4. **Processing**: Digital processors, computers, DSP chips
5. **Examples**: Sampled audio, daily stock prices, sensor readings

### DT Signal Processing System (Block Diagram)

```
    ┌─────────────────┐       ┌─────────────────┐
    │   Discrete       │       │   Discrete       │
    │   Input Signal   │       │   Output Signal  │
    │    x[n]          ├──────►│    y[n]          │
    │                  │       │                  │
    │  (e.g., sampled  │  DT   │  (e.g., filtered │
    │   audio)         │ System│   samples)       │
    └─────────────────┘       └─────────────────┘
```

---

## 1.1.4 Analog vs. Digital Signals

Beyond CT and DT, signals can also be classified by whether their **amplitude** is continuous or discrete:

| Property | Analog | Digital |
|----------|--------|---------|
| **Time axis** | Continuous | Discrete |
| **Amplitude** | Continuous | Discrete (quantized) |
| **Representation** | Smooth curve | Binary numbers |
| **Example** | Voltage on a wire | Data in a computer |
| **Processing** | Analog hardware | Digital hardware |

### Four Types of Signals

```
                        Time
                 Continuous    Discrete
              ┌─────────────┬──────────────┐
  Continuous  │   ANALOG    │   SAMPLED    │
  Amplitude   │   Signal    │   Signal     │
              │  x(t) ∈ ℝ  │  x[n] ∈ ℝ   │
              ├─────────────┼──────────────┤
  Discrete    │  QUANTIZED  │  DIGITAL     │
  Amplitude   │   Signal    │   Signal     │
              │             │  x[n] ∈ {L}  │
              └─────────────┴──────────────┘
```

### Conversion Between Signal Types

```
                    Sampling              Quantization
   Analog ───────────────────► Sampled ──────────────────► Digital
   Signal    (T = 1/fs)        Signal     (L levels)       Signal
   x(t)                        x[n]                      x_q[n]

              ┌────────┐      ┌────────┐      ┌────────┐
   x(t) ────►│ Sampler ├────►│Quantiz.├────►│ Encoder├────► Binary
              │  (ADC)  │     │        │      │        │    Output
              └────────┘      └────────┘      └────────┘
```

---

## 1.1.5 Comparison: CT vs DT Signals

| Feature | Continuous-Time $x(t)$ | Discrete-Time $x[n]$ |
|---------|------------------------|----------------------|
| **Independent variable** | Continuous ($t \in \mathbb{R}$) | Discrete ($n \in \mathbb{Z}$) |
| **Defined at** | Every instant | Only integer values |
| **Graphical repr.** | Smooth curve | Stem plot |
| **Notation** | Parentheses: $x(t)$ | Brackets: $x[n]$ |
| **Mathematics** | Integrals, derivatives | Summations, differences |
| **Physical source** | Natural phenomena | Sampled or digital |
| **Processing** | Analog circuits | Digital processors |
| **Convolution** | $\int_{-\infty}^{\infty}$ | $\sum_{k=-\infty}^{\infty}$ |
| **Transform** | Fourier/Laplace | DTFT/Z-Transform |

---

## 1.1.6 Practical Examples

### Example 1: CT Signal — RC Circuit Voltage

Consider a capacitor charging through a resistor:

$$v_C(t) = V_0 \left(1 - e^{-t/RC}\right) u(t)$$

```
    R
   ┌──/\/\/──┬──────┐
   │         │      │
 ──┤       ──┴──    │
V_0│       ──┬── C  │
   │         │      │
   └─────────┴──────┘

v_C(t):
    │
V_0 ┤ · · · · · · · · · · · · · · · · ·
    │                    ╭───────────────
    │                 ╱
    │              ╱
    │           ╱
    │        ╱
    │     ╱        τ = RC (time constant)
    │   ╱
    │  ╱
  0 ┤─╱──────────────────────────────→ t
    │ 0    τ   2τ   3τ   4τ   5τ
    │
    │  At t = τ: v_C = 0.632 × V_0
    │  At t = 5τ: v_C ≈ V_0 (99.3%)
```

### Example 2: DT Signal — Moving Average Filter

For stock prices sampled daily:

$$y[n] = \frac{1}{3}\left(x[n] + x[n-1] + x[n-2]\right)$$

```
Input x[n]:          Output y[n] (smoothed):
    │                    │
 50 ┤     ↑              │
 45 ┤  ↑     ↑        43 ┤        ↑
 40 ┤        ↑  ↑     40 ┤  ↑  ↑     ↑
 35 ┤  ↑              38 ┤     ↑
 30 ┤                    │
  0 ┼──┼──┼──┼──┼──→  0 ┼──┼──┼──┼──┼──→
     1  2  3  4  5       1  2  3  4  5
```

---

## 1.1.7 Real-World Applications

| Domain | CT Signals | DT Signals |
|--------|-----------|------------|
| **Audio** | Microphone output | MP3 samples |
| **Video** | Camera sensor output | MPEG frames |
| **Medical** | ECG/EEG waveform | Digitized patient data |
| **Communications** | Radio waves | Digital modulation |
| **Control** | Motor speed signal | Sampled feedback |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Signal | Function carrying information, typically $x(t)$ or $x[n]$ |
| CT Signal | Defined for all real $t$; notation $x(t)$ |
| DT Signal | Defined only for integer $n$; notation $x[n]$ |
| Analog | Continuous time + continuous amplitude |
| Digital | Discrete time + discrete (quantized) amplitude |
| Sampled Signal | Continuous amplitude at discrete times |
| CT Processing | Integrals, derivatives, analog circuits |
| DT Processing | Summations, differences, digital processors |
| Conversion | CT → DT via sampling; DT → CT via reconstruction |

---

## Quick Revision Questions

1. **What is the fundamental difference between $x(t)$ and $x[n]$ in terms of their independent variable?**
   - $x(t)$ is defined for all real values of $t$ (continuous); $x[n]$ is defined only at integer values of $n$ (discrete).

2. **Give two examples each of continuous-time and discrete-time signals from daily life.**
   - CT: Voice signal, temperature change over a day. DT: Daily stock closing price, monthly rainfall data.

3. **What are the four types of signals based on the nature of time and amplitude?**
   - Analog, Sampled, Quantized, and Digital.

4. **What mathematical operation is used for convolution in CT vs DT?**
   - CT: Integration ($\int$); DT: Summation ($\sum$).

5. **Why is the bracket notation $x[n]$ used instead of $x(n)$ for discrete-time signals?**
   - To clearly distinguish discrete-time signals from continuous-time signals and avoid ambiguity in mathematical expressions.

6. **In the ADC process, what is the role of sampling and quantization?**
   - Sampling discretizes the time axis (CT → DT), and quantization discretizes the amplitude axis (continuous amplitude → discrete levels).

---

[← Back to Unit 1](README.md) | [Next: Signal Classification →](02-signal-classification.md)
