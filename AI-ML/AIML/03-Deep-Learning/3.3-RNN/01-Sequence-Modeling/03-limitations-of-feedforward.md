# Limitations of Feedforward Networks for Sequences

> **Unit 1, Chapter 3** — Why standard feedforward networks fail at sequential data: fixed input size, no memory, no temporal modeling.

---

## 📋 Overview

Feedforward neural networks (FNNs) are powerful universal function approximators, but they have fundamental limitations when dealing with sequential data. Understanding these limitations motivates the need for **recurrent** architectures.

---

## ❌ Limitation 1: Fixed Input Size

Feedforward networks require a **fixed-dimensional input vector**.

```
Feedforward Network:
                    ┌─────────────────┐
   x ∈ ℝⁿ  ───────▶│  W₁, W₂, ..., Wₖ │───────▶  y ∈ ℝᵐ
   (FIXED size n)   └─────────────────┘          (FIXED size m)

Problem: Sentences have VARIABLE lengths!
   "Hi"                     → 1 word   → x ∈ ℝ¹ˣᵈ
   "I love deep learning"   → 4 words  → x ∈ ℝ⁴ˣᵈ
   "The quick brown fox..." → 9 words  → x ∈ ℝ⁹ˣᵈ

   How do you feed these into the same network?
```

### Common (Bad) Workarounds

```
Workaround 1: Pad/Truncate to fixed length
   "Hi"       → "Hi  <pad> <pad> <pad>"     ← Wastes computation
   "Very long sentence..." → "Very long"     ← Loses information!

Workaround 2: Concatenate all inputs
   Input = [x₁ | x₂ | ... | x_maxlen]       ← Huge input dimension
                                              ← Position-specific weights

Workaround 3: Bag of words
   Count word frequencies                     ← Loses word order entirely!
```

---

## ❌ Limitation 2: No Memory

Each input is processed **independently** — the network has no memory of previous inputs.

```
Time step 1:  x₁ = "The"    →  FNN  →  y₁
Time step 2:  x₂ = "cat"    →  FNN  →  y₂  (doesn't know about "The")
Time step 3:  x₃ = "sat"    →  FNN  →  y₃  (doesn't know about "The cat")

┌──────────────────────────────────────────────────────┐
│  Each prediction is made in COMPLETE ISOLATION       │
│  No information flows between time steps             │
│  "sat" doesn't know "The cat" came before it         │
└──────────────────────────────────────────────────────┘
```

### Why Memory Matters

```
Context 1: "The bank of the river was muddy"
                 ↑
           bank = riverbank (geographical)

Context 2: "I went to the bank to deposit money"
                          ↑
                    bank = financial institution

Without memory of previous words, a FNN processing "bank"
alone cannot determine which meaning is correct!
```

---

## ❌ Limitation 3: No Temporal Modeling

FNNs treat inputs as **unordered** — they cannot capture the notion of time or sequence.

```
Feedforward sees these as requiring DIFFERENT weights:

Position 1:  "Hello" → uses weights for position 1
Position 2:  "Hello" → uses weights for position 2  (DIFFERENT weights!)

But the meaning of "Hello" doesn't change based on position!
We want to learn ONE set of rules for processing words,
applied at EVERY position. This is PARAMETER SHARING.

FNN approach:
   x = [x₁ | x₂ | x₃ | ... | x_T]   (concatenated)
   y = W · x + b

   W has SEPARATE columns for each position:
   W = [W_pos1 | W_pos2 | W_pos3 | ... | W_pos_T]

   Problems:
   • Cannot generalize across positions
   • Parameters grow linearly with sequence length
   • Cannot handle sequences longer than training max
```

---

## ❌ Limitation 4: No Variable-Length Output

```
Machine Translation Example:
   Input:  "I love you"     (3 words, English)
   Output: "Je t'aime"      (2 words, French)
   Output: "Ich liebe dich"  (3 words, German)
   Output: "私はあなたを愛しています"  (varies, Japanese)

FNN: Fixed output dimension → cannot produce variable-length output!
```

---

## 🧮 Mathematical Comparison

### Feedforward Network

```
y = f(W · x + b)

where x must have fixed dimension d_in
      y has fixed dimension d_out
      No notion of time t
```

### What We Need (Recurrent Processing)

```
h_t = f(W_xh · x_t + W_hh · h_{t-1} + b)

where x_t is input at time t (any T)
      h_{t-1} carries information from the past
      Same weights W_xh, W_hh at every time step
      Sequence can be any length
```

---

## 📊 Side-by-Side Comparison

