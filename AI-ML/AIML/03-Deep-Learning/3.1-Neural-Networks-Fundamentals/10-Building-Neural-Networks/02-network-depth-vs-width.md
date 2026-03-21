# 2. Network Depth vs Width

> **Unit 10 · Building Neural Networks** — Understanding the tradeoffs between deep and wide networks

---

## Chapter Overview

When designing a neural network, two fundamental choices determine its capacity: **depth** (number of layers) and **width** (neurons per layer). A network with 2 layers of 1000 neurons has the same parameter count as one with 10 layers of ~140 neurons each — but they learn *very* differently. Deep networks learn **hierarchical features** with exponentially greater efficiency, while wide networks offer simpler optimization landscapes. This chapter explores the theory behind depth vs width, when each is preferred, and how landmark architectures balance the two.

---

## 1. Definitions

```
  WIDTH = Number of neurons in a hidden layer
  DEPTH = Number of hidden layers
  
  Wide and Shallow:            Deep and Narrow:
  ┌──────────────────┐         ┌────────┐
  │ Input: 784       │         │ 784    │  Input
  ├──────────────────┤         ├────────┤
  │ Hidden: 2000     │         │ 128    │  Hidden 1
  ├──────────────────┤         ├────────┤
  │ Output: 10       │         │ 128    │  Hidden 2
  └──────────────────┘         ├────────┤
                               │ 128    │  Hidden 3
  2 layers                     ├────────┤
  ~1.6M params                 │ 128    │  Hidden 4
                               ├────────┤
                               │ 128    │  Hidden 5
                               ├────────┤
                               │ 10     │  Output
                               └────────┘
                               6 layers
                               ~170K params
  
  BOTH can approximate any continuous function (Universal Approx. Theorem)
  BUT they do so VERY differently.
```

---

## 2. Depth: Hierarchical Feature Learning

### What Depth Provides

```
  Deep networks learn HIERARCHICAL FEATURES:
  
  Image Classification Example:
  
  Layer 1: Edges, gradients
  ┌─────────────────────────────────────────────┐
  │  ╱  ╲  ─  │  ╲╱  ──  │╲  ╱│              │
  │  Simple edge detectors                      │
  └─────────────────────────────────────────────┘
           ↓
  Layer 2: Textures, corners
  ┌─────────────────────────────────────────────┐
  │  ┌─┐  ╱╲  ┌┐  ▓░  ██  ░▒▓                │
  │  Combinations of edges → patterns           │
  └─────────────────────────────────────────────┘
           ↓
  Layer 3: Parts (eyes, wheels, windows)
  ┌─────────────────────────────────────────────┐
  │  👁  🚗  🏠  ←  parts of objects           │
  │  Combinations of textures → object parts    │
  └─────────────────────────────────────────────┘
           ↓
  Layer 4: Objects (faces, cars, houses)
  ┌─────────────────────────────────────────────┐
  │  🧑  🚗  🏠  ←  complete objects           │
  │  Combinations of parts → whole objects      │
  └─────────────────────────────────────────────┘
  
  Each layer builds on features from the previous layer.
  This hierarchy CANNOT be efficiently captured by a single wide layer!
```

### Depth Efficiency

```
  DEPTH EFFICIENCY THEOREM (various authors):
  
  Some functions require exponentially more neurons 
  in a shallow network than in a deep one.
  
  Example: Computing XOR of n bits
  
  Shallow (1 hidden layer):  O(2ⁿ) neurons needed!
  Deep (O(n) layers):        O(n) neurons needed
  
  
  Visualization:
  
  Complexity that can be represented:
  
  With 100 neurons:
  ┌──────────────────────────────────────────────────────────┐
  │  1 layer:   Can represent X functions                    │
  │  2 layers:  Can represent X² functions                   │
  │  3 layers:  Can represent X³ functions                   │
  │  ...                                                     │
  │  k layers:  Can represent X^k functions                  │
  │                                                          │
  │  Each additional layer MULTIPLIES the representational   │
  │  power (in the best case).                               │
  └──────────────────────────────────────────────────────────┘
  
  This is why deep learning works:
  Depth gives EXPONENTIAL representational power
  relative to the number of parameters!
```

