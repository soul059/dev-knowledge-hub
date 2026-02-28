# 3.4 The Complex Logarithmic Function

[← Previous: Hyperbolic Functions](03-hyperbolic-functions.md) | [Next: Power Functions →](05-power-functions.md)

---

## Chapter Overview

The complex logarithm is the first truly **multi-valued** function we encounter systematically. Because $e^z$ is periodic (period $2\pi i$), its inverse — the logarithm — assigns infinitely many values to each input. Managing this multi-valuedness through **branch cuts** and **principal values** is fundamental to all of complex analysis.

---

## 1. Definition

### 1.1 Multi-valued Logarithm

The **complex logarithm** of $z \ne 0$ is any $w$ such that $e^w = z$. Writing $z = |z|e^{i\theta}$ and $w = u + iv$:

$$e^{u+iv} = |z|e^{i\theta} \implies u = \ln|z|, \quad v = \theta + 2k\pi$$

$$\boxed{\log z = \ln|z| + i\arg(z) = \ln|z| + i(\theta + 2k\pi), \quad k \in \mathbb{Z}}$$

This gives **infinitely many values** for each $z \ne 0$.

### 1.2 Principal Logarithm

$$\boxed{\text{Log}(z) = \ln|z| + i\,\text{Arg}(z)}$$

where $\text{Arg}(z) \in (-\pi, \pi]$ is the principal argument. All values are:

$$\log z = \text{Log}(z) + 2k\pi i, \quad k \in \mathbb{Z}$$

### 1.3 Example

$\log(-1) = \ln 1 + i(\pi + 2k\pi) = i(2k+1)\pi$

Principal value: $\text{Log}(-1) = i\pi$

Other values: $\ldots, -3\pi i, -\pi i,\; i\pi,\; 3\pi i,\; 5\pi i, \ldots$ — wait, let's be precise:

$\log(-1) = i\pi + 2k\pi i = i(2k+1)\pi$ for $k \in \mathbb{Z}$.

```
Values of log(-1) on the imaginary axis:

    Im ▲
       │    • 5πi     (k=2)
       │
       │    • 3πi     (k=1)
       │
       │    • πi      (k=0)  ← principal value
       │
  ─────O─────────────► Re
       │
       │    • -πi     (k=-1)
       │
       │    • -3πi    (k=-2)

  All values equally spaced, separation 2πi
```

---

## 2. Branch Cuts and Branches

### 2.1 The Problem

$\log z$ is multi-valued, but for analysis we need single-valued functions. We achieve this by:

1. **Removing a ray** from the plane (a **branch cut**) to prevent the argument from being continuous and multi-valued
2. Selecting one value continuously — this is a **branch**

### 2.2 Principal Branch

The **principal branch** $\text{Log}(z)$ uses $\text{Arg}(z) \in (-\pi, \pi]$ with branch cut along the **negative real axis** $(-\infty, 0]$.

```
Branch cut for Log(z):

            Im
             ▲
             │
  ═══════════O──────────► Re
  ← branch cut              ← Log(z) is analytic here
  along (-∞, 0]               (in the cut plane ℂ\(-∞,0])
             │
```

### 2.3 Analyticity of Log(z)

$\text{Log}(z)$ is analytic on $\mathbb{C} \setminus (-\infty, 0]$ with derivative:

$$\frac{d}{dz}\text{Log}(z) = \frac{1}{z}$$

### 2.4 General Branches

We can place the branch cut along any ray from the origin. For a branch cut along $\arg(z) = \alpha$:

$$\log_\alpha z = \ln|z| + i\theta, \quad \alpha < \theta < \alpha + 2\pi$$

---

## 3. Properties

### 3.1 Identities (Multi-valued)

| Identity | Caveat |
|----------|--------|
| $\log(z_1 z_2) = \log z_1 + \log z_2$ | Holds as sets of values |
| $\log(z_1/z_2) = \log z_1 - \log z_2$ | Holds as sets |
| $\log(z^n) = n\log z$ | For integer $n$, as sets |
| $e^{\log z} = z$ | Always true |
| $\log(e^z) = z + 2k\pi i$ | NOT simply $z$ |

### 3.2 Warning: Principal Value Does NOT Preserve Identities

