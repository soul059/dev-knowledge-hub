# Practical Transformer Training Tricks

> **Navigation:**
> Previous: [← Warmup Schedules](03-warmup-schedules.md) | Next: [Inference Optimization →](05-inference-optimization.md)

---

## Overview

Training Transformers at scale requires a toolkit of engineering tricks that go far beyond simply calling `loss.backward()` and `optimizer.step()`. Modern large language models use **gradient accumulation** to simulate large batch sizes on limited hardware, **mixed precision training** (FP16/BF16) to halve memory usage and double throughput, **gradient checkpointing** to trade compute for memory, **gradient clipping** to prevent exploding gradients, and careful **dropout placement** to regularize effectively. Together these techniques make it possible to train billion-parameter models on hardware that would otherwise be completely insufficient. This chapter covers each trick with the math behind it, when to apply it, and a full practical training loop that combines everything.

---

## 1. Gradient Accumulation

### The Problem

The Transformer paper used an effective batch size of ~25,000 tokens. A single GPU might only fit 4,096 tokens in memory.

### The Solution

Accumulate gradients over multiple micro-batches before calling `optimizer.step()`:

```
Effective batch size = micro_batch_size × accumulation_steps × num_GPUs
```

### How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                    GRADIENT ACCUMULATION                          │
│                                                                  │
│  Step 1: Forward + backward on micro-batch 1  →  accumulate ∇₁  │
│  Step 2: Forward + backward on micro-batch 2  →  accumulate ∇₂  │
│  Step 3: Forward + backward on micro-batch 3  →  accumulate ∇₃  │
│  Step 4: Forward + backward on micro-batch 4  →  accumulate ∇₄  │
│                                                                  │
│  Step 5: Average: ∇ = (∇₁ + ∇₂ + ∇₃ + ∇₄) / 4                 │
│  Step 6: optimizer.step()  →  update weights                     │
│  Step 7: optimizer.zero_grad()                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Mathematical Equivalence

With accumulation steps `A`:

```
∇_accumulated = (1/A) · Σᵢ₌₁ᴬ ∇L(micro_batch_i)
```

This is **mathematically identical** to computing the gradient on one large batch (assuming the loss is averaged within each micro-batch).

### Numerical Example

```
Micro-batch size:       8 sequences
Accumulation steps:     4
Number of GPUs:         2

Effective batch size = 8 × 4 × 2 = 64 sequences per optimizer step
```

---

## 2. Mixed Precision Training (FP16 / BF16)

### Data Types Comparison

```
┌────────────┬──────┬──────────────────┬──────────────────────────┐
│ Type       │ Bits │ Range            │ Precision (decimal)      │
├────────────┼──────┼──────────────────┼──────────────────────────┤
│ FP32       │ 32   │ ±3.4 × 10³⁸     │ ~7 significant digits    │
│ FP16       │ 16   │ ±65,504          │ ~3.3 significant digits  │
│ BF16       │ 16   │ ±3.4 × 10³⁸     │ ~2.4 significant digits  │
│ TF32       │ 19   │ ±3.4 × 10³⁸     │ ~3.3 significant digits  │
└────────────┴──────┴──────────────────┴──────────────────────────┘
```

### How Mixed Precision Works

```
┌─────────────────────────────────────────────────────────────┐
│               MIXED PRECISION TRAINING FLOW                  │
│                                                              │
│  Master Weights (FP32)                                       │
│        │                                                     │
│        ▼                                                     │
│  Cast to FP16/BF16 ──► Forward Pass (FP16) ──► Loss (FP32)  │
│                                                    │         │
│                                              Loss Scaling    │
│                                              (multiply by S) │
│                                                    │         │
│                                                    ▼         │
│                                    Backward Pass (FP16)      │
│                                                    │         │
│                                          Unscale gradients   │
│                                          (divide by S)       │
│                                                    │         │
│                                                    ▼         │
│  Master Weights ◄───── Optimizer Step (FP32) ◄── Grads(FP32) │
│  (FP32 update)                                               │
└─────────────────────────────────────────────────────────────-─┘
```

### Why Loss Scaling?

