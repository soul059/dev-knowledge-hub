# Learning Rate Warmup and Scheduling

> **Navigation:**
> Previous: [← Label Smoothing](02-label-smoothing.md) | Next: [Training Tricks →](04-training-tricks.md)

---

## Overview

The learning rate schedule is one of the most impactful hyperparameters in Transformer training. The original "Attention Is All You Need" paper introduced a distinctive schedule that **linearly increases** the learning rate during a warmup phase and then **decays** it proportionally to the inverse square root of the step number. This warmup-then-decay pattern has become a foundational principle in training deep models with the Adam optimizer. Warmup prevents destructively large updates in early training when the optimizer's moment estimates are unreliable, while the subsequent decay progressively reduces step sizes to allow fine-grained convergence. Modern variations — cosine annealing, linear decay, and warmup-stable-decay (WSD) — have since been developed, but the core insight that Transformers need careful learning rate scheduling remains universally applied.

---

## 1. The Original Transformer Learning Rate Schedule

### The Formula

From Vaswani et al. (2017):

```
lr(step) = d_model^(-0.5) · min(step^(-0.5), step · warmup_steps^(-1.5))
```

### Breaking It Down

The schedule has two phases determined by the `min()` function:

**Phase 1: Linear Warmup** (when `step < warmup_steps`)
```
step · warmup_steps^(-1.5)  <  step^(-0.5)

lr(step) = d_model^(-0.5) · step · warmup_steps^(-1.5)
         = step / (d_model^(0.5) · warmup_steps^(1.5))
```
This is **linear in step** — the LR increases proportionally from 0.

**Phase 2: Inverse Square Root Decay** (when `step ≥ warmup_steps`)
```
step^(-0.5)  <  step · warmup_steps^(-1.5)

lr(step) = d_model^(-0.5) · step^(-0.5)
         = 1 / (d_model^(0.5) · step^(0.5))
```
This **decays** as `1/√step`.

### Peak Learning Rate

The two phases meet at `step = warmup_steps`:

```
lr_peak = d_model^(-0.5) · warmup_steps^(-0.5)
        = 1 / sqrt(d_model · warmup_steps)
```

For the original Transformer: `d_model = 512`, `warmup_steps = 4000`:

```
lr_peak = 1 / sqrt(512 · 4000) = 1 / sqrt(2,048,000) ≈ 6.99 × 10⁻⁴
```

---

## 2. Worked Numerical Example

### Parameters

- `d_model = 512` → `d_model^(-0.5) = 1/√512 ≈ 0.04419`
- `warmup_steps = 4000` → `warmup_steps^(-1.5) = 1/4000^1.5 ≈ 3.953 × 10⁻⁶`

### Computing LR at Various Steps

| Step | Phase | Calculation | Learning Rate |
|------|-------|-------------|---------------|
| 1 | Warmup | 0.04419 · 1 · 3.953×10⁻⁶ | **1.75 × 10⁻⁷** |
| 100 | Warmup | 0.04419 · 100 · 3.953×10⁻⁶ | **1.75 × 10⁻⁵** |
| 1000 | Warmup | 0.04419 · 1000 · 3.953×10⁻⁶ | **1.75 × 10⁻⁴** |
| 2000 | Warmup | 0.04419 · 2000 · 3.953×10⁻⁶ | **3.49 × 10⁻⁴** |
| 4000 | Peak | 0.04419 · 4000 · 3.953×10⁻⁶ | **6.99 × 10⁻⁴** |
| 8000 | Decay | 0.04419 · 8000⁻⁰·⁵ | **4.94 × 10⁻⁴** |
| 16000 | Decay | 0.04419 · 16000⁻⁰·⁵ | **3.49 × 10⁻⁴** |
| 100000 | Decay | 0.04419 · 100000⁻⁰·⁵ | **1.40 × 10⁻⁴** |

---

## 3. ASCII Visualization of Learning Rate Schedule

```
  Learning Rate
  ×10⁻⁴
   7.0 │                    *  (peak at step 4000)
       │                  *  *
   6.0 │                *      *
       │              *          *
   5.0 │            *              *
       │          *                  *
   4.0 │        *                      *
       │      *                          *
   3.0 │    *                               *
       │   *                                   *
   2.0 │  *                                       *
       │ *                                            *
   1.0 │*                                                  *
       │                                                        *
   0.0 └──────────┬──────────┬──────────┬──────────┬──────────────►
                 4000       8000      16000      40000         Step
       │← Warmup →│←──────── Inverse √ Decay ──────────────►│

  Phase 1: Linear warmup          Phase 2: 1/√step decay
  lr ∝ step                       lr ∝ 1/√step
```

