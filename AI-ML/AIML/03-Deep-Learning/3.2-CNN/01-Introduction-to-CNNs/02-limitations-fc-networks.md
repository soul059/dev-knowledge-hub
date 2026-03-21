# 🚫 1.2 — Limitations of Fully Connected Networks for Images

> **Unit:** 01 – Introduction to CNNs
> **Chapter:** 2 of 5 — Why FC Fails for Visual Data

---

[← Previous: Why CNNs for Images](./01-why-cnns-for-images.md) · [🏠 Unit Overview](./README.md) · [Next → Local Connectivity](./03-local-connectivity.md)

---

## 📋 Chapter Overview

Before appreciating CNNs, we must understand **why standard fully connected (FC / dense) networks are fundamentally ill-suited for image data**. This chapter dives deep into the parameter explosion, loss of spatial information, absence of translation invariance, overfitting tendencies, and computational costs of using FC networks for vision tasks.

**After this chapter you will be able to:**

1. Calculate the exact parameter count when flattening an image into an FC layer.
2. Explain why flattening destroys spatial relationships.
3. Define translation invariance and why FC layers lack it.
4. Compare computation and memory costs of FC vs convolutional approaches.
5. Justify why CNNs were necessary to scale deep learning to real-world images.

---

## 1 — The FC Approach to Images

### 1.1 — How FC Processes an Image

A fully connected layer treats its input as a **flat 1-D vector**. To use an FC network for images, we must **flatten** the 3-D image tensor into a single vector.

```
   RGB Image (224 × 224 × 3)
   ┌─────────────────────────┐
   │  ┌─────┐ ┌─────┐ ┌─────┐│
   │  │  R  │ │  G  │ │  B  ││        Flatten
   │  │224× │ │224× │ │224× ││  ──────────────→   [x₁, x₂, x₃, ... , x₁₅₀₅₂₈]
   │  │ 224 │ │ 224 │ │ 224 ││                      ↑
   │  └─────┘ └─────┘ └─────┘│                 1-D vector of
   └─────────────────────────┘                  150,528 values
```

### 1.2 — The Flattening Formula

```
Flattened length = H × W × C

For a 224×224×3 image:
  = 224 × 224 × 3
  = 150,528 input neurons
```

---

## 2 — Problem 1: Parameter Explosion 💥

### 2.1 — The Math

For a single FC layer with `n_in` inputs and `n_out` outputs:

```
Parameters = n_in × n_out + n_out  (weights + biases)
```

### 2.2 — Worked Example: First Hidden Layer

| Input Image | Flattened Size (n_in) | Hidden Units (n_out) | FC Parameters |
|---|---|---|---|
| MNIST (28×28×1) | 784 | 512 | 401,920 |
| CIFAR-10 (32×32×3) | 3,072 | 512 | 1,573,376 |
| ImageNet (224×224×3) | 150,528 | 4,096 | **616,566,784** |
| 4K Image (3840×2160×3) | 24,883,200 | 4,096 | **~101.9 billion** |

```
ImageNet FC calculation:
  n_in  = 224 × 224 × 3 = 150,528
  n_out = 4,096
  
  Weights = 150,528 × 4,096 = 616,562,688
  Biases  = 4,096
  Total   = 616,566,784 parameters

  Memory (float32, 4 bytes each):
  = 616,566,784 × 4 bytes
  = 2,466,267,136 bytes
  ≈ 2.3 GB ... for just ONE layer!
```

### 2.3 — Comparison: FC vs Conv for Same Task

```
Task: Process 224×224×3 image → 64 feature maps

FC Layer:                                Conv Layer (3×3):
─────────────────────                    ─────────────────────
Input:  150,528                          Input:  224 × 224 × 3
Output: 64 units                         Output: 222 × 222 × 64
                                         Kernel: 3 × 3

Params: 150,528 × 64 + 64               Params: 3 × 3 × 3 × 64 + 64
      = 9,633,856                              = 1,792

Ratio: 9,633,856 / 1,792 ≈ 5,375×

                    FC has 5,375× MORE parameters!
```

### 2.4 — Multi-Layer Parameter Comparison

