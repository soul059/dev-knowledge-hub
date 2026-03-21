# 🏆 AlexNet: The Deep Learning Revolution

[← Previous: LeNet-5](01-lenet-5.md) | [Back to Unit Overview](../README.md) | [Next: VGGNet →](03-vggnet.md)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Historical Context: The ImageNet Moment](#historical-context-the-imagenet-moment)
- [Architecture Deep Dive](#architecture-deep-dive)
- [ASCII Architecture Diagram](#ascii-architecture-diagram)
- [Layer-by-Layer Analysis](#layer-by-layer-analysis)
- [Key Innovations](#key-innovations)
- [Parameter Calculations](#parameter-calculations)
- [Mathematical Foundations](#mathematical-foundations)
- [Worked Example](#worked-example)
- [PyTorch Implementation](#pytorch-implementation)
- [Applications](#applications)
- [Summary Table](#summary-table)
- [Revision Questions](#revision-questions)

---

## Overview

**AlexNet** is a deep convolutional neural network designed by **Alex Krizhevsky, Ilya Sutskever, and Geoffrey Hinton** that won the **ImageNet Large Scale Visual Recognition Challenge (ILSVRC) 2012** by a dramatic margin. It is widely credited with **igniting the modern deep learning revolution**.

### Key Facts

| Property | Value |
|----------|-------|
| **Year** | 2012 |
| **Authors** | Krizhevsky, Sutskever, Hinton |
| **Paper** | ImageNet Classification with Deep Convolutional Neural Networks |
| **Competition** | ILSVRC-2012 (ImageNet) |
| **Task** | 1000-class image classification |
| **Input Size** | 227 × 227 × 3 (RGB) |
| **Parameters** | ~60 million |
| **Depth** | 8 learnable layers (5 Conv + 3 FC) |
| **Top-5 Error** | 15.3% (vs. 26.2% runner-up) |
| **Key Innovations** | ReLU, Dropout, Data Augmentation, Multi-GPU |

---

## Historical Context: The ImageNet Moment

### Before AlexNet (2010-2011)

The ImageNet challenge (ILSVRC) required classifying images into **1,000 categories** from a dataset of **1.2 million** training images. Before 2012, the best systems used:

- Hand-crafted features (SIFT, HOG, GIST)
- Traditional ML classifiers (SVMs, Random Forests)
- Ensemble methods with Fisher Vectors

These systems plateaued around **25-26% top-5 error rate**.

```
ILSVRC Top-5 Error Rate Timeline:
══════════════════════════════════

  30% │ ██████████████████████████  (2010: 28.2% - NEC)
      │ ████████████████████████    (2011: 25.8% - Xerox)
      │ ███████████████             (2012: 15.3% - AlexNet) ← HUGE gap!
  20% │
      │
  15% │ ██████████████              (2012: 15.3%)
      │
      │
      └────────────────────────────────────────

  The 10.9% improvement was unprecedented — most years
  saw incremental 1-2% improvements.
```

### The Impact

AlexNet didn't just win — it **shattered** the competition:

1. **15.3% top-5 error** vs. 26.2% for the runner-up (10.9% absolute improvement)
2. Proved that **deep neural networks + GPUs** could dramatically outperform hand-crafted features
3. Triggered a **paradigm shift**: within 2 years, virtually all competitive entries used deep learning
4. Started the modern era of AI investment, research, and applications

---

## Architecture Deep Dive

AlexNet scaled up the LeNet-5 paradigm dramatically:

| Comparison | LeNet-5 | AlexNet | Factor |
|-----------|---------|---------|--------|
| Input size | 32×32×1 | 227×227×3 | ~150× more pixels |
| Depth | 7 layers | 8 layers | Deeper |
| Parameters | ~60K | ~60M | **1000× more** |
| Filter count | 6, 16 | 96, 256, 384, 384, 256 | Much wider |
| Training data | 60K images | 1.2M images | 20× more |
| Training time | Minutes (CPU) | 5-6 days (2 GPUs) | GPU-era |

### Design Principles

1. **Deeper and wider** than previous networks
2. **ReLU activation** for faster training (vs. tanh/sigmoid)
3. **Dropout regularization** to prevent overfitting on 60M parameters
4. **Data augmentation** to artificially expand the training set
5. **Multi-GPU training** to handle the computational requirements
6. **Local Response Normalization (LRN)** for lateral inhibition (later deprecated)

---

## ASCII Architecture Diagram

```
                               AlexNet Architecture
                               ====================

  INPUT      CONV1      POOL1     CONV2      POOL2     CONV3     CONV4     CONV5
 227×227×3  55×55×96   27×27×96  27×27×256  13×13×256 13×13×384 13×13×384 13×13×256

┌────────┐ ┌────────┐ ┌───────┐ ┌────────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
│        │ │ 96     │ │       │ │ 256    │ │       │ │ 384   │ │ 384   │ │ 256   │
│227×227 │►│ maps   │►│ max   │►│ maps   │►│ max   │►│ maps  │►│ maps  │►│ maps  │
│  RGB   │ │ 55×55  │ │ pool  │ │ 27×27  │ │ pool  │ │13×13  │ │13×13  │ │13×13  │
│        │ │        │ │27×27  │ │        │ │13×13  │ │       │ │       │ │       │
└────────┘ └────────┘ └───────┘ └────────┘ └───────┘ └───────┘ └───────┘ └───────┘
            11×11       3×3       5×5        3×3       3×3       3×3       3×3
            stride=4   stride=2  stride=1   stride=2  stride=1  stride=1  stride=1
            +ReLU                pad=2                 pad=1     pad=1     pad=1
                                 +ReLU                 +ReLU     +ReLU     +ReLU
                                                                             │
                                                                        POOL5 (max 3×3)
                                                                             │
                                                                        6×6×256
                                                                             │
                                 ┌─────────────────────────────────────────────┘
                                 ▼
                           ┌──────────┐   ┌──────────┐   ┌──────────┐
                           │  FC6     │──►│  FC7     │──►│  FC8     │
                           │  4096    │   │  4096    │   │  1000    │
                           │ +ReLU   │   │ +ReLU   │   │ +Softmax │
                           │ +Dropout│   │ +Dropout│   │          │
                           └──────────┘   └──────────┘   └──────────┘


Detailed Data Flow:
───────────────────

  [227×227×3]
       │
  Conv1: 96 filters, 11×11, stride 4, pad 0
       │
  [55×55×96] ── ReLU ── LRN
       │
  MaxPool: 3×3, stride 2
       │
  [27×27×96]
       │
  Conv2: 256 filters, 5×5, stride 1, pad 2
       │
  [27×27×256] ── ReLU ── LRN
       │
  MaxPool: 3×3, stride 2
       │
  [13×13×256]
       │
  Conv3: 384 filters, 3×3, stride 1, pad 1
       │
  [13×13×384] ── ReLU
       │
  Conv4: 384 filters, 3×3, stride 1, pad 1
       │
  [13×13×384] ── ReLU
       │
  Conv5: 256 filters, 3×3, stride 1, pad 1
       │
  [13×13×256] ── ReLU
       │
  MaxPool: 3×3, stride 2
       │
  [6×6×256] = 9,216
       │
  Flatten → FC6: 4096 ── ReLU ── Dropout(0.5)
       │
  FC7: 4096 ── ReLU ── Dropout(0.5)
       │
  FC8: 1000 ── Softmax
       │
  Output: 1000 class probabilities


Original Dual-GPU Architecture:
───────────────────────────────

  GPU 1 (48 filters)                     GPU 2 (48 filters)
  ┌──────────────────┐                   ┌──────────────────┐
  │ Conv1: 48 maps   │                   │ Conv1: 48 maps   │
  │ Conv2: 128 maps  │                   │ Conv2: 128 maps  │
  │ Conv3: 192 maps  │←── cross-GPU ───► │ Conv3: 192 maps  │
  │ Conv4: 192 maps  │    connections    │ Conv4: 192 maps  │
  │ Conv5: 128 maps  │                   │ Conv5: 128 maps  │
  └────────┬─────────┘                   └────────┬─────────┘
           │                                       │
           └──────────────┬────────────────────────┘
                          ▼
                    FC6: 4096
                    FC7: 4096
                    FC8: 1000

  Conv3 receives input from BOTH GPUs (cross-GPU communication)
  Conv4 receives input only from the SAME GPU
```

---

## Layer-by-Layer Analysis

### Layer 1: Conv1 (11×11, stride 4, 96 filters)

| Property | Value |
|----------|-------|
| Input | 227 × 227 × 3 |
| Filter size | 11 × 11 |
| Filters | 96 |
| Stride | 4 |
| Padding | 0 |
| Output | 55 × 55 × 96 |
| Activation | ReLU |
| Normalization | Local Response Normalization |

```
Output_H = (227 - 11) / 4 + 1 = 216/4 + 1 = 55

Parameters:
  Weights: 11 × 11 × 3 × 96 = 34,848
  Biases:  96
  Total:   34,944
```

**Why 11×11 with stride 4?** The input images are much larger (227×227 vs. LeNet's 32×32), so a large initial receptive field quickly reduces spatial dimensions. Stride 4 aggressively downsamples from 227→55, reducing computation in subsequent layers.

### Pool1: Max Pooling (3×3, stride 2)

```
Output_H = (55 - 3) / 2 + 1 = 27
Output: 27 × 27 × 96

Note: Overlapping pooling (kernel=3, stride=2) was found to
reduce overfitting compared to non-overlapping pooling (kernel=2, stride=2).
```

### Layer 2: Conv2 (5×5, 256 filters)

| Property | Value |
|----------|-------|
| Input | 27 × 27 × 96 |
| Filter size | 5 × 5 |
| Filters | 256 |
| Stride | 1 |
| Padding | 2 |
| Output | 27 × 27 × 256 |

```
Output_H = (27 - 5 + 2×2) / 1 + 1 = 27

Parameters:
  Weights: 5 × 5 × 96 × 256 = 614,400
  Biases:  256
  Total:   614,656
```

### Pool2: Max Pooling (3×3, stride 2)

```
Output: 13 × 13 × 256
```

### Layer 3: Conv3 (3×3, 384 filters)

| Property | Value |
|----------|-------|
| Input | 13 × 13 × 256 |
| Filter size | 3 × 3 |
| Filters | 384 |
| Stride | 1 |
| Padding | 1 |
| Output | 13 × 13 × 384 |

```
Parameters:
  Weights: 3 × 3 × 256 × 384 = 884,736
  Biases:  384
  Total:   885,120
```

### Layer 4: Conv4 (3×3, 384 filters)

| Property | Value |
|----------|-------|
| Input | 13 × 13 × 384 |
| Filter size | 3 × 3 |
| Filters | 384 |
| Stride | 1 |
| Padding | 1 |
| Output | 13 × 13 × 384 |

```
Parameters:
  Weights: 3 × 3 × 384 × 384 = 1,327,104
  Biases:  384
  Total:   1,327,488
```

### Layer 5: Conv5 (3×3, 256 filters)

| Property | Value |
|----------|-------|
| Input | 13 × 13 × 384 |
| Filter size | 3 × 3 |
| Filters | 256 |
| Stride | 1 |
| Padding | 1 |
| Output | 13 × 13 × 256 |

```
Parameters:
  Weights: 3 × 3 × 384 × 256 = 884,736
  Biases:  256
  Total:   884,992
```

### Pool5: Max Pooling (3×3, stride 2)

```
Output_H = (13 - 3) / 2 + 1 = 6
Output: 6 × 6 × 256 = 9,216
```

### Layer 6: FC6 (9216 → 4096)

```
Parameters:
  Weights: 9,216 × 4,096 = 37,748,736
  Biases:  4,096
  Total:   37,752,832

Activation: ReLU
Regularization: Dropout(p=0.5)
```

### Layer 7: FC7 (4096 → 4096)

```
Parameters:
  Weights: 4,096 × 4,096 = 16,777,216
  Biases:  4,096
  Total:   16,781,312

Activation: ReLU
Regularization: Dropout(p=0.5)
```

### Layer 8: FC8 (4096 → 1000)

```
Parameters:
  Weights: 4,096 × 1,000 = 4,096,000
  Biases:  1,000
  Total:   4,097,000

Activation: Softmax
```

---

## Key Innovations

### 1. ReLU Activation Function

AlexNet was one of the first major networks to use **Rectified Linear Units (ReLU)** instead of tanh or sigmoid.

```
              Sigmoid / Tanh                   ReLU
           ┌─────────────────┐          ┌─────────────────┐
     1.0 ──│──────────╱──────│──        │            ╱    │
           │        ╱        │          │          ╱      │
     0.5 ──│──────╱──────────│──        │        ╱        │
           │    ╱            │          │      ╱          │
     0.0 ──│──╱──────────────│──   0.0 ─│────╱────────────│──
           │╱                │          │╱                │
    -0.5 ──│─────────────────│──        │                 │
           └─────────────────┘          └─────────────────┘

  f(x) = 1/(1+e^(-x))             f(x) = max(0, x)
  Saturates for large |x|          Never saturates for x > 0
  Gradient vanishes                Constant gradient = 1
  Slow convergence                 6× faster convergence
```

**Why ReLU is faster:**
- **No saturation for positive values**: Gradient is always 1 for x > 0
- **Sparse activation**: Many neurons output 0, creating sparse representations
- **Computationally cheap**: Just a threshold comparison vs. exponentials
- **6× faster convergence** reported by Krizhevsky et al. on CIFAR-10

```
ReLU:
  f(x) = max(0, x)

  f'(x) = { 1  if x > 0
           { 0  if x < 0
           { undefined at x = 0 (use sub-gradient = 0 or 1)
```

### 2. Dropout Regularization

With 60M parameters and only 1.2M training images, overfitting was a major concern. AlexNet introduced **dropout** to neural network training.

```
Without Dropout:                    With Dropout (p=0.5):
─────────────────                   ─────────────────────
  ○ ─── ○ ─── ○                      ○ ─── ○ ─── ○
  │╲  ╱│╲  ╱│                      │     │╲    │
  │ ╲╱ │ ╲╱ │                      │     │ ╲   │
  │ ╱╲ │ ╱╲ │                      │     │  ╲  │
  │╱  ╲│╱  ╲│                      │     │   ╲ │
  ○ ─── ○ ─── ○                      ○     ✗     ○     ← randomly dropped
  │╲  ╱│╲  ╱│                      │           │
  │ ╲╱ │ ╲╱ │                      │           │
  ○ ─── ○ ─── ○                      ○     ✗     ○     ← randomly dropped

All connections active                50% of neurons zeroed out
Co-adaptation possible                Forces redundant representations
```

**How it works:**
```
Training:
  For each training sample, independently set each neuron's output to 0
  with probability p = 0.5

  h_train = h * mask,  where mask ~ Bernoulli(1 - p)

Testing / Inference:
  Use all neurons but scale outputs by (1 - p):

  h_test = h * (1 - p)

  OR use "inverted dropout" during training (PyTorch default):
  h_train = (h * mask) / (1 - p)   ← scale up during training
  h_test  = h                       ← no change at test time
```

### 3. Data Augmentation

AlexNet used two forms of data augmentation to reduce overfitting:

**a) Random crops and horizontal flips:**
```
Original: 256×256 image
                  ┌─────────────────────────┐
                  │                         │
                  │    ┌─────────────────┐  │
                  │    │   Random crop   │  │
                  │    │   227×227       │  │
                  │    │                 │  │
                  │    └─────────────────┘  │
                  │                         │
                  └─────────────────────────┘

Training:  Extract random 227×227 crops (+ horizontal flips)
           → 2048 possible crops × 2 (flip) = 4096 variations per image

Testing:   Extract 5 crops (4 corners + center) × 2 (flip) = 10 crops
           Average predictions over all 10 crops
```

**b) PCA color augmentation (Fancy PCA):**
```
For each training image:
  1. Compute PCA on the RGB pixel values across the training set
  2. Add multiples of the principal components with random magnitudes:

     I'_xy = [I^R_xy, I^G_xy, I^B_xy]^T + [p1, p2, p3] [α1λ1, α2λ2, α3λ3]^T

  Where:
    p_i = eigenvectors of the 3×3 RGB covariance matrix
    λ_i = eigenvalues
    α_i ~ N(0, 0.1)  (random, drawn once per image per epoch)

  This captures natural variations in illumination and color.
  Reduced top-1 error by over 1%.
```

### 4. Local Response Normalization (LRN)

Inspired by **lateral inhibition** in biological neurons:

```
                         a^i_xy
b^i_xy = ──────────────────────────────────────
          (k + α · ∑_{j=max(0,i-n/2)}^{min(N-1,i+n/2)} (a^j_xy)²)^β

Where:
  a^i_xy = activity of neuron at position (x,y) in feature map i (after ReLU)
  b^i_xy = normalized activity
  N      = total number of feature maps in the layer
  n      = number of adjacent feature maps for normalization (n=5)
  k      = bias term (k=2)
  α      = scaling constant (α=10^-4)
  β      = exponent (β=0.75)
```

**Note:** LRN was later shown to have **minimal benefit** and was replaced by **Batch Normalization** in subsequent architectures. VGGNet showed similar performance without LRN.

### 5. Multi-GPU Training

```
Hardware: 2× NVIDIA GTX 580 GPUs (3GB VRAM each)

Strategy: Model Parallelism
───────────────────────────

  The 96 Conv1 filters were split: 48 on GPU 1, 48 on GPU 2.
  Most layers communicate only within the same GPU.
  Conv3 and FC layers communicate across GPUs.

  This reduced training time from ~12 days to ~5-6 days.
  Also improved accuracy by ~1.7% (top-1) vs. single-GPU half-width network.
```

---

## Parameter Calculations

### Complete Parameter Summary

```
Layer     Config                   Params         % of Total
──────────────────────────────────────────────────────────────
Conv1     96 × (11×11×3) + 96        34,944        0.06%
Conv2     256 × (5×5×96) + 256      614,656        1.02%
Conv3     384 × (3×3×256) + 384     885,120        1.47%
Conv4     384 × (3×3×384) + 384   1,327,488        2.20%
Conv5     256 × (3×3×384) + 256     884,992        1.47%
──────────────────────────────────────────────────────────────
Conv Total                        3,747,200        6.22%
──────────────────────────────────────────────────────────────
FC6       9216 × 4096 + 4096     37,752,832       62.62%
FC7       4096 × 4096 + 4096     16,781,312       27.84%
FC8       4096 × 1000 + 1000      4,097,000        6.80%
──────────────────────────────────────────────────────────────
FC Total                         58,631,144       97.26%
──────────────────────────────────────────────────────────────
GRAND TOTAL                      ~60,278,344      (hidden, not shown)
──────────────────────────────────────────────────────────────

Key Insight: ~93.8% of parameters are in FC layers!
             Convolutional layers are parameter-efficient.
             This motivated later architectures (GoogLeNet, ResNet) to
             reduce or eliminate FC layers.
```

### Memory Requirements (Forward Pass)

```
Layer      Output Shape      Activations     Memory (float32)
──────────────────────────────────────────────────────────────
Input      227×227×3            154,587         602 KB
Conv1      55×55×96             290,400       1,135 KB
Pool1      27×27×96              69,984         273 KB
Conv2      27×27×256            186,624         729 KB
Pool2      13×13×256             43,264         169 KB
Conv3      13×13×384             64,896         254 KB
Conv4      13×13×384             64,896         254 KB
Conv5      13×13×256             43,264         169 KB
Pool5      6×6×256                9,216          36 KB
FC6        4,096                  4,096          16 KB
FC7        4,096                  4,096          16 KB
FC8        1,000                  1,000           4 KB
──────────────────────────────────────────────────────────────
Total activations:             936,323       ~3,657 KB ≈ 3.6 MB

Parameters: ~60M × 4 bytes = ~240 MB
```

---

## Mathematical Foundations

### Output Size Formula

```
For each convolutional or pooling layer:

              Input_size - Filter_size + 2 × Padding
Output_size = ───────────────────────────────────────── + 1
                            Stride

Applied to AlexNet:
  Conv1: (227 - 11 + 2×0) / 4 + 1 = 55
  Pool1: (55 - 3 + 2×0) / 2 + 1 = 27
  Conv2: (27 - 5 + 2×2) / 1 + 1 = 27
  Pool2: (27 - 3 + 2×0) / 2 + 1 = 13
  Conv3: (13 - 3 + 2×1) / 1 + 1 = 13
  Conv4: (13 - 3 + 2×1) / 1 + 1 = 13
  Conv5: (13 - 3 + 2×1) / 1 + 1 = 13
  Pool5: (13 - 3 + 2×0) / 2 + 1 = 6
```

### Receptive Field Calculation

The **receptive field** is the region of the input image that influences a particular output neuron.

```
Layer    Kernel  Stride  Receptive Field  Jump
──────────────────────────────────────────────
Input     -       -          1              1
Conv1    11       4         11              4
Pool1     3       2         19              8
Conv2     5       1         51              8
Pool2     3       2         67             16
Conv3     3       1         99             16
Conv4     3       1        131             16
Conv5     3       1        163             16
Pool5     3       2        195             32

Final receptive field: 195 × 195 pixels
(out of 227 × 227 input → covers ~74% of the image width)
```

### FLOPs Calculation

```
FLOPs for a Conv layer = 2 × K × K × C_in × C_out × H_out × W_out

Conv1: 2 × 11 × 11 × 3 × 96 × 55 × 55     = 211M FLOPs
Conv2: 2 × 5 × 5 × 96 × 256 × 27 × 27      = 448M FLOPs
Conv3: 2 × 3 × 3 × 256 × 384 × 13 × 13     = 299M FLOPs
Conv4: 2 × 3 × 3 × 384 × 384 × 13 × 13     = 449M FLOPs
Conv5: 2 × 3 × 3 × 384 × 256 × 13 × 13     = 299M FLOPs
FC6:   2 × 9216 × 4096                       =  75M FLOPs
FC7:   2 × 4096 × 4096                       =  34M FLOPs
FC8:   2 × 4096 × 1000                       =   8M FLOPs
──────────────────────────────────────────────────────────
Total: ~1.5 billion FLOPs (for one forward pass)

Note: Conv layers dominate FLOPs, while FC layers dominate parameters.
```

---

## Worked Example

### Forward Pass Through AlexNet

Let's trace a 227×227×3 image of a "golden retriever" through the network.

#### Step 1: Conv1 Processing

```
Input: [227, 227, 3] — an RGB photo of a golden retriever

Filter 0 (horizontal edge detector example):
  11×11×3 = 363 weights + 1 bias

  At position (0,0):
                    10  10  2
  z[0,0,0] = b_0 + ∑   ∑   ∑  w[p,q,c,0] · x[p,q,c]
                   p=0 q=0 c=0

  After ReLU: a[0,0,0] = max(0, z[0,0,0])

  Slide with stride 4:
    Position (0,0) → (0,4) → (0,8) → ... → (0,216)
    = 55 positions horizontally
    = 55 × 55 = 3,025 positions per filter
    × 96 filters = 290,400 output activations

Output: [55, 55, 96]
```

#### Step 2: Pool1

```
Max pooling with 3×3 kernel, stride 2:

Example 3×3 region from feature map 0:
┌──────┬──────┬──────┐
│ 0.82 │ 0.45 │ 0.93 │
├──────┼──────┼──────┤
│ 0.31 │ 0.67 │ 0.28 │  → max = 0.93
├──────┼──────┼──────┤
│ 0.55 │ 0.14 │ 0.76 │
└──────┴──────┴──────┘

Output: [27, 27, 96]
```

#### Step 3: Conv2 → Pool2

```
Conv2: 256 filters of 5×5×96, padding=2
  [27, 27, 96] → [27, 27, 256]
  Each output value = sum of 5×5×96 = 2,400 multiply-adds + bias

Pool2: 3×3 max pool, stride 2
  [27, 27, 256] → [13, 13, 256]
```

#### Step 4: Conv3 → Conv4 → Conv5

```
Conv3: [13,13,256] → [13,13,384]  (3×3×256×384 = 884,736 weights)
Conv4: [13,13,384] → [13,13,384]  (3×3×384×384 = 1,327,104 weights)
Conv5: [13,13,384] → [13,13,256]  (3×3×384×256 = 884,736 weights)
```

#### Step 5: Pool5 → Flatten

```
Pool5: [13,13,256] → [6,6,256]
Flatten: [6,6,256] → [9216]
```

#### Step 6: Fully Connected Layers

```
FC6: [9216] → [4096] (W: 9216×4096) → ReLU → Dropout(0.5)
FC7: [4096] → [4096] (W: 4096×4096) → ReLU → Dropout(0.5)
FC8: [4096] → [1000] (W: 4096×1000) → Softmax

Output probabilities (top 5):
  golden_retriever:     0.847  ← correct!
  Labrador_retriever:   0.065
  cocker_spaniel:       0.023
  Irish_setter:         0.018
  tennis_ball:          0.009
```

---

## PyTorch Implementation

### AlexNet from Scratch

```python
import torch
import torch.nn as nn


class AlexNet(nn.Module):
    """
    AlexNet architecture for ImageNet classification.

    Input: 227×227×3 RGB image
    Output: 1000 class probabilities
    Parameters: ~60 million
    """

    def __init__(self, num_classes=1000):
        super(AlexNet, self).__init__()

        # Feature extraction (convolutional layers)
        self.features = nn.Sequential(
            # Conv1: 96 filters, 11×11, stride 4
            nn.Conv2d(3, 96, kernel_size=11, stride=4, padding=0),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),              # 55→27

            # Conv2: 256 filters, 5×5, pad 2
            nn.Conv2d(96, 256, kernel_size=5, stride=1, padding=2),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),              # 27→13

            # Conv3: 384 filters, 3×3, pad 1
            nn.Conv2d(256, 384, kernel_size=3, stride=1, padding=1),
            nn.ReLU(inplace=True),

            # Conv4: 384 filters, 3×3, pad 1
            nn.Conv2d(384, 384, kernel_size=3, stride=1, padding=1),
            nn.ReLU(inplace=True),

            # Conv5: 256 filters, 3×3, pad 1
            nn.Conv2d(384, 256, kernel_size=3, stride=1, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),              # 13→6
        )

        # Classification (fully connected layers)
        self.classifier = nn.Sequential(
            nn.Dropout(p=0.5),
            nn.Linear(256 * 6 * 6, 4096),
            nn.ReLU(inplace=True),

            nn.Dropout(p=0.5),
            nn.Linear(4096, 4096),
            nn.ReLU(inplace=True),

            nn.Linear(4096, num_classes),
        )

    def forward(self, x):
        x = self.features(x)           # [batch, 256, 6, 6]
        x = x.view(x.size(0), -1)      # [batch, 9216]
        x = self.classifier(x)         # [batch, num_classes]
        return x


# ─── Verify Architecture ───
model = AlexNet(num_classes=1000)
print(model)

# Count parameters
total_params = sum(p.numel() for p in model.parameters())
print(f"\nTotal parameters: {total_params:,}")

# Test forward pass
x = torch.randn(1, 3, 227, 227)
output = model(x)
print(f"Input shape:  {x.shape}")
print(f"Output shape: {output.shape}")
```

### Layer-by-Layer Shape Verification

```python
def trace_alexnet_shapes(model, input_tensor):
    """Print the output shape after each layer."""
    x = input_tensor
    print(f"{'Layer':<30} {'Output Shape':<20} {'#Params':>12}")
    print("─" * 65)

    for name, layer in model.features.named_children():
        x = layer(x)
        params = sum(p.numel() for p in layer.parameters())
        print(f"features.{name} ({layer.__class__.__name__:>12})"
              f"  {str(list(x.shape)):<20} {params:>12,}")

    x = x.view(x.size(0), -1)
    print(f"{'Flatten':<30} {str(list(x.shape)):<20} {'0':>12}")

    for name, layer in model.classifier.named_children():
        x = layer(x)
        params = sum(p.numel() for p in layer.parameters())
        print(f"classifier.{name} ({layer.__class__.__name__:>10})"
              f"  {str(list(x.shape)):<20} {params:>12,}")


model = AlexNet()
x = torch.randn(1, 3, 227, 227)
trace_alexnet_shapes(model, x)
```

### Training with ImageNet (Conceptual Pipeline)

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchvision import datasets, transforms
from torch.utils.data import DataLoader


# ─── Data Augmentation (as used in the original paper) ───
train_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.RandomCrop(227),
    transforms.RandomHorizontalFlip(),
    transforms.ColorJitter(brightness=0.1, contrast=0.1, saturation=0.1),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
])

test_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(227),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
])

# For actual ImageNet training:
# train_dataset = datasets.ImageNet(root='./data', split='train',
#                                    transform=train_transform)
# val_dataset = datasets.ImageNet(root='./data', split='val',
#                                  transform=test_transform)

# ─── Training Configuration (matching original paper) ───
model = AlexNet(num_classes=1000)
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = model.to(device)

# Original paper: SGD with momentum
optimizer = optim.SGD(model.parameters(),
                      lr=0.01,          # initial LR
                      momentum=0.9,
                      weight_decay=0.0005)

# Reduce LR by 10× when validation error plateaus
scheduler = optim.lr_scheduler.ReduceLROnPlateau(
    optimizer, mode='min', factor=0.1, patience=3
)

criterion = nn.CrossEntropyLoss()


# ─── Weight Initialization (original paper) ───
def init_weights(m):
    """Initialize weights as described in the AlexNet paper."""
    if isinstance(m, nn.Conv2d) or isinstance(m, nn.Linear):
        nn.init.normal_(m.weight, mean=0.0, std=0.01)
        if m.bias is not None:
            # Conv2, Conv4, Conv5, FC layers: bias = 1
            # Conv1, Conv3: bias = 0
            nn.init.constant_(m.bias, 0)

model.apply(init_weights)

# Set specific biases to 1 (as in original paper)
model.features[3].bias.data.fill_(1)   # Conv2
model.features[8].bias.data.fill_(1)   # Conv4
model.features[10].bias.data.fill_(1)  # Conv5
model.classifier[1].bias.data.fill_(1) # FC6
model.classifier[4].bias.data.fill_(1) # FC7


# ─── Training Loop (simplified) ───
def train_one_epoch(model, loader, criterion, optimizer, device):
    model.train()
    running_loss = 0.0
    correct = 0
    total = 0

    for images, labels in loader:
        images, labels = images.to(device), labels.to(device)

        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()

        running_loss += loss.item() * images.size(0)
        _, predicted = outputs.max(1)
        total += labels.size(0)
        correct += predicted.eq(labels).sum().item()

    return running_loss / total, 100.0 * correct / total


print(f"Model on {device}")
print(f"Parameters: {sum(p.numel() for p in model.parameters()):,}")
```

### Using Pretrained AlexNet from torchvision

```python
import torchvision.models as models

# Load pretrained AlexNet
alexnet = models.alexnet(weights='IMAGENET1K_V1')
alexnet.eval()

# Use as feature extractor
features = alexnet.features
classifier = alexnet.classifier

# Fine-tune for a different number of classes
alexnet.classifier[6] = nn.Linear(4096, 10)  # 10 classes instead of 1000
```

---

## Applications

### 1. ImageNet Classification (Primary)
- **1000 categories** covering animals, objects, scenes, food, vehicles, etc.
- Achieved **15.3% top-5 error** (human-level is estimated at ~5.1%)

### 2. Feature Extraction / Transfer Learning
- AlexNet features (especially FC7) became a **universal image representation**
- Could be extracted and used as features for other vision tasks:
  - Object detection (R-CNN used AlexNet features)
  - Scene classification
  - Image retrieval
  - Fine-grained recognition

### 3. Impact on the Field
- **Triggered the deep learning revolution** in computer vision
- Inspired subsequent architectures: ZFNet (2013), VGGNet (2014), GoogLeNet (2014)
- Demonstrated that **scaling up (data + compute + model size)** works
- Popularized GPU training as essential for deep learning
- Led to massive industry investment in AI/deep learning

### 4. Art and Creative Applications
- Neural style transfer (using AlexNet/VGG features)
- Deep dream visualizations
- Understanding what neural networks "see"

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Architecture** | 5 Conv + 3 FC |
| **Input Size** | 227 × 227 × 3 (RGB) |
| **Parameters** | ~60 million |
| **FLOPs** | ~1.5 billion |
| **Activation** | ReLU (first major use) |
| **Pooling** | 3×3 max pooling, stride 2 (overlapping) |
| **Normalization** | Local Response Normalization (deprecated) |
| **Regularization** | Dropout (p=0.5) in FC layers |
| **Data Augmentation** | Random crops, flips, PCA color jitter |
| **Training** | SGD, momentum=0.9, weight decay=5×10⁻⁴ |
| **Hardware** | 2× GTX 580 GPUs (6 days training) |
| **Top-5 Error** | 15.3% (ILSVRC-2012 winner) |
| **Key Innovation** | Proved deep CNNs + GPUs = state-of-the-art |
| **Where are the params?** | ~94% in FC layers |

---

## Revision Questions

### Q1: The ImageNet Breakthrough
**Why was AlexNet's victory in ILSVRC-2012 considered a watershed moment in AI?**

<details>
<summary>Answer</summary>

AlexNet achieved **15.3% top-5 error** vs. the runner-up's **26.2%** — a 10.9% absolute improvement, which was unprecedented (typical yearly improvements were 1-2%). This proved that deep convolutional neural networks trained on GPUs could vastly outperform traditional computer vision approaches based on hand-crafted features. It triggered a paradigm shift where virtually all subsequent competition entries used deep learning, and sparked massive industry investment in AI.
</details>

### Q2: ReLU Advantage
**Explain why AlexNet used ReLU instead of sigmoid/tanh. What are the specific advantages?**

<details>
<summary>Answer</summary>

1. **No saturation for positive values**: sigmoid/tanh gradients vanish for large |x|, causing very slow learning in deep networks. ReLU has a constant gradient of 1 for x > 0.
2. **Faster convergence**: Krizhevsky reported 6× faster training on CIFAR-10 compared to tanh.
3. **Sparse activation**: Many ReLU outputs are 0, creating sparser, more efficient representations.
4. **Computational simplicity**: ReLU is just `max(0, x)` — much cheaper than computing exponentials for sigmoid/tanh.
</details>

### Q3: Parameter Distribution
**Why do the fully connected layers contain ~94% of AlexNet's parameters? What problem does this create, and how did the authors address it?**

<details>
<summary>Answer</summary>

The FC layers have huge parameter counts because they connect every input to every output:
- FC6: 9,216 × 4,096 = ~37.7M parameters
- FC7: 4,096 × 4,096 = ~16.8M parameters

This creates a severe **overfitting risk** — 60M parameters trained on only 1.2M images. The authors addressed this with:
1. **Dropout (p=0.5)** in FC6 and FC7 — randomly zeroes out 50% of neurons during training
2. **Data augmentation** — random crops, horizontal flips, and PCA color jittering
3. **Weight decay** — L2 regularization with λ = 5×10⁻⁴

This parameter distribution motivated later architectures (GoogLeNet, ResNet) to reduce or eliminate FC layers using global average pooling.
</details>

### Q4: Overlapping Pooling
**AlexNet used 3×3 max pooling with stride 2 (overlapping). How does this differ from non-overlapping pooling, and what benefit was reported?**

<details>
<summary>Answer</summary>

- **Non-overlapping pooling**: kernel size = stride (e.g., 2×2 pool with stride 2). Each input element contributes to exactly one output.
- **Overlapping pooling**: kernel size > stride (e.g., 3×3 pool with stride 2). Adjacent pooling windows share input elements.

Krizhevsky et al. reported that overlapping pooling reduced the top-1 error by ~0.4% and the top-5 error by ~0.3% compared to non-overlapping pooling. The overlapping makes the model slightly harder to overfit.
</details>

### Q5: Input Size Discrepancy
**The original paper sometimes cites 224×224 input and sometimes 227×227. Why does the math only work with 227×227?**

<details>
<summary>Answer</summary>

With Conv1 using kernel=11, stride=4, and padding=0:
- Input 224: (224 - 11) / 4 + 1 = 213/4 + 1 = 53.25 + 1 → **not an integer!**
- Input 227: (227 - 11) / 4 + 1 = 216/4 + 1 = 54 + 1 = **55** ✓

The correct input size that makes the math work cleanly is **227×227**. The paper's mention of 224 is likely a typo or refers to a preprocessing step before padding. Some implementations use 224×224 with padding=2 in Conv1 to get the same result.
</details>

### Q6: LeNet-5 to AlexNet
**Compare LeNet-5 and AlexNet. What stayed the same and what changed?**

<details>
<summary>Answer</summary>

**Same principles:**
- Conv → Pool → Conv → Pool → FC pipeline
- Spatial dimensions decrease while channels increase
- End-to-end training with backpropagation

**Key changes:**

| Aspect | LeNet-5 | AlexNet |
|--------|---------|---------|
| Scale | 60K params | 60M params (1000×) |
| Depth | 5 layers | 8 layers |
| Input | 32×32 gray | 227×227 RGB |
| Activation | tanh | ReLU |
| Pooling | avg | max (overlapping) |
| Regularization | none | Dropout + augmentation |
| Training | CPU, minutes | 2 GPUs, 6 days |
| Task | 10 digits | 1000 categories |
</details>

---

## Key Takeaways

```
╔══════════════════════════════════════════════════════════════════════╗
║  AlexNet Key Takeaways                                              ║
║                                                                      ║
║  1. Won ILSVRC-2012 with 15.3% top-5 error (10.9% better than #2)  ║
║  2. Sparked the modern deep learning revolution                      ║
║  3. Introduced ReLU, Dropout, Data Augmentation to mainstream CNN   ║
║  4. 60M parameters — 94% in FC layers → overfitting challenge       ║
║  5. Showed that scale (data + compute + model) is key               ║
║  6. Made GPU training the standard for deep learning                ║
║  7. Every subsequent CNN architecture built on AlexNet's insights   ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

[← Previous: LeNet-5](01-lenet-5.md) | [Back to Unit Overview](../README.md) | [Next: VGGNet →](03-vggnet.md)
