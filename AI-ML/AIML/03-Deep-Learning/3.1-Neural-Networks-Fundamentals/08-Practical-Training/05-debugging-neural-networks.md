# 5. Debugging Neural Networks

> **Unit 8 · Practical Training** — A systematic approach to finding and fixing training failures

---

## Chapter Overview

Neural networks fail silently. Unlike traditional software that throws errors, a neural network with bugs will simply produce poor results — and it's up to *you* to figure out why. From loss functions that never decrease to NaN values that appear mysteriously, debugging neural networks requires a systematic methodology. This chapter presents a comprehensive debugging checklist inspired by **Andrej Karpathy's recipe**, covers the most common failure modes and their fixes, and provides a step-by-step process for diagnosing and resolving training issues.

---

## 1. The Challenge of Debugging Neural Networks

```
  Traditional Software Debugging:
  ┌──────────────────────────────────────┐
  │  Code → Error message → Fix code    │
  │  Clear cause and effect             │
  │  Deterministic bugs                 │
  └──────────────────────────────────────┘
  
  Neural Network Debugging:
  ┌──────────────────────────────────────────────────────────┐
  │  Code runs fine... but accuracy is 52% instead of 95%   │
  │  No error messages!                                      │
  │  Could be: data, architecture, hyperparameters, loss,   │
  │  optimizer, preprocessing, augmentation, labels, seed,  │
  │  initialization, normalization, learning rate, batch     │
  │  size, regularization, gradient flow, numerical issues  │
  │  ... or any COMBINATION of these!                       │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Andrej Karpathy's Recipe: The Gold Standard

Andrej Karpathy (former Director of AI at Tesla, now at OpenAI) shared a widely-adopted recipe for training neural networks. The core principle: **start simple, verify everything, and add complexity incrementally.**

```
  ┌──────────────────────────────────────────────────────────────┐
  │  KARPATHY'S RECIPE FOR TRAINING NEURAL NETWORKS             │
  │                                                              │
  │  1. BECOME ONE WITH THE DATA                                 │
  │     • Visualize samples, check distributions                │
  │     • Look for corrupt data, wrong labels                   │
  │     • Understand class balance                              │
  │                                                              │
  │  2. SET UP END-TO-END PIPELINE + GET DUMB BASELINE          │
  │     • Build simplest possible model                         │
  │     • Verify loss makes sense at initialization             │
  │     • Get a baseline metric                                 │
  │                                                              │
  │  3. OVERFIT A SINGLE BATCH                     ← KEY STEP! │
  │     • Take 1 batch, train until 100% accuracy              │
  │     • If this fails → bug in model/loss/optimizer          │
  │     • This MUST work before proceeding                      │
  │                                                              │
  │  4. REGULARIZE                                               │
  │     • Add data, dropout, weight decay                       │
  │     • Get val performance to match train                    │
  │                                                              │
  │  5. TUNE                                                     │
  │     • Use LR finder, random search                          │
  │     • Find best hyperparameters                             │
  │                                                              │
  │  6. SQUEEZE OUT THE JUICE                                   │
  │     • Ensembles, test-time augmentation                     │
  │     • Knowledge distillation                                │
  └──────────────────────────────────────────────────────────────┘
```

### Step 3 Deep Dive: Overfit a Single Batch

This is the most powerful debugging technique:

```python
# THE MOST IMPORTANT DEBUGGING STEP
# If your model can't overfit a single batch, something is fundamentally wrong!

import torch
import torch.nn as nn
import torch.optim as optim

model = MyModel().to(device)
optimizer = optim.Adam(model.parameters(), lr=1e-3)
criterion = nn.CrossEntropyLoss()

# Get ONE batch
X_batch, y_batch = next(iter(train_loader))
X_batch, y_batch = X_batch.to(device), y_batch.to(device)

# Train on ONLY this batch
for step in range(1000):
    optimizer.zero_grad()
    outputs = model(X_batch)
    loss = criterion(outputs, y_batch)
    loss.backward()
    optimizer.step()
    
    if step % 100 == 0:
        acc = (outputs.argmax(1) == y_batch).float().mean()
        print(f"Step {step:4d} | Loss: {loss.item():.6f} | Acc: {acc:.4f}")