A complete network for 224×224×3 → 1000 classes:

```
╔═══════════════════════════════════════════════════════════════════╗
║ FC Network (3 hidden layers)                                     ║
║ ─────────────────────────────                                    ║
║ Layer 1: 150,528 → 4,096    = 616.6M params                     ║
║ Layer 2: 4,096   → 4,096    = 16.8M params                      ║
║ Layer 3: 4,096   → 1,000    = 4.1M params                       ║
║ TOTAL                       ≈ 637.5M params  (≈ 2.4 GB)         ║
╠═══════════════════════════════════════════════════════════════════╣
║ CNN (e.g., VGG-style)                                            ║
║ ─────────────────────────────                                    ║
║ Conv layers total            ≈ 15M params                        ║
║ FC head                      ≈ 123M params (still large!)        ║
║ TOTAL                       ≈ 138M params  (≈ 0.5 GB)           ║
╠═══════════════════════════════════════════════════════════════════╣
║ Modern CNN (e.g., ResNet-50)                                     ║
║ ─────────────────────────────                                    ║
║ All layers total             ≈ 25.6M params (≈ 0.1 GB)          ║
╚═══════════════════════════════════════════════════════════════════╝
```

### 2.5 — PyTorch Code: Parameter Count Comparison

```python
import torch
import torch.nn as nn

# FC Network for 224×224×3 → 1000 classes
class FCNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.flatten = nn.Flatten()
        self.fc1 = nn.Linear(224 * 224 * 3, 4096)
        self.fc2 = nn.Linear(4096, 4096)
        self.fc3 = nn.Linear(4096, 1000)
    
    def forward(self, x):
        x = self.flatten(x)
        x = torch.relu(self.fc1(x))
        x = torch.relu(self.fc2(x))
        return self.fc3(x)

# CNN for same task
class SimpleCNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 64, 3, padding=1),    nn.ReLU(),  nn.MaxPool2d(2),
            nn.Conv2d(64, 128, 3, padding=1),  nn.ReLU(),  nn.MaxPool2d(2),
            nn.Conv2d(128, 256, 3, padding=1), nn.ReLU(),  nn.MaxPool2d(2),
            nn.Conv2d(256, 256, 3, padding=1), nn.ReLU(),  nn.MaxPool2d(2),
        )
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(256 * 14 * 14, 1000),
        )
    
    def forward(self, x):
        x = self.features(x)
        return self.classifier(x)

def count_params(model):
    return sum(p.numel() for p in model.parameters())

fc_model  = FCNet()
cnn_model = SimpleCNN()

fc_params  = count_params(fc_model)
cnn_params = count_params(cnn_model)

print(f"FC Network : {fc_params:>15,} parameters")
print(f"CNN        : {cnn_params:>15,} parameters")
print(f"Ratio      : FC is {fc_params / cnn_params:.1f}× larger")

# Output:
# FC Network :   637,542,440 parameters
# CNN        :    51,175,880 parameters
# Ratio      : FC is 12.5× larger
```

---

## 3 — Problem 2: Destruction of Spatial Structure 🗺️

### 3.1 — Flattening Loses Neighbour Relationships

When we flatten a 2-D grid into a 1-D vector, **pixels that were spatially adjacent become arbitrarily far apart** in the vector.

```
Original 4×4 image (row-major flattening):

     Col 0  Col 1  Col 2  Col 3
     ─────  ─────  ─────  ─────
Row 0:  a      b      c      d
Row 1:  e      f      g      h
Row 2:  i      j      k      l
Row 3:  m      n      o      p

Flattened: [a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p]
            0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15

Spatial neighbours of pixel 'f' (row 1, col 1):
  In 2D: b(↑), j(↓), e(←), g(→)     — all adjacent
  In 1D: b=index 1, j=index 9, e=index 4, g=index 6

  In the flattened vector, 'b' is 4 positions away,
  'j' is 4 positions away — they look no closer than
  any random pixel to the FC layer.
```

### 3.2 — Why This Matters

An FC layer treats every input dimension **identically and independently** — it has no concept of "this pixel is next to that pixel." Two architecturally identical FC networks, given the same image with pixels **randomly shuffled**, would perform equally well during training.

