# 4. Training Monitoring

> **Unit 8 · Practical Training** — Using loss curves, accuracy plots, and logging tools to diagnose training problems

---

## Chapter Overview

Training a neural network without monitoring is like driving blindfolded. **Loss curves**, **accuracy plots**, **gradient norms**, and **weight distributions** tell you *exactly* what's happening inside your network — whether it's learning, overfitting, underfitting, or experiencing numerical instability. This chapter teaches you to read these diagnostic signals, set up **TensorBoard** and **Weights & Biases** for real-time monitoring, and detect common training problems from visual patterns.

---

## 1. What to Monitor During Training

### The Essential Metrics

```
  ┌──────────────────────────────────────────────────────────────┐
  │  MUST MONITOR (every training run):                          │
  │  ─────────────────────────────────                           │
  │  1. Training loss (per batch and per epoch)                 │
  │  2. Validation loss (per epoch)                             │
  │  3. Training accuracy / metric (per epoch)                  │
  │  4. Validation accuracy / metric (per epoch)                │
  │  5. Learning rate (if using scheduler)                      │
  │                                                              │
  │  SHOULD MONITOR (for debugging):                            │
  │  ──────────────────────────────                              │
  │  6. Gradient norms (per layer)                              │
  │  7. Weight norms / distributions (per layer)                │
  │  8. Activation statistics                                    │
  │  9. GPU memory usage                                         │
  │  10. Training throughput (samples/sec)                       │
  │                                                              │
  │  NICE TO HAVE (for publication / analysis):                 │
  │  ──────────────────────────────────                          │
  │  11. Confusion matrix (periodic)                            │
  │  12. Sample predictions (periodic)                          │
  │  13. Learning rate vs loss correlation                      │
  │  14. Per-class accuracy breakdown                           │
  └──────────────────────────────────────────────────────────────┘
```

---

## 2. Training and Validation Loss Curves

### The Most Important Plot

```
  IDEAL TRAINING:
  
  Loss ↑
       │
  2.5  │●
       │●                        Training loss
  2.0  │ ●                       ────────────
       │  ●●                     Validation loss
  1.5  │    ●●                   ─ ─ ─ ─ ─ ─
       │  ○   ●●●
  1.0  │   ○○    ●●●●●
       │     ○○     ●●●●●●●──── converged
  0.5  │       ○○      ○○○○○○── converged
       │         ○○○○○○
  0.2  │              
       └────────────────────────────→ Epochs
        0   5   10  15  20  25  30
  
  ✓ Both curves decrease
  ✓ Small gap between them
  ✓ Both plateau at similar values
```

### Pattern Recognition: What Each Curve Shape Means