FP16 can't represent very small gradient values (below ~6×10⁻⁸). Loss scaling multiplies the loss by a large factor (e.g., 1024) before backward, then divides gradients by the same factor after. This keeps small gradients within FP16's representable range.

### Memory Savings

| Component | FP32 | Mixed Precision | Savings |
|-----------|------|-----------------|---------|
| Model weights | 4 bytes/param | 2 bytes (FP16) + 4 bytes (master) | 0% (need both) |
| Activations | 4 bytes/value | 2 bytes/value | **50%** |
| Gradients | 4 bytes/param | 2 bytes/param | **50%** |
| **Net effect** | Baseline | ~60-70% of baseline | **30-40% savings** |

For a 1B parameter model: ~4GB → ~2.5GB for activations alone.

---

## 3. Gradient Checkpointing

### The Memory Problem

Standard backpropagation stores all intermediate activations:

```
Memory for activations ∝ num_layers × batch_size × seq_len × d_model
```

For a 24-layer Transformer with batch=32, seq=2048, d_model=1024:
```
Activations ≈ 24 × 32 × 2048 × 1024 × 4 bytes ≈ 6.4 GB (FP32)
```

### The Trade-off

Gradient checkpointing **discards** intermediate activations during the forward pass and **recomputes** them during backward:

```
Standard:      Store all activations    →  Fast backward, HIGH memory
Checkpointing: Store only checkpoints   →  Slow backward, LOW memory
```

### How It Works

```
┌───────────────────────────────────────────────────────────┐
│          STANDARD vs. GRADIENT CHECKPOINTING               │
│                                                            │
│  Standard (all activations saved):                         │
│  Layer: [1]──[2]──[3]──[4]──[5]──[6]──[7]──[8]           │
│  Saved:  ✓    ✓    ✓    ✓    ✓    ✓    ✓    ✓            │
│  Memory: ████████████████████████████████████  (8 units)   │
│                                                            │
│  Checkpointed (every 4 layers):                            │
│  Layer: [1]──[2]──[3]──[4]──[5]──[6]──[7]──[8]           │
│  Saved:  ✓    ✗    ✗    ✓    ✗    ✗    ✗    ✓            │
│  Memory: ████████████  (3 units)                           │
│  Recompute: layers 2,3 from 1; layers 5,6,7 from 4       │
│                                                            │
│  Memory:  ~√N layers saved (for N layers)                  │
│  Compute: ~33% more FLOPs (one extra forward pass)         │
└───────────────────────────────────────────────────────────-┘
```

### Memory vs. Compute Trade-off

| Strategy | Memory | Compute Overhead |
|----------|--------|-----------------|
| No checkpointing | O(N) | 0% |
| Checkpoint every √N layers | O(√N) | ~33% |
| Checkpoint every layer | O(1) | ~100% |

---

## 4. Large Batch Training

### Why Large Batches Matter

```
Gradient variance ∝ 1 / batch_size
```

Larger batches give more stable gradient estimates, allowing larger learning rates and faster convergence in wall-clock time (when parallelized across GPUs).

### Linear Scaling Rule

When multiplying batch size by `k`:

```
New LR = Base LR × k
New warmup = Base warmup × k  (optional, keeps warmup tokens constant)
```

### Example

```
Base:   batch=256,  lr=0.001
×4:     batch=1024, lr=0.004
×16:    batch=4096, lr=0.016  (with warmup to prevent instability)
```

### Diminishing Returns

```
  Training Loss
       │
       │ *
       │  *
       │    *
       │       *
       │            *
       │                   *  * ── diminishing returns
       │                          * * * * * * ── (very large batches)
       └──────────────────────────────────────────► Batch Size
            256   1K    4K   16K   64K
```

---

## 5. Gradient Clipping

### The Problem: Exploding Gradients

In deep Transformers, gradients can occasionally spike to very large values, causing destructive weight updates.

### The Solution: Clip by Global Norm

```
g = all_gradients_concatenated
global_norm = ‖g‖₂ = √(Σᵢ gᵢ²)

if global_norm > max_norm:
    g = g × (max_norm / global_norm)
```

### Typical Values

