# 1. Architecture Design Principles

> **Unit 10 · Building Neural Networks** — Practical guidelines for designing effective neural network architectures

---

## Chapter Overview

Designing a neural network architecture is more art than science — but there are **well-established principles** that dramatically increase your chances of success. The key philosophy is: **start simple, verify it works, then add complexity.** This chapter covers how to match architecture to problem type, how to design input and output layers for different tasks, rules of thumb for hidden layer sizing, progressive network design, and a practical workflow you can follow for any new project.

---

## 1. The Fundamental Principle: Start Simple

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  "Start with the simplest model that could possibly work,  │
  │   then add complexity only when needed."                     │
  │                                                              │
  │  Why?                                                        │
  │  1. Simple models are EASIER TO DEBUG                       │
  │  2. Simple models give you a BASELINE to compare against    │
  │  3. Simple models train FASTER (iterate quickly)            │
  │  4. You might be SURPRISED how well a simple model works   │
  │  5. Complex models without baselines are UNINTERPRETABLE    │
  │                                                              │
  │  The progression:                                            │
  │                                                              │
  │  Logistic Regression → 1-layer MLP → 2-layer MLP → Deep   │
  │  "Can a linear model  "Does adding    "Do more    "Now     │
  │   do this?"           nonlinearity    layers      add task- │
  │                       help?"          help?"      specific  │
  │                                                   tricks"   │
  └──────────────────────────────────────────────────────────────┘
```

---

## 2. Match Architecture to Problem

### Architecture Selection Guide

```
  ┌──────────────────────────────────────────────────────────────┐
  │  DATA TYPE           │  ARCHITECTURE                        │
  ├──────────────────────┼──────────────────────────────────────┤
  │  Tabular data        │  MLP (fully connected)               │
  │  (spreadsheets,      │  • Start with 2-3 hidden layers     │
  │   structured data)   │  • Consider: XGBoost first!         │
  ├──────────────────────┼──────────────────────────────────────┤
  │  Images              │  CNN (Convolutional)                 │
  │  (photos, medical    │  • Conv layers → Pool → FC layers  │
  │   scans, satellite)  │  • Use pretrained (ResNet, etc.)    │
  ├──────────────────────┼──────────────────────────────────────┤
  │  Text / Sequences    │  Transformer / RNN                   │
  │  (NLP, time series)  │  • Transformers dominate NLP now    │
  │                      │  • LSTM/GRU for simple sequences    │
  ├──────────────────────┼──────────────────────────────────────┤
  │  Audio               │  CNN on spectrograms or             │
  │  (speech, music)     │  Transformer (Whisper, etc.)        │
  ├──────────────────────┼──────────────────────────────────────┤
  │  Graphs              │  GNN (Graph Neural Network)          │
  │  (social networks,   │  • GCN, GraphSAGE, GAT             │
  │   molecules)         │                                      │
  ├──────────────────────┼──────────────────────────────────────┤
  │  Multi-modal         │  Hybrid architectures                │
  │  (image + text)      │  • Separate encoders → fusion      │
  └──────────────────────┴──────────────────────────────────────┘
```

---

## 3. Input Layer Design

### Different Task Types

```
  ┌──────────────────────────────────────────────────────────────┐
  │  TASK                    │ INPUT SHAPE      │ PREPROCESSING  │
  ├──────────────────────────┼──────────────────┼────────────────┤
  │  Tabular classification  │ (batch, features)│ Normalize/     │
  │  (e.g., 20 features)    │ (B, 20)         │ Standardize    │
  ├──────────────────────────┼──────────────────┼────────────────┤
  │  Image (grayscale)       │ (B, 1, H, W)    │ Scale to [0,1] │
  │  (e.g., MNIST 28×28)    │ (B, 1, 28, 28)  │ or normalize   │
  ├──────────────────────────┼──────────────────┼────────────────┤
  │  Image (color)           │ (B, 3, H, W)    │ ImageNet norm  │
  │  (e.g., CIFAR 32×32)    │ (B, 3, 32, 32)  │ per-channel    │
  ├──────────────────────────┼──────────────────┼────────────────┤
  │  Text (tokens)           │ (B, seq_len)     │ Tokenize →     │
  │                          │ integers         │ Embedding      │
  ├──────────────────────────┼──────────────────┼────────────────┤
  │  Time series             │ (B, time, feat)  │ Normalize per  │
  │                          │ (B, T, F)        │ feature        │
  └──────────────────────────┴──────────────────┴────────────────┘
  
  KEY PRINCIPLE: Always normalize/standardize inputs!
  
  Raw pixels [0-255] → Normalized [0-1] or standardized (μ=0, σ=1)
  Raw features → StandardScaler or MinMaxScaler
