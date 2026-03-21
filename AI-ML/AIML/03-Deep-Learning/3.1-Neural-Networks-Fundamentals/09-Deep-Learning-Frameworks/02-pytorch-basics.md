# 2. PyTorch Basics

> **Unit 9 · Deep Learning Frameworks** — Building neural networks with define-by-run philosophy and full Python control

---

## Chapter Overview

**PyTorch** is Facebook/Meta's open-source deep learning framework, beloved by researchers for its **Pythonic feel** and **define-by-run** (eager execution) approach. Unlike TensorFlow's original graph-based paradigm, PyTorch builds the computational graph dynamically as your code runs — making debugging as natural as debugging any Python program. This chapter covers PyTorch's core concepts (tensors, autograd, nn.Module), the canonical training loop, DataLoader/Dataset abstractions, and a complete end-to-end example with comparison to Keras.

---

## 1. PyTorch Philosophy: Define-by-Run

```
  DEFINE-AND-RUN (TensorFlow v1 style):
  ┌──────────────────────────────────────────────────┐
  │  1. Define computation graph (symbolic)          │
  │  2. Compile the graph                            │
  │  3. Feed data through the compiled graph         │
  │  → Hard to debug (graph ≠ Python code)          │
  └──────────────────────────────────────────────────┘
  
  DEFINE-BY-RUN (PyTorch style):
  ┌──────────────────────────────────────────────────┐
  │  1. Write normal Python code                     │
  │  2. Code IS the graph — built on-the-fly        │
  │  3. Use pdb, print(), breakpoints normally!      │
  │  → Natural Python debugging ✓                   │
  └──────────────────────────────────────────────────┘
  
  Example:
  
  # This IS the computation graph:
  x = torch.randn(32, 784)        # create input tensor
  h = torch.relu(W1 @ x + b1)     # hidden layer
  if h.mean() > 0:                 # ← dynamic! impossible in static graph
      h = torch.relu(W2 @ h + b2)
  y = W3 @ h + b3                  # output
  
  The graph is built AS the code runs.
  Different inputs can create DIFFERENT graphs!
```

---

## 2. Tensors

### Creating Tensors

```python
import torch

# From Python data
x = torch.tensor([1.0, 2.0, 3.0])
X = torch.tensor([[1, 2], [3, 4]], dtype=torch.float32)

# Common constructors
zeros = torch.zeros(3, 4)           # 3×4 matrix of zeros
ones = torch.ones(3, 4)             # 3×4 matrix of ones
rand = torch.randn(3, 4)            # 3×4 from N(0,1)
arange = torch.arange(0, 10, 2)     # [0, 2, 4, 6, 8]
eye = torch.eye(3)                   # 3×3 identity matrix

# From NumPy (shared memory!)
import numpy as np
np_array = np.array([1.0, 2.0, 3.0])
tensor = torch.from_numpy(np_array)  # shares memory with np_array
back_to_np = tensor.numpy()          # shares memory with tensor

print(f"Shape: {rand.shape}")        # torch.Size([3, 4])
print(f"Dtype: {rand.dtype}")        # torch.float32
print(f"Device: {rand.device}")      # cpu
```

### Tensor Operations

```python
# Arithmetic
a = torch.tensor([1.0, 2.0, 3.0])
b = torch.tensor([4.0, 5.0, 6.0])
print(a + b)          # tensor([5., 7., 9.])
print(a * b)          # element-wise: tensor([4., 10., 18.])
print(a @ b)          # dot product: tensor(32.)

# Matrix operations
W = torch.randn(10, 784)
x = torch.randn(32, 784)  # batch of 32
out = x @ W.T              # (32, 784) @ (784, 10) → (32, 10)

# Reshaping
x = torch.randn(2, 3, 4)
print(x.view(2, 12).shape)      # (2, 12) — must be contiguous
print(x.reshape(6, 4).shape)    # (6, 4) — always works
print(x.permute(2, 0, 1).shape) # (4, 2, 3) — reorder dimensions
print(x.unsqueeze(0).shape)     # (1, 2, 3, 4) — add dimension
print(x.squeeze().shape)        # removes dimensions of size 1
```

### GPU Tensors

```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

# Move tensor to GPU
x_gpu = x.to(device)
x_gpu = x.cuda()     # shorthand (if GPU available)

# Create directly on GPU
y_gpu = torch.randn(3, 4, device=device)

# Move back to CPU
x_cpu = x_gpu.cpu()
x_np = x_gpu.cpu().numpy()  # must be on CPU for numpy
```

---