- `max_norm = 1.0` (most common for Transformers)
- `max_norm = 0.5` (more aggressive, used for unstable training)
- `max_norm = 5.0` (more permissive)

### ASCII Diagram

```
  Gradient Norm
       │
  5.0  │        *            *
       │       * *          *
  3.0  │      *   *  *     *
       │     *     **  *  *
  1.0  │────*──────────*──*──────── max_norm = 1.0
       │   *             *
  0.0  └──────────────────────────► Step
       Without clipping: norms spike to 5.0+
       With clipping: norms capped at 1.0
```

---

## 6. Dropout in Transformers

### Where Dropout Is Applied

```
┌─────────────────────────────────────────────────────────┐
│              TRANSFORMER LAYER WITH DROPOUT               │
│                                                          │
│  Input x                                                 │
│    │                                                     │
│    ├──► Multi-Head Attention ──► Dropout(p) ──┐          │
│    │                                          │          │
│    └──────────────── + ◄──────────────────────┘ (Add)    │
│                      │                                   │
│                 LayerNorm                                 │
│                      │                                   │
│    ├──► FFN: Linear→ReLU→Dropout(p)→Linear ──┐          │
│    │                                          │          │
│    └──────────────── + ◄──────────────────────┘ (Add)    │
│                      │                                   │
│                 LayerNorm                                 │
│                      │                                   │
│  Output                                                  │
│                                                          │
│  Additional dropout locations:                           │
│  • After attention weights (before V multiply)           │
│  • On input embeddings + positional encoding             │
└──────────────────────────────────────────────────────────-┘
```

### Dropout Rates by Model Size

| Model Size | Dropout Rate | Rationale |
|------------|-------------|-----------|
| Small (< 100M) | 0.1 – 0.3 | More regularization needed |
| Medium (100M – 1B) | 0.1 | Standard |
| Large (1B – 10B) | 0.0 – 0.1 | Enough parameters to memorize |
| Very Large (> 10B) | 0.0 | Data acts as regularizer |

---

## 7. Data Parallelism and Model Parallelism

### Data Parallelism (DP / DDP)

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  GPU 0   │   │  GPU 1   │   │  GPU 2   │   │  GPU 3   │
│ Full     │   │ Full     │   │ Full     │   │ Full     │
│ Model    │   │ Model    │   │ Model    │   │ Model    │
│          │   │          │   │          │   │          │
│ Batch 0  │   │ Batch 1  │   │ Batch 2  │   │ Batch 3  │
└────┬─────┘   └────┬─────┘   └────┬─────┘   └────┬─────┘
     │              │              │              │
     └──────── AllReduce Gradients ───────────────┘
                         │
                  Synchronized Update
```

Each GPU holds a **full copy** of the model but processes a different data batch.

### Model Parallelism (Tensor / Pipeline)

```
  Tensor Parallelism:              Pipeline Parallelism:
  ┌────────┬────────┐              ┌────────┐
  │ GPU 0  │ GPU 1  │              │ GPU 0  │  Layers 1-6
  │ Left   │ Right  │              │        │
  │ half   │ half   │              ├────────┤
  │ of     │ of     │              │ GPU 1  │  Layers 7-12
  │ weight │ weight │              │        │
  │ matrix │ matrix │              ├────────┤
  └────────┴────────┘              │ GPU 2  │  Layers 13-18
                                   │        │
  Split individual layers           ├────────┤
  across GPUs                      │ GPU 3  │  Layers 19-24
                                   └────────┘
                                   Split layers across GPUs
