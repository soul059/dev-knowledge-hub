# Understanding What CNNs Learn — From Edges to Objects

> **Unit 9 · Visualizing CNNs · Topic 6/6**

---

## Table of Contents

1.  [Overview — Hierarchical Feature Learning](#1-overview--hierarchical-feature-learning)
2.  [Layer Hierarchy — What Each Layer Learns](#2-layer-hierarchy--what-each-layer-learns)
3.  [Feature Transferability](#3-feature-transferability)
4.  [Texture Bias vs Shape Bias](#4-texture-bias-vs-shape-bias)
5.  [Adversarial Examples](#5-adversarial-examples)
6.  [Feature Space Analysis — t-SNE / UMAP](#6-feature-space-analysis--t-sne--umap)
7.  [Ablation Studies](#7-ablation-studies)
8.  [Network Dissection](#8-network-dissection)
9.  [Worked Example — Layer-by-Layer Walk-Through](#9-worked-example--layer-by-layer-walk-through)
10. [PyTorch — Feature Extraction & t-SNE](#10-pytorch--feature-extraction--t-sne)
11. [PyTorch — FGSM Adversarial Attack](#11-pytorch--fgsm-adversarial-attack)
12. [Why Interpretability Matters](#12-why-interpretability-matters)
13. [Summary Table](#13-summary-table)
14. [Revision Questions](#14-revision-questions)
15. [Navigation](#15-navigation)
16. [References](#16-references)

---

## 1  Overview — Hierarchical Feature Learning

The single most important insight of deep learning is that a network
stacked with many layers learns a **hierarchy of representations**, where
each successive layer captures increasingly abstract features.  A CNN
does not jump from raw pixels to "cat" in one step; it builds up through
intermediate feature detectors that compose simple patterns into complex
ones.

```
 Hierarchical Feature Learning — The Big Picture
 ═══════════════════════════════════════════════════

  Pixels         Low-level       Mid-level       High-level      Output
 ┌──────┐      ┌──────────┐   ┌───────────┐   ┌────────────┐   ┌──────┐
 │ RGB  │─────►│  Edges   │──►│ Textures  │──►│  Object    │──►│Class │
 │values│      │  Colors  │   │ Corners   │   │  Parts     │   │Label │
 │      │      │  Blobs   │   │ Patterns  │   │  Wholes    │   │      │
 └──────┘      └──────────┘   └───────────┘   └────────────┘   └──────┘
                 Conv 1          Conv 2-3        Conv 4-5       FC / GAP

     ◄─────── Increasing abstraction / complexity ─────────►
     ◄─────── Increasing receptive field size     ─────────►
     ◄─────── Decreasing spatial resolution       ─────────►
```

Key observations from Zeiler & Fergus (2014) and subsequent work:

| Property               | Early Layers           | Later Layers             |
|-------------------------|------------------------|--------------------------|
| Feature type            | Edges, colors, blobs   | Object parts, full shapes|
| Receptive field         | Small (3×3 – 11×11)    | Large (up to full image) |
| Spatial resolution      | High                   | Low                      |
| Generality              | Universal across tasks | Task-specific            |
| Transfer learning value | Very high              | Moderate to low          |

This hierarchy emerges **automatically** from back-propagation — the
network discovers that composing simple features is the most efficient
way to reduce the training loss.

---

## 2  Layer Hierarchy — What Each Layer Learns

### 2.1  Layer 1 — Edges, Colors, Gabor-like Filters

The very first convolutional layer almost universally learns oriented
edge detectors and color-opponent filters.  These are strikingly similar
to the receptive fields of neurons in the primary visual cortex (V1).

```
  Examples of Conv1 Filters (5×5)
  ════════════════════════════════

  Horizontal Edge     Vertical Edge      Diagonal Edge      Color Opponent
  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
  │ 0  0  0  0 0│    │ 0  0 -1  0 0│    │-1 -1  0  0 0│    │ R  R  R  G G│
  │ 0  0  0  0 0│    │ 0  0 -1  0 0│    │-1  0  0  0 0│    │ R  R  R  G G│
  │-1 -1 -1 -1-1│    │ 0  0  0  0 0│    │ 0  0  0  0 0│    │ R  R  R  G G│
  │ 1  1  1  1 1│    │ 0  0  1  0 0│    │ 0  0  0  1 1│    │-G -G -G -R-R│
  │ 1  1  1  1 1│    │ 0  0  1  0 0│    │ 0  0  0  1 1│    │-G -G -G -R-R│
  └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
   Responds to ──     Responds to |      Responds to \      Responds to
   horizontal lines   vertical lines     diagonal lines      Red-vs-Green
```

### 2.2  Layer 2 — Textures, Corners, Simple Combinations

Layer 2 neurons combine Layer 1 edges into slightly more complex motifs:
corners, T-junctions, simple textures, and small circular arcs.

```
  How Conv2 Composes Conv1 Features
  ═══════════════════════════════════

  Conv1 filter A   Conv1 filter B         Conv2 neuron
  (horizontal ─)   (vertical |)          (corner ┐)
       ─                │                  ───┐
       ─                │                     │
       ─                │                     │
       ────  +  ────    │     ──────►     ────┤
                                              │

  Two edge detectors combine to form a corner detector.
  Similar compositions yield T-junctions, crosses, arcs.
```

### 2.3  Layer 3 — Patterns, Recurring Motifs, Object Parts

By Layer 3 the receptive field is large enough to span recognizable
patterns: grid textures, honeycomb patterns, simple repeating motifs,
and early object parts (e.g., wheels, eyes).

```
  Conv3 Feature Examples
  ══════════════════════

  Grid pattern        Circular motif      Text-like pattern
  ┌──┬──┬──┬──┐       ┌──────────┐        ┌──────────┐
  │  │  │  │  │       │  ╭────╮  │        │ ═══ ═══  │
  ├──┼──┼──┼──┤       │ │      │ │        │ ═══ ═══  │
  │  │  │  │  │       │ │      │ │        │          │
  ├──┼──┼──┼──┤       │  ╰────╯  │        │ ═══ ═══  │
  │  │  │  │  │       └──────────┘        └──────────┘
  └──┴──┴──┴──┘
```

### 2.4  Layers 4-5 — Object Parts and Whole Objects

The deepest convolutional layers respond to high-level semantic content:
dog faces, bicycle wheels, bird wings, text regions.

```
  Full Hierarchy — Composition Cascade
  ═════════════════════════════════════

  Layer 1          Layer 2           Layer 3           Layer 4-5
  ────────         ────────          ────────          ──────────
  ─  │  ╲ ╱       ┐ ┘ ┌ └          Eye shapes        Dog Face
                   T  +  ⌐          Fur textures      Whole Dog
  Color            Small arcs       Wheel spokes      Bicycle
  blobs            Parallel         Petal patterns    Flower
                   lines

      ◄──── Simple ──────────────── Complex ────►
      ◄──── Local  ──────────────── Global  ────►
```

### 2.5  Mathematical Perspective — Compositionality

Formally, each layer computes:

```
Layer l:

    a_l = σ( W_l * a_{l-1} + b_l )

where:
    a_l       = activation (feature map) at layer l
    W_l       = learned convolutional filters at layer l
    *         = convolution operator
    b_l       = bias terms
    σ         = non-linear activation (ReLU, etc.)

The full mapping from input x to output y composes L such operations:

    y = f_L ∘ f_{L-1} ∘ ... ∘ f_2 ∘ f_1 (x)

Each f_l is a non-linear transformation that re-represents the data
in a new coordinate system better suited for the downstream task.
```

The non-linearity σ is **critical** — without it the entire network
collapses to a single linear transformation and no hierarchy forms.

---

## 3  Feature Transferability

### 3.1  Core Finding

Yosinski et al. (2014) demonstrated one of the most practically
important results about CNNs: **early-layer features are general and
transfer well across tasks, while later-layer features are increasingly
task-specific**.

```
 Transferability Across Tasks
 ════════════════════════════

 Layer    Generality      Transfer performance
 ─────    ──────────      ────────────────────
  1       Very High       Barely drops when transferred
  2       High            Small drop
  3       Moderate        Noticeable drop
  4       Low             Significant drop
  5       Very Low        Large drop — task-specific features

 ┌──────────────────────────────────────────────────────┐
 │  Performance                                         │
 │  ▲                                                   │
 │  │ ████                                              │
 │  │ ████ ████                                         │
 │  │ ████ ████ ████                                    │
 │  │ ████ ████ ████ ████                               │
 │  │ ████ ████ ████ ████ ████                          │
 │  │ ████ ████ ████ ████ ████  ← Performance drop      │
 │  │ ████ ████ ████ ████ ████    when transferring     │
 │  └──────────────────────────────────► Layer          │
 │    L1    L2    L3    L4    L5                         │
 └──────────────────────────────────────────────────────┘
```

### 3.2  Why This Happens

```
 Task A: Cat vs Dog             Task B: Car vs Bus
 ────────────────────           ───────────────────

 Shared early layers:           Shared early layers:
   • Edge detectors               • Same edge detectors!
   • Color blobs                  • Same color blobs!
   • Texture filters              • Same texture filters!

 Divergent later layers:        Divergent later layers:
   • Fur textures                 • Metal/glass textures
   • Ear shapes                   • Wheel shapes
   • Snout detectors              • Headlight detectors
```

### 3.3  Practical Transfer Learning Recipe

```
 Transfer Learning Strategy
 ══════════════════════════

 Source Network (ImageNet-trained VGG-16):

 ┌─────────────────────────────────────────────────────────┐
 │ Conv1 │ Conv2 │ Conv3 │ Conv4 │ Conv5 │ FC6 │ FC7 │ FC8│
 │  64   │  128  │  256  │  512  │  512  │4096 │4096 │1000│
 └───┬───┴───┬───┴───┬───┴───┬───┴───┬───┴──┬──┴──┬──┴──┬─┘
     │       │       │       │       │      │     │     │
     ▼       ▼       ▼       ▼       ▼      ▼     ▼     ▼

 Small dataset:  FREEZE all    ─────────►  REPLACE & TRAIN
                 conv layers                  FC layers

 Medium dataset: FREEZE 1-3   ─► FINE-TUNE 4-5 ─► TRAIN FC

 Large dataset:  INITIALIZE   ─► FINE-TUNE ALL LAYERS
                 from pretrained
```

---

## 4  Texture Bias vs Shape Bias

### 4.1  The Surprising Discovery

Geirhos et al. (2019) showed that **standard ImageNet-trained CNNs
rely predominantly on texture rather than shape** to classify objects —
contrary to the assumption that CNNs learn shape-based representations
like humans do.

```
 Texture-Shape Conflict Experiment
 ══════════════════════════════════

 Original Images:

 ┌───────────┐        ┌───────────┐
 │  🐱 Cat   │        │  🐘 Eleph │
 │  (cat     │        │  (elephant│
 │   texture │        │   texture │
 │   + shape)│        │   + shape)│
 └───────────┘        └───────────┘

 Style Transfer → Cat shape + Elephant texture:

 ┌────────────────────┐
 │ Cat silhouette     │
 │ filled with        │
 │ elephant skin      │
 │ texture            │
 └────────────────────┘

 Human classification:  "Cat"       ← Shape bias
 CNN   classification:  "Elephant"  ← Texture bias !!

 The CNN "sees" the elephant skin texture and ignores the
 cat-shaped outline.
```

### 4.2  Why This Matters

| Aspect               | Texture Bias (CNN)              | Shape Bias (Human)            |
|------------------------|--------------------------------|-------------------------------|
| Robustness             | Fragile to texture changes     | Robust to texture changes     |
| Noise handling         | Easily confused by noise       | Robust to noise               |
| Generalization         | Poor on sketches/drawings      | Good on sketches/drawings     |
| Adversarial vuln.      | Highly vulnerable               | Naturally resistant           |
| Feature type           | Statistical / local textures   | Global geometric structure    |

### 4.3  Stylized-ImageNet Training

Geirhos et al. proposed training on **Stylized-ImageNet** (SIN) — a
dataset where every image has been style-transferred with a random
painting, destroying local texture while preserving global shape.

```
 Stylized-ImageNet Pipeline
 ══════════════════════════

  ImageNet Image            Random Art Style         Stylized Image
 ┌──────────────┐         ┌──────────────┐         ┌──────────────┐
 │              │         │              │         │              │
 │   [Dog]      │    +    │  [Monet      │   ──►   │   [Dog shape │
 │   photo      │         │   painting]  │         │    Monet     │
 │              │         │              │         │    texture]  │
 └──────────────┘         └──────────────┘         └──────────────┘

 Training on SIN forces the network to:
   ✓ Ignore local texture patterns (they are random)
   ✓ Learn global shape-based features instead
   ✓ Develop more human-like representations
```

Results from SIN training:

```
 Accuracy Comparison (% on various test sets)
 ═════════════════════════════════════════════

                        Standard     SIN-trained    Improvement
                        ResNet-50    ResNet-50
 ───────────────────   ──────────   ───────────   ───────────
 Clean ImageNet          76.1         79.3           +3.2
 Noise corruptions       ~45          ~58            +13
 Sketch recognition      ~18          ~32            +14
 Silhouettes             ~12          ~25            +13
 Shape-bias score        22%          81%            +59%
```

---

## 5  Adversarial Examples

### 5.1  Definition

An **adversarial example** is an input deliberately modified by a small,
often imperceptible, perturbation that causes the network to produce an
incorrect output with high confidence.

```
 Adversarial Example — The Classic Diagram
 ══════════════════════════════════════════

   Original Image       Perturbation          Adversarial Image
  ┌──────────────┐    ┌──────────────┐      ┌──────────────┐
  │              │    │              │      │              │
  │   [Panda]   │ +  │ ε·sign(∇L)  │  =   │   [Gibbon]   │
  │              │    │  (noise)     │      │              │
  │   57.7%     │    │              │      │   99.3% !!   │
  │   "panda"   │    │              │      │   "gibbon"   │
  └──────────────┘    └──────────────┘      └──────────────┘

  The perturbation is so small that humans cannot see it,
  yet the network's prediction changes entirely.
```

### 5.2  FGSM — Fast Gradient Sign Method

Goodfellow et al. (2015) proposed the simplest and most influential
adversarial attack: the **Fast Gradient Sign Method (FGSM)**.

```
 FGSM Formula
 ════════════

    x_adv = x + ε · sign(∇_x L(θ, x, y))

 where:
    x        = original input image
    x_adv    = adversarial image
    ε        = perturbation magnitude (e.g., 0.03 for [0,1] images)
    ∇_x L    = gradient of the loss with respect to the input
    sign(·)  = element-wise sign function (-1, 0, or +1)
    L(θ,x,y) = cross-entropy loss for model parameters θ,
                input x, true label y

 Step-by-step:
    1. Forward pass:  compute loss L(θ, x, y)
    2. Backward pass: compute ∂L/∂x  (gradient w.r.t. input pixels)
    3. Create perturbation: δ = ε · sign(∂L/∂x)
    4. Add to original: x_adv = x + δ
    5. Clip to valid range: x_adv = clamp(x_adv, 0, 1)
```

### 5.3  Adversarial Example Generation Pipeline

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║          ADVERSARIAL EXAMPLE GENERATION PIPELINE                    ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║                                                                     ║
 ║   ┌───────────┐      ┌──────────────────────────┐                   ║
 ║   │ Original  │      │    Trained CNN Model      │                   ║
 ║   │ Image x   │─────►│    (frozen weights θ)     │                   ║
 ║   │           │      │                           │                   ║
 ║   └───────────┘      └────────────┬─────────────┘                   ║
 ║        │                          │                                  ║
 ║        │                          ▼                                  ║
 ║        │              ┌──────────────────────┐                       ║
 ║        │              │  Forward Pass        │                       ║
 ║        │              │  ŷ = f(x; θ)         │                       ║
 ║        │              │  L = CrossEntropy(ŷ,y)│                      ║
 ║        │              └──────────┬───────────┘                       ║
 ║        │                         │                                   ║
 ║        │                         ▼                                   ║
 ║        │              ┌──────────────────────┐                       ║
 ║        │              │  Backward Pass       │                       ║
 ║        │              │  ∂L/∂x = ∇_x L      │                       ║
 ║        │              │  (gradient w.r.t.    │                       ║
 ║        │              │   input pixels)      │                       ║
 ║        │              └──────────┬───────────┘                       ║
 ║        │                         │                                   ║
 ║        │                         ▼                                   ║
 ║        │              ┌──────────────────────┐                       ║
 ║        │              │  Compute Perturbation│                       ║
 ║        │              │  δ = ε·sign(∇_x L)   │                       ║
 ║        │              └──────────┬───────────┘                       ║
 ║        │                         │                                   ║
 ║        ▼                         ▼                                   ║
 ║  ┌───────────┐         ┌─────────────────┐                           ║
 ║  │     x     │────+───►│ x_adv = x + δ   │                          ║
 ║  └───────────┘         │ clamp(x_adv,0,1)│                          ║
 ║                        └────────┬────────┘                           ║
 ║                                 │                                    ║
 ║                                 ▼                                    ║
 ║                     ┌────────────────────────┐                       ║
 ║                     │  Verify Misclassification│                     ║
 ║                     │  f(x_adv; θ) ≠ y         │                     ║
 ║                     │  with high confidence     │                     ║
 ║                     └────────────────────────┘                       ║
 ║                                                                     ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

### 5.4  Why Adversarial Examples Exist — The Linear Hypothesis

Goodfellow et al. argued that adversarial vulnerability is a consequence
of the **linear** nature of high-dimensional models:

```
 The Linear Hypothesis
 ═════════════════════

 Consider a linear model: ŷ = wᵀ x

 Adding perturbation δ = ε · sign(w):

    wᵀ (x + δ) = wᵀx + wᵀ(ε · sign(w))
                = wᵀx + ε · Σ|wᵢ|
                = wᵀx + ε · n · mean(|wᵢ|)

 Key insight: the perturbation to each pixel is tiny (ε),
 but the aggregate effect grows with dimensionality n.

 For a 224×224×3 image:  n = 150,528 dimensions
 Even ε = 0.01 creates:  total shift ≈ 0.01 × 150,528 × mean(|wᵢ|)
                         = a HUGE shift in output space

 Modern CNNs behave approximately linearly in high-dimensional
 input space, making them vulnerable to the same linear attack.
```

### 5.5  Defenses Against Adversarial Attacks

```
 Defense Strategies
 ══════════════════

 ┌────────────────────┬──────────────────────────────────────────────┐
 │ Defense             │ Description                                  │
 ├────────────────────┼──────────────────────────────────────────────┤
 │ Adversarial         │ Include adversarial examples in the          │
 │ Training            │ training set: min_θ E[max_δ L(θ, x+δ, y)]   │
 ├────────────────────┼──────────────────────────────────────────────┤
 │ Input               │ Apply JPEG compression, bit-depth            │
 │ Transformation      │ reduction, spatial smoothing                 │
 ├────────────────────┼──────────────────────────────────────────────┤
 │ Defensive           │ Train a denoising autoencoder to             │
 │ Distillation        │ remove perturbations before classification   │
 ├────────────────────┼──────────────────────────────────────────────┤
 │ Certified           │ Provably robust predictions within an        │
 │ Defenses            │ ε-ball using randomized smoothing            │
 ├────────────────────┼──────────────────────────────────────────────┤
 │ Ensemble            │ Average predictions from multiple models     │
 │ Adversarial         │ trained with diverse adversarial examples    │
 │ Training            │                                              │
 └────────────────────┴──────────────────────────────────────────────┘
```

Adversarial training remains the most reliable defense.  The min-max
formulation is:

```
 Adversarial Training Objective
 ══════════════════════════════

    min_θ  E_(x,y)~D [ max_{||δ||_∞ ≤ ε}  L(θ, x + δ, y) ]

 Inner maximization:  find the worst-case perturbation δ
                      within the ε-ball around x

 Outer minimization:  update model weights θ to be robust
                      against those worst-case perturbations

 Practical implementation (PGD — Projected Gradient Descent):
    For each training batch:
      1. Generate adversarial examples with k steps of PGD
      2. Train the network on those adversarial examples
      3. Repeat
```

---

## 6  Feature Space Analysis — t-SNE / UMAP

### 6.1  Idea

The penultimate layer of a CNN (the layer just before the final
classifier) produces a compact **feature vector** for each input image.
Visualizing these vectors with dimensionality reduction (t-SNE, UMAP)
reveals the network's internal organization of concepts.

```
 Feature Space Visualization Pipeline
 ═════════════════════════════════════

 ┌─────────┐    ┌───────────────────┐    ┌──────────┐    ┌──────────┐
 │ Dataset │───►│ CNN Feature       │───►│  t-SNE / │───►│ 2D Scatter│
 │ Images  │    │ Extractor         │    │  UMAP    │    │ Plot      │
 │ (N imgs)│    │ (penultimate      │    │ (reduce  │    │ (colored  │
 │         │    │  layer → Rd)      │    │  d → 2)  │    │ by class) │
 └─────────┘    └───────────────────┘    └──────────┘    └──────────┘

 Example output (t-SNE of 10 ImageNet classes):

      ▲ t-SNE dim 2
      │
      │     ○○○       ◇◇◇
      │    ○○○○○    ◇◇◇◇◇
      │     ○○○     ◇◇◇◇          ○ = dogs
      │                             ◇ = cats
      │  △△△△                       △ = cars
      │ △△△△△△       ■■■           ■ = ships
      │  △△△          ■■■■
      │                ■■■
      └─────────────────────────► t-SNE dim 1

 Observations:
   • Semantically similar classes cluster together
   • Dogs and cats are close (both animals)
   • Cars and ships are close (both vehicles)
   • Animals and vehicles are far apart
```

### 6.2  Nearest Neighbors: Feature Space vs Pixel Space

```
 Nearest Neighbors Comparison
 ════════════════════════════

 Query Image: [Photo of a golden retriever running on grass]

 Pixel-space nearest neighbors (L2 on raw pixels):
 ┌──────────────────────────────────────────────────────┐
 │  1. Green grass field (no dog)     ← similar colors  │
 │  2. Yellow car on green background ← similar colors  │
 │  3. Blurry golden photo           ← similar colors  │
 │  4. Sunset with green trees       ← similar colors  │
 └──────────────────────────────────────────────────────┘
 → Pixel neighbors match color/brightness, NOT semantics

 Feature-space nearest neighbors (L2 on CNN features):
 ┌──────────────────────────────────────────────────────┐
 │  1. Golden retriever sitting       ← same breed!     │
 │  2. Labrador retriever running     ← similar breed!  │
 │  3. Golden retriever puppy         ← same breed!     │
 │  4. Golden retriever in snow       ← same breed!     │
 └──────────────────────────────────────────────────────┘
 → Feature neighbors match semantic content, NOT pixels
```

### 6.3  t-SNE vs UMAP

```
 ┌─────────────────┬────────────────────┬────────────────────┐
 │ Property         │ t-SNE              │ UMAP               │
 ├─────────────────┼────────────────────┼────────────────────┤
 │ Speed            │ Slow (O(n²))       │ Fast (O(n log n))  │
 │ Preserves        │ Local structure    │ Local + some global│
 │ Reproducibility  │ Non-deterministic  │ More stable        │
 │ Scalability      │ ~10K points        │ ~1M points         │
 │ Hyperparameters  │ perplexity         │ n_neighbors,       │
 │                  │                    │ min_dist            │
 └─────────────────┴────────────────────┴────────────────────┘
```

---

## 7  Ablation Studies

Ablation studies systematically **remove or damage** parts of a network
to measure their contribution to overall performance.

### 7.1  Types of Ablation

```
 Ablation Study Taxonomy
 ═══════════════════════

 1. Layer ablation:     Remove entire layers
 2. Channel ablation:   Zero out specific filters/channels
 3. Neuron ablation:    Zero out individual neurons
 4. Weight ablation:    Set random weights to zero (pruning)
 5. Skip ablation:      Remove skip connections (in ResNets)

 ┌─────────────────────────────────────────────────────────┐
 │                                                         │
 │  Normal network:   L1 → L2 → L3 → L4 → L5 → Output    │
 │                                                         │
 │  Ablate Layer 3:   L1 → L2 → ██ → L4 → L5 → Output    │
 │                              ↑                          │
 │                         (zeroed out                     │
 │                          or identity)                   │
 │                                                         │
 │  Measure: accuracy drop = importance of Layer 3         │
 │                                                         │
 └─────────────────────────────────────────────────────────┘
```

### 7.2  Channel Ablation Results

```
 Example: Ablating channels in Conv5 of VGG-16
 ══════════════════════════════════════════════

 Channels removed    Accuracy    Drop from baseline (76.1%)
 ────────────────    ────────    ──────────────────────────
      0  (baseline)   76.1%      —
     50  (of 512)     75.8%      −0.3%   (high redundancy)
    100              74.9%      −1.2%
    200              72.1%      −4.0%
    300              66.3%      −9.8%
    400              53.2%     −22.9%   (critical channels)
    500              12.4%     −63.7%

 Key insight: many channels are redundant — removing 20%
 of channels barely affects accuracy.  This motivates
 network pruning for model compression.
```

---

## 8  Network Dissection

### 8.1  Quantifying Concept Alignment

**Network Dissection** (Bau et al., 2017) measures how well individual
neurons align with human-interpretable semantic concepts (objects,
parts, textures, scenes, materials, colors).

```
 Network Dissection Pipeline
 ═══════════════════════════

 Step 1: Collect concept masks from Broden dataset
 ┌──────────────────────────────────────────────┐
 │  Image              Concept Mask ("tree")     │
 │  ┌──────────┐      ┌──────────┐              │
 │  │  🌳  🏠  │      │ ██       │              │
 │  │       🌳 │      │       ██ │              │
 │  │  🚗     │      │          │              │
 │  └──────────┘      └──────────┘              │
 └──────────────────────────────────────────────┘

 Step 2: For each neuron k in layer l, compute activation map
 ┌──────────────────────────────────────────────┐
 │  Activation map A_k     Thresholded M_k       │
 │  ┌──────────┐           ┌──────────┐          │
 │  │ 0.2  0.9 │           │      ██  │          │
 │  │ 0.1  0.8 │  T_k=0.5  │      ██  │          │
 │  │ 0.3  0.1 │  ──────►  │          │          │
 │  └──────────┘           └──────────┘          │
 └──────────────────────────────────────────────┘

 Step 3: Compute IoU between M_k and each concept mask
```

### 8.2  IoU Score

```
 Intersection over Union (IoU)
 ═════════════════════════════

                |M_k ∩ C_concept|
 IoU(k, c) = ─────────────────────
                |M_k ∪ C_concept|

 where:
    M_k        = thresholded activation map of neuron k
    C_concept  = binary mask for semantic concept c

 A neuron is considered a "detector" for concept c if:

    IoU(k, c) > threshold   (typically 0.04)

 Higher IoU → stronger alignment with the concept.
```

### 8.3  Key Results

```
 Concept Detectors Found in ResNet (examples)
 ═════════════════════════════════════════════

 Layer       Neuron    Best Concept    IoU
 ─────────   ──────    ────────────    ────
 Conv1        #23      Vertical edge   0.31
 Conv2        #67      Brick texture   0.18
 Conv3       #142      Tree            0.14
 Conv4       #301      Dog head        0.11
 Conv5       #488      Waterfall       0.09
 Conv5       #211      Sky             0.22

 Trend: early layers → textures/colors
        later layers → objects/scenes
```

### 8.4  Interpretability Score

```
 The fraction of neurons in a layer that match ANY concept
 with IoU > threshold is the layer's "interpretability score":

                  # neurons matching a concept
 Interp(l) = ──────────────────────────────────
                   total neurons in layer l

 Typical values:
    • Conv1:  ~80% interpretable
    • Conv3:  ~50% interpretable
    • Conv5:  ~30% interpretable  (many neurons encode
                                   distributed/polysemantic
                                   features not in the
                                   concept vocabulary)
```

---

## 9  Worked Example — Layer-by-Layer Walk-Through

Let us trace what a VGG-16 network "sees" when processing an image
of a **golden retriever sitting on grass**.

```
 Input Image: Golden retriever on grass (224 × 224 × 3)
 ════════════════════════════════════════════════════════

 ┌────────────────────────────────────────────┐
 │              ╱╲                            │
 │             ╱  ╲    ← ears                 │
 │            │ ◉◉ │   ← eyes                │
 │            │  ▼  │   ← nose                │
 │            │ ═══ │   ← mouth               │
 │           ╱│    │╲                          │
 │          ╱ │    │ ╲  ← fur body            │
 │         │  │    │  │                       │
 │         │  └────┘  │                       │
 │   ░░░░░░│░░░░░░░░░░│░░░░░░░░░░ ← grass    │
 │   ░░░░░░░░░░░░░░░░░░░░░░░░░░░░            │
 └────────────────────────────────────────────┘
```

### Layer-by-Layer Progression

```
 Conv1 (64 filters, 3×3) → 224 × 224 × 64
 ──────────────────────────────────────────
 What activates:
   • Filter #3:   Horizontal edges → grass/body boundary
   • Filter #17:  Vertical edges   → sides of face
   • Filter #42:  Green channel    → grass region lights up
   • Filter #58:  Orange/yellow    → fur region lights up

 Conv2 (128 filters, 3×3) → 112 × 112 × 128
 ──────────────────────────────────────────
 What activates:
   • Filter #22:  Fur-like texture  → body and head
   • Filter #89:  Corners           → eye sockets, ear tips
   • Filter #104: Grass texture     → lower half of image

 Conv3 (256 filters, 3×3) → 56 × 56 × 256
 ──────────────────────────────────────────
 What activates:
   • Filter #67:  Eye-like circles  → both eye locations
   • Filter #143: Ear-like curves   → ear positions
   • Filter #201: Grass pattern     → ground region

 Conv4 (512 filters, 3×3) → 28 × 28 × 512
 ──────────────────────────────────────────
 What activates:
   • Filter #45:  Dog face          → head region strongly
   • Filter #312: Snout/nose        → center of face
   • Filter #401: Fluffy fur body   → torso region

 Conv5 (512 filters, 3×3) → 14 × 14 × 512
 ──────────────────────────────────────────
 What activates:
   • Filter #78:  "Dog" concept     → entire dog region
   • Filter #234: Outdoor scene     → whole image, weakly
   • Filter #456: "Golden retriever"→ very strong on head+body

 FC / Classifier
 ──────────────────────────────────────────
 Top-5 predictions:
   1. Golden retriever    92.3%
   2. Labrador retriever   4.1%
   3. Cocker spaniel       1.2%
   4. Irish setter         0.8%
   5. Tennis ball          0.3%    ← spurious (grass color)
```

```
 Spatial Resolution Decrease Through Layers
 ═══════════════════════════════════════════

 Layer    Size          Channels    Receptive Field
 ─────    ──────────    ────────    ───────────────
 Input    224 × 224         3        1 × 1
 Conv1    224 × 224        64        3 × 3
 Pool1    112 × 112        64        4 × 4
 Conv2    112 × 112       128       10 × 10
 Pool2     56 ×  56       128       16 × 16
 Conv3     56 ×  56       256       40 × 40
 Pool3     28 ×  28       256       48 × 48
 Conv4     28 ×  28       512      100 × 100
 Pool4     14 ×  14       512      116 × 116
 Conv5     14 ×  14       512      196 × 196
 Pool5      7 ×   7       512      212 × 212
```

---

## 10  PyTorch — Feature Extraction & t-SNE

```python
"""
Feature Extraction and t-SNE Visualization
===========================================
Extract penultimate-layer features from a pre-trained CNN and
project them to 2-D with t-SNE for visual analysis.
"""

import torch
import torch.nn as nn
import torchvision.models as models
import torchvision.transforms as transforms
import torchvision.datasets as datasets
from torch.utils.data import DataLoader
import numpy as np
import matplotlib.pyplot as plt
from sklearn.manifold import TSNE

# ── 1. Load a pre-trained ResNet-18 ─────────────────────────────────
model = models.resnet18(weights=models.ResNet18_Weights.IMAGENET1K_V1)
model.eval()

# ── 2. Register a hook on the penultimate layer (avgpool) ───────────
# We will collect the 512-dimensional feature vector produced by the
# global average pooling layer, just before the final FC classifier.

features_store = []   # list to accumulate feature vectors
labels_store   = []   # list to accumulate ground-truth labels

def hook_fn(module, input, output):
    """Hook that captures the output of a layer."""
    # output shape: (batch_size, 512, 1, 1) → squeeze to (batch_size, 512)
    features_store.append(output.squeeze().detach().cpu().numpy())

hook_handle = model.avgpool.register_forward_hook(hook_fn)

# ── 3. Prepare a small dataset (CIFAR-10 for simplicity) ────────────
# CIFAR-10 images are 32×32, but ResNet expects 224×224.
# We resize up — not ideal for production, but fine for demonstration.

transform = transforms.Compose([
    transforms.Resize((224, 224)),              # upscale for ResNet
    transforms.ToTensor(),                       # [0,1] float tensor
    transforms.Normalize(                        # ImageNet normalization
        mean=[0.485, 0.456, 0.406],
        std=[0.229, 0.224, 0.225]
    ),
])

dataset = datasets.CIFAR10(
    root="./data", train=False,
    download=True, transform=transform
)

# Use a subset (1000 images) for faster t-SNE
subset_indices = list(range(1000))
subset = torch.utils.data.Subset(dataset, subset_indices)
loader = DataLoader(subset, batch_size=64, shuffle=False)

# ── 4. Extract features ─────────────────────────────────────────────
print("Extracting features...")
with torch.no_grad():                           # no gradients needed
    for images, labels in loader:
        _ = model(images)                       # forward pass triggers hook
        labels_store.append(labels.numpy())

# Concatenate all batches into single arrays
all_features = np.concatenate(features_store, axis=0)   # (1000, 512)
all_labels   = np.concatenate(labels_store, axis=0)     # (1000,)
print(f"Feature matrix shape: {all_features.shape}")

# ── 5. Run t-SNE ────────────────────────────────────────────────────
print("Running t-SNE (this may take a minute)...")
tsne = TSNE(
    n_components=2,        # project to 2-D
    perplexity=30,         # balance local/global structure
    learning_rate=200,     # step size
    n_iter=1000,           # optimization iterations
    random_state=42        # reproducibility
)
features_2d = tsne.fit_transform(all_features)   # (1000, 2)

# ── 6. Plot the t-SNE embedding ─────────────────────────────────────
class_names = [
    "airplane", "automobile", "bird", "cat", "deer",
    "dog", "frog", "horse", "ship", "truck"
]

plt.figure(figsize=(12, 10))
for class_idx in range(10):
    mask = all_labels == class_idx                # boolean mask for class
    plt.scatter(
        features_2d[mask, 0],                     # x-coordinates
        features_2d[mask, 1],                     # y-coordinates
        label=class_names[class_idx],
        alpha=0.6,
        s=15
    )

plt.title("t-SNE of ResNet-18 Penultimate Layer Features (CIFAR-10)")
plt.xlabel("t-SNE Dimension 1")
plt.ylabel("t-SNE Dimension 2")
plt.legend(markerscale=3, fontsize=9)
plt.tight_layout()
plt.savefig("tsne_features.png", dpi=150)
plt.show()

# ── 7. Clean up the hook ────────────────────────────────────────────
hook_handle.remove()
print("Done! Plot saved to tsne_features.png")
```

### Multi-Layer Feature Extraction

```python
"""
Extract features from EVERY convolutional block to compare
how representations evolve layer by layer.
"""

import torch
import torchvision.models as models
import torchvision.transforms as transforms
from PIL import Image
import matplotlib.pyplot as plt

# ── Load model and image ────────────────────────────────────────────
model = models.vgg16(weights=models.VGG16_Weights.IMAGENET1K_V1)
model.eval()

transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406],
                         [0.229, 0.224, 0.225]),
])

img = Image.open("golden_retriever.jpg").convert("RGB")
x   = transform(img).unsqueeze(0)                # (1, 3, 224, 224)

# ── Register hooks on selected layers ───────────────────────────────
# VGG-16 features: layers 0,5,10,17,24 are the first conv of each block
target_layers = {
    "Conv1_1 (edges)":    model.features[0],     # 3  → 64
    "Conv2_1 (textures)": model.features[5],     # 64 → 128
    "Conv3_1 (patterns)": model.features[10],    # 128 → 256
    "Conv4_1 (parts)":    model.features[17],    # 256 → 512
    "Conv5_1 (objects)":  model.features[24],    # 512 → 512
}

activations = {}

def make_hook(name):
    """Factory function that returns a hook storing activations by name."""
    def hook(module, inp, out):
        activations[name] = out.detach()
    return hook

# Register all hooks
handles = []
for name, layer in target_layers.items():
    handles.append(layer.register_forward_hook(make_hook(name)))

# ── Forward pass (hooks capture activations) ─────────────────────────
with torch.no_grad():
    _ = model(x)

# ── Visualize the first 8 channels of each layer ────────────────────
fig, axes = plt.subplots(5, 8, figsize=(20, 12))
for row, (name, act) in enumerate(activations.items()):
    for col in range(8):
        ax = axes[row, col]
        # act shape: (1, C, H, W) → take channel `col`
        feature_map = act[0, col].cpu().numpy()
        ax.imshow(feature_map, cmap="viridis")
        ax.axis("off")
        if col == 0:
            ax.set_title(name, fontsize=8, loc="left")

plt.suptitle("Feature Maps at Each VGG-16 Block", fontsize=14)
plt.tight_layout()
plt.savefig("layer_features.png", dpi=150)
plt.show()

# Clean up hooks
for h in handles:
    h.remove()
```

---

## 11  PyTorch — FGSM Adversarial Attack

```python
"""
FGSM Adversarial Attack
========================
Demonstrates the Fast Gradient Sign Method on a pre-trained
ResNet-50. Shows how a tiny perturbation fools the network.
"""

import torch
import torch.nn as nn
import torchvision.models as models
import torchvision.transforms as transforms
from PIL import Image
import matplotlib.pyplot as plt
import json
import urllib

# ── 1. Load pre-trained ResNet-50 ────────────────────────────────────
model = models.resnet50(weights=models.ResNet50_Weights.IMAGENET1K_V1)
model.eval()

# ── 2. Load ImageNet class labels ────────────────────────────────────
# Download the human-readable ImageNet labels
url = ("https://raw.githubusercontent.com/anishathalye/"
       "imagenet-simple-labels/master/imagenet-simple-labels.json")
labels = json.loads(urllib.request.urlopen(url).read().decode())

# ── 3. Image preprocessing ──────────────────────────────────────────
transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),                       # [0, 1] range
])

# ImageNet normalization (applied separately so we can undo it later)
normalize = transforms.Normalize(
    mean=[0.485, 0.456, 0.406],
    std=[0.229, 0.224, 0.225]
)

# Load an image
img = Image.open("panda.jpg").convert("RGB")
x = transform(img).unsqueeze(0)                  # (1, 3, 224, 224)

# ── 4. Original prediction ──────────────────────────────────────────
x_norm = normalize(x.squeeze()).unsqueeze(0)
with torch.no_grad():
    logits = model(x_norm)
    probs  = torch.softmax(logits, dim=1)
    orig_class = probs.argmax().item()
    orig_conf  = probs[0, orig_class].item()

print(f"Original prediction: {labels[orig_class]} ({orig_conf:.1%})")

# ── 5. FGSM attack function ─────────────────────────────────────────
def fgsm_attack(model, images, labels_tensor, epsilon, normalize_fn):
    """
    Perform the FGSM attack.

    Parameters
    ----------
    model         : nn.Module   — the target classifier
    images        : Tensor      — original images in [0,1] range,
                                  shape (B, 3, H, W)
    labels_tensor : Tensor      — ground-truth class indices, shape (B,)
    epsilon       : float       — perturbation magnitude
    normalize_fn  : callable    — ImageNet normalization function

    Returns
    -------
    perturbed : Tensor — adversarial images in [0,1] range
    """

    # Enable gradient computation w.r.t. the input image
    images.requires_grad_(True)

    # Forward pass through the model
    x_normalized = normalize_fn(images.squeeze()).unsqueeze(0)
    logits = model(x_normalized)

    # Compute the cross-entropy loss
    loss = nn.CrossEntropyLoss()(logits, labels_tensor)

    # Backward pass to compute ∂L/∂x
    model.zero_grad()
    loss.backward()

    # Retrieve the gradient of the loss w.r.t. input pixels
    data_grad = images.grad.data                  # shape: (1,3,224,224)

    # Create the perturbation: ε · sign(∇_x L)
    perturbation = epsilon * data_grad.sign()

    # Add perturbation to original image and clamp to valid [0,1] range
    perturbed = (images + perturbation).clamp(0, 1)

    return perturbed.detach()

# ── 6. Run the FGSM attack with different epsilon values ─────────────
epsilons = [0, 0.005, 0.01, 0.02, 0.05, 0.1, 0.2]
true_label = torch.tensor([orig_class])            # use predicted class

print("\n  ε       | Prediction          | Confidence | Fooled?")
print("  ────────┼─────────────────────┼────────────┼────────")

adv_images = []
for eps in epsilons:
    if eps == 0:
        perturbed = x.clone()
    else:
        perturbed = fgsm_attack(model, x.clone(), true_label,
                                eps, normalize)

    # Classify the adversarial image
    x_adv_norm = normalize(perturbed.squeeze()).unsqueeze(0)
    with torch.no_grad():
        logits = model(x_adv_norm)
        probs  = torch.softmax(logits, dim=1)
        pred   = probs.argmax().item()
        conf   = probs[0, pred].item()

    fooled = "YES" if pred != orig_class else "no"
    print(f"  {eps:<8.3f}| {labels[pred]:<20s}| {conf:>9.1%}  | {fooled}")
    adv_images.append(perturbed)

# ── 7. Visualize original vs adversarial images ─────────────────────
fig, axes = plt.subplots(1, len(epsilons), figsize=(3*len(epsilons), 4))
for i, (eps, adv_img) in enumerate(zip(epsilons, adv_images)):
    ax = axes[i]
    # Convert tensor (1,3,H,W) → numpy (H,W,3) for display
    img_np = adv_img.squeeze().permute(1, 2, 0).numpy()
    ax.imshow(img_np)
    ax.set_title(f"ε = {eps}", fontsize=10)
    ax.axis("off")

    # Show the prediction below
    x_adv_norm = normalize(adv_img.squeeze()).unsqueeze(0)
    with torch.no_grad():
        pred = model(x_adv_norm).argmax().item()
    ax.set_xlabel(labels[pred], fontsize=8)

plt.suptitle("FGSM Adversarial Examples at Increasing ε", fontsize=14)
plt.tight_layout()
plt.savefig("fgsm_adversarial.png", dpi=150)
plt.show()

# ── 8. Visualize the perturbation itself ─────────────────────────────
# The perturbation is barely visible — let's amplify it for display.
eps_demo = 0.02
adv_demo = fgsm_attack(model, x.clone(), true_label, eps_demo, normalize)

perturbation = (adv_demo - x).squeeze()           # (3, 224, 224)
pert_display = perturbation.permute(1, 2, 0).numpy()

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# Original
axes[0].imshow(x.squeeze().permute(1, 2, 0).numpy())
axes[0].set_title(f"Original: {labels[orig_class]} ({orig_conf:.1%})")
axes[0].axis("off")

# Perturbation (amplified 10× for visibility)
axes[1].imshow(0.5 + 10 * pert_display)           # shift to [0,1] center
axes[1].set_title(f"Perturbation (10× amplified, ε={eps_demo})")
axes[1].axis("off")

# Adversarial
x_adv_norm = normalize(adv_demo.squeeze()).unsqueeze(0)
with torch.no_grad():
    adv_pred = model(x_adv_norm).argmax().item()
    adv_conf = torch.softmax(model(x_adv_norm), dim=1).max().item()
axes[2].imshow(adv_demo.squeeze().permute(1, 2, 0).numpy())
axes[2].set_title(f"Adversarial: {labels[adv_pred]} ({adv_conf:.1%})")
axes[2].axis("off")

plt.suptitle("FGSM Attack Decomposition", fontsize=14)
plt.tight_layout()
plt.savefig("fgsm_decomposition.png", dpi=150)
plt.show()
```

---

## 12  Why Interpretability Matters

Understanding what CNNs learn is not merely an academic curiosity — it
has real-world consequences across multiple domains.

```
 Four Pillars of CNN Interpretability
 ═════════════════════════════════════

 ┌──────────────────────────────────────────────────────────────────┐
 │                                                                  │
 │   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐    │
 │   │  TRUST   │   │ DEBUGGING│   │ FAIRNESS │   │REGULATION│    │
 │   │          │   │          │   │          │   │          │    │
 │   │ Doctors  │   │ Engineers│   │ Auditors │   │ Lawmakers│    │
 │   │ must     │   │ need to  │   │ must     │   │ require  │    │
 │   │ trust AI │   │ find and │   │ detect   │   │ explain- │    │
 │   │ diagnoses│   │ fix bugs │   │ bias in  │   │ able AI  │    │
 │   │ before   │   │ in model │   │ model    │   │ decisions│    │
 │   │ acting   │   │ behavior │   │ decisions│   │          │    │
 │   └──────────┘   └──────────┘   └──────────┘   └──────────┘    │
 │                                                                  │
 └──────────────────────────────────────────────────────────────────┘
```

### 12.1  Trust

Medical imaging AI (radiology, pathology) must provide evidence for its
predictions.  Saliency maps and Grad-CAM overlays show radiologists
**where** the model is looking, building confidence that the diagnosis
is based on relevant anatomy rather than confounding artifacts (e.g.,
a hospital label on the X-ray).

### 12.2  Debugging

Visualization reveals when models learn **spurious correlations** — e.g.,
classifying "cow" based on green grass backgrounds rather than the
animal itself.  Understanding internal representations helps engineers
curate better training data and architecture choices.

### 12.3  Fairness

Interpretability tools can detect when models discriminate based on
protected attributes (race, gender, age).  Network dissection can
reveal whether specific neurons encode demographic features that should
not influence the final prediction.

### 12.4  Regulation

The EU AI Act, GDPR "right to explanation," and similar regulations
increasingly require that automated decisions be explainable.  Tools
that reveal CNN internals are essential for compliance.

---

## 13  Summary Table

| Topic | Key Idea | Seminal Work |
|---|---|---|
| Hierarchical features | CNNs learn edges → textures → parts → objects through stacked layers | Zeiler & Fergus, 2014 |
| Feature transferability | Early layers are general; later layers are task-specific | Yosinski et al., 2014 |
| Texture bias | CNNs classify by texture, not shape; differs from humans | Geirhos et al., 2019 |
| Stylized-ImageNet | Training on style-transferred images shifts bias toward shape | Geirhos et al., 2019 |
| Adversarial examples | Tiny imperceptible perturbations fool CNNs with high confidence | Goodfellow et al., 2015 |
| FGSM | x_adv = x + ε·sign(∇_x L) — single-step attack | Goodfellow et al., 2015 |
| Linear hypothesis | Adversarial vulnerability arises from high-dimensional linearity | Goodfellow et al., 2015 |
| Adversarial training | Min-max: minimize loss under worst-case perturbation | Madry et al., 2018 |
| t-SNE / UMAP | Dimensionality reduction reveals semantic cluster structure in feature space | van der Maaten, 2008 / McInnes, 2018 |
| Ablation studies | Remove parts of the network to measure their contribution | — |
| Network dissection | IoU between neuron activations and semantic concept masks | Bau et al., 2017 |
| Interpretability | Needed for trust, debugging, fairness, and regulatory compliance | — |

---

## 14  Revision Questions

**Q1.  Hierarchical Features**
> Describe the types of features learned at layers 1 through 5 in a
> typical deep CNN.  Why is the non-linear activation function essential
> for this hierarchy to emerge?

**Q2.  Transfer Learning**
> Why do early convolutional layers transfer well across vastly different
> tasks (e.g., medical imaging ↔ autonomous driving), while later layers
> do not?  How does the Yosinski et al. (2014) experiment quantify this?

**Q3.  Texture vs Shape Bias**
> Explain the Geirhos et al. (2019) texture-shape conflict experiment.
> How does Stylized-ImageNet training shift the network toward
> shape-based reasoning, and why does this improve robustness?

**Q4.  FGSM Attack**
> Derive the FGSM perturbation formula x_adv = x + ε·sign(∇_x L).
> Using the linear hypothesis, explain why a perturbation with tiny
> per-pixel magnitude can cause a large change in the network output.

**Q5.  Network Dissection**
> What does the IoU metric in Network Dissection measure?  Why does the
> interpretability score tend to decrease in deeper layers?

**Q6.  Feature Space Geometry**
> Why do nearest neighbors in CNN feature space match semantically
> similar images, while nearest neighbors in pixel space often match
> images with similar colors but unrelated content?  How does t-SNE
> help reveal this cluster structure?

---

## 15  Navigation

| Previous | Up | Next |
|---|---|---|
| [← Activation Maximization](./05-activation-maximization.md) | [Unit 9 Index](../09-Visualizing-CNNs/) | [Depthwise Separable Convolutions →](../10-Advanced-Topics/01-depthwise-separable-conv.md) |

---

## 16  References

1. **Zeiler, M. D. & Fergus, R.** (2014). *Visualizing and Understanding
   Convolutional Networks.* ECCV 2014. arXiv:1311.1901.

2. **Yosinski, J., Clune, J., Bengio, Y. & Lipson, H.** (2014). *How
   Transferable Are Features in Deep Neural Networks?* NeurIPS 2014.
   arXiv:1411.1792.

3. **Geirhos, R., Rubisch, P., Michaelis, C., Bethge, M., Wichmann, F.
   A. & Brendel, W.** (2019). *ImageNet-trained CNNs Are Biased Towards
   Textures; Increasing Shape Bias Improves Accuracy and Robustness.*
   ICLR 2019. arXiv:1811.12231.

4. **Goodfellow, I. J., Shlens, J. & Szegedy, C.** (2015). *Explaining
   and Harnessing Adversarial Examples.* ICLR 2015. arXiv:1412.6572.

5. **Madry, A., Makelov, A., Schmidt, L., Tsipras, D. & Vladu, A.**
   (2018). *Towards Deep Learning Models Resistant to Adversarial
   Attacks.* ICLR 2018. arXiv:1706.06083.

6. **Bau, D., Zhou, B., Khosla, A., Oliva, A. & Torralba, A.** (2017).
   *Network Dissection: Quantifying Interpretability of Deep Visual
   Representations.* CVPR 2017. arXiv:1704.05796.

7. **van der Maaten, L. & Hinton, G.** (2008). *Visualizing Data Using
   t-SNE.* JMLR, 9, 2579–2605.

8. **McInnes, L., Healy, J. & Melville, J.** (2018). *UMAP: Uniform
   Manifold Approximation and Projection for Dimension Reduction.*
   arXiv:1802.03426.

9. **Olah, C., Mordvintsev, A. & Schubert, L.** (2017). *Feature
   Visualization.* Distill. https://distill.pub/2017/feature-visualization

10. **Szegedy, C., Zaremba, W., Sutskever, I., Bruna, J., Erhan, D.,
    Goodfellow, I. & Fergus, R.** (2014). *Intriguing Properties of
    Neural Networks.* ICLR 2014. arXiv:1312.6199.