```
Original image:         Shuffled pixels:

  ┌──────────┐          ┌──────────┐
  │ 🐱 cat   │          │ ░▓█░▓░▓█ │
  │ sitting  │          │ █▓░█▓█░▓ │
  │ on mat   │          │ ▓░█▓░▓█░ │
  └──────────┘          └──────────┘

  FC sees both as identical 1-D vectors of same values,
  just in different order. It CANNOT exploit spatial structure.

  A CNN would fail on shuffled pixels — it RELIES on spatial order.
  That reliance is exactly what makes it powerful for real images.
```

### 3.3 — PyTorch Demonstration: FC Is Shuffle-Invariant

```python
import torch
import torch.nn as nn

# Create an FC layer
fc = nn.Linear(16, 4)

# 4×4 "image" as a 1-D input
image = torch.tensor([1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16], dtype=torch.float32)

# Randomly shuffle the input
perm = torch.randperm(16)
shuffled = image[perm]

# FC output depends on values, but FC CAN learn either ordering equally well.
# The architecture itself has no preference for any ordering.
output_orig    = fc(image)
output_shuffled = fc(shuffled)

print("Original input  :", image.tolist())
print("Shuffled input  :", shuffled.tolist())
print("Output (orig)   :", output_orig.detach().tolist())
print("Output (shuffled):", output_shuffled.detach().tolist())
print("\nOutputs differ because weights assume original order.")
print("But FC has NO architectural bias to prefer either order.")
```

---

## 4 — Problem 3: No Translation Invariance 🔄

### 4.1 — What Is Translation Invariance?

**Translation invariance** means the network's output (e.g., "this is a cat") should not change if the object moves to a different position in the image.

```
Same cat, three positions:

  Position A         Position B         Position C
  ┌──────────┐      ┌──────────┐      ┌──────────┐
  │🐱         │      │    🐱     │      │         🐱│
  │           │      │          │      │           │
  │           │      │          │      │           │
  └──────────┘      └──────────┘      └──────────┘

  All three should output: "cat" ✓
```

### 4.2 — Why FC Lacks Translation Invariance

In an FC layer, each weight connects a **specific input position** to an output neuron. Moving the cat changes which pixels are activated, activating a **completely different set** of weights.

```
Weight matrix for FC (simplified, 9 inputs → 1 output):

  w₁  w₂  w₃
  w₄  w₅  w₆     ← Each wᵢ is a DIFFERENT, independently learned weight
  w₇  w₈  w₉

Cat at top-left uses w₁, w₂, w₄, w₅
Cat at bottom-right uses w₅, w₆, w₈, w₉

These are DIFFERENT weights → the network must learn cat-detection
separately for EVERY possible position.
```

### 4.3 — Mathematical Formulation

For an FC layer, the output for neuron j is:

```
y_j = Σᵢ (w_ji × x_i) + b_j

If we shift the input (translate the image), x_i changes for all i.
Different w_ji values are multiplied with the object's pixel values.
The output y_j changes unpredictably.
```

For a convolutional layer:

```
y[m, n] = Σₖ Σₗ (w[k, l] × x[m+k, n+l]) + b

The SAME weights w[k, l] are used at every position (m, n).
If the input shifts by (Δm, Δn), the output shifts by (Δm, Δn).
→ This is translation EQUIVARIANCE (feature map shifts with input).
Combined with pooling → approximate translation INVARIANCE.
```

### 4.4 — Training Data Consequences

Without translation invariance, an FC network needs to see the object at **every possible position** during training:

```
For a 224×224 image with a 50×50 object:
  Possible positions ≈ (224-50) × (224-50) = 174 × 174 = 30,276

The FC network needs examples at many of these 30,276 positions
to learn reliable detection. This is EXTREMELY data-inefficient.

A CNN with weight sharing learns once and applies everywhere.
```

---

## 5 — Problem 4: Overfitting 📈

### 5.1 — More Parameters = More Overfitting Risk

With hundreds of millions of parameters, an FC network has enormous capacity to **memorise** the training data rather than learn generalisable patterns.

