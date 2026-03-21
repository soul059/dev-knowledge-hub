# Grad-CAM — Gradient-weighted Class Activation Mapping

> **Unit 9 · Visualizing CNNs · Topic 3/6**

*Grad-CAM produces coarse localization heatmaps that highlight the
image regions a CNN "looks at" when predicting a particular class —
without requiring any architectural modification or retraining.*

---

## Table of Contents

1. [Overview](#1--overview)
2. [Class Activation Mapping (CAM) — The Predecessor](#2--class-activation-mapping-cam--the-predecessor)
3. [Grad-CAM Algorithm — Step by Step](#3--grad-cam-algorithm--step-by-step)
4. [Mathematical Formulation](#4--mathematical-formulation)
5. [Full Pipeline — ASCII Diagram](#5--full-pipeline--ascii-diagram)
6. [Worked Example with a Small Feature Map](#6--worked-example-with-a-small-feature-map)
7. [Grad-CAM++ Improvements](#7--grad-cam-improvements)
8. [Guided Grad-CAM](#8--guided-grad-cam)
9. [PyTorch Implementation](#9--pytorch-implementation)
10. [Applications](#10--applications)
11. [Summary Table](#11--summary-table)
12. [Revision Questions](#12--revision-questions)
13. [Navigation](#navigation)
14. [References](#references)

---

## 1 · Overview

### What is Grad-CAM?

Grad-CAM (Gradient-weighted Class Activation Mapping) is a visual
explanation technique introduced by Selvaraju et al. (2017). Given any
CNN-based model and a target class, Grad-CAM produces a **coarse
heatmap** over the input image that highlights which spatial regions
were most important for that prediction.

### Why does it matter?

| Concern | How Grad-CAM Helps |
|---|---|
| **Interpretability** | Shows *where* a model is looking, building trust |
| **Debugging** | Reveals if a model uses spurious correlations (e.g., watermark instead of object) |
| **Bias detection** | Exposes whether the model focuses on protected attributes |
| **Architecture-agnostic** | Works with any CNN — no GAP layer required, no retraining |
| **Class-discriminative** | Produces *different* heatmaps for different classes in the same image |

### Key Insight

The **gradients** of any target class score flowing back into the
**final convolutional layer** encode channel-wise importance. By
weighting each feature map channel by these gradients and summing,
we get a localization map that highlights the discriminative regions.

---

## 2 · Class Activation Mapping (CAM) — The Predecessor

### How CAM Works

CAM (Zhou et al., 2016) was the first technique to exploit the spatial
information retained in convolutional feature maps for class-specific
localization.

```
CAM Pipeline
============

  Input         Conv Layers        GAP           FC          Softmax
 ┌──────┐     ┌────────────┐    ┌──────┐     ┌──────┐     ┌──────┐
 │      │────▶│ Feature     │───▶│Global│────▶│ w_1  │────▶│ P(c) │
 │ Image│     │ Maps A^k    │    │ Avg  │     │ w_2  │     │      │
 │      │     │ (k channels)│    │ Pool │     │ ...  │     │      │
 └──────┘     └────────────┘    └──────┘     │ w_K  │     └──────┘
                   │                          └──────┘
                   │                             │
                   ▼                             │
              ┌─────────────────────────────┐    │
              │  CAM = Σ_k  w^c_k · A^k    │◀───┘
              │  (weighted sum of feat maps)│
              └─────────────────────────────┘
                   │
                   ▼
              ┌───────────┐
              │  Upsample │
              │  to input │
              │  size     │
              └───────────┘
                   │
                   ▼
              Heatmap overlaid on image
```

### CAM Formula

```
M_CAM^c(x, y) = Σ_k  w^c_k · A^k(x, y)

where:
  A^k(x, y) = activation of feature map k at spatial location (x, y)
  w^c_k     = weight connecting the GAP output of channel k
              to class c in the final FC layer
```

### Limitations of CAM

| Limitation | Detail |
|---|---|
| **Requires GAP** | The architecture must use Global Average Pooling before the final FC layer |
| **Requires retraining** | If a model was not built with GAP, you must modify it and retrain |
| **Single FC layer only** | Cannot be applied to models with multiple FC layers or complex heads |
| **Not generalizable** | Does not extend to tasks beyond classification (e.g., captioning, VQA) |

> **Grad-CAM removes all of these limitations** by replacing the
> learned FC weights with gradient-based importance weights.

---

## 3 · Grad-CAM Algorithm — Step by Step

Grad-CAM generalizes CAM by using **gradients** instead of FC weights.
It works with *any* differentiable CNN without architectural changes.

### Step-by-Step Procedure

```
Step 1 — Forward Pass
=====================
  Feed the input image through the network.
  Record the activations A^k of the LAST convolutional layer.
  (k = 1, 2, ..., K  feature map channels)
  Obtain the class score y^c (logit, before softmax).

Step 2 — Backward Pass (Compute Gradients)
==========================================
  Compute the gradient of the class score y^c with respect
  to each feature map A^k:

      ∂y^c / ∂A^k_{ij}

  This gives a gradient tensor of the same shape as A^k
  (height u × width v for each of K channels).

Step 3 — Global Average Pool the Gradients
===========================================
  For each channel k, average the gradient values across
  all spatial positions (i, j) to obtain a single scalar:

      α^c_k = (1/Z) Σ_i Σ_j  ∂y^c / ∂A^k_{ij}

  where Z = u × v (number of spatial positions).
  α^c_k captures the "importance" of channel k for class c.

Step 4 — Weighted Combination
=============================
  Compute the weighted sum of all K feature maps:

      L^c(i,j) = Σ_k  α^c_k · A^k(i,j)

Step 5 — Apply ReLU
====================
  Apply ReLU to keep only positive contributions:

      L^c_Grad-CAM = ReLU( L^c )

  Negative values correspond to pixels belonging to
  OTHER classes, which we discard.

Step 6 — Upsample to Input Resolution
======================================
  Bilinear interpolation from (u × v) → (H × W)
  to match the original input image dimensions.

Step 7 — Overlay Heatmap
=========================
  Normalize L^c_Grad-CAM to [0, 1], apply a colormap,
  and superimpose on the input image.
```

---

## 4 · Mathematical Formulation

### Core Equations

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Importance weight for channel k, class c:                       │
│                                                                  │
│              1                                                   │
│  α^c_k  =  ───  Σ_i Σ_j  ∂y^c / ∂A^k_{ij}                     │
│              Z                                                   │
│                                                                  │
│  where Z = u × v  (spatial dimensions of the feature map)        │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Grad-CAM heatmap for class c:                                   │
│                                                                  │
│  L^c_Grad-CAM  =  ReLU( Σ_k  α^c_k · A^k )                     │
│                                                                  │
│  Shape: (u × v) — same as a single feature map                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Why ReLU?

```
Without ReLU:
  Positive values  →  pixels that INCREASE the score for class c
  Negative values  →  pixels that DECREASE the score (belong to other classes)

With ReLU:
  We zero out the negatives because we only want the regions
  that have a POSITIVE influence on the class of interest.

  L^c_Grad-CAM = max(0, Σ_k α^c_k · A^k)
```

### Connection to CAM

```
Theorem (Selvaraju et al., 2017):
─────────────────────────────────
For a network with GAP followed by a single FC layer,
Grad-CAM is mathematically equivalent to CAM.

Proof sketch:
  If the architecture uses GAP, then:
      y^c = Σ_k  w^c_k · (1/Z) Σ_i Σ_j A^k_{ij}

  Taking the gradient:
      ∂y^c / ∂A^k_{ij} = w^c_k / Z

  Plugging into the Grad-CAM formula:
      α^c_k = (1/Z) Σ_i Σ_j (w^c_k / Z) = w^c_k

  So α^c_k reduces to the FC weight w^c_k — exactly CAM.
```

### Gradient Flow Through Multiple Layers

```
For deeper heads (multiple FC layers after conv):

  y^c = f( g( GAP(A^k) ) )

  The chain rule still yields valid gradients:

      ∂y^c / ∂A^k_{ij}   (computed by standard backprop)

  Grad-CAM works regardless of how many layers follow the
  conv layer — this is the key generalization over CAM.
```

---

## 5 · Full Pipeline — ASCII Diagram

```
                         GRAD-CAM  FULL  PIPELINE
 ═══════════════════════════════════════════════════════════════════

                    FORWARD PASS
                    ────────────
 ┌────────────┐    ┌──────────────────────────┐    ┌─────────────┐
 │            │    │   Convolutional Backbone  │    │  Classifier │
 │  Input     │───▶│                          │───▶│  Head       │
 │  Image     │    │  Conv1 → Conv2 → ... →   │    │  (FC, etc.) │
 │  (H × W)   │    │  Last Conv Layer         │    │             │
 │            │    │                          │    │  Output:    │
 └────────────┘    │  Feature Maps A^k        │    │  y^c (logit)│
                    │  shape: (K × u × v)      │    └──────┬──────┘
                    └────────────┬─────────────┘           │
                                 │                          │
                      ┌──────────┘                          │
                      │  Hook captures A^k                  │
                      ▼                                     │
               ┌──────────────┐                             │
               │  Stored      │                             │
               │  Feature     │                             │
               │  Maps A^k    │                             │
               │  (K × u × v) │                             │
               └──────┬───────┘                             │
                      │                                     │
                      │         BACKWARD PASS               │
                      │         ─────────────               │
                      │                                     │
                      │    ┌────────────────────────────┐   │
                      │    │  Compute gradients:        │   │
                      │    │  ∂y^c / ∂A^k_{ij}         │◀──┘
                      │    │                            │
                      │    │  Gradient tensor:          │
                      │    │  shape: (K × u × v)        │
                      │    └────────────┬───────────────┘
                      │                 │
                      │                 │  Hook captures gradients
                      │                 ▼
                      │    ┌────────────────────────────┐
                      │    │  Global Average Pool       │
                      │    │  across spatial dims (i,j) │
                      │    │                            │
                      │    │  α^c_k = mean over (u × v) │
                      │    │  shape: (K,)               │
                      │    └────────────┬───────────────┘
                      │                 │
                      ▼                 ▼
               ┌──────────────────────────────────────┐
               │                                      │
               │  Weighted combination:               │
               │  L^c = Σ_k  α^c_k · A^k             │
               │                                      │
               │  shape: (u × v)                      │
               │                                      │
               └─────────────────┬────────────────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │  ReLU                  │
                    │  L^c_Grad-CAM =        │
                    │    max(0, L^c)          │
                    └────────────┬───────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │  Upsample              │
                    │  (u × v) → (H × W)     │
                    │  bilinear interpolation │
                    └────────────┬───────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │  Normalize to [0, 1]   │
                    │  Apply colormap        │
                    │  (e.g., jet, turbo)    │
                    └────────────┬───────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │  Overlay on original   │
                    │  input image           │
                    │                        │
                    │  ┌──────────────────┐  │
                    │  │  🔴🟠🟡  Hot =   │  │
                    │  │     Important     │  │
                    │  │  🔵🟢  Cold =     │  │
                    │  │     Unimportant   │  │
                    │  └──────────────────┘  │
                    └────────────────────────┘
```

---

## 6 · Worked Example with a Small Feature Map

Let us trace through Grad-CAM manually with a tiny network that has
**K = 2** feature map channels, each of size **2 × 2**.

### Setup

```
Feature Map A^1 (channel 1):       Feature Map A^2 (channel 2):

    ┌─────┬─────┐                      ┌─────┬─────┐
    │ 1.0 │ 0.5 │                      │ 0.2 │ 0.8 │
    ├─────┼─────┤                      ├─────┼─────┤
    │ 0.3 │ 0.7 │                      │ 0.6 │ 0.1 │
    └─────┴─────┘                      └─────┴─────┘

  Suppose the network produces class scores:
    y^(cat)  = 3.2
    y^(dog)  = 1.5

  We want the Grad-CAM heatmap for class "cat" (c = cat).
```

### Step 2 — Compute Gradients of y^cat w.r.t. A^k

```
Assume backpropagation gives us:

  Gradients w.r.t. A^1:              Gradients w.r.t. A^2:
  ∂y^cat / ∂A^1_{ij}                 ∂y^cat / ∂A^2_{ij}

    ┌──────┬──────┐                     ┌──────┬──────┐
    │  0.4 │  0.2 │                     │ -0.1 │  0.3 │
    ├──────┼──────┤                     ├──────┼──────┤
    │  0.6 │  0.8 │                     │  0.1 │ -0.3 │
    └──────┴──────┘                     └──────┴──────┘
```

### Step 3 — Global Average Pool the Gradients

```
α^cat_1 = (1/4) × (0.4 + 0.2 + 0.6 + 0.8)
        = (1/4) × 2.0
        = 0.5

α^cat_2 = (1/4) × (-0.1 + 0.3 + 0.1 + (-0.3))
        = (1/4) × 0.0
        = 0.0

Interpretation:
  Channel 1 is important (α = 0.5) for predicting "cat"
  Channel 2 is irrelevant (α = 0.0) for predicting "cat"
```

### Step 4 — Weighted Combination

```
L^cat(i,j) = α^cat_1 · A^1(i,j)  +  α^cat_2 · A^2(i,j)
           = 0.5 · A^1(i,j)       +  0.0 · A^2(i,j)
           = 0.5 · A^1(i,j)

    ┌───────────────┬───────────────┐
    │ 0.5 × 1.0     │ 0.5 × 0.5     │
    │  = 0.50       │  = 0.25       │
    ├───────────────┼───────────────┤
    │ 0.5 × 0.3     │ 0.5 × 0.7     │
    │  = 0.15       │  = 0.35       │
    └───────────────┴───────────────┘

Result:
    ┌──────┬──────┐
    │ 0.50 │ 0.25 │
    ├──────┼──────┤
    │ 0.15 │ 0.35 │
    └──────┴──────┘
```

### Step 5 — Apply ReLU

```
ReLU(L^cat):    (all values already positive, so no change)

    ┌──────┬──────┐
    │ 0.50 │ 0.25 │
    ├──────┼──────┤
    │ 0.15 │ 0.35 │
    └──────┴──────┘

Interpretation:
  Top-left pixel (0.50) is the MOST important for "cat"
  Bottom-left pixel (0.15) is the LEAST important for "cat"
```

### Step 6 — Upsample & Normalize

```
Normalize to [0, 1] by dividing by max (0.50):

    ┌──────┬──────┐
    │ 1.00 │ 0.50 │
    ├──────┼──────┤
    │ 0.30 │ 0.70 │
    └──────┴──────┘

Then bilinearly upsample to the original image size (H × W)
and overlay with a colormap.

  ┌──────────────────────────┐
  │  ██  Hot (top-left)      │   ← model focuses here for "cat"
  │  ▓▓  Warm (bottom-right) │
  │  ░░  Cool (top-right)    │
  │  ·   Cold (bottom-left)  │
  └──────────────────────────┘
```

---

## 7 · Grad-CAM++ Improvements

Chattopadhyay et al. (2018) proposed **Grad-CAM++**, which improves
localization especially when **multiple instances** of a class appear
in the image.

### Key Difference

```
Grad-CAM:
  α^c_k = (1/Z) Σ_i Σ_j  ∂y^c / ∂A^k_{ij}
                ─────────────────────────────
                  uniform average over all
                  spatial positions

Grad-CAM++:
  α^c_k = Σ_i Σ_j  w^c_{kij} · ReLU( ∂y^c / ∂A^k_{ij} )
                     ─────────
                     pixel-wise
                     weight

  where w^c_{kij} depends on higher-order gradients
  (second and third derivatives of y^c w.r.t. A^k_{ij}).
```

### How Pixel-Wise Weights Are Computed

```
                   ∂²y^c / ∂(A^k_{ij})²
w^c_{kij} = ─────────────────────────────────────────────────
            2 · ∂²y^c / ∂(A^k_{ij})²  +
            Σ_a Σ_b [ A^k_{ab} · ∂³y^c / ∂(A^k_{ij})³ ]

Intuition:
  Pixels where the gradient has a larger SECOND derivative
  (i.e., the gradient is changing rapidly) get higher weights.
  This helps focus on each individual object instance rather
  than averaging over the entire feature map uniformly.
```

### Grad-CAM vs Grad-CAM++ Comparison

```
Scenario: Image contains TWO cats

  Grad-CAM:                     Grad-CAM++:
  ┌──────────────────┐          ┌──────────────────┐
  │      ░░░░        │          │      ████        │
  │    ░░████░░      │          │    ████████      │
  │    ░░████░░      │          │    ████████      │
  │      ░░░░        │          │      ████        │
  │                  │          │                  │
  │      ░░          │          │      ████        │
  │    ░░░░░░        │          │    ████████      │
  │      ░░          │          │      ████        │
  └──────────────────┘          └──────────────────┘

  Grad-CAM may highlight only       Grad-CAM++ gives better
  the dominant instance or a         coverage of BOTH instances
  blurred region between them.       with higher confidence.
```

### When to Use Grad-CAM++

| Scenario | Recommendation |
|---|---|
| Single object in image | Grad-CAM is sufficient |
| Multiple instances of same class | Grad-CAM++ is better |
| Need for finer spatial detail | Grad-CAM++ preferred |
| Computational constraints | Grad-CAM is cheaper (no higher-order derivatives) |

---

## 8 · Guided Grad-CAM

Guided Grad-CAM combines the **class-discriminative** coarse heatmap
of Grad-CAM with the **fine-grained pixel detail** of Guided
Backpropagation to produce high-resolution, class-specific
visualizations.

### Guided Backpropagation Recap

```
Standard Backprop:
  Gate gradients by sign of input to ReLU only.
  Pass gradient if A^k_{ij} > 0.

Guided Backprop:
  Gate gradients by BOTH:
    (a) sign of input (A^k_{ij} > 0)   AND
    (b) sign of the incoming gradient (∂L/∂output > 0)

  Effect: only positive gradients that correspond to
  positive activations are propagated, producing a
  cleaner, sharper saliency map.
```

### Guided Grad-CAM Formula

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  Guided Grad-CAM  =  GuidedBackprop  ⊙  Upsample(L^c)    │
│                                                            │
│  where ⊙ denotes element-wise multiplication               │
│                                                            │
│  GuidedBackprop:  shape (3 × H × W)  — fine-grained       │
│  L^c upsampled:   shape (1 × H × W)  — coarse, class-disc │
│  Result:          shape (3 × H × W)  — fine + class-disc   │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### Visual Comparison

```
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│               │  │               │  │               │
│  Guided       │  │   Grad-CAM    │  │  Guided       │
│  Backprop     │  │   (coarse)    │  │  Grad-CAM     │
│               │  │               │  │               │
│  High-res     │  │  ░░░░████░░░  │  │  Fine-grained │
│  edges &      │  │  ░░████████░  │  │  AND class-   │
│  textures     │  │  ░░████████░  │  │  specific     │
│  but NOT      │  │  ░░░░████░░░  │  │               │
│  class-       │  │               │  │  Best of      │
│  specific     │  │  Class-aware  │  │  both worlds  │
│               │  │  but coarse   │  │               │
└───────────────┘  └───────────────┘  └───────────────┘
     (a)                (b)          (a) ⊙ upsample(b)
```

### Pipeline Diagram

```
                    ┌──────────────┐
                    │  Input Image │
                    └──────┬───────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
     ┌────────────────┐       ┌──────────────────┐
     │ Guided         │       │ Grad-CAM         │
     │ Backpropagation│       │ (last conv layer)│
     │                │       │                  │
     │ Output:        │       │ Output:          │
     │ (3 × H × W)    │       │ (u × v)          │
     └───────┬────────┘       └────────┬─────────┘
             │                         │
             │                    ┌────┴─────┐
             │                    │ Upsample │
             │                    │ to H × W  │
             │                    └────┬─────┘
             │                         │
             └──────────┬──────────────┘
                        │
                   ┌────┴─────────┐
                   │  Element-    │
                   │  wise        │
                   │  Multiply ⊙  │
                   └────┬─────────┘
                        │
                        ▼
               ┌────────────────┐
               │ Guided         │
               │ Grad-CAM       │
               │ (3 × H × W)    │
               └────────────────┘
```

---

## 9 · PyTorch Implementation

### Complete Working Code

```python
"""
Grad-CAM implementation in PyTorch.
Works with any CNN that has convolutional layers.
"""

import torch
import torch.nn.functional as F
import numpy as np
import cv2
from torchvision import models, transforms
from PIL import Image
import matplotlib.pyplot as plt


# ──────────────────────────────────────────────────────────────
# 1. Grad-CAM Core Class
# ──────────────────────────────────────────────────────────────

class GradCAM:
    """
    Grad-CAM: Gradient-weighted Class Activation Mapping.

    Usage:
        model = models.resnet50(pretrained=True)
        target_layer = model.layer4[-1]       # last conv block
        grad_cam = GradCAM(model, target_layer)
        heatmap = grad_cam(input_tensor, class_idx=243)  # 243 = 'bull mastiff'
    """

    def __init__(self, model, target_layer):
        """
        Args:
            model: a CNN model (e.g., ResNet, VGG).
            target_layer: the convolutional layer to compute
                          Grad-CAM with respect to (usually the
                          last conv layer for best results).
        """
        self.model = model
        self.model.eval()  # set to evaluation mode

        self.target_layer = target_layer

        # Storage for hooked activations and gradients
        self.activations = None   # A^k
        self.gradients = None     # ∂y^c / ∂A^k

        # ── Register hooks on the target layer ──

        # Forward hook: captures the output (feature maps) of the layer
        self.target_layer.register_forward_hook(self._save_activation)

        # Backward hook: captures the gradients flowing into the layer
        self.target_layer.register_full_backward_hook(self._save_gradient)

    def _save_activation(self, module, input, output):
        """Forward hook — store feature map activations."""
        self.activations = output.detach()

    def _save_gradient(self, module, grad_input, grad_output):
        """Backward hook — store gradients of the feature maps."""
        # grad_output is a tuple; [0] is the gradient tensor
        self.gradients = grad_output[0].detach()

    def __call__(self, input_tensor, class_idx=None):
        """
        Generate a Grad-CAM heatmap.

        Args:
            input_tensor: preprocessed image tensor, shape (1, 3, H, W).
            class_idx: target class index. If None, uses the
                       predicted (argmax) class.

        Returns:
            heatmap: numpy array of shape (H_feat, W_feat),
                     values in [0, 1].
        """
        # ── Step 1: Forward pass ──
        output = self.model(input_tensor)       # shape: (1, num_classes)

        if class_idx is None:
            class_idx = output.argmax(dim=1).item()

        # ── Step 2: Backward pass ──
        # Zero all existing gradients
        self.model.zero_grad()

        # Select the score for the target class
        class_score = output[0, class_idx]       # scalar

        # Backpropagate to compute ∂y^c / ∂A^k
        class_score.backward(retain_graph=True)

        # ── Step 3: Compute importance weights α^c_k ──
        # self.gradients has shape (1, K, u, v)
        # Global average pool over spatial dimensions (u, v)
        weights = self.gradients.mean(dim=(2, 3))  # shape: (1, K)

        # ── Step 4: Weighted combination of feature maps ──
        # self.activations has shape (1, K, u, v)
        # Multiply each channel by its weight and sum over K
        cam = (weights[:, :, None, None] * self.activations).sum(dim=1)
        # cam shape: (1, u, v)

        # ── Step 5: Apply ReLU ──
        cam = F.relu(cam)

        # ── Step 6: Normalize to [0, 1] ──
        cam = cam.squeeze(0)                      # shape: (u, v)
        cam = cam - cam.min()
        if cam.max() != 0:
            cam = cam / cam.max()

        return cam.cpu().numpy()


# ──────────────────────────────────────────────────────────────
# 2. Utility Functions
# ──────────────────────────────────────────────────────────────

def preprocess_image(image_path, size=224):
    """
    Load and preprocess an image for a pretrained ImageNet model.

    Args:
        image_path: path to the image file.
        size: resize dimension (default 224 for ImageNet models).

    Returns:
        input_tensor: shape (1, 3, 224, 224), normalized.
        original_image: numpy array (H, W, 3) in RGB, for overlay.
    """
    # Standard ImageNet normalization
    transform = transforms.Compose([
        transforms.Resize((size, size)),
        transforms.ToTensor(),
        transforms.Normalize(
            mean=[0.485, 0.456, 0.406],
            std=[0.229, 0.224, 0.225]
        ),
    ])

    image = Image.open(image_path).convert("RGB")
    original_image = np.array(image.resize((size, size)))

    input_tensor = transform(image).unsqueeze(0)  # add batch dim
    return input_tensor, original_image


def apply_heatmap(heatmap, original_image, alpha=0.5, colormap=cv2.COLORMAP_JET):
    """
    Overlay a Grad-CAM heatmap on the original image.

    Args:
        heatmap: numpy array (u, v), values in [0, 1].
        original_image: numpy array (H, W, 3), uint8 RGB.
        alpha: blending factor for the overlay.
        colormap: OpenCV colormap constant.

    Returns:
        overlay: numpy array (H, W, 3), uint8 RGB.
    """
    h, w = original_image.shape[:2]

    # Upsample heatmap to match the original image size
    heatmap_resized = cv2.resize(heatmap, (w, h))

    # Convert to uint8 and apply colormap
    heatmap_uint8 = np.uint8(255 * heatmap_resized)
    heatmap_colored = cv2.applyColorMap(heatmap_uint8, colormap)
    heatmap_colored = cv2.cvtColor(heatmap_colored, cv2.COLOR_BGR2RGB)

    # Blend: overlay = alpha * heatmap + (1 - alpha) * image
    overlay = np.uint8(alpha * heatmap_colored + (1 - alpha) * original_image)

    return overlay


# ──────────────────────────────────────────────────────────────
# 3. Main — Putting It All Together
# ──────────────────────────────────────────────────────────────

def main():
    """Run Grad-CAM on a sample image using ResNet-50."""

    # ── Load pretrained model ──
    model = models.resnet50(weights=models.ResNet50_Weights.DEFAULT)
    model.eval()

    # ── Choose the target convolutional layer ──
    # For ResNet-50, the last conv layer is inside layer4
    target_layer = model.layer4[-1].conv3  # last 1x1 conv in the bottleneck

    # ── Instantiate Grad-CAM ──
    grad_cam = GradCAM(model, target_layer)

    # ── Load and preprocess image ──
    image_path = "sample.jpg"  # replace with your image path
    input_tensor, original_image = preprocess_image(image_path)

    # ── Generate heatmap ──
    # class_idx=None → use predicted class
    heatmap = grad_cam(input_tensor, class_idx=None)

    # ── Create overlay ──
    overlay = apply_heatmap(heatmap, original_image, alpha=0.5)

    # ── Visualize ──
    fig, axes = plt.subplots(1, 3, figsize=(15, 5))

    axes[0].imshow(original_image)
    axes[0].set_title("Original Image")
    axes[0].axis("off")

    axes[1].imshow(heatmap, cmap="jet")
    axes[1].set_title("Grad-CAM Heatmap")
    axes[1].axis("off")

    axes[2].imshow(overlay)
    axes[2].set_title("Grad-CAM Overlay")
    axes[2].axis("off")

    plt.tight_layout()
    plt.savefig("grad_cam_result.png", dpi=150, bbox_inches="tight")
    plt.show()

    print(f"Predicted class index: {grad_cam.model(input_tensor).argmax().item()}")


if __name__ == "__main__":
    main()
```

### Hook Mechanism — How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                     PyTorch Hooks                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Forward Hook: register_forward_hook(fn)                    │
│  ─────────────────────────────────────                      │
│  Called AFTER forward() of the layer completes.             │
│  fn(module, input, output) — we store `output` (= A^k).    │
│                                                             │
│  Backward Hook: register_full_backward_hook(fn)             │
│  ──────────────────────────────────────────────              │
│  Called AFTER backward() computes gradients for the layer.  │
│  fn(module, grad_input, grad_output) — we store             │
│  `grad_output[0]` (= ∂y^c / ∂A^k).                        │
│                                                             │
│  Why hooks?                                                 │
│    → We need intermediate activations and their gradients.  │
│    → PyTorch normally only keeps gradients for leaf tensors │
│      (parameters). Hooks let us intercept non-leaf values.  │
│                                                             │
│  IMPORTANT: Always call .detach() on hooked tensors to      │
│  avoid memory leaks from retaining the computation graph.   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Choosing the Target Layer

```
Model               Recommended Target Layer
──────────────────   ─────────────────────────────────
VGG-16/19            model.features[29]        (last conv in features)
ResNet-50/101/152    model.layer4[-1].conv3    (last bottleneck conv)
                     model.layer4[-1]          (last bottleneck block)
DenseNet-121/161     model.features.denseblock4 (last dense block)
Inception-v3         model.Mixed_7c            (last inception block)
EfficientNet-B0      model.features[-1]        (last MBConv block)
MobileNet-v2         model.features[-1]        (last inverted residual)

General rule: use the LAST convolutional layer — it has the
richest semantic information while still retaining some spatial
resolution.
```

### Generating Heatmaps for Multiple Classes

```python
def grad_cam_multi_class(model, target_layer, input_tensor, class_indices):
    """
    Generate Grad-CAM heatmaps for multiple classes.

    Useful for comparing what the model looks at when predicting
    different classes in the same image.
    """
    grad_cam = GradCAM(model, target_layer)
    heatmaps = {}

    for cls_idx in class_indices:
        heatmap = grad_cam(input_tensor, class_idx=cls_idx)
        heatmaps[cls_idx] = heatmap

    return heatmaps


# Example: compare "cat" (281) vs "dog" (243) heatmaps
# heatmaps = grad_cam_multi_class(model, target_layer, img, [281, 243])
```

---

## 10 · Applications

### 10.1 Model Debugging

```
Problem:  A model classifies "horse" images with 95% accuracy,
          but the Grad-CAM heatmap highlights the bottom-left
          corner — a copyright watermark — not the horse.

Diagnosis: The model learned a SPURIOUS CORRELATION between
           the watermark and the "horse" label.

Fix:      Remove watermarks, augment with different watermarks,
          or mask them during training.

  ┌────────────────────────────┐
  │                 🐴          │
  │              🐴🐴🐴         │
  │             🐴🐴🐴🐴        │
  │                            │
  │                            │
  │  ████████                  │   ← Grad-CAM highlights HERE
  │  █ PHOTO █                  │     (the watermark, not the horse!)
  │  ████████                  │
  └────────────────────────────┘
```

### 10.2 Bias Detection

```
Scenario: A gender classifier is evaluated with Grad-CAM.

Finding:  For "female" predictions, the model focuses on
          long hair and makeup rather than facial features.
          For "male" predictions, it focuses on facial hair.

Risk:     The model encodes and amplifies societal stereotypes.
          Women with short hair or men without beards may be
          systematically misclassified.

Action:   Retrain with more diverse data; apply fairness
          constraints; use Grad-CAM as a monitoring tool.
```

### 10.3 Medical Imaging

```
Application: Chest X-ray classification (e.g., pneumonia detection)

  ┌─────────────────────────────┐
  │                             │
  │    ┌───┐       ┌───┐       │
  │    │   │       │   │       │
  │    │ L │       │ R │       │
  │    │   │  ████ │   │       │
  │    │   │ █████ │   │       │
  │    └───┘ █████ └───┘       │
  │          ████               │
  │                             │
  └─────────────────────────────┘
          ^^^^
          Grad-CAM highlights the
          region of consolidation
          (pneumonia indicator)

Benefits for clinicians:
  ✓ Confirms model is looking at the right anatomical region
  ✓ Builds trust in AI-assisted diagnosis
  ✓ Helps identify false positives caused by artifacts
  ✓ Required by FDA for explainability in clinical AI tools
```

### 10.4 Additional Use Cases

| Domain | Application |
|---|---|
| **Autonomous driving** | Verify the model focuses on traffic signs, pedestrians, lane markings |
| **Content moderation** | Understand which image regions trigger NSFW detection |
| **Wildlife monitoring** | Confirm species classification focuses on distinguishing features (e.g., beak shape) |
| **Quality control** | Highlight defect regions on manufacturing images |
| **Education** | Help students understand what a CNN has learned |

---

## 11 · Summary Table

| Aspect | Detail |
|---|---|
| **Full name** | Gradient-weighted Class Activation Mapping |
| **Paper** | Selvaraju et al., ICCV 2017 |
| **Input** | Any CNN + target class index + input image |
| **Output** | Coarse heatmap (size of last conv feature map) |
| **Key formula** | L^c = ReLU(Σ_k α^c_k · A^k), α^c_k = mean(∂y^c/∂A^k) |
| **Architecture requirement** | None — works with any differentiable CNN |
| **Retraining required?** | No |
| **Resolution** | Coarse (limited by conv feature map size, e.g., 7×7 for ResNet) |
| **Class-discriminative?** | Yes — different heatmaps for different classes |
| **Predecessor** | CAM (requires GAP + single FC layer) |
| **Extension: Grad-CAM++** | Pixel-wise gradient weighting; better for multiple instances |
| **Extension: Guided Grad-CAM** | Element-wise product with guided backprop; high-res + class-specific |
| **Computational cost** | One forward + one backward pass (same as training a single step) |
| **Limitations** | Coarse resolution; may miss small objects; assumes last conv is best |

---

## 12 · Revision Questions

**Q1.** What is the key advantage of Grad-CAM over the original CAM
technique? Explain why Grad-CAM does not require Global Average
Pooling in the architecture.

> *Hint: Consider how the importance weights α^c_k are computed in
> each method — CAM uses FC weights (requiring GAP), while Grad-CAM
> uses gradient-based weights that can be computed for any
> differentiable layer.*

**Q2.** Write out the Grad-CAM formula. Why is ReLU applied to the
weighted sum of feature maps, and what would happen if we omitted it?

> *Hint: Negative values in the weighted sum correspond to features
> that reduce the target class score. Keeping them would highlight
> regions important for OTHER classes.*

**Q3.** In the worked example, channel 2 received an importance weight
of α = 0.0. Does this mean channel 2 is never useful for any class,
or only that it is not useful for the specific class "cat"? Explain.

> *Hint: α^c_k is class-specific. The same channel might receive a
> high weight for class "dog" but zero for class "cat".*

**Q4.** Describe how Guided Grad-CAM combines two visualization
techniques. What does each component contribute?

> *Hint: Guided Backpropagation provides fine-grained pixel-level
> detail, while Grad-CAM provides coarse class-discriminative
> localization. Their element-wise product yields both properties.*

**Q5.** A medical AI system correctly classifies a chest X-ray as
"pneumonia" but the Grad-CAM heatmap highlights the text annotation
in the corner of the image instead of the lung region. What does this
reveal about the model, and how would you address it?

> *Hint: The model has learned a spurious correlation. The fix
> involves data cleaning and augmentation, not model architecture
> changes.*

**Q6.** Explain the improvement that Grad-CAM++ brings over standard
Grad-CAM. In what specific scenario does Grad-CAM++ perform
noticeably better?

> *Hint: Grad-CAM++ uses pixel-wise gradient weighting (involving
> second- and third-order derivatives) instead of uniform spatial
> averaging. It excels when multiple instances of the target class
> appear in the same image.*

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [← Feature Map Visualization](./02-feature-map-visualization.md) | [Unit 9 Index](../09-Visualizing-CNNs/) | [Saliency Maps →](./04-saliency-maps.md) |

---

## References

1. **Selvaraju, R. R., Cogswell, M., Das, A., Vedantam, R., Parikh, D., & Batra, D.** (2017).
   *Grad-CAM: Visual Explanations from Deep Networks via Gradient-based Localization.*
   IEEE International Conference on Computer Vision (ICCV), 618–626.
   [arXiv:1610.02391](https://arxiv.org/abs/1610.02391)

2. **Zhou, B., Khosla, A., Lapedriza, A., Oliva, A., & Torralba, A.** (2016).
   *Learning Deep Features for Discriminative Localization.*
   IEEE Conference on Computer Vision and Pattern Recognition (CVPR), 2921–2929.
   [arXiv:1512.04150](https://arxiv.org/abs/1512.04150)

3. **Chattopadhyay, A., Sarkar, A., Howlader, P., & Balasubramanian, V. N.** (2018).
   *Grad-CAM++: Generalized Gradient-based Visual Explanations for Deep Convolutional Networks.*
   IEEE Winter Conference on Applications of Computer Vision (WACV), 839–847.
   [arXiv:1710.11063](https://arxiv.org/abs/1710.11063)

4. **Springenberg, J. T., Dosovitskiy, A., Brox, T., & Riedmiller, M.** (2015).
   *Striving for Simplicity: The All Convolutional Net.*
   ICLR Workshop. [arXiv:1412.6806](https://arxiv.org/abs/1412.6806)

5. **PyTorch Documentation — Hooks.**
   [https://pytorch.org/docs/stable/generated/torch.nn.Module.html#torch.nn.Module.register_forward_hook](https://pytorch.org/docs/stable/generated/torch.nn.Module.html)

---

*Last updated: 2025-07-15*
