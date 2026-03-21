# 2. Learning Rate Finder

> **Unit 8 · Practical Training** — Leslie Smith's range test to discover the optimal learning rate

---

## Chapter Overview

Choosing the right learning rate is the single most impactful decision in neural network training. Too low and training is painfully slow; too high and the loss diverges. In 2015, Leslie Smith proposed the **Learning Rate Range Test** — a simple, brilliant technique that systematically finds the optimal learning rate in a single short training run. This chapter explains the algorithm, how to read the resulting plot, its connection to the one-cycle policy, and provides complete PyTorch implementations.

---

## 1. The Learning Rate Problem

### Why Learning Rate is So Critical

```
  The learning rate (α) controls step size in parameter space:
  
  θ_{t+1} = θ_t - α · ∇L(θ_t)
                  ↑
            Too small → tiny steps → slow convergence
            Too large → huge steps → overshoot → diverge
            Just right → fast convergence to good minimum
  
  
  Effect of Learning Rate on Training:
  
  Loss ↑
       │
  2.5  │ ●                                    α = 1e-5 (too slow)
       │  ●
  2.0  │   ●●
       │     ●●●●
  1.5  │   ●      ●●●●●●●●●●●●●●──────────   α = 1e-3 (just right!)
       │    ●●●
  1.0  │       ●●●●●
       │            ●●●●●●
  0.5  │                   ●●──────────
       │
  0.0  │   α = 1e-1: ●╱╲╱╲╱╲╱ → NaN!        α = 1e-1 (too high)
       └────────────────────────────────→ Epochs
           0    5   10   15   20   25
```

### Traditional Approach: Trial and Error

```
  Attempt 1: lr=0.1    → Loss explodes        ✗
  Attempt 2: lr=0.01   → Loss decreases slowly ✗
  Attempt 3: lr=0.001  → Seems okay            ?
  Attempt 4: lr=0.003  → Actually better!      ✓
  Attempt 5: lr=0.005  → Slightly worse         
  
  → Hours/days wasted on manual experiments!
  
  Smith's LR Range Test: Find optimal LR in ~1 epoch!
```

---

## 2. Leslie Smith's Learning Rate Range Test

### The Algorithm

```
  ┌──────────────────────────────────────────────────────────────┐
  │  LEARNING RATE RANGE TEST                                    │
  │                                                              │
  │  1. Set LR to a very small value (e.g., 1e-7)              │
  │  2. Train for one mini-batch                                │
  │  3. Record the loss                                          │
  │  4. INCREASE the LR by a multiplicative factor              │
  │  5. Repeat steps 2-4 for several hundred mini-batches       │
  │  6. Plot Loss vs Learning Rate (log scale)                  │
  │  7. Pick LR from the steepest descent region                │
  └──────────────────────────────────────────────────────────────┘
```

### Exponential LR Increase

```
  The LR increases exponentially over N mini-batches:
  
  lr_i = lr_min × (lr_max / lr_min)^(i / N)
  
  Example: lr_min = 1e-7, lr_max = 10, N = 300
  
  lr₀   = 1e-7
  lr₅₀  = 1e-7 × (10/1e-7)^(50/300)  ≈ 4.6e-6
  lr₁₀₀ = 1e-7 × (10/1e-7)^(100/300) ≈ 2.2e-4
  lr₁₅₀ = 1e-7 × (10/1e-7)^(150/300) ≈ 1e-2
  lr₂₀₀ = 1e-7 × (10/1e-7)^(200/300) ≈ 0.46
  lr₃₀₀ = 10
  
  LR ↑ (log scale)
   10 │                              ╱
      │                           ╱
    1 │                        ╱
      │                     ╱
  0.1 │                  ╱
      │               ╱
  1e-3│            ╱           Exponential increase
      │         ╱              on log scale = 
  1e-5│      ╱                 straight line!
      │   ╱
  1e-7│╱
      └──────────────────────────→ Mini-batch iteration
       0    50   100  150  200  250  300
```

---