```

### When to Use What

| Strategy | When | Limitation |
|----------|------|------------|
| **Data Parallel** | Model fits on one GPU | Communication overhead |
| **Pipeline Parallel** | Model too large for one GPU | Bubble overhead (idle GPUs) |
| **Tensor Parallel** | Individual layers too large | High communication bandwidth needed |
| **FSDP / ZeRO** | Model states too large | Communication vs. memory trade-off |

---

## 8. Full Practical Training Loop

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.cuda.amp import GradScaler, autocast
import math


def create_transformer_training_loop(
    model: nn.Module,
    train_dataloader,
    num_epochs: int = 10,
    lr: float = 3e-4,
    warmup_steps: int = 2000,
    total_steps: int = 100000,
    gradient_accumulation_steps: int = 4,
    max_grad_norm: float = 1.0,
    use_amp: bool = True,
    label_smoothing: float = 0.1,
    weight_decay: float = 0.01,
    checkpoint_every: int = 1000,
):
    """
    Complete Transformer training loop with all standard tricks:
    - Mixed precision (AMP)
    - Gradient accumulation
    - Gradient clipping
    - Learning rate warmup + cosine decay
    - Label smoothing
    - Weight decay (AdamW)
    """

    # --- Optimizer: AdamW with decoupled weight decay ---
    # Separate parameters that should/shouldn't have weight decay
    no_decay = ["bias", "LayerNorm.weight", "layernorm.weight"]
    param_groups = [
        {
            "params": [p for n, p in model.named_parameters()
                       if not any(nd in n for nd in no_decay) and p.requires_grad],
            "weight_decay": weight_decay,
        },
        {
            "params": [p for n, p in model.named_parameters()
                       if any(nd in n for nd in no_decay) and p.requires_grad],
            "weight_decay": 0.0,
        },
    ]
    optimizer = torch.optim.AdamW(param_groups, lr=lr, betas=(0.9, 0.98), eps=1e-9)

    # --- LR Schedule: Linear warmup + cosine decay ---
    def lr_lambda(step):
        if step < warmup_steps:
            return step / max(warmup_steps, 1)
        progress = (step - warmup_steps) / max(total_steps - warmup_steps, 1)
        return max(0.1, 0.5 * (1 + math.cos(math.pi * progress)))

    scheduler = torch.optim.lr_scheduler.LambdaLR(optimizer, lr_lambda)

    # --- Loss Function: Cross-entropy with label smoothing ---
    criterion = nn.CrossEntropyLoss(label_smoothing=label_smoothing)

    # --- Mixed Precision: GradScaler for FP16 ---
    scaler = GradScaler(enabled=use_amp)

    # --- Gradient Checkpointing (enable on model) ---
    if hasattr(model, 'gradient_checkpointing_enable'):
        model.gradient_checkpointing_enable()

    # --- Training Loop ---
    model.train()
    global_step = 0
    optimizer.zero_grad()

    for epoch in range(num_epochs):
        for batch_idx, batch in enumerate(train_dataloader):
            input_ids = batch["input_ids"].cuda()
            labels = batch["labels"].cuda()

            # ---- Forward pass with mixed precision ----
            with autocast(device_type="cuda", enabled=use_amp):
                logits = model(input_ids)  # (B, T, vocab_size)
                # Reshape for cross-entropy: (B*T, vocab_size) vs (B*T,)
                loss = criterion(
                    logits.view(-1, logits.size(-1)),
                    labels.view(-1)
                )
                # Scale loss for gradient accumulation
                loss = loss / gradient_accumulation_steps

            # ---- Backward pass with loss scaling ----
            scaler.scale(loss).backward()

            # ---- Optimizer step every N accumulation steps ----
            if (batch_idx + 1) % gradient_accumulation_steps == 0:
                # Unscale gradients for clipping
                scaler.unscale_(optimizer)

                # Gradient clipping
                grad_norm = torch.nn.utils.clip_grad_norm_(
                    model.parameters(), max_grad_norm
                )

                # Optimizer step (with scaler for mixed precision)
                scaler.step(optimizer)
                scaler.update()

                # LR scheduler step
                scheduler.step()

                # Zero gradients for next accumulation cycle
                optimizer.zero_grad()

                global_step += 1

                # ---- Logging ----
                if global_step % 100 == 0:
                    current_lr = scheduler.get_last_lr()[0]
                    print(
                        f"Step {global_step:>6d} | "
                        f"Loss: {loss.item() * gradient_accumulation_steps:.4f} | "
                        f"Grad Norm: {grad_norm:.4f} | "
                        f"LR: {current_lr:.2e}"
                    )

                # ---- Save checkpoint ----
                if global_step % checkpoint_every == 0:
                    torch.save({
                        'step': global_step,
                        'model_state_dict': model.state_dict(),
                        'optimizer_state_dict': optimizer.state_dict(),
                        'scheduler_state_dict': scheduler.state_dict(),
                        'scaler_state_dict': scaler.state_dict(),
                        'loss': loss.item(),
                    }, f"checkpoint_step_{global_step}.pt")

                if global_step >= total_steps:
                    return model

    return model


# --- Utility: Resume from checkpoint ---
def load_checkpoint(model, optimizer, scheduler, scaler, checkpoint_path):
    """Resume training from a saved checkpoint."""
    checkpoint = torch.load(checkpoint_path)
    model.load_state_dict(checkpoint['model_state_dict'])
    optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
    scheduler.load_state_dict(checkpoint['scheduler_state_dict'])
    scaler.load_state_dict(checkpoint['scaler_state_dict'])
    return checkpoint['step']
```