---

## 3. Width: Capacity and Approximation

### What Width Provides

```
  UNIVERSAL APPROXIMATION THEOREM (Cybenko, 1989):
  
  A single hidden layer with enough neurons can approximate
  ANY continuous function to arbitrary precision.
  
  BUT "enough neurons" might be ASTRONOMICALLY many!
  
  Width = raw approximation capacity
  
  Narrow layer (32 neurons):
  ├── Can partition input space into ~32 regions
  ├── Limited expressiveness
  └── Fast computation
  
  Wide layer (1024 neurons):
  ├── Can partition input space into ~1024 regions
  ├── High expressiveness
  └── More computation per layer
  
  
  Think of width as "resolution":
  ┌──────────────────────────────────────────────────┐
  │  Low width (32):   Coarse decision boundaries   │
  │     ┌──────┐                                     │
  │     │Class │   Few, simple regions               │
  │     │  A   │                                     │
  │     └──────┘                                     │
  │                                                  │
  │  High width (512): Fine decision boundaries      │
  │     ┌──┬──┬──┐                                   │
  │     │A │B │A │  Many, complex regions            │
  │     ├──┼──┼──┤                                   │
  │     │B │A │B │                                   │
  │     └──┴──┴──┘                                   │
  └──────────────────────────────────────────────────┘
```

---

## 4. Deep vs Wide: Tradeoffs

```
  ┌──────────────────────────────────────────────────────────────────┐
  │              DEEP NETWORKS                                       │
  │  ─────────────────────────                                       │
  │  ✓ Hierarchical feature learning                                │
  │  ✓ Parameter efficient (same capacity with fewer params)        │
  │  ✓ Compositional representations                                │
  │  ✓ Can learn abstract concepts                                  │
  │                                                                  │
  │  ✗ Harder to train (vanishing/exploding gradients)              │
  │  ✗ Need skip connections for very deep (>20 layers)             │
  │  ✗ Longer training time per step (sequential computation)       │
  │  ✗ More hyperparameters to tune                                 │
  ├──────────────────────────────────────────────────────────────────┤
  │              WIDE NETWORKS                                       │
  │  ─────────────────────────                                       │
  │  ✓ Easier to optimize (smoother loss landscape)                 │
  │  ✓ Better parallelization (GPU friendly — one big matmul)      │
  │  ✓ Simpler architecture decisions                               │
  │  ✓ Easier to train (less gradient flow issues)                  │
  │                                                                  │
  │  ✗ Less parameter efficient                                     │
  │  ✗ Can't learn hierarchical features as well                    │
  │  ✗ More prone to memorization (overfitting)                     │
  │  ✗ More memory usage (wider activations)                        │
  └──────────────────────────────────────────────────────────────────┘
```

### Diminishing Returns of Depth

```
  Performance vs Depth (holding total params constant):
  
  Accuracy ↑
           │
  96%      │                  ●●●●●●●── diminishing returns
           │              ●●●
  94%      │           ●●
           │         ●●
  92%      │       ●●
           │     ●●
  90%      │   ●●     sweet spot: 3-8 layers for MLPs
           │  ●
  88%      │●
           └──┬────┬────┬────┬────┬────┬──→ Depth (layers)
              1    2    4    8   16   32
  
  Beyond the sweet spot:
  • Gradients vanish → need skip connections
  • Optimization becomes harder
  • Diminishing returns without architectural tricks
```

### Practical Tradeoff Illustration

```python
import torch.nn as nn

# Same total parameters (~200K), different depth/width:

# Wide and shallow: 2 layers, 480 neurons each
shallow = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 480), nn.ReLU(),    # 784×480 = 376,320
    nn.Linear(480, 10),                 # 480×10 = 4,800
)  # Total: ~381K params

# Balanced: 4 layers, 200 neurons each
balanced = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 200), nn.ReLU(),    # 784×200 = 156,800
    nn.Linear(200, 200), nn.ReLU(),    # 200×200 = 40,000
    nn.Linear(200, 200), nn.ReLU(),    # 200×200 = 40,000
    nn.Linear(200, 10),                 # 200×10 = 2,000
)  # Total: ~239K params

# Deep and narrow: 8 layers, 100 neurons each
deep = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 100), nn.ReLU(),    # 784×100 = 78,400
    nn.Linear(100, 100), nn.ReLU(),    # 100×100 = 10,000 (×6)
    nn.Linear(100, 100), nn.ReLU(),
    nn.Linear(100, 100), nn.ReLU(),
    nn.Linear(100, 100), nn.ReLU(),
    nn.Linear(100, 100), nn.ReLU(),
    nn.Linear(100, 100), nn.ReLU(),
    nn.Linear(100, 10),                # 100×10 = 1,000
)  # Total: ~140K params

# In practice, the balanced model often wins!
```

