# Vision Transformer (ViT) — Images as Sequences of Patches

> **Unit 6 · Modern CNN Architectures · Topic 5/6**

---

## Table of Contents

1. [Overview & Motivation](#1-overview--motivation)
2. [Patch Embedding — Images to Sequences](#2-patch-embedding--images-to-sequences)
3. [Class Token & Positional Embedding](#3-class-token--positional-embedding)
4. [Transformer Encoder](#4-transformer-encoder)
5. [Full ViT Architecture](#5-full-vit-architecture)
6. [Mathematical Formulation](#6-mathematical-formulation)
7. [When ViT Outperforms CNNs](#7-when-vit-outperforms-cnns)
8. [Hybrid Approaches](#8-hybrid-approaches)
9. [DeiT — Data-Efficient Image Transformers](#9-deit--data-efficient-image-transformers)
10. [Worked Examples](#10-worked-examples)
11. [PyTorch Implementation](#11-pytorch-implementation)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)
14. [Navigation](#14-navigation)

---

## 1. Overview & Motivation

The **Vision Transformer (ViT)** (Dosovitskiy et al., 2021) applies the Transformer architecture — originally designed for NLP — directly to images. It treats an image as a **sequence of patches**, analogous to a sequence of words.

```
NLP Transformer:                    Vision Transformer:
"The cat sat on mat"               ┌──┬──┬──┬──┐
 ↓   ↓   ↓  ↓  ↓                  │p₁│p₂│p₃│p₄│  224×224 image
[The][cat][sat][on][mat]           │p₅│p₆│p₇│p₈│  → 16×16 patches
  ↓    ↓    ↓   ↓   ↓             │p₉│p₁₀│p₁₁│p₁₂│  → 196 patches
  Word embeddings                  │p₁₃│p₁₄│p₁₅│p₁₆│  (14×14 grid)
  ↓                                └──┴──┴──┴──┘
  Transformer                       ↓
  ↓                                Patch embeddings
  Classification                    ↓
                                   Transformer
                                    ↓
                                   Classification

Key insight: Self-attention has NO inductive bias for locality
→ Can learn both local AND global patterns
→ But needs MORE DATA than CNNs
```

---

## 2. Patch Embedding — Images to Sequences

### Step 1: Split Image into Patches

```
Input image: x ∈ ℝ^(H × W × C)  (e.g., 224 × 224 × 3)
Patch size:  P × P               (e.g., 16 × 16)

Number of patches: N = (H × W) / P² = (224 × 224) / (16 × 16) = 196

Each patch: xₚ ∈ ℝ^(P² · C)     (e.g., 16 × 16 × 3 = 768 values)

Image (224 × 224 × 3):
┌────┬────┬────┬─────────────────────┐
│ p₁ │ p₂ │ p₃ │ ...    p₁₄         │
├────┼────┼────┤                     │  14 columns
│ p₁₅│ p₁₆│ p₁₇│ ...    p₂₈        │
├────┼────┼────┤                     │  14 rows
│ ...│ ...│ ...│         ...         │
│    │    │    │                     │
│p₁₈₃│p₁₈₄│p₁₈₅│ ...    p₁₉₆       │  = 196 patches total
└────┴────┴────┴─────────────────────┘
 16px  16px
```

### Step 2: Linear Projection to Embeddings

```
Flatten each patch and project to D dimensions:

xₚ ∈ ℝ^(P²·C) → E · xₚ = z₀ ∈ ℝ^D

Where E ∈ ℝ^(D × P²·C) is a learnable projection matrix

Equivalently: a Conv2d with kernel=P, stride=P:
  nn.Conv2d(3, D, kernel_size=16, stride=16)
  
  This extracts non-overlapping 16×16 patches and
  projects each to D dimensions in one operation.

Result: Sequence of N patch embeddings
  Z₀ = [z₁, z₂, ..., z_N]  ∈ ℝ^(N × D)
```

---

## 3. Class Token & Positional Embedding

### Class Token [CLS]

```
Prepend a learnable [CLS] token to the sequence:

  z₀ = [x_class ; z₁E ; z₂E ; ... ; z_NE]  ∈ ℝ^((N+1) × D)

  x_class ∈ ℝ^D is a learnable parameter

Purpose:
  - Aggregates information from all patches via self-attention
  - Its output state is used for classification
  - Analogous to BERT's [CLS] token

  [CLS] z₁  z₂  z₃  z₄  z₅  ... z₁₉₆
    ↕   ↕   ↕   ↕   ↕   ↕       ↕      ← self-attention
    ↕   ↕   ↕   ↕   ↕   ↕       ↕         (all attend to all)
    ↓
  Classification head (MLP)
```

### Positional Embedding

```
Add learnable position embeddings to preserve spatial information:

  z₀ = [x_class ; z₁E ; z₂E ; ... ; z_NE] + E_pos

  E_pos ∈ ℝ^((N+1) × D) — learnable 1D positional embeddings

Why needed?
  Transformers are permutation-invariant — without position info,
  the model can't distinguish [p₁,p₂,p₃] from [p₃,p₁,p₂]

Position embedding similarity (learned):
        1   2   3   4   5   6   7   8   9  10  11  12  13  14
    1  [█   ▓   ▒   ░   ·   ·   ·   ·   ·   ·   ·   ░   ▒   ▓]
    2  [▓   █   ▓   ▒   ░   ·   ·   ·   ·   ·   ·   ·   ░   ▒]
    3  [▒   ▓   █   ▓   ▒   ░   ·   ·   ·   ·   ·   ·   ·   ░]
    ...
    
  → Learns 2D structure automatically from 1D positions!
  → Nearby patches have similar position embeddings
  → Same row/column patches show correlation patterns
```

---

## 4. Transformer Encoder

### Multi-Head Self-Attention (MSA)

```
Self-Attention Mechanism:

Input: z ∈ ℝ^(N+1 × D)

For each head h (H heads total):
  Qₕ = z · W_Q^h     ∈ ℝ^((N+1) × dₕ)    where dₕ = D/H
  Kₕ = z · W_K^h     ∈ ℝ^((N+1) × dₕ)
  Vₕ = z · W_V^h     ∈ ℝ^((N+1) × dₕ)

  Attention(Q,K,V) = softmax(Q·K^T / √dₕ) · V

  ┌───────────────────────────────────────────────┐
  │                                               │
  │  Scores = Q · K^T / √dₕ                      │
  │                                               │
  │       k₁   k₂   k₃  ...  k₁₉₇              │
  │  q₁ [0.8  0.1  0.3  ...  0.2 ]              │
  │  q₂ [0.2  0.9  0.1  ...  0.1 ]              │
  │  q₃ [0.3  0.1  0.7  ...  0.3 ]              │
  │  ... [... ]                                   │
  │  q₁₉₇[0.1  0.2  0.3  ...  0.6 ]             │
  │                                               │
  │  After softmax: each row sums to 1            │
  │  → Weighted combination of all values          │
  └───────────────────────────────────────────────┘

Multi-head: Concat all heads and project
  MSA(z) = Concat(head₁, ..., headₕ) · W_O
  Where W_O ∈ ℝ^(D × D)

Complexity: O(N² · D) — quadratic in sequence length!
  For ViT-B with 196 patches: 196² = 38,416 attention pairs
  For ViT at 384×384 with 16×16 patches: (384/16)² = 576 patches
    → 576² = 331,776 attention pairs (much more!)
```

### MLP Block

```
Feed-Forward Network applied to each token independently:

MLP(z) = GELU(z · W₁ + b₁) · W₂ + b₂

Where:
  W₁ ∈ ℝ^(D × 4D)    — expand by 4×
  W₂ ∈ ℝ^(4D × D)    — project back
  GELU = x · Φ(x)     — Gaussian Error Linear Unit

GELU Activation:
           │      ╱──────
           │     ╱
       0   ├────╱
           │  ╱
           │╱
           └───────────── x
           Unlike ReLU: smooth, allows small negative gradients
```

### Transformer Encoder Block

```
Transformer Block (repeated L times):

Input z_l
    │
    ├────────────────────── Residual
    ▼                           │
┌──────────────┐                │
│  LayerNorm   │                │
└──────┬───────┘                │
       ▼                        │
┌──────────────┐                │
│  Multi-Head  │                │
│  Self-Attn   │                │
└──────┬───────┘                │
       │                        │
       └────── Add ◄────────────┘
               │
    ├──────────┴──────── Residual
    ▼                        │
┌──────────────┐             │
│  LayerNorm   │             │
└──────┬───────┘             │
       ▼                     │
┌──────────────┐             │
│  MLP         │             │
│  (D→4D→D)   │             │
└──────┬───────┘             │
       │                     │
       └────── Add ◄─────────┘
               │
               ▼
           Output z_{l+1}

Equations:
  z'_l = MSA(LN(z_l)) + z_l
  z_{l+1} = MLP(LN(z'_l)) + z'_l

Note: Pre-norm (LN before attention/MLP) — different from original Transformer
```

---

## 5. Full ViT Architecture

```
Complete ViT Pipeline:

Input Image (224×224×3)
       │
       ▼
┌──────────────────────┐
│ Patch Embedding      │  Split into 14×14 = 196 patches of 16×16
│ Conv2d(3,D,16,16)    │  Each patch → D-dim vector
└──────────┬───────────┘
           │  (196 × D)
           ▼
┌──────────────────────┐
│ Prepend [CLS] token  │  (197 × D)
│ Add position embed   │
│ Dropout              │
└──────────┬───────────┘
           │
     ┌─────┴─────┐
     │ Transformer│ × L layers
     │ Encoder    │
     │ Block      │
     └─────┬─────┘
           │  (197 × D)
           ▼
┌──────────────────────┐
│ Extract [CLS] output │  Take first token: z_L^0 ∈ ℝ^D
│ LayerNorm            │
└──────────┬───────────┘
           │  (D,)
           ▼
┌──────────────────────┐
│ Classification Head  │  MLP: D → num_classes
│ (Linear or MLP)      │
└──────────┬───────────┘
           │
           ▼
      Class Logits (num_classes,)


ViT Model Variants:
┌───────────┬────────┬────────┬────────┬────────┬──────────┐
│  Model    │ Layers │ Hidden │ MLP    │ Heads  │ Params   │
│           │ (L)    │ (D)    │ (4D)   │ (H)    │ (M)     │
├───────────┼────────┼────────┼────────┼────────┼──────────┤
│ ViT-Tiny  │   12   │  192   │  768   │   3    │   5.7    │
│ ViT-Small │   12   │  384   │ 1536   │   6    │  22.1    │
│ ViT-Base  │   12   │  768   │ 3072   │  12    │  86.6    │
│ ViT-Large │   24   │ 1024   │ 4096   │  16    │ 304.4    │
│ ViT-Huge  │   32   │ 1280   │ 5120   │  16    │ 632.0    │
└───────────┴────────┴────────┴────────┴────────┴──────────┘
```

---

## 6. Mathematical Formulation

### Complete Forward Pass

```
Given image x ∈ ℝ^(H×W×C), patch size P, embedding dim D, L layers:

1. Patch embedding:
   xₚ = [x¹ₚ; x²ₚ; ...; xᴺₚ],  N = HW/P²
   z₀ = [x_class; x¹ₚE; x²ₚE; ...; xᴺₚE] + E_pos
   Where E ∈ ℝ^(P²C × D),  E_pos ∈ ℝ^((N+1) × D)

2. Transformer encoder (for l = 1, ..., L):
   z'_l = MSA(LN(z_{l-1})) + z_{l-1}
   z_l  = MLP(LN(z'_l)) + z'_l

3. Classification:
   y = MLP_head(LN(z_L⁰))
   Where z_L⁰ is the [CLS] token output from the final layer

MSA details:
   [Q,K,V] = LN(z) · [W_Q, W_K, W_V]
   A = softmax(QK^T / √(D/H))     ∈ ℝ^((N+1)×(N+1))
   SA(z) = A · V
   MSA(z) = [SA₁(z); ...; SAₕ(z)] · W_O
```

### Computational Complexity

```
Self-Attention: O(N² · D)  — quadratic in number of patches
MLP:            O(N · D²)  — linear in patches, quadratic in dimension
Total per layer: O(N² · D + N · D²)

For ViT-Base (N=196, D=768):
  Attention: 196² × 768 ≈ 29.5M per layer
  MLP:       196 × 768² ≈ 115.6M per layer
  MLP dominates for small N!

For higher resolution (N=576, D=768):
  Attention: 576² × 768 ≈ 254.8M per layer ← attention now dominates
  MLP:       576 × 768² ≈ 339.7M per layer

CNN comparison:
  Conv layer: O(K² · C_in · C_out · H · W) — linear in spatial dims
  → CNNs scale better with resolution!
```

---

## 7. When ViT Outperforms CNNs

```
Performance vs Dataset Size:

Accuracy
  │
  │                                    ViT ──────·
  │                              ·──────
  │                        ·────
  │                  ·────
  │            ·────
  │      ·────                    CNN ──────·
  │ ·────                   ·──────
  │                    ·───
  │               ·───
  │          ·───
  │     ·───
  │·───
  └─────────────────────────────────────
    1K   10K  100K  1M   10M  100M  300M
                Dataset Size

Key observations:
  • Small data (< 1M):     CNN > ViT (inductive bias helps)
  • Medium data (~10M):    CNN ≈ ViT
  • Large data (> 100M):   ViT > CNN (no inductive bias limits)
```

### Why CNNs Win on Small Data

```
CNN Inductive Biases:
┌────────────────────────────────────────────────────────────┐
│ 1. LOCALITY:     Convolution operates on local patches     │
│    → Don't need to LEARN that nearby pixels are related    │
│                                                            │
│ 2. TRANSLATION EQUIVARIANCE: Same filter everywhere        │
│    → "Cat" detected anywhere without extra training        │
│                                                            │
│ 3. HIERARCHICAL: Progressively larger receptive fields     │
│    → Edges → textures → parts → objects (built-in!)       │
└────────────────────────────────────────────────────────────┘

ViT has NONE of these:
┌────────────────────────────────────────────────────────────┐
│ • Every patch attends to EVERY other patch from layer 1    │
│ • No shared weights across spatial locations               │
│ • Must LEARN spatial relationships from scratch            │
│ • Needs massive data to discover what CNNs get for free    │
└────────────────────────────────────────────────────────────┘

But with enough data:
  ViT's flexibility becomes an ADVANTAGE
  → Can learn non-local relationships that CNNs miss
  → No receptive field limitation
  → Better representation of global context
```

### Scaling Properties

```
┌───────────────────┬────────────────┬─────────────┬───────────────┐
│ Model             │ Pretrain Data  │ ImageNet Top1│ Params (M)    │
├───────────────────┼────────────────┼─────────────┼───────────────┤
│ ResNet-152 (CNN)  │ ImageNet-1K    │   78.3%     │    60         │
│ ViT-B/16          │ ImageNet-1K    │   77.9%     │    86         │
│ ViT-B/16          │ ImageNet-21K   │   84.0%     │    86         │
│ ViT-L/16          │ JFT-300M       │   87.8%     │   304         │
│ ViT-H/14          │ JFT-300M       │   88.6%     │   632         │
│ EfficientNet-L2   │ JFT-300M       │   88.4%     │   480         │
└───────────────────┴────────────────┴─────────────┴───────────────┘

ViT needs ~14M+ images to outperform CNNs of similar size
```

---

## 8. Hybrid Approaches

### CNN + Transformer Hybrids

```
Approach 1: CNN Feature Extractor → Transformer
─────────────────────────────────────────────────
  Image → CNN (ResNet stages 1-3) → Feature maps → Flatten → Transformer
  
  Benefits:
  - CNN provides good local features
  - Transformer models global relationships
  - Works well with less data than pure ViT

Approach 2: Interleave CNN and Attention
─────────────────────────────────────────
  Image → [Conv + Attention] × L → Output
  
  Examples: CoAtNet, BoTNet

CoAtNet Architecture:
  Stage 0: MBConv (CNN) — local features
  Stage 1: MBConv (CNN) — local features  
  Stage 2: Transformer — global attention
  Stage 3: Transformer — global attention
  
  → Best of both worlds!


Approach 3: Patch embedding via CNN stem
────────────────────────────────────────
  Instead of:  16×16 Conv with stride 16 (harsh downsampling)
  Use:         Multiple small convolutions (3×3 stride 2, repeated)
  
  → Better for smaller datasets
  → Smoother downsampling preserves more information
```

---

## 9. DeiT — Data-Efficient Image Transformers

DeiT (Touvron et al., 2021) trains ViT **on ImageNet-1K only** (no JFT-300M) using strong regularization and distillation.

### Key Training Strategies

```
DeiT Training Recipe:
┌────────────────────────────────────────────────┐
│ Data Augmentation:                              │
│   • RandAugment                                │
│   • Mixup (α=0.8)                              │
│   • CutMix (α=1.0)                             │
│   • Random Erasing (p=0.25)                    │
│                                                 │
│ Regularization:                                 │
│   • Stochastic Depth (drop rate 0.1)           │
│   • Label Smoothing (ε=0.1)                    │
│                                                 │
│ Optimization:                                   │
│   • AdamW (lr=5e-4, wd=0.05)                  │
│   • Cosine LR decay                           │
│   • 300 epochs with warmup                     │
│   • Batch size 1024                            │
└────────────────────────────────────────────────┘
```

### Distillation Token

```
DeiT Distillation:

Standard ViT:
  [CLS] [p₁] [p₂] ... [pₙ] → Transformer → [CLS]_out → Class prediction

DeiT with distillation:
  [CLS] [DIST] [p₁] [p₂] ... [pₙ] → Transformer → [CLS]_out → Soft label loss
                                                     [DIST]_out → Hard label loss
                                                                   (from teacher)

Teacher: RegNetY-16GF (CNN) — provides "hard" labels
Student: DeiT (ViT) — learns from both true labels and teacher predictions

Loss = (1-λ)·CE(y_cls, y_true) + λ·CE(y_dist, y_teacher)

Results:
┌──────────────┬──────────┬──────────┬─────────────┐
│ Model        │ Params(M)│ GFLOPs   │ Top-1 Acc(%)│
├──────────────┼──────────┼──────────┼─────────────┤
│ DeiT-Ti      │    5.7   │   1.3    │   72.2      │
│ DeiT-S       │   22.1   │   4.6    │   79.8      │
│ DeiT-B       │   86.6   │  17.6    │   81.8      │
│ DeiT-B ↑384  │   86.6   │  55.5    │   83.1      │
│ DeiT-B⚗      │   86.6   │  17.6    │   83.4      │
└──────────────┴──────────┴──────────┴─────────────┘
  ⚗ = with distillation token
  ↑384 = fine-tuned at 384×384 resolution
```

---

## 10. Worked Examples

### Example 1: Patch Embedding Dimensions

**Problem:** Compute all dimensions for ViT-Base processing a 224×224×3 image with 16×16 patches.

```
Step 1: Number of patches
  N = (H/P) × (W/P) = (224/16) × (224/16) = 14 × 14 = 196

Step 2: Each patch flattened
  Patch dim = P × P × C = 16 × 16 × 3 = 768

Step 3: Patch embedding
  E ∈ ℝ^(768 × 768)  → 768 → 768 (D = 768 for ViT-Base)
  Params for patch embed: 768 × 768 = 589,824

Step 4: Add [CLS] token
  Sequence length = N + 1 = 197
  [CLS] token: 768 params

Step 5: Positional embedding
  E_pos ∈ ℝ^(197 × 768)
  Params: 197 × 768 = 151,296

Step 6: Input to transformer
  z₀ ∈ ℝ^(197 × 768)

Step 7: Per transformer layer params
  MSA: W_Q, W_K, W_V each (768 × 768) + W_O (768 × 768) = 4 × 768² = 2,359,296
  MLP: W₁ (768 × 3072) + W₂ (3072 × 768) = 2 × 768 × 3072 = 4,718,592
  LayerNorm: 2 × (768 + 768) = 3,072
  Total per layer ≈ 7.1M params

Step 8: Total params (12 layers)
  Patch embed: 0.59M
  CLS + pos: 0.15M
  Transformer: 12 × 7.1M = 85.2M
  Head: 768 × 1000 = 0.77M
  Total ≈ 86.6M ✓
```

### Example 2: Self-Attention Computation

**Problem:** For a 4-patch image with D=8 and H=2 heads, compute attention weights.

```
Sequence (with CLS): 5 tokens × 8 dims
d_h = D / H = 8 / 2 = 4 per head

For head 1:
  Q = z · W_Q ∈ ℝ^(5×4)
  K = z · W_K ∈ ℝ^(5×4)
  V = z · W_V ∈ ℝ^(5×4)

  Scores = Q · K^T / √4 = Q · K^T / 2

  Example scores (before softmax):
       CLS  p₁   p₂   p₃   p₄
  CLS [2.0  1.5  0.3  0.8  0.5]
  p₁  [0.5  3.0  2.1  0.1  0.2]
  p₂  [0.3  2.0  2.8  1.5  0.3]
  p₃  [0.7  0.2  1.6  2.5  1.8]
  p₄  [0.4  0.1  0.3  1.9  2.4]

  After softmax (row-wise):
       CLS   p₁    p₂    p₃    p₄
  CLS [0.40  0.24  0.07  0.12  0.09]  ← CLS attends mostly to itself & p₁
  p₁  [0.06  0.48  0.31  0.04  0.05]  ← p₁ attends to itself & p₂ (neighbor!)
  ...

  Output = softmax(QK^T/√d) · V → weighted combination of values
```

---

## 11. PyTorch Implementation

### Minimal ViT

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class PatchEmbedding(nn.Module):
    """Split image into patches and embed them."""
    def __init__(self, img_size=224, patch_size=16, in_channels=3, embed_dim=768):
        super().__init__()
        self.num_patches = (img_size // patch_size) ** 2
        # Conv2d acts as patch extractor + linear projection
        self.proj = nn.Conv2d(in_channels, embed_dim,
                              kernel_size=patch_size, stride=patch_size)

    def forward(self, x):
        # x: (B, 3, 224, 224) → (B, D, 14, 14) → (B, 196, D)
        x = self.proj(x)                  # (B, D, H/P, W/P)
        x = x.flatten(2).transpose(1, 2)  # (B, N, D)
        return x


class MultiHeadAttention(nn.Module):
    def __init__(self, embed_dim=768, num_heads=12, dropout=0.0):
        super().__init__()
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        self.scale = self.head_dim ** -0.5

        self.qkv = nn.Linear(embed_dim, embed_dim * 3)
        self.proj = nn.Linear(embed_dim, embed_dim)
        self.attn_drop = nn.Dropout(dropout)
        self.proj_drop = nn.Dropout(dropout)

    def forward(self, x):
        B, N, D = x.shape
        # Compute Q, K, V in one shot
        qkv = self.qkv(x).reshape(B, N, 3, self.num_heads, self.head_dim)
        qkv = qkv.permute(2, 0, 3, 1, 4)  # (3, B, H, N, d_h)
        q, k, v = qkv.unbind(0)

        # Scaled dot-product attention
        attn = (q @ k.transpose(-2, -1)) * self.scale  # (B, H, N, N)
        attn = attn.softmax(dim=-1)
        attn = self.attn_drop(attn)

        # Apply attention to values
        x = (attn @ v).transpose(1, 2).reshape(B, N, D)  # (B, N, D)
        x = self.proj(x)
        x = self.proj_drop(x)
        return x


class TransformerBlock(nn.Module):
    def __init__(self, embed_dim=768, num_heads=12, mlp_ratio=4.0,
                 dropout=0.0, drop_path=0.0):
        super().__init__()
        self.norm1 = nn.LayerNorm(embed_dim)
        self.attn = MultiHeadAttention(embed_dim, num_heads, dropout)
        self.norm2 = nn.LayerNorm(embed_dim)
        mlp_hidden = int(embed_dim * mlp_ratio)
        self.mlp = nn.Sequential(
            nn.Linear(embed_dim, mlp_hidden),
            nn.GELU(),
            nn.Dropout(dropout),
            nn.Linear(mlp_hidden, embed_dim),
            nn.Dropout(dropout),
        )

    def forward(self, x):
        x = x + self.attn(self.norm1(x))
        x = x + self.mlp(self.norm2(x))
        return x


class VisionTransformer(nn.Module):
    def __init__(self, img_size=224, patch_size=16, in_channels=3,
                 num_classes=1000, embed_dim=768, depth=12,
                 num_heads=12, mlp_ratio=4.0, dropout=0.0):
        super().__init__()
        self.patch_embed = PatchEmbedding(img_size, patch_size,
                                          in_channels, embed_dim)
        num_patches = self.patch_embed.num_patches

        self.cls_token = nn.Parameter(torch.zeros(1, 1, embed_dim))
        self.pos_embed = nn.Parameter(
            torch.zeros(1, num_patches + 1, embed_dim))
        self.pos_drop = nn.Dropout(dropout)

        self.blocks = nn.Sequential(*[
            TransformerBlock(embed_dim, num_heads, mlp_ratio, dropout)
            for _ in range(depth)
        ])

        self.norm = nn.LayerNorm(embed_dim)
        self.head = nn.Linear(embed_dim, num_classes)

        # Initialize
        nn.init.trunc_normal_(self.cls_token, std=0.02)
        nn.init.trunc_normal_(self.pos_embed, std=0.02)

    def forward(self, x):
        B = x.shape[0]
        x = self.patch_embed(x)                         # (B, N, D)

        cls = self.cls_token.expand(B, -1, -1)          # (B, 1, D)
        x = torch.cat([cls, x], dim=1)                  # (B, N+1, D)
        x = self.pos_drop(x + self.pos_embed)           # Add position

        x = self.blocks(x)                              # Transformer
        x = self.norm(x)

        cls_output = x[:, 0]                            # [CLS] token
        return self.head(cls_output)                     # (B, num_classes)


# --- Usage ---
model = VisionTransformer(
    img_size=224, patch_size=16, num_classes=10,
    embed_dim=768, depth=12, num_heads=12,
)
x = torch.randn(2, 3, 224, 224)
print(model(x).shape)  # (2, 10)
print(f"Params: {sum(p.numel() for p in model.parameters()):,}")
```

### Using Pretrained ViT

```python
import timm

# ViT variants
vit_b16 = timm.create_model('vit_base_patch16_224', pretrained=True,
                              num_classes=10)
vit_s16 = timm.create_model('vit_small_patch16_224', pretrained=True,
                              num_classes=10)

# DeiT variants
deit_b = timm.create_model('deit_base_patch16_224', pretrained=True,
                             num_classes=10)
deit_s = timm.create_model('deit_small_patch16_224', pretrained=True,
                             num_classes=10)

# With torchvision
from torchvision.models import vit_b_16, ViT_B_16_Weights
model = vit_b_16(weights=ViT_B_16_Weights.IMAGENET1K_V1)
model.heads = nn.Linear(768, 10)
```

---

## 12. Summary Table

| Concept | Key Idea |
|---|---|
| **Core idea** | Treat image as sequence of patches, apply Transformer |
| **Patch embedding** | Split into P×P patches, flatten, linear project to D dims |
| **[CLS] token** | Learnable token prepended; its output used for classification |
| **Positional embedding** | Learnable 1D embeddings; learns 2D structure automatically |
| **Self-attention** | O(N²·D) — every patch attends to every other patch |
| **MLP block** | Two linear layers with GELU: D → 4D → D |
| **ViT-Base** | 12 layers, D=768, 12 heads, 86.6M params |
| **vs CNNs** | ViT needs >10M images; with enough data, ViT wins |
| **DeiT** | Data-efficient ViT with strong augmentation + distillation |
| **Hybrid** | CNN stem + Transformer (CoAtNet) — best of both worlds |

---

## 13. Revision Questions

1. **Explain how ViT converts an image into a sequence.** For a 384×384 image with 16×16 patches, how many patches are there? What are the dimensions of the input to the transformer?

2. **Why does ViT need a [CLS] token?** Could you use global average pooling instead? What are the trade-offs?

3. **Derive the computational complexity** of self-attention in ViT. Why does this become problematic at high resolutions? How does it compare with convolution?

4. **Why do CNNs outperform ViT on small datasets?** List the inductive biases that CNNs have and ViT lacks. At what dataset size does ViT catch up?

5. **Explain DeiT's key innovations** that allow training ViT effectively on ImageNet-1K alone. What is the distillation token and how does it work?

6. **Compare ViT-Base and ResNet-50** in terms of parameters, FLOPs, accuracy, and data requirements. When would you choose each?

---

## 14. Navigation

| Previous | Up | Next |
|---|---|---|
| [← NASNet](./04-nasnet.md) | [Unit 6 Index](../06-Modern-CNN-Architectures/) | [Architecture Search →](./06-architecture-search.md) |

---

> **References:**  
> - Dosovitskiy, A. et al. (2021). *An Image is Worth 16×16 Words: Transformers for Image Recognition at Scale.* ICLR.  
> - Touvron, H. et al. (2021). *Training Data-Efficient Image Transformers & Distillation Through Attention.* ICML.  
> - Dai, Z. et al. (2021). *CoAtNet: Marrying Convolution and Attention for All Data Sizes.* NeurIPS.