## 3. Reading the Loss vs LR Plot

### The Ideal Plot

```
  Loss vs Learning Rate (THE KEY PLOT)
  
  Loss ↑
       │
  2.5  │●●●●
       │    ●●
  2.0  │      ●●           Zone A: LR too small
       │        ●●         (loss barely changes)
  1.5  │          ●●
       │     Zone A  ●     Zone B: OPTIMAL RANGE
  1.0  │              ●    (loss drops steeply!)
       │          Zone  ●
  0.5  │            B   ●
       │                 ●●
  0.3  │          minimum  ●     Zone C: LR too large
       │           loss     ●●   (loss starts increasing)
  0.5  │                      ●●
  1.0  │                  Zone  ●●●  
  2.0  │                    C      ●●●●●
       │                               ●●● → diverges!
       └───┬────┬────┬────┬────┬────┬────┬──→ LR (log scale)
         1e-7  1e-6  1e-5  1e-4  1e-3  1e-2  1e-1
  
                          ↑              ↑
                     Best LR         Max LR
                  (steepest          (loss starts
                   descent)          increasing)
```

### How to Pick the Learning Rate

```
  ┌────────────────────────────────────────────────────────────────┐
  │  RULES FOR PICKING THE LEARNING RATE:                         │
  │                                                                │
  │  Method 1 (Conservative — recommended for SGD):               │
  │    Pick LR where loss is DECREASING FASTEST                   │
  │    (steepest downward slope)                                   │
  │    → Typically about 1/10 of the LR where loss is minimum    │
  │                                                                │
  │  Method 2 (For one-cycle policy):                             │
  │    Pick the LR where loss STARTS to decrease as max_lr        │
  │    Pick the LR 10x smaller as min_lr (called div_factor=10)  │
  │                                                                │
  │  Method 3 (Rule of thumb):                                    │
  │    Take the LR at minimum loss                                 │
  │    Divide by 10                                                │
  │    Use that as your learning rate                              │
  │                                                                │
  │  ⚠ NEVER pick the LR at the minimum loss point itself —      │
  │    that's already at the edge of instability!                  │
  └────────────────────────────────────────────────────────────────┘
```

### Example Reading

```
  Given this plot:
  
  Loss minimum at LR = 5e-2
  Steepest descent at LR ≈ 5e-3
  Loss starts decreasing at LR ≈ 1e-4
  
  Recommendations:
  ┌──────────────────────────────────────────────────┐
  │  For SGD:        LR = 5e-3  (steepest descent)  │
  │  For Adam:       LR = 5e-4  (Adam needs lower)  │
  │  For one-cycle:  max_lr = 5e-2, min_lr = 5e-4   │
  └──────────────────────────────────────────────────┘
```

---

## 4. Smoothing the Loss Curve

Raw loss values are noisy. Apply exponential moving average:

```
  Smoothed loss (exponential moving average):
  
  avg_loss₀ = loss₀
  avg_lossᵢ = β × avg_lossᵢ₋₁ + (1 - β) × lossᵢ     (β ≈ 0.98)
  
  Corrected (remove bias toward 0):
  smoothed_lossᵢ = avg_lossᵢ / (1 - βⁱ)
  
  
  Raw Loss ↑                        Smoothed Loss ↑
           │ ●                                     │
           │●●● ●                                  │●●●
           │  ●●●●●                                │   ●●●
           │   ●●  ●●●                             │      ●●●
           │  ●  ●   ●●●                           │         ●●
           │      ●●  ●●●●                         │           ●●
           │    ●   ●● ● ●●                        │             ●●●
           │          ●   ●●●●●                    │                ●●●●●
           └──────────────────→                    └──────────────────────→
           Noisy — hard to read!                   Smooth — clear signal!
```

---

## 5. PyTorch Implementation

### From-Scratch Implementation

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
import matplotlib.pyplot as plt
import math
import copy

