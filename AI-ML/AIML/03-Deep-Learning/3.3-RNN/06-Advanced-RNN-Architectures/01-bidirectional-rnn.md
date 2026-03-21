# Bidirectional RNN

> **Unit 6, Chapter 1** — Forward and backward RNNs, concatenated hidden states, capturing future context, BiLSTM, and PyTorch `bidirectional=True`.

---

## 📋 Overview

A standard RNN only sees the **past** — h_t depends on x₁ through x_t. But many tasks benefit from seeing the **future** too. A **Bidirectional RNN** runs two separate RNNs: one forward (left→right) and one backward (right→left), then combines their outputs.

---

## 🏗️ Architecture

```
Forward RNN (→):
   x₁     x₂     x₃     x₄     x₅
    │      │      │      │      │
    ▼      ▼      ▼      ▼      ▼
  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐
  │h→₁│─▶│h→₂│─▶│h→₃│─▶│h→₄│─▶│h→₅│
  └───┘  └───┘  └───┘  └───┘  └───┘

Backward RNN (←):
   x₁     x₂     x₃     x₄     x₅
    │      │      │      │      │
    ▼      ▼      ▼      ▼      ▼
  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐
  │h←₁│◀─│h←₂│◀─│h←₃│◀─│h←₄│◀─│h←₅│
  └───┘  └───┘  └───┘  └───┘  └───┘

Combined output at each time step:
   h_t = [h→_t ; h←_t]     (concatenation)

   ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
   │h→₁   │  │h→₂   │  │h→₃   │  │h→₄   │  │h→₅   │
   │──────│  │──────│  │──────│  │──────│  │──────│
   │h←₁   │  │h←₂   │  │h←₃   │  │h←₄   │  │h←₅   │
   └──────┘  └──────┘  └──────┘  └──────┘  └──────┘
   2n dims   2n dims   2n dims   2n dims   2n dims

h→₃ sees: x₁, x₂, x₃ (past context)
h←₃ sees: x₅, x₄, x₃ (future context)
h₃ = [h→₃; h←₃] sees: ENTIRE sequence (centered at position 3)
```

---

## 🧮 Equations

```
Forward RNN:
   h→_t = f(W→_xh · x_t + W→_hh · h→_{t-1} + b→)

Backward RNN:
   h←_t = f(W←_xh · x_t + W←_hh · h←_{t+1} + b←)

Combined output at each time step:
   h_t = [h→_t ; h←_t] ∈ ℝ^{2n}

Output:
   y_t = W_y · h_t + b_y
       = W_y · [h→_t ; h←_t] + b_y

Parameters: 2× a unidirectional RNN
   (two separate sets of weights, no sharing between directions)
```

---

## ❓ Why Bidirectional?

```
Example: Named Entity Recognition

   "I work at Apple in California"

   Processing "Apple" with forward RNN:
   h→ sees: "I work at Apple" → Apple could be fruit or company

   Processing "Apple" with backward RNN:
   h← sees: "in California Apple" → more likely a company

   BiRNN at "Apple": sees BOTH contexts → correct classification!

Example: Sentiment with negation
   "I don't think this movie is good at all"

   Forward at "good": context is "I don't think this movie is good"
   Backward at "good": context is "at all good"
   Both needed to understand the sentiment is NEGATIVE
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn

# ─── Using PyTorch's built-in bidirectional ───
bilstm = nn.LSTM(
    input_size=10,
    hidden_size=20,
    num_layers=2,
    batch_first=True,
    bidirectional=True  # ← This is all you need!
)

x = torch.randn(4, 15, 10)  # batch=4, seq=15, features=10
output, (h_n, c_n) = bilstm(x)

print(f"Input:  {x.shape}")          # (4, 15, 10)
print(f"Output: {output.shape}")      # (4, 15, 40) ← 2×hidden_size!
print(f"h_n:    {h_n.shape}")        # (4, 4, 20) ← 2×num_layers
print(f"c_n:    {c_n.shape}")        # (4, 4, 20)

# Access forward/backward hidden states
# h_n shape: (num_layers * num_directions, batch, hidden)
# For 2 layers bidirectional: indices 0,1 = layer1 (fwd,bwd), 2,3 = layer2 (fwd,bwd)
h_forward_last  = h_n[-2]   # Forward direction, last layer
h_backward_last = h_n[-1]   # Backward direction, last layer

# Concatenate for classification
h_combined = torch.cat([h_forward_last, h_backward_last], dim=1)
print(f"Combined: {h_combined.shape}")  # (4, 40)

# ─── Complete BiLSTM classifier ───
class BiLSTMClassifier(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.bilstm = nn.LSTM(embed_dim, hidden_dim, batch_first=True,
                              bidirectional=True, num_layers=2, dropout=0.3)
        self.fc = nn.Linear(hidden_dim * 2, num_classes)  # 2× for bidirectional
    
    def forward(self, x):
        embedded = self.embedding(x)
        output, (h_n, _) = self.bilstm(embedded)
        
        # Concatenate last forward and backward hidden states
        h_fwd = h_n[-2]  # Last layer, forward
        h_bwd = h_n[-1]  # Last layer, backward
        combined = torch.cat([h_fwd, h_bwd], dim=1)
        
        return self.fc(combined)

model = BiLSTMClassifier(10000, 128, 256, 5)
x = torch.randint(0, 10000, (32, 50))
logits = model(x)
print(f"\nBiLSTM Classifier output: {logits.shape}")  # (32, 5)
```

---

## ⚠️ When NOT to Use Bidirectional

```
Bidirectional CANNOT be used for:
  ✗ Language modeling (predicting next word — can't see future!)
  ✗ Real-time/streaming processing (need to wait for full sequence)
  ✗ Autoregressive generation (each step depends on future)

Bidirectional IS useful for:
  ✓ Classification (sentiment, topic)
  ✓ Sequence labeling (NER, POS tagging)
  ✓ Encoding in seq2seq (encoder sees full input)
  ✓ Any task where full sequence is available before prediction
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Forward RNN | Processes sequence left-to-right (past context) |
| Backward RNN | Processes sequence right-to-left (future context) |
| Output | Concatenation [h→_t; h←_t] at each step (2n dims) |
| Parameters | 2× unidirectional (separate weights per direction) |
| Key benefit | Each position sees entire sequence context |
| Limitation | Cannot be used for autoregressive/streaming tasks |
| PyTorch | `nn.LSTM(..., bidirectional=True)` |

---

## ❓ Revision Questions

1. **Draw the bidirectional RNN for a 4-step sequence. What information does h₂ encode?**

2. **Why can't you use a bidirectional RNN for next-word prediction?**

3. **If hidden_size=100 and bidirectional=True, what is the output dimension at each time step?**

4. **For a BiLSTM with 2 layers, what is the shape of h_n? How do you extract the final forward and backward states?**

5. **Compare the computational cost of a unidirectional LSTM with hidden_size=200 vs a bidirectional LSTM with hidden_size=100.**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [When to Use Which](../05-GRU/05-when-to-use-which.md) |
| ➡️ Next | [Deep RNN](02-deep-rnn.md) |
| ⬆️ Unit Overview | [README](../README.md) |