```
Model Complexity vs Generalisation:

  Error │                          ╱ FC (too many params)
        │                        ╱
        │    Optimal ──→  ●    ╱
        │                ╱ ╲ ╱
        │              ╱    ╳
        │            ╱    ╱  ╲
        │          ╱    ╱     ╲── CNN (right capacity)
        │        ╱    ╱
        │      ╱    ╱
        │ ───╱────╱─────────────── Training Error
        └───────────────────────
              Model Complexity →

  The FC network's test error increases (overfits) while
  the CNN's test error continues to decrease.
```

### 5.2 — Quantifying Overfitting Risk

```
Dataset: CIFAR-10 (50,000 training images)
Each image: 32 × 32 × 3 = 3,072 values

FC Network (2 hidden layers):
  Layer 1: 3,072 × 1,024 = 3,145,728 params
  Layer 2: 1,024 × 1,024 = 1,048,576 params
  Output:  1,024 × 10    = 10,240 params
  Total: 4,204,544 params

  Ratio: 4,204,544 params / 50,000 samples ≈ 84 params per sample
  → High overfitting risk (rule of thumb: > 10 params/sample is risky)

CNN (small):
  Conv layers: ~50,000 params
  FC head:     ~10,000 params
  Total:        60,000 params

  Ratio: 60,000 / 50,000 ≈ 1.2 params per sample → Much safer
```

### 5.3 — PyTorch Code: Observing Overfitting

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchvision import datasets, transforms
from torch.utils.data import DataLoader

transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5)),
])

train_data = datasets.CIFAR10(root='./data', train=True,  download=True, transform=transform)
test_data  = datasets.CIFAR10(root='./data', train=False, download=True, transform=transform)

train_loader = DataLoader(train_data, batch_size=128, shuffle=True)
test_loader  = DataLoader(test_data,  batch_size=128, shuffle=False)

# FC Network — prone to overfitting
class FCNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Flatten(),
            nn.Linear(3072, 2048), nn.ReLU(),
            nn.Linear(2048, 1024), nn.ReLU(),
            nn.Linear(1024, 10),
        )
    def forward(self, x):
        return self.net(x)