```

---

## 4. Output Layer Design

### Matching Output to Task

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │  TASK                │ OUTPUT     │ ACTIVATION  │ LOSS              │
  │                      │ NEURONS    │             │                   │
  ├──────────────────────┼────────────┼─────────────┼───────────────────┤
  │  Binary classif.     │ 1          │ Sigmoid     │ BCEWithLogitsLoss │
  │  (spam / not spam)   │            │ (or none)   │                   │
  ├──────────────────────┼────────────┼─────────────┼───────────────────┤
  │  Multi-class (C cls) │ C          │ None (logits)│ CrossEntropyLoss │
  │  (digit 0-9)         │ 10         │ (Softmax    │ (includes softmax)│
  │                      │            │  in loss)   │                   │
  ├──────────────────────┼────────────┼─────────────┼───────────────────┤
  │  Multi-label          │ C          │ Sigmoid     │ BCEWithLogitsLoss │
  │  (tags: dog,outdoor)  │ per label  │ (per label) │ (per label)       │
  ├──────────────────────┼────────────┼─────────────┼───────────────────┤
  │  Regression          │ 1          │ None        │ MSELoss           │
  │  (predict price)     │            │ (linear)    │ or L1Loss         │
  ├──────────────────────┼────────────┼─────────────┼───────────────────┤
  │  Multi-output regr.  │ D          │ None        │ MSELoss           │
  │  (predict x,y,z)     │ 3          │ (linear)    │                   │
  ├──────────────────────┼────────────┼─────────────┼───────────────────┤
  │  Bounded regression  │ 1          │ Sigmoid     │ MSELoss           │
  │  (predict 0-1)       │            │ (bounds     │ (on transformed)  │
  │                      │            │  output)    │                   │
  └──────────────────────┴────────────┴─────────────┴───────────────────┘
```

```python
import torch.nn as nn

# Binary classification
output_layer = nn.Linear(hidden_size, 1)
criterion = nn.BCEWithLogitsLoss()  # includes sigmoid

# Multi-class (10 classes)
output_layer = nn.Linear(hidden_size, 10)
criterion = nn.CrossEntropyLoss()  # includes softmax

# Regression
output_layer = nn.Linear(hidden_size, 1)
criterion = nn.MSELoss()

# Multi-label (5 possible labels)
output_layer = nn.Linear(hidden_size, 5)
criterion = nn.BCEWithLogitsLoss()  # sigmoid per output
```

---

## 5. Hidden Layer Sizing Rules of Thumb

```
  ┌──────────────────────────────────────────────────────────────┐
  │  RULES OF THUMB FOR HIDDEN LAYER SIZE                       │
  │  (starting points — always tune!)                           │
  │                                                              │
  │  Rule 1: Between input and output size                      │
  │  ──────                                                      │
  │  hidden ≈ √(input_size × output_size)                       │
  │  e.g., input=784, output=10 → hidden ≈ √7840 ≈ 89 → 128   │
  │                                                              │
  │  Rule 2: Powers of 2                                        │
  │  ──────                                                      │
  │  Use 64, 128, 256, 512, 1024                               │
  │  Reason: GPU tensor cores work best with multiples of 8/16  │
  │                                                              │
  │  Rule 3: Funnel shape                                       │
  │  ──────                                                      │
  │  Each layer slightly narrower than the previous             │
  │  784 → 512 → 256 → 128 → 10                               │
  │  (compresses information hierarchically)                    │
  │                                                              │
  │  Rule 4: First hidden layer ≈ 2× to 4× output size        │
  │  ──────                                                      │
  │  For 10-class: first hidden = 32 to 256                    │
  │                                                              │
  │  Rule 5: Don't overthink it!                                │
  │  ──────                                                      │
  │  Start with 128 or 256, adjust based on:                    │
  │  • Overfitting → reduce size                               │
  │  • Underfitting → increase size                            │
  └──────────────────────────────────────────────────────────────┘
```

### Common Architecture Patterns

```
  Pattern 1: CONSTANT WIDTH
  ┌──────┐
  │ 784  │  Input
  ├──────┤
  │ 256  │  Hidden 1
  ├──────┤
  │ 256  │  Hidden 2      All hidden layers same size
  ├──────┤
  │ 256  │  Hidden 3      Simple, works well
  ├──────┤
  │  10  │  Output
  └──────┘
  
  Pattern 2: FUNNEL (DECREASING)
  ┌──────┐
  │ 784  │  Input
  ├──────┤
  │ 512  │  Hidden 1
  ├──────┤
  │ 256  │  Hidden 2      Progressively compress
  ├──────┤
  │ 128  │  Hidden 3      Most popular pattern
  ├──────┤
  │  10  │  Output
  └──────┘
  
  Pattern 3: DIAMOND
  ┌──────┐
  │ 784  │  Input
  ├──────┤
  │ 512  │  Hidden 1
  ├──────┤
  │1024  │  Hidden 2      Expand then compress
  ├──────┤
  │ 256  │  Hidden 3      Used in some architectures
  ├──────┤
  │  10  │  Output
  └──────┘
  
  Pattern 4: BOTTLENECK
  ┌──────┐
  │ 784  │  Input
  ├──────┤
  │  64  │  Bottleneck    Force information compression
  ├──────┤                (like autoencoder)
  │ 256  │  Expansion
  ├──────┤
  │  10  │  Output
  └──────┘
```

