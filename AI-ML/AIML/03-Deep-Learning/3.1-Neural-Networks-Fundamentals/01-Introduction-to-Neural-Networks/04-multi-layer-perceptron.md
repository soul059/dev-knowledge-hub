# 🏗️ Multi-Layer Perceptron (MLP)

> **Chapter 1.4 — Building Deeper Networks: From Single Neurons to Universal Function Approximators**

| Topic | Details |
|---|---|
| **Subject** | Neural Networks — Multi-Layer Perceptron Architecture |
| **Prerequisites** | Chapters 1.2-1.3 (Perceptron, Activation Functions) |
| **Key Insight** | By stacking layers of neurons with non-linear activations, MLPs can learn arbitrarily complex functions |

---

## 📌 Overview

The Multi-Layer Perceptron (MLP) — also called a feedforward neural network or dense network — is the simplest form of deep learning architecture. It overcomes the single perceptron's limitation (can only learn linear boundaries) by stacking multiple layers of neurons. Each layer transforms the data into increasingly useful representations, culminating in a prediction. Understanding MLPs is essential because every modern deep learning architecture (CNNs, RNNs, Transformers) builds upon these fundamental principles.

---

## 1. MLP Architecture

### 1.1 The Three Types of Layers

```
    ┌────────────────────────────────────────────────────────────┐
    │                   MULTI-LAYER PERCEPTRON                    │
    │                                                            │
    │   Input Layer      Hidden Layers       Output Layer         │
    │   (Layer 0)      (Layers 1 to L-1)     (Layer L)           │
    │                                                            │
    │   ┌───┐          ┌───┐    ┌───┐        ┌───┐              │
    │   │ x₁├─────────►│ h₁├───►│ h₁├───────►│ o₁│              │
    │   ├───┤    ╲  ╱  ├───┤    ├───┤   ╲ ╱  ├───┤              │
    │   │ x₂├────╲╱───►│ h₂├───►│ h₂├────╲──►│ o₂│              │
    │   ├───┤    ╱╲    ├───┤    ├───┤   ╱ ╲  ├───┤              │
    │   │ x₃├───╱──╲──►│ h₃├───►│ h₃├──╱───►│ o₃│              │
    │   ├───┤  ╱    ╲  ├───┤    ├───┤        └───┘              │
    │   │ x₄├─╱──────►│ h₄│    └───┘                            │
    │   └───┘          └───┘                                     │
    │                                                            │
    │   n⁰=4 neurons  n¹=4    n²=3         n³=3 neurons         │
    │   (features)     neurons  neurons     (outputs/classes)    │
    │                                                            │
    │   Does NOT        Apply non-linear    Task-specific        │
    │   compute —       activations         activation           │
    │   just passes     (ReLU, GELU, etc.)  (softmax, sigmoid,  │
    │   features                            linear)              │
    └────────────────────────────────────────────────────────────┘
```

### 1.2 Fully Connected (Dense) Layers

In a fully connected layer, **every neuron** in layer l is connected to **every neuron** in layer l-1:

```
Layer l-1 (3 neurons)    →    Layer l (4 neurons)
    ○─────────────────────────────○
    │╲                         ╱│
    │  ╲                     ╱  │
    │    ╲                 ╱    │
    ○──────╲─────────────╱──────○
    │        ╲         ╱        │
    │          ╲     ╱          │
    │            ╲ ╱            │
    ○──────────────╳────────────○
                 ╱ ╲
               ╱     ╲          ○

    Total connections: 3 × 4 = 12 weights + 4 biases = 16 parameters
```

**Key property**: "Fully connected" = "Dense" — every pair of neurons between adjacent layers has its own weight.

### 1.3 Depth vs Width

| Concept | Definition | Effect |
|---|---|---|
| **Depth** | Number of layers (L) | Enables learning hierarchical features; more complex functions |
| **Width** | Number of neurons per layer (n^[l]) | More capacity per layer; wider representation |
| **Shallow & Wide** | Few layers, many neurons each | Can approximate functions but may need exponentially many neurons |
| **Deep & Narrow** | Many layers, fewer neurons each | More parameter-efficient; learns compositional features |