```
  ┌──────────────────────────────────────────────────────────────────┐
  │  PATTERN 1: OVERFITTING                                         │
  │                                                                  │
  │  Loss ↑                                                         │
  │       │●                                                        │
  │       │ ●                  val loss increases!                   │
  │       │  ●  ○─────○────○────○                                   │
  │       │   ●○    ╱                                               │
  │       │    ●●  ╱                                                │
  │       │     ●●╱                                                 │
  │       │      ●●●●●●●──── train loss still decreasing           │
  │       └────────────────────→                                    │
  │                                                                  │
  │  Diagnosis: Model memorizes training data                       │
  │  Fix: More data, regularization, dropout, early stopping        │
  ├──────────────────────────────────────────────────────────────────┤
  │  PATTERN 2: UNDERFITTING                                        │
  │                                                                  │
  │  Loss ↑                                                         │
  │       │●                                                        │
  │       │ ●                                                       │
  │       │  ●●                                                     │
  │       │    ●●●●●●●●●●●●●●●●──── train loss still HIGH         │
  │       │  ○  ○○○○○○○○○○○○○○○──── val loss also HIGH            │
  │       └────────────────────→                                    │
  │                                                                  │
  │  Diagnosis: Model too simple or LR too low                      │
  │  Fix: Bigger model, higher LR, train longer, check data        │
  ├──────────────────────────────────────────────────────────────────┤
  │  PATTERN 3: LEARNING RATE TOO HIGH                              │
  │                                                                  │
  │  Loss ↑                                                         │
  │       │●  ●                                                     │
  │       │ ●╱ ╲●                                                   │
  │       │       ╲●  ●                                             │
  │       │         ●╱ ╲●  ●                                       │
  │       │               ╲╱ ╲●●───                                │
  │       └────────────────────→                                    │
  │                                                                  │
  │  Diagnosis: Loss oscillates / spikes / doesn't converge         │
  │  Fix: Reduce learning rate by 3-10×                             │
  ├──────────────────────────────────────────────────────────────────┤
  │  PATTERN 4: LEARNING RATE TOO LOW                               │
  │                                                                  │
  │  Loss ↑                                                         │
  │       │●                                                        │
  │       │ ●                                                       │
  │       │  ●                                                      │
  │       │   ●                                                     │
  │       │    ●●                                                   │
  │       │      ●●●                                                │
  │       │         ●●●●●●●●●●●──── VERY slow decrease            │
  │       └────────────────────→                                    │
  │                                                                  │
  │  Diagnosis: Training too slow, loss still decreasing            │
  │  Fix: Increase learning rate by 3-10×                           │
  ├──────────────────────────────────────────────────────────────────┤
  │  PATTERN 5: GOOD FIT                                            │
  │                                                                  │
  │  Loss ↑                                                         │
  │       │●                                                        │
  │       │ ●                                                       │
  │       │  ●○                                                     │
  │       │   ●○○                                                   │
  │       │    ●●○○                                                 │
  │       │     ●●●○○○                                              │
  │       │       ●●●●○○○○○○○──── small gap                       │
  │       │          ●●●●●●●●──── converged                        │
  │       └────────────────────→                                    │
  │                                                                  │
  │  Diagnosis: Training is healthy!                                │
  │  Fix: Nothing — save this model!                                │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 3. Accuracy Curves

```
  Accuracy ↑
  1.0 │                  ●●●●●●●●●● train accuracy
      │              ●●●●
  0.9 │          ●●●●
      │       ●●●     ○○○○○○○○○○○○ val accuracy
  0.8 │     ●●   ○○○○○
      │    ●  ○○○
  0.7 │   ●○○○
      │  ●○
  0.6 │ ●○
      │●○
  0.5 │○
      └────────────────────────────→ Epochs
  
  Train-Val gap = overfitting indicator
  
  ┌────────────────────────────────────────────────────┐
  │  Gap < 2%:   Healthy                               │
  │  Gap 2-5%:   Mild overfitting (often acceptable)  │
  │  Gap 5-10%:  Significant overfitting              │
  │  Gap > 10%:  Severe overfitting → regularize!     │
  └────────────────────────────────────────────────────┘
```

---

## 4. Gradient Monitoring

### Gradient Norms

```
  Gradient norms reveal training health:
  
  ||∇L||₂ for each layer over training
  
  Grad Norm ↑
            │
     1e+5   │●                        EXPLODING!
            │ ●
     1e+3   │  ●●
            │    ●●●
     1e+1   │       ●●●●●● Healthy    HEALTHY
            │              ●●●●●●●●
     1e-1   │              
            │                          
     1e-3   │                         If gradients → 0
            │                         = VANISHING!
     1e-5   │                ●●●●●●●●●●●●
            └────────────────────────────→ Training Steps
  
  ┌──────────────────────────────────────────────────────────────┐
  │  Gradient Norm   │ Status        │ Action                    │
  ├──────────────────┼───────────────┼───────────────────────────┤
  │  Steady ~1       │ Healthy       │ Continue training         │
  │  Growing rapidly │ Exploding     │ Clip gradients, reduce LR │
  │  Shrinking → 0   │ Vanishing     │ Skip connections, BN      │
  │  Oscillating     │ Unstable      │ Reduce LR                 │
  │  Exactly 0       │ Dead neurons  │ Check architecture        │
  └──────────────────┴───────────────┴───────────────────────────┘
```

### Monitoring Gradients in PyTorch

```python
def compute_gradient_norms(model):
    """Compute gradient L2 norm for each layer."""
    norms = {}
    for name, param in model.named_parameters():
        if param.grad is not None:
            norms[name] = param.grad.data.norm(2).item()
    return norms

