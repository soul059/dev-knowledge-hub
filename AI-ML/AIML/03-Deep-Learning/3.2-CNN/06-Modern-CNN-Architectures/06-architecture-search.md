# Architecture Search — Manual vs Automated CNN Design

> **Unit 6 · Modern CNN Architectures · Topic 6/6**

---

## Table of Contents

1. [Overview](#1-overview)
2. [Manual Architecture Design](#2-manual-architecture-design)
3. [NAS Methods Overview](#3-nas-methods-overview)
4. [RL-Based NAS](#4-rl-based-nas)
5. [Evolutionary NAS](#5-evolutionary-nas)
6. [Gradient-Based NAS (DARTS)](#6-gradient-based-nas-darts)
7. [Search Space Design](#7-search-space-design)
8. [Proxy Tasks & Weight Sharing](#8-proxy-tasks--weight-sharing)
9. [Hardware-Aware NAS](#9-hardware-aware-nas)
10. [Practical Considerations](#10-practical-considerations)
11. [Worked Example](#11-worked-example)
12. [Summary Table](#12-summary-table)
13. [Revision Questions](#13-revision-questions)
14. [Navigation](#14-navigation)

---

## 1. Overview

Architecture design is the process of defining the structure of a neural network: how many layers, what operations, how they connect, and what hyperparameters they use.

```
Evolution of Architecture Design:

1990s-2012:   Hand-crafted (LeNet, AlexNet)
2013-2015:    Deeper manual designs (VGG, Inception, ResNet)
2016-2017:    Design principles emerge (ResNeXt, DenseNet)
2017-2018:    NAS takes over (NASNet, AmoebaNet)
2019-2020:    Efficient NAS (DARTS, ENAS, ProxylessNAS)
2020-present: Automated + human knowledge (EfficientNetV2, ConvNeXt)

┌──────────────────────────────────────────────────────┐
│                                                      │
│  Human Design         Fully Automated      Hybrid   │
│  ←────────────────────────────────────────────────►  │
│                                                      │
│  VGG  ResNet          NASNet  DARTS       EfficientV2│
│  Simple, intuitive    Expensive, optimal   Balanced  │
│  Suboptimal           Impractical          Practical │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 2. Manual Architecture Design

### Key Design Principles (Learned from Human Experience)

```
Principle 1: USE RESIDUAL CONNECTIONS
──────────────────────────────────────
  ResNet showed: depth works only with skip connections
  Rule: Every block should have x + F(x) structure

Principle 2: PROGRESSIVE CHANNEL EXPANSION
──────────────────────────────────────────
  Early layers: few channels, large spatial size
  Later layers: many channels, small spatial size
  Typical: 64 → 128 → 256 → 512 (double at each downsample)

Principle 3: BOTTLENECK DESIGN
──────────────────────────────
  Reduce channels before expensive operations (3×3, 5×5)
  1×1 reduce → 3×3 process → 1×1 expand

Principle 4: BATCH NORMALIZATION EVERYWHERE
────────────────────────────────────────────
  Conv → BN → Activation (standard pattern)

Principle 5: ACTIVATION FUNCTION CHOICE
────────────────────────────────────────
  ReLU (simple, fast) or SiLU/Swish (smoother, better accuracy)

Principle 6: STEM DESIGN
─────────────────────────
  Aggressive downsampling in first layers (stride 2)
  Reduces compute for all subsequent layers

Principle 7: GLOBAL AVERAGE POOLING
────────────────────────────────────
  Replace FC layers with GAP → single FC
  Reduces parameters dramatically
```

### ConvNeXt — Modernized Manual Design

ConvNeXt (Liu et al., 2022) systematically "modernized" ResNet using design insights from transformers and NAS:

```
ResNet-50 → ConvNeXt-T (step by step):

Step 1: Match training recipe     76.1% → 78.8%  (+2.7%)
  • AdamW, cosine decay, Mixup, CutMix, RandAugment
  
Step 2: Macro design changes      78.8% → 79.4%  (+0.6%)
  • Stage ratio (3,4,6,3) → (3,3,9,3) — like Swin Transformer
  • Use "patchify" stem (4×4 stride 4 conv)

Step 3: ResNeXt-ify               79.4% → 80.5%  (+1.1%)
  • Use depthwise separable convolutions
  • Increase width to 96 channels

Step 4: Larger kernel             80.5% → 80.6%  (+0.1%)
  • 3×3 → 7×7 depthwise conv
  • Move depthwise conv to top of block

Step 5: Micro design changes      80.6% → 82.1%  (+1.5%)
  • Replace ReLU with GELU
  • Fewer activations and norms
  • Replace BN with LayerNorm
  • Separate downsampling layers

Final: ConvNeXt-T = 82.1% (vs Swin-T 81.3%)
  → Pure convolution beats Transformer with right design!
```

---

## 3. NAS Methods Overview

```
NAS Taxonomy:
                    NAS Methods
                   ╱     │      ╲
                 ╱       │        ╲
        RL-Based    Evolutionary   Gradient-Based
         │              │              │
     ┌───┴───┐     ┌────┴────┐    ┌────┴────┐
     │ NAS   │     │AmoebaNet│    │ DARTS   │
     │NASNet │     │ CARS    │    │Proxyless│
     │MnasNet│     │ NSGA-Net│    │ FBNet   │
     └───────┘     └─────────┘    └─────────┘
     
     Cost: High      Medium        Low
     Quality: Good   Very Good     Good
     Speed: Slow     Medium        Fast
```

---

## 4. RL-Based NAS

### Mechanism

```
Controller (RNN) → samples architecture → trains child → gets reward
    ↑__________________________________________________|

Controller as Policy Network:

  State:  current partial architecture
  Action: next architecture decision (op type, connections, etc.)
  Reward: validation accuracy (after training child network)
  
  Policy update: REINFORCE
  ∇θJ ≈ (1/K) Σₖ (Rₖ - b) · ∇θ log π(aₖ|θ)
```

### Advantages and Disadvantages

```
✅ Advantages:
  • Can handle complex, discrete search spaces
  • No gradient through architecture (works with non-differentiable metrics)
  • Can incorporate hardware constraints in reward
  
❌ Disadvantages:
  • Extremely expensive (thousands of GPU-days)
  • High variance in REINFORCE gradients
  • Each child network trained independently (wasteful)
  • Reward is delayed and noisy
```

---

## 5. Evolutionary NAS

### Mechanism

```
Evolutionary NAS Algorithm:

1. Initialize POPULATION of random architectures
2. Evaluate fitness (accuracy) of each
3. Repeat until convergence:
   a. SELECT parents (tournament selection)
   b. MUTATE to create offspring
      - Add/remove layer
      - Change operation type
      - Modify connections
      - Change hyperparameters
   c. Train and evaluate offspring
   d. REPLACE weakest in population
   
  Generation 1:              Generation 2:
  ┌──────┐  fitness=72%     ┌──────┐  fitness=75%
  │arch_1│ ──mutate──────►  │arch_5│
  ├──────┤  fitness=68%     ├──────┤  fitness=73%
  │arch_2│ ──mutate──────►  │arch_6│
  ├──────┤  fitness=74%     ├──────┤  fitness=77%
  │arch_3│ ──mutate──────►  │arch_7│
  ├──────┤  fitness=65%     ├──────┤  (removed)
  │arch_4│ ──die────────►   │      │
  └──────┘                  └──────┘
```

### Mutation Operations

```
Mutation Types for CNN Architecture:

1. OPERATION MUTATION:
   Conv 3×3 → Conv 5×5     (change kernel size)
   Conv → DW Sep Conv       (change operation type)
   ReLU → Swish             (change activation)

2. CONNECTION MUTATION:
   Add skip connection
   Remove skip connection
   Change input source

3. STRUCTURAL MUTATION:
   Insert new layer
   Remove existing layer
   Change number of channels

4. HYPERPARAMETER MUTATION:
   Change learning rate
   Change batch size
   Change expansion ratio

Crossover (less common in NAS):
  Parent A: [conv3, conv5, pool, conv3]
  Parent B: [conv5, conv3, conv3, pool]
  Child:    [conv3, conv3, pool, pool]  (swap at random point)
```

### AmoebaNet Results

```
AmoebaNet (Real et al., 2019):
  • Regularized evolution + aging (remove oldest, not worst)
  • 450 GPUs × 7 days
  • Found AmoebaNet-A: 83.9% top-1 on ImageNet
  • Slightly better than NASNet: evolution ≥ RL for NAS
```

---

## 6. Gradient-Based NAS (DARTS)

```
DARTS Key Ideas (recap):

1. CONTINUOUS RELAXATION:
   Discrete choice → softmax-weighted sum
   ō(x) = Σᵢ softmax(αᵢ) · oᵢ(x)

2. BI-LEVEL OPTIMIZATION:
   Outer: min_α L_val(w*(α), α)       ← architecture
   Inner: w*(α) = argmin_w L_train(w, α)  ← weights

3. ADVANTAGES:
   • ~1000× faster than RL/evolution
   • Uses standard gradient descent
   • Single GPU, 1-2 days

4. DISADVANTAGES:
   • Performance gap between search & evaluation
   • Tends to find skip-connection-heavy architectures
   • High memory (all ops active during search)
   • Sensitive to hyperparameters
```

### DARTS Improvements

```
Known Issues and Fixes:

Issue 1: Skip connection dominance
  Fix: Progressive DARTS (P-DARTS) — gradually reduce search space
  Fix: Fair DARTS — add auxiliary loss to balance operation selection

Issue 2: Performance gap
  Fix: PC-DARTS — partial channel connection to reduce memory
  Fix: SNAS — use Gumbel-Softmax for more discrete-like search

Issue 3: High memory
  Fix: ProxylessNAS — binarize paths (one active at a time)
  Fix: PC-DARTS — sample subset of channels

Issue 4: Search instability
  Fix: RobustDARTS — early stopping based on eigenvalue analysis
  Fix: DARTS- — auxiliary skip connection regularization
```

---

## 7. Search Space Design

```
Search Space is CRITICAL — constrains what NAS can find:

Too Large:  Intractable search, wasted computation
Too Small:  May miss good architectures
Just Right: Includes good architectures, excludable to search

Common Search Spaces:

1. CELL-BASED (NASNet, DARTS):
   ┌─────────────────────────────────┐
   │ Search for one cell, stack it   │
   │ + Operations: {conv, pool, id}  │
   │ + Connections: DAG within cell  │
   │ + Fixed: number of cells, stages│
   │ Size: ~10^12-10^18              │
   └─────────────────────────────────┘

2. HIERARCHICAL (Auto-DeepLab):
   ┌─────────────────────────────────┐
   │ Search at multiple levels:      │
   │ + Inner: cell structure         │
   │ + Outer: network-level path     │
   │   (how cells connect, downsamp) │
   │ Size: ~10^20                    │
   └─────────────────────────────────┘

3. ONE-SHOT / SUPERNET (OFA, BigNAS):
   ┌─────────────────────────────────┐
   │ Train ONE super-network         │
   │ Sample sub-networks from it     │
   │ + Kernel size: {3, 5, 7}       │
   │ + Depth: {2, 3, 4} per stage   │
   │ + Width: {0.65, 0.8, 1.0}×     │
   │ Size: ~10^19                    │
   └─────────────────────────────────┘

4. BLOCK-BASED (MnasNet, EfficientNet):
   ┌─────────────────────────────────┐
   │ Search for block configurations │
   │ + Block type: {MBConv, Conv}    │
   │ + Expansion: {1, 3, 6}         │
   │ + Kernel: {3, 5, 7}            │
   │ + SE: {yes, no}                │
   │ + Layers per stage             │
   │ Size: ~10^15                    │
   └─────────────────────────────────┘
```

---

## 8. Proxy Tasks & Weight Sharing

### Proxy Tasks — Cheaper Approximations

```
Full evaluation is too expensive:
  Train each architecture for 300 epochs on ImageNet → weeks per arch

Proxy strategies:
┌──────────────────────┬───────────────────────┬────────────────┐
│ Proxy                │ Method                │ Speedup        │
├──────────────────────┼───────────────────────┼────────────────┤
│ Fewer epochs         │ Train 20 instead of   │ ~15×           │
│                      │ 300 epochs            │                │
│ Smaller dataset      │ CIFAR-10 instead of   │ ~100×          │
│                      │ ImageNet              │                │
│ Smaller model        │ Fewer channels/layers │ ~10-50×        │
│ Lower resolution     │ 32×32 instead of      │ ~50×           │
│                      │ 224×224               │                │
│ Subset of train data │ 10% of ImageNet       │ ~10×           │
└──────────────────────┴───────────────────────┴────────────────┘

Risk: Proxy ranking may differ from full training ranking!
  Architecture A > B on CIFAR-10 doesn't guarantee A > B on ImageNet
```

### Weight Sharing — The Key Speedup

```
Without Weight Sharing:
  Search 1000 architectures × 50 epoch training each = 50,000 GPU-epochs

With Weight Sharing (ENAS, One-Shot):
  Train ONE super-network (containing all architectures)
  Each architecture = subgraph of super-network → shares weights
  
  ┌──────────────────────────────────────────────┐
  │           SUPER-NETWORK                       │
  │                                               │
  │  ┌───┐   ┌───┐   ┌───┐   ┌───┐             │
  │  │3×3│───│5×5│───│3×3│───│5×5│  ← all ops   │
  │  └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘             │
  │    │  ╲   ╱  │  ╲   ╱  │                     │
  │  ┌─┴─┐ ╲╱ ┌─┴─┐ ╲╱ ┌─┴─┐                   │
  │  │pool│ ╱╲ │id │ ╱╲ │pool│  ← all ops        │
  │  └───┘╱  ╲└───┘╱  ╲└───┘                    │
  │       ╱    ╲  ╱    ╲                          │
  └──────────────────────────────────────────────┘
  
  Arch A = [3×3 → id → 3×3]     ← subgraph
  Arch B = [5×5 → pool → 5×5]   ← different subgraph
  
  Both use SAME weights → no retraining!
  
  Cost: ~1 GPU-day (vs thousands)
```

### Once-For-All (OFA)

```
OFA Training:
1. Train super-network with progressive shrinking
2. At deployment: extract sub-network for target hardware
3. No retraining needed!

                    Super-network
                   ╱      │       ╲
                 ╱        │         ╲
         ┌──────┐    ┌────┴────┐   ┌───────┐
         │ Phone│    │ Tablet  │   │ Cloud │
         │ Sub  │    │ Sub     │   │ Sub   │
         │ Net  │    │ Net     │   │ Net   │
         └──────┘    └─────────┘   └───────┘
         2ms latency  5ms latency  GPU-optimized
```

---

## 9. Hardware-Aware NAS

### Why Hardware Matters

```
FLOPs ≠ Latency!

Same FLOPs, different latency:
┌────────────────────────────┬─────────┬──────────┐
│ Operation                  │ FLOPs   │ Latency  │
├────────────────────────────┼─────────┼──────────┤
│ Dense 1×1 conv (256→256)   │  X      │  1.0ms   │
│ Group conv (g=8, 256→256)  │  X/8    │  0.9ms   │  ← Not 8× faster!
│ Depthwise conv (256, 3×3)  │  X/256  │  0.5ms   │  ← Not 256× faster!
└────────────────────────────┴─────────┴──────────┘

Reasons:
  • Memory bandwidth bottleneck
  • Parallelism limitations
  • Cache efficiency
  • Kernel launch overhead
  • Different hardware → different bottlenecks
```

### Hardware-Aware Objective

```
Multi-objective optimization:

maximize  Accuracy(arch)
subject to:
  Latency(arch, device) ≤ T_target
  Memory(arch) ≤ M_target
  Energy(arch, device) ≤ E_target

MnasNet approach (soft constraint):
  Reward = Accuracy(a) × [Latency(a)/T]^w
  
  w = { β  if Latency ≤ T   (small penalty)
      { α  if Latency > T   (large penalty)

ProxylessNAS approach:
  Differentiable latency prediction
  Add latency as regularization term:
  Loss = CE_loss + λ · Σᵢ pᵢ · latency(opᵢ)
  Where pᵢ = softmax(αᵢ) = probability of operation i

Hardware Latency Lookup Tables:
  Pre-measure latency of each operation on target device
  Total latency ≈ Σ latency(each_operation)
  
  ┌───────────────┬─────────┬────────┬──────────┐
  │ Operation     │ GPU(ms) │ CPU(ms)│ Mobile(ms│
  ├───────────────┼─────────┼────────┼──────────┤
  │ Conv3×3(64,64)│  0.12   │  2.3   │   4.5    │
  │ DWSep(64,64)  │  0.08   │  1.1   │   1.8    │
  │ MBConv6(64,64)│  0.35   │  5.2   │   8.7    │
  │ Self-Attn(64) │  0.42   │  12.1  │   25.3   │
  └───────────────┴─────────┴────────┴──────────┘
```

---

## 10. Practical Considerations

### When to Use NAS vs Manual Design

```
Use MANUAL DESIGN when:
  ✓ Limited compute budget
  ✓ Well-understood problem domain
  ✓ Time pressure (need results quickly)
  ✓ Existing architecture works well (just modify)
  ✓ Need interpretable design choices

Use NAS when:
  ✓ Deploying to specific hardware (mobile, edge)
  ✓ Need best possible accuracy within constraints
  ✓ Have significant compute budget
  ✓ Building architecture for repeated use (amortize search cost)
  ✓ Exploring novel search spaces

Practical pipeline:
  1. Start with pretrained model (ResNet, EfficientNet)
  2. Fine-tune for your task
  3. If accuracy insufficient → try larger/different pretrained model
  4. If latency/efficiency is critical → use NAS or known efficient arch
  5. NAS only if nothing else works and you have compute budget
```

### Common Pitfalls

```
1. SEARCH-EVALUATION GAP:
   Architecture found on proxy may not transfer well
   Fix: Use same training recipe for search and final training

2. UNFAIR COMPARISONS:
   NAS models often use better training recipes
   ConvNeXt showed: training recipe matters as much as architecture

3. REPRODUCIBILITY:
   Many NAS methods are sensitive to random seeds
   Fix: Report mean and std over multiple runs

4. COMPUTATIONAL COST:
   Even "efficient" NAS methods cost significant compute
   Fix: Consider weight sharing methods (OFA, BigNAS)

5. OVER-ENGINEERING:
   Complex architectures → harder to understand and debug
   Fix: Use simple, well-tested architectures as baseline
```

---

## 11. Worked Example

### Example: Choosing an Architecture for Mobile Deployment

**Problem:** Deploy an image classifier on an Android phone with <5ms latency and >75% ImageNet top-1 accuracy.

```
Step 1: Identify constraints
  - Latency < 5ms on Snapdragon 888
  - Accuracy > 75% on ImageNet
  - Model size < 20MB (for app size)
  
Step 2: Check known architectures
  ┌──────────────────┬──────┬────────┬──────────┬──────────┐
  │ Model            │Params│MFLOPs  │ Top-1(%) │ ~Latency │
  ├──────────────────┼──────┼────────┼──────────┼──────────┤
  │ MobileNetV3-Small│ 2.9M │  56    │  67.4    │  ~2ms ✓  │
  │ MobileNetV3-Large│ 5.4M │ 219    │  75.2    │  ~5ms ✓  │
  │ EfficientNet-B0  │ 5.3M │ 390    │  77.1    │  ~7ms ✗  │
  │ ShuffleNetV2 1.5×│ 3.5M │ 299    │  72.6    │  ~4ms ✓  │
  │ ShuffleNetV2 2.0×│ 7.4M │ 591    │  74.9    │  ~6ms ✗  │
  └──────────────────┴──────┴────────┴──────────┴──────────┘

Step 3: Best candidate — MobileNetV3-Large
  - 75.2% accuracy ✓ (meets threshold)
  - ~5ms latency ✓ (borderline, need to measure)
  - 5.4M params → ~21MB FP32, ~5MB INT8 ✓

Step 4: Optimization
  - INT8 quantization → 4× size reduction, ~1.5× speed
  - With INT8: ~3ms latency, ~5MB, ~74.8% acc ✓

Step 5: If accuracy insufficient
  - Use hardware-aware NAS (ProxylessNAS, OFA) targeting Snapdragon 888
  - Or fine-tune MobileNetV3-Large on your specific dataset
```

---

## 12. Summary Table

| Concept | Key Idea |
|---|---|
| **Manual design** | Human expert designs architecture using known principles |
| **RL-based NAS** | RNN controller generates architectures; REINFORCE training |
| **Evolutionary NAS** | Population of architectures; mutation + selection |
| **Gradient-based NAS** | DARTS: continuous relaxation, bi-level optimization |
| **Search space** | Cell-based, hierarchical, one-shot, or block-based |
| **Proxy tasks** | Cheaper approximation (fewer epochs, smaller data/model) |
| **Weight sharing** | Train one super-network; extract sub-networks |
| **Hardware-aware NAS** | Include actual latency/energy in objective |
| **ConvNeXt** | Modern manual design ≈ NAS-found architectures |
| **OFA** | Once-for-all: one super-network, many deployment targets |
| **FLOPs ≠ latency** | Memory access, parallelism, and cache affect real speed |

---

## 13. Revision Questions

1. **Compare manual architecture design with NAS.** List the pros and cons of each approach. When would you prefer manual design over NAS?

2. **Describe three NAS search strategies** (RL, evolutionary, gradient-based). For each, explain the mechanism, computational cost, and a specific method.

3. **What is the search space** in NAS? Compare cell-based and hierarchical search spaces. How does search space design affect NAS quality?

4. **Explain weight sharing** in NAS. How does ENAS achieve ~1000× speedup over original NAS? What are the drawbacks of weight sharing?

5. **Why is hardware-aware NAS important?** Give an example where two architectures with the same FLOPs have very different latency. How does ProxylessNAS handle this?

6. **ConvNeXt achieved competitive accuracy with a pure CNN** and no NAS. What design changes from ResNet-50 were most impactful? What does this tell us about the value of NAS?

---

## 14. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Vision Transformer (ViT)](./05-vision-transformer-vit.md) | [Unit 6 Index](../06-Modern-CNN-Architectures/) | [Unit 7: Data Augmentation →](../07-Training-CNNs/01-data-augmentation-images.md) |

---

> **References:**  
> - Liu, Z. et al. (2022). *A ConvNet for the 2020s.* CVPR (ConvNeXt).  
> - Cai, H. et al. (2020). *Once-for-All: Train One Network and Specialize it for Efficient Deployment.* ICLR.  
> - Tan, M. et al. (2019). *MnasNet: Platform-Aware Neural Architecture Search for Mobile.* CVPR.  
> - Real, E. et al. (2019). *Regularized Evolution for Image Classifier Architecture Search.* AAAI.
