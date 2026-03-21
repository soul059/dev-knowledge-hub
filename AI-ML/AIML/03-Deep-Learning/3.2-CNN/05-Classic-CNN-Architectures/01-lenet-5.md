# 🏛️ LeNet-5: The Pioneer of Convolutional Neural Networks

[← Back to Unit Overview](../README.md) | [Next: AlexNet →](02-alexnet.md)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Historical Context](#historical-context)
- [Architecture Deep Dive](#architecture-deep-dive)
- [Layer-by-Layer Analysis](#layer-by-layer-analysis)
- [Parameter Calculations](#parameter-calculations)
- [ASCII Architecture Diagram](#ascii-architecture-diagram)
- [Mathematical Foundations](#mathematical-foundations)
- [PyTorch Implementation](#pytorch-implementation)
- [Worked Example](#worked-example)
- [Applications](#applications)
- [Summary Table](#summary-table)
- [Revision Questions](#revision-questions)

---

## Overview

**LeNet-5** is a convolutional neural network architecture proposed by **Yann LeCun, Léon Bottou, Yoshua Bengio, and Patrick Haffner** in their 1998 paper *"Gradient-Based Learning Applied to Document Recognition."* It is widely regarded as the **founding architecture** of modern CNNs and was specifically designed for **handwritten digit recognition** on the MNIST dataset.

### Key Facts

| Property | Value |
|----------|-------|
| **Year** | 1998 |
| **Authors** | LeCun, Bottou, Bengio, Haffner |
| **Paper** | Gradient-Based Learning Applied to Document Recognition |
| **Task** | Handwritten digit recognition (MNIST) |
| **Input Size** | 32 × 32 × 1 (grayscale) |
| **Parameters** | ~60,000 |
| **Depth** | 7 layers (2 Conv + 2 Pool + 3 FC) |
| **Key Innovation** | First successful CNN trained with backpropagation |

---

## Historical Context

### The Problem: Reading Handwritten Digits

In the 1990s, the US Postal Service needed to automatically read ZIP codes on mail. This required a system that could:
- Recognize handwritten digits (0–9)
- Handle enormous variation in handwriting styles
- Process millions of pieces of mail per day
- Achieve very high accuracy (>99%)

### Why LeNet-5 Mattered

Before LeNet-5, most pattern recognition systems relied on **hand-engineered features** — domain experts would manually design feature extractors for edges, corners, strokes, etc. LeNet-5 demonstrated that a neural network could **learn these features automatically** from raw pixel data via backpropagation.

```
Traditional Pipeline:
  Raw Image → Hand-Crafted Features → Classifier → Prediction
                  (manual effort)

LeNet-5 Pipeline:
  Raw Image → [Learned Feature Extraction + Classification] → Prediction
                   (end-to-end learning)
```

### Real-World Deployment

LeNet-5 wasn't just an academic exercise — it was deployed commercially:
- Read **millions of checks per day** for banks
- Processed ZIP codes for the **US Postal Service**
- One of the first neural networks to achieve **production-scale deployment**

---

## Architecture Deep Dive

LeNet-5 consists of **7 learnable layers**: 2 convolutional layers, 2 subsampling (pooling) layers, and 3 fully connected layers. The architecture follows the pattern:

```
Input → Conv → Pool → Conv → Pool → FC → FC → FC → Output
```

### Design Principles

1. **Spatial hierarchy**: Early layers detect local features (edges, corners); later layers combine them into higher-level patterns
2. **Spatial reduction**: Feature maps shrink spatially through pooling, concentrating information
3. **Channel expansion**: Number of feature maps increases deeper in the network (1 → 6 → 16)
4. **Translation invariance**: Pooling provides robustness to small translations of the input

---

## ASCII Architecture Diagram

```
                              LeNet-5 Architecture
                              ====================

 INPUT        CONV1       POOL1       CONV2       POOL2      FC1      FC2     OUTPUT
 32×32×1      28×28×6     14×14×6     10×10×16    5×5×16     120      84      10

┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌───────┐  ┌─────┐  ┌────┐  ┌────┐
│        │  │ 6 feat │  │ 6 feat │  │16 feat │  │16 feat│  │     │  │    │  │    │
│ 32×32  │─►│ maps   │─►│ maps   │─►│ maps   │─►│ maps  │─►│ 120 │─►│ 84 │─►│ 10 │
│ gray   │  │ 28×28  │  │ 14×14  │  │ 10×10  │  │  5×5  │  │     │  │    │  │    │
│        │  │        │  │        │  │        │  │       │  │     │  │    │  │    │
└────────┘  └────────┘  └────────┘  └────────┘  └───────┘  └─────┘  └────┘  └────┘
              5×5 conv    2×2 avg     5×5 conv    2×2 avg   flatten
              stride=1    stride=2    stride=1    stride=2
              +tanh       +tanh       +tanh       +tanh      +tanh   +tanh  softmax


Data Flow with Dimensions:
──────────────────────────

  [32×32×1] ──5×5 conv──► [28×28×6] ──2×2 pool──► [14×14×6] ──5×5 conv──► [10×10×16]
                                                                               │
                                                                          2×2 pool
                                                                               │
                                                                          [5×5×16]
                                                                               │
                                                                          flatten
                                                                               │
                                                                           [400]
                                                                               │
                                                                      FC (400→120)
                                                                               │
                                                                           [120]
                                                                               │
                                                                      FC (120→84)
                                                                               │
                                                                            [84]
                                                                               │
                                                                      FC (84→10)
                                                                               │
                                                                            [10]
                                                                          (output)
```

---

## Layer-by-Layer Analysis

### Layer 1: Convolutional Layer (C1)

| Property | Value |
|----------|-------|
| Input | 32 × 32 × 1 |
| Filter size | 5 × 5 |
| Number of filters | 6 |
| Stride | 1 |
| Padding | 0 (valid) |
| Activation | tanh |
| Output | 28 × 28 × 6 |

**Output size calculation:**

```
Output_H = (Input_H - Filter_H) / Stride + 1
         = (32 - 5) / 1 + 1
         = 28

Output: 28 × 28 × 6
```

**Parameters:**
```
Weights: 5 × 5 × 1 × 6 = 150
Biases:  6
Total:   156
```

Each of the 6 filters slides across the 32×32 input image to produce one 28×28 feature map. The 6 feature maps detect different low-level features like edges at various orientations.

### Layer 2: Subsampling / Pooling Layer (S2)

| Property | Value |
|----------|-------|
| Input | 28 × 28 × 6 |
| Pool size | 2 × 2 |
| Stride | 2 |
| Operation | Average pooling |
| Output | 14 × 14 × 6 |

**Output size calculation:**

```
Output_H = (Input_H - Pool_H) / Stride + 1
         = (28 - 2) / 2 + 1
         = 14

Output: 14 × 14 × 6
```

**Note on original LeNet-5:** The original paper used a *trainable* subsampling layer — each output was the average of 4 inputs, multiplied by a trainable coefficient, plus a trainable bias, then passed through a sigmoid activation. Modern implementations typically use standard average or max pooling.

**Original subsampling parameters:**
```
Coefficient: 1 per feature map × 6 = 6
Bias:        1 per feature map × 6 = 6
Total:       12
```

### Layer 3: Convolutional Layer (C3)

| Property | Value |
|----------|-------|
| Input | 14 × 14 × 6 |
| Filter size | 5 × 5 |
| Number of filters | 16 |
| Stride | 1 |
| Padding | 0 (valid) |
| Activation | tanh |
| Output | 10 × 10 × 16 |

**Output size calculation:**

```
Output_H = (14 - 5) / 1 + 1 = 10

Output: 10 × 10 × 16
```

**Important detail — Sparse connectivity in the original paper:**

In the original LeNet-5, NOT every C3 filter was connected to ALL 6 input feature maps. LeCun used a specific **connection table** to break symmetry and reduce computation:

```
Connection Table (which S2 feature maps connect to which C3 filters):
──────────────────────────────────────────────────────────────────────
              Input Feature Maps (S2)
               0  1  2  3  4  5
Filter  0:     ×  ×  ×  .  .  .    (connected to 3 inputs)
Filter  1:     .  ×  ×  ×  .  .    (connected to 3 inputs)
Filter  2:     .  .  ×  ×  ×  .    (connected to 3 inputs)
Filter  3:     .  .  .  ×  ×  ×    (connected to 3 inputs)
Filter  4:     ×  .  .  .  ×  ×    (connected to 3 inputs)
Filter  5:     ×  ×  .  .  .  ×    (connected to 3 inputs)
Filter  6:     ×  ×  ×  ×  .  .    (connected to 4 inputs)
Filter  7:     .  ×  ×  ×  ×  .    (connected to 4 inputs)
Filter  8:     .  .  ×  ×  ×  ×    (connected to 4 inputs)
Filter  9:     ×  .  .  ×  ×  ×    (connected to 4 inputs)
Filter 10:     ×  ×  .  .  ×  ×    (connected to 4 inputs)
Filter 11:     ×  ×  ×  .  .  ×    (connected to 4 inputs)
Filter 12:     ×  ×  .  ×  ×  .    (connected to 4 inputs)
Filter 13:     .  ×  ×  .  ×  ×    (connected to 4 inputs)
Filter 14:     ×  .  ×  ×  .  ×    (connected to 4 inputs)
Filter 15:     ×  ×  ×  ×  ×  ×    (connected to all 6)

× = connected    . = not connected
```

**Parameters (original sparse connectivity):**
```
Filters 0–5:   6 × (5 × 5 × 3 + 1) = 6 × 76 =   456
Filters 6–14:  9 × (5 × 5 × 4 + 1) = 9 × 101 =  909
Filter 15:     1 × (5 × 5 × 6 + 1) = 1 × 151 =  151
Total:                                           1,516
```

**Parameters (modern full connectivity):**
```
Weights: 5 × 5 × 6 × 16 = 2,400
Biases:  16
Total:   2,416
```

### Layer 4: Subsampling / Pooling Layer (S4)

| Property | Value |
|----------|-------|
| Input | 10 × 10 × 16 |
| Pool size | 2 × 2 |
| Stride | 2 |
| Operation | Average pooling |
| Output | 5 × 5 × 16 |

```
Output: 5 × 5 × 16 = 400 values
```

**Original parameters:** 16 coefficients + 16 biases = **32**

### Layer 5: Fully Connected Layer (C5/FC1)

| Property | Value |
|----------|-------|
| Input | 5 × 5 × 16 = 400 |
| Output | 120 |
| Activation | tanh |

In the original paper, this was described as a convolutional layer with 120 filters of size 5×5, but since the input is exactly 5×5, it's equivalent to a fully connected layer.

```
Parameters: 400 × 120 + 120 = 48,120
```

### Layer 6: Fully Connected Layer (F6/FC2)

| Property | Value |
|----------|-------|
| Input | 120 |
| Output | 84 |
| Activation | tanh |

```
Parameters: 120 × 84 + 84 = 10,164
```

**Why 84?** The number 84 = 12 × 7 was chosen because the output of F6 was compared against **7×12 bitmap representations** of each digit character in the original design.

### Layer 7: Output Layer

| Property | Value |
|----------|-------|
| Input | 84 |
| Output | 10 |
| Activation | Euclidean RBF (original) / Softmax (modern) |

```
Parameters: 84 × 10 + 10 = 850
```

The original LeNet-5 used **Euclidean Radial Basis Function (RBF)** units for the output layer, where each unit computed the Euclidean distance between the input vector and a fixed parameter vector. Modern implementations use **softmax** instead.

---

## Parameter Calculations

### Total Parameter Count

```
Layer          Type             Parameters
─────────────────────────────────────────────
C1             Conv 5×5              156
S2             Avg Pool               12*
C3             Conv 5×5            1,516*
S4             Avg Pool               32*
C5/FC1         FC                 48,120
F6/FC2         FC                 10,164
Output         FC                    850
─────────────────────────────────────────────
Total (original):              ~60,850

* Original paper values with sparse connections
  and trainable pooling coefficients.

Modern Implementation (fully connected C3, standard pooling):
─────────────────────────────────────────────────────────────
C1             Conv 5×5              156
S2             Avg Pool                0
C3             Conv 5×5            2,416
S4             Avg Pool                0
FC1            FC                 48,120
FC2            FC                 10,164
Output         FC                    850
─────────────────────────────────────────────────────────────
Total (modern):                ~61,706
```

### Comparison with Fully Connected Network

If we tried to solve the same problem (32×32 input → 10 classes) with a fully connected network having similar hidden layers:

```
FC Approach:
  Input:  32 × 32 = 1,024 neurons
  Hidden1: 400 neurons    →  1,024 × 400 + 400 = 410,000 params
  Hidden2: 120 neurons    →  400 × 120 + 120   =  48,120 params
  Hidden3:  84 neurons    →  120 × 84 + 84     =  10,164 params
  Output:   10 neurons    →  84 × 10 + 10      =     850 params
  ──────────────────────────────────────────────────────────────
  Total:                                        ~469,134 params

LeNet-5: ~61,706 params → 7.6× fewer parameters!
```

The key savings come from **weight sharing** in convolutional layers — the same filter is applied at every spatial location.

---

## Mathematical Foundations

### Convolution Operation

The 2D convolution in LeNet-5 is defined as:

```
                  K-1  K-1
y[i,j,m] = b_m + ∑    ∑   w[p, q, c, m] · x[i+p, j+q, c]
                 p=0  q=0

Where:
  x     = input feature map (H × W × C_in)
  w     = filter weights (K × K × C_in × C_out)
  b_m   = bias for output channel m
  y     = output feature map
  K     = kernel size (5 for LeNet-5)
  C_in  = number of input channels
  C_out = number of output channels (filters)
```

### Average Pooling

```
                    1    P-1  P-1
y[i,j,c] = ─────── ∑    ∑   x[i·s+p, j·s+q, c]
             P×P   p=0  q=0

Where:
  P = pool size (2 for LeNet-5)
  s = stride (2 for LeNet-5)
```

### Tanh Activation

```
             e^z - e^(-z)
tanh(z) = ──────────────── ,   output range: (-1, +1)
             e^z + e^(-z)

Derivative:
tanh'(z) = 1 - tanh²(z)
```

### Output Layer (Original — Euclidean RBF)

```
y_i = ∑_j (x_j - w_ij)²

Where:
  x = input from F6 (84 values)
  w_ij = fixed bitmap weights for class i
  y_i = "distance" — lower means more likely class i
```

### Loss Function (Modern — Cross-Entropy)

```
L = -∑_i t_i · log(p_i)

Where:
  t = one-hot target vector
  p = softmax output probabilities

Softmax:
           e^(z_i)
p_i = ──────────────
       ∑_j e^(z_j)
```

---

## Worked Example

### Forward Pass Through LeNet-5

Let's trace a **single 32×32 grayscale image** of the digit "7" through the network.

#### Step 1: Input Layer
```
Input: 32 × 32 × 1 tensor (1,024 values, pixel intensities in [0,1])

Example 3×3 corner of the image:
┌─────┬─────┬─────┐
│ 0.0 │ 0.0 │ 0.0 │
├─────┼─────┼─────┤
│ 0.0 │ 0.0 │ 0.8 │
├─────┼─────┼─────┤
│ 0.0 │ 0.0 │ 0.9 │
└─────┴─────┴─────┘
```

#### Step 2: C1 — First Convolution (5×5, 6 filters)

```
For filter k=0, at position (0,0):

          4   4
y[0,0,0] = ∑   ∑  w[p,q,0,0] · x[p,q,0]  +  b_0
         p=0 q=0

This 5×5 filter slides across the 32×32 input:
  - Horizontal positions: 32 - 5 + 1 = 28
  - Vertical positions:   32 - 5 + 1 = 28

One filter → one 28×28 feature map
Six filters → six 28×28 feature maps

Output after tanh: 28 × 28 × 6
Total activations: 28 × 28 × 6 = 4,704
```

#### Step 3: S2 — First Pooling (2×2 average, stride 2)

```
For each 2×2 block, compute the average:

┌──────┬──────┐
│ 0.20 │ 0.85 │  average = (0.20 + 0.85 + 0.10 + 0.60) / 4
├──────┼──────┤         = 1.75 / 4
│ 0.10 │ 0.60 │         = 0.4375
└──────┴──────┘

Each 28×28 map → 14×14 map
Output: 14 × 14 × 6
Total activations: 14 × 14 × 6 = 1,176
```

#### Step 4: C3 — Second Convolution (5×5, 16 filters)

```
Each filter now convolves over a 14×14×C input
(C = 3, 4, or 6 depending on the connection table)

Output: (14 - 5 + 1) × (14 - 5 + 1) × 16 = 10 × 10 × 16
Total activations: 10 × 10 × 16 = 1,600
```

#### Step 5: S4 — Second Pooling (2×2 average, stride 2)

```
Each 10×10 map → 5×5 map
Output: 5 × 5 × 16
Total activations: 5 × 5 × 16 = 400
```

#### Step 6: Flatten + FC1 (400 → 120)

```
Flatten: [5, 5, 16] → [400]

FC1: z = W · x + b
     Where W is 120×400, b is 120×1
     Output after tanh: [120]
```

#### Step 7: FC2 (120 → 84)

```
FC2: z = W · x + b
     Where W is 84×120, b is 84×1
     Output after tanh: [84]
```

#### Step 8: Output (84 → 10)

```
Output: z = W · x + b
        Where W is 10×84, b is 10×1
        Apply softmax:

Raw logits:  [0.1, -0.5, 0.3, -1.2, 0.8, -0.3, 0.2, 3.5, -0.4, 0.1]
                                                        ↑
                                                    digit "7"

Softmax probabilities:
  P(0) = 0.020    P(5) = 0.013
  P(1) = 0.011    P(6) = 0.022
  P(2) = 0.024    P(7) = 0.597  ← highest!
  P(3) = 0.005    P(8) = 0.012
  P(4) = 0.040    P(9) = 0.020

Prediction: argmax = 7 ✓

Loss (if true label is 7):
  L = -log(0.597) = 0.516
```

---

## PyTorch Implementation

### Modern LeNet-5

```python
import torch
import torch.nn as nn
import torch.nn.functional as F


class LeNet5(nn.Module):
    """
    Modern implementation of LeNet-5.
    Uses ReLU instead of tanh, max pooling instead of average pooling,
    and softmax (via CrossEntropyLoss) instead of RBF output.
    """
    def __init__(self, num_classes=10):
        super(LeNet5, self).__init__()

        # Feature extraction layers
        self.conv1 = nn.Conv2d(in_channels=1, out_channels=6,
                               kernel_size=5, stride=1, padding=0)
        self.conv2 = nn.Conv2d(in_channels=6, out_channels=16,
                               kernel_size=5, stride=1, padding=0)

        # Pooling layer
        self.pool = nn.AvgPool2d(kernel_size=2, stride=2)

        # Classification layers
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, num_classes)

    def forward(self, x):
        # x: [batch, 1, 32, 32]

        # C1: Conv → Tanh → S2: Pool
        x = self.pool(torch.tanh(self.conv1(x)))   # [batch, 6, 14, 14]

        # C3: Conv → Tanh → S4: Pool
        x = self.pool(torch.tanh(self.conv2(x)))    # [batch, 16, 5, 5]

        # Flatten
        x = x.view(x.size(0), -1)                   # [batch, 400]

        # FC1
        x = torch.tanh(self.fc1(x))                 # [batch, 120]

        # FC2
        x = torch.tanh(self.fc2(x))                 # [batch, 84]

        # Output (no softmax — handled by CrossEntropyLoss)
        x = self.fc3(x)                             # [batch, 10]

        return x


# Verify architecture
model = LeNet5()
print(model)

# Count parameters
total_params = sum(p.numel() for p in model.parameters())
trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"\nTotal parameters:     {total_params:,}")
print(f"Trainable parameters: {trainable_params:,}")

# Test with random input
x = torch.randn(1, 1, 32, 32)
output = model(x)
print(f"\nInput shape:  {x.shape}")
print(f"Output shape: {output.shape}")
print(f"Predictions:  {F.softmax(output, dim=1).detach()}")
```

**Expected output:**
```
LeNet5(
  (conv1): Conv2d(1, 6, kernel_size=(5, 5), stride=(1, 1))
  (conv2): Conv2d(6, 16, kernel_size=(5, 5), stride=(1, 1))
  (pool): AvgPool2d(kernel_size=2, stride=2, padding=0)
  (fc1): Linear(in_features=400, out_features=120, bias=True)
  (fc2): Linear(in_features=120, out_features=84, bias=True)
  (fc3): Linear(in_features=84, out_features=10, bias=True)
)

Total parameters:     61,706
Trainable parameters: 61,706

Input shape:  torch.Size([1, 1, 32, 32])
Output shape: torch.Size([1, 10])
```

### Full Training Pipeline on MNIST

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchvision import datasets, transforms
from torch.utils.data import DataLoader


# ─── Hyperparameters ───
BATCH_SIZE = 64
LEARNING_RATE = 0.001
EPOCHS = 10
DEVICE = torch.device('cuda' if torch.cuda.is_available() else 'cpu')


# ─── Data Loading ───
# MNIST images are 28×28, but LeNet-5 expects 32×32, so we pad
transform = transforms.Compose([
    transforms.Pad(2),                  # 28×28 → 32×32
    transforms.ToTensor(),              # [0, 255] → [0.0, 1.0]
    transforms.Normalize((0.1307,), (0.3081,))  # MNIST mean & std
])

train_dataset = datasets.MNIST(root='./data', train=True,
                                download=True, transform=transform)
test_dataset = datasets.MNIST(root='./data', train=False,
                               download=True, transform=transform)

train_loader = DataLoader(train_dataset, batch_size=BATCH_SIZE, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=BATCH_SIZE, shuffle=False)


# ─── Model, Loss, Optimizer ───
model = LeNet5(num_classes=10).to(DEVICE)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=LEARNING_RATE)


# ─── Training Loop ───
def train(model, loader, criterion, optimizer, device):
    model.train()
    total_loss = 0
    correct = 0
    total = 0

    for batch_idx, (data, target) in enumerate(loader):
        data, target = data.to(device), target.to(device)

        optimizer.zero_grad()
        output = model(data)
        loss = criterion(output, target)
        loss.backward()
        optimizer.step()

        total_loss += loss.item() * data.size(0)
        pred = output.argmax(dim=1)
        correct += pred.eq(target).sum().item()
        total += data.size(0)

    avg_loss = total_loss / total
    accuracy = 100. * correct / total
    return avg_loss, accuracy


# ─── Evaluation Loop ───
def evaluate(model, loader, criterion, device):
    model.eval()
    total_loss = 0
    correct = 0
    total = 0

    with torch.no_grad():
        for data, target in loader:
            data, target = data.to(device), target.to(device)
            output = model(data)
            loss = criterion(output, target)

            total_loss += loss.item() * data.size(0)
            pred = output.argmax(dim=1)
            correct += pred.eq(target).sum().item()
            total += data.size(0)

    avg_loss = total_loss / total
    accuracy = 100. * correct / total
    return avg_loss, accuracy


# ─── Run Training ───
print(f"Training LeNet-5 on {DEVICE}")
print("=" * 60)

for epoch in range(1, EPOCHS + 1):
    train_loss, train_acc = train(model, train_loader, criterion,
                                   optimizer, DEVICE)
    test_loss, test_acc = evaluate(model, test_loader, criterion, DEVICE)

    print(f"Epoch {epoch:2d}/{EPOCHS} │ "
          f"Train Loss: {train_loss:.4f}  Acc: {train_acc:.2f}% │ "
          f"Test Loss: {test_loss:.4f}  Acc: {test_acc:.2f}%")

print("=" * 60)
print("Training complete!")
```

**Expected output (approximate):**
```
Training LeNet-5 on cuda
============================================================
Epoch  1/10 │ Train Loss: 0.2845  Acc: 91.35% │ Test Loss: 0.0892  Acc: 97.25%
Epoch  2/10 │ Train Loss: 0.0785  Acc: 97.62% │ Test Loss: 0.0578  Acc: 98.10%
Epoch  3/10 │ Train Loss: 0.0548  Acc: 98.28% │ Test Loss: 0.0445  Acc: 98.55%
...
Epoch 10/10 │ Train Loss: 0.0178  Acc: 99.43% │ Test Loss: 0.0352  Acc: 98.95%
============================================================
Training complete!
```

### Visualizing Feature Maps

```python
import matplotlib.pyplot as plt


def visualize_feature_maps(model, image, device):
    """Visualize intermediate feature maps of LeNet-5."""
    model.eval()
    activations = []

    # Hook to capture intermediate outputs
    def hook_fn(module, input, output):
        activations.append(output.detach().cpu())

    hooks = []
    hooks.append(model.conv1.register_forward_hook(hook_fn))
    hooks.append(model.conv2.register_forward_hook(hook_fn))

    # Forward pass
    with torch.no_grad():
        _ = model(image.unsqueeze(0).to(device))

    # Remove hooks
    for h in hooks:
        h.remove()

    # Plot
    fig, axes = plt.subplots(2, 8, figsize=(16, 4))
    fig.suptitle("LeNet-5 Feature Maps", fontsize=14)

    # Conv1 feature maps (6 maps)
    for i in range(6):
        axes[0, i].imshow(activations[0][0, i], cmap='viridis')
        axes[0, i].set_title(f'C1-{i}')
        axes[0, i].axis('off')
    for i in range(6, 8):
        axes[0, i].axis('off')

    # Conv2 feature maps (first 8 of 16 maps)
    for i in range(8):
        axes[1, i].imshow(activations[1][0, i], cmap='viridis')
        axes[1, i].set_title(f'C3-{i}')
        axes[1, i].axis('off')

    plt.tight_layout()
    plt.savefig('lenet5_features.png', dpi=150, bbox_inches='tight')
    plt.show()


# Usage:
# sample_image = test_dataset[0][0]
# visualize_feature_maps(model, sample_image, DEVICE)
```

---

## Applications

### 1. Handwritten Digit Recognition (Primary Application)
- US Postal Service ZIP code reading
- Bank check digit recognition
- MNIST benchmark (>99% accuracy achievable with modern tricks)

### 2. Document Recognition
- Character recognition in scanned documents
- License plate recognition (early systems)
- Form processing and data entry automation

### 3. Educational Value
- **Foundation for learning CNNs** — simple enough to understand every layer
- Used in virtually every deep learning course as the introductory CNN
- Great for understanding the mechanics of convolution, pooling, and backpropagation

### 4. Embedded/Edge Deployment
- With only ~60K parameters, LeNet-5 can run on extremely resource-constrained devices
- Microcontrollers, FPGAs, mobile sensors
- Real-time inference with minimal power consumption

### 5. Historical Significance
- Proved that neural networks could **learn visual features automatically**
- Demonstrated **end-to-end training** from pixels to predictions
- Laid the groundwork for every subsequent CNN architecture

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Architecture** | 2 Conv + 2 Pool + 3 FC |
| **Input Size** | 32 × 32 × 1 (grayscale) |
| **Activation** | tanh (original), ReLU (modern) |
| **Pooling** | 2×2 average pooling, stride 2 |
| **Parameters** | ~60,000 |
| **Filter Sizes** | 5×5 only |
| **Feature Maps** | 6 → 16 |
| **Output** | 10 classes (digits 0–9) |
| **Loss** | Euclidean RBF (original), Cross-Entropy (modern) |
| **Optimizer** | SGD (original), Adam (modern) |
| **MNIST Accuracy** | ~99% (with modern training) |
| **Key Innovation** | First successful CNN + backpropagation |
| **Limitation** | Only works on small, simple images |

---

## Revision Questions

### Q1: Architecture Recall
**What is the complete layer sequence of LeNet-5, including the dimensions at each stage?**

<details>
<summary>Answer</summary>

```
Input:  32×32×1
C1:     Conv 5×5, 6 filters  → 28×28×6  → tanh
S2:     AvgPool 2×2, stride 2 → 14×14×6
C3:     Conv 5×5, 16 filters → 10×10×16 → tanh
S4:     AvgPool 2×2, stride 2 → 5×5×16
FC1:    400 → 120 → tanh
FC2:    120 → 84  → tanh
Output: 84 → 10  → softmax
```
</details>

### Q2: Parameter Efficiency
**Why does LeNet-5 have far fewer parameters than an equivalent fully connected network? Quantify the difference for a 32×32 input.**

<details>
<summary>Answer</summary>

LeNet-5 achieves parameter efficiency through two mechanisms:
1. **Weight sharing**: The same 5×5 filter is applied at every spatial position, so a filter with 25 weights produces a full 28×28 feature map.
2. **Local connectivity**: Each neuron connects only to a 5×5 local region, not to all input pixels.

A fully connected network with comparable hidden layers would require ~469,000 parameters vs. LeNet-5's ~61,700 — about **7.6× more parameters**.
</details>

### Q3: Sparse Connectivity in C3
**Why did LeCun use a sparse connection table in the C3 layer instead of connecting all 6 input feature maps to all 16 output filters?**

<details>
<summary>Answer</summary>

Two reasons:
1. **Break symmetry**: If all filters receive the same inputs, they might learn redundant features. Different input combinations force different filters to learn different feature combinations.
2. **Reduce computation**: Sparse connections reduce the number of multiply-add operations and parameters, which mattered with 1990s hardware.
</details>

### Q4: Output Size Calculation
**If the input to LeNet-5 were 36×36×1 instead of 32×32×1 (with the same architecture), what would the output of each layer be?**

<details>
<summary>Answer</summary>

```
Input:  36×36×1
C1:     (36-5)/1 + 1 = 32 → 32×32×6
S2:     32/2 = 16            → 16×16×6
C3:     (16-5)/1 + 1 = 12   → 12×12×16
S4:     12/2 = 6             → 6×6×16
Flatten: 6×6×16 = 576
FC1:    576 → 120  (would need to change FC1 input size!)
```

The FC1 layer would need modification since it expects 400 inputs (5×5×16) but would receive 576 inputs (6×6×16).
</details>

### Q5: Historical Significance
**What were two key insights from LeNet-5 that influenced all subsequent CNN architectures?**

<details>
<summary>Answer</summary>

1. **Learned feature hierarchies**: LeNet-5 showed that early layers automatically learn simple features (edges, gradients) and deeper layers learn complex features (digit parts, shapes) — establishing the principle of hierarchical feature learning.

2. **End-to-end trainability**: The entire pipeline from raw pixels to classification could be trained with gradient descent and backpropagation, eliminating the need for hand-crafted feature engineering. This principle drives all modern deep learning.
</details>

### Q6: Modern vs. Original
**List three differences between the original LeNet-5 (1998) and modern implementations.**

<details>
<summary>Answer</summary>

| Aspect | Original (1998) | Modern |
|--------|----------------|--------|
| **Activation** | tanh / sigmoid | ReLU |
| **Pooling** | Trainable average pooling (with learnable coefficient + bias) | Standard max or average pooling |
| **Output layer** | Euclidean RBF units | Softmax with cross-entropy loss |
| **C3 connections** | Sparse (connection table) | Fully connected |
| **Optimizer** | SGD with hand-tuned schedule | Adam / AdamW |
| **Regularization** | None | Dropout, weight decay, data augmentation |
</details>

---

## Key Takeaways

```
╔══════════════════════════════════════════════════════════════════╗
║  LeNet-5 Key Takeaways                                         ║
║                                                                  ║
║  1. First practical CNN — proved CNNs work for vision tasks      ║
║  2. ~60K params — tiny by today's standards                      ║
║  3. Conv → Pool → Conv → Pool → FC pattern became the template  ║
║  4. Weight sharing + local connectivity = parameter efficiency   ║
║  5. Deployed commercially (banks, postal service) in the 1990s   ║
║  6. Foundation for understanding ALL modern CNN architectures    ║
╚══════════════════════════════════════════════════════════════════╝
```

---

[← Back to Unit Overview](../README.md) | [Next: AlexNet →](02-alexnet.md)