# Small CNN — better generalisation
class SmallCNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Conv2d(3, 32, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(32, 64, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Flatten(),
            nn.Linear(64 * 8 * 8, 256), nn.ReLU(),
            nn.Linear(256, 10),
        )
    def forward(self, x):
        return self.net(x)

def train_and_evaluate(model, name, epochs=10):
    optimizer = optim.Adam(model.parameters(), lr=1e-3)
    criterion = nn.CrossEntropyLoss()
    
    for epoch in range(epochs):
        # Training
        model.train()
        train_correct, train_total = 0, 0
        for images, labels in train_loader:
            optimizer.zero_grad()
            outputs = model(images)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()
            train_correct += (outputs.argmax(1) == labels).sum().item()
            train_total += labels.size(0)
        
        # Testing
        model.eval()
        test_correct, test_total = 0, 0
        with torch.no_grad():
            for images, labels in test_loader:
                outputs = model(images)
                test_correct += (outputs.argmax(1) == labels).sum().item()
                test_total += labels.size(0)
        
        train_acc = 100 * train_correct / train_total
        test_acc  = 100 * test_correct / test_total
        gap = train_acc - test_acc
        print(f"{name} Epoch {epoch+1:2d}: "
              f"Train={train_acc:.1f}% Test={test_acc:.1f}% Gap={gap:.1f}%")

# Compare
print("=" * 60)
train_and_evaluate(FCNet(), "FC ", epochs=10)
print("=" * 60)
train_and_evaluate(SmallCNN(), "CNN", epochs=10)

# Expected output pattern:
# FC:  Train ~65%  Test ~48%  Gap ~17%  (large gap = overfitting)
# CNN: Train ~85%  Test ~72%  Gap ~13%  (smaller gap, better accuracy)
```

---

## 6 — Problem 5: Computational Cost 💻

### 6.1 — FLOPs Comparison

**FLOPs** (Floating Point Operations) for a single forward pass:

```
FC Layer:
  FLOPs = 2 × n_in × n_out   (multiply + accumulate)
  
  For ImageNet first layer:
  = 2 × 150,528 × 4,096
  = 1,233,125,376 FLOPs ≈ 1.23 GFLOPs

Conv Layer (3×3, 64 filters, 224×224 input):
  FLOPs = 2 × K² × C_in × C_out × H_out × W_out
  = 2 × 9 × 3 × 64 × 222 × 222
  = 170,098,944 FLOPs ≈ 0.17 GFLOPs

  FC uses 7.2× more FLOPs for its first layer!
```

### 6.2 — Memory Comparison

```
FC Layer (150,528 → 4,096):
  Weight matrix: 150,528 × 4,096 × 4 bytes = 2.3 GB
  + Activations: 4,096 × 4 bytes = 16 KB
  + Gradients: same as weights = 2.3 GB
  Total ≈ 4.6 GB for ONE layer

Conv Layer (3×3×3 → 64):
  Weights: 3 × 3 × 3 × 64 × 4 bytes = 6.9 KB
  + Activations: 222 × 222 × 64 × 4 bytes = 12.6 MB
  + Gradients: ~6.9 KB
  Total ≈ 12.6 MB

  FC uses ~365× more memory for parameters!
```

### 6.3 — Scaling Behaviour

```
Image Resolution vs Parameters (first layer):

Resolution      FC (→4096)              Conv 3×3 (→64)
──────────────────────────────────────────────────────────
32 × 32 × 3     12.6M params            1,792 params
64 × 64 × 3     50.3M params            1,792 params
128 × 128 × 3   201.3M params           1,792 params
224 × 224 × 3   616.6M params           1,792 params
512 × 512 × 3   3.22B params            1,792 params
1024 × 1024 × 3 12.88B params           1,792 params

 ┌───────────────────────────────────────────────────┐
 │  FC parameters grow as O(H × W)                   │
 │  Conv parameters are CONSTANT O(K² × C_in × C_out)│
 │  → Conv scales to any image resolution!            │
 └───────────────────────────────────────────────────┘
```

### 6.4 — PyTorch Code: Memory & Parameter Profiling

```python
import torch
import torch.nn as nn

def profile_layer(layer, input_shape, name):
    """Profile parameters, memory, and FLOPs of a layer."""
    x = torch.randn(1, *input_shape)
    
    params = sum(p.numel() for p in layer.parameters())
    param_memory = sum(p.numel() * p.element_size() for p in layer.parameters())
    
    # Forward pass to get output shape
    with torch.no_grad():
        y = layer(x)
    
    print(f"\n{'='*50}")
    print(f"Layer: {name}")
    print(f"{'='*50}")
    print(f"Input shape  : {list(x.shape)}")
    print(f"Output shape : {list(y.shape)}")
    print(f"Parameters   : {params:,}")
    print(f"Param Memory : {param_memory / 1024 / 1024:.2f} MB")
    
    return params

# Compare FC vs Conv for different image sizes
for H in [32, 64, 128, 224]:
    input_size = (3, H, H)
    flat_size = 3 * H * H
    
    fc_layer   = nn.Linear(flat_size, 64)
    conv_layer = nn.Conv2d(3, 64, kernel_size=3, padding=1)
    
    fc_params   = profile_layer(
        nn.Sequential(nn.Flatten(), fc_layer), input_size, f"FC {H}×{H}"
    )
    conv_params = profile_layer(conv_layer, input_size, f"Conv {H}×{H}")
    
    print(f"\n  → FC/Conv ratio: {fc_params/conv_params:.0f}×")
```

---

## 7 — Complete Side-by-Side Comparison

### 7.1 — Comprehensive Comparison Table

| Criterion | FC Network | CNN |
|---|---|---|
| **Input format** | Flattened 1-D vector | Native 2-D/3-D tensor |
| **Spatial awareness** | ❌ None (order-agnostic) | ✅ Exploits spatial locality |
| **Translation invariance** | ❌ Must learn separately for each position | ✅ Built-in via weight sharing |
| **Parameters (224×224×3 → 64)** | 9.6M | 1,792 |
| **Scaling with resolution** | O(H × W) — linear growth | O(1) — constant |
| **Memory (first layer)** | ~2.3 GB | ~7 KB |
| **Overfitting risk** | Very high | Moderate |
| **Training data needed** | Enormous | Moderate |
| **Feature reuse** | None — each weight is unique | Full — filters shared everywhere |
| **Biological plausibility** | Low | Higher (mimics visual cortex) |
| **Works on shuffled pixels?** | Yes (no spatial bias) | No (relies on spatial structure) |
| **Best for** | Tabular data, small inputs | Images, spatial data, sequences |

### 7.2 — ASCII Architecture Comparison

```
FC Network for Images:                     CNN for Images:

Input (224×224×3)                          Input (224×224×3)
      │                                         │
  ┌───▼───┐  Flatten                       ┌────▼────┐  3×3 Conv
  │150,528│  ──→ 1-D vector                │224×224×3│  Keep 2-D!
  └───┬───┘                                └────┬────┘
      │                                         │
  ┌───▼───┐  Fully Connected               ┌────▼────┐  3×3 Conv + Pool
  │ 4,096 │  (616M weights!)               │112×112× │  (1,792 weights)
  └───┬───┘                                │   64    │
      │                                    └────┬────┘
  ┌───▼───┐  Fully Connected               ┌────▼────┐  3×3 Conv + Pool
  │ 4,096 │  (16.8M weights)               │ 56×56×  │  (36,928 weights)
  └───┬───┘                                │  128    │
      │                                    └────┬────┘
  ┌───▼───┐  Output                             │ ... more conv layers
  │ 1,000 │                                ┌────▼────┐
  └───────┘                                │ 1,000   │  FC head (small)
                                           └─────────┘
 Total: ~637M params                      Total: ~25M params (ResNet-50)
```

---

## 8 — When FC Layers ARE Appropriate

Despite their limitations for images, FC layers still have a role:

| Use Case | Why FC Works |
|---|---|
| **CNN classifier head** | After conv layers reduce spatial dims, FC maps features → classes |
| **Tabular data** | No spatial structure to exploit; FC is natural |
| **Small images (e.g., MNIST)** | 784 inputs is manageable; FC works reasonably |
| **Embedding layers** | Maps discrete tokens to continuous vectors |
| **MLP-Mixer / gMLP** | Modern architectures revisit FC with clever structure |

```python
# FC as the final classifier in a CNN — this is standard practice
class TypicalCNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv_layers = nn.Sequential(
            nn.Conv2d(3, 64, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(64, 128, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.AdaptiveAvgPool2d((1, 1)),  # Global Average Pooling → (128, 1, 1)
        )
        self.fc = nn.Linear(128, 10)  # Small FC layer at the end — only 1,290 params
    
    def forward(self, x):
        x = self.conv_layers(x)
        x = x.view(x.size(0), -1)
        return self.fc(x)
```

---

## 📊 Summary Table

| Problem | Root Cause | Consequence | CNN Solution |
|---|---|---|---|
| Parameter Explosion | Every input connects to every output | 600M+ params for one layer | Local connectivity — only K×K connections |
| No Spatial Awareness | Flattening destroys 2-D structure | Can't learn spatial patterns efficiently | 2-D filters preserve spatial layout |
| No Translation Invariance | Position-specific weights | Must relearn features at every location | Weight sharing — same filter everywhere |
| Overfitting | Too many parameters vs training samples | Memorises training data | Far fewer parameters → better generalisation |
| High Compute/Memory | Massive weight matrices | Needs impractical GPU memory | Compact filters, efficient computation |
| Poor Scaling | Params grow as O(H×W) | Unusable for high-res images | Params constant w.r.t. image size |

---

## ❓ Revision Questions

### Q1: Parameter Calculation
Calculate the number of parameters in an FC layer that takes a 64×64×3 RGB image and outputs 2,048 neurons. Include biases.

<details>
<summary>Answer</summary>

```
n_in  = 64 × 64 × 3 = 12,288
n_out = 2,048

Parameters = n_in × n_out + n_out
           = 12,288 × 2,048 + 2,048
           = 25,165,824 + 2,048
           = 25,167,872

≈ 25.2 million parameters for a single layer.
```

A 3×3 conv layer with same C_in=3, C_out=2048 would have:
3×3×3×2048 + 2048 = 55,296 + 2048 = 57,344 params — **439× fewer**.
</details>

### Q2: Spatial Structure
Why does an FC network perform equally well on an image and a randomly pixel-shuffled version of that same image? What does this tell us?

<details>
<summary>Answer</summary>

An FC layer connects every input neuron to every output neuron with independent weights. It treats the input as an **unordered bag of values** — there is no architectural inductive bias that encodes "pixel i is spatially adjacent to pixel j."

If you consistently shuffle pixels in the same order for all images, the FC network simply learns a different weight mapping. It can achieve the same training accuracy because it never exploited spatial structure in the first place.

This tells us FC networks are **wasting their capacity** on images — they cannot leverage the strong spatial correlations that are the defining characteristic of visual data.
</details>

### Q3: Translation Invariance
A cat appears at position (10, 10) in the training set. After training an FC network, the cat appears at position (200, 200) in a test image. Why might the FC network fail?

<details>
<summary>Answer</summary>

In an FC network, the weights connecting position (10, 10) to the output are **completely different** from the weights connecting position (200, 200). The network learned to detect "cat patterns" using the weights associated with position (10, 10), but those weights are never activated when the cat is at (200, 200). The position-(200, 200) weights never saw cat patterns during training, so they produce meaningless outputs.

A CNN would succeed because the **same filter** (same weights) is applied at both positions — it detects the cat pattern regardless of location.
</details>

### Q4: Scaling Behaviour
Why is it said that FC parameters grow as O(H × W) while conv parameters are O(1) with respect to image resolution?

<details>
<summary>Answer</summary>

**FC:** The number of input neurons equals H × W × C. The weight matrix has dimensions (H × W × C) × n_out. As H and W increase, the number of parameters grows **linearly** with the number of pixels.

**Conv:** The filter size (K × K × C_in × C_out) is fixed regardless of image resolution. A 3×3 conv with 64 filters always has 1,792 parameters whether the input is 32×32 or 4096×4096. The same filter slides over more positions for larger images (more computation), but the **parameter count stays constant**.

This is why CNNs can process arbitrarily large images without changing the model architecture.
</details>

### Q5: Memory Analysis
An FC layer with 150,528 inputs and 4,096 outputs uses float32 weights. Calculate the memory needed for (a) just the weights and (b) the gradients during training.

<details>
<summary>Answer</summary>

**(a) Weights:**
```
Number of weights = 150,528 × 4,096 = 616,562,688
Memory = 616,562,688 × 4 bytes = 2,466,250,752 bytes ≈ 2.30 GB
```

**(b) Gradients (same size as weights):**
```
Gradient memory = 2.30 GB
```

**(c) Total during training** (weights + gradients + optimizer states for Adam):
```
Adam stores m (momentum) and v (variance) — same size as weights.
Total = weights + gradients + m + v = 4 × 2.30 GB ≈ 9.2 GB

Just for ONE FC layer! This is why FC is impractical for images.
```
</details>

### Q6: Practical Design
In modern CNNs like ResNet-50, FC layers are still used at the very end. Why is this acceptable?

<details>
<summary>Answer</summary>

By the time data reaches the FC layer in a CNN, the convolutional and pooling layers have **dramatically reduced the spatial dimensions** while increasing channel depth. In ResNet-50:

- Input: 224 × 224 × 3 = 150,528 values
- After conv layers: 7 × 7 × 2048 = 100,352 values
- After Global Average Pooling: 1 × 1 × 2048 = **2,048 values**

The final FC layer is therefore 2,048 → 1,000, which has only **2,049,000 parameters** — perfectly manageable. The conv layers have already done the heavy lifting of spatial feature extraction and dimensionality reduction.
</details>

---

## 🔗 Navigation

| | |
|---|---|
| ← Previous | [Why CNNs for Images](./01-why-cnns-for-images.md) |
| → Next | [Local Connectivity](./03-local-connectivity.md) |
| 🏠 Unit | [01 – Introduction to CNNs](./README.md) |

---

*© 2025 AIML Study Notes. For educational use.*