```
    Shallow & Wide                  Deep & Narrow
    ┌─────────────┐                 ┌───┐
    │ ○ ○ ○ ○ ○ ○ │                 │ ○ │
    │ ○ ○ ○ ○ ○ ○ │                 ├───┤
    │ ○ ○ ○ ○ ○ ○ │    vs          │ ○ │
    └─────────────┘                 ├───┤
    3 layers × 6 neurons            │ ○ │
    = 18 neurons                    ├───┤
                                    │ ○ │
                                    ├───┤
                                    │ ○ │
                                    ├───┤
                                    │ ○ │
                                    └───┘
                                    6 layers × 1 neuron
                                    = 6 neurons

    Both can approximate the same function,
    but deep networks do it more efficiently
```

---

## 2. Mathematical Formulation

### 2.1 Notation Conventions

```
Superscript [l] denotes layer l

    L         — total number of layers (not counting input)
    n^[l]     — number of neurons in layer l
    W^[l]     — weight matrix for layer l,    shape: (n^[l], n^[l-1])
    b^[l]     — bias vector for layer l,      shape: (n^[l], 1)
    z^[l]     — pre-activation for layer l,   shape: (n^[l], 1)
    a^[l]     — activation for layer l,       shape: (n^[l], 1)
    g^[l](·)  — activation function for layer l
    
    a^[0] = x  (input layer — just the raw features)
```

### 2.2 Forward Pass Equations

For each layer l = 1, 2, ..., L:

```
    z^[l] = W^[l] · a^[l-1] + b^[l]      ← linear transformation
    a^[l] = g^[l](z^[l])                  ← non-linear activation

Expanded for a 3-layer network:

    Layer 1: z^[1] = W^[1]·x + b^[1]     a^[1] = g^[1](z^[1])
    Layer 2: z^[2] = W^[2]·a^[1] + b^[2]  a^[2] = g^[2](z^[2])
    Layer 3: z^[3] = W^[3]·a^[2] + b^[3]  a^[3] = g^[3](z^[3])

    Final output: ŷ = a^[L] = a^[3]
```

### 2.3 Detailed Example — 3-Layer MLP

```
Architecture: Input(3) → Hidden1(4, ReLU) → Hidden2(3, ReLU) → Output(2, Softmax)

Layer dimensions:
    n^[0] = 3  (input features)
    n^[1] = 4  (first hidden layer)
    n^[2] = 3  (second hidden layer)
    n^[3] = 2  (output classes)
```

```
         INPUT           HIDDEN 1          HIDDEN 2         OUTPUT
         (n=3)            (n=4)             (n=3)            (n=2)

         ┌───┐   W^[1]   ┌───┐   W^[2]   ┌───┐   W^[3]   ┌───┐
    x₁ = │1.0├──────────►│ h₁├──────────►│ h₁├──────────►│ o₁│ = ŷ₁
         ├───┤  (4×3)    ├───┤  (3×4)    ├───┤  (2×3)    ├───┤
    x₂ = │2.0├──────────►│ h₂├──────────►│ h₂├──────────►│ o₂│ = ŷ₂
         ├───┤           ├───┤           ├───┤           └───┘
    x₃ = │3.0├──────────►│ h₃├──────────►│ h₃│
         └───┘           ├───┤           └───┘
                         │ h₄│
                         └───┘

    W^[1]: (4×3)    b^[1]: (4×1)
    W^[2]: (3×4)    b^[2]: (3×1)
    W^[3]: (2×3)    b^[3]: (2×1)
```

---

## 3. Counting Parameters

### 3.1 Formula

For a fully connected network:

```
Total parameters = Σ (weights in layer l + biases in layer l)
                 = Σ (n^[l] × n^[l-1] + n^[l])
                   l=1 to L
```

### 3.2 Worked Example

For our 3-layer MLP: Input(3) → Hidden(4) → Hidden(3) → Output(2)