class LRFinder:
    """
    Leslie Smith's Learning Rate Range Test.
    
    Usage:
        lr_finder = LRFinder(model, optimizer, criterion, device)
        lr_finder.range_test(train_loader, start_lr=1e-7, end_lr=10, num_iter=300)
        lr_finder.plot()
        lr_finder.reset()  # restore model to original state
    """
    
    def __init__(self, model, optimizer, criterion, device='cpu'):
        self.model = model
        self.optimizer = optimizer
        self.criterion = criterion
        self.device = device
        
        # Save initial state to restore later
        self.model_state = copy.deepcopy(model.state_dict())
        self.optimizer_state = copy.deepcopy(optimizer.state_dict())
        
        # Storage for results
        self.lrs = []
        self.losses = []
    
    def range_test(self, train_loader, start_lr=1e-7, end_lr=10, 
                   num_iter=300, smooth_f=0.05, diverge_th=5):
        """
        Run the LR range test.
        
        Args:
            train_loader: Training data loader
            start_lr: Starting learning rate (very small)
            end_lr: Ending learning rate (very large)
            num_iter: Number of iterations to run
            smooth_f: Smoothing factor for loss (0 = no smoothing)
            diverge_th: Stop if loss exceeds diverge_th × best_loss
        """
        # Compute multiplicative factor for exponential increase
        gamma = (end_lr / start_lr) ** (1 / num_iter)
        
        # Set initial learning rate
        for param_group in self.optimizer.param_groups:
            param_group['lr'] = start_lr
        
        # Create LR scheduler for exponential increase
        scheduler = optim.lr_scheduler.ExponentialLR(self.optimizer, gamma=gamma)
        
        best_loss = float('inf')
        avg_loss = 0.0
        
        self.model.train()
        data_iter = iter(train_loader)
        
        for iteration in range(num_iter):
            # Get next batch (restart if needed)
            try:
                inputs, targets = next(data_iter)
            except StopIteration:
                data_iter = iter(train_loader)
                inputs, targets = next(data_iter)
            
            inputs, targets = inputs.to(self.device), targets.to(self.device)
            
            # Forward pass
            self.optimizer.zero_grad()
            outputs = self.model(inputs)
            loss = self.criterion(outputs, targets)
            
            # Smooth the loss
            if iteration == 0:
                avg_loss = loss.item()
            else:
                avg_loss = smooth_f * loss.item() + (1 - smooth_f) * avg_loss
            
            # Correct for bias in early iterations
            smoothed_loss = avg_loss / (1 - (1 - smooth_f) ** (iteration + 1))
            
            # Record
            current_lr = self.optimizer.param_groups[0]['lr']
            self.lrs.append(current_lr)
            self.losses.append(smoothed_loss)
            
            # Track best loss
            if smoothed_loss < best_loss:
                best_loss = smoothed_loss
            
            # Stop if loss diverges
            if smoothed_loss > diverge_th * best_loss:
                print(f"Stopping early: loss diverged at LR={current_lr:.2e}")
                break
            
            # Backward pass and step
            loss.backward()
            self.optimizer.step()
            scheduler.step()
        
        print(f"LR range test complete. {len(self.lrs)} iterations.")
        print(f"LR range: [{self.lrs[0]:.2e}, {self.lrs[-1]:.2e}]")
    
    def plot(self, skip_start=10, skip_end=5, suggest_lr=True):
        """Plot loss vs learning rate."""
        lrs = self.lrs[skip_start:-skip_end] if skip_end > 0 else self.lrs[skip_start:]
        losses = self.losses[skip_start:-skip_end] if skip_end > 0 else self.losses[skip_start:]
        
        fig, ax = plt.subplots(figsize=(10, 6))
        ax.plot(lrs, losses, linewidth=2)
        ax.set_xscale('log')
        ax.set_xlabel('Learning Rate (log scale)', fontsize=14)
        ax.set_ylabel('Loss', fontsize=14)
        ax.set_title("Leslie Smith's LR Range Test", fontsize=16)
        ax.grid(True, alpha=0.3)
        
        if suggest_lr:
            # Find steepest negative gradient
            min_grad_idx = None
            min_grad = float('inf')
            for i in range(1, len(losses) - 1):
                grad = (losses[i+1] - losses[i-1]) / (
                    math.log10(lrs[i+1]) - math.log10(lrs[i-1])
                )
                if grad < min_grad:
                    min_grad = grad
                    min_grad_idx = i
            
            if min_grad_idx is not None:
                suggested_lr = lrs[min_grad_idx]
                ax.axvline(x=suggested_lr, color='red', linestyle='--', 
                          label=f'Suggested LR: {suggested_lr:.2e}')
                ax.legend(fontsize=12)
                print(f"\nSuggested learning rate: {suggested_lr:.2e}")
        
        plt.tight_layout()
        plt.savefig('lr_range_test.png', dpi=150)
        plt.show()
    
    def reset(self):
        """Restore model and optimizer to initial state."""
        self.model.load_state_dict(self.model_state)
        self.optimizer.load_state_dict(self.optimizer_state)


