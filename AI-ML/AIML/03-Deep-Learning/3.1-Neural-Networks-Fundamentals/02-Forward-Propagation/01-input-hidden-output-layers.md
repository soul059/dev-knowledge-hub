# 📥 Input, Hidden & Output Layers

> **Chapter 2.1 — Understanding the Three Layer Types in Neural Networks**

| Topic | Details |
|---|---|
| **Subject** | Forward Propagation — Layer Types |
| **Prerequisites** | Chapter 1.4 (MLP Architecture) |
| **Key Insight** | Each layer type has a distinct role: input provides raw features, hidden layers learn representations, output layers produce task-specific predictions |

---

## 📌 Overview

Every neural network, regardless of its complexity, is composed of three fundamental types of layers: **input**, **hidden**, and **output**. Understanding the role and design of each layer type is critical for building effective networks. This chapter provides a detailed examination of what each layer does, how to size them, and how they relate to the concept of learned representations and feature hierarchies.

---

## 1. Input Layer — The Data Interface

### 1.1 Purpose

The input layer is the network's connection to the outside world. It does **not perform any computation** — it simply passes the raw feature values to the first hidden layer.

```
    ┌─────────────────────────────────────────────────┐
    │                 INPUT LAYER                      │
    │                                                  │
    │   • One neuron per input feature                 │
    │   • No weights, no biases, no activation         │
    │   • Just a "pass-through" of the data            │
    │   • a^[0] = x (the input vector)                 │
    │                                                  │
    │   n^[0] = number of features in the dataset      │
    └─────────────────────────────────────────────────┘
```

### 1.2 Input Dimensionality by Data Type

| Data Type | Raw Shape | Input Size (n⁰) | Notes |
|---|---|---|---|
| **Tabular (structured)** | m × n table | n features | Numerical + encoded categoricals |
| **Grayscale image** | H × W | H × W (flattened) | e.g., MNIST: 28×28 = 784 |
| **Color image** | H × W × 3 | H × W × 3 (flattened) | e.g., CIFAR: 32×32×3 = 3072 |
| **Text (word embedding)** | Sequence of d-dim vectors | d (per token) | e.g., 768 for BERT embeddings |
| **Audio (spectrogram)** | T × F | T × F (flattened) | T time steps, F frequency bins |
| **Time series** | T × n | T × n (or n per step) | T time steps, n measurements |

### 1.3 Input Preprocessing Matters

The input layer simply passes data through, but what you feed it dramatically affects training:

```
    Raw pixels [0, 255]          Normalized [0, 1]         Standardized (μ=0, σ=1)
    
    ┌──────────────────┐        ┌──────────────────┐      ┌──────────────────┐
    │ 128, 255, 0, 64  │   →    │ 0.50, 1.0, 0, .25│  →   │ 0.2, 1.5, -1.7  │
    └──────────────────┘        └──────────────────┘      └──────────────────┘
    
    ❌ Large range →              ✅ Bounded →              ✅✅ Zero-centered →
    unstable gradients            stable training            fastest convergence
```

**Common preprocessing:**
- **Min-Max Scaling**: x' = (x - min) / (max - min) → [0, 1]
- **Standardization**: x' = (x - μ) / σ → mean 0, std 1
- **Image normalization**: x' = x / 255.0, then optionally subtract channel means

---

## 2. Hidden Layers — The Representation Learners

### 2.1 Purpose

Hidden layers are where the **magic happens**. They transform the raw input into increasingly useful **learned representations** that make the final prediction task easier.

```
    ┌────────────────────────────────────────────────────────┐
    │                    HIDDEN LAYERS                        │
    │                                                        │
    │   • Perform computation: z = Wx + b, a = g(z)          │
    │   • Learn internal representations of the data         │
    │   • Each layer builds on the previous layer             │
    │   • Non-linear activation is ESSENTIAL                  │
    │   • Number and size are HYPERPARAMETERS (you choose)    │
    └────────────────────────────────────────────────────────┘
```

### 2.2 Feature Hierarchy — What Hidden Layers Learn