# Expected output:
# Step    0 | Loss: 2.302585 | Acc: 0.0938  (random for 10 classes)
# Step  100 | Loss: 0.543210 | Acc: 0.8125
# Step  200 | Loss: 0.032100 | Acc: 0.9688
# Step  300 | Loss: 0.001234 | Acc: 1.0000  ← MUST reach ~100%!
# Step  400 | Loss: 0.000123 | Acc: 1.0000
```

```
  ┌──────────────────────────────────────────────────────────────┐
  │  IF SINGLE-BATCH OVERFIT FAILS:                              │
  │                                                              │
  │  Loss doesn't decrease at all:                              │
  │  → Learning rate too low or too high                        │
  │  → Gradient not flowing (check model architecture)          │
  │  → Loss function wrong                                      │
  │  → Labels wrong / shuffled                                  │
  │                                                              │
  │  Loss decreases but accuracy stays low:                     │
  │  → Labels might be wrong                                    │
  │  → Metric calculation bug                                   │
  │  → Softmax applied twice                                    │
  │                                                              │
  │  Loss goes to NaN:                                          │
  │  → Numerical instability                                    │
  │  → Learning rate way too high                               │
  │  → log(0) somewhere in loss                                 │
  │                                                              │
  │  IF SINGLE-BATCH OVERFIT SUCCEEDS:                          │
  │  → Model and training loop are correct!                     │
  │  → Problem is data, regularization, or hyperparameters      │
  └──────────────────────────────────────────────────────────────┘
```

---

## 3. Common Issues and Fixes

### Issue 1: Loss Not Decreasing

```
  SYMPTOM: Loss stays flat or barely moves
  
  Loss ↑
       │●●●●●●●●●●●●●●●●●●●●●●●●●●●●── flat!
       │
       └────────────────────────────→ Epochs
  
  DEBUGGING CHECKLIST:
  ┌──────────────────────────────────────────────────────────────┐
  │  □ Check learning rate                                       │
  │    → Try: 1e-5, 1e-4, 1e-3, 1e-2, 1e-1                    │
  │    → Most common cause of flat loss!                        │
  │                                                              │
  │  □ Check data pipeline                                       │
  │    → Are inputs normalized?                                  │
  │    → Are labels correct? (Visualize some!)                  │
  │    → Is the DataLoader returning data correctly?            │
  │                                                              │
  │  □ Check loss function                                       │
  │    → Are you using the right loss for your task?            │
  │    → CrossEntropyLoss expects RAW logits (no softmax!)     │
  │    → BCEWithLogitsLoss vs BCELoss                           │
  │                                                              │
  │  □ Check model output shape                                  │
  │    → Does output match number of classes?                   │
  │    → Are dimensions correct?                                │
  │                                                              │
  │  □ Verify gradient flow                                      │
  │    → Print grad norms: are they 0? (dead network)           │
  │    → Check for accidental torch.no_grad() context           │
  │    → Is requires_grad=True for parameters?                  │
  │                                                              │
  │  □ Check optimizer                                           │
  │    → Are model parameters passed to optimizer?              │
  │    → optimizer = Adam(model.parameters(), ...) ← correct?   │
  │    → Did you call optimizer.zero_grad()?                    │
  └──────────────────────────────────────────────────────────────┘
```

```python
# Quick sanity checks for loss not decreasing:

# 1. Check initial loss matches expected value
# For CrossEntropyLoss with C classes, initial loss ≈ -ln(1/C)
n_classes = 10
expected_initial_loss = -torch.log(torch.tensor(1.0 / n_classes))
print(f"Expected initial loss for {n_classes} classes: {expected_initial_loss:.4f}")
# Should be ≈ 2.3026 for 10 classes

# 2. Check that parameters are being updated
for name, param in model.named_parameters():
    print(f"{name}: requires_grad={param.requires_grad}, grad={param.grad is not None}")

# 3. Check gradient magnitude
def check_gradients(model):
    for name, param in model.named_parameters():
        if param.grad is not None:
            print(f"{name}: grad_norm={param.grad.norm():.6f}")
        else:
            print(f"{name}: NO GRADIENT!")