---

## 6. Progressive Network Design

### Step-by-Step Approach

```
  ┌──────────────────────────────────────────────────────────────┐
  │  PROGRESSIVE DESIGN WORKFLOW                                 │
  │                                                              │
  │  Step 1: BASELINE (no neural network!)                      │
  │  ├── Logistic regression / Random forest                    │
  │  ├── Establish minimum acceptable performance               │
  │  └── If this works well enough → don't use DL!             │
  │                                                              │
  │  Step 2: SIMPLEST NEURAL NETWORK                            │
  │  ├── 1 hidden layer, 128 neurons, ReLU                     │
  │  ├── Adam optimizer, lr=1e-3                                │
  │  ├── No regularization yet                                  │
  │  └── Can it overfit the training data?                      │
  │                                                              │
  │  Step 3: ADD DEPTH                                          │
  │  ├── 2-3 hidden layers                                      │
  │  ├── Does validation performance improve?                   │
  │  └── If yes → try 4-5 layers                               │
  │                                                              │
  │  Step 4: ADD WIDTH                                          │
  │  ├── Try doubling hidden size (128 → 256)                  │
  │  ├── Does it help or cause overfitting?                     │
  │  └── Find the sweet spot                                    │
  │                                                              │
  │  Step 5: ADD REGULARIZATION                                 │
  │  ├── Batch normalization (almost always helps)              │
  │  ├── Dropout (0.2-0.5)                                      │
  │  ├── Weight decay (1e-5 to 1e-3)                           │
  │  └── Target: train_acc ≈ val_acc                           │
  │                                                              │
  │  Step 6: TUNE HYPERPARAMETERS                               │
  │  ├── LR range test → one-cycle policy                      │
  │  ├── Batch size experiments                                 │
  │  └── Optuna for remaining hyperparameters                   │
  │                                                              │
  │  Step 7: ADVANCED TECHNIQUES                                │
  │  ├── Skip connections (if deep)                             │
  │  ├── Learning rate scheduling                               │
  │  ├── Data augmentation                                      │
  │  └── Transfer learning (if applicable)                      │
  └──────────────────────────────────────────────────────────────┘
```

---

## 7. Architecture Search Overview

### Neural Architecture Search (NAS)

```
  NAS: Automatically find the best architecture
  
  ┌──────────────────────────────────────────────────────────────┐
  │  SEARCH SPACE         →  What architectures to consider     │
  │  (layers, connections,   (all possible combinations)         │
  │   activations, widths)                                       │
  │                                                              │
  │  SEARCH STRATEGY      →  How to explore the space           │
  │  (random, evolutionary,  (which architectures to try)       │
  │   reinforcement learning,                                    │
  │   gradient-based)                                            │
  │                                                              │
  │  EVALUATION STRATEGY  →  How to measure each architecture  │
  │  (full training,         (train and check performance)      │
  │   weight sharing,                                            │
  │   one-shot)                                                  │
  └──────────────────────────────────────────────────────────────┘
  
  NAS Found Architectures:
  • EfficientNet (Google, 2019) — state-of-the-art image classification
  • NASNet (Google, 2017) — competitive with hand-designed models
  • DARTS (CMU, 2019) — differentiable architecture search
  
  ⚠ NAS is EXPENSIVE:
  Original NAS: 1800 GPU-days ($$$)
  Efficient NAS (ENAS, DARTS): 1-4 GPU-days
  
  For most practitioners: use proven architectures!
  NAS is for framework/model developers, not typical users.
```

---

## 8. Practical Design Workflow (Complete)