```
Layer 1: W^[1] = 4 × 3 = 12 weights + 4 biases = 16 parameters
Layer 2: W^[2] = 3 × 4 = 12 weights + 3 biases = 15 parameters
Layer 3: W^[3] = 2 × 3 =  6 weights + 2 biases =  8 parameters
                                                  ────────────
                                          Total:  39 parameters
```

### 3.3 Parameter Count for Common Architectures

| Architecture | Layers | Parameters |
|---|---|---|
| 784 → 128 → 10 | 2 | 784×128+128 + 128×10+10 = **101,770** |
| 784 → 256 → 128 → 64 → 10 | 4 | **235,146** |
| 784 → 512 → 512 → 10 | 3 | **668,170** |
| 3072 → 1024 → 512 → 256 → 10 | 4 | **3,935,498** |

> As networks get wider or deeper, parameter counts grow quickly. A modern large language model can have **billions** of parameters.

---

## 4. What MLPs Learn — Feature Hierarchy

### 4.1 Representation Learning

Each hidden layer learns to transform the input into increasingly useful representations:

```
    RAW INPUT              HIDDEN 1              HIDDEN 2            OUTPUT
    (pixel values)      (edge detectors)     (shape detectors)    (classification)

    ┌──────────┐        ┌──────────┐        ┌──────────┐        ┌──────┐
    │ 0.2 0.8  │        │ edge ↗   │        │ circle ○ │        │ cat  │
    │ 0.1 0.9  │  ───►  │ edge →   │  ───►  │ line ─   │  ───►  │ 0.85 │
    │ 0.3 0.7  │        │ edge ↘   │        │ angle ∟  │        │ dog  │
    │ 0.5 0.4  │        │ edge ↓   │        │ curve ⌒  │        │ 0.15 │
    └──────────┘        └──────────┘        └──────────┘        └──────┘

    Simple features ──────────────────────────────► Complex features
```

### 4.2 Non-Linear Decision Boundaries

Unlike a single perceptron (linear boundary), MLPs can learn complex boundaries:

```
    Single Perceptron              MLP (2+ layers)

    x₂                            x₂
    │      ╱                       │     ╭──────╮
    │    ╱  ●●                     │    │ ●●    │
    │  ╱  ●●●                      │   │ ●●●    │
    │╱  ○○○○                       │   │  ○○○○  │
    │ ○○○○                         │    │ ○○○○ │
    ┼──────── x₁                   │     ╰──────╯
                                   ┼──────────── x₁
    Linear boundary only           Can learn ANY shape!
```

---

## 5. MLP for Classification vs Regression

### 5.1 Output Layer Configuration

| Task | Output Activation | Loss Function | Output Interpretation |
|---|---|---|---|
| **Binary Classification** | Sigmoid (1 neuron) | Binary Cross-Entropy | P(class=1) |
| **Multi-class Classification** | Softmax (K neurons) | Categorical Cross-Entropy | P(class=k) for each k |
| **Multi-label Classification** | Sigmoid (K neurons) | Binary CE per label | Independent P(label=k) |
| **Regression (single)** | Linear / None (1 neuron) | MSE | Predicted value |
| **Regression (multi)** | Linear / None (K neurons) | MSE | K predicted values |

### 5.2 Architecture Patterns

```
BINARY CLASSIFICATION                 MULTI-CLASS (K=5)
Input → Hidden → Hidden → [1, σ]     Input → Hidden → Hidden → [5, softmax]
                           ↓                                      ↓
                         P(y=1)                              [P₁,P₂,P₃,P₄,P₅]

REGRESSION                            MULTI-OUTPUT REGRESSION
Input → Hidden → Hidden → [1, —]     Input → Hidden → Hidden → [3, —]
                           ↓                                      ↓
                         ŷ ∈ ℝ                               [ŷ₁, ŷ₂, ŷ₃]
```

---

## 6. Complete Python Implementation