## 3. Autograd: Automatic Differentiation

```
  Autograd: PyTorch's automatic differentiation engine
  
  ┌──────────────────────────────────────────────────────────────┐
  │  Forward pass: compute output from input                     │
  │  ────────────                                                │
  │  x (requires_grad=True) → operations → y = f(x)            │
  │                                                              │
  │  Backward pass: compute gradients automatically              │
  │  ─────────────                                               │
  │  y.backward() → computes dy/dx → stored in x.grad           │
  │                                                              │
  │  PyTorch records ALL operations on tensors with              │
  │  requires_grad=True, building a dynamic computation graph.  │
  └──────────────────────────────────────────────────────────────┘
```

```python
# Simple gradient computation
x = torch.tensor(3.0, requires_grad=True)
y = x ** 2 + 2 * x + 1   # y = x² + 2x + 1
y.backward()               # compute dy/dx
print(x.grad)              # tensor(8.) because dy/dx = 2x + 2 = 2(3) + 2 = 8

# With vectors
W = torch.randn(3, 4, requires_grad=True)
x = torch.randn(4)
y = W @ x                  # (3,)
loss = y.sum()              # scalar — backward requires scalar
loss.backward()
print(W.grad.shape)         # (3, 4) — same shape as W

# IMPORTANT: Gradients accumulate! Reset before each backward:
W.grad.zero_()  # or optimizer.zero_grad() in practice
```

### Computation Graph

```
  Example: y = (x * w + b).relu().sum()
  
  Forward:
  x ──┐
       ├── mul ──┐
  w ──┘         ├── add ── relu ── sum ── y (loss)
           b ──┘
  
  Backward (y.backward()):
  x.grad ←── mul_bwd ←── add_bwd ←── relu_bwd ←── sum_bwd ←── 1
  w.grad ←──┘             b.grad ←──┘
  
  Each node stores the backward function.
  y.backward() traverses the graph in reverse.
```

---

## 4. nn.Module: Defining Models

```python
import torch.nn as nn

class MLP(nn.Module):
    def __init__(self, input_size=784, hidden_size=256, num_classes=10, dropout=0.3):
        super().__init__()
        
        # Define layers
        self.flatten = nn.Flatten()
        self.layer1 = nn.Linear(input_size, hidden_size)
        self.bn1 = nn.BatchNorm1d(hidden_size)
        self.relu1 = nn.ReLU()
        self.drop1 = nn.Dropout(dropout)
        
        self.layer2 = nn.Linear(hidden_size, hidden_size // 2)
        self.bn2 = nn.BatchNorm1d(hidden_size // 2)
        self.relu2 = nn.ReLU()
        self.drop2 = nn.Dropout(dropout)
        
        self.output = nn.Linear(hidden_size // 2, num_classes)
    
    def forward(self, x):
        """Define the forward pass."""
        x = self.flatten(x)           # (B, 1, 28, 28) → (B, 784)
        
        x = self.layer1(x)            # (B, 784) → (B, 256)
        x = self.bn1(x)
        x = self.relu1(x)
        x = self.drop1(x)
        
        x = self.layer2(x)            # (B, 256) → (B, 128)
        x = self.bn2(x)
        x = self.relu2(x)
        x = self.drop2(x)
        
        x = self.output(x)            # (B, 128) → (B, 10)
        return x                       # return RAW LOGITS (no softmax!)


# Create model
model = MLP(input_size=784, hidden_size=256, num_classes=10)

# Inspect
print(model)
total_params = sum(p.numel() for p in model.parameters())
print(f"Total parameters: {total_params:,}")
```

### Using nn.Sequential

```python
# For simple linear stacks (like Keras Sequential):
model = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 256),
    nn.BatchNorm1d(256),
    nn.ReLU(),
    nn.Dropout(0.3),
    nn.Linear(256, 128),
    nn.BatchNorm1d(128),
    nn.ReLU(),
    nn.Dropout(0.3),
    nn.Linear(128, 10),
)
```

---

## 5. The Training Loop

Unlike Keras's `model.fit()`, PyTorch requires you to write the training loop explicitly. This gives you **complete control**.

```
  THE CANONICAL PYTORCH TRAINING LOOP:
  
  for each epoch:
  │
  ├── model.train()            ← enable dropout, BN training mode
  │
  ├── for each batch:
  │   │
  │   ├── 1. optimizer.zero_grad()    ← reset gradients
  │   │
  │   ├── 2. outputs = model(X)       ← forward pass
  │   │
  │   ├── 3. loss = criterion(outputs, y)  ← compute loss
  │   │
  │   ├── 4. loss.backward()          ← backward pass (compute gradients)
  │   │
  │   └── 5. optimizer.step()         ← update parameters
  │
  ├── model.eval()             ← disable dropout, BN eval mode
  │
  └── with torch.no_grad():   ← no gradient computation
      └── validation loop
```

