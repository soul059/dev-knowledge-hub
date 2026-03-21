# The Concept of Recurrence

> **Unit 1, Chapter 4** — Hidden state as memory, information flow through time, unrolled vs folded views, and parameter sharing across time steps.

---

## 📋 Overview

**Recurrence** is the fundamental idea behind RNNs: at each time step, the network uses both the current input **and** a summary of everything it has seen so far (the hidden state). This creates a loop that allows information to persist across time.

---

## 🔄 The Core Idea: Hidden State as Memory

```
┌────────────────────────────────────────────────────────────┐
│                  THE RECURRENCE EQUATION                    │
│                                                            │
│   h_t = f(W_xh · x_t + W_hh · h_{t-1} + b_h)            │
│                                                            │
│   Where:                                                   │
│     h_t     = hidden state at time t (the "memory")       │
│     x_t     = input at time t                              │
│     h_{t-1} = hidden state from previous time step        │
│     W_xh    = input-to-hidden weights                      │
│     W_hh    = hidden-to-hidden weights (recurrence!)      │
│     b_h     = bias                                         │
│     f       = activation function (usually tanh)           │
└────────────────────────────────────────────────────────────┘

The hidden state h_t is a compressed summary of ALL inputs
seen from time step 1 through time step t.

h₁ encodes: x₁
h₂ encodes: x₁, x₂
h₃ encodes: x₁, x₂, x₃
...
h_t encodes: x₁, x₂, ..., x_t
```

---

## 📐 Folded vs Unrolled View

### Folded (Compact) View

```
          x_t
           │
           ▼
     ┌───────────┐
     │           │
h_{t-1}──▶│  RNN Cell  │──▶ h_t
     │           │
     └─────┬─────┘
           │
           ▼
          y_t

The loop arrow from h_t back to h_{t-1} represents recurrence:
the output hidden state becomes the input for the next step.

Compact representation — ONE cell, self-loop:

           ┌──────┐
     x_t──▶│      │──▶ y_t
           │ RNN  │
      ┌───▶│      │───┐
      │    └──────┘   │
      │    h_{t-1}    │
      └───────────────┘
           h_t (fed back)
```

### Unrolled (Through Time) View

```
   x₁         x₂         x₃         x₄         x₅
    │          │          │          │          │
    ▼          ▼          ▼          ▼          ▼
┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐
│       │  │       │  │       │  │       │  │       │
│ RNN   │─▶│ RNN   │─▶│ RNN   │─▶│ RNN   │─▶│ RNN   │
│ Cell  │  │ Cell  │  │ Cell  │  │ Cell  │  │ Cell  │
│       │  │       │  │       │  │       │  │       │
└───┬───┘  └───┬───┘  └───┬───┘  └───┬───┘  └───┬───┘
    │          │          │          │          │
    ▼          ▼          ▼          ▼          ▼
   y₁         y₂         y₃         y₄         y₅

h₀──▶ h₁──▶ h₂──▶ h₃──▶ h₄──▶ h₅

Key insight: ALL cells share the SAME weights (W_xh, W_hh, W_hy)
This is the SAME cell, copied across time steps.
```

---

## 🔗 Information Flow Through Time

```
Step 1: h₁ = f(W_xh·x₁ + W_hh·h₀ + b)
              ↓
        h₁ captures information about x₁

Step 2: h₂ = f(W_xh·x₂ + W_hh·h₁ + b)
              ↓
        h₂ captures x₂ AND (via h₁) information about x₁

Step 3: h₃ = f(W_xh·x₃ + W_hh·h₂ + b)
              ↓
        h₃ captures x₃ AND (via h₂) information about x₁, x₂

Information propagation:
   x₁ ──▶ h₁ ──▶ h₂ ──▶ h₃ ──▶ ... ──▶ h_T
              └──▶ h₂ ──▶ h₃ ──▶ ... ──▶ h_T
                       └──▶ h₃ ──▶ ... ──▶ h_T

Each h_t is a LOSSY COMPRESSION of the entire history.
Recent information tends to be better preserved (more on this
in the vanishing gradient discussion).
```

---

## 🔄 Parameter Sharing Across Time Steps

```
┌─────────────────────────────────────────────────────────────┐
│                    PARAMETER SHARING                         │
│                                                             │
│  Time 1:  h₁ = f(W_xh · x₁ + W_hh · h₀ + b)   ← Same W  │
│  Time 2:  h₂ = f(W_xh · x₂ + W_hh · h₁ + b)   ← Same W  │
│  Time 3:  h₃ = f(W_xh · x₃ + W_hh · h₂ + b)   ← Same W  │
│  ...                                                        │
│  Time T:  h_T = f(W_xh · x_T + W_hh · h_{T-1} + b) ← Same W │
│                                                             │
│  Benefits:                                                  │
│  ✓ Constant number of parameters (regardless of T)         │
│  ✓ Can process sequences of ANY length                     │
│  ✓ Learns general rules, not position-specific rules       │
│  ✓ Knowledge transfers across positions                    │
│  ✓ Similar to weight sharing in CNNs (filters)             │
└─────────────────────────────────────────────────────────────┘
```

### Analogy: Reading a Book