---

## 4. Why Warmup Is Needed

### Problem Without Warmup

The Adam optimizer maintains running estimates of:
- **First moment** (mean of gradients): `m_t = β₁·m_{t-1} + (1-β₁)·g_t`
- **Second moment** (mean of squared gradients): `v_t = β₂·v_{t-1} + (1-β₂)·g_t²`

At step 1, these are initialized to zero and estimated from a single batch:

```
Step 1:  m₁ = (1 - 0.9) · g₁ = 0.1·g₁     (very noisy estimate of mean)
         v₁ = (1 - 0.999) · g₁² = 0.001·g₁²  (very noisy estimate of variance)
```

With bias correction:
```
m̂₁ = m₁ / (1 - β₁¹) = 0.1·g₁ / 0.1 = g₁
v̂₁ = v₁ / (1 - β₂¹) = 0.001·g₁² / 0.001 = g₁²

Update = lr · g₁ / √(g₁²) = lr · sign(g₁)
```

The update magnitude equals `lr` regardless of gradient magnitude — a large learning rate causes **destructive updates** based on unreliable statistics.

### How Warmup Solves This

```
Step     LR Factor    Adam Moment Quality    Net Update Quality
────     ─────────    ──────────────────     ──────────────────
1        0.00025      Very poor              Very small (safe)
100      0.025        Poor                   Small
1000     0.25         Moderate               Moderate
4000     1.00         Good                   Full (safe)
```

By the time the LR reaches its peak, Adam has accumulated enough gradient statistics for reliable updates.

### Additional Benefits

| Benefit | Explanation |
|---------|-------------|
| **Stable loss landscape exploration** | Small steps early prevent falling into sharp minima |
| **Gradient norm stabilization** | Gradients often spike early; warmup reduces their impact |
| **Layer norm initialization** | LayerNorm parameters need time to calibrate |
| **Attention pattern formation** | Random attention → structured attention requires gentle updates |

---

## 5. Modern Learning Rate Schedules

### 5.1 Cosine Annealing

After warmup, decay the LR following a half-cosine curve to zero (or a minimum LR):

```
lr(step) = lr_min + 0.5 · (lr_max - lr_min) · (1 + cos(π · (step - warmup_steps) / (total_steps - warmup_steps)))
```

```
  LR
  lr_max │       ************
         │     **            ****
         │   **                  ***
         │  *                       ***
         │ *                           **
         │*                              **
  lr_min │                                 **************
         └────────────────────────────────────────────────► Step
         │warmup│        cosine decay                    │
```

**Advantages:** Smooth decay, spends more time near low LR, commonly used in GPT-3, LLaMA, etc.

### 5.2 Linear Decay

After warmup, linearly decrease LR to zero:

```
lr(step) = lr_max · (1 - (step - warmup_steps) / (total_steps - warmup_steps))
```

```
  LR
  lr_max │       *
         │      * *
         │     *   *
         │    *     *
         │   *       *
         │  *         *
         │ *           *
  0      │*             *
         └──────────────────► Step
         │warm│ linear decay│
```

### 5.3 Warmup-Stable-Decay (WSD)

A three-phase schedule used in modern LLM training (MiniCPM, etc.):

```
  LR
  lr_max │       ************************
         │      *                        *
         │     *                          *
         │    *                            *
         │   *                              *
         │  *                                *
  0      │*                                   *
         └────────────────────────────────────────► Step
         │warm│       stable              │decay│
```

Phases: (1) Linear warmup, (2) Constant LR, (3) Rapid decay (cosine or linear).

### Comparison Table

| Schedule | Warmup | Decay Shape | Total Steps Needed? | Used In |
|----------|--------|-------------|---------------------|---------|
| Original Transformer | Linear | Inverse √ | No | Transformer (2017) |
| Cosine Annealing | Linear | Cosine to 0 | Yes | GPT-3, LLaMA, ViT |
| Linear Decay | Linear | Linear to 0 | Yes | BERT, RoBERTa |
| WSD | Linear | Stable + rapid | Yes | MiniCPM |
| Constant + Decay | None | Step decay | No | Older CNNs |

---

## 6. PyTorch Implementation