```
    Layer 1              Layer 2              Layer 3            Layer 4
    (Low-level)         (Mid-level)          (High-level)       (Abstract)
    
    ┌──────────┐        ┌──────────┐        ┌──────────┐       ┌──────────┐
    │ Edges    │        │ Textures │        │ Parts    │       │ Objects  │
    │ ─  │  /  │   →    │ ≋≋≋  ▓▓▓│   →    │ 👁 👃 👄│   →   │ 😀 🐱 🚗│
    │ \  ─  │  │        │ :::  ╬╬╬│        │ 🦶 🖐 👂│       │ 🏠 🌳 ✈️│
    └──────────┘        └──────────┘        └──────────┘       └──────────┘
    
    Learned features become progressively more abstract and task-specific.
    Earlier layers = general features (transferable)
    Later layers = task-specific features
```

### 2.3 Layer Sizing Guidelines

| Guideline | Recommendation | Reasoning |
|---|---|---|
| **Start size** | First hidden layer ≈ input size or slightly smaller | Sufficient capacity to learn initial features |
| **Tapering** | Gradually decrease layer sizes toward output | Progressively compress representation |
| **Expanding** | Sometimes increase, then decrease (bottleneck) | Autoencoder-style; forces compression |
| **Minimum size** | At least as large as the output | Must encode enough information for prediction |
| **Power of 2** | Use 32, 64, 128, 256, 512, 1024 | GPU optimization (memory alignment) |
| **Overfitting check** | Start big, add regularization | Better to regularize than underfit |

### 2.4 Common Hidden Layer Patterns

```
    Funnel / Tapering               Constant Width              Bottleneck
    ┌──────────────┐                ┌────────────┐              ┌──────────┐
    │  512 neurons │                │ 256 neurons│              │ 512      │
    │  256 neurons │                │ 256 neurons│              │ 128      │ ← bottleneck
    │  128 neurons │                │ 256 neurons│              │ 32       │ ← compressed
    │   64 neurons │                │ 256 neurons│              │ 128      │
    └──────────────┘                └────────────┘              │ 512      │
    Good for classification         Simple, works well          └──────────┘
                                                                Autoencoders

    Diamond / Expanding             ResNet-style
    ┌──────────┐                    ┌────────────┐
    │  128     │                    │ 64 + skip  │
    │  256     │                    │ 64 + skip  │
    │  512     │ ← widest          │ 128 + skip │
    │  256     │                    │ 128 + skip │
    │  128     │                    │ 256 + skip │
    └──────────┘                    └────────────┘
    Good for complex patterns       Skip connections!
```

### 2.5 How Many Hidden Layers?

| Problem Complexity | Recommended Depth | Example |
|---|---|---|
| **Simple / Linear** | 0-1 hidden layers | Logistic regression, simple classification |
| **Moderate** | 2-3 hidden layers | Tabular data, simple images |
| **Complex** | 4-8 hidden layers | Image classification, NLP tasks |
| **Very Complex** | 10-100+ layers | ImageNet, language models, autonomous driving |
| **State-of-the-art** | 100-1000+ layers | ResNet-152, GPT (96 layers), ViT |

---

## 3. Output Layer — The Prediction Interface

### 3.1 Purpose

The output layer produces the network's final prediction. Its design is entirely determined by the **task type**.

```
    ┌─────────────────────────────────────────────────────────┐
    │                    OUTPUT LAYER                          │
    │                                                         │
    │   • Task-specific activation function                   │
    │   • Number of neurons = number of outputs needed        │
    │   • Determines the loss function to use                 │
    │   • a^[L] = ŷ (the prediction)                          │
    └─────────────────────────────────────────────────────────┘
```

### 3.2 Output Layer Configuration — Complete Guide

| Task | # Neurons | Activation | Loss Function | Output Interpretation |
|---|---|---|---|---|
| **Binary Classification** | 1 | Sigmoid | BCELoss | P(y=1) ∈ (0,1) |
| **Multi-class (K classes)** | K | Softmax | CrossEntropyLoss | [P(y=1), ..., P(y=K)], sum=1 |
| **Multi-label** | K | Sigmoid (each) | BCELoss (each) | Independent P(labelₖ=1) |
| **Regression (single)** | 1 | None (Linear) | MSELoss | ŷ ∈ ℝ |
| **Regression (multi)** | K | None (Linear) | MSELoss | [ŷ₁, ..., ŷₖ] ∈ ℝᴷ |
| **Bounded regression** | 1 | Sigmoid × (max-min) + min | MSELoss | ŷ ∈ [min, max] |
| **Ordinal regression** | K-1 | Sigmoid (each) | BCELoss | Cumulative probabilities |

### 3.3 Output Layer Examples