# In training loop:
for epoch in range(num_epochs):
    for X, y in train_loader:
        optimizer.zero_grad()
        loss = criterion(model(X), y)
        loss.backward()
        
        # Log gradient norms BEFORE optimizer.step()
        if step % log_interval == 0:
            grad_norms = compute_gradient_norms(model)
            for name, norm in grad_norms.items():
                writer.add_scalar(f'grad_norm/{name}', norm, step)
        
        optimizer.step()
```

---

## 5. Weight Distributions

```
  Track how weights evolve during training:
  
  Epoch 0 (random init):     Epoch 10:              Epoch 50:
  
  Count ↑                    Count ↑                Count ↑
        │    ╱╲                    │   ╱╲                  │  ╱╲
        │   ╱  ╲                   │  ╱  ╲                 │ ╱  ╲
        │  ╱    ╲                  │ ╱    ╲                │╱    ╲
        │ ╱      ╲                 │╱      ╲              ╱╱      ╲╲
        │╱        ╲                ╱        ╲            ╱╱         ╲╲
        └────────────→             └────────────→        └──────────────→
        -0.1  0  0.1              -0.3  0  0.3          -0.5  0  0.5
        
  ✓ Healthy: Distribution gradually shifts and spreads
  ✗ Problem: All weights → 0 (dying layers)
  ✗ Problem: Weights → very large values (exploding)
  ✗ Problem: Bimodal distribution (training issue)
```

```python
def log_weight_stats(model, writer, step):
    """Log weight statistics to TensorBoard."""
    for name, param in model.named_parameters():
        if 'weight' in name:
            writer.add_histogram(f'weights/{name}', param.data, step)
            writer.add_scalar(f'weight_norm/{name}', param.data.norm(2).item(), step)
            writer.add_scalar(f'weight_mean/{name}', param.data.mean().item(), step)
            writer.add_scalar(f'weight_std/{name}', param.data.std().item(), step)
```

---

## 6. TensorBoard Setup and Usage

### Installation and Basic Setup

```python
# pip install tensorboard

import torch
from torch.utils.tensorboard import SummaryWriter

# Create writer — logs go to 'runs/' directory
writer = SummaryWriter('runs/experiment_001')

# Or with more specific naming:
writer = SummaryWriter(f'runs/lr{lr}_bs{batch_size}_layers{n_layers}')
```

### Complete Training Loop with TensorBoard

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torch.utils.tensorboard import SummaryWriter
from torchvision import datasets, transforms

# ── Setup ──
writer = SummaryWriter('runs/mnist_mlp_v1')
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])
train_data = datasets.MNIST('./data', train=True, download=True, transform=transform)
val_data = datasets.MNIST('./data', train=False, transform=transform)
train_loader = DataLoader(train_data, batch_size=128, shuffle=True)
val_loader = DataLoader(val_data, batch_size=512)

model = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 256), nn.ReLU(), nn.Dropout(0.3),
    nn.Linear(256, 128), nn.ReLU(), nn.Dropout(0.3),
    nn.Linear(128, 10)
).to(device)

optimizer = optim.Adam(model.parameters(), lr=1e-3)
criterion = nn.CrossEntropyLoss()

# ── Log model architecture ──
dummy_input = torch.randn(1, 1, 28, 28).to(device)
writer.add_graph(model, dummy_input)

# ── Training loop ──
global_step = 0
for epoch in range(30):
    # ── Training phase ──
    model.train()
    train_loss = 0
    train_correct = 0
    train_total = 0
    
    for batch_idx, (X, y) in enumerate(train_loader):
        X, y = X.to(device), y.to(device)
        
        optimizer.zero_grad()
        outputs = model(X)
        loss = criterion(outputs, y)
        loss.backward()
        
        # Log batch-level metrics
        if global_step % 100 == 0:
            writer.add_scalar('batch/train_loss', loss.item(), global_step)
            
            # Gradient norms
            total_norm = 0
            for p in model.parameters():
                if p.grad is not None:
                    total_norm += p.grad.data.norm(2).item() ** 2
            total_norm = total_norm ** 0.5
            writer.add_scalar('batch/gradient_norm', total_norm, global_step)
        
        optimizer.step()
        
        train_loss += loss.item() * X.size(0)
        train_correct += (outputs.argmax(1) == y).sum().item()
        train_total += y.size(0)
        global_step += 1
    
    # Epoch-level training metrics
    epoch_train_loss = train_loss / train_total
    epoch_train_acc = train_correct / train_total
    
    # ── Validation phase ──
    model.eval()
    val_loss = 0
    val_correct = 0
    val_total = 0
    
    with torch.no_grad():
        for X, y in val_loader:
            X, y = X.to(device), y.to(device)
            outputs = model(X)
            loss = criterion(outputs, y)
            val_loss += loss.item() * X.size(0)
            val_correct += (outputs.argmax(1) == y).sum().item()
            val_total += y.size(0)
    
    epoch_val_loss = val_loss / val_total
    epoch_val_acc = val_correct / val_total
    
    # ── Log epoch metrics ──
    writer.add_scalars('loss', {
        'train': epoch_train_loss,
        'val': epoch_val_loss
    }, epoch)
    
    writer.add_scalars('accuracy', {
        'train': epoch_train_acc,
        'val': epoch_val_acc
    }, epoch)
    
    writer.add_scalar('lr', optimizer.param_groups[0]['lr'], epoch)
    
    # Log weight distributions every 5 epochs
    if epoch % 5 == 0:
        for name, param in model.named_parameters():
            writer.add_histogram(f'params/{name}', param.data, epoch)
            if param.grad is not None:
                writer.add_histogram(f'grads/{name}', param.grad.data, epoch)
    
    print(f"Epoch {epoch+1:2d} | Train Loss: {epoch_train_loss:.4f} | "
          f"Val Loss: {epoch_val_loss:.4f} | Val Acc: {epoch_val_acc:.4f}")

writer.close()
```