---

## 9. Real-World Applications

| Trick | Where Used | Impact |
|-------|-----------|--------|
| **Gradient Accumulation** | All LLM training (GPT, LLaMA) | Enables large batch sizes on limited hardware |
| **Mixed Precision (BF16)** | TPU/GPU training (all modern models) | 2× throughput, 30-40% memory savings |
| **Gradient Checkpointing** | Training models >1B params | Reduces activation memory by ~60-70% |
| **Gradient Clipping** | All Transformer training | Prevents training divergence from gradient spikes |
| **AdamW + Weight Decay** | Standard for all Transformers | Better generalization than Adam + L2 |
| **Pipeline Parallelism** | GPT-3, Megatron-LM, PaLM | Distributes 175B+ models across 100s of GPUs |
| **FSDP / ZeRO** | DeepSpeed, PyTorch FSDP | Reduces per-GPU memory for optimizer states |

---

## 10. Summary Table

| Trick | Purpose | Typical Setting | Memory Impact | Compute Impact |
|-------|---------|-----------------|---------------|----------------|
| Gradient Accumulation | Large effective batch | 4-32 steps | None | None |
| Mixed Precision (BF16) | Speed + memory | Always on | -30-40% | +50-100% throughput |
| Gradient Checkpointing | Memory savings | When OOM | -60-70% activations | +33% compute |
| Gradient Clipping | Training stability | max_norm=1.0 | None | Negligible |
| Dropout | Regularization | 0.0–0.1 | None | Negligible |
| Weight Decay (AdamW) | Regularization | 0.01–0.1 | None | Negligible |
| Large Batch + Scaling | Faster convergence | Linear scaling | More GPU memory | None per step |
| Data Parallelism | Multi-GPU training | DDP / FSDP | Model on each GPU | Communication overhead |

---

## 11. Revision Questions

1. **You have a GPU with 24GB memory and a model that requires 20GB for a batch size of 8. How would you train with an effective batch size of 64?**
   *Hint: Calculate the number of gradient accumulation steps needed.*

2. **Explain why BF16 is generally preferred over FP16 for Transformer training. What specific problem does BF16's larger exponent range solve?**
   *Hint: Consider what happens when gradient values are very small in FP16.*

3. **If gradient checkpointing adds ~33% compute overhead, under what circumstances is it NOT worth using?**
   *Hint: Think about GPU memory vs. GPU compute utilization.*

4. **Why do we exclude bias terms and LayerNorm parameters from weight decay? What would happen if we applied weight decay to these parameters?**
   *Hint: Consider the role of bias and scale parameters in the network.*

5. **A training run shows periodic gradient norm spikes of 10-100× the average. What could cause this, and how does gradient clipping help without hurting convergence?**
   *Hint: Think about data quality, learning rate, and what clipping preserves (gradient direction).*

6. **Compare the memory requirements of training a 7B parameter model with and without gradient checkpointing and mixed precision. Estimate approximate memory savings.**
   *Hint: Consider model params (4 bytes each FP32), optimizer states (8 bytes/param for Adam), activations, and gradients.*

---

> **Navigation:**
> Previous: [← Warmup Schedules](03-warmup-schedules.md) | Next: [Inference Optimization →](05-inference-optimization.md)