```
    BINARY CLASSIFICATION              MULTI-CLASS (5 classes)
    ┌─────────────────────┐            ┌─────────────────────────┐
    │ Hidden Layer        │            │ Hidden Layer             │
    │ [h₁, h₂, ..., hₙ]  │            │ [h₁, h₂, ..., hₙ]       │
    │       │              │            │       │                  │
    │    [1 neuron]        │            │    [5 neurons]           │
    │       │              │            │       │                  │
    │    sigmoid           │            │    softmax               │
    │       │              │            │       │                  │
    │    P(spam) = 0.87    │            │  [0.05, 0.10, 0.72,     │
    │                      │            │   0.08, 0.05]            │
    └─────────────────────┘            │  Σ = 1.00                │
                                       └─────────────────────────┘
    
    REGRESSION                         MULTI-OUTPUT REGRESSION
    ┌─────────────────────┐            ┌─────────────────────────┐
    │ Hidden Layer        │            │ Hidden Layer             │
    │ [h₁, h₂, ..., hₙ]  │            │ [h₁, h₂, ..., hₙ]       │
    │       │              │            │       │                  │
    │    [1 neuron]        │            │    [3 neurons]           │
    │       │              │            │       │                  │
    │    linear (no act.)  │            │    linear (no act.)      │
    │       │              │            │       │                  │
    │    price = $342,500  │            │  [lat, lon, altitude]    │
    └─────────────────────┘            │  [37.7, -122.4, 150.2]  │
                                       └─────────────────────────┘
```

---

## 4. Complete Architecture Example

### 4.1 MNIST Digit Classification

```
    Task: Classify 28×28 grayscale handwritten digits (0-9)

    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │  INPUT          HIDDEN 1        HIDDEN 2       OUTPUT       │
    │  n⁰=784        n¹=256          n²=128         n³=10        │
    │  (28×28)        (ReLU)          (ReLU)         (Softmax)    │
    │                                                             │
    │  ┌─────┐       ┌──────┐        ┌──────┐      ┌──────┐     │
    │  │ x₁  │──────►│ ReLU │───────►│ ReLU │─────►│  P₀  │     │
    │  │ x₂  │──────►│      │───────►│      │─────►│  P₁  │     │
    │  │ ...  │──────►│ 256  │───────►│ 128  │─────►│  ... │     │
    │  │ x₇₈₄│──────►│ neur.│───────►│ neur.│─────►│  P₉  │     │
    │  └─────┘       └──────┘        └──────┘      └──────┘     │
    │                                                             │
    │  Parameters:                                                │
    │  Layer 1: 784×256 + 256 = 200,960                          │
    │  Layer 2: 256×128 + 128 = 32,896                           │
    │  Layer 3: 128×10  + 10  = 1,290                            │
    │  Total:                   235,146                           │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

### 4.2 PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torchvision import datasets, transforms

# Define the model
class MNISTClassifier(nn.Module):
    def __init__(self):
        super().__init__()
        # Input layer: 784 features (28×28 pixels, flattened)
        self.hidden1 = nn.Linear(784, 256)   # Hidden layer 1
        self.hidden2 = nn.Linear(256, 128)   # Hidden layer 2
        self.output  = nn.Linear(128, 10)    # Output layer (10 classes)
        self.dropout = nn.Dropout(0.2)       # Regularization
    
    def forward(self, x):
        # Input layer: flatten image to vector
        x = x.view(-1, 784)          # (batch, 28, 28) → (batch, 784)
        
        # Hidden layer 1: Linear → ReLU → Dropout
        x = F.relu(self.hidden1(x))
        x = self.dropout(x)
        
        # Hidden layer 2: Linear → ReLU → Dropout
        x = F.relu(self.hidden2(x))
        x = self.dropout(x)
        
        # Output layer: Linear (no softmax — CrossEntropyLoss handles it)
        x = self.output(x)
        return x

model = MNISTClassifier()
print(model)
print(f"\nTotal parameters: {sum(p.numel() for p in model.parameters()):,}")

# Layer-by-layer shapes
x = torch.randn(1, 1, 28, 28)
print(f"\nInput shape:          {x.shape}")
x_flat = x.view(-1, 784)
print(f"After flatten:        {x_flat.shape}")
h1 = F.relu(model.hidden1(x_flat))
print(f"After hidden1 (ReLU): {h1.shape}")
h2 = F.relu(model.hidden2(h1))
print(f"After hidden2 (ReLU): {h2.shape}")
out = model.output(h2)
print(f"Output (logits):      {out.shape}")
probs = F.softmax(out, dim=1)
print(f"After softmax:        {probs.shape}")
print(f"Probabilities:        {probs.detach().numpy().round(3)}")
print(f"Sum:                  {probs.sum().item():.4f}")
```