```python
import numpy as np

class MLP:
    """
    Multi-Layer Perceptron from scratch using NumPy.
    Supports arbitrary architecture with ReLU hidden layers.
    """
    
    def __init__(self, layer_sizes, activation='relu', output_activation='softmax'):
        """
        Parameters:
            layer_sizes: list of ints, e.g., [784, 128, 64, 10]
            activation: hidden layer activation ('relu', 'tanh', 'sigmoid')
            output_activation: output layer activation ('softmax', 'sigmoid', 'linear')
        """
        self.layer_sizes = layer_sizes
        self.L = len(layer_sizes) - 1  # number of weight layers
        self.activation = activation
        self.output_activation = output_activation
        
        # Initialize weights (He initialization for ReLU)
        self.weights = {}
        self.biases = {}
        for l in range(1, self.L + 1):
            scale = np.sqrt(2.0 / layer_sizes[l-1])  # He init
            self.weights[l] = np.random.randn(layer_sizes[l], layer_sizes[l-1]) * scale
            self.biases[l] = np.zeros((layer_sizes[l], 1))
        
        self._print_architecture()
    
    def _print_architecture(self):
        print(f"MLP Architecture: {' → '.join(map(str, self.layer_sizes))}")
        total_params = 0
        for l in range(1, self.L + 1):
            n_w = self.weights[l].size
            n_b = self.biases[l].size
            total_params += n_w + n_b
            act = self.output_activation if l == self.L else self.activation
            print(f"  Layer {l}: {self.layer_sizes[l-1]} → {self.layer_sizes[l]} "
                  f"({n_w} weights + {n_b} biases = {n_w + n_b} params) [{act}]")
        print(f"  Total parameters: {total_params:,}")
    
    def _activate(self, z, layer):
        """Apply activation function"""
        if layer < self.L:
            # Hidden layer
            if self.activation == 'relu':
                return np.maximum(0, z)
            elif self.activation == 'tanh':
                return np.tanh(z)
            elif self.activation == 'sigmoid':
                return 1 / (1 + np.exp(-np.clip(z, -500, 500)))
        else:
            # Output layer
            if self.output_activation == 'softmax':
                z_stable = z - np.max(z, axis=0, keepdims=True)
                exp_z = np.exp(z_stable)
                return exp_z / np.sum(exp_z, axis=0, keepdims=True)
            elif self.output_activation == 'sigmoid':
                return 1 / (1 + np.exp(-np.clip(z, -500, 500)))
            else:  # linear
                return z
    
    def forward(self, X):
        """
        Forward pass through the network.
        
        Parameters:
            X: input matrix, shape (n_features, m_samples)
        
        Returns:
            output: predictions, shape (n_output, m_samples)
        """
        self.cache = {'a0': X}
        a = X
        
        for l in range(1, self.L + 1):
            z = self.weights[l] @ a + self.biases[l]
            a = self._activate(z, l)
            self.cache[f'z{l}'] = z
            self.cache[f'a{l}'] = a
        
        return a
    
    def predict(self, X):
        """Get predictions (class labels for classification)"""
        probs = self.forward(X)
        if self.output_activation == 'softmax':
            return np.argmax(probs, axis=0)
        elif self.output_activation == 'sigmoid':
            return (probs >= 0.5).astype(int)
        else:
            return probs


# ──── Demo ────
np.random.seed(42)

# Create a simple MLP for MNIST-like classification
mlp = MLP(layer_sizes=[784, 256, 128, 10], activation='relu', output_activation='softmax')

# Forward pass with random data (batch of 5 samples)
X_dummy = np.random.randn(784, 5)
output = mlp.forward(X_dummy)

print(f"\nInput shape:  {X_dummy.shape}  (784 features × 5 samples)")
print(f"Output shape: {output.shape}  (10 classes × 5 samples)")
print(f"Output (sample 1): {output[:, 0].round(4)}")
print(f"Sum of probabilities: {output[:, 0].sum():.6f}")
print(f"Predicted classes: {mlp.predict(X_dummy)}")
```