```
┌────────────────────────────────────────────────────────────────┐
│          FEEDFORWARD                    RECURRENT              │
│                                                                │
│  x₁  x₂  x₃  x₄  x₅              x₁  x₂  x₃  x₄  x₅      │
│  │   │   │   │   │               │   │   │   │   │        │
│  ▼   ▼   ▼   ▼   ▼               ▼   ▼   ▼   ▼   ▼        │
│  ┌───────────────────┐          ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐   │
│  │  Big weight matrix│          │h₁│→│h₂│→│h₃│→│h₄│→│h₅│   │
│  │  (position-specific)│        └──┘ └──┘ └──┘ └──┘ └──┘   │
│  └───────┬───────────┘           │   │   │   │   │        │
│          ▼                       ▼   ▼   ▼   ▼   ▼        │
│         [y]                     y₁  y₂  y₃  y₄  y₅        │
│                                                                │
│  • Fixed length only            • Any sequence length          │
│  • No memory                    • Hidden state = memory        │
│  • Huge parameters              • Shared parameters            │
│  • No temporal awareness        • Natural time modeling        │
│  • Independent inputs           • Context-aware processing    │
└────────────────────────────────────────────────────────────────┘
```

---

## 💻 Demonstration: FNN Failure on Sequences

```python
import torch
import torch.nn as nn
import numpy as np

# Task: Predict the next number in a sequence
# Pattern: each number = sum of previous two (Fibonacci-like)

def generate_sequence(n, start=(1, 1)):
    seq = list(start)
    for _ in range(n - 2):
        seq.append(seq[-1] + seq[-2])
    return seq

seq = generate_sequence(10)
print(f"Sequence: {seq}")
# [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]

# ─── Approach 1: Feedforward (fails) ───
class FeedforwardPredictor(nn.Module):
    """Takes SINGLE number, tries to predict next — impossible!"""
    def __init__(self):
        super().__init__()
        self.fc = nn.Sequential(
            nn.Linear(1, 32), nn.ReLU(),
            nn.Linear(32, 1)
        )
    
    def forward(self, x):
        return self.fc(x)

# Given x=5, should output be 8? But 5 could come from
# different sequences with different "next" values!
# Without knowing the PREVIOUS element, prediction is impossible.

# ─── Approach 2: Fixed window (limited) ───
class WindowPredictor(nn.Module):
    """Takes fixed window of k previous numbers"""
    def __init__(self, window_size=3):
        super().__init__()
        self.fc = nn.Sequential(
            nn.Linear(window_size, 32), nn.ReLU(),
            nn.Linear(32, 1)
        )
    
    def forward(self, x):
        return self.fc(x)

# Window = [3, 5, 8] → predict 13 ✓
# But: window size is FIXED, and long-range dependencies are lost

# ─── Approach 3: RNN (correct approach) ───
class RNNPredictor(nn.Module):
    """Processes sequence of ANY length with memory"""
    def __init__(self, hidden_dim=32):
        super().__init__()
        self.rnn = nn.RNN(1, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, 1)
    
    def forward(self, x):
        # x shape: (batch, seq_len, 1) — any seq_len!
        output, h_n = self.rnn(x)
        return self.fc(h_n.squeeze(0))

print("\nFeedforward: Cannot handle variable-length sequences")
print("Window FNN:  Fixed context, misses long-range patterns")
print("RNN:         Variable length + memory = correct approach")
```

---

## 📋 Summary Table

| Limitation | Description | RNN Solution |
|-----------|-------------|--------------|
| Fixed Input Size | Must predefine input dimension | Process one step at a time, any length |
| No Memory | Each input processed independently | Hidden state carries past information |
| No Temporal Modeling | Cannot capture order/time | Sequential processing respects order |
| Position-Specific Weights | Different weights per position | Parameter sharing across time steps |
| Fixed Output Size | Cannot produce variable-length output | Generate outputs step by step |
| Parameter Growth | Parameters ∝ max sequence length | Fixed parameters regardless of length |

---

## ❓ Revision Questions

1. **If you concatenate a 5-word sentence into a single vector for a feedforward network, what happens when you encounter a 10-word sentence at test time?**

2. **Explain with an example why "no memory" is problematic for language understanding.**

3. **What is the difference between position-specific weights (FNN) and shared weights (RNN)? Why is sharing better for sequences?**

4. **A feedforward network is trained on sequences padded to length 100. How does this waste computation for a 3-word input?**

5. **Why can't a feedforward network solve the problem "given a sequence of numbers following a pattern, predict the next number" as well as an RNN?**

6. **Name two real-world tasks where the input and output sequences have different lengths. Why does this make feedforward networks unsuitable?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Types of Sequence Problems](02-types-of-sequence-problems.md) |
| ➡️ Next | [The Concept of Recurrence](04-recurrence-concept.md) |
| ⬆️ Unit Overview | [README](../README.md) |