---

## 5. When to Prefer Depth vs Width

```
  ┌──────────────────────────────────────────────────────────────┐
  │  PREFER MORE DEPTH WHEN:                                     │
  │  ─────────────────────                                       │
  │  • Data has hierarchical structure (images, text, audio)    │
  │  • Features at different abstraction levels matter          │
  │  • You need parameter efficiency                            │
  │  • You can use skip connections / residual blocks           │
  │  • Task requires compositional reasoning                    │
  │                                                              │
  │  PREFER MORE WIDTH WHEN:                                     │
  │  ──────────────────────                                      │
  │  • Data is tabular / flat (no spatial/temporal structure)   │
  │  • Features are all at the same abstraction level           │
  │  • You have lots of GPU memory                              │
  │  • Training stability is a concern                          │
  │  • You need fast inference (wide = parallel on GPU)         │
  │                                                              │
  │  GENERAL RULE:                                               │
  │  ──────────────                                              │
  │  Start with moderate depth (3-5 layers) and moderate width  │
  │  (128-512 neurons). Adjust based on underfitting/overfitting│
  └──────────────────────────────────────────────────────────────┘
```

---

## 6. Common Architectures and Their Depth/Width

```
  ┌────────────────────┬────────┬───────────┬────────────────────┐
  │  Architecture      │ Depth  │ Width     │ Parameters         │
  ├────────────────────┼────────┼───────────┼────────────────────┤
  │  Simple MLP        │ 2-3    │ 128-512   │ ~100K-1M           │
  │  LeNet-5 (1998)    │ 5      │ 6-120     │ ~60K               │
  │  AlexNet (2012)    │ 8      │ 96-4096   │ ~60M               │
  │  VGG-16 (2014)     │ 16     │ 64-512    │ ~138M              │
  │  ResNet-50 (2015)  │ 50     │ 64-2048   │ ~25M               │
  │  ResNet-152         │ 152    │ 64-2048   │ ~60M               │
  │  Wide ResNet-28-10  │ 28     │ 10× wider │ ~36M              │
  │  DenseNet-121       │ 121    │ 32 (growth)│ ~8M               │
  │  EfficientNet-B0    │ ~18    │ balanced  │ ~5M                │
  │  GPT-3 (2020)      │ 96     │ 12,288    │ ~175B              │
  │  GPT-4 (2023)      │ ?      │ ?         │ ~1.8T (estimated) │
  └────────────────────┴────────┴───────────┴────────────────────┘
  
  Trend: Models have gotten DEEPER over time.
  But this was only possible after:
  1. ReLU activation (2011) — reduces vanishing gradient
  2. Batch normalization (2015) — stabilizes training
  3. Residual connections (2015) — enables 100+ layers
  4. Transformers (2017) — enable massive width AND depth
```

---

## 7. The Double Descent Phenomenon

```
  DOUBLE DESCENT (Belkin et al., 2019):
  
  Classical view: More parameters = more overfitting
  Reality: Performance follows a DOUBLE DESCENT curve!
  
  Test Error ↑
             │
             │●                              Classical regime
             │ ●                             ──────────────
             │  ●●                           More params → overfit
             │    ●●
             │      ●●●         ← Interpolation threshold
             │         ●●●●     (model can memorize all training data)
             │             ●●●
             │                ●●●●           Modern regime
             │                    ●●●        ──────────────
             │                       ●●●●    More params → BETTER!
             │                           ●●──
             └──────────────────────────────────→ Model Size
             
             Under-         Over-      "More is more"
             parameterized  parameterized

  
  Implication: Don't be afraid of LARGE models!
  Very wide or very deep models (past the interpolation threshold)
  can generalize BETTER, not worse — if properly trained.
  
  This is why GPT-3 (175B params) works better than GPT-2 (1.5B).
```