```
You (the reader) are the hidden state.
Each page is an input x_t.
Your brain processes each page the SAME WAY (shared "weights").
After each page, your understanding (h_t) is updated.
By the end, h_T summarizes the entire book.

Your reading process doesn't change based on page number —
you apply the same comprehension skills to every page.
That's parameter sharing!
```

---

## 🧮 Mathematical Formulation

### Complete Recurrence Relations

```
Hidden state update:
   h_t = tanh(W_xh · x_t + W_hh · h_{t-1} + b_h)

Output computation:
   y_t = W_hy · h_t + b_y

Dimensions:
   x_t ∈ ℝ^d       (input dimension d)
   h_t ∈ ℝ^n       (hidden dimension n)
   y_t ∈ ℝ^m       (output dimension m)

   W_xh ∈ ℝ^{n×d}  (input to hidden)
   W_hh ∈ ℝ^{n×n}  (hidden to hidden — the recurrence!)
   W_hy ∈ ℝ^{m×n}  (hidden to output)
   b_h  ∈ ℝ^n      (hidden bias)
   b_y  ∈ ℝ^m      (output bias)

Total parameters: n·d + n·n + m·n + n + m
                  (INDEPENDENT of sequence length T!)
```

### Initial Hidden State

```
h₀ is typically:
   • Zero vector: h₀ = 0⃗  (most common)
   • Learned parameter: h₀ = trainable vector
   • Computed from input: h₀ = f(encoder_output)  (in seq2seq)
```

---

## 💻 Python Implementation: Recurrence From Scratch

```python
import numpy as np
import torch
import torch.nn as nn

# ─── Manual Recurrence (NumPy) ───
np.random.seed(42)

# Dimensions
d = 3    # input dimension
n = 4    # hidden dimension
T = 5    # sequence length

# Initialize weights
W_xh = np.random.randn(n, d) * 0.1   # input → hidden
W_hh = np.random.randn(n, n) * 0.1   # hidden → hidden (recurrence!)
b_h  = np.zeros(n)

# Input sequence: 5 time steps, each with 3 features
X = np.random.randn(T, d)

# Forward pass through time
h = np.zeros(n)  # h₀ = zero vector
hidden_states = [h.copy()]

print("=== Recurrence Step by Step ===\n")
for t in range(T):
    # THE RECURRENCE EQUATION
    h = np.tanh(W_xh @ X[t] + W_hh @ h + b_h)
    hidden_states.append(h.copy())
    print(f"t={t+1}: x_t = {X[t].round(3)}")
    print(f"       h_t = {h.round(3)}")
    print(f"       ||h_t|| = {np.linalg.norm(h):.4f}\n")

print("Each h_t incorporates ALL previous inputs via recurrence!")
print(f"h₅ implicitly encodes x₁, x₂, x₃, x₄, x₅")

# ─── PyTorch RNN (same concept) ───
rnn = nn.RNN(input_size=d, hidden_size=n, batch_first=True)
x_tensor = torch.tensor(X, dtype=torch.float32).unsqueeze(0)  # (1, T, d)
output, h_final = rnn(x_tensor)

print(f"\nPyTorch RNN output shape: {output.shape}")    # (1, 5, 4)
print(f"PyTorch final hidden: {h_final.shape}")          # (1, 1, 4)
```

---

## 🔍 Unrolled View as Computational Graph

```
For backpropagation, the unrolled view is essential:

          x₁        x₂        x₃
          │         │         │
          ▼         ▼         ▼
h₀ ──▶ [W_xh]   [W_xh]   [W_xh]
        [W_hh] ──▶[W_hh] ──▶[W_hh] ──▶ h₃
        [tanh]   [tanh]   [tanh]
          │         │         │
          ▼         ▼         ▼
        [W_hy]   [W_hy]   [W_hy]
          │         │         │
          ▼         ▼         ▼
         y₁        y₂        y₃
          │         │         │
          ▼         ▼         ▼
         L₁        L₂        L₃

Total Loss: L = L₁ + L₂ + L₃

Gradients flow BACKWARD through this entire graph.
This is called Backpropagation Through Time (BPTT).
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Recurrence | Using output of previous step as input to current step |
| Hidden State | Compressed memory of all previous inputs |
| Folded View | Compact diagram with self-loop on single cell |
| Unrolled View | Cell copied across time steps (for computation/BPTT) |
| Parameter Sharing | Same W_xh, W_hh, W_hy at every time step |
| h₀ | Initial hidden state, typically zero vector |
| Information Flow | x₁ influences h₁, which influences h₂, etc. |
| Lossy Compression | h_t cannot perfectly preserve all past information |

---

## ❓ Revision Questions

1. **Write the recurrence equation for h_t. What does each term represent?**

2. **What information does h₅ contain in a 5-step sequence? How does information from x₁ reach h₅?**

3. **Compare the folded and unrolled views of an RNN. Why is the unrolled view important for training?**

4. **If an RNN has input dimension d=10 and hidden dimension n=50, how many parameters are in W_xh and W_hh? Does this change if the sequence length increases from 20 to 100?**

5. **Explain parameter sharing in RNNs using an analogy. Why is it better than having separate weights for each time step?**

6. **What are three common choices for initializing h₀? Which is most commonly used?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Limitations of Feedforward](03-limitations-of-feedforward.md) |
| ➡️ Next | [RNN Architecture](../02-Basic-RNN/01-rnn-architecture.md) |
| ⬆️ Unit Overview | [README](../README.md) |