**Expected Output:**
```
MLP Architecture: 784 → 256 → 128 → 10
  Layer 1: 784 → 256 (200704 weights + 256 biases = 200960 params) [relu]
  Layer 2: 256 → 128 (32768 weights + 128 biases = 32896 params) [relu]
  Layer 3: 128 → 10 (1280 weights + 10 biases = 1290 params) [softmax]
  Total parameters: 235,146

Input shape:  (784, 5)  (784 features × 5 samples)
Output shape: (10, 5)  (10 classes × 5 samples)
Output (sample 1): [0.0723 0.1148 0.0854 0.1032 0.1041 0.0967 0.1189 0.1135 0.0911 0.1001]
Sum of probabilities: 1.000000
Predicted classes: [6 6 3 6 6]
```

---

## 7. PyTorch MLP Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class PyTorchMLP(nn.Module):
    """Production-quality MLP in PyTorch"""
    
    def __init__(self, input_size, hidden_sizes, output_size, dropout=0.2):
        super().__init__()
        
        layers = []
        prev_size = input_size
        
        for hidden_size in hidden_sizes:
            layers.append(nn.Linear(prev_size, hidden_size))
            layers.append(nn.ReLU())
            layers.append(nn.Dropout(dropout))
            prev_size = hidden_size
        
        layers.append(nn.Linear(prev_size, output_size))
        # No activation here — use with CrossEntropyLoss (includes softmax)
        
        self.network = nn.Sequential(*layers)
    
    def forward(self, x):
        return self.network(x)

# Create model
model = PyTorchMLP(input_size=784, hidden_sizes=[256, 128, 64], output_size=10)
print(model)
print(f"\nTotal parameters: {sum(p.numel() for p in model.parameters()):,}")

# Forward pass
x = torch.randn(32, 784)  # Batch of 32 samples
logits = model(x)
print(f"Output shape: {logits.shape}")  # (32, 10)

# Get predictions
probs = F.softmax(logits, dim=1)
predictions = torch.argmax(probs, dim=1)
print(f"Predictions shape: {predictions.shape}")
```

---

## 8. Applications of MLPs

| Application | Input | Architecture | Output |
|---|---|---|---|
| **Digit recognition** | 784 pixels (28×28 flattened) | 784→256→128→10 | Digit class (0-9) |
| **Tabular data** | Structured features | Features→128→64→1 | Regression/Classification |
| **Recommendation** | User + Item embeddings | 256→128→64→1 | Rating prediction |
| **Function approximation** | x values | 1→64→64→1 | f(x) estimate |
| **Anomaly detection** | Sensor readings | Autoencoder MLP | Reconstruction error |

---

## 📊 Summary Table

| Concept | Key Point |
|---|---|
| **MLP** | Input → Hidden layers → Output; fully connected; feedforward |
| **Fully connected** | Every neuron connects to every neuron in adjacent layer |
| **Depth** | Number of layers — enables hierarchical feature learning |
| **Width** | Neurons per layer — capacity within each layer |
| **Forward pass** | z^[l] = W^[l]·a^[l-1] + b^[l], a^[l] = g(z^[l]) |
| **Parameters** | Total = Σ(n^[l] × n^[l-1] + n^[l]) for l=1..L |
| **Hidden activation** | ReLU (default), GELU (Transformers) |
| **Output activation** | Softmax (multi-class), Sigmoid (binary), Linear (regression) |
| **Universal approximation** | Single hidden layer with enough neurons can approximate any continuous function |
| **In practice** | Deeper is more parameter-efficient than wider |

---

## ❓ Revision Questions

1. **Draw the architecture of an MLP with input size 5, two hidden layers of 8 and 4 neurons, and output size 3. How many total parameters does it have?**

2. **Why do we need non-linear activations between layers? Prove mathematically that without them, a 10-layer network is equivalent to a single linear transformation.**

3. **For a classification task with 100 input features and 5 output classes, design an appropriate MLP architecture. Specify layer sizes, activations, and the loss function.**

4. **Explain the difference between depth and width. Why might a deeper network be preferred over a wider one?**

5. **Write the forward pass equations for a 2-layer MLP (one hidden layer). What are the shapes of all weight matrices and bias vectors?**

6. **What is the difference between the output layer configuration for binary classification, multi-class classification, and regression? Give specific activations and loss functions for each.**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Activation Functions](03-activation-functions.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Universal Approximation Theorem →](05-universal-approximation-theorem.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*