# ── Usage Example ──
model = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 256),
    nn.ReLU(),
    nn.Dropout(0.3),
    nn.Linear(256, 128),
    nn.ReLU(),
    nn.Dropout(0.3),
    nn.Linear(128, 10)
)

optimizer = optim.SGD(model.parameters(), lr=1e-7, momentum=0.9)
criterion = nn.CrossEntropyLoss()

lr_finder = LRFinder(model, optimizer, criterion, device='cpu')
lr_finder.range_test(train_loader, start_lr=1e-7, end_lr=10, num_iter=300)
lr_finder.plot(suggest_lr=True)

# Restore model to its original untrained state
lr_finder.reset()

# Now set the found LR and train normally
suggested_lr = 3e-3  # from the plot
for param_group in optimizer.param_groups:
    param_group['lr'] = suggested_lr
```

### Using torch-lr-finder Library

```python
# pip install torch-lr-finder

from torch_lr_finder import LRFinder

model = MyModel()
optimizer = optim.Adam(model.parameters(), lr=1e-7)
criterion = nn.CrossEntropyLoss()

lr_finder = LRFinder(model, optimizer, criterion, device="cuda")
lr_finder.range_test(train_loader, end_lr=1, num_iter=300, step_mode="exp")
lr_finder.plot()          # shows the plot
lr_finder.reset()         # restore model
```

---

## 6. Connection to One-Cycle Policy

Leslie Smith also proposed the **One-Cycle Policy** — a learning rate schedule that uses the range test to set bounds.

```
  ONE-CYCLE POLICY (Smith, 2018):
  
  LR ↑
     │
  max│         ╱──────╲
  _lr│        ╱        ╲
     │       ╱          ╲
     │      ╱            ╲
     │     ╱              ╲
  min│    ╱                ╲
  _lr│───╱    Warmup        ╲──────── Anneal
     │   Phase 1          Phase 2    Phase 3
     └──────────────────────────────────────→ Training Progress
     │   30%    │         50%       │  20%  │
     
  Phase 1: Linear warmup from min_lr to max_lr  (~30% of training)
  Phase 2: Linear decay from max_lr to min_lr   (~50% of training)
  Phase 3: Anneal from min_lr to min_lr/100     (~20% of training)
  
  
  Using LR Range Test to set One-Cycle parameters:
  ┌─────────────────────────────────────────────────────────┐
  │  max_lr = LR at steepest descent in range test         │
  │  min_lr = max_lr / div_factor   (div_factor = 25)      │
  │  final_lr = min_lr / div_final  (div_final = 1e4)      │
  └─────────────────────────────────────────────────────────┘
```

### One-Cycle in PyTorch

```python
# After finding max_lr from LR range test
max_lr = 3e-3

model = MyModel().to(device)
optimizer = optim.SGD(model.parameters(), lr=max_lr/25, momentum=0.9, weight_decay=1e-4)