### Launching TensorBoard

```bash
# In terminal:
tensorboard --logdir=runs --port=6006

# Open in browser: http://localhost:6006

# Compare multiple experiments:
tensorboard --logdir=runs  # will show all experiments in runs/
```

### TensorBoard Dashboard Layout

```
  ┌──────────────────────────────────────────────────────────────┐
  │  TensorBoard Dashboard                                       │
  ├──────────────────────────────────────────────────────────────┤
  │                                                              │
  │  SCALARS tab:                                                │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
  │  │ loss/train   │  │ loss/val     │  │ accuracy     │      │
  │  │   ↘_____     │  │   ↘___       │  │      ___↗   │      │
  │  └──────────────┘  └──────────────┘  └──────────────┘      │
  │                                                              │
  │  GRAPHS tab:                                                 │
  │  ┌──────────────────────────────────────────┐               │
  │  │  [Input] → [Linear] → [ReLU] → [Output] │               │
  │  └──────────────────────────────────────────┘               │
  │                                                              │
  │  HISTOGRAMS tab:                                             │
  │  ┌──────────────┐  ┌──────────────┐                         │
  │  │ layer1.weight│  │ layer1.grad  │                         │
  │  │   ▃▅█▅▃     │  │    ▂▅█▅▂    │                         │
  │  └──────────────┘  └──────────────┘                         │
  └──────────────────────────────────────────────────────────────┘
```

---

## 7. Weights & Biases (W&B) Basics

### Setup

```python
# pip install wandb
# wandb login  (one-time setup)

import wandb

# Initialize a run
wandb.init(
    project="mnist-experiments",
    name="mlp-v1-lr1e3-bs128",
    config={
        "learning_rate": 1e-3,
        "batch_size": 128,
        "epochs": 30,
        "hidden_size": 256,
        "dropout": 0.3,
        "optimizer": "Adam",
        "architecture": "MLP",
    }
)
```

### Logging with W&B