```python
import torch
import torch.nn as nn
import torch.optim as optim

# ═══════════════════════════════════════════════
#  STEP 1: Start with the simplest model
# ═══════════════════════════════════════════════

class SimpleModel(nn.Module):
    """Start here. If this doesn't work, something else is wrong."""
    def __init__(self, input_size, num_classes):
        super().__init__()
        self.net = nn.Sequential(
            nn.Flatten(),
            nn.Linear(input_size, 128),
            nn.ReLU(),
            nn.Linear(128, num_classes)
        )
    
    def forward(self, x):
        return self.net(x)

# ═══════════════════════════════════════════════
#  STEP 2: If underfitting, add depth and width
# ═══════════════════════════════════════════════

class MediumModel(nn.Module):
    """More capacity. Use if SimpleModel underfits."""
    def __init__(self, input_size, num_classes):
        super().__init__()
        self.net = nn.Sequential(
            nn.Flatten(),
            nn.Linear(input_size, 512),
            nn.ReLU(),
            nn.Linear(512, 256),
            nn.ReLU(),
            nn.Linear(256, 128),
            nn.ReLU(),
            nn.Linear(128, num_classes)
        )
    
    def forward(self, x):
        return self.net(x)

# ═══════════════════════════════════════════════
#  STEP 3: If overfitting, add regularization
# ═══════════════════════════════════════════════

class RegularizedModel(nn.Module):
    """Add BN + Dropout. Use if MediumModel overfits."""
    def __init__(self, input_size, num_classes, dropout=0.3):
        super().__init__()
        self.net = nn.Sequential(
            nn.Flatten(),
            nn.Linear(input_size, 512),
            nn.BatchNorm1d(512),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(512, 256),
            nn.BatchNorm1d(256),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(256, 128),
            nn.BatchNorm1d(128),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(128, num_classes)
        )
    
    def forward(self, x):
        return self.net(x)


# ═══════════════════════════════════════════════
#  Evaluation function to compare architectures
# ═══════════════════════════════════════════════

def evaluate_architecture(ModelClass, input_size, num_classes, 
                          train_loader, val_loader, epochs=20):
    """Train and evaluate a model architecture."""
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    model = ModelClass(input_size, num_classes).to(device)
    
    n_params = sum(p.numel() for p in model.parameters())
    print(f"\n{ModelClass.__name__}: {n_params:,} parameters")
    
    optimizer = optim.Adam(model.parameters(), lr=1e-3)
    criterion = nn.CrossEntropyLoss()
    
    for epoch in range(epochs):
        model.train()
        for X, y in train_loader:
            X, y = X.to(device), y.to(device)
            optimizer.zero_grad()
            loss = criterion(model(X), y)
            loss.backward()
            optimizer.step()
    
    # Evaluate
    model.eval()
    correct = total = 0
    with torch.no_grad():
        for X, y in val_loader:
            X, y = X.to(device), y.to(device)
            correct += (model(X).argmax(1) == y).sum().item()
            total += y.size(0)
    
    acc = correct / total
    print(f"  Val accuracy: {acc:.4f}")
    return acc

# Compare architectures:
for Model in [SimpleModel, MediumModel, RegularizedModel]:
    evaluate_architecture(Model, 784, 10, train_loader, val_loader)
```

---

## 9. Summary Table

| Principle | Guideline | Why |
|-----------|-----------|-----|
| **Start simple** | Linear → 1 layer → 2 layers → ... | Easier to debug, gives baseline |
| **Match arch to data** | MLP for tabular, CNN for images, etc. | Domain structure matters |
| **Output matches task** | 1 neuron (binary), C neurons (multi-class) | Loss function compatibility |
| **Normalize inputs** | [0,1] or μ=0, σ=1 | Faster, more stable training |
| **Powers of 2 for width** | 64, 128, 256, 512 | GPU optimization |
| **Funnel or constant** | Decreasing or equal layer sizes | Information compression |
| **BN + Dropout** | Add after seeing overfitting | Regularization |
| **Overfit first** | Single batch → full data → regularize | Verify model works |

---

## 10. Revision Questions

1. **Why should you "start simple and add complexity" when designing neural network architectures?** Give a concrete example of how this workflow would look for a new image classification task.

2. **Design the output layer and loss function for each of these tasks:** (a) predicting if an email is spam, (b) classifying an image as one of 100 categories, (c) predicting house price, (d) tagging an image with multiple labels.

3. **State three rules of thumb for choosing hidden layer sizes.** For a network with 1000 input features and 5 output classes, what hidden layer sizes would you try first?

4. **Compare the "funnel," "constant," "diamond," and "bottleneck" architecture patterns.** When would you use each?

5. **What is Neural Architecture Search (NAS)?** Why is it impractical for most practitioners, and what should they do instead?

6. **Describe the complete progressive network design workflow, from baseline through advanced techniques.** At each step, what signals tell you whether to add more complexity or more regularization?

---

| [← Unit 9: Deep Learning Frameworks](../09-Deep-Learning-Frameworks/) | [Unit 10 Home](README.md) | [02 → Depth vs Width](02-network-depth-vs-width.md) |
|:---|:---:|---:|