# One-cycle scheduler
scheduler = optim.lr_scheduler.OneCycleLR(
    optimizer,
    max_lr=max_lr,
    epochs=30,
    steps_per_epoch=len(train_loader),
    pct_start=0.3,       # 30% warmup
    div_factor=25,        # initial_lr = max_lr/25
    final_div_factor=1e4, # final_lr = initial_lr/1e4
    anneal_strategy='cos' # cosine annealing
)

# Training loop
for epoch in range(30):
    model.train()
    for batch_idx, (X, y) in enumerate(train_loader):
        X, y = X.to(device), y.to(device)
        optimizer.zero_grad()
        loss = criterion(model(X), y)
        loss.backward()
        optimizer.step()
        scheduler.step()  # step EVERY BATCH, not every epoch!
    
    # Validation
    val_acc = evaluate(model, val_loader)
    print(f"Epoch {epoch+1}: val_acc={val_acc:.4f}, lr={scheduler.get_last_lr()[0]:.6f}")
```

### Why One-Cycle Works

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Benefits of the one-cycle policy:                          │
  │                                                              │
  │  1. SUPER-CONVERGENCE: Trains 5-10x faster than fixed LR!  │
  │     (Smith & Topin, 2017)                                   │
  │                                                              │
  │  2. HIGH LR PHASE acts as regularizer                       │
  │     → Large LR prevents settling in sharp minima            │
  │     → Model finds flatter minima that generalize better     │
  │                                                              │
  │  3. WARMUP prevents early divergence                        │
  │     → Gradients are large and noisy at start                │
  │     → Low LR lets weights stabilize first                   │
  │                                                              │
  │  4. ANNEALING at end allows fine-tuning                     │
  │     → Very low LR polishes the final solution               │
  └──────────────────────────────────────────────────────────────┘
```

---

## 7. Practical Usage Guide

### Complete Workflow

```
  ┌──────────────────────────────────────────────────────────────┐
  │  STEP-BY-STEP WORKFLOW                                       │
  │                                                              │
  │  1. Build your model and data pipeline                      │
  │  2. Verify everything works (overfit a single batch)        │
  │  3. Run LR Range Test:                                       │
  │     a. Set lr_min = 1e-7 (or 1e-8)                          │
  │     b. Set lr_max = 1 (or 10)                               │
  │     c. Run for 200-500 mini-batches                         │
  │     d. Plot loss vs LR                                       │
  │  4. Read the plot:                                           │
  │     a. Find where loss STARTS decreasing                    │
  │     b. Find where loss is MINIMUM                           │
  │     c. Find where loss STARTS increasing                    │
  │     d. Pick LR from steepest descent region                 │
  │  5. Train with found LR (or use one-cycle with it)          │
  │  6. If validation loss plateaus, repeat with narrower range │
  └──────────────────────────────────────────────────────────────┘
```

### Tips and Caveats

```
  ┌──────────────────────────────────────────────────────────────┐
  │  DO:                                                         │
  │  ✓ Always reset model after range test                      │
  │  ✓ Use same batch size you'll use for training              │
  │  ✓ Run on a representative subset of data                   │
  │  ✓ Skip first ~10 and last ~5 points when reading plot      │
  │  ✓ Use exponential increase, not linear                     │
  │  ✓ Re-run test if you change model architecture             │
  │                                                              │
  │  DON'T:                                                      │
  │  ✗ Pick the LR at the minimum loss (too aggressive!)        │
  │  ✗ Use too few iterations (< 100) — plot will be noisy      │
  │  ✗ Use too many iterations (> 1000) — model changes too much│
  │  ✗ Forget to reset model before actual training             │
  │  ✗ Run test with different data augmentation than training   │
  └──────────────────────────────────────────────────────────────┘
```

### Optimizer-Specific Adjustments

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Optimizer         Typical Found LR    Adjustment           │
  │  ───────────────────────────────────────────────────────────│
  │  SGD + Momentum    LR directly          Use as-is           │
  │  Adam              LR / 3 to LR / 10   Adam needs lower    │
  │  AdamW             LR / 3 to LR / 10   Same as Adam        │
  │  RMSProp           LR / 3               Slightly lower      │
  │                                                              │
  │  Why? Adam already adapts per-parameter learning rates,     │
  │  so the effective LR is larger than the set LR.             │
  └──────────────────────────────────────────────────────────────┘
