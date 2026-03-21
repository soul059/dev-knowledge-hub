# Activation Maximization — Visualizing What Neurons Prefer

> **Unit 9 · Visualizing CNNs · Topic 5/6**

---

## Table of Contents

1.  [Overview](#1-overview)
2.  [Core Idea — Optimization in Input Space](#2-core-idea--optimization-in-input-space)
3.  [Mathematical Formulation](#3-mathematical-formulation)
4.  [Regularization Techniques](#4-regularization-techniques)
5.  [Optimization Loop — Detailed Diagram](#5-optimization-loop--detailed-diagram)
6.  [Feature Visualization at Different Layers](#6-feature-visualization-at-different-layers)
7.  [DeepDream Connection](#7-deepdream-connection)
8.  [Class-Level Visualization](#8-class-level-visualization)
9.  [Worked Example — Step-by-Step Gradient Ascent](#9-worked-example--step-by-step-gradient-ascent)
10. [PyTorch Implementation — Activation Maximization](#10-pytorch-implementation--activation-maximization)
11. [PyTorch Implementation — DeepDream](#11-pytorch-implementation--deepdream)
12. [Limitations and Caveats](#12-limitations-and-caveats)
13. [Summary Table](#13-summary-table)
14. [Revision Questions](#14-revision-questions)
15. [Navigation](#15-navigation)
16. [References](#16-references)

---

## 1  Overview

Every neuron in a trained CNN has learned to detect a specific pattern in
its receptive field.  **Activation Maximization** is a technique that asks
the reverse question:

> *"What input image would make a particular neuron fire as strongly
> as possible?"*

Instead of feeding a real image forward and reading off activations, we
**start from noise** and iteratively modify the image so that a chosen
neuron's activation increases.  The result is a synthetic image that
reveals the neuron's **preferred stimulus** — the ideal pattern it has
learned to detect.

```
Traditional Forward Pass              Activation Maximization
─────────────────────────              ──────────────────────
  Real Image ──► Network ──► Read      Noise ──► Network ──► Read
                  activations                      activation
                                                      │
                                       Gradient  ◄────┘
                                       ascent on
                                       the INPUT
                                            │
                                       Modified ──► Network ──► Higher
                                       Image                    activation
                                            │
                                           ...        (repeat until
                                            ▼          convergence)
                                       Synthetic
                                       "Preferred"
                                       Image
```

### Why It Matters

| Benefit | Explanation |
|---|---|
| **Neuron-level understanding** | See exactly what pattern each filter has learned |
| **Layer-level understanding** | Compare early-layer vs. deep-layer features |
| **Class-level understanding** | Visualize the network's "ideal" representative of a class |
| **Debugging** | Detect spurious correlations the network has memorized |
| **Architecture comparison** | Compare features learned by different architectures |
| **Inspiration for DeepDream** | Same core idea powers artistic neural style transfer |

---

## 2  Core Idea — Optimization in Input Space

In standard training we optimize **weights** while keeping the input
fixed.  Activation maximization flips this: we **freeze the weights** and
optimize the **input image** to maximize a target activation.

```
┌─────────────────────────────────────────────────────────────┐
│                  STANDARD TRAINING                          │
│                                                             │
│   Fixed Input  ───►  Network(θ)  ───►  Loss                │
│                         ▲                 │                 │
│                         │     ∂L/∂θ       │                 │
│                         └─────────────────┘                 │
│                      update θ (gradient DESCENT)            │
├─────────────────────────────────────────────────────────────┤
│               ACTIVATION MAXIMIZATION                       │
│                                                             │
│   Variable x  ───►  Network(θ*)  ───►  a_l(x)              │
│       ▲               (frozen)            │                 │
│       │          ∂a_l/∂x                  │                 │
│       └───────────────────────────────────┘                 │
│                      update x (gradient ASCENT)             │
└─────────────────────────────────────────────────────────────┘
```

The objective is:

```
x* = argmax   a_l(x)  -  λ · R(x)
        x

where:
  x     = the input image (H × W × C), treated as a learnable variable
  a_l   = activation of the target neuron / channel / class logit
  R(x)  = regularization term (keeps the image natural-looking)
  λ     = regularization strength
  θ*    = frozen, pre-trained network weights
```

We solve this via **gradient ascent** — moving the input in the direction
that increases the target activation.

---

## 3  Mathematical Formulation

### 3.1  Objective Function

Let `f` be a trained CNN with frozen parameters `θ*`.  Given a target
neuron at layer `l`, channel `c`, spatial position `(i, j)`:

```
Objective:

  maximize   J(x) = a_{l,c,i,j}(x)  -  λ · R(x)
     x

where a_{l,c,i,j}(x) is the activation of the neuron at
layer l, channel c, spatial position (i, j) when input x
is fed through f.

For channel-level maximization (whole feature map):

  maximize   J(x) = (1/HW) Σ_i Σ_j  a_{l,c,i,j}(x)  -  λ · R(x)

For class-level maximization (output logit for class k):

  maximize   J(x) = z_k(x)  -  λ · R(x)

  where z_k is the pre-softmax logit for class k.
```

### 3.2  Gradient Ascent Update Rule

Starting from a random or zero-initialized image `x_0`, the update at
each iteration `t` is:

```
                   ∂ J(x_t)
x_{t+1} = x_t + η ─────────
                     ∂ x_t

Expanding:

                   ∂ a_l(x_t)         ∂ R(x_t)
x_{t+1} = x_t + η ──────────  - η·λ  ─────────
                      ∂ x_t             ∂ x_t

where:
  η  = step size (learning rate)
  λ  = regularization coefficient
```

### 3.3  Computing the Gradient

The gradient `∂a_l/∂x` is computed via **backpropagation** — exactly the
same mechanism used in training, but now we backpropagate from the
target activation all the way to the input pixels instead of stopping at
the first layer's weights.

```
Forward pass:
  x ──► conv1 ──► relu ──► conv2 ──► relu ──► ... ──► a_l
                                                        │
Backward pass (chain rule):                             │
                                                        ▼
  ∂a_l    ∂a_l     ∂a_l     ∂a_l                    scalar
  ──── ◄─ ──── ◄── ──── ◄── ──── ◄── ... ◄──────── activation
  ∂x      ∂h_1     ∂h_2     ∂h_3                    value
```

### 3.4  Optional: Gradient Normalization

To stabilize updates across layers with different gradient magnitudes,
we often normalize the gradient before applying it:

```
            g_t
x_{t+1} = x_t + η · ──────
                     ‖g_t‖₂

              ∂ a_l(x_t)
where g_t  =  ──────────  -  λ · ∇_x R(x_t)
                 ∂ x_t

This ensures a consistent step size regardless of activation scale.
```

---

## 4  Regularization Techniques

Without regularization, gradient ascent produces **high-frequency noise**
that technically maximizes the activation but is uninterpretable to
humans.  Regularization constrains the search to images with natural
statistics.

```
  No Regularization                With Regularization
  ┌───────────────────┐            ┌───────────────────┐
  │ ░▓█░▓█░▓█░▓█░▓█░ │            │                   │
  │ █░▓█░▓█░▓█░▓█░▓█ │            │    ╱╲   ╱╲        │
  │ ░▓█░▓█░▓█░▓█░▓█░ │            │   ╱  ╲ ╱  ╲       │
  │ █░▓█░▓█░▓█░▓█░▓█ │            │  ╱    ╳    ╲      │
  │ ░▓█░▓█░▓█░▓█░▓█░ │            │ ╱    ╱ ╲    ╲     │
  │ (adversarial noise)│            │(recognizable edge)│
  └───────────────────┘            └───────────────────┘
   Maximizes neuron ✓               Maximizes neuron ✓
   Human-readable   ✗               Human-readable   ✓
```

### 4.1  L2 Norm Penalty (Weight Decay on Pixels)

```
R_L2(x) = ‖x‖₂²  =  Σ_i  x_i²

Gradient:  ∇_x R_L2 = 2x

Effect:   Penalizes large pixel values, keeping the image
          close to the origin (gray).  Prevents extreme
          pixel intensities.

Update with L2:
  x_{t+1} = x_t + η · ∂a_l/∂x  -  η·λ·2·x_t
           = (1 - 2ηλ) · x_t  +  η · ∂a_l/∂x
```

### 4.2  Total Variation (TV) Penalty

```
R_TV(x) = Σ_{i,j} ( |x_{i+1,j} - x_{i,j}|² + |x_{i,j+1} - x_{i,j}|² )

Effect:   Penalizes high-frequency spatial changes
          (neighboring pixels with very different values).
          Promotes piecewise smooth images.

         Before TV                    After TV
  ┌─────────────────────┐     ┌─────────────────────┐
  │ 0.9 0.1 0.8 0.2    │     │ 0.5 0.5 0.6 0.6    │
  │ 0.2 0.8 0.1 0.9    │     │ 0.5 0.5 0.6 0.6    │
  │ (checkerboard noise)│     │ (smooth gradient)   │
  └─────────────────────┘     └─────────────────────┘
```

### 4.3  Gaussian Blur

Applied periodically during optimization (not a loss term, but a
transformation applied directly to the image every few iterations):

```
Every k iterations:
  x_t ← GaussianBlur(x_t, σ)

Effect:   Removes high-frequency noise that gradient ascent
          tends to introduce.  Acts as an implicit smoothness
          prior.

Kernel example (3×3, σ=0.5):
  ┌─────────────────┐
  │ 1/16  2/16  1/16│
  │ 2/16  4/16  2/16│
  │ 1/16  2/16  1/16│
  └─────────────────┘
```

### 4.4  Pixel Clipping

```
Every iteration:
  x_t ← clip(x_t, min_val, max_val)

Typical ranges:
  • [0, 1]     for normalized images
  • [-2, 2]    for ImageNet-normalized images
  • [-1, 1]    for centered images

Effect:   Hard constraint preventing pixel values from
          exploding to extreme ranges.
```

### 4.5  Spatial Jitter (Random Shift)

```
Before each forward pass:
  x_jittered = roll(x_t, shift=(Δh, Δw))     # random small shift
  compute a_l(x_jittered)
  backpropagate gradients
  apply gradient to x_t (unshifted)

Typical shift: Δh, Δw ∈ {-2, -1, 0, 1, 2}

Effect:   Makes features spatially robust, avoids the network
          exploiting exact pixel positions.  Produces more
          coherent spatial structure in the visualization.

┌──────────┐    jitter     ┌──────────┐
│ original │  ──(+2,+1)──► │ shifted  │ ──► forward pass
│  image   │               │  image   │     & backprop
└──────────┘               └──────────┘
      ▲                         │
      │    apply gradient       │
      └─────────────────────────┘
```

### 4.6  Summary of Regularization Effects

```
┌──────────────────┬───────────────────────────────────────────┐
│  Technique       │  What It Prevents                        │
├──────────────────┼───────────────────────────────────────────┤
│  L2 decay        │  Extreme pixel values far from gray      │
│  Total Variation │  High-frequency checkerboard artifacts   │
│  Gaussian blur   │  Fine-grained noise accumulation         │
│  Pixel clipping  │  Out-of-distribution pixel magnitudes    │
│  Spatial jitter  │  Position-specific artifacts             │
└──────────────────┴───────────────────────────────────────────┘

Typical combination for best results:
  L2 decay  +  Gaussian blur (every 4 steps)  +  clipping  +  jitter
```

---

## 5  Optimization Loop — Detailed Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                ACTIVATION MAXIMIZATION PIPELINE                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐                                                   │
│  │ Initialize x │  x_0 ~ N(0, 0.01) or uniform noise               │
│  │  (H × W × 3) │  shape matches network input (e.g. 224×224×3)    │
│  └──────┬───────┘                                                   │
│         │                                                           │
│         ▼                                                           │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  FOR t = 0, 1, 2, ..., T-1:                                 │   │
│  │  ┌────────────────────────────────────────────────────────┐  │   │
│  │  │ Step 1: SPATIAL JITTER                                 │  │   │
│  │  │   x_jit = roll(x_t, random shift Δh, Δw)              │  │   │
│  │  └───────────────────────┬────────────────────────────────┘  │   │
│  │                          ▼                                   │   │
│  │  ┌────────────────────────────────────────────────────────┐  │   │
│  │  │ Step 2: FORWARD PASS                                   │  │   │
│  │  │   a_l = f(x_jit; θ*)   [target neuron activation]     │  │   │
│  │  └───────────────────────┬────────────────────────────────┘  │   │
│  │                          ▼                                   │   │
│  │  ┌────────────────────────────────────────────────────────┐  │   │
│  │  │ Step 3: BACKWARD PASS                                  │  │   │
│  │  │   g_t = ∂a_l / ∂x_jit                                 │  │   │
│  │  └───────────────────────┬────────────────────────────────┘  │   │
│  │                          ▼                                   │   │
│  │  ┌────────────────────────────────────────────────────────┐  │   │
│  │  │ Step 4: NORMALIZE GRADIENT                             │  │   │
│  │  │   g_t = g_t / (‖g_t‖₂ + 1e-8)                        │  │   │
│  │  └───────────────────────┬────────────────────────────────┘  │   │
│  │                          ▼                                   │   │
│  │  ┌────────────────────────────────────────────────────────┐  │   │
│  │  │ Step 5: GRADIENT ASCENT UPDATE                         │  │   │
│  │  │   x_{t+1} = x_t + η · g_t                             │  │   │
│  │  └───────────────────────┬────────────────────────────────┘  │   │
│  │                          ▼                                   │   │
│  │  ┌────────────────────────────────────────────────────────┐  │   │
│  │  │ Step 6: REGULARIZATION                                 │  │   │
│  │  │   x_{t+1} = (1 - weight_decay) · x_{t+1}  [L2 decay] │  │   │
│  │  │   IF t % blur_every == 0:                              │  │   │
│  │  │       x_{t+1} = GaussianBlur(x_{t+1}, σ)              │  │   │
│  │  │   x_{t+1} = clip(x_{t+1}, lo, hi)         [clipping]  │  │   │
│  │  └───────────────────────┬────────────────────────────────┘  │   │
│  │                          │                                   │   │
│  │              ◄───── loop back ──────                         │   │
│  └──────────────────────────────────────────────────────────────┘   │
│         │                                                           │
│         ▼                                                           │
│  ┌──────────────┐                                                   │
│  │ POST-PROCESS │  Normalize to [0,1], denormalize if needed        │
│  │  & DISPLAY   │  x_T is the "preferred stimulus" for neuron l     │
│  └──────────────┘                                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6  Feature Visualization at Different Layers

Activation maximization reveals a clear **hierarchy of learned
representations** when applied to neurons at different depths:

```
Layer Depth       What Neurons Prefer          Example Visualization
───────────────────────────────────────────────────────────────────────

EARLY LAYERS      Low-level features           ┌─────────────┐
(conv1, conv2)    • Oriented edges             │ ╱╱╱╱╱╱╱╱╱╱╱ │
                  • Color gradients            │ ╱╱╱╱╱╱╱╱╱╱╱ │
                  • Gabor-like filters         │ ╱╱╱╱╱╱╱╱╱╱╱ │
                  • Simple textures            └─────────────┘
                                                (diagonal edges)
                                               ┌─────────────┐
                                               │░▓░▓░▓░▓░▓░▓ │
                                               │▓░▓░▓░▓░▓░▓░ │
                                               │░▓░▓░▓░▓░▓░▓ │
                                               └─────────────┘
                                                (checkerboard)

MIDDLE LAYERS     Mid-level features           ┌─────────────┐
(conv3, conv4)    • Textures (fur, scales)     │ ∿∿∿∿∿∿∿∿∿∿ │
                  • Repeating patterns         │ ∿∿∿∿∿∿∿∿∿∿ │
                  • Object parts (eyes, wheels)│ ∿∿∿∿∿∿∿∿∿∿ │
                  • Corners / junctions        └─────────────┘
                                                (wave texture)
                                               ┌─────────────┐
                                               │    ◉    ◉   │
                                               │      ▽      │
                                               │    ╲___╱    │
                                               └─────────────┘
                                                (face-like parts)

DEEP LAYERS       High-level features          ┌─────────────┐
(conv5, fc)       • Full objects               │    /\_/\     │
                  • Object classes             │   ( o.o )    │
                  • Scene compositions         │    > ^ <     │
                  • Semantic concepts           └─────────────┘
                                                (cat-like object)
                                               ┌─────────────┐
                                               │  ┌──┬──┐    │
                                               │  │  │  │    │
                                               │  ├──┼──┤    │
                                               │  │  │  │    │
                                               │  └──┴──┘    │
                                               └─────────────┘
                                                (grid / window)
```

### Hierarchy Diagram

```
Input ──► [Conv1] ──► [Conv2] ──► [Conv3] ──► [Conv4] ──► [Conv5] ──► [FC]
           │           │           │           │           │           │
           ▼           ▼           ▼           ▼           ▼           ▼
         Edges      Corners     Textures     Parts      Objects     Classes
         Colors     Patterns    Motifs       Faces      Scenes      Concepts
         ◄── simple ──────────────────────────────────── complex ──►
         ◄── local  ──────────────────────────────────── global  ──►
         ◄── low receptive field ─────────── large receptive field ──►
```

This hierarchy matches the findings of Zeiler & Fergus (2014) and
Olah et al. (2017), confirming that CNNs learn a progressively more
abstract feature hierarchy.

---

## 7  DeepDream Connection

**DeepDream** (Mordvintsev et al., 2015, Google) applies activation
maximization to a **real image** rather than starting from noise.
Instead of maximizing a single neuron, it maximizes the **L2 norm of
an entire layer's activations**, amplifying whatever patterns the
network already detects in the image.

### 7.1  DeepDream Objective

```
For a chosen layer l with activation tensor A_l(x) of shape (C, H, W):

  Objective:  maximize  ‖A_l(x)‖₂²  =  Σ_c Σ_i Σ_j  a_{l,c,i,j}(x)²

  Gradient:   ∂‖A_l‖² / ∂x   (backpropagated to input)

Intuition:  "Amplify whatever features the network already
             sees in this image."
```

### 7.2  Multi-Scale / Octave Processing

DeepDream processes the image at multiple scales (octaves) to produce
coherent patterns at different spatial frequencies:

```
┌──────────────────────────────────────────────────────────┐
│                  MULTI-OCTAVE DEEPDREAM                   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Original Image (e.g. 512 × 512)                        │
│         │                                                │
│         ▼                                                │
│  ┌─── Octave 1 ───┐                                     │
│  │ Downscale to    │   128 × 128                         │
│  │ smallest size   │──────────────────────┐              │
│  └─────────────────┘                      │              │
│                                           ▼              │
│                                  ┌──────────────────┐    │
│                                  │ Run N gradient   │    │
│                                  │ ascent steps at  │    │
│                                  │ this resolution  │    │
│                                  └────────┬─────────┘    │
│                                           │              │
│  ┌─── Octave 2 ───┐                      ▼              │
│  │ Upscale result  │   256 × 256                         │
│  │ to next size    │◄─────────────────────┘              │
│  │ + add detail    │                                     │
│  │ from original   │──────────────────────┐              │
│  └─────────────────┘                      │              │
│                                           ▼              │
│                                  ┌──────────────────┐    │
│                                  │ Run N gradient   │    │
│                                  │ ascent steps     │    │
│                                  └────────┬─────────┘    │
│                                           │              │
│  ┌─── Octave 3 ───┐                      ▼              │
│  │ Upscale result  │   512 × 512                         │
│  │ to full size    │◄─────────────────────┘              │
│  │ + add detail    │                                     │
│  │ from original   │──────────────────────┐              │
│  └─────────────────┘                      │              │
│                                           ▼              │
│                                  ┌──────────────────┐    │
│                                  │ Run N gradient   │    │
│                                  │ ascent steps     │    │
│                                  └────────┬─────────┘    │
│                                           │              │
│                                           ▼              │
│                                   Final Dream Image      │
│                                                          │
└──────────────────────────────────────────────────────────┘

Detail injection at each octave:
  detail = original_at_this_scale - upscaled_from_previous
  image  = upscaled_result + detail
```

### 7.3  DeepDream vs. Activation Maximization

```
┌──────────────────────┬────────────────────┬──────────────────────┐
│                      │ Act. Maximization  │ DeepDream            │
├──────────────────────┼────────────────────┼──────────────────────┤
│ Starting image       │ Random noise       │ Real photograph      │
│ Target               │ Single neuron/class│ Whole layer norm     │
│ Goal                 │ Understand neuron  │ Artistic effect      │
│ Multi-scale          │ Usually no         │ Yes (octaves)        │
│ Result               │ Abstract pattern   │ Psychedelic image    │
│ Regularization       │ Heavy (L2, blur…)  │ Light (clip only)    │
└──────────────────────┴────────────────────┴──────────────────────┘
```

---

## 8  Class-Level Visualization

When we maximize the **output class logit** (pre-softmax score) for a
specific class, we generate an image that represents the network's
"ideal" exemplar of that class.

```
Target:  z_k(x)  =  the logit for class k  (e.g., "golden retriever")

Objective:  x* = argmax  z_k(x) - λ · R(x)
                    x

Result:  A synthetic image showing what the network considers
         the most "golden-retriever-like" input.
```

### What Class Visualizations Reveal

```
  Class: "Golden Retriever"          Class: "Volcano"

  ┌───────────────────┐              ┌───────────────────┐
  │  Multiple fuzzy   │              │     ╱▲╲           │
  │  golden shapes    │              │    ╱██╲           │
  │  with eye-like    │              │   ╱████╲  flames  │
  │  patterns, snout  │              │  ╱██████╲ & smoke │
  │  shapes repeated  │              │ ╱████████╲        │
  │  across image     │              │    lava            │
  └───────────────────┘              └───────────────────┘

  The network's "platonic ideal"     Triangular shape + red-orange
  of a dog: fur texture + face       at top = volcano signature
  features, but NOT a real dog.

  Class: "Dumbbell"                  Class: "Cup"

  ┌───────────────────┐              ┌───────────────────┐
  │   ●═══════════●   │              │     ┌──────┐      │
  │   ●═══════════●   │              │     │      │      │
  │  (often with arm  │              │     │      ├──┐   │
  │   textures! The   │              │     │      │  │   │
  │   network learned │              │     └──────┘──┘   │
  │   arm+dumbbell)   │              │   (circular rim + │
  └───────────────────┘              │    handle shape)   │
                                     └───────────────────┘

  Reveals bias: dumbbells in          Network learned the
  training always had arms!           semantic shape of a cup.
```

### Interpretation Caveats

Class-level visualizations often look **alien and surreal** because:
- The network may rely on texture more than shape
- Multiple instances of features are tiled across the image
- Features from co-occurring objects may bleed in (e.g., arms with
  dumbbells, grass with dogs)

---

## 9  Worked Example — Step-by-Step Gradient Ascent

Consider a tiny network with a single convolutional filter followed by
ReLU, applied to a 1D "image" of 4 pixels.

### Network Setup

```
Filter w = [1, -1, 2]   (size 3, stride 1, no padding)
Activation: ReLU

Input x = [x₁, x₂, x₃, x₄]   (4 pixels, initialized to 0.1)

Convolution output (valid mode):
  h₁ = w₁·x₁ + w₂·x₂ + w₃·x₃ = 1·x₁ + (-1)·x₂ + 2·x₃
  h₂ = w₁·x₂ + w₂·x₃ + w₃·x₄ = 1·x₂ + (-1)·x₃ + 2·x₄

After ReLU:
  a₁ = max(0, h₁)
  a₂ = max(0, h₂)

Target: maximize a₁  (first output neuron)

Learning rate η = 0.1,  no regularization for simplicity.
```

### Iteration 0 (Initial State)

```
x = [0.1, 0.1, 0.1, 0.1]

h₁ = 1(0.1) + (-1)(0.1) + 2(0.1) = 0.1 - 0.1 + 0.2 = 0.2
a₁ = max(0, 0.2) = 0.2    ← target activation

Gradients (since a₁ > 0, ReLU passes gradients through):
  ∂a₁/∂x₁ = w₁ = 1
  ∂a₁/∂x₂ = w₂ = -1
  ∂a₁/∂x₃ = w₃ = 2
  ∂a₁/∂x₄ = 0              (x₄ doesn't affect h₁)

Gradient ascent update:
  x₁ ← 0.1 + 0.1 × 1  = 0.20
  x₂ ← 0.1 + 0.1 ×(-1)= 0.00
  x₃ ← 0.1 + 0.1 × 2  = 0.30
  x₄ ← 0.1 + 0.1 × 0  = 0.10
```

### Iteration 1

```
x = [0.20, 0.00, 0.30, 0.10]

h₁ = 1(0.20) + (-1)(0.00) + 2(0.30) = 0.20 + 0.00 + 0.60 = 0.80
a₁ = max(0, 0.80) = 0.80    ← increased from 0.2!  ✓

Gradients unchanged (same filter weights, ReLU still active):
  ∂a₁/∂x = [1, -1, 2, 0]

Update:
  x₁ ← 0.20 + 0.1 × 1  = 0.30
  x₂ ← 0.00 + 0.1 ×(-1)= -0.10
  x₃ ← 0.30 + 0.1 × 2  = 0.50
  x₄ ← 0.10 + 0.1 × 0  = 0.10
```

### Iteration 2

```
x = [0.30, -0.10, 0.50, 0.10]

h₁ = 1(0.30) + (-1)(-0.10) + 2(0.50) = 0.30 + 0.10 + 1.00 = 1.40
a₁ = max(0, 1.40) = 1.40    ← growing fast!  ✓

Update:
  x₁ ← 0.30 + 0.1 × 1  = 0.40
  x₂ ← -0.10 + 0.1×(-1)= -0.20
  x₃ ← 0.50 + 0.1 × 2  = 0.70
  x₄ ← 0.10 + 0.1 × 0  = 0.10
```

### Iteration 3

```
x = [0.40, -0.20, 0.70, 0.10]

h₁ = 1(0.40) + (-1)(-0.20) + 2(0.70) = 0.40 + 0.20 + 1.40 = 2.00
a₁ = max(0, 2.00) = 2.00    ← doubled since iteration 1!  ✓

Activation trajectory:
  Iter 0: a₁ = 0.20
  Iter 1: a₁ = 0.80   (+0.60)
  Iter 2: a₁ = 1.40   (+0.60)
  Iter 3: a₁ = 2.00   (+0.60)   ← linear growth (constant gradient)
```

### Observations

```
Pixel evolution:                 Interpretation:
  x₁: 0.10 → 0.20 → 0.30 → 0.40   ↑ grows (positive weight w₁=+1)
  x₂: 0.10 → 0.00 →-0.10 →-0.20   ↓ shrinks (negative weight w₂=-1)
  x₃: 0.10 → 0.30 → 0.50 → 0.70   ↑↑ grows fastest (largest weight w₃=+2)
  x₄: 0.10 → 0.10 → 0.10 → 0.10   — unchanged (not in receptive field)

The "preferred input" for this neuron:
  x₃ large & positive, x₁ moderate & positive, x₂ negative

Without regularization, x₃ → +∞ and x₂ → -∞.
This is why regularization (e.g., L2 decay) is critical!
```

---

## 10  PyTorch Implementation — Activation Maximization

```python
"""
Activation Maximization for a specific layer and channel in a pre-trained CNN.
Generates a synthetic image that maximally activates the target neuron.
"""

import torch
import torch.nn as nn
import torchvision.models as models
import torchvision.transforms as T
import matplotlib.pyplot as plt
import numpy as np
from PIL import Image
from scipy.ndimage import gaussian_filter


# ──────────────────────────────────────────────────────────
# 1. Load a pre-trained model and freeze all parameters
# ──────────────────────────────────────────────────────────
model = models.vgg16(weights=models.VGG16_Weights.IMAGENET1K_V1)
model.eval()

# Freeze all weights — we only optimize the input image
for param in model.parameters():
    param.requires_grad = False


# ──────────────────────────────────────────────────────────
# 2. Hook to capture intermediate activations
# ──────────────────────────────────────────────────────────
class ActivationHook:
    """Register a forward hook on a target layer to capture its output."""

    def __init__(self, layer):
        self.activation = None
        self.hook = layer.register_forward_hook(self._hook_fn)

    def _hook_fn(self, module, input, output):
        # Store the activation tensor (keep it in the compute graph)
        self.activation = output

    def close(self):
        self.hook.remove()


# ──────────────────────────────────────────────────────────
# 3. Regularization utilities
# ──────────────────────────────────────────────────────────
def apply_gaussian_blur(image_tensor, sigma=0.5):
    """Apply Gaussian blur to the image tensor (numpy-based)."""
    # image_tensor shape: (1, 3, H, W) — detach and convert
    img_np = image_tensor.detach().cpu().numpy()[0]   # (3, H, W)
    for c in range(3):
        img_np[c] = gaussian_filter(img_np[c], sigma=sigma)
    blurred = torch.from_numpy(img_np).unsqueeze(0)   # (1, 3, H, W)
    return blurred.to(image_tensor.device)


def total_variation_loss(x):
    """Compute the total variation of the image (encourages smoothness)."""
    # x shape: (1, C, H, W)
    tv_h = torch.sum((x[:, :, 1:, :] - x[:, :, :-1, :]) ** 2)
    tv_w = torch.sum((x[:, :, :, 1:] - x[:, :, :, :-1]) ** 2)
    return tv_h + tv_w


def spatial_jitter(x, max_shift=2):
    """Randomly shift the image by a small amount."""
    shift_h = torch.randint(-max_shift, max_shift + 1, (1,)).item()
    shift_w = torch.randint(-max_shift, max_shift + 1, (1,)).item()
    return torch.roll(x, shifts=(shift_h, shift_w), dims=(2, 3))


# ──────────────────────────────────────────────────────────
# 4. The main activation maximization function
# ──────────────────────────────────────────────────────────
def activation_maximization(
    model,
    target_layer,         # e.g., model.features[24] (conv layer)
    target_channel,       # which feature map channel to maximize
    img_size=224,         # spatial resolution
    n_steps=300,          # number of gradient ascent iterations
    lr=0.05,              # step size
    l2_weight=1e-4,       # L2 regularization strength
    tv_weight=1e-5,       # total variation regularization strength
    blur_every=4,         # apply Gaussian blur every N steps
    blur_sigma=0.5,       # Gaussian blur sigma
    clip_range=(-3, 3),   # pixel clipping bounds (for normalized image)
    jitter=2,             # spatial jitter range (pixels)
):
    """
    Perform activation maximization to generate a synthetic image
    that maximally activates the specified channel in the target layer.

    Returns:
        optimized_image: (H, W, 3) numpy array in [0, 1] range
        activation_history: list of activation values per step
    """

    device = next(model.parameters()).device

    # Initialize from random noise (small magnitude, centered at 0)
    input_img = torch.randn(1, 3, img_size, img_size, device=device) * 0.01
    input_img.requires_grad_(True)

    # Register hook on the target layer
    hook = ActivationHook(target_layer)
    activation_history = []

    for step in range(n_steps):
        # --- Step 1: Spatial jitter ---
        jittered = spatial_jitter(input_img, max_shift=jitter)

        # --- Step 2: Forward pass ---
        model(jittered)
        activation = hook.activation  # shape: (1, C, H', W')

        # Target: mean activation of the chosen channel across spatial dims
        target_act = activation[0, target_channel].mean()

        # --- Step 3: Compute regularization losses ---
        l2_loss = l2_weight * torch.norm(input_img)
        tv_loss = tv_weight * total_variation_loss(input_img)

        # Total objective: maximize activation, minimize regularization
        objective = target_act - l2_loss - tv_loss

        # --- Step 4: Backward pass ---
        model.zero_grad()
        if input_img.grad is not None:
            input_img.grad.zero_()
        objective.backward()

        # --- Step 5: Gradient ascent (with gradient normalization) ---
        grad = input_img.grad.data
        grad_norm = torch.norm(grad) + 1e-8
        input_img.data += lr * (grad / grad_norm)

        # --- Step 6: Apply regularization transforms ---
        # Gaussian blur every few steps
        if (step + 1) % blur_every == 0:
            input_img.data = apply_gaussian_blur(input_img, sigma=blur_sigma)

        # Pixel clipping
        input_img.data.clamp_(*clip_range)

        # Record progress
        activation_history.append(target_act.item())

        if (step + 1) % 50 == 0:
            print(f"Step {step+1:4d}/{n_steps}  "
                  f"Activation: {target_act.item():.4f}")

    hook.close()

    # --- Post-process: convert to displayable image ---
    result = input_img.data.cpu().numpy()[0]          # (3, H, W)
    result = np.transpose(result, (1, 2, 0))          # (H, W, 3)
    result = (result - result.min()) / (result.max() - result.min() + 1e-8)

    return result, activation_history


# ──────────────────────────────────────────────────────────
# 5. Run and visualize
# ──────────────────────────────────────────────────────────
if __name__ == "__main__":
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    model = model.to(device)

    # Target: channel 42 of VGG16 layer features[24]
    # (this is the 5th conv block, 1st conv layer)
    target_layer = model.features[24]
    target_channel = 42

    print(f"Target layer: {target_layer}")
    print(f"Target channel: {target_channel}")
    print(f"Device: {device}")
    print("-" * 50)

    img, history = activation_maximization(
        model,
        target_layer=target_layer,
        target_channel=target_channel,
        n_steps=300,
        lr=0.05,
    )

    # ---- Plotting ----
    fig, axes = plt.subplots(1, 2, figsize=(12, 5))

    # Left: generated image
    axes[0].imshow(img)
    axes[0].set_title(f"Activation Maximization\nLayer features[24], "
                      f"Channel {target_channel}")
    axes[0].axis("off")

    # Right: activation over iterations
    axes[1].plot(history)
    axes[1].set_xlabel("Iteration")
    axes[1].set_ylabel("Mean Activation")
    axes[1].set_title("Activation During Optimization")
    axes[1].grid(True, alpha=0.3)

    plt.tight_layout()
    plt.savefig("activation_maximization_result.png", dpi=150)
    plt.show()
```

### Class-Level Activation Maximization

```python
"""
Class-level activation maximization: generate an image that maximizes
the output logit for a specific ImageNet class.
"""

def class_activation_maximization(
    model,
    target_class,        # ImageNet class index (0–999)
    img_size=224,
    n_steps=500,
    lr=0.05,
    l2_weight=1e-4,
    tv_weight=1e-5,
    blur_every=4,
    blur_sigma=0.5,
    clip_range=(-3, 3),
    jitter=4,
):
    """Generate an image that maximizes the logit for target_class."""

    device = next(model.parameters()).device

    input_img = torch.randn(1, 3, img_size, img_size, device=device) * 0.01
    input_img.requires_grad_(True)

    activation_history = []

    for step in range(n_steps):
        # Spatial jitter
        jittered = spatial_jitter(input_img, max_shift=jitter)

        # Forward pass — get class logits (pre-softmax)
        logits = model(jittered)                    # (1, 1000)
        target_logit = logits[0, target_class]

        # Regularization
        l2_loss = l2_weight * torch.norm(input_img)
        tv_loss = tv_weight * total_variation_loss(input_img)

        objective = target_logit - l2_loss - tv_loss

        model.zero_grad()
        if input_img.grad is not None:
            input_img.grad.zero_()
        objective.backward()

        # Gradient ascent with normalization
        grad = input_img.grad.data
        input_img.data += lr * (grad / (torch.norm(grad) + 1e-8))

        # Regularization transforms
        if (step + 1) % blur_every == 0:
            input_img.data = apply_gaussian_blur(input_img, sigma=blur_sigma)
        input_img.data.clamp_(*clip_range)

        activation_history.append(target_logit.item())

        if (step + 1) % 100 == 0:
            print(f"Step {step+1:4d}/{n_steps}  "
                  f"Logit: {target_logit.item():.4f}")

    # Post-process
    result = input_img.data.cpu().numpy()[0]
    result = np.transpose(result, (1, 2, 0))
    result = (result - result.min()) / (result.max() - result.min() + 1e-8)

    return result, activation_history


# Example usage:
#   img, hist = class_activation_maximization(model, target_class=207)
#   # Class 207 = "golden retriever" in ImageNet
```

---

## 11  PyTorch Implementation — DeepDream

```python
"""
DeepDream implementation using PyTorch.
Amplifies patterns detected by a chosen layer in a pre-trained network.
"""

import torch
import torch.nn as nn
import torchvision.models as models
import torchvision.transforms as T
import numpy as np
from PIL import Image


# ──────────────────────────────────────────────────────────
# 1. Model setup
# ──────────────────────────────────────────────────────────
model = models.vgg16(weights=models.VGG16_Weights.IMAGENET1K_V1)
model.eval()
for param in model.parameters():
    param.requires_grad = False

# ImageNet normalization constants
MEAN = torch.tensor([0.485, 0.456, 0.406]).view(1, 3, 1, 1)
STD  = torch.tensor([0.229, 0.224, 0.225]).view(1, 3, 1, 1)


# ──────────────────────────────────────────────────────────
# 2. Image I/O utilities
# ──────────────────────────────────────────────────────────
def load_image(path, max_size=512):
    """Load an image, resize, and convert to a normalized tensor."""
    img = Image.open(path).convert("RGB")
    # Maintain aspect ratio, limit max dimension
    ratio = max_size / max(img.size)
    new_size = (int(img.size[0] * ratio), int(img.size[1] * ratio))
    img = img.resize(new_size, Image.LANCZOS)

    tensor = T.ToTensor()(img).unsqueeze(0)           # (1, 3, H, W)
    tensor = (tensor - MEAN) / STD                     # normalize
    return tensor


def tensor_to_image(tensor):
    """Convert a normalized tensor back to a displayable numpy array."""
    img = tensor.clone().detach().cpu()
    img = img * STD + MEAN                             # denormalize
    img = img.squeeze(0).permute(1, 2, 0).numpy()     # (H, W, 3)
    img = np.clip(img, 0, 1)
    return img


# ──────────────────────────────────────────────────────────
# 3. Core DeepDream step: maximize L2 norm of layer output
# ──────────────────────────────────────────────────────────
def deepdream_step(model, image, target_layer, n_steps=20, lr=0.01):
    """
    Run gradient ascent on the input image to amplify activations
    at the specified layer.

    Args:
        model:        Pre-trained model (frozen).
        image:        Input tensor (1, 3, H, W), normalized.
        target_layer: Layer whose activations to amplify.
        n_steps:      Gradient ascent iterations at this scale.
        lr:           Step size.

    Returns:
        Modified image tensor with amplified patterns.
    """
    input_img = image.clone().detach().requires_grad_(True)
    hook = ActivationHook(target_layer)

    for _ in range(n_steps):
        # Forward pass
        model(input_img)
        activation = hook.activation       # (1, C, H', W')

        # Objective: maximize L2 norm of all activations in this layer
        loss = activation.norm()

        # Backward pass
        model.zero_grad()
        if input_img.grad is not None:
            input_img.grad.zero_()
        loss.backward()

        # Gradient ascent with normalization
        grad = input_img.grad.data
        grad_norm = grad / (torch.mean(torch.abs(grad)) + 1e-8)
        input_img.data += lr * grad_norm

        # Clip to valid normalized range
        input_img.data.clamp_(-2.5, 2.5)

    hook.close()
    return input_img.detach()


# ──────────────────────────────────────────────────────────
# 4. Multi-octave DeepDream
# ──────────────────────────────────────────────────────────
def deepdream(
    model,
    image,
    target_layer,
    n_octaves=4,          # number of scales to process
    octave_scale=1.3,     # upscale factor between octaves
    n_steps=20,           # gradient ascent steps per octave
    lr=0.01,              # step size
):
    """
    Multi-scale DeepDream: process the image at multiple resolutions
    to produce coherent patterns at different spatial frequencies.

    The key idea:
      1. Create a pyramid of progressively smaller images.
      2. Start dreaming at the smallest scale.
      3. Upscale, inject high-frequency detail from the original,
         and dream again.
    """

    device = image.device
    original = image.clone()

    # ---- Build the octave pyramid (smallest → largest) ----
    octaves = [original]
    for _ in range(n_octaves - 1):
        h, w = octaves[-1].shape[2], octaves[-1].shape[3]
        new_h, new_w = int(h / octave_scale), int(w / octave_scale)
        if new_h < 32 or new_w < 32:
            break
        downscaled = nn.functional.interpolate(
            octaves[-1], size=(new_h, new_w),
            mode="bilinear", align_corners=False
        )
        octaves.append(downscaled)

    # Reverse so we process from smallest to largest
    octaves = octaves[::-1]

    # ---- Process each octave ----
    detail = torch.zeros_like(octaves[0])

    for i, octave_base in enumerate(octaves):
        h, w = octave_base.shape[2], octave_base.shape[3]
        print(f"Octave {i+1}/{len(octaves)}: {h} × {w}")

        # Upscale accumulated detail to current resolution
        if detail.shape[2] != h or detail.shape[3] != w:
            detail = nn.functional.interpolate(
                detail, size=(h, w),
                mode="bilinear", align_corners=False
            )

        # Inject detail into the base image at this scale
        input_img = (octave_base + detail).to(device)

        # Run gradient ascent at this scale
        dreamed = deepdream_step(
            model, input_img, target_layer,
            n_steps=n_steps, lr=lr
        )

        # Extract the detail (difference) added by dreaming
        detail = dreamed - octave_base

    return dreamed


# ──────────────────────────────────────────────────────────
# 5. Run DeepDream
# ──────────────────────────────────────────────────────────
if __name__ == "__main__":
    import matplotlib.pyplot as plt

    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    model = model.to(device)
    MEAN, STD = MEAN.to(device), STD.to(device)

    # Load an input photograph
    input_image = load_image("input_photo.jpg", max_size=512).to(device)

    # Choose which layer to amplify (deeper = more complex patterns)
    # VGG16 feature layers: 0-2 (early), 5-9 (mid), 17-28 (deep)
    target_layer = model.features[24]   # deep layer → complex patterns

    # Run multi-octave DeepDream
    dream_result = deepdream(
        model,
        input_image,
        target_layer,
        n_octaves=4,
        octave_scale=1.3,
        n_steps=30,
        lr=0.015,
    )

    # Display results
    fig, axes = plt.subplots(1, 2, figsize=(14, 6))

    axes[0].imshow(tensor_to_image(input_image.cpu()))
    axes[0].set_title("Original Image")
    axes[0].axis("off")

    axes[1].imshow(tensor_to_image(dream_result.cpu()))
    axes[1].set_title("DeepDream Result (layer features[24])")
    axes[1].axis("off")

    plt.tight_layout()
    plt.savefig("deepdream_result.png", dpi=150)
    plt.show()
```

---

## 12  Limitations and Caveats

### 12.1  Adversarial-Like Images

```
┌─────────────────────────────────────────────────────────────┐
│  KEY LIMITATION: Generated images often look adversarial    │
│                                                             │
│  The optimization finds inputs that exploit the network's   │
│  learned features — these may not correspond to patterns    │
│  humans would recognize.                                    │
│                                                             │
│  Human perception:  "random noise / psychedelic patterns"   │
│  Network says:      "99.7% confident this is a cat"         │
│                                                             │
│  This reveals a FUNDAMENTAL GAP between learned features    │
│  and human-interpretable concepts.                          │
└─────────────────────────────────────────────────────────────┘
```

### 12.2  Sensitivity to Regularization

```
 Too little regularization         Too much regularization
 ┌───────────────────┐              ┌───────────────────┐
 │ ░▓█▒░▓█▒░▓█▒░▓█▒ │              │                   │
 │ High-freq noise   │              │     (blank /       │
 │ that is hard to   │              │      washed-out    │
 │ interpret         │              │      gray image)   │
 │                   │              │                   │
 └───────────────────┘              └───────────────────┘
 Activation: very high              Activation: very low
 Interpretability: low              Interpretability: N/A
```

Choosing regularization hyperparameters requires careful tuning and is
**highly model-dependent**.

### 12.3  Full List of Limitations

| Limitation | Explanation |
|---|---|
| **Not unique** | Many different images can maximize the same neuron |
| **Hyperparameter sensitive** | Results vary dramatically with lr, regularization weights |
| **Texture bias** | CNNs often prefer texture over shape, skewing visualizations |
| **No spatial grounding** | The generated image may tile features unrealistically |
| **Computational cost** | Hundreds of forward/backward passes per visualization |
| **Model-specific** | Visualizations for VGG look different from ResNet, even for similar concepts |
| **May not reflect real usage** | The neuron's actual role during inference on real images may differ from its "ideal" stimulus |
| **Local optima** | Gradient ascent may converge to suboptimal solutions |

### 12.4  Mitigations

- **Combine with other methods**: Use alongside Grad-CAM, saliency maps,
  and feature inversion for a more complete picture.
- **Use advanced regularization**: Learned priors (e.g., GAN-based
  generators) produce much more realistic class visualizations
  (Nguyen et al., 2016).
- **Dataset-level analysis**: Look at many examples, not just one
  visualization, to draw conclusions about a neuron's role.

---

## 13  Summary Table

| Aspect | Details |
|---|---|
| **What it does** | Generates a synthetic image that maximally activates a target neuron, channel, or class output |
| **Direction** | Gradient **ascent** on the **input** image (weights frozen) |
| **Objective** | `x* = argmax_x  a_l(x) - λR(x)` |
| **Update rule** | `x_{t+1} = x_t + η · ∂a_l/∂x` |
| **Starting point** | Random noise (or real image for DeepDream) |
| **Regularization** | L2 decay, total variation, Gaussian blur, pixel clipping, spatial jitter |
| **Early layers** | Edges, colors, simple textures |
| **Middle layers** | Textures, patterns, object parts |
| **Deep layers** | Objects, scenes, semantic concepts |
| **Class-level** | Maximizes output logit — shows network's "ideal" class exemplar |
| **DeepDream** | Activation maximization on a real image, multi-octave processing |
| **Key limitation** | Generated images can look adversarial / unnatural |
| **Key papers** | Erhan et al. (2009), Simonyan et al. (2014), Mordvintsev et al. (2015), Olah et al. (2017) |
| **Use cases** | Neuron interpretation, model debugging, architecture comparison, art |

---

## 14  Revision Questions

**Q1.** In activation maximization, we perform gradient _ascent_ rather
than gradient descent. Explain why, and write out the update rule.

> **A1.** We want to _maximize_ the target activation, not minimize a
> loss. The update rule is: `x_{t+1} = x_t + η · (∂a_l/∂x_t)`.  We
> move the input in the direction that _increases_ the activation.

---

**Q2.** Why is regularization essential in activation maximization? What
happens without it?

> **A2.** Without regularization, gradient ascent produces
> high-frequency adversarial noise that technically maximizes the
> neuron's activation but is completely uninterpretable to humans.
> Regularization (L2 decay, total variation, Gaussian blur, clipping)
> constrains the solution to images with natural-looking statistics.

---

**Q3.** Compare what you would see when applying activation
maximization to a neuron in `conv1` versus a neuron in `conv5` of
VGG-16.

> **A3.** `conv1` neurons have small receptive fields and learn
> low-level features, so their preferred stimuli are simple edges,
> color blobs, or Gabor-like patterns. `conv5` neurons have large
> receptive fields and respond to high-level semantic concepts, so
> their preferred stimuli show object parts, full objects, or complex
> texture compositions.

---

**Q4.** How does DeepDream differ from standard activation
maximization? Describe both the starting point and the objective.

> **A4.** Standard activation maximization starts from **random noise**
> and maximizes a **single neuron or class logit**. DeepDream starts
> from a **real photograph** and maximizes the **L2 norm of an entire
> layer's activations** (amplifying whatever the network already
> detects). DeepDream also uses multi-octave processing for
> multi-scale coherence.

---

**Q5.** In the worked example (Section 9), pixel `x₃` grows the
fastest while `x₂` decreases. Relate this to the filter weights
`w = [1, -1, 2]`.

> **A5.** `x₃` has the largest positive weight (`w₃ = 2`), so the
> gradient `∂a₁/∂x₃ = 2` is the largest, causing `x₃` to grow the
> fastest. `x₂` has a negative weight (`w₂ = -1`), so the gradient
> pushes it negative to increase the activation (`(-1) × (-value) > 0`
> contributes positively to the activation).

---

**Q6.** A colleague generates a class-level activation maximization for
"dumbbell" and notices arm-like textures in the image. What does this
reveal about the training data or the model?

> **A6.** This reveals **dataset bias**: in the training set, dumbbells
> almost always appear with human arms holding them. The model has
> learned to associate arm textures with the dumbbell class, showing
> that it relies on context/co-occurrence rather than purely the object
> itself. This is a form of spurious correlation.

---

## 15  Navigation

| Previous | Up | Next |
|---|---|---|
| [← Saliency Maps](./04-saliency-maps.md) | [Unit 9 Index](../09-Visualizing-CNNs/) | [Understanding What CNNs Learn →](./06-understanding-cnns-learn.md) |

---

## 16  References

1. **Erhan, D., Bengio, Y., Courville, A., & Vincent, P.** (2009).
   *Visualizing Higher-Layer Features of a Deep Network*. University of
   Montreal Technical Report 1341.

2. **Simonyan, K., Vedaldi, A., & Zisserman, A.** (2014). *Deep Inside
   Convolutional Networks: Visualising Image Classification Models and
   Saliency Maps*. ICLR Workshop.

3. **Mordvintsev, A., Olah, C., & Tyka, M.** (2015). *Inceptionism:
   Going Deeper into Neural Networks*. Google Research Blog.

4. **Yosinski, J., Clune, J., Nguyen, A., Fuchs, T., & Lipson, H.**
   (2015). *Understanding Neural Networks Through Deep Visualization*.
   ICML Deep Learning Workshop.

5. **Nguyen, A., Dosovitskiy, A., Yosinski, J., Brox, T., & Clune, J.**
   (2016). *Synthesizing the Preferred Inputs for Neurons in Neural
   Networks via Deep Generator Networks*. NeurIPS.

6. **Olah, C., Mordvintsev, A., & Schubert, L.** (2017). *Feature
   Visualization*. Distill. https://distill.pub/2017/feature-visualization/

7. **Mahendran, A. & Vedaldi, A.** (2015). *Understanding Deep Image
   Representations by Inverting Them*. CVPR.

---

*End of Topic 5/6 — Activation Maximization*