```python
for epoch in range(num_epochs):
    # ... training code ...
    
    # Log metrics (automatic plots!)
    wandb.log({
        "epoch": epoch,
        "train/loss": train_loss,
        "train/accuracy": train_acc,
        "val/loss": val_loss,
        "val/accuracy": val_acc,
        "lr": optimizer.param_groups[0]['lr'],
    })
    
    # Log confusion matrix periodically
    if epoch % 10 == 0:
        wandb.log({
            "confusion_matrix": wandb.plot.confusion_matrix(
                y_true=all_labels, preds=all_preds,
                class_names=[str(i) for i in range(10)]
            )
        })

# Save model as artifact
artifact = wandb.Artifact('model', type='model')
artifact.add_file('best_model.pth')
wandb.log_artifact(artifact)

wandb.finish()
```

### TensorBoard vs W&B Comparison

```
  ┌───────────────────┬──────────────────┬──────────────────────┐
  │  Feature          │  TensorBoard     │  Weights & Biases    │
  ├───────────────────┼──────────────────┼──────────────────────┤
  │  Cost             │  Free            │  Free (basic)        │
  │  Setup            │  Local server    │  Cloud-based         │
  │  Collaboration    │  Manual sharing  │  Built-in teams      │
  │  Experiment       │  Manual naming   │  Auto-tracking       │
  │  comparison       │                  │                      │
  │  Hyperparameter   │  HParams plugin  │  Sweeps              │
  │  search           │                  │                      │
  │  Alerts           │  No              │  Yes (email/Slack)   │
  │  Model registry   │  No              │  Yes                 │
  │  Report generation│  No              │  Yes                 │
  │  Best for         │  Quick local     │  Team projects       │
  │                   │  experiments     │  long-term tracking  │
  └───────────────────┴──────────────────┴──────────────────────┘
```

---

## 8. Detecting Problems from Curves

### Diagnostic Flowchart

```
  START: Examine training curves
  │
  ├─ Is train loss decreasing?
  │  ├─ NO  →  Check: LR too high? Data pipeline broken?
  │  │         Labels shuffled? Loss function correct?
  │  └─ YES ↓
  │
  ├─ Is train loss VERY LOW (near 0)?
  │  ├─ YES → Likely memorizing. Check val loss.
  │  └─ NO  ↓
  │
  ├─ Is val loss decreasing?
  │  ├─ NO  → OVERFITTING. Add regularization.
  │  └─ YES ↓
  │
  ├─ Is val loss much higher than train loss?
  │  ├─ YES → OVERFITTING. Add dropout/weight decay.
  │  └─ NO  ↓
  │
  ├─ Are both losses still high?
  │  ├─ YES → UNDERFITTING. Increase model capacity.
  │  └─ NO  → GOOD FIT! Save model.
  │
  └─ Are there spikes in loss?
     ├─ YES → LR too high or bad data batches
     └─ NO  → Training is smooth ✓
```

### Common Curve Patterns and Fixes

```
  ┌──────────────────────────────────────────────────────────────┐
  │  PATTERN              │ LIKELY CAUSE    │ FIX               │
  ├───────────────────────┼─────────────────┼────────────────────┤
  │  Loss flat from start │ LR too low      │ Increase LR 10×   │
  │                       │ Dead ReLUs      │ Use LeakyReLU      │
  │                       │ Wrong loss fn   │ Check loss function│
  ├───────────────────────┼─────────────────┼────────────────────┤
  │  Loss oscillates      │ LR too high     │ Decrease LR 3-10× │
  │  wildly               │ Batch too small │ Increase batch size│
  ├───────────────────────┼─────────────────┼────────────────────┤
  │  Loss goes to NaN     │ LR way too high │ Reduce LR 10×     │
  │                       │ Numerical issues│ Gradient clipping  │
  │                       │ log(0) or 0/0   │ Add epsilon        │
  ├───────────────────────┼─────────────────┼────────────────────┤
  │  Loss spikes then     │ Bad batch of    │ Check data pipeline│
  │  recovers             │ data            │ Clip gradients     │
  ├───────────────────────┼─────────────────┼────────────────────┤
  │  Train↓ Val↑          │ Overfitting     │ Regularization     │
  │  (diverging)          │                 │ Early stopping     │
  ├───────────────────────┼─────────────────┼────────────────────┤
  │  Both plateau HIGH    │ Underfitting    │ Bigger model       │
  │                       │                 │ Better features    │
  ├───────────────────────┼─────────────────┼────────────────────┤
  │  Loss plateaus then   │ LR schedule     │ Use LR scheduler   │
  │  decreases again      │ kicked in       │ (this is normal!)  │
  └───────────────────────┴─────────────────┴────────────────────┘
```