# After one backward pass:
loss.backward()
check_gradients(model)
```

### Issue 2: NaN Loss

```
  SYMPTOM: Loss becomes NaN (Not a Number)
  
  Loss ↑
       │●
       │ ●●
       │   ●●●●
       │       ●●●  → NaN NaN NaN NaN NaN ...
       │
       └────────────────────────────→ Steps
  
  COMMON CAUSES AND FIXES:
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  CAUSE 1: Learning rate too high                            │
  │  Fix: Reduce by 10× (e.g., 1e-2 → 1e-3)                  │
  │                                                              │
  │  CAUSE 2: Gradient explosion                                │
  │  Fix: Gradient clipping                                      │
  │     torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)│
  │                                                              │
  │  CAUSE 3: log(0) in loss computation                        │
  │  Fix: Add epsilon: log(x + 1e-8) instead of log(x)         │
  │       Use numerically stable loss (CrossEntropyLoss)        │
  │                                                              │
  │  CAUSE 4: Division by zero                                  │
  │  Fix: Add epsilon to denominators                           │
  │       Check for zero-variance features in normalization     │
  │                                                              │
  │  CAUSE 5: Extreme input values                              │
  │  Fix: Normalize inputs to reasonable range                  │
  │       Check for inf or NaN in data                          │
  │                                                              │
  │  CAUSE 6: exp() overflow                                    │
  │  Fix: Use LogSoftmax instead of Softmax + log               │
  │       Use logsumexp trick                                   │
  └──────────────────────────────────────────────────────────────┘
```

```python
# Debugging NaN loss:

# 1. Check for NaN in inputs
def check_for_nan(loader, name="data"):
    for X, y in loader:
        if torch.isnan(X).any():
            print(f"NaN found in {name} inputs!")
            return True
        if torch.isinf(X).any():
            print(f"Inf found in {name} inputs!")
            return True
    print(f"No NaN/Inf in {name}")
    return False

check_for_nan(train_loader, "train")

# 2. Check for NaN after each layer
def forward_with_nan_check(model, x):
    for i, layer in enumerate(model):
        x = layer(x)
        if torch.isnan(x).any():
            print(f"NaN appeared after layer {i}: {layer}")
            return x
    return x

# 3. Enable anomaly detection (slow but catches NaN sources)
torch.autograd.set_detect_anomaly(True)  # will raise error at NaN source

# 4. Gradient clipping
optimizer.zero_grad()
loss.backward()
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
optimizer.step()
```

### Issue 3: Overfitting

```
  SYMPTOM: Train loss ↓ but val loss ↑
  
  FIXES (in order of impact):
  ┌──────────────────────────────────────────────────────────────┐
  │  1. GET MORE DATA (most effective!)                         │
  │     → Data augmentation (rotation, flip, crop)              │
  │     → Synthetic data generation                             │
  │                                                              │
  │  2. ADD REGULARIZATION                                      │
  │     → Dropout (0.2-0.5)                                     │
  │     → Weight decay (1e-5 to 1e-3)                           │
  │     → Batch normalization                                   │
  │                                                              │
  │  3. REDUCE MODEL CAPACITY                                   │
  │     → Fewer layers or fewer neurons                         │
  │     → Simpler architecture                                  │
  │                                                              │
  │  4. EARLY STOPPING                                          │
  │     → Stop training when val loss stops improving           │
  │     → Use patience of 5-10 epochs                           │
  │                                                              │
  │  5. CROSS-VALIDATION                                        │
  │     → Ensure result is robust, not data-split dependent     │
  └──────────────────────────────────────────────────────────────┘
```

### Issue 4: Underfitting

```
  SYMPTOM: Both train and val loss are high
  
  FIXES (in order of priority):
  ┌──────────────────────────────────────────────────────────────┐
  │  1. INCREASE MODEL CAPACITY                                 │
  │     → More layers, more neurons                             │
  │     → More complex architecture (e.g., ResNet)              │
  │                                                              │
  │  2. TRAIN LONGER                                            │
  │     → More epochs                                           │
  │     → Check if loss is still decreasing                     │
  │                                                              │
  │  3. REDUCE REGULARIZATION                                   │
  │     → Lower dropout                                         │
  │     → Lower weight decay                                    │
  │     → Remove data augmentation temporarily                  │
  │                                                              │
  │  4. INCREASE LEARNING RATE                                  │
  │     → LR may be too low to make progress                   │
  │     → Use LR range test                                     │
  │                                                              │
  │  5. CHECK FEATURES                                          │
  │     → Are input features informative enough?                │
  │     → Feature engineering might help                        │
  │                                                              │
  │  6. CHECK DATA                                              │
  │     → Is the task learnable from this data?                 │
  │     → Is there too much noise?                              │
  └──────────────────────────────────────────────────────────────┘