```

---

## 8. Worked Example: Complete Pipeline

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchvision import datasets, transforms
from torch.utils.data import DataLoader

# ── Step 1: Prepare data ──
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.5,), (0.5,))
])
train_data = datasets.FashionMNIST('./data', train=True, download=True, transform=transform)
train_loader = DataLoader(train_data, batch_size=128, shuffle=True)

# ── Step 2: Define model ──
model = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 512), nn.ReLU(), nn.Dropout(0.3),
    nn.Linear(512, 256), nn.ReLU(), nn.Dropout(0.3),
    nn.Linear(256, 10)
)

# ── Step 3: Run LR Range Test ──
optimizer = optim.SGD(model.parameters(), lr=1e-7, momentum=0.9, weight_decay=1e-4)
criterion = nn.CrossEntropyLoss()

lr_finder = LRFinder(model, optimizer, criterion, device='cpu')
lr_finder.range_test(train_loader, start_lr=1e-7, end_lr=10, num_iter=300)
lr_finder.plot(suggest_lr=True)
# Output: Suggested learning rate: 3.16e-02

# ── Step 4: Reset and train with one-cycle ──
lr_finder.reset()

max_lr = 3e-2  # from the plot
optimizer = optim.SGD(model.parameters(), lr=max_lr/25, momentum=0.9, weight_decay=1e-4)
scheduler = optim.lr_scheduler.OneCycleLR(
    optimizer, max_lr=max_lr,
    epochs=20, steps_per_epoch=len(train_loader)
)

for epoch in range(20):
    model.train()
    total_loss = 0
    for X, y in train_loader:
        optimizer.zero_grad()
        loss = criterion(model(X), y)
        loss.backward()
        optimizer.step()
        scheduler.step()
        total_loss += loss.item()
    
    avg_loss = total_loss / len(train_loader)
    lr_now = scheduler.get_last_lr()[0]
    print(f"Epoch {epoch+1:2d} | Loss: {avg_loss:.4f} | LR: {lr_now:.6f}")
```

---

## 9. Summary Table

| Aspect | Details |
|--------|---------|
| **What** | Exponentially increase LR over mini-batches, plot loss vs LR |
| **Who** | Leslie Smith (2015, 2017) |
| **Range** | Start: ~1e-7, End: ~1 to 10 |
| **Duration** | 200-500 mini-batches (partial epoch) |
| **Pick LR at** | Steepest descent (NOT at minimum loss!) |
| **Smoothing** | Exponential moving average (β ≈ 0.98) |
| **Stop condition** | Loss > 5× best loss → diverged |
| **One-cycle max_lr** | LR at steepest descent |
| **One-cycle min_lr** | max_lr / 25 |
| **Key library** | `torch-lr-finder`, `fastai` |

---

## 10. Revision Questions

1. **Describe the Learning Rate Range Test algorithm step by step.** Why does the learning rate increase exponentially rather than linearly?

2. **Given an LR range test plot, how do you pick the optimal learning rate?** Why is it a bad idea to pick the LR at the minimum loss point?

3. **How does the LR range test connect to the one-cycle policy?** What values does the range test provide for configuring one-cycle?

4. **Explain why the one-cycle policy achieves "super-convergence."** What regularization effect does the high LR phase provide?

5. **What smoothing technique is applied to the loss curve, and why?** Write the exponential moving average formula.

6. **After running an LR range test with SGD, you find the optimal LR is 0.01. You decide to switch to Adam. Should you use the same LR? Why or why not?**

---

| [← 01 Hyperparameter Tuning](01-hyperparameter-tuning.md) | [Unit 8 Home](README.md) | [03 → Batch Size Selection](03-batch-size-selection.md) |
|:---|:---:|---:|