```python
import torch
import torch.optim as optim
import math


class TransformerLRScheduler(torch.optim.lr_scheduler._LRScheduler):
    """
    Original Transformer learning rate schedule:
    lr = d_model^(-0.5) * min(step^(-0.5), step * warmup_steps^(-1.5))
    """

    def __init__(self, optimizer, d_model: int, warmup_steps: int = 4000, last_epoch: int = -1):
        self.d_model = d_model
        self.warmup_steps = warmup_steps
        super().__init__(optimizer, last_epoch)

    def get_lr(self):
        step = max(self.last_epoch, 1)  # Avoid step=0 division

        # Original formula from "Attention Is All You Need"
        scale = self.d_model ** (-0.5) * min(
            step ** (-0.5),
            step * self.warmup_steps ** (-1.5)
        )
        return [scale for _ in self.base_lrs]


class CosineAnnealingWarmup(torch.optim.lr_scheduler._LRScheduler):
    """
    Linear warmup followed by cosine annealing to lr_min.
    Used in GPT-3, LLaMA, and most modern LLMs.
    """

    def __init__(self, optimizer, warmup_steps: int, total_steps: int,
                 lr_max: float = 1e-3, lr_min: float = 1e-5, last_epoch: int = -1):
        self.warmup_steps = warmup_steps
        self.total_steps = total_steps
        self.lr_max = lr_max
        self.lr_min = lr_min
        super().__init__(optimizer, last_epoch)

    def get_lr(self):
        step = self.last_epoch

        if step < self.warmup_steps:
            # Linear warmup
            lr = self.lr_max * step / max(self.warmup_steps, 1)
        else:
            # Cosine annealing
            progress = (step - self.warmup_steps) / max(self.total_steps - self.warmup_steps, 1)
            lr = self.lr_min + 0.5 * (self.lr_max - self.lr_min) * (1 + math.cos(math.pi * progress))

        return [lr for _ in self.base_lrs]


# --- Demo ---
if __name__ == "__main__":
    # Create a dummy model and optimizer
    model = torch.nn.Linear(512, 512)
    optimizer = optim.Adam(model.parameters(), lr=1.0, betas=(0.9, 0.98), eps=1e-9)

    # --- Original Transformer Schedule ---
    scheduler = TransformerLRScheduler(optimizer, d_model=512, warmup_steps=4000)

    print("=== Original Transformer Schedule (d_model=512, warmup=4000) ===")
    print(f"{'Step':>8}  {'Learning Rate':>15}")
    print("-" * 26)
    for step in [1, 100, 500, 1000, 2000, 4000, 8000, 16000, 50000, 100000]:
        scheduler.last_epoch = step
        lr = scheduler.get_lr()[0]
        print(f"{step:>8}  {lr:>15.8f}")

    # --- Cosine Annealing Schedule ---
    optimizer2 = optim.Adam(model.parameters(), lr=1.0)
    scheduler2 = CosineAnnealingWarmup(
        optimizer2, warmup_steps=2000, total_steps=100000,
        lr_max=3e-4, lr_min=1e-5
    )

    print("\n=== Cosine Annealing Schedule (warmup=2000, total=100000) ===")
    print(f"{'Step':>8}  {'Learning Rate':>15}")
    print("-" * 26)
    for step in [0, 500, 1000, 2000, 10000, 25000, 50000, 75000, 100000]:
        scheduler2.last_epoch = step
        lr = scheduler2.get_lr()[0]
        print(f"{step:>8}  {lr:>15.8f}")

    # --- Plot with text (no matplotlib needed) ---
    print("\n=== ASCII Plot: Cosine Annealing Schedule ===")
    width = 60
    height = 15
    steps = list(range(0, 100001, 2000))
    lrs = []
    for s in steps:
        scheduler2.last_epoch = s
        lrs.append(scheduler2.get_lr()[0])

    max_lr = max(lrs)
    for row in range(height, -1, -1):
        threshold = max_lr * row / height
        line = f"{threshold:.1e} |"
        for lr in lrs:
            line += "*" if lr >= threshold else " "
        print(line)
    print(" " * 10 + "+" + "-" * len(steps))
    print(" " * 10 + "0" + " " * (len(steps) // 2 - 1) + "steps" + " " * (len(steps) // 2 - 4) + "100k")
```

### Quick Setup with PyTorch Built-in Schedulers