```

### Issue 5: Training is Unstable

```
  SYMPTOM: Loss oscillates wildly, occasional spikes
  
  Loss ↑
       │●
       │ ●  ●
       │  ●╱ ╲●
       │       ╲●  ●
       │         ●╱ ╲●
       │              ╲●●──
       └────────────────────→
  
  FIXES:
  ┌──────────────────────────────────────────────────────────────┐
  │  1. REDUCE LEARNING RATE                                    │
  │     → Divide by 3 or 10                                    │
  │                                                              │
  │  2. ADD GRADIENT CLIPPING                                   │
  │     → clip_grad_norm_(params, max_norm=1.0)                │
  │                                                              │
  │  3. INCREASE BATCH SIZE                                     │
  │     → Reduces gradient noise                               │
  │                                                              │
  │  4. ADD BATCH NORMALIZATION                                 │
  │     → Stabilizes internal distributions                    │
  │                                                              │
  │  5. ADD LEARNING RATE WARMUP                                │
  │     → Start low, ramp up over first few epochs             │
  │                                                              │
  │  6. CHECK FOR BAD DATA                                      │
  │     → Outliers, corrupt samples, wrong labels              │
  │     → The spikes might correspond to bad batches            │
  └──────────────────────────────────────────────────────────────┘
```

---

## 4. Systematic Debugging Checklist

```
  ┌──────────────────────────────────────────────────────────────┐
  │             NEURAL NETWORK DEBUGGING CHECKLIST               │
  │                                                              │
  │  PHASE 1: DATA VERIFICATION                                 │
  │  ─────────────────────────                                   │
  │  □ Visualize random samples from training data              │
  │  □ Check label distribution (class imbalance?)              │
  │  □ Verify data normalization / preprocessing                │
  │  □ Check for NaN/Inf in data                                │
  │  □ Verify data augmentation is correct                      │
  │  □ Check DataLoader: shuffle=True for training?             │
  │  □ Print shapes at each step of data pipeline               │
  │                                                              │
  │  PHASE 2: MODEL VERIFICATION                                │
  │  ──────────────────────────                                  │
  │  □ Check model output shape matches target shape            │
  │  □ Count parameters: too many → overfit, too few → underfit │
  │  □ Verify forward pass produces reasonable output           │
  │  □ Check initial loss = -ln(1/C) for classification        │
  │  □ Verify no duplicate softmax (model + loss function)      │
  │                                                              │
  │  PHASE 3: TRAINING LOOP VERIFICATION                        │
  │  ─────────────────────────────────                           │
  │  □ Overfit a single batch (CRITICAL!)                       │
  │  □ Verify optimizer.zero_grad() is called                   │
  │  □ Verify loss.backward() is called                         │
  │  □ Verify optimizer.step() is called                        │
  │  □ Check order: zero_grad → forward → loss → backward → step│
  │  □ Verify model is in train mode during training            │
  │  □ Verify model is in eval mode during validation           │
  │  □ Check that torch.no_grad() is used for validation        │
  │                                                              │
  │  PHASE 4: HYPERPARAMETER VERIFICATION                       │
  │  ───────────────────────────────────                         │
  │  □ Try learning rate: 1e-5, 1e-4, 1e-3, 1e-2, 1e-1        │
  │  □ Start with Adam optimizer (most forgiving)               │
  │  □ Start with NO regularization (dropout=0, wd=0)          │
  │  □ Use standard batch size (32 or 64)                       │
  │                                                              │
  │  PHASE 5: GRADIENT VERIFICATION                              │
  │  ────────────────────────────                                │
  │  □ Check gradient norms (should be ~1, not 0 or inf)       │
  │  □ Check each layer has gradients (no dead layers)          │
  │  □ Enable torch.autograd.set_detect_anomaly(True)          │
  └──────────────────────────────────────────────────────────────┘
```

---

## 5. Common Code Bugs

### The Most Frequent Bugs

```python
# ─── BUG 1: Forgetting to call zero_grad() ───
# WRONG:
for X, y in train_loader:
    loss = criterion(model(X), y)
    loss.backward()
    optimizer.step()  # gradients ACCUMULATE across batches!

# CORRECT:
for X, y in train_loader:
    optimizer.zero_grad()  # ← reset gradients!
    loss = criterion(model(X), y)
    loss.backward()
    optimizer.step()


# ─── BUG 2: Applying softmax before CrossEntropyLoss ───
# WRONG:
class BadModel(nn.Module):
    def forward(self, x):
        x = self.layers(x)
        return torch.softmax(x, dim=1)  # ← softmax already applied!