---

## 9. What to Log: Complete Checklist

```python
class TrainingLogger:
    """Complete training logger for neural networks."""
    
    def __init__(self, log_dir, use_wandb=False):
        self.writer = SummaryWriter(log_dir)
        self.use_wandb = use_wandb
        self.step = 0
    
    def log_batch(self, loss, model, batch_idx, epoch):
        """Log per-batch metrics."""
        self.step += 1
        
        if self.step % 50 == 0:  # every 50 batches
            self.writer.add_scalar('batch/loss', loss.item(), self.step)
            
            # Gradient norm
            total_norm = sum(
                p.grad.data.norm(2).item() ** 2 
                for p in model.parameters() if p.grad is not None
            ) ** 0.5
            self.writer.add_scalar('batch/grad_norm', total_norm, self.step)
    
    def log_epoch(self, epoch, train_loss, val_loss, train_acc, val_acc, lr):
        """Log per-epoch metrics."""
        metrics = {
            'epoch/train_loss': train_loss,
            'epoch/val_loss': val_loss,
            'epoch/train_acc': train_acc,
            'epoch/val_acc': val_acc,
            'epoch/lr': lr,
            'epoch/overfit_gap': train_acc - val_acc,
        }
        
        for name, value in metrics.items():
            self.writer.add_scalar(name, value, epoch)
        
        if self.use_wandb:
            wandb.log({k.split('/')[-1]: v for k, v in metrics.items()})
    
    def log_weights(self, model, epoch):
        """Log weight distributions (every few epochs)."""
        for name, param in model.named_parameters():
            self.writer.add_histogram(f'weights/{name}', param.data, epoch)
            self.writer.add_scalar(
                f'weight_stats/{name}_norm', param.data.norm(2).item(), epoch
            )
    
    def log_images(self, tag, images, epoch, n=8):
        """Log sample images."""
        grid = torchvision.utils.make_grid(images[:n], normalize=True)
        self.writer.add_image(tag, grid, epoch)
    
    def close(self):
        self.writer.close()
        if self.use_wandb:
            wandb.finish()
```

---

## 10. Summary Table

| Metric | Frequency | Tool | Why Monitor |
|--------|-----------|------|-------------|
| **Train loss** | Every batch | TensorBoard/W&B | Training progress |
| **Val loss** | Every epoch | TensorBoard/W&B | Generalization |
| **Train/Val accuracy** | Every epoch | TensorBoard/W&B | Performance |
| **Learning rate** | Every epoch | TensorBoard/W&B | Schedule verification |
| **Gradient norms** | Every N batches | TensorBoard | Vanishing/exploding |
| **Weight distributions** | Every N epochs | TensorBoard | Weight health |
| **GPU memory** | Start of training | nvidia-smi | Memory planning |
| **Throughput** | Periodic | Custom logging | Efficiency |
| **Confusion matrix** | End of training | W&B/matplotlib | Error analysis |

---

## 11. Revision Questions

1. **Sketch the loss curves for overfitting, underfitting, and a well-fitted model.** What is the key visual difference between each?

2. **Your training loss is decreasing normally but validation loss starts increasing after epoch 15. What is happening and what are three strategies to fix it?**

3. **What do gradient norms tell you about training health?** Describe what gradient norms look like when gradients are (a) vanishing, (b) exploding, and (c) healthy.

4. **Compare TensorBoard and Weights & Biases.** When would you choose one over the other?

5. **Write PyTorch code to log training loss, validation loss, gradient norms, and weight histograms to TensorBoard.** Include the SummaryWriter setup and the logging calls inside the training loop.

6. **Your loss curve shows periodic spikes that recover quickly. What could cause this and how would you diagnose the issue?**

---

| [← 03 Batch Size Selection](03-batch-size-selection.md) | [Unit 8 Home](README.md) | [05 → Debugging Neural Networks](05-debugging-neural-networks.md) |
|:---|:---:|---:|
