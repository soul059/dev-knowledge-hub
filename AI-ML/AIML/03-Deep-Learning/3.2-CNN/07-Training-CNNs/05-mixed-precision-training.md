# Mixed Precision Training — FP16 Speed with FP32 Accuracy

> **Unit 7 · Training CNNs · Topic 5/5**

---

## Table of Contents

1. [Overview & Motivation](#1-overview--motivation)
2. [FP16 vs FP32 Numerical Formats](#2-fp16-vs-fp32-numerical-formats)
3. [Speed & Memory Benefits](#3-speed--memory-benefits)
4. [Loss Scaling & Gradient Scaling](#4-loss-scaling--gradient-scaling)
5. [torch.cuda.amp — Automatic Mixed Precision](#5-torchcudaamp--automatic-mixed-precision)
6. [Which Operations Stay FP32](#6-which-operations-stay-fp32)
7. [Practical Speedup Numbers](#7-practical-speedup-numbers)
8. [Worked Examples](#8-worked-examples)
9. [Complete Training Code](#9-complete-training-code)
10. [Summary Table](#10-summary-table)
11. [Revision Questions](#11-revision-questions)
12. [Navigation](#12-navigation)

---

## 1. Overview & Motivation

Mixed precision training uses **FP16 (half precision)** for most operations and **FP32 (full precision)** only where necessary, achieving ~2× speedup and ~50% memory reduction with virtually no accuracy loss.

```
Standard Training (FP32 everywhere):
  Weights: FP32 (32 bits per parameter)
  Activations: FP32
  Gradients: FP32
  → Slow, memory-hungry

Mixed Precision Training:
  Master weights: FP32 (for precision)
  Forward pass:   FP16 (fast compute)    ← 2× faster on Tensor Cores
  Activations:    FP16 (saved memory)    ← 2× less memory
  Gradients:      FP16 (fast backward)
  Weight update:  FP32 (precision)       ← accumulate in FP32
  
  ┌──────────────────────────────────────────────────┐
  │           Mixed Precision Training               │
  │                                                  │
  │  FP32 Master Weights ──copy──► FP16 Weights     │
  │         ▲                         │              │
  │         │                    Forward Pass (FP16) │
  │    Weight Update                  │              │
  │    (FP32)                    Activations (FP16)  │
  │         ▲                         │              │
  │         │                    Loss (FP32)         │
  │    Unscale                        │              │
  │    Gradients                 Loss Scaling        │
  │         ▲                         │              │
  │         │                    Backward (FP16)     │
  │    FP16 Gradients ◄──────── Scaled Gradients     │
  └──────────────────────────────────────────────────┘
```

---

## 2. FP16 vs FP32 Numerical Formats

### IEEE 754 Floating Point

```
FP32 (float32 / single precision):
  ┌─┬──────────┬───────────────────────┐
  │S│ Exponent │       Mantissa         │
  │1│  8 bits  │       23 bits          │  = 32 bits total
  └─┴──────────┴───────────────────────┘
  Range: ±1.18e-38 to ±3.40e+38
  Precision: ~7 decimal digits
  Smallest positive normal: 1.18 × 10⁻³⁸

FP16 (float16 / half precision):
  ┌─┬──────────┬────────────┐
  │S│ Exponent │  Mantissa   │
  │1│  5 bits  │  10 bits    │  = 16 bits total
  └─┴──────────┴────────────┘
  Range: ±6.10e-5 to ±65,504
  Precision: ~3.3 decimal digits
  Smallest positive normal: 6.10 × 10⁻⁵

BF16 (bfloat16 — Google Brain format):
  ┌─┬──────────┬──────────┐
  │S│ Exponent │ Mantissa  │
  │1│  8 bits  │  7 bits   │  = 16 bits total
  └─┴──────────┴──────────┘
  Range: same as FP32 (±1.18e-38 to ±3.40e+38)
  Precision: ~2.4 decimal digits
  Same exponent range as FP32 → NO overflow issues!

Comparison:
┌────────┬───────┬──────────┬───────────┬───────────────────┐
│ Format │ Bits  │ Exponent │ Mantissa  │ Range             │
├────────┼───────┼──────────┼───────────┼───────────────────┤
│ FP32   │  32   │  8 bits  │  23 bits  │ ±3.4 × 10³⁸      │
│ FP16   │  16   │  5 bits  │  10 bits  │ ±65,504           │
│ BF16   │  16   │  8 bits  │   7 bits  │ ±3.4 × 10³⁸      │
│ TF32*  │  19   │  8 bits  │  10 bits  │ ±3.4 × 10³⁸      │
└────────┴───────┴──────────┴───────────┴───────────────────┘
* TF32 is an NVIDIA Ampere format used internally by Tensor Cores
```

### Key Numerical Issues with FP16

```
Problem 1: OVERFLOW
  FP16 max value = 65,504
  Loss values or gradients > 65,504 → Inf
  Solution: Loss scaling (multiply loss before backward, divide after)

Problem 2: UNDERFLOW  
  FP16 smallest subnormal ≈ 5.96 × 10⁻⁸
  Small gradients → rounded to 0 → no learning!
  
  Gradient distribution:
  │
  │     ┌─────┐
  │     │     │
  │ ┌───┤     ├───┐     Many gradients are TINY
  │ │   │     │   │     and underflow to 0 in FP16
  │ │   │     │   │
  ──┴───┴─────┴───┴──► gradient magnitude
  10⁻⁸  10⁻⁵  10⁻²
   ↑
   FP16 underflow zone

  Solution: Loss scaling shifts the distribution RIGHT:
  │
  │              ┌─────┐
  │              │     │
  │          ┌───┤     ├───┐
  │          │   │     │   │
  ───────────┴───┴─────┴───┴──► gradient magnitude
          10⁻⁵  10⁻²  10¹
          Now within FP16 representable range!

Problem 3: PRECISION LOSS
  FP16 has ~3 decimal digits of precision
  Weight updates: w_new = w_old + lr × gradient
  If lr × gradient << w_old, the update is lost!
  
  Example: w = 1024.0 (FP16), update = 0.001
    1024.0 + 0.001 = 1024.0 in FP16! (update lost)
  
  Solution: Keep master weights in FP32 for accumulation
```

---

## 3. Speed & Memory Benefits

### Speed Benefits

```
Tensor Cores (NVIDIA Volta, Turing, Ampere, Hopper):
  FP32: Standard CUDA cores
  FP16: Tensor Cores → matrix multiply in 1 cycle!

  Mixed-Precision Matrix Multiply:
    D[FP32] = A[FP16] × B[FP16] + C[FP32]
    
    Accumulate in FP32 for precision,
    but multiply in FP16 for speed!

Theoretical speedup:
┌──────────────┬──────────────┬────────────────────┐
│ GPU          │ FP32 TFLOPS  │ FP16 Tensor TFLOPS │
├──────────────┼──────────────┼────────────────────┤
│ V100         │    15.7      │     125.0 (8×)     │
│ A100         │    19.5      │     312.0 (16×)    │
│ RTX 3090     │    35.6      │     142.0 (4×)     │
│ RTX 4090     │    82.6      │     330.0 (4×)     │
│ H100         │    67.0      │    1979.0 (30×)    │
└──────────────┴──────────────┴────────────────────┘

Practical speedup (including memory transfers, non-matmul ops):
  Typical: 1.5× to 3× for CNN training
  Best case (compute-bound): 2× to 4×
```

### Memory Benefits

```
Memory savings:
  FP16 = 2 bytes per element (vs 4 bytes for FP32)
  
  Model weights:     50% saving (FP16 copy for forward)
  Activations:       50% saving (stored in FP16 for backward)
  Gradients:         50% saving (computed in FP16)
  
  BUT: Master weights still FP32 (no saving there)
  
  Net memory saving ≈ 40-50% for activations + gradients
  
  Practical impact:
  ┌────────────────────────────────────────────────────┐
  │ ResNet-50 batch size analysis (V100 32GB):        │
  │                                                    │
  │  FP32:  Max batch size ≈ 64                       │
  │  Mixed: Max batch size ≈ 128 (2× larger!)        │
  │                                                    │
  │  Larger batch → better GPU utilization → faster!  │
  └────────────────────────────────────────────────────┘
```

---

## 4. Loss Scaling & Gradient Scaling

### Static Loss Scaling

```
Multiply loss by a large constant before backward():

  scaled_loss = loss × scale_factor     (e.g., scale = 1024)
  scaled_loss.backward()                 (gradients also scaled)
  
  # Unscale gradients before optimizer step
  for p in model.parameters():
      p.grad /= scale_factor
  
  optimizer.step()

This shifts the gradient distribution into FP16's representable range:

  Before scaling:    gradients in [10⁻⁸, 10⁻²] → many underflow
  After ×1024:       gradients in [10⁻⁵, 10¹]   → within range!
  After unscaling:   gradients back to original magnitudes

Problem: Fixed scale factor may be too large (overflow) or too small (underflow)
```

### Dynamic Loss Scaling (GradScaler)

```
Dynamic scaling automatically adjusts the scale factor:

Algorithm:
  1. Start with large scale (e.g., 2^16 = 65536)
  2. Scale loss and backward
  3. Check gradients for Inf/NaN
  4. If Inf/NaN found:
     - Skip this optimizer step (don't update weights)
     - DECREASE scale by factor (e.g., ×0.5)
  5. If no Inf/NaN for N consecutive steps:
     - INCREASE scale by factor (e.g., ×2)
  6. Repeat

Scale factor over training:
  │
  │     ╱╲    ╱╲╱╲
  │    ╱  ╲  ╱     ╲───────────    Stabilizes
  │   ╱    ╲╱
  │  ╱
  │ ╱
  └────────────────────────────── Iterations
  
  Initially fluctuates as it finds the right scale,
  then stabilizes
```

---

## 5. torch.cuda.amp — Automatic Mixed Precision

### Core Components

```python
import torch
from torch.cuda.amp import autocast, GradScaler

# autocast: Automatically casts operations to FP16 where safe
# GradScaler: Handles loss scaling and gradient unscaling

# Basic pattern:
scaler = GradScaler()

for inputs, targets in dataloader:
    optimizer.zero_grad()
    
    # Forward pass in FP16 (where appropriate)
    with autocast(device_type='cuda'):
        outputs = model(inputs)
        loss = criterion(outputs, targets)
    
    # Backward pass with scaled loss
    scaler.scale(loss).backward()
    
    # Unscale gradients and step
    scaler.step(optimizer)
    
    # Update scale factor
    scaler.update()
```

### How autocast Works

```
autocast automatically selects precision for each operation:

with autocast(device_type='cuda'):
    # These run in FP16 (safe and fast):
    y = conv(x)         # Convolutions
    y = linear(x)       # Matrix multiplies
    y = bmm(a, b)       # Batch matrix multiply
    
    # These STAY in FP32 (need precision):
    loss = cross_entropy(y, targets)  # Loss
    y = softmax(x)                    # Softmax
    y = layer_norm(x)                 # LayerNorm
    y = batch_norm(x)                 # BatchNorm

Outside autocast:
    # Everything is back to FP32
    scaler.scale(loss).backward()
    scaler.step(optimizer)
```

### GradScaler Detailed API

```python
scaler = GradScaler(
    init_scale=2**16,        # Initial scale factor (65536)
    growth_factor=2.0,       # Scale increase factor
    backoff_factor=0.5,      # Scale decrease factor
    growth_interval=2000,    # Steps between scale increases
    enabled=True,            # Set False to disable (for debugging)
)

# Full training step:
optimizer.zero_grad()

with autocast(device_type='cuda'):
    output = model(input)
    loss = loss_fn(output, target)

# 1. Scale loss and backward
scaler.scale(loss).backward()

# 2. (Optional) Gradient clipping — must unscale first!
scaler.unscale_(optimizer)
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)

# 3. Step optimizer (skips if gradients contain Inf/NaN)
scaler.step(optimizer)

# 4. Update scale factor
scaler.update()

# Check if step was skipped
# (useful for logging)
```

---

## 6. Which Operations Stay FP32

```
Operations by Precision Category:

FP16-SAFE (autocast to FP16):
┌────────────────────────────────────────────────┐
│ • Convolutions (Conv1d, Conv2d, Conv3d)        │
│ • Linear layers (fully connected)               │
│ • Matrix multiplications (mm, bmm, matmul)     │
│ • Depthwise convolutions                        │
│ • Grouped convolutions                          │
│                                                 │
│ These are COMPUTE-BOUND → benefit most from FP16│
│ Dominated by matrix multiply → Tensor Core accel│
└────────────────────────────────────────────────┘

MUST STAY FP32 (precision-critical):
┌────────────────────────────────────────────────┐
│ • Loss functions (CrossEntropy, MSE, etc.)     │
│ • Softmax                                       │
│ • Log, Exp                                      │
│ • Norms (LayerNorm, BatchNorm, GroupNorm)       │
│ • Sum, Mean (reductions over many elements)    │
│ • Attention score computation (Q·K^T/√d)       │
│                                                 │
│ These involve ACCUMULATION or NORMALIZATION    │
│ → Small errors compound → need FP32 precision  │
└────────────────────────────────────────────────┘

DEPENDS ON CONTEXT:
┌────────────────────────────────────────────────┐
│ • ReLU, GELU — FP16 safe (simple element-wise) │
│ • Dropout — FP16 safe                           │
│ • Residual add — FP16 usually safe              │
│ • Concatenation — FP16 safe                     │
│ • Interpolation — FP32 recommended              │
└────────────────────────────────────────────────┘

BFloat16 note:
  BF16 has same exponent range as FP32
  → NO overflow/underflow issues
  → Loss scaling often NOT needed with BF16
  → Simpler to use but slightly less precise than FP16+scaling
```

---

## 7. Practical Speedup Numbers

```
Real-World Benchmarks (V100, batch size 64):

┌─────────────────┬──────────┬──────────┬─────────┬──────────┐
│ Model           │ FP32(ms) │ AMP(ms)  │ Speedup │ Acc Diff │
├─────────────────┼──────────┼──────────┼─────────┼──────────┤
│ ResNet-50       │   210    │   130    │  1.6×   │  ~0.0%   │
│ ResNet-152      │   560    │   310    │  1.8×   │  ~0.0%   │
│ EfficientNet-B4 │   380    │   200    │  1.9×   │  ~0.0%   │
│ ViT-B/16        │   450    │   210    │  2.1×   │  ~0.0%   │
│ Swin-T          │   320    │   180    │  1.8×   │  ~0.0%   │
└─────────────────┴──────────┴──────────┴─────────┴──────────┘

Memory savings (ResNet-50, V100):
┌────────────────┬────────────┬───────────┐
│                │ FP32       │ AMP       │
├────────────────┼────────────┼───────────┤
│ Model memory   │  100 MB    │  150 MB*  │
│ Batch 64 total │  8.2 GB    │  4.8 GB   │
│ Max batch size │  96        │  192      │
└────────────────┴────────────┴───────────┘
* Model memory slightly higher due to FP32 master copy + FP16 copy

A100 with BF16 (TF32 for internal compute):
  ResNet-50:  2.5× speedup over FP32
  ViT-L:      2.8× speedup over FP32
```

---

## 8. Worked Examples

### Example 1: FP16 Underflow Problem

```
Scenario: Training with weight_decay = 1e-4, lr = 1e-5

Gradient for a parameter:
  g = 2.5 × 10⁻⁷     (very common in deep networks)

In FP32: No problem (representable)
In FP16: 2.5 × 10⁻⁷ < 6.10 × 10⁻⁵ (FP16 min normal)
         → Rounds to 0 → This parameter NEVER updates!

With loss scaling (scale = 2¹⁰ = 1024):
  Scaled gradient = 2.5 × 10⁻⁷ × 1024 = 2.56 × 10⁻⁴
  2.56 × 10⁻⁴ > 6.10 × 10⁻⁵ → Representable in FP16 ✓
  
  After unscaling: gradient restored to 2.5 × 10⁻⁷ in FP32
  Weight update in FP32: w_new = w_old - lr × 2.5 × 10⁻⁷  ✓
```

### Example 2: Memory Savings Calculation

```
ResNet-50 memory analysis:

Parameters: 25.6M
  FP32: 25.6M × 4 bytes = 102.4 MB
  FP16: 25.6M × 2 bytes =  51.2 MB
  Mixed (FP32 master + FP16 copy) = 102.4 + 51.2 = 153.6 MB

Activations for batch_size=64, input=224×224:
  FP32: ~6.5 GB (must store for backward pass)
  FP16: ~3.25 GB (50% savings!)

Gradients:
  FP32: 102.4 MB
  FP16:  51.2 MB

Optimizer states (Adam: m + v per param):
  FP32: 2 × 102.4 = 204.8 MB
  Mixed: Still FP32 for precision = 204.8 MB

Total:
  FP32:  102.4 + 6500 + 102.4 + 204.8 ≈ 6.9 GB
  Mixed: 153.6 + 3250 + 51.2 + 204.8  ≈ 3.7 GB
  
Savings: ~46% → Can nearly DOUBLE batch size!
```

---

## 9. Complete Training Code

```python
import torch
import torch.nn as nn
from torch.cuda.amp import autocast, GradScaler
from torch.utils.data import DataLoader
from torchvision import models, datasets, transforms

# ============================================================
# Setup
# ============================================================
device = torch.device('cuda')
model = models.resnet50(weights=models.ResNet50_Weights.IMAGENET1K_V2)
model.fc = nn.Linear(2048, 10)
model = model.to(device)

criterion = nn.CrossEntropyLoss(label_smoothing=0.1)
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=0.01)

# Mixed precision components
scaler = GradScaler()
use_amp = True

# Data
transform = transforms.Compose([
    transforms.RandomResizedCrop(224),
    transforms.RandomHorizontalFlip(),
    transforms.ToTensor(),
    transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
])

# train_dataset = datasets.ImageFolder('data/train', transform)
# train_loader = DataLoader(train_dataset, batch_size=128, 
#                           shuffle=True, num_workers=4, pin_memory=True)

# ============================================================
# Training Loop with AMP
# ============================================================
num_epochs = 50

for epoch in range(num_epochs):
    model.train()
    running_loss = 0.0
    
    for batch_idx, (images, labels) in enumerate(train_loader):
        images = images.to(device, non_blocking=True)
        labels = labels.to(device, non_blocking=True)
        
        optimizer.zero_grad(set_to_none=True)  # More memory efficient
        
        # Forward pass with automatic mixed precision
        with autocast(device_type='cuda', enabled=use_amp):
            outputs = model(images)
            loss = criterion(outputs, labels)
        
        if use_amp:
            # Backward pass with scaled loss
            scaler.scale(loss).backward()
            
            # Gradient clipping (optional but recommended)
            scaler.unscale_(optimizer)
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            
            # Optimizer step (skipped if Inf/NaN gradients)
            scaler.step(optimizer)
            scaler.update()
        else:
            # Standard FP32 training
            loss.backward()
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            optimizer.step()
        
        running_loss += loss.item()
    
    avg_loss = running_loss / len(train_loader)
    print(f"Epoch {epoch+1}/{num_epochs}, Loss: {avg_loss:.4f}, "
          f"Scale: {scaler.get_scale():.0f}")

# ============================================================
# Inference with AMP (simpler — no GradScaler needed)
# ============================================================
model.eval()
with torch.no_grad():
    with autocast(device_type='cuda'):
        outputs = model(images)
        predictions = outputs.argmax(dim=1)
```

### BFloat16 Training (Ampere+ GPUs)

```python
# BFloat16 is simpler — no loss scaling needed!
with autocast(device_type='cuda', dtype=torch.bfloat16):
    outputs = model(images)
    loss = criterion(outputs, labels)

loss.backward()  # No scaler needed with BF16
optimizer.step()

# Or set globally:
torch.set_float32_matmul_precision('medium')  # Use TF32 for matmuls
```

### Distributed Mixed Precision Training

```python
import torch.distributed as dist
from torch.nn.parallel import DistributedDataParallel as DDP

# AMP works seamlessly with DDP
model = DDP(model, device_ids=[local_rank])
scaler = GradScaler()

for images, labels in train_loader:
    optimizer.zero_grad()
    with autocast(device_type='cuda'):
        outputs = model(images)
        loss = criterion(outputs, labels)
    
    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

---

## 10. Summary Table

| Concept | Key Idea |
|---|---|
| **FP32** | 32-bit float: high precision, slow, memory-hungry |
| **FP16** | 16-bit float: fast on Tensor Cores, risk of overflow/underflow |
| **BF16** | 16-bit with FP32 exponent range: no overflow, less precision |
| **Mixed precision** | FP16 for compute, FP32 for accumulation |
| **Loss scaling** | Multiply loss to shift gradients into FP16 range |
| **GradScaler** | Dynamic loss scaling — auto-adjusts scale factor |
| **autocast** | Automatically chooses FP16 vs FP32 per operation |
| **Speed benefit** | 1.5-3× training speedup on modern GPUs |
| **Memory benefit** | ~50% activation memory savings → 2× larger batch |
| **FP32 operations** | Loss, softmax, normalization, reductions |
| **FP16 operations** | Convolutions, linear layers, matrix multiplies |
| **Master weights** | FP32 copy for precise accumulation of small updates |

---

## 11. Revision Questions

1. **Explain why FP16 alone (without loss scaling) fails** for training deep networks. What are the three main numerical issues? Give a concrete example of gradient underflow.

2. **How does GradScaler work?** Describe the dynamic scaling algorithm. What happens when gradients contain Inf/NaN? Why is dynamic scaling better than static?

3. **Which operations must stay in FP32** during mixed precision training? Why can't softmax or loss functions use FP16?

4. **Calculate the memory savings** for training ViT-Base (86M params) with batch size 32, input 224×224. Compare FP32 vs mixed precision for model, activations, and gradients.

5. **Compare FP16, BF16, and TF32.** What are the trade-offs? Which format needs loss scaling? Which is simplest to use?

6. **Write a complete PyTorch training loop** using AMP with gradient clipping for an EfficientNet-B3 model. Include both the autocast context and GradScaler.

---

## 12. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Learning Rate Policies](./04-learning-rate-policies.md) | [Unit 7 Index](../07-Training-CNNs/) | [Unit 8: Image Classification →](../08-CNN-Applications/01-image-classification.md) |

---

> **References:**  
> - Micikevicius, P. et al. (2018). *Mixed Precision Training.* ICLR.  
> - NVIDIA (2020). *Automatic Mixed Precision for Deep Learning.* (PyTorch AMP documentation)  
> - Kalamkar, D. et al. (2019). *A Study of BFLOAT16 for Deep Learning Training.*