---

## 8. Worked Example: Comparing Depth vs Width

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
from torchvision import datasets, transforms
import time

transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])
train_data = datasets.MNIST('./data', train=True, download=True, transform=transform)
test_data = datasets.MNIST('./data', train=False, transform=transform)

def make_model(depth, width):
    """Create MLP with specified depth and width."""
    layers = [nn.Flatten(), nn.Linear(784, width), nn.ReLU()]
    for _ in range(depth - 2):
        layers.extend([nn.Linear(width, width), nn.ReLU()])
    layers.append(nn.Linear(width, 10))
    return nn.Sequential(*layers)

def train_and_evaluate(model, epochs=15, lr=1e-3, batch_size=128):
    """Train model and return test accuracy."""
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    model = model.to(device)
    train_loader = DataLoader(train_data, batch_size=batch_size, shuffle=True)
    test_loader = DataLoader(test_data, batch_size=512)
    
    optimizer = optim.Adam(model.parameters(), lr=lr)
    criterion = nn.CrossEntropyLoss()
    
    n_params = sum(p.numel() for p in model.parameters())
    start = time.time()
    
    for epoch in range(epochs):
        model.train()
        for X, y in train_loader:
            X, y = X.to(device), y.to(device)
            optimizer.zero_grad()
            criterion(model(X), y).backward()
            optimizer.step()
    
    model.eval()
    correct = 0
    with torch.no_grad():
        for X, y in test_loader:
            X, y = X.to(device), y.to(device)
            correct += (model(X).argmax(1) == y).sum().item()
    
    acc = correct / len(test_data)
    elapsed = time.time() - start
    print(f"  Depth={depth:2d}, Width={width:4d} | "
          f"Params={n_params:>8,} | Acc={acc:.4f} | Time={elapsed:.1f}s")
    return acc

# Experiment: varying depth and width
configs = [
    (2, 512),   # shallow and wide
    (3, 256),   # moderate
    (5, 160),   # deeper, narrower
    (8, 100),   # deep and narrow
    (12, 80),   # very deep and narrow
]

print("Depth vs Width Experiment (MNIST, 15 epochs):")
print("=" * 65)
for depth, width in configs:
    model = make_model(depth, width)
    train_and_evaluate(model)
```

---

## 9. Summary Table

| Aspect | Depth (More Layers) | Width (More Neurons) |
|--------|:-------------------:|:--------------------:|
| **Feature learning** | Hierarchical (edges → parts → objects) | Flat (all at same level) |
| **Parameter efficiency** | High (exponential expressiveness) | Low (polynomial expressiveness) |
| **Training difficulty** | Harder (vanishing gradients) | Easier (smoother loss) |
| **GPU utilization** | Sequential layers | Parallel within layer |
| **Overfitting risk** | Lower (with same params) | Higher |
| **Best for** | Structured data (images, text) | Tabular / flat data |
| **Key enablers** | Skip connections, BN, ReLU | More memory, larger GPU |

---

## 10. Revision Questions

1. **Explain the difference between depth and width in neural networks.** How does each contribute to a model's representational capacity?

2. **What is the depth efficiency property?** Give an example of a function that requires exponentially more neurons in a shallow network than a deep one.

3. **Why do deep networks learn hierarchical features while wide networks don't?** Use the example of image classification to illustrate.

4. **Compare a network with 2 layers of 1000 neurons vs 10 layers of 140 neurons (similar total parameters).** Which would you expect to perform better on image classification and why?

5. **What is the double descent phenomenon?** How does it challenge the classical view that more parameters always lead to more overfitting?

6. **Name three architectural innovations that enabled training very deep networks (50+ layers).** Explain how each one helps.

---

| [← 01 Architecture Design](01-architecture-design-principles.md) | [Unit 10 Home](README.md) | [03 → Skip Connections](03-skip-connections.md) |
|:---|:---:|---:|