loss = nn.CrossEntropyLoss()  # applies log_softmax internally!
# Result: softmax(softmax(x)) → terrible performance

# CORRECT:
class GoodModel(nn.Module):
    def forward(self, x):
        return self.layers(x)  # return RAW logits

loss = nn.CrossEntropyLoss()  # will apply log_softmax


# ─── BUG 3: Not switching between train/eval mode ───
# WRONG:
# model stays in train mode during validation
# → dropout still active → worse val performance reported

# CORRECT:
model.train()   # before training loop
# ... training ...
model.eval()    # before validation loop
with torch.no_grad():
    # ... validation ...


# ─── BUG 4: Not using torch.no_grad() during validation ───
# WRONG:
model.eval()
for X, y in val_loader:
    outputs = model(X)  # still computing gradients → wastes memory!

# CORRECT:
model.eval()
with torch.no_grad():  # ← no gradient computation
    for X, y in val_loader:
        outputs = model(X)


# ─── BUG 5: Moving model to GPU but not data ───
# WRONG:
model = model.to('cuda')
for X, y in train_loader:
    output = model(X)  # X is on CPU, model on GPU → error!

# CORRECT:
model = model.to(device)
for X, y in train_loader:
    X, y = X.to(device), y.to(device)  # move data too!
    output = model(X)


# ─── BUG 6: Wrong dimension in loss function ───
# For CrossEntropyLoss:
# Input:  (batch_size, num_classes)  ← 2D!
# Target: (batch_size,)              ← 1D!  (class indices, NOT one-hot)

# WRONG:
output = model(X)  # shape: (32, 10)
target = F.one_hot(y, 10).float()  # shape: (32, 10) ← wrong!
loss = nn.CrossEntropyLoss()(output, target)

# CORRECT:
output = model(X)  # shape: (32, 10)
loss = nn.CrossEntropyLoss()(output, y)  # y shape: (32,)


# ─── BUG 7: Data leakage ───
# WRONG: Fitting normalizer on ALL data including test
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
X_all_scaled = scaler.fit_transform(X_all)  # fit on everything!
X_train, X_test = X_all_scaled[:split], X_all_scaled[split:]

# CORRECT: Fit only on training data
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)  # fit on train only
X_test_scaled = scaler.transform(X_test)         # transform test
```

---

## 6. Diagnostic Code Toolkit

```python
import torch
import torch.nn as nn

def diagnose_model(model, sample_input, target, criterion):
    """Run comprehensive diagnostics on a model."""
    print("=" * 60)
    print("NEURAL NETWORK DIAGNOSTIC REPORT")
    print("=" * 60)
    
    # 1. Parameter count
    total_params = sum(p.numel() for p in model.parameters())
    trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
    print(f"\n[1] Parameters: {total_params:,} total, {trainable:,} trainable")
    
    # 2. Forward pass check
    model.eval()
    with torch.no_grad():
        output = model(sample_input)
    print(f"\n[2] Input shape:  {sample_input.shape}")
    print(f"    Output shape: {output.shape}")
    print(f"    Output range: [{output.min():.4f}, {output.max():.4f}]")
    print(f"    Output has NaN: {torch.isnan(output).any()}")
    
    # 3. Initial loss check
    model.train()
    loss = criterion(output, target)
    n_classes = output.shape[-1]
    expected_loss = -torch.log(torch.tensor(1.0 / n_classes))
    print(f"\n[3] Initial loss: {loss.item():.4f}")
    print(f"    Expected (random): {expected_loss.item():.4f}")
    if abs(loss.item() - expected_loss.item()) > 0.5:
        print("    ⚠ Initial loss deviates significantly from expected!")
    
    # 4. Gradient flow check
    loss.backward()
    print(f"\n[4] Gradient flow check:")
    for name, param in model.named_parameters():
        if param.grad is None:
            print(f"    ⚠ {name}: NO GRADIENT")
        elif param.grad.norm() == 0:
            print(f"    ⚠ {name}: gradient is ZERO")
        else:
            print(f"    ✓ {name}: grad_norm={param.grad.norm():.6f}")
    
    # 5. Single batch overfit test
    print(f"\n[5] Single-batch overfit test (100 steps):")
    optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
    for step in range(100):
        optimizer.zero_grad()
        output = model(sample_input)
        loss = criterion(output, target)
        loss.backward()
        optimizer.step()
        
        if step in [0, 9, 49, 99]:
            acc = (output.argmax(1) == target).float().mean()
            print(f"    Step {step+1:3d}: loss={loss.item():.6f}, acc={acc:.4f}")
    
    final_acc = (model(sample_input).argmax(1) == target).float().mean()
    if final_acc < 0.95:
        print("    ⚠ Could NOT overfit single batch! Likely a bug.")
    else:
        print("    ✓ Successfully overfit single batch!")
    
    print("\n" + "=" * 60)