$$\text{Log}(z_1 z_2) \ne \text{Log}(z_1) + \text{Log}(z_2) \quad \text{in general}$$

**Counterexample**: $z_1 = z_2 = -1$
- $\text{Log}(z_1) + \text{Log}(z_2) = i\pi + i\pi = 2\pi i$
- $\text{Log}(z_1 z_2) = \text{Log}(1) = 0 \ne 2\pi i$

### 3.3 Discontinuity on the Branch Cut

$\text{Log}(z)$ has a jump discontinuity across the negative real axis:

$$\lim_{y \to 0^+} \text{Log}(x + iy) = \ln|x| + i\pi$$
$$\lim_{y \to 0^-} \text{Log}(x + iy) = \ln|x| - i\pi$$

Jump $= 2\pi i$ for $x < 0$.

---

## 4. Mapping Properties

### 4.1 Image of $\text{Log}(z)$

$\text{Log}: \mathbb{C} \setminus (-\infty, 0] \to \{w : -\pi < \text{Im}(w) < \pi\}$

This is the **inverse** of the exponential restricted to the horizontal strip $-\pi < y < \pi$.

```
Mapping w = Log(z):

z-plane (cut plane):            w-plane (horizontal strip):
    Im                              Im
     ▲                               ▲
     │     z-plane                 π ─┤─────────────────
     │     (minus negative         │  │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
  ═══O═══════► Re       Log(z)    │  │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
     │         ─────────►          0 ─┤▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
     │                             │  │▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
     │                          -π ─┤─────────────────
                                      └────────────────► Re

  Circles |z|=r → vertical lines u = ln r
  Rays arg(z)=θ → horizontal lines v = θ
```

---

## 5. Design Example

### Example: Find all values of $\log(1+i)$ and identify the principal value.

**Step 1.** $|1+i| = \sqrt{2}$, so $\ln|1+i| = \frac{1}{2}\ln 2$.

**Step 2.** $\text{Arg}(1+i) = \pi/4$.

**Step 3.** All values:
$$\log(1+i) = \frac{1}{2}\ln 2 + i\left(\frac{\pi}{4} + 2k\pi\right), \quad k \in \mathbb{Z}$$

**Principal value**: $\text{Log}(1+i) = \frac{1}{2}\ln 2 + i\frac{\pi}{4}$

---

## 6. Real-World Applications

| Application | Usage |
|-------------|-------|
| **Signal Processing** | Phase unwrapping — managing the multi-valuedness of $\arg$ |
| **Control Theory** | Bode plots: $20\log_{10}|H(j\omega)|$ vs. $\angle H(j\omega)$ |
| **Quantum Mechanics** | Berry phase via complex logarithm |
| **Complex Integration** | $\int \frac{dz}{z} = \log z$ — branch cuts determine contour integral values |

---

## Summary Table

| Concept | Key Formula |
|---------|-------------|
| Multi-valued log | $\log z = \ln\|z\| + i(\theta + 2k\pi)$ |
| Principal log | $\text{Log}(z) = \ln\|z\| + i\,\text{Arg}(z)$ |
| Branch cut (principal) | Negative real axis $(-\infty, 0]$ |
| Derivative | $\frac{d}{dz}\text{Log}(z) = 1/z$ |
| Log is NOT additive | $\text{Log}(z_1z_2) \ne \text{Log}(z_1) + \text{Log}(z_2)$ in general |
| Jump on cut | $2\pi i$ |
| $e^{\log z}$ | $= z$ always |
| $\log(e^z)$ | $= z + 2k\pi i$ |

---

## Quick Revision Questions

1. **Find** all values of $\log(-i)$ and identify the principal value.

2. **Explain** why we need a branch cut for $\text{Log}(z)$ and what happens at the cut.

3. **Give** a specific counterexample showing $\text{Log}(z_1 z_2) \ne \text{Log}(z_1) + \text{Log}(z_2)$.

4. **Derive** the formula $(\text{Log}(z))' = 1/z$ using the inverse function theorem.

5. **Describe** the image of the unit circle $|z| = 1$ under $w = \text{Log}(z)$.

6. **Find** all values of $\log(i^2)$ and $2\log(i)$ and explain why they differ.

---

[← Previous: Hyperbolic Functions](03-hyperbolic-functions.md) | [Next: Power Functions →](05-power-functions.md)