**Expected Output:**
```
MNISTClassifier(
  (hidden1): Linear(in_features=784, out_features=256, bias=True)
  (hidden2): Linear(in_features=256, out_features=128, bias=True)
  (output): Linear(in_features=128, out_features=10, bias=True)
  (dropout): Dropout(p=0.2, inplace=False)
)

Total parameters: 235,146

Input shape:          torch.Size([1, 1, 28, 28])
After flatten:        torch.Size([1, 784])
After hidden1 (ReLU): torch.Size([1, 256])
After hidden2 (ReLU): torch.Size([1, 128])
Output (logits):      torch.Size([1, 10])
After softmax:        torch.Size([1, 10])
Probabilities:        [[0.098 0.112 0.089 0.105 0.087 0.104 0.102 0.095 0.108 0.100]]
Sum:                  1.0000
```

---

## 5. The Feature Hierarchy Concept

### 5.1 What Does Each Layer "See"?

```
    Input Layer        Hidden Layer 1       Hidden Layer 2      Output Layer
    (raw pixels)       (learned features)   (combined features) (class scores)
    
    Each neuron in hidden layer 1 has learned to detect a specific pattern:
    
    Neuron h₁¹: responds to horizontal edges  ───
    Neuron h₂¹: responds to vertical edges    │
    Neuron h₃¹: responds to diagonal edges    ╱
    Neuron h₄¹: responds to curves            ⌒
    
    Each neuron in hidden layer 2 combines layer 1 features:
    
    Neuron h₁²: detects loops (combines curves + edges) → useful for 0, 6, 8, 9
    Neuron h₂²: detects straight lines (combines edges)  → useful for 1, 4, 7
    Neuron h₃²: detects intersections                    → useful for 4, 8
```

### 5.2 Representation Dimensionality

```
    784 dims ──────► 256 dims ──────► 128 dims ──────► 10 dims
    (pixel space)   (feature space)  (concept space)  (class space)
    
    ┌───────┐      ┌───────┐       ┌───────┐       ┌───────┐
    │●●●●●●●│      │●● ● ●●│       │ ●  ● ●│       │ ●     │
    │●●●●●●●│  →   │● ●● ● │   →   │●  ●  │   →   │  ●    │
    │●●●●●●●│      │ ●● ●● │       │ ●  ●  │       │       │
    │●●●●●●●│      │●● ● ●●│       │●  ●  │       │       │
    └───────┘      └───────┘       └───────┘       └───────┘
    
    High-dimensional               Progressively lower dimensions
    redundant pixels                Compressed, useful features
```

---

## 📊 Summary Table

| Layer Type | Role | Computation | Sizing | Activation |
|---|---|---|---|---|
| **Input** | Pass features to network | None (a⁰ = x) | n⁰ = # features | None |
| **Hidden** | Learn internal representations | z = Wx + b, a = g(z) | Hyperparameter (32-4096+) | ReLU, GELU, etc. |
| **Output** | Produce predictions | z = Wx + b, a = g(z) | n^L = # outputs | Task-dependent |

---

## ❓ Revision Questions

1. **What is the role of each of the three layer types? Why does the input layer not perform any computation?**

2. **For a dataset with 50 numerical features and 8 output classes, design the complete architecture specifying input size, hidden layer sizes, and output layer configuration.**

3. **Explain the concept of feature hierarchy. What kinds of features do early vs. later hidden layers typically learn?**

4. **Why do we use powers of 2 for hidden layer sizes? Is this a mathematical requirement or a practical one?**

5. **Compare the output layer configuration for: (a) binary classification, (b) multi-class with 20 classes, (c) regression predicting house price. Specify neurons, activation, and loss function for each.**

6. **What is the relationship between input preprocessing and network performance? Why is standardization typically better than raw inputs?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Universal Approximation Theorem](../01-Introduction-to-Neural-Networks/05-universal-approximation-theorem.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Weight Matrices & Bias Vectors →](02-weight-matrices-bias-vectors.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