# Usage:
# diagnose_model(model, X_batch, y_batch, nn.CrossEntropyLoss())
```

---

## 7. Problem → Solution Quick Reference

```
  ┌─────────────────────────────────────────────────────────────────────────┐
  │  PROBLEM                     │ FIRST THING TO TRY                      │
  ├──────────────────────────────┼──────────────────────────────────────────┤
  │  Loss = NaN                  │ Reduce LR by 10×, add gradient clipping│
  │  Loss doesn't decrease       │ Increase LR by 10×, check data labels  │
  │  Loss decreases then spikes │ Reduce LR, add grad clipping           │
  │  Loss oscillates wildly     │ Reduce LR, increase batch size         │
  │  Train acc high, val acc low│ Add dropout, weight decay, get more data│
  │  Both acc stuck at low      │ Bigger model, higher LR, check data    │
  │  Accuracy = 1/num_classes   │ Model outputting same thing for all     │
  │  Accuracy drops suddenly    │ LR too high, bad data batch             │
  │  Very slow convergence      │ Increase LR, use Adam instead of SGD   │
  │  GPU out of memory          │ Reduce batch size, use mixed precision  │
  │  Results not reproducible   │ Set all random seeds (see Chapter 6)    │
  │  Good on train, random val  │ Data leak, wrong preprocessing         │
  │  Loss = constant (e.g. 2.3)│ Model not learning: check all gradients │
  │  Accuracy = 50% (binary)   │ Labels might be random / shuffled      │
  └──────────────────────────────┴──────────────────────────────────────────┘
```

---

## 8. Applications of Debugging Skills

| Scenario | Key Debugging Focus |
|----------|-------------------|
| **Image Classification** | Data augmentation correctness, input normalization, pretrained model compatibility |
| **NLP** | Tokenization, padding, attention masks, vocabulary handling |
| **Object Detection** | Anchor box design, loss balancing (classification vs regression) |
| **GANs** | Mode collapse detection, generator/discriminator balance |
| **Reinforcement Learning** | Reward shaping, exploration debugging, value function accuracy |
| **Medical AI** | Class imbalance, data leakage between train/test patients |

---

## 9. Summary Table

| Issue | Symptom | Root Cause | Quick Fix |
|-------|---------|------------|-----------|
| **Loss flat** | No decrease | LR too low, dead network | Increase LR, check gradients |
| **NaN loss** | Loss becomes NaN | Numerical instability | Reduce LR, clip gradients |
| **Overfitting** | Train↓ Val↑ | Too much capacity | Dropout, weight decay, more data |
| **Underfitting** | Both losses high | Too little capacity | Bigger model, higher LR |
| **Oscillating** | Loss bounces | LR too high | Reduce LR, increase batch |
| **Slow training** | Barely improving | LR too low | LR finder, use Adam |
| **Double softmax** | Poor accuracy | Softmax + CrossEntropy | Remove manual softmax |
| **Mode bug** | Bad val metrics | train/eval mode wrong | model.train() / model.eval() |

---

## 10. Revision Questions

1. **Describe Andrej Karpathy's recipe for training neural networks.** Why is the "overfit a single batch" step so critical?

2. **Your model's loss stays constant at 2.302 (for a 10-class classification problem). What does this tell you, and what are three things you should check?**

3. **List five common code bugs in PyTorch training loops and explain how to fix each one.** Include the "double softmax" bug.

4. **You observe NaN values in your loss after 500 training steps. Describe a systematic procedure to identify the source of the NaN.** Include at least three possible causes.

5. **Write a Python function that takes a model, a data batch, and a loss function, and performs a comprehensive diagnostic check.** The function should verify: initial loss, gradient flow, and single-batch overfitting ability.

6. **Your model trains to 99% training accuracy but only 62% validation accuracy. Diagnose the problem and list at least four strategies to fix it, in order of expected impact.**

---

| [← 04 Training Monitoring](04-training-monitoring.md) | [Unit 8 Home](README.md) | [06 → Reproducibility](06-reproducibility.md) |
|:---|:---:|---:|
