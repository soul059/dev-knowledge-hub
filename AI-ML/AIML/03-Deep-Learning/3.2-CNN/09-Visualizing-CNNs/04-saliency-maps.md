# Saliency Maps — Gradient-Based Input Attribution

> **Unit 9 · Visualizing CNNs · Topic 4/6**

---

## Table of Contents

1. [Overview](#1-overview)
2. [Vanilla Gradient Method](#2-vanilla-gradient-method)
3. [Mathematical Formulation](#3-mathematical-formulation)
4. [Forward / Backward Pass Diagram](#4-forward--backward-pass-diagram)
5. [SmoothGrad](#5-smoothgrad)
6. [Guided Backpropagation](#6-guided-backpropagation)
7. [Integrated Gradients](#7-integrated-gradients)
8. [Comparison of Methods](#8-comparison-of-methods)
9. [Worked Example — 4×4 Input](#9-worked-example--4×4-input)
10. [PyTorch Implementations](#10-pytorch-implementations)
11. [Limitations and Caveats](#11-limitations-and-caveats)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)
14. [Navigation](#14-navigation)
15. [References](#15-references)

---

## 1  Overview

A **saliency map** answers a deceptively simple question:

> *"Which pixels in the input image were most responsible for the
> network's prediction?"*

Instead of probing hidden layers (feature maps, Grad-CAM heatmaps),
saliency methods look directly at the **input space** and assign every
single pixel an **importance score**.

```
Input Image (H × W × 3)          Saliency Map (H × W)
┌─────────────────────┐           ┌─────────────────────┐
│                     │           │   ░░░               │
│    🐕  (dog)        │  ──────►  │  ░████░             │
│                     │           │  ░████░             │
│   grass / sky       │           │   ░░░               │
└─────────────────────┘           └─────────────────────┘
  raw RGB pixels                   bright = important
                                   dark   = irrelevant
```

### Why Saliency Maps Matter

| Benefit | Explanation |
|---|---|
| **Pixel-level resolution** | Attribution at each individual pixel, much finer than Grad-CAM |
| **Model debugging** | Reveals if a network is looking at the object or the background |
| **Adversarial detection** | Unusual saliency patterns may flag adversarial inputs |
| **Scientific insight** | In medical / satellite imagery, highlights which regions drive the decision |
| **Architecture-agnostic** | Works on any differentiable network, not just CNNs |

### Core Intuition

All gradient-based saliency methods rest on one idea:

```
If changing pixel x_ij by a tiny amount Δ causes a large change
in the class score y^c, then pixel x_ij is "important."

                ∂y^c
  importance ≈  ────   (gradient of class score w.r.t. input pixel)
                ∂x_ij
```

Different methods differ in **how** they compute, aggregate, or refine
this gradient signal — yielding cleaner, more theoretically grounded,
or more visually interpretable maps.

---

## 2  Vanilla Gradient Method

The simplest saliency technique, introduced by **Simonyan et al. (2014)**,
computes the gradient of the predicted class score with respect to
every input pixel, then takes the **absolute value**.

### Algorithm

```
VANILLA-GRADIENT SALIENCY
──────────────────────────
Input :  image  x  ∈  R^{H × W × 3}
         model  F
         target class  c

1.  Forward pass   :  y^c = F(x)[c]        (class-c logit or softmax)
2.  Backward pass  :  g  = ∂y^c / ∂x       (gradient w.r.t. input)
3.  Aggregate      :  S  = max_channel |g|  (take abs, max over RGB)

Output:  S  ∈  R^{H × W}   (saliency map)
```

### Channel Aggregation Strategies

```
Given gradient tensor  g ∈ R^{H × W × 3}

Strategy A — Max across channels (most common):
    S(i,j) = max( |g(i,j,R)|, |g(i,j,G)|, |g(i,j,B)| )

Strategy B — L2 norm across channels:
    S(i,j) = sqrt( g(i,j,R)² + g(i,j,G)² + g(i,j,B)² )

Strategy C — Sum of absolute values:
    S(i,j) = |g(i,j,R)| + |g(i,j,G)| + |g(i,j,B)|
```

### Properties

- ✅ **Extremely fast** — one forward + one backward pass
- ✅ **Easy to implement** — three lines of PyTorch
- ❌ **Very noisy** — gradients fluctuate wildly across nearby pixels
- ❌ **Gradient saturation** — ReLU kills gradients for inactive neurons
- ❌ **No theoretical completeness** — does not satisfy axioms like sensitivity

---

## 3  Mathematical Formulation

### Setup

```
Let:
  x ∈ R^{H × W × C}       input image (C = 3 for RGB)
  F : R^{H×W×C} → R^K      neural network mapping input to K class scores
  y = F(x) ∈ R^K            output logit vector
  y^c = F(x)_c              score for target class c
```

### Gradient Computation

The saliency map is defined as:

```
         ∂ y^c
S(x) =  ─────
          ∂ x

where S ∈ R^{H × W × C}
```

Using the **chain rule** through every layer:

```
∂ y^c     ∂ y^c     ∂ a^(L)    ∂ a^(L-1)          ∂ a^(1)
───── = ─────── · ──────── · ────────── · ... · ────────
 ∂ x    ∂ a^(L)   ∂ a^(L-1)  ∂ a^(L-2)            ∂ x

where a^(l) = activation at layer l
```

### Linearization Interpretation

Near input x, the network can be locally approximated as:

```
F(x + δ)_c  ≈  F(x)_c  +  S(x)^T · δ

This is a first-order Taylor expansion.

The saliency map S(x) gives the "slope" of the class score
in every input direction — which pixels, if perturbed by δ,
cause the largest change in y^c.
```

### For a Single Pixel

```
                 ∂ y^c
S(i, j, k) =  ─────────
               ∂ x(i,j,k)

where  i = row,  j = col,  k = channel (R, G, or B)

Scalar saliency (after aggregation over channels):

    s(i,j) = || S(i,j,:) ||_p      (typically p = ∞ i.e. max)
```

---

## 4  Forward / Backward Pass Diagram

```
FORWARD PASS (compute class score)
═══════════════════════════════════════════════════════════════════

  Input x                Conv1          ReLU       Conv2
 ┌──────┐   ┌───────┐   ┌──────┐   ┌──────┐   ┌───────┐
 │ H×W×3│──►│ W1,b1 │──►│ z^(1)│──►│a^(1) │──►│ W2,b2 │──►
 └──────┘   └───────┘   └──────┘   └──────┘   └───────┘
                                                    │
              ┌────────────────────────────────────┘
              ▼
          ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
          │ z^(2)│──►│a^(2) │──►│  GAP │──►│  FC  │──► y^c
          └──────┘   └──────┘   └──────┘   └──────┘
                                          (class score)


BACKWARD PASS (compute ∂y^c/∂x)
═══════════════════════════════════════════════════════════════════

  ∂y^c/∂x           ∂y^c/∂z^(1)     ∂y^c/∂a^(1)    ∂y^c/∂z^(2)
 ┌──────┐   ┌───────┐   ┌──────┐   ┌──────┐   ┌───────┐
 │RESULT│◄──│∂z1/∂x │◄──│∂a1/∂z│◄──│∂z2/∂a│◄──│∂a2/∂z │◄──
 └──────┘   └───────┘   └──────┘   └──────┘   └───────┘
  Saliency!  =W1^T        ReLU'      =W2^T       ReLU'
                         (0 or 1)               (0 or 1)
              ┌────────────────────────────────────┘
              │
          ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
     ◄──  │∂GAP  │◄──│  ∂FC │◄──│∂y^c/ │◄──│  1   │
          │      │   │ =W_fc│   │∂y^c  │   │(seed)│
          └──────┘   └──────┘   └──────┘   └──────┘
```

### Detailed Gradient Flow Through Each Layer Type

```
CONVOLUTION LAYER
─────────────────
Forward:   z^(l) = W^(l) * a^(l-1) + b^(l)
Backward:  ∂y^c/∂a^(l-1) = W^(l)^T  ★  ∂y^c/∂z^(l)
           (★ denotes transposed convolution / full correlation)

ReLU LAYER
──────────
Forward:   a^(l) = max(0, z^(l))
Backward:  ∂y^c/∂z^(l) = ∂y^c/∂a^(l) ⊙ 𝟙[z^(l) > 0]
           (element-wise mask: gradient passes only where input > 0)

FULLY-CONNECTED LAYER
─────────────────────
Forward:   z = W · a + b
Backward:  ∂y^c/∂a = W^T · ∂y^c/∂z

GLOBAL AVERAGE POOLING
──────────────────────
Forward:   a_k = (1/HW) Σ_{i,j} z_k(i,j)
Backward:  ∂y^c/∂z_k(i,j) = (1/HW) · ∂y^c/∂a_k
           (gradient distributed equally across spatial locations)
```

---

## 5  SmoothGrad

**SmoothGrad** (Smilkov et al., 2017) dramatically reduces noise by
averaging gradients computed on multiple **noisy copies** of the input.

### Core Idea

```
Vanilla gradient at a single point x is noisy because the
loss landscape has many local fluctuations.

Solution: sample nearby points and AVERAGE their gradients.

                     1   N
  SmoothGrad(x)  =  ─  Σ   ∂y^c / ∂(x + ε_i)
                     N  i=1

  where  ε_i ~ N(0, σ²I)     (isotropic Gaussian noise)
```

### Algorithm

```
SMOOTHGRAD
──────────
Input :  image x,  model F,  class c
         N = number of samples (typically 20-50)
         σ = noise standard deviation

1.  Initialize accumulator:  S_total = 0

2.  For i = 1 to N:
      a.  x̃_i = x + ε_i          where ε_i ~ N(0, σ²I)
      b.  y_i^c = F(x̃_i)[c]      forward pass on noisy copy
      c.  g_i  = ∂y_i^c / ∂x̃_i   backward pass
      d.  S_total += g_i

3.  S = S_total / N               average gradient

4.  Return |S| aggregated over channels

Output:  saliency map  S ∈ R^{H × W}
```

### Noise Standard Deviation Selection

```
The noise level σ is typically set as a fraction of the input range:

    σ = noise_level × (x_max - x_min)

Recommended values:
┌──────────────┬─────────────────────────────────────────┐
│ noise_level  │ Effect                                  │
├──────────────┼─────────────────────────────────────────┤
│ 0.05  (5%)   │ Mild smoothing, fast but still noisy    │
│ 0.10 (10%)   │ Good balance (most commonly used)       │
│ 0.15 (15%)   │ Aggressive smoothing, may blur edges    │
│ 0.20 (20%)   │ Too much — may distort attributions     │
└──────────────┴─────────────────────────────────────────┘

For ImageNet-normalized inputs (range ~[-2, 3]):
    σ ≈ 0.10 × 5 = 0.5

For [0, 1] normalized inputs:
    σ ≈ 0.10 × 1 = 0.1
```

### Visual Comparison

```
Vanilla Gradient                    SmoothGrad (N=50, σ=10%)
┌───────────────────┐               ┌───────────────────┐
│ ·░▓░·▒·░▓░·▒▓·░▒ │               │ ···░░░░░···░░···· │
│ ▓░·▒▓░▓░▒▓·░▒▓░▒ │               │ ··░▓▓▓▓░··░░····· │
│ ░▒▓░▒·░▒▓░▒▓░▒▓░ │               │ ·░▓████▓░·······  │
│ ·▒░▓▒░▓▒░▓▒░▓▒░▓ │               │ ·░▓████▓░·······  │
│ ▓░▒░▓▒░▓▒░▓▒░▓▒░ │               │ ··░▓▓▓▓░··░░····· │
│ ░▓▒░▓▒░▓▒░▓▒░▓▒░ │               │ ···░░░░░···░░···· │
└───────────────────┘               └───────────────────┘
     very noisy                        clean, focused
```

### Why SmoothGrad Works — Intuition

```
The gradient landscape near x is rough:

Score y^c
    │       ╱╲   ╱╲
    │ ╱╲  ╱╱  ╲╱╱  ╲╱╲    ← local fluctuations
    │╱  ╲╱              ╲
    └──────────────────────► pixel value

Gradient at a single point = slope at that point = noisy

SmoothGrad averages over a neighborhood:

Score y^c
    │
    │     ──────────
    │   ╱            ╲     ← averaged / smoothed trend
    │ ╱                ╲
    └──────────────────────► pixel value

Gradient of the smoothed function = clean, stable signal
```

---

## 6  Guided Backpropagation

**Guided Backpropagation** (Springenberg et al., 2015) modifies the
backward pass through ReLU layers to only propagate **positive**
gradients, producing sharper, edge-like attributions.

### Standard vs Guided ReLU Backward Pass

```
STANDARD BACKPROP THROUGH ReLU
──────────────────────────────
Forward:    a = max(0, z)
Backward:   ∂y^c     ∂y^c
            ─── =    ─── · 𝟙[z > 0]
            ∂z       ∂a

  → Only mask: was the input positive during forward pass?


GUIDED BACKPROPAGATION THROUGH ReLU
────────────────────────────────────
Forward:    a = max(0, z)
Backward:   ∂y^c     ∂y^c                    ∂y^c
            ─── =    ─── · 𝟙[z > 0] · 𝟙[ ─── > 0 ]
            ∂z       ∂a                      ∂a

  → Two masks:  (1) was input positive?  AND
                (2) is the incoming gradient positive?
```

### Gate Comparison Diagram

```
                         Forward          Backprop signal
                         input z          (from above)
                           │                   │
                           ▼                   ▼
                    ┌──────────────┐    ┌──────────────┐
                    │   z > 0 ?    │    │  grad > 0 ?  │
                    └──────┬───┬──┘    └──────┬───┬──┘
                       yes │   │ no       yes │   │ no
                           ▼   ▼              ▼   ▼

  Standard Backprop:     pass  block        pass  pass
                                           (no mask on grad sign)

  Guided Backprop:       pass  block        pass  block
                                           (ALSO mask negative grads)

  DeconvNet:             pass  pass         pass  block
                        (no mask on z)     (only mask grad sign)
```

### Intuition

```
Guided backpropagation says:

  "Only propagate gradient signal that corresponds to
   POSITIVE influences on the output."

  Negative gradients represent features that SUPPRESS
  the class score — we discard them because they act
  as noise when we want to highlight what the network
  IS looking for, not what it is avoiding.

Result: crisp, edge-like saliency maps that resemble
        a partial reconstruction of the input.
```

### Side-by-Side Visualization

```
Vanilla Gradient          Guided Backprop           Input Image
┌─────────────┐           ┌─────────────┐           ┌─────────────┐
│ ░▒▓░▒▓░▒▓░▒ │           │             │           │             │
│ ▒▓░▒▓░▒▓░▒▓ │           │   ╱──╲      │           │   /  \      │
│ ░▒█▒░▒█░▒█░ │           │  │ ○  │     │           │  ( ◉  )    │
│ ▓░▒▓░▒▓░▒▓░ │           │   ╲──╱      │           │   \  /      │
│ ░▒▓░▒▓░▒▓░▒ │           │             │           │             │
└─────────────┘           └─────────────┘           └─────────────┘
  noisy everywhere          sharp edges               original
```

### Caveats of Guided Backpropagation

- Produces visually appealing maps but **not class-discriminative**
  (the same edges appear regardless of target class)
- Acts more like an **edge detector** than a true attribution method
- Not faithful to the model's actual computation (modifies gradients)

---

## 7  Integrated Gradients

**Integrated Gradients** (Sundararajan et al., 2017) is a
theoretically rigorous method that computes the **path integral**
of gradients from a baseline input to the actual input.

### Definition

```
                                    1
IG_i(x) = (x_i - x'_i)  ×  ∫     ∂F(x' + α·(x - x'))_c
                              0    ─────────────────────── dα
                                         ∂x_i

where:
  x     = actual input image
  x'    = baseline (typically all-zeros / black image)
  α     = interpolation parameter ∈ [0, 1]
  F(·)_c = class-c score
  i     = index of a single input feature (pixel × channel)
```

### Path Integral — Visual Intuition

```
  Baseline x'                                    Input x
  (black image)                                  (actual image)
       │                                              │
       ▼                                              ▼
  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
  │         │    │  ░░░░   │    │  ▒▒▒▒   │    │  ████   │
  │  x'     │───►│x'+0.33Δ│───►│x'+0.67Δ│───►│   x    │
  │         │    │         │    │         │    │         │
  └─────────┘    └─────────┘    └─────────┘    └─────────┘
     α = 0         α = 0.33       α = 0.67       α = 1.0

  At each interpolation point, compute gradient ∂F/∂x.
  The integral = area under the gradient curve from α=0 to α=1.
  Multiply by (x - x') to get final attribution.
```

### Axiomatic Foundation

Integrated Gradients is the **unique** method satisfying:

```
AXIOM 1: SENSITIVITY
─────────────────────
If the input and baseline differ in exactly one feature,
and the network gives different outputs, then that feature
must receive a non-zero attribution.

  x differs from x' only in feature i
  F(x) ≠ F(x')
  ⟹  IG_i(x) ≠ 0

(Vanilla gradient can violate this if the gradient is zero
 at x due to saturation, even though the feature matters.)


AXIOM 2: IMPLEMENTATION INVARIANCE
───────────────────────────────────
Two networks that are functionally identical (same input-output
mapping) must produce identical attributions, regardless of
internal architecture.

  F₁(z) = F₂(z) for all z
  ⟹  IG(x; F₁) = IG(x; F₂)


AXIOM 3: COMPLETENESS (SUMMATION TO DELTA)
───────────────────────────────────────────
The sum of all attributions equals the difference between
the output at the input and the output at the baseline:

  Σ_i  IG_i(x) = F(x)_c - F(x')_c

This means attributions fully "explain" the prediction change.
```

### Numerical Approximation (Riemann Sum)

```
In practice, the integral is approximated with M steps:

                                   1    M        ∂F(x' + (k/M)·(x-x'))_c
IG_i(x) ≈ (x_i - x'_i)  ×  ───  Σ    ────────────────────────────────
                                  M   k=1              ∂x_i

Typical values:  M = 20 to 300 steps
More steps → more accurate, but slower.

Convergence check:
  Σ_i IG_i(x)  should ≈  F(x)_c - F(x')_c
  (If not close, increase M.)
```

### Baseline Selection

```
┌──────────────┬──────────────────────────────────────────────┐
│ Baseline     │ When to use                                  │
├──────────────┼──────────────────────────────────────────────┤
│ Black image  │ Default for natural images (x' = 0)          │
│ (all zeros)  │ Represents "absence of information"          │
├──────────────┼──────────────────────────────────────────────┤
│ White image  │ If black background is a feature itself      │
│ (all ones)   │                                              │
├──────────────┼──────────────────────────────────────────────┤
│ Blurred x    │ Removes texture while keeping structure      │
│              │ Useful for texture-sensitive tasks            │
├──────────────┼──────────────────────────────────────────────┤
│ Random noise │ Average IG over multiple random baselines    │
│              │ for robustness ("Expected Gradients")        │
└──────────────┴──────────────────────────────────────────────┘
```

---

## 8  Comparison of Methods

### Quality / Speed / Theory Trade-offs

```
                    Noise     Class-       Axiomatic    Computational
Method              Level   Discriminative  Rigor       Cost
────────────────────────────────────────────────────────────────────
Vanilla Gradient     High       Yes          Low        1 fwd + 1 bwd
SmoothGrad           Low        Yes          Low        N fwd + N bwd
Guided Backprop      Low        NO ✗         None       1 fwd + 1 bwd
Integrated Grads     Low        Yes          HIGH ✓     M fwd + M bwd
────────────────────────────────────────────────────────────────────

Where:  N = SmoothGrad samples (20-50)
        M = Integrated Gradients steps (50-300)
```

### Decision Flowchart

```
                  Need saliency map?
                        │
                        ▼
              ┌──── Speed critical? ────┐
              │                         │
             YES                       NO
              │                         │
              ▼                         ▼
     Vanilla Gradient          Need theoretical
     (fastest, noisiest)       guarantees?
                                   │
                          ┌────────┴────────┐
                         YES               NO
                          │                 │
                          ▼                 ▼
                  Integrated            Want cleaner
                  Gradients             vanilla map?
                  (gold standard)           │
                                   ┌───────┴───────┐
                                  YES              NO
                                   │                │
                                   ▼                ▼
                              SmoothGrad      Guided Backprop
                              (clean, fast)   (sharp edges, but
                                               not discriminative)
```

### Side-by-Side Output Sketch

```
Original    Vanilla    SmoothGrad   Guided BP   Integrated
 Image      Grad       (N=50)                    Gradients
┌──────┐   ┌──────┐   ┌──────┐    ┌──────┐    ┌──────┐
│ 🐱   │   │░▒▓▒░▓│   │  ▓▓  │    │ /──\ │    │  ▓▓  │
│      │   │▓░▒▓▒░│   │ ▓██▓ │    │| ◉◉ |│    │ ▓██▓ │
│ cat  │   │░▒▓▒░▓│   │ ▓██▓ │    │ \──/ │    │ ▓██▓ │
│      │   │▓░▒▓▒░│   │  ▓▓  │    │      │    │  ░░  │
└──────┘   └──────┘   └──────┘    └──────┘    └──────┘
            noisy      clean       sharp but    clean &
                       focused     same for     class-
                                   any class    specific
```

---

## 9  Worked Example — 4×4 Input

### Setup

```
Input x (4×4, single channel — grayscale for simplicity):

    x = ┌───┬───┬───┬───┐
        │ 1 │ 0 │ 2 │ 0 │
        ├───┼───┼───┼───┤
        │ 0 │ 3 │ 0 │ 1 │
        ├───┼───┼───┼───┤
        │ 2 │ 0 │ 1 │ 0 │
        ├───┼───┼───┼───┤
        │ 0 │ 1 │ 0 │ 2 │
        └───┴───┴───┴───┘

Tiny network:
  Conv (single 2×2 filter, no padding, stride 1) → ReLU → Flatten → FC (1 output)

Filter W:                   FC weights w_fc:
  ┌────┬────┐
  │ 1  │ -1 │               w_fc = [0.5, -0.3, 0.8, -0.2,
  ├────┼────┤                       0.1,  0.4, -0.6, 0.7, 0.3]
  │ -1 │  1 │
  └────┴────┘               bias b_fc = 0.1
bias b_conv = 0
```

### Step 1 — Forward Pass (Convolution)

```
Output size: (4-2+1) × (4-2+1) = 3 × 3

z(i,j) = Σ_{m,n} W(m,n) · x(i+m, j+n)

z(0,0) = 1·1 + (-1)·0 + (-1)·0 + 1·3 =  4
z(0,1) = 1·0 + (-1)·2 + (-1)·3 + 1·0 = -5
z(0,2) = 1·2 + (-1)·0 + (-1)·0 + 1·1 =  3
z(1,0) = 1·0 + (-1)·3 + (-1)·2 + 1·0 = -5
z(1,1) = 1·3 + (-1)·0 + (-1)·0 + 1·1 =  4
z(1,2) = 1·0 + (-1)·1 + (-1)·1 + 1·0 = -2
z(2,0) = 1·2 + (-1)·0 + (-1)·0 + 1·1 =  3
z(2,1) = 1·0 + (-1)·1 + (-1)·1 + 1·0 = -2
z(2,2) = 1·1 + (-1)·0 + (-1)·0 + 1·2 =  3

    z = ┌────┬────┬────┐
        │  4 │ -5 │  3 │
        ├────┼────┼────┤
        │ -5 │  4 │ -2 │
        ├────┼────┼────┤
        │  3 │ -2 │  3 │
        └────┴────┴────┘
```

### Step 2 — ReLU

```
    a = max(0, z) = ┌───┬───┬───┐
                    │ 4 │ 0 │ 3 │
                    ├───┼───┼───┤
                    │ 0 │ 4 │ 0 │
                    ├───┼───┼───┤
                    │ 3 │ 0 │ 3 │
                    └───┴───┴───┘

ReLU mask (1 where z > 0):
    M = ┌───┬───┬───┐
        │ 1 │ 0 │ 1 │
        ├───┼───┼───┤
        │ 0 │ 1 │ 0 │
        ├───┼───┼───┤
        │ 1 │ 0 │ 1 │
        └───┴───┴───┘
```

### Step 3 — Flatten + FC

```
Flatten a:  [4, 0, 3, 0, 4, 0, 3, 0, 3]

y = w_fc · a_flat + b_fc
  = 0.5·4 + (-0.3)·0 + 0.8·3 + (-0.2)·0 + 0.1·4
    + 0.4·0 + (-0.6)·3 + 0.7·0 + 0.3·3 + 0.1
  = 2.0 + 0 + 2.4 + 0 + 0.4 + 0 - 1.8 + 0 + 0.9 + 0.1
  = 4.0
```

### Step 4 — Backward Pass (Vanilla Gradient)

```
∂y/∂y = 1  (seed)

∂y/∂a_flat = w_fc = [0.5, -0.3, 0.8, -0.2, 0.1, 0.4, -0.6, 0.7, 0.3]

Reshape to 3×3:
∂y/∂a = ┌──────┬──────┬──────┐
        │  0.5 │ -0.3 │  0.8 │
        ├──────┼──────┼──────┤
        │ -0.2 │  0.1 │  0.4 │
        ├──────┼──────┼──────┤
        │ -0.6 │  0.7 │  0.3 │
        └──────┴──────┴──────┘

Through ReLU (multiply by mask M):
∂y/∂z = ∂y/∂a ⊙ M = ┌──────┬──────┬──────┐
                      │  0.5 │  0   │  0.8 │
                      ├──────┼──────┼──────┤
                      │  0   │  0.1 │  0   │
                      ├──────┼──────┼──────┤
                      │ -0.6 │  0   │  0.3 │
                      └──────┴──────┴──────┘

Through Convolution (transpose / "full" correlation with flipped W):
Flipped W = ┌────┬────┐
            │  1 │ -1 │     (W is symmetric here, so flip = W)
            ├────┼────┤
            │ -1 │  1 │
            └────┴────┘

∂y/∂x is computed by convolving ∂y/∂z (zero-padded) with flipped W:

Pad ∂y/∂z with zeros:
        ┌───┬──────┬──────┬──────┬───┐
        │ 0 │  0   │  0   │  0   │ 0 │
        ├───┼──────┼──────┼──────┼───┤
        │ 0 │  0.5 │  0   │  0.8 │ 0 │
        ├───┼──────┼──────┼──────┼───┤
        │ 0 │  0   │  0.1 │  0   │ 0 │
        ├───┼──────┼──────┼──────┼───┤
        │ 0 │ -0.6 │  0   │  0.3 │ 0 │
        ├───┼──────┼──────┼──────┼───┤
        │ 0 │  0   │  0   │  0   │ 0 │
        └───┴──────┴──────┴──────┴───┘

Compute ∂y/∂x(i,j) = Σ_{m,n} flipped_W(m,n) · padded(i+m, j+n):

∂y/∂x(0,0) = 1·0.5 + (-1)·0   + (-1)·0   + 1·0.1 =  0.6
∂y/∂x(0,1) = 1·0   + (-1)·0.8 + (-1)·0.1 + 1·0   = -0.9
∂y/∂x(0,2) = 1·0.8 + (-1)·0   + (-1)·0   + 1·0   =  0.8
∂y/∂x(0,3) = 1·0   + (-1)·0   + (-1)·0   + 1·0   =  0.0
∂y/∂x(1,0) = 1·0   + (-1)·0.5 + (-1)·(-0.6)+1·0   =  0.1
∂y/∂x(1,1) = 1·0.1 + (-1)·0   + (-1)·0   + 1·(-0.6)=-0.5
∂y/∂x(1,2) = 1·0   + (-1)·0.1 + (-1)·0.3 + 1·0   = -0.4
∂y/∂x(1,3) = 1·0   + (-1)·0   + (-1)·0   + 1·0.3 =  0.3
∂y/∂x(2,0) = 1·(-0.6)+(-1)·0  + (-1)·0   + 1·0   = -0.6
∂y/∂x(2,1) = 1·0   + (-1)·(-0.6)+(-1)·0  + 1·0   =  0.6
∂y/∂x(2,2) = 1·0.3 + (-1)·0   + (-1)·0   + 1·0   =  0.3
∂y/∂x(2,3) = 1·0   + (-1)·0.3 + (-1)·0   + 1·0   = -0.3
∂y/∂x(3,0) = 1·0   + (-1)·(-0.6)+(-1)·0  + 1·0   =  0.6
∂y/∂x(3,1) = 1·0   + (-1)·0   + (-1)·0   + 1·0   =  0.0
∂y/∂x(3,2) = 1·0   + (-1)·0   + (-1)·0.3 + 1·0   = -0.3
∂y/∂x(3,3) = 1·0   + (-1)·0   + (-1)·0   + 1·0   =  0.0

Gradient ∂y/∂x:                  Saliency |∂y/∂x|:
┌──────┬──────┬──────┬──────┐    ┌─────┬─────┬─────┬─────┐
│  0.6 │ -0.9 │  0.8 │  0.0 │    │ 0.6 │ 0.9 │ 0.8 │ 0.0 │
├──────┼──────┼──────┼──────┤    ├─────┼─────┼─────┼─────┤
│  0.1 │ -0.5 │ -0.4 │  0.3 │    │ 0.1 │ 0.5 │ 0.4 │ 0.3 │
├──────┼──────┼──────┼──────┤    ├─────┼─────┼─────┼─────┤
│ -0.6 │  0.6 │  0.3 │ -0.3 │    │ 0.6 │ 0.6 │ 0.3 │ 0.3 │
├──────┼──────┼──────┼──────┤    ├─────┼─────┼─────┼─────┤
│  0.6 │  0.0 │ -0.3 │  0.0 │    │ 0.6 │ 0.0 │ 0.3 │ 0.0 │
└──────┴──────┴──────┴──────┘    └─────┴─────┴─────┴─────┘
                                   ↑ most important: (0,1) = 0.9
```

### SmoothGrad on the Same Example

```
Suppose N = 3 noisy copies with σ = 0.2:

Copy 1 noise ε₁:                Copy 1 input x̃₁ = x + ε₁:
┌──────┬──────┬──────┬──────┐    ┌──────┬──────┬──────┬──────┐
│+0.10 │-0.05 │+0.15 │-0.20 │    │ 1.10 │-0.05 │ 2.15 │-0.20 │
│-0.12 │+0.08 │+0.03 │-0.11 │    │-0.12 │ 3.08 │ 0.03 │ 0.89 │
│+0.18 │-0.14 │+0.22 │+0.05 │    │ 2.18 │-0.14 │ 1.22 │ 0.05 │
│-0.07 │+0.19 │-0.09 │+0.16 │    │-0.07 │ 1.19 │-0.09 │ 2.16 │
└──────┴──────┴──────┴──────┘    └──────┴──────┴──────┴──────┘

Compute gradient g₁ = ∂y/∂x̃₁   (same procedure as above)
Repeat for copies 2 and 3.

         g₁ + g₂ + g₃
SG(x) = ──────────────   then take |SG(x)|
              3

The noisy oscillations in individual gradients cancel out,
leaving a smoother, more reliable attribution map.
```

---

## 10  PyTorch Implementations

### 10.1  Vanilla Gradient Saliency

```python
import torch
import torch.nn.functional as F
from torchvision import models, transforms
from PIL import Image
import matplotlib.pyplot as plt
import numpy as np


def vanilla_gradient_saliency(model, input_tensor, target_class):
    """
    Compute vanilla gradient saliency map.

    Args:
        model: pretrained CNN (e.g., ResNet-50)
        input_tensor: preprocessed image, shape (1, 3, H, W)
        target_class: integer class index for attribution

    Returns:
        saliency: numpy array of shape (H, W), normalized to [0, 1]
    """
    # Enable gradient computation on the input image
    input_tensor.requires_grad_(True)

    # Forward pass — compute class scores (logits)
    logits = model(input_tensor)                    # shape: (1, num_classes)

    # Select the score for the target class
    score = logits[0, target_class]                 # scalar

    # Backward pass — compute gradient of score w.r.t. input pixels
    score.backward()

    # Extract gradient: shape (1, 3, H, W) → (3, H, W)
    gradient = input_tensor.grad.data[0]            # (3, H, W)

    # Aggregate across channels: take max of absolute values
    saliency = gradient.abs().max(dim=0)[0]         # (H, W)

    # Normalize to [0, 1] for visualization
    saliency = (saliency - saliency.min()) / (saliency.max() - saliency.min() + 1e-8)

    return saliency.cpu().numpy()


# ── Usage Example ──────────────────────────────────────────────
# Load a pretrained model
model = models.resnet50(weights=models.ResNet50_Weights.IMAGENET1K_V1)
model.eval()

# Preprocessing pipeline (ImageNet normalization)
preprocess = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
])

# Load and preprocess image
img = Image.open("cat.jpg").convert("RGB")
input_tensor = preprocess(img).unsqueeze(0)         # (1, 3, 224, 224)

# Compute saliency for class 281 (tabby cat)
saliency = vanilla_gradient_saliency(model, input_tensor, target_class=281)

# Visualize
fig, axes = plt.subplots(1, 2, figsize=(10, 4))
axes[0].imshow(img)
axes[0].set_title("Original Image")
axes[1].imshow(saliency, cmap="hot")
axes[1].set_title("Vanilla Gradient Saliency")
for ax in axes:
    ax.axis("off")
plt.tight_layout()
plt.savefig("vanilla_saliency.png", dpi=150)
plt.show()
```

### 10.2  SmoothGrad

```python
def smoothgrad(model, input_tensor, target_class, n_samples=50,
               noise_level=0.10):
    """
    Compute SmoothGrad saliency map.

    Averages vanilla gradients over N noisy copies of the input.

    Args:
        model: pretrained CNN
        input_tensor: shape (1, 3, H, W)
        target_class: integer class index
        n_samples: number of noisy copies (default 50)
        noise_level: fraction of input range used as σ (default 0.10)

    Returns:
        saliency: numpy array (H, W), normalized to [0, 1]
    """
    model.eval()

    # Compute noise standard deviation from the input's value range
    x = input_tensor.detach()
    stdev = noise_level * (x.max() - x.min())

    # Accumulator for gradients
    total_grad = torch.zeros_like(x)                # (1, 3, H, W)

    for i in range(n_samples):
        # Create a noisy copy of the input
        noise = torch.randn_like(x) * stdev
        noisy_input = (x + noise).requires_grad_(True)

        # Forward pass
        logits = model(noisy_input)
        score = logits[0, target_class]

        # Backward pass
        model.zero_grad()
        score.backward()

        # Accumulate the gradient
        total_grad += noisy_input.grad.data

    # Average over all samples
    avg_grad = total_grad / n_samples               # (1, 3, H, W)

    # Aggregate: max of absolute gradient across channels
    saliency = avg_grad[0].abs().max(dim=0)[0]      # (H, W)

    # Normalize
    saliency = (saliency - saliency.min()) / (saliency.max() - saliency.min() + 1e-8)

    return saliency.cpu().numpy()


# ── Usage ──────────────────────────────────────────────────────
saliency_sg = smoothgrad(model, input_tensor, target_class=281,
                         n_samples=50, noise_level=0.10)
plt.imshow(saliency_sg, cmap="hot")
plt.title("SmoothGrad (N=50, σ=10%)")
plt.axis("off")
plt.show()
```

### 10.3  Guided Backpropagation

```python
class GuidedReLU(torch.autograd.Function):
    """
    Custom ReLU that masks BOTH:
      1. locations where input was negative (standard)
      2. locations where incoming gradient is negative (guided)
    """
    @staticmethod
    def forward(ctx, x):
        ctx.save_for_backward(x)
        return x.clamp(min=0)

    @staticmethod
    def backward(ctx, grad_output):
        x, = ctx.saved_tensors
        # Mask 1: forward input was positive
        # Mask 2: incoming gradient is positive
        return grad_output * (x > 0).float() * (grad_output > 0).float()


def replace_relu_with_guided(model):
    """
    Recursively replace all nn.ReLU modules with GuidedReLU.
    Works on any model architecture.
    """
    for name, module in model.named_children():
        if isinstance(module, torch.nn.ReLU):
            # Replace with a wrapper that calls GuidedReLU
            setattr(model, name, GuidedReLUModule())
        else:
            replace_relu_with_guided(module)


class GuidedReLUModule(torch.nn.Module):
    """nn.Module wrapper around the GuidedReLU autograd Function."""
    def forward(self, x):
        return GuidedReLU.apply(x)


def guided_backprop(model, input_tensor, target_class):
    """
    Compute guided backpropagation saliency map.

    Args:
        model: pretrained CNN (will be modified in-place — pass a deepcopy)
        input_tensor: shape (1, 3, H, W)
        target_class: integer class index

    Returns:
        saliency: numpy array (H, W), normalized to [0, 1]
    """
    import copy
    # Deep copy so we don't modify the original model
    guided_model = copy.deepcopy(model)
    guided_model.eval()

    # Replace all ReLU activations with GuidedReLU
    replace_relu_with_guided(guided_model)

    # Enable gradient on input
    x = input_tensor.detach().requires_grad_(True)

    # Forward pass
    logits = guided_model(x)
    score = logits[0, target_class]

    # Backward pass with guided gradients
    guided_model.zero_grad()
    score.backward()

    # Extract and aggregate gradient
    gradient = x.grad.data[0]                       # (3, H, W)
    saliency = gradient.abs().max(dim=0)[0]          # (H, W)

    # Normalize
    saliency = (saliency - saliency.min()) / (saliency.max() - saliency.min() + 1e-8)

    return saliency.cpu().numpy()


# ── Usage ──────────────────────────────────────────────────────
saliency_gbp = guided_backprop(model, input_tensor, target_class=281)
plt.imshow(saliency_gbp, cmap="hot")
plt.title("Guided Backpropagation")
plt.axis("off")
plt.show()
```

### 10.4  Integrated Gradients

```python
def integrated_gradients(model, input_tensor, target_class,
                         baseline=None, m_steps=200):
    """
    Compute Integrated Gradients attribution.

    Args:
        model: pretrained CNN
        input_tensor: shape (1, 3, H, W), the actual image
        baseline: shape (1, 3, H, W), reference input
                  (default: black image, all zeros)
        m_steps: number of interpolation steps for Riemann sum
        target_class: integer class index

    Returns:
        saliency: numpy array (H, W), normalized to [0, 1]
        convergence_delta: float, should be ≈ 0 if m_steps is enough
    """
    model.eval()

    # Default baseline: black image (all zeros, same shape as input)
    if baseline is None:
        baseline = torch.zeros_like(input_tensor)

    x = input_tensor.detach()
    x_prime = baseline.detach()

    # ── Step 1: Generate interpolated inputs along the straight-line path ──
    # alpha values from 0 to 1, shape (m_steps+1, 1, 1, 1) for broadcasting
    alphas = torch.linspace(0, 1, m_steps + 1).view(-1, 1, 1, 1)

    # Interpolated inputs: x' + α·(x - x')  for each α
    # Shape: (m_steps+1, 3, H, W)
    delta = x - x_prime                             # difference vector
    interpolated = x_prime + alphas * delta          # broadcast over steps

    # ── Step 2: Compute gradients at every interpolation point ──
    # Process in batches to avoid OOM on large models
    batch_size = 32
    all_grads = []

    for start in range(0, m_steps + 1, batch_size):
        end = min(start + batch_size, m_steps + 1)
        batch = interpolated[start:end].requires_grad_(True)

        logits = model(batch)
        scores = logits[:, target_class]             # (batch,)

        # Sum scores so we get one scalar for backward
        total_score = scores.sum()
        model.zero_grad()
        total_score.backward()

        all_grads.append(batch.grad.data.clone())    # (batch, 3, H, W)

    gradients = torch.cat(all_grads, dim=0)          # (m_steps+1, 3, H, W)

    # ── Step 3: Approximate integral via trapezoidal rule ──
    # Trapezoidal: average consecutive pairs, then mean
    avg_gradients = (gradients[:-1] + gradients[1:]) / 2.0   # (m_steps, 3, H, W)
    avg_gradients = avg_gradients.mean(dim=0)                 # (3, H, W)

    # ── Step 4: Scale by (x - x') ──
    ig = delta[0] * avg_gradients                    # (3, H, W)

    # ── Step 5: Convergence check (completeness axiom) ──
    with torch.no_grad():
        score_input = model(x)[0, target_class].item()
        score_baseline = model(x_prime)[0, target_class].item()
    expected_diff = score_input - score_baseline
    actual_sum = ig.sum().item()
    convergence_delta = abs(actual_sum - expected_diff)

    # ── Step 6: Aggregate and normalize ──
    saliency = ig.abs().max(dim=0)[0]                # (H, W)
    saliency = (saliency - saliency.min()) / (saliency.max() - saliency.min() + 1e-8)

    return saliency.cpu().numpy(), convergence_delta


# ── Usage ──────────────────────────────────────────────────────
saliency_ig, delta = integrated_gradients(
    model, input_tensor, target_class=281, m_steps=200
)
print(f"Convergence delta: {delta:.4f}")  # should be close to 0

plt.imshow(saliency_ig, cmap="hot")
plt.title(f"Integrated Gradients (M=200, Δ={delta:.3f})")
plt.axis("off")
plt.show()
```

### 10.5  All Methods — Unified Comparison Plot

```python
def plot_all_saliency_methods(model, input_tensor, img_pil, target_class):
    """
    Generate and display saliency maps from all four methods side by side.
    """
    import copy

    # Compute all four maps
    sal_vanilla = vanilla_gradient_saliency(model, input_tensor.clone(),
                                            target_class)

    sal_smooth  = smoothgrad(model, input_tensor.clone(), target_class,
                             n_samples=50, noise_level=0.10)

    sal_guided  = guided_backprop(model, input_tensor.clone(), target_class)

    sal_ig, delta = integrated_gradients(model, input_tensor.clone(),
                                         target_class, m_steps=200)

    # Plot
    fig, axes = plt.subplots(1, 5, figsize=(20, 4))
    titles = [
        "Original",
        "Vanilla Gradient",
        f"SmoothGrad (N=50)",
        "Guided Backprop",
        f"Integrated Grad (Δ={delta:.3f})",
    ]
    images = [np.array(img_pil), sal_vanilla, sal_smooth, sal_guided, sal_ig]
    cmaps  = [None, "hot", "hot", "hot", "hot"]

    for ax, title, image, cmap in zip(axes, titles, images, cmaps):
        ax.imshow(image, cmap=cmap)
        ax.set_title(title, fontsize=10)
        ax.axis("off")

    plt.tight_layout()
    plt.savefig("saliency_comparison.png", dpi=150, bbox_inches="tight")
    plt.show()


# ── Usage ──────────────────────────────────────────────────────
plot_all_saliency_methods(model, input_tensor, img, target_class=281)
```

---

## 11  Limitations and Caveats

### 11.1  Gradient Saturation

```
Problem:
  ReLU networks have piecewise-linear activation functions.
  Once a neuron saturates (output = 0), its gradient is exactly 0.
  This means the saliency map gives ZERO attribution to pixels
  that strongly influenced the neuron into saturation.

Example:
  Pixel x_ij = 100 pushes a neuron to output 50 (saturated ReLU region)
  Gradient ∂y/∂x_ij may still be computed correctly here since
  ReLU has gradient 1 for positive inputs, but in deeper layers
  the gradient can vanish through many ReLU gates.

                     activation
                        │    ╱
                        │   ╱ ReLU region:
                        │  ╱  gradient = 1
                        │ ╱
  ──────────────────────┼╱────── input z
     gradient = 0       │
     (dead region)      │

  For smooth activations (sigmoid, tanh), saturation is worse:
  gradient → 0 at extreme input values even when the feature matters.
```

### 11.2  Adversarial Manipulation of Saliency Maps

```
Adversarial saliency attacks can:

1. Make the model predict CORRECTLY while showing a MISLEADING
   saliency map (saliency points to irrelevant regions).

2. Make two different models with identical predictions show
   completely different saliency maps.

The attack works by adding a small perturbation δ to the input:

    x_adv = x + δ

such that:
    F(x_adv)  ≈  F(x)              (same prediction)
    S(x_adv)  ≠  S(x)              (completely different saliency)

This means saliency maps can be unreliable for high-stakes
decisions (medical, legal) where adversarial manipulation
is a concern.

Reference: Ghorbani et al., "Interpretation of Neural Networks
is Fragile" (2019)
```

### 11.3  Other Known Issues

```
┌─────────────────────────┬────────────────────────────────────────────┐
│ Issue                   │ Details                                    │
├─────────────────────────┼────────────────────────────────────────────┤
│ Class insensitivity     │ Guided backprop produces similar maps      │
│ (Guided BP)             │ regardless of target class — acts as       │
│                         │ an edge detector, not a class explainer    │
├─────────────────────────┼────────────────────────────────────────────┤
│ Baseline dependence     │ Integrated Gradients results change with   │
│ (IG)                    │ different baselines — no universally       │
│                         │ "correct" baseline exists                  │
├─────────────────────────┼────────────────────────────────────────────┤
│ Computational cost      │ SmoothGrad (N passes) and IG (M passes)   │
│                         │ are much slower than vanilla gradient      │
├─────────────────────────┼────────────────────────────────────────────┤
│ Resolution limit        │ Gradient-based maps are at input           │
│                         │ resolution but may miss higher-level       │
│                         │ semantic relationships                     │
├─────────────────────────┼────────────────────────────────────────────┤
│ Noisy for deep nets     │ Gradients through many layers accumulate   │
│                         │ noise — deeper models → noisier maps       │
├─────────────────────────┼────────────────────────────────────────────┤
│ Not causal              │ Saliency shows correlation (which pixels   │
│                         │ gradient is large) not causation (which    │
│                         │ pixels the model truly "uses")             │
└─────────────────────────┴────────────────────────────────────────────┘
```

### 11.4  Mitigation Strategies

```
1. Use MULTIPLE methods and compare:
   If vanilla, SmoothGrad, and IG all agree on a region → high confidence

2. Use complementary approaches:
   Combine gradient-based (pixel-level) with Grad-CAM (region-level)

3. Sanity checks (Adebayo et al., 2018):
   - Randomize model weights → saliency map should change dramatically
   - Randomize labels → saliency map should look random
   If saliency map does NOT change → method is unreliable

4. Increase IG steps / SmoothGrad samples until convergence:
   - IG: check completeness axiom  Σ IG_i ≈ F(x) - F(x')
   - SmoothGrad: variance of the map decreases with more samples
```

---

## 12  Summary Table

| Property | Vanilla Gradient | SmoothGrad | Guided Backprop | Integrated Gradients |
|---|---|---|---|---|
| **Core idea** | Raw ∂y^c/∂x | Average gradient over noisy copies | Mask negative gradients at ReLU | Path integral from baseline |
| **Visual quality** | Noisy | Clean | Sharp, edge-like | Clean, focused |
| **Class-discriminative** | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes |
| **Axiomatic guarantees** | ❌ None | ❌ None | ❌ None | ✅ Sensitivity + Completeness |
| **Completeness** | ❌ | ❌ | ❌ | ✅ Σ IG = F(x) − F(x') |
| **Passes required** | 1 fwd + 1 bwd | N × (fwd + bwd) | 1 fwd + 1 bwd | M × (fwd + bwd) |
| **Typical N or M** | — | 20–50 | — | 50–300 |
| **Hyperparameters** | None | σ, N | None | baseline, M |
| **Modifies backward** | No | No | Yes (ReLU gate) | No |
| **Fails sanity checks** | Sometimes | Sometimes | Often | Rarely |
| **Best use case** | Quick look | Production-quality maps | Visualization / debugging | Rigorous attribution |
| **Key paper** | Simonyan 2014 | Smilkov 2017 | Springenberg 2015 | Sundararajan 2017 |

---

## 13  Revision Questions

### Q1 — Conceptual

> **What does a saliency map tell us, and why is the gradient of the
> class score with respect to the input a reasonable measure of pixel
> importance?**

**Answer:** A saliency map assigns an importance score to each input
pixel indicating how much that pixel influences the network's
prediction for a given class. The gradient ∂y^c/∂x measures the
first-order sensitivity: a large gradient magnitude means a small
perturbation of that pixel causes a large change in the class score,
implying the network's decision is sensitive to — and therefore
"relying on" — that pixel.

---

### Q2 — SmoothGrad Derivation

> **Show that SmoothGrad is equivalent to computing the gradient of a
> Gaussian-smoothed version of the class score function.**

**Answer:**

```
Let  F_σ(x) = E_{ε~N(0,σ²I)}[ F(x + ε)_c ]

Then:
  ∂F_σ(x)     ∂                                   1   N
  ──────── =  ─── E[ F(x+ε)_c ] = E[ ∂F(x+ε)/∂x ] ≈ ─  Σ  ∂F(x+ε_i)/∂x
    ∂x        ∂x                                   N  i=1

This is exactly the SmoothGrad formula!

So SmoothGrad = vanilla gradient of a Gaussian-convolved score surface.
The convolution removes high-frequency noise, leaving the true signal.
```

---

### Q3 — Integrated Gradients Completeness

> **Prove the completeness axiom for Integrated Gradients:
> Σ_i IG_i(x) = F(x)_c − F(x')_c.**

**Answer:**

```
Σ_i IG_i(x) = Σ_i (x_i - x'_i) · ∫₀¹ (∂F/∂x_i)(x' + α(x-x')) dα

            = ∫₀¹  Σ_i (x_i - x'_i) · (∂F/∂x_i) dα

            = ∫₀¹  ∇F · (x - x') dα

            = ∫₀¹  d/dα [ F(x' + α(x - x'))_c ] dα
                   (by chain rule: dF/dα = ∇F · (x-x'))

            = F(x' + 1·(x-x'))_c - F(x' + 0·(x-x'))_c
                   (fundamental theorem of calculus)

            = F(x)_c - F(x')_c                          ∎
```

---

### Q4 — Guided Backpropagation Critique

> **Why do Adebayo et al. (2018) argue that Guided Backpropagation
> fails basic sanity checks? What is the implication?**

**Answer:** Adebayo et al. showed that if you progressively
randomize network weights from the top layer down, Guided
Backpropagation maps remain visually similar to those from the
trained network. A faithful attribution method should degrade as
the model is randomized (since a random model carries no learned
information). The fact that Guided Backprop maps barely change
implies they act primarily as an edge detector on the input
rather than reflecting what the network has actually learned.

---

### Q5 — Implementation

> **In the Integrated Gradients implementation, why do we use a
> trapezoidal rule instead of a simple left Riemann sum, and how
> do we verify the approximation is accurate?**

**Answer:** The trapezoidal rule averages the function values at the
left and right endpoints of each sub-interval, providing second-order
accuracy O(1/M²) compared to first-order O(1/M) for a simple Riemann
sum. This means far fewer steps are needed for the same accuracy. To
verify, we check the **completeness axiom**: Σ_i IG_i(x) should
approximately equal F(x)_c − F(x')_c. If the convergence delta is
large (say > 5% of the score difference), we increase M.

---

### Q6 — Practical

> **You compute a saliency map for a medical image classifier and find
> that the highest-attribution pixels are in the image corner (which
> contains a hospital logo), not on the pathology. What does this reveal
> and how would you address it?**

**Answer:** This reveals a **shortcut learning** / **spurious
correlation** problem: the model has learned to associate the hospital
logo with a particular diagnosis (e.g., if most positive cases come
from one hospital). To address this: (1) remove or mask the logo from
training data, (2) augment with random logos, (3) crop to the region
of interest, and (4) retrain and verify the saliency now highlights
clinically relevant features. This is one of the most valuable
practical uses of saliency maps — catching dataset biases before
deployment.

---

## 14  Navigation

| Previous | Up | Next |
|---|---|---|
| [← Grad-CAM](./03-grad-cam.md) | [Unit 9 Index](../09-Visualizing-CNNs/) | [Activation Maximization →](./05-activation-maximization.md) |

---

## 15  References

1. **Simonyan, K., Vedaldi, A., & Zisserman, A.** (2014).
   *"Deep Inside Convolutional Networks: Visualising Image Classification
   Models and Saliency Maps."*
   ICLR Workshop. — Introduced vanilla gradient saliency.

2. **Smilkov, D., Thorat, N., Kim, B., Viégas, F., & Wattenberg, M.** (2017).
   *"SmoothGrad: Removing Noise by Adding Noise."*
   arXiv:1706.03825. — Proposed averaging gradients over noisy copies.

3. **Springenberg, J. T., Dosovitskiy, A., Brox, T., & Riedmiller, M.** (2015).
   *"Striving for Simplicity: The All Convolutional Net."*
   ICLR Workshop. — Introduced Guided Backpropagation.

4. **Sundararajan, M., Taly, A., & Yan, Q.** (2017).
   *"Axiomatic Attribution for Deep Networks."*
   ICML. — Introduced Integrated Gradients with axiomatic justification.

5. **Adebayo, J., Gilmer, J., Muelly, M., Goodfellow, I., Hardt, M., & Kim, B.** (2018).
   *"Sanity Checks for Saliency Maps."*
   NeurIPS. — Showed many saliency methods fail randomization tests.

6. **Ghorbani, A., Abid, A., & Zou, J.** (2019).
   *"Interpretation of Neural Networks is Fragile."*
   AAAI. — Demonstrated adversarial manipulation of saliency maps.

7. **Baehrens, D., Schroeter, T., Harmeling, S., Kawanabe, M., Hansen, K., & Müller, K.-R.** (2010).
   *"How to Explain Individual Classification Decisions."*
   JMLR. — Early gradient-based explanation framework.

8. **Erhan, D., Bengio, Y., Courville, A., & Vincent, P.** (2009).
   *"Visualizing Higher-Layer Features of a Deep Network."*
   University of Montreal Technical Report. — Foundational visualization work.