```python
import torch.optim as optim
from torch.optim.lr_scheduler import LambdaLR

def get_transformer_schedule(optimizer, d_model, warmup_steps):
    """One-liner using LambdaLR for the original Transformer schedule."""
    def lr_lambda(step):
        step = max(step, 1)
        return d_model ** (-0.5) * min(step ** (-0.5), step * warmup_steps ** (-1.5))

    return LambdaLR(optimizer, lr_lambda)


def get_cosine_schedule_with_warmup(optimizer, warmup_steps, total_steps, lr_min_ratio=0.1):
    """Cosine schedule with warmup, similar to HuggingFace's implementation."""
    def lr_lambda(step):
        if step < warmup_steps:
            return step / max(warmup_steps, 1)
        progress = (step - warmup_steps) / max(total_steps - warmup_steps, 1)
        return max(lr_min_ratio, 0.5 * (1 + math.cos(math.pi * progress)))

    return LambdaLR(optimizer, lr_lambda)


# Usage:
# optimizer = optim.AdamW(model.parameters(), lr=3e-4, weight_decay=0.01)
# scheduler = get_cosine_schedule_with_warmup(optimizer, warmup_steps=2000, total_steps=100000)
#
# for step in range(total_steps):
#     loss = train_step()
#     loss.backward()
#     optimizer.step()
#     scheduler.step()
#     optimizer.zero_grad()
```

---

## 7. Real-World Applications

| Model / System | Schedule Used | Warmup Steps | Peak LR | Total Steps |
|----------------|---------------|-------------|---------|-------------|
| **Transformer (2017)** | Inverse √ decay | 4,000 | ~7×10⁻⁴ | 300K |
| **BERT-base** | Linear decay | 10,000 | 1×10⁻⁴ | 1M |
| **GPT-3 175B** | Cosine annealing | 375 | 6×10⁻⁵ | 300B tokens |
| **LLaMA 65B** | Cosine annealing | 2,000 | 1.5×10⁻⁴ | 1.4T tokens |
| **ViT-Large** | Cosine annealing | 10,000 | 1×10⁻³ | 300 epochs |
| **T5-11B** | Inverse √ decay | 10,000 | 1×10⁻² | 1M |
| **Chinchilla** | Cosine annealing | — | 1×10⁻⁴ | 1.4T tokens |

### Key Observations

- Modern LLMs almost universally use **cosine annealing with warmup**
- Warmup steps are typically **0.1–1% of total training steps**
- Larger models often use **lower peak learning rates** (inverse relationship)
- The `AdamW` optimizer (with decoupled weight decay) is standard

---

## 8. Summary Table

| Concept | Details |
|---------|---------|
| **Original formula** | `lr = d_model^(-0.5) · min(step^(-0.5), step · warmup_steps^(-1.5))` |
| **Warmup phase** | Linear increase from 0 to peak LR |
| **Decay phase** | Inverse square root: `lr ∝ 1/√step` |
| **Default warmup_steps** | 4,000 (original paper) |
| **Why warmup?** | Adam moments need time to stabilize; prevents destructive early updates |
| **Modern standard** | Cosine annealing with linear warmup |
| **Peak LR (base Transformer)** | `1/√(d_model · warmup_steps)` ≈ 7×10⁻⁴ |
| **Optimizer** | Adam(β₁=0.9, β₂=0.98, ε=10⁻⁹) or AdamW |

---

## 9. Revision Questions

1. **Calculate the peak learning rate for a Transformer with d_model = 1024 and warmup_steps = 8000. At what step does the peak occur?**
   *Hint: Use lr_peak = 1/√(d_model · warmup_steps).*

2. **Why is the warmup phase linear rather than, say, exponential? What would go wrong with an exponential warmup?**
   *Hint: Consider how quickly exponential growth reaches the peak and the sensitivity of early training.*

3. **Explain why cosine annealing has largely replaced inverse square root decay in modern LLM training.**
   *Hint: Think about the total training budget and how much time is spent at various LR levels.*

4. **If you double the batch size during training, how should you adjust the learning rate and warmup steps? Explain the linear scaling rule.**
   *Hint: Larger batches give lower-variance gradients, allowing larger steps.*

5. **What happens if you skip warmup entirely and start with the peak learning rate? Describe the likely training dynamics.**
   *Hint: Consider Adam's moment estimates at step 1 and the effect of large updates on LayerNorm and attention.*

6. **In the WSD (Warmup-Stable-Decay) schedule, why might a long stable phase at peak LR be beneficial for pre-training large models?**
   *Hint: Think about how long it takes for a large model to move from random initialization to a useful region of parameter space.*

---

> **Navigation:**
> Previous: [← Label Smoothing](02-label-smoothing.md) | Next: [Training Tricks →](04-training-tricks.md)