```python
import torch
import torch.nn as nn
import torch.optim as optim

# Setup
model = MLP().to(device)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=1e-3, weight_decay=1e-4)

# Training loop
num_epochs = 30
for epoch in range(num_epochs):
    
    # ── Training Phase ──
    model.train()
    train_loss = 0
    train_correct = 0
    train_total = 0
    
    for X, y in train_loader:
        X, y = X.to(device), y.to(device)
        
        # The 5-step pattern:
        optimizer.zero_grad()           # 1. Zero gradients
        outputs = model(X)             # 2. Forward pass
        loss = criterion(outputs, y)   # 3. Compute loss
        loss.backward()                # 4. Backward pass
        optimizer.step()               # 5. Update weights
        
        # Track metrics
        train_loss += loss.item() * X.size(0)
        train_correct += (outputs.argmax(1) == y).sum().item()
        train_total += y.size(0)
    
    # ── Validation Phase ──
    model.eval()
    val_loss = 0
    val_correct = 0
    val_total = 0
    
    with torch.no_grad():  # no gradient computation (saves memory + speed)
        for X, y in val_loader:
            X, y = X.to(device), y.to(device)
            outputs = model(X)
            loss = criterion(outputs, y)
            
            val_loss += loss.item() * X.size(0)
            val_correct += (outputs.argmax(1) == y).sum().item()
            val_total += y.size(0)
    
    # ── Print Progress ──
    print(f"Epoch {epoch+1:2d}/{num_epochs} | "
          f"Train Loss: {train_loss/train_total:.4f} | "
          f"Train Acc: {train_correct/train_total:.4f} | "
          f"Val Loss: {val_loss/val_total:.4f} | "
          f"Val Acc: {val_correct/val_total:.4f}")
```

### Why the 5-Step Order Matters

```
  ┌──────────────────────────────────────────────────────────────┐
  │  STEP                   │ WHAT HAPPENS                      │
  ├─────────────────────────┼───────────────────────────────────┤
  │  1. optimizer.zero_grad()│ Clear gradients from last step   │
  │                          │ Without this: gradients ACCUMULATE│
  │                          │ across batches (usually a bug!)  │
  │                          │                                   │
  │  2. outputs = model(X)  │ Forward pass through the network │
  │                          │ Builds computation graph          │
  │                          │ Dropout active (model.train())   │
  │                          │                                   │
  │  3. loss = criterion()  │ Compute scalar loss value        │
  │                          │ This is the root of the graph    │
  │                          │                                   │
  │  4. loss.backward()     │ Backpropagate through graph      │
  │                          │ Fills param.grad for all params  │
  │                          │ Graph is destroyed after this    │
  │                          │                                   │
  │  5. optimizer.step()    │ Update params using gradients    │
  │                          │ θ = θ - lr * θ.grad (for SGD)   │
  └─────────────────────────┴───────────────────────────────────┘
```

---

## 6. Dataset and DataLoader

### Custom Dataset

```python
from torch.utils.data import Dataset, DataLoader

class CustomDataset(Dataset):
    def __init__(self, data, labels, transform=None):
        self.data = data
        self.labels = labels
        self.transform = transform
    
    def __len__(self):
        """Return total number of samples."""
        return len(self.data)
    
    def __getitem__(self, idx):
        """Return one sample (x, y) at index idx."""
        x = self.data[idx]
        y = self.labels[idx]
        
        if self.transform:
            x = self.transform(x)
        
        return x, y

# Create dataset
dataset = CustomDataset(X_train, y_train)
print(f"Dataset size: {len(dataset)}")
print(f"Sample: {dataset[0][0].shape}, {dataset[0][1]}")
```

### DataLoader

```python
train_loader = DataLoader(
    dataset,
    batch_size=128,
    shuffle=True,          # shuffle every epoch
    num_workers=4,         # parallel data loading
    pin_memory=True,       # speed up CPU→GPU transfer
    drop_last=True,        # drop incomplete last batch
)

# Iterating
for batch_idx, (X, y) in enumerate(train_loader):
    print(f"Batch {batch_idx}: X.shape={X.shape}, y.shape={y.shape}")
    break
# Output: Batch 0: X.shape=torch.Size([128, 1, 28, 28]), y.shape=torch.Size([128])
```

