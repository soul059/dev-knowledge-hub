# NASNet — Neural Architecture Search for CNN Design

> **Unit 6 · Modern CNN Architectures · Topic 4/6**

---

## Table of Contents

1. [Overview & Motivation](#1-overview--motivation)
2. [Neural Architecture Search Concept](#2-neural-architecture-search-concept)
3. [NAS with Reinforcement Learning](#3-nas-with-reinforcement-learning)
4. [NASNet Search Space](#4-nasnet-search-space)
5. [Normal and Reduction Cells](#5-normal-and-reduction-cells)
6. [NAS Computational Cost](#6-nas-computational-cost)
7. [DARTS — Differentiable Architecture Search](#7-darts--differentiable-architecture-search)
8. [EfficientNAS & Beyond](#8-efficientnas--beyond)
9. [Worked Examples](#9-worked-examples)
10. [PyTorch Implementation](#10-pytorch-implementation)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)
13. [Navigation](#13-navigation)

---

## 1. Overview & Motivation

Traditionally, CNN architectures were designed by human experts through trial-and-error. **Neural Architecture Search (NAS)** automates this process using machine learning to **search for optimal architectures**.

```
Manual Design vs NAS:

Manual:  Human intuition → Design → Train → Evaluate → Repeat
         ↑_____________________________________________↓
         Months of expert effort

NAS:     Search Algorithm → Generate Architecture → Train → Evaluate → Update
         ↑_______________________________________________________________↓
         Automated (but computationally expensive)
```

**Key result:** NASNet (Zoph et al., 2018) found architectures that **surpass human-designed CNNs** on ImageNet, achieving 82.7% top-1 accuracy.

---

## 2. Neural Architecture Search Concept

NAS consists of three components:

```
┌─────────────────────────────────────────────────────────────┐
│                  Neural Architecture Search                  │
│                                                              │
│  ┌─────────────┐   ┌──────────────┐   ┌─────────────────┐  │
│  │  SEARCH     │   │  SEARCH      │   │  PERFORMANCE    │  │
│  │  SPACE      │──►│  STRATEGY    │──►│  ESTIMATION     │  │
│  │             │   │              │   │                 │  │
│  │ What archs  │   │ How to       │   │ How to evaluate │  │
│  │ are valid?  │   │ explore?     │   │ each candidate? │  │
│  │             │   │              │   │                 │  │
│  │ - Operations│   │ - RL         │   │ - Full training │  │
│  │ - Connection│   │ - Evolution  │   │ - Proxy tasks   │  │
│  │ - Topology  │   │ - Gradient   │   │ - Weight sharing│  │
│  │             │   │ - Random     │   │ - Predictors    │  │
│  └─────────────┘   └──────────────┘   └─────────────────┘  │
│         ▲                                      │             │
│         └──────────── Feedback ────────────────┘             │
└─────────────────────────────────────────────────────────────┘
```

### Search Space

Defines the set of all possible architectures:

```
Operations available:
  ┌─────────────────────────────┐
  │ • 3×3 convolution           │
  │ • 5×5 convolution           │
  │ • 7×7 convolution           │
  │ • 3×3 depthwise separable   │
  │ • 5×5 depthwise separable   │
  │ • 7×7 depthwise separable   │
  │ • 3×3 dilated convolution   │
  │ • 1×7 then 7×1 convolution  │
  │ • 3×3 average pooling       │
  │ • 3×3 max pooling           │
  │ • Identity (skip connection)│
  │ • Zero (no connection)      │
  └─────────────────────────────┘

Possible architectures ≈ 10^28 (impossibly large for brute force!)
```

---

## 3. NAS with Reinforcement Learning

The original NAS paper (Zoph & Le, 2017) used an **RNN controller** trained with REINFORCE:

```
NAS-RL Framework:

┌───────────────────┐
│  RNN Controller   │  (the "agent")
│  (LSTM)           │
│                   │
│  Generates        │
│  architecture     │
│  description as   │
│  a sequence of    │
│  tokens           │
└────────┬──────────┘
         │  Action: architecture description
         ▼
┌───────────────────┐
│  Build & Train    │  (the "environment")
│  Child Network    │
│                   │
│  Train for T      │
│  epochs on proxy  │
│  dataset          │
└────────┬──────────┘
         │  Reward: validation accuracy
         ▼
┌───────────────────┐
│  Update Controller│  (policy gradient)
│  via REINFORCE     │
│                   │
│  J(θ) = E[R(a)]  │
│  ∇θJ ≈ R·∇θlog(π)│
└───────────────────┘

Controller generates architectures as sequences:

  [filter_h] → [filter_w] → [stride_h] → [stride_w] → [num_filters]
       ↓            ↓           ↓            ↓              ↓
  Layer 1:    3         3        1            1           64
  Layer 2:    5         5        1            1          128
  Layer 3:    3         3        2            2          256
  ...
```

### REINFORCE Update

```
Policy gradient:
  ∇θ J(θ) = E_a~π(a|θ) [R(a) · ∇θ log π(a|θ)]

With baseline b for variance reduction:
  ∇θ J(θ) ≈ (1/K) Σₖ (Rₖ - b) · ∇θ log π(aₖ|θ)

Where:
  θ     = controller parameters
  a     = sampled architecture  
  R(a)  = validation accuracy of architecture a
  π     = controller policy (probability of generating a)
  b     = exponential moving average of past rewards
  K     = number of sampled architectures per batch
```

---

## 4. NASNet Search Space

Instead of searching for the **entire architecture**, NASNet searches for **cells** — modular building blocks that are stacked to form the full network.

```
NASNet Approach: Search for cells, stack into network

Instead of searching this:          Search THIS and stack:
┌──────────────────┐               ┌───────────┐
│ Layer 1: ???     │               │  Normal   │ ← searched cell
│ Layer 2: ???     │               │  Cell     │
│ Layer 3: ???     │               ├───────────┤
│ ...              │               │  Normal   │
│ Layer N: ???     │               │  Cell     │
│                  │               ├───────────┤
│ ~10^28 possible  │               │ Reduction │ ← searched cell (stride=2)
│ architectures    │               │  Cell     │
│                  │               ├───────────┤
│                  │               │  Normal   │
│                  │               │  Cell     │
│                  │               ├───────────┤
│                  │               │  ...      │
└──────────────────┘               └───────────┘
                                   ~10^12 cell configs
                                   (still huge, but tractable)
```

### Cell Search Space

Each cell is a directed acyclic graph (DAG):

```
Cell Structure (B=5 blocks):

For each block i (i=1 to B):
  1. Select input 1:  hᵢ₁ ∈ {h₀, h₁, ..., hᵢ₋₁}  (previous hidden states)
  2. Select input 2:  hᵢ₂ ∈ {h₀, h₁, ..., hᵢ₋₁}
  3. Select op 1:     oᵢ₁ ∈ {identity, 3×3 conv, 5×5 conv, 3×3 pool, ...}
  4. Select op 2:     oᵢ₂ ∈ {identity, 3×3 conv, 5×5 conv, 3×3 pool, ...}
  5. Combine method:  add or concatenate

  Block output: hᵢ = combine(oᵢ₁(hᵢ₁), oᵢ₂(hᵢ₂))

Example Block:
  ┌─────┐         ┌──────────┐
  │ h₀  │────────►│ 5×5 sep  │──┐
  └─────┘         └──────────┘  │
                                 ├── Add ──► h₃
  ┌─────┐         ┌──────────┐  │
  │ h₁  │────────►│ 3×3 pool │──┘
  └─────┘         └──────────┘

Cell output: Concat all unused block outputs
```

---

## 5. Normal and Reduction Cells

NASNet searches for **two types of cells**:

```
Normal Cell:                          Reduction Cell:
  - Preserves spatial dimensions       - Reduces spatial dimensions by 2×
  - stride=1 in all operations         - stride=2 in operations applied
  - Stacked N times per stage           to cell inputs (h₀, h₁)

Full Network Assembly:

  Input (224×224×3)
      │
  ┌───┴───┐
  │ Stem  │  3×3 conv, stride 2 → (112×112×C)
  └───┬───┘
      │
  ┌───┴────────┐ ×N
  │ Normal Cell│     ← (112×112×C)
  └───┬────────┘
      │
  ┌───┴─────────────┐
  │ Reduction Cell  │  ← (112×112) → (56×56×2C)
  └───┬─────────────┘
      │
  ┌───┴────────┐ ×N
  │ Normal Cell│     ← (56×56×2C)
  └───┬────────┘
      │
  ┌───┴─────────────┐
  │ Reduction Cell  │  ← (56×56) → (28×28×4C)
  └───┬─────────────┘
      │
  ┌───┴────────┐ ×N
  │ Normal Cell│     ← (28×28×4C)
  └───┬────────┘
      │
  ┌───┴──────────┐
  │ Global Pool  │
  │ + Softmax    │
  └──────────────┘


Example Discovered Normal Cell (NASNet-A):

  h[i-1] ──────────────── 1×1 conv ──────┐
     │                                     │
     ├── sep 5×5 ──┐                      │
     │              ├─ add ─── h₁         │
     ├── sep 3×3 ──┘                      │
     │                                     │
  h[i-2] ── sep 5×5 ──┐                  │
     │                  ├─ add ─── h₂     │
     ├── sep 3×3 ──────┘                  │
     │                                     │
     ├── avg 3×3 ──┐                      │
     │              ├─ add ─── h₃         │
  h₁ ── identity ─┘                      │
     │                                     │
  h₁ ── sep 3×3 ──┐                      │
     │              ├─ add ─── h₄         │
  h₁ ── identity ─┘                      │
     │                                     │
  h₂ ── sep 3×3 ──┐                      │
     │              ├─ add ─── h₅         │
  h₂ ── avg 3×3 ──┘                      │
                                           │
  Output = concat(h₃, h₄, h₅) ◄──────────┘
  (h₁, h₂ are used internally)
```

---

## 6. NAS Computational Cost

```
Original NAS (2017):
  • Search on CIFAR-10 with small proxy network
  • 800 GPUs × 28 days = 22,400 GPU-days
  • Each architecture: train for 50 epochs
  • ~12,800 architectures evaluated
  • Cost: ~$50,000-$150,000 (depending on GPU type)

NASNet (2018) — Cell-based search:
  • Search on CIFAR-10 for cells only
  • 500 GPUs × 4 days = 2,000 GPU-days
  • Transfer cells to ImageNet
  • ~10× cheaper than original NAS

ENAS (2018) — Efficient NAS:
  • Weight sharing among child networks
  • Single GPU × 0.5 days = 0.5 GPU-days
  • ~1000× cheaper!

DARTS (2019):
  • Gradient-based, no RL
  • Single GPU × 1.5 days = 1.5 GPU-days

Cost Progression:
  22,400 GPU-days ──► 2,000 ──► 1.5 ──► 0.5
  Original NAS       NASNet    DARTS    ENAS
  ($150K)            ($15K)    ($10)    ($5)
```

---

## 7. DARTS — Differentiable Architecture Search

DARTS (Liu et al., 2019) makes NAS differentiable by **relaxing the discrete choice** of operations into a continuous softmax-weighted mixture.

### Continuous Relaxation

```
Discrete NAS:
  ō(x) = oₖ(x)   where k = argmax α    ← not differentiable

DARTS — Continuous relaxation:
  ō(x) = Σᵢ [exp(αᵢ) / Σⱼ exp(αⱼ)] · oᵢ(x)    ← softmax-weighted mix
       = Σᵢ softmax(α)ᵢ · oᵢ(x)

  Where:
    αᵢ = architecture parameter for operation i (learnable!)
    oᵢ = candidate operation (conv, pool, identity, etc.)

  At the end: pick operation with highest αᵢ


Visualization:

  Discrete:                    Continuous (DARTS):
  
  x ─── 3×3 conv ──► y        x ─┬─ 0.05·(3×3 conv)  ─┐
        (chosen)                   ├─ 0.70·(sep 3×3)    ├─ Σ ──► y
                                   ├─ 0.15·(max pool)   │
                                   ├─ 0.08·(identity)   │
                                   └─ 0.02·(zero)      ─┘
                               
                               Weights = softmax(α)
                               After search: pick sep 3×3 (highest weight)
```

### Bi-Level Optimization

```
DARTS optimizes TWO sets of parameters simultaneously:

  min_α  L_val(w*(α), α)           ← outer: architecture params on val set
  s.t.   w*(α) = argmin_w L_train(w, α)  ← inner: network weights on train set

Approximation (one-step):
  1. Update w by one gradient step: w' = w - ξ · ∇_w L_train(w, α)
  2. Update α: α' = α - η · ∇_α L_val(w', α)

This alternates between:
  • Training the network weights (on training set)
  • Updating architecture parameters (on validation set)
  → Prevents overfitting the architecture to training data
```

### DARTS Algorithm

```
Algorithm: DARTS

1. Create mixed operation edges with architecture params α
2. While not converged:
   a. Update weights w by ∇_w L_train(w, α)    — standard SGD
   b. Update architecture α by ∇_α L_val(w, α) — architecture SGD
3. Derive final architecture:
   For each edge: keep operation with largest α
   For each node: keep top-2 strongest input edges
4. Train derived architecture from scratch (full training)

Advantages over RL-based NAS:
  ✓ ~1000× faster (gradient-based, not sampling)
  ✓ No need for a separate controller network
  ✓ Smooth optimization landscape
  
Disadvantages:
  ✗ Search and evaluation architectures may differ
  ✗ Tends to favor skip connections (performance collapse)
  ✗ High GPU memory (all operations active simultaneously)
```

---

## 8. EfficientNAS & Beyond

### ENAS — Efficient Neural Architecture Search

```
Key idea: WEIGHT SHARING among all child architectures

Instead of training each architecture from scratch:
  
  ┌────────────────────────────────┐
  │     SUPER-NETWORK              │
  │  (contains ALL possible ops)   │
  │                                │
  │  ┌──┐   ┌──┐   ┌──┐          │
  │  │3×3├──►│5×5├──►│3×3│         │
  │  └──┘╲  └──┘  ╱└──┘         │
  │       ╲      ╱              │
  │        ╲    ╱               │
  │  ┌──┐  ╲┌──┐╱  ┌──┐        │
  │  │id ├───►│pool├───►│id │    │
  │  └──┘   └──┘   └──┘        │
  └────────────────────────────────┘
  
  Each sampled architecture = a SUBGRAPH of the super-network
  All subgraphs share weights → no retraining!
  
  → 1 GPU × 12 hours instead of 500 GPUs × 4 days
```

### One-Shot NAS Methods

```
Search Strategy Comparison:

┌──────────────┬──────────────┬───────────┬──────────────────┐
│   Method     │ Search Cost  │ Strategy  │ Key Innovation   │
├──────────────┼──────────────┼───────────┼──────────────────┤
│ NAS (2017)   │ 22400 GPU-d  │ RL        │ First NAS paper  │
│ NASNet(2018) │  2000 GPU-d  │ RL        │ Cell-based search│
│ AmoebaNet    │  3150 GPU-d  │Evolution  │ Evolutionary NAS │
│ ENAS (2018)  │  0.5 GPU-d   │ RL+share  │ Weight sharing   │
│ DARTS (2019) │  1.5 GPU-d   │ Gradient  │ Differentiable   │
│ ProxylessNAS │  8.3 GPU-d   │ Gradient  │ Direct on target │
│ FBNet (2019) │  9 GPU-d     │ Gradient  │ HW-aware DARTS   │
│ OFA (2020)   │  ~50 GPU-d   │ Progressive│Once-for-all     │
│ MnasNet      │ ~40000 GPU-h │ RL        │ Platform-aware   │
└──────────────┴──────────────┴───────────┴──────────────────┘
```

### Hardware-Aware NAS

```
Standard NAS objective:
  maximize  Accuracy(arch)

Hardware-aware NAS objective:
  maximize  Accuracy(arch) × [Latency(arch) / Target]^w

MnasNet reward function:
  R = ACC(m) × [LAT(m) / T]^w

  Where:
    w = -0.07  (if LAT(m) ≤ T, small penalty)
    w = -0.07  (if LAT(m) > T, same penalty — soft constraint)

  This finds architectures that maximize accuracy while
  meeting latency targets on actual hardware (e.g., Pixel phone)
```

---

## 9. Worked Examples

### Example 1: DARTS Softmax Computation

**Problem:** Given architecture parameters α = [1.5, 3.0, 0.8, 0.2, -1.0] for operations [sep 3×3, sep 5×5, max pool, identity, zero], compute softmax weights and determine which operation is selected.

```
Step 1: Compute exp(αᵢ):
  exp(1.5) = 4.482
  exp(3.0) = 20.086
  exp(0.8) = 2.226
  exp(0.2) = 1.221
  exp(-1.0) = 0.368

Step 2: Sum = 4.482 + 20.086 + 2.226 + 1.221 + 0.368 = 28.383

Step 3: Softmax weights:
  sep 3×3:   4.482 / 28.383 = 0.158  (15.8%)
  sep 5×5:  20.086 / 28.383 = 0.708  (70.8%)  ← SELECTED
  max pool:  2.226 / 28.383 = 0.078  (7.8%)
  identity:  1.221 / 28.383 = 0.043  (4.3%)
  zero:      0.368 / 28.383 = 0.013  (1.3%)

Step 4: Mixed output = 0.158·sep3(x) + 0.708·sep5(x) + 0.078·pool(x) 
                      + 0.043·x + 0.013·0

After search: Discretize → select sep 5×5 (highest weight)
```

### Example 2: NAS Search Space Size

**Problem:** How many possible cells exist in NASNet's search space with B=5 blocks and 13 operations?

```
For each block i:
  - Choose input 1: i+1 choices (from h₀ to hᵢ₋₁, plus 2 cell inputs)
  - Choose input 2: i+1 choices
  - Choose op 1: 13 choices
  - Choose op 2: 13 choices
  - Combine: 1 choice (add, in practice)

Block 1: 2 × 2 × 13 × 13 = 676
Block 2: 3 × 3 × 13 × 13 = 1,521
Block 3: 4 × 4 × 13 × 13 = 2,704
Block 4: 5 × 5 × 13 × 13 = 4,225
Block 5: 6 × 6 × 13 × 13 = 6,084

Total cells = 676 × 1521 × 2704 × 4225 × 6084
            ≈ 7.1 × 10^16

For two cell types (normal + reduction):
  ≈ (7.1 × 10^16)² ≈ 5 × 10^33

This is why smart search strategies are needed!
```

---

## 10. PyTorch Implementation

### DARTS Mixed Operations

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

# Candidate operations
OPS = {
    'sep_3x3': lambda C: SepConv(C, C, 3, 1),
    'sep_5x5': lambda C: SepConv(C, C, 5, 2),
    'dil_3x3': lambda C: DilConv(C, C, 3, 2, 2),
    'avg_3x3': lambda C: nn.AvgPool2d(3, 1, 1),
    'max_3x3': lambda C: nn.MaxPool2d(3, 1, 1),
    'skip':    lambda C: nn.Identity(),
    'zero':    lambda C: Zero(),
}

class SepConv(nn.Module):
    """Separable convolution: DW + PW (applied twice)."""
    def __init__(self, in_ch, out_ch, kernel, padding):
        super().__init__()
        self.op = nn.Sequential(
            nn.Conv2d(in_ch, in_ch, kernel, 1, padding,
                      groups=in_ch, bias=False),
            nn.Conv2d(in_ch, out_ch, 1, bias=False),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
        )
    def forward(self, x):
        return self.op(x)

class DilConv(nn.Module):
    """Dilated separable convolution."""
    def __init__(self, in_ch, out_ch, kernel, padding, dilation):
        super().__init__()
        self.op = nn.Sequential(
            nn.Conv2d(in_ch, in_ch, kernel, 1, padding,
                      dilation=dilation, groups=in_ch, bias=False),
            nn.Conv2d(in_ch, out_ch, 1, bias=False),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
        )
    def forward(self, x):
        return self.op(x)

class Zero(nn.Module):
    """Zero operation (no connection)."""
    def forward(self, x):
        return x * 0


class MixedOp(nn.Module):
    """DARTS mixed operation — softmax-weighted sum of all ops."""
    def __init__(self, channels):
        super().__init__()
        self.ops = nn.ModuleList([
            OPS[name](channels) for name in OPS
        ])

    def forward(self, x, weights):
        """
        x: input tensor
        weights: softmax(alpha) for this edge
        """
        return sum(w * op(x) for w, op in zip(weights, self.ops))


class DARTSCell(nn.Module):
    """A DARTS cell with mixed operations on each edge."""
    def __init__(self, channels, num_nodes=4):
        super().__init__()
        self.num_nodes = num_nodes
        self.ops = nn.ModuleList()

        # Each node i receives input from all previous nodes (0..i-1)
        # Node 0 and 1 are the two cell inputs
        for i in range(2, num_nodes + 2):
            for j in range(i):
                self.ops.append(MixedOp(channels))

    def forward(self, s0, s1, alphas):
        """
        s0, s1: two input states
        alphas: architecture parameters (list of softmax weights per edge)
        """
        states = [s0, s1]
        offset = 0
        for i in range(self.num_nodes):
            s = sum(
                self.ops[offset + j](states[j], alphas[offset + j])
                for j in range(len(states))
            )
            states.append(s)
            offset += len(states) - 1
        # Concat unused states (last num_nodes states)
        return torch.cat(states[2:], dim=1)


# Architecture parameters
num_edges = sum(i for i in range(2, 6))  # 14 edges for 4 intermediate nodes
num_ops = len(OPS)  # 7 operations
alphas = nn.Parameter(torch.randn(num_edges, num_ops) * 1e-3)

# Softmax weights
weights = F.softmax(alphas, dim=-1)
print(f"Edges: {num_edges}, Ops per edge: {num_ops}")
print(f"Architecture params: {alphas.numel()}")
```

### Using Pretrained NASNet via timm

```python
import timm
import torch

# NASNet variants
model = timm.create_model('nasnetalarge', pretrained=True, num_classes=10)

x = torch.randn(1, 3, 331, 331)  # NASNet-A Large uses 331×331
with torch.no_grad():
    out = model(x)
print(out.shape)  # (1, 10)
print(f"Params: {sum(p.numel() for p in model.parameters()):,}")

# Other NAS-found architectures available in timm
# timm.create_model('pnasnet5large', ...)
# timm.create_model('mnasnet_100', ...)
```

---

## 11. Summary Table

| Concept | Key Idea |
|---|---|
| **NAS** | Automate architecture design using search algorithms |
| **Three components** | Search space, search strategy, performance estimation |
| **NAS-RL** | RNN controller generates architectures, trained with REINFORCE |
| **NASNet** | Search for cells (not whole architectures), transfer across datasets |
| **Normal cell** | Preserves spatial dimensions (stride=1) |
| **Reduction cell** | Halves spatial dimensions (stride=2) |
| **DARTS** | Continuous relaxation — softmax over operations, gradient-based |
| **Bi-level optimization** | Weights on train set, architecture on val set |
| **ENAS** | Weight sharing via super-network; ~1000× cheaper |
| **Hardware-aware NAS** | Include latency/energy in reward function |
| **Cost evolution** | 22,400 GPU-days → 1.5 GPU-days (13,000× reduction) |

---

## 12. Revision Questions

1. **Describe the three components of NAS** (search space, search strategy, performance estimation). Give an example of each.

2. **How does NAS with reinforcement learning work?** What is the controller? What is the reward? Write the REINFORCE gradient update equation.

3. **What is the key insight of NASNet** that makes it more efficient than the original NAS? How do normal and reduction cells differ?

4. **Explain DARTS** — how does it make architecture search differentiable? What is the continuous relaxation? What is bi-level optimization?

5. **Compare RL-based NAS vs DARTS vs ENAS** in terms of search cost, search strategy, and final architecture quality.

6. **What is hardware-aware NAS?** Write MnasNet's reward function. Why is it important to include actual latency rather than just FLOPs?

---

## 13. Navigation

| Previous | Up | Next |
|---|---|---|
| [← ShuffleNet](./03-shufflenet.md) | [Unit 6 Index](../06-Modern-CNN-Architectures/) | [Vision Transformer (ViT) →](./05-vision-transformer-vit.md) |

---

> **References:**  
> - Zoph, B. & Le, Q.V. (2017). *Neural Architecture Search with Reinforcement Learning.* ICLR.  
> - Zoph, B. et al. (2018). *Learning Transferable Architectures for Scalable Image Recognition.* CVPR.  
> - Liu, H. et al. (2019). *DARTS: Differentiable Architecture Search.* ICLR.  
> - Pham, H. et al. (2018). *Efficient Neural Architecture Search via Parameter Sharing.* ICML.