### Using torchvision Datasets

```python
from torchvision import datasets, transforms

transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])

train_data = datasets.MNIST('./data', train=True, download=True, transform=transform)
test_data = datasets.MNIST('./data', train=False, transform=transform)

train_loader = DataLoader(train_data, batch_size=128, shuffle=True, num_workers=4)
test_loader = DataLoader(test_data, batch_size=512)
```

---

## 7. Complete Example: MNIST MLP in PyTorch

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, random_split
from torchvision import datasets, transforms

# ═══════════════════════════════════════════════════════════
#  1. SETUP
# ═══════════════════════════════════════════════════════════

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")

# ═══════════════════════════════════════════════════════════
#  2. DATA
# ═══════════════════════════════════════════════════════════

transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])

full_train = datasets.MNIST('./data', train=True, download=True, transform=transform)
test_data = datasets.MNIST('./data', train=False, transform=transform)

train_data, val_data = random_split(full_train, [50000, 10000])

train_loader = DataLoader(train_data, batch_size=128, shuffle=True, num_workers=4)
val_loader = DataLoader(val_data, batch_size=512)
test_loader = DataLoader(test_data, batch_size=512)

# ═══════════════════════════════════════════════════════════
#  3. MODEL
# ═══════════════════════════════════════════════════════════

class MLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.network = nn.Sequential(
            nn.Flatten(),
            nn.Linear(784, 256),
            nn.BatchNorm1d(256),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(256, 128),
            nn.BatchNorm1d(128),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(128, 10),
        )
    
    def forward(self, x):
        return self.network(x)

model = MLP().to(device)
print(f"Parameters: {sum(p.numel() for p in model.parameters()):,}")

# ═══════════════════════════════════════════════════════════
#  4. TRAINING SETUP
# ═══════════════════════════════════════════════════════════

criterion = nn.CrossEntropyLoss()
optimizer = optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)
scheduler = optim.lr_scheduler.ReduceLROnPlateau(
    optimizer, mode='min', factor=0.5, patience=5
)

# ═══════════════════════════════════════════════════════════
#  5. TRAINING LOOP
# ═══════════════════════════════════════════════════════════

best_val_acc = 0
patience = 10
patience_counter = 0

for epoch in range(100):
    # Training
    model.train()
    train_loss = 0
    train_correct = 0
    
    for X, y in train_loader:
        X, y = X.to(device), y.to(device)
        optimizer.zero_grad()
        out = model(X)
        loss = criterion(out, y)
        loss.backward()
        optimizer.step()
        train_loss += loss.item() * X.size(0)
        train_correct += (out.argmax(1) == y).sum().item()
    
    # Validation
    model.eval()
    val_loss = 0
    val_correct = 0
    
    with torch.no_grad():
        for X, y in val_loader:
            X, y = X.to(device), y.to(device)
            out = model(X)
            loss = criterion(out, y)
            val_loss += loss.item() * X.size(0)
            val_correct += (out.argmax(1) == y).sum().item()
    
    train_acc = train_correct / len(train_data)
    val_acc = val_correct / len(val_data)
    avg_val_loss = val_loss / len(val_data)
    
    scheduler.step(avg_val_loss)
    
    # Early stopping
    if val_acc > best_val_acc:
        best_val_acc = val_acc
        patience_counter = 0
        torch.save(model.state_dict(), 'best_model.pth')
    else:
        patience_counter += 1
        if patience_counter >= patience:
            print(f"Early stopping at epoch {epoch+1}")
            break
    
    lr = optimizer.param_groups[0]['lr']
    print(f"Epoch {epoch+1:3d} | Train Acc: {train_acc:.4f} | "
          f"Val Acc: {val_acc:.4f} | Val Loss: {avg_val_loss:.4f} | LR: {lr:.6f}")

# ═══════════════════════════════════════════════════════════
#  6. EVALUATE ON TEST SET
# ═══════════════════════════════════════════════════════════

model.load_state_dict(torch.load('best_model.pth'))
model.eval()
test_correct = 0

with torch.no_grad():
    for X, y in test_loader:
        X, y = X.to(device), y.to(device)
        test_correct += (model(X).argmax(1) == y).sum().item()

test_acc = test_correct / len(test_data)
print(f"\nTest Accuracy: {test_acc:.4f}")
```

---

## 8. Comparison: PyTorch vs Keras

```
  ┌──────────────────────┬──────────────────────┬──────────────────────┐
  │  CONCEPT             │  KERAS               │  PYTORCH             │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Model definition    │  Sequential/         │  nn.Module +         │
  │                      │  Functional/Subclass │  forward()           │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Training            │  model.fit()         │  Manual loop:        │
  │                      │  (one line!)         │  zero_grad→forward→  │
  │                      │                      │  loss→backward→step  │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Evaluation          │  model.evaluate()    │  Manual loop with    │
  │                      │                      │  torch.no_grad()     │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Prediction          │  model.predict()     │  model(x) with       │
  │                      │                      │  torch.no_grad()     │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Callbacks           │  Built-in system     │  Write your own      │
  │                      │                      │  (or use libraries)  │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Data pipeline       │  tf.data.Dataset     │  DataLoader +        │
  │                      │  or numpy arrays     │  Dataset class       │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  GPU placement       │  Automatic           │  Manual .to(device)  │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Loss function       │  Compile argument    │  Separate object     │
  │                      │  (strings accepted)  │  (must instantiate)  │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Softmax             │  In last layer       │  In loss function    │
  │                      │  (model outputs      │  (model outputs      │
  │                      │   probabilities)     │   raw logits)        │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Lines of code       │  ~20 lines           │  ~50 lines           │
  │  (basic training)    │                      │                      │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Control             │  Less (abstracted)   │  More (explicit)     │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │  Debugging           │  Harder (abstracted) │  Easier (explicit)   │
  └──────────────────────┴──────────────────────┴──────────────────────┘
```

### Key Softmax Difference

```
  KERAS convention:
  ┌────────────────────────────────────────────────────────────┐
  │  Last layer: Dense(10, activation='softmax')              │
  │  Output: probabilities [0.1, 0.05, 0.7, ...]            │
  │  Loss: 'sparse_categorical_crossentropy'                  │
  └────────────────────────────────────────────────────────────┘
  
  PYTORCH convention:
  ┌────────────────────────────────────────────────────────────┐
  │  Last layer: nn.Linear(128, 10)  (NO softmax!)           │
  │  Output: raw logits [2.1, -0.5, 4.3, ...]               │
  │  Loss: nn.CrossEntropyLoss()  (applies softmax internally)│
  └────────────────────────────────────────────────────────────┘
  
  ⚠ CRITICAL: In PyTorch, do NOT put softmax in your model
  if using CrossEntropyLoss. This is the #1 beginner mistake!
```

---

## 9. Summary Table

| Concept | PyTorch Implementation | Key Note |
|---------|----------------------|----------|
| **Create model** | `class Model(nn.Module)` | Define `__init__` and `forward` |
| **Forward pass** | `output = model(x)` | Calls `forward()` internally |
| **Loss function** | `nn.CrossEntropyLoss()` | Includes softmax! |
| **Optimizer** | `optim.Adam(model.parameters())` | Pass model params |
| **Zero gradients** | `optimizer.zero_grad()` | MUST call each iteration |
| **Backward pass** | `loss.backward()` | Fills `.grad` on all params |
| **Update weights** | `optimizer.step()` | Uses `.grad` to update |
| **Train mode** | `model.train()` | Enables dropout/BN training |
| **Eval mode** | `model.eval()` | Disables dropout/BN eval |
| **No gradients** | `with torch.no_grad():` | For validation/inference |
| **GPU** | `model.to(device)`, `x.to(device)` | Manual placement |
| **Data** | `DataLoader(Dataset, batch_size)` | Parallel loading |

---

## 10. Revision Questions

1. **Explain PyTorch's "define-by-run" philosophy.** How does it differ from TensorFlow's original graph-based approach? What advantage does it give for debugging?

2. **Write the 5-step PyTorch training loop** and explain what each step does. Why is the order important? What happens if you forget `optimizer.zero_grad()`?

3. **What is `torch.no_grad()` and why should you use it during validation?** What happens if you don't use it?

4. **Compare how Keras and PyTorch handle softmax in classification.** Why does PyTorch's `CrossEntropyLoss` include softmax, and what common bug does this convention cause?

5. **Write a complete PyTorch program** that defines a 3-layer MLP, loads MNIST, trains for 20 epochs with Adam, and prints training and validation accuracy each epoch.

6. **Explain the purpose of `model.train()` and `model.eval()`.** Which layers behave differently between these modes, and how?

---

| [← 01 TensorFlow & Keras](01-tensorflow-keras-basics.md) | [Unit 9 Home](README.md) | [03 → Framework Comparison](03-framework-comparison.md) |
|:---|:---:|---:|
