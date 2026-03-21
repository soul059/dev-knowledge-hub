# Truncated Backpropagation Through Time

> **Unit 9, Chapter 1** — Practical strategies for training RNNs on long sequences by truncating gradient computation to manageable windows.

---

## 📋 Overview

Full Backpropagation Through Time (BPTT) requires unrolling the entire sequence and computing gradients all the way back to the first time step. For sequences of thousands or millions of steps, this is **computationally infeasible** — both in memory and time. **Truncated BPTT (TBPTT)** solves this by limiting how far back gradients flow while still allowing the forward pass to process the full sequence.

---

## 🔍 The Problem with Full BPTT

### Memory and Computation Cost

For a sequence of length `T`:

```
Full BPTT Requirements:
┌──────────────────────────────────────────────────────┐
│  Memory:  O(T)  — must store all hidden states       │
│  Time:    O(T)  — backprop through all time steps     │
│  For T = 10,000: store 10,000 hidden state vectors   │
│  For T = 1,000,000: practically impossible           │
└──────────────────────────────────────────────────────┘
```

### Real-World Sequence Lengths

| Domain | Typical Sequence Length |
|--------|----------------------|
| Short text (tweet) | 20–50 tokens |
| Document | 500–5,000 tokens |
| Audio (1 min @ 16kHz) | 960,000 samples |
| Video (1 min @ 30fps) | 1,800 frames |
| Financial tick data (1 day) | 50,000+ ticks |

Full BPTT is simply not practical for most real-world sequential data.

---

## ✂️ Truncated BPTT: The Core Idea

### Concept

Instead of backpropagating through the entire sequence, we:
1. **Forward pass** through `k₁` time steps
2. **Backward pass** through only the last `k₂` time steps (where `k₂ ≤ k₁`)
3. **Carry forward** the hidden state to the next chunk (no reset)
4. Repeat until the sequence is consumed

```
Full sequence: x₁, x₂, x₃, x₄, x₅, x₆, x₇, x₈, x₉, x₁₀, x₁₁, x₁₂

Truncated BPTT (k₁ = k₂ = 4):

  Chunk 1:  x₁ ─→ x₂ ─→ x₃ ─→ x₄     ← Backprop through these 4 steps
            ◄──── ◄──── ◄──── ◄────     Update weights

            h₄ carried forward ─────┐
                                     ▼
  Chunk 2:  x₅ ─→ x₆ ─→ x₇ ─→ x₈     ← Backprop through these 4 steps
            ◄──── ◄──── ◄──── ◄────     Update weights

            h₈ carried forward ─────┐
                                     ▼
  Chunk 3:  x₉ ─→ x₁₀ ─→ x₁₁ ─→ x₁₂  ← Backprop through these 4 steps
            ◄──── ◄──── ◄──── ◄────     Update weights
```

### Key Insight

The **hidden state** flows forward across chunk boundaries (preserving context), but **gradients** are truncated at chunk boundaries (limiting computation).

---

## 📐 Mathematical Formulation

### Full BPTT Gradient

For the total loss L = Σₜ Lₜ, the gradient w.r.t. Wₕₕ is:

```
∂L/∂W_hh = Σ_{t=1}^{T} Σ_{k=1}^{t} (∂Lₜ/∂hₜ) · (∏_{j=k+1}^{t} ∂hⱼ/∂hⱼ₋₁) · (∂hₖ/∂W_hh)
```

### Truncated BPTT Gradient (truncation length τ)

```
∂L/∂W_hh ≈ Σ_{t=1}^{T} Σ_{k=max(1, t-τ+1)}^{t} (∂Lₜ/∂hₜ) · (∏_{j=k+1}^{t} ∂hⱼ/∂hⱼ₋₁) · (∂hₖ/∂W_hh)
```

The inner sum is limited to at most `τ` terms instead of `t` terms.

### What We Lose

```
Full BPTT captures dependencies:    t ─────────────────── 1  (length T)
Truncated BPTT captures:            t ──────── t-τ+1       (length τ)

Dependencies beyond τ steps are NOT captured by gradients,
but hidden state still carries some information forward.
```

---

## 🔧 Variants of Truncated BPTT

### Variant 1: k₁ = k₂ (Most Common)

Forward and backward use the same chunk size.

```
Forward:  ──[k steps]──┃──[k steps]──┃──[k steps]──
Backward: ◄──[k steps]──┃◄──[k steps]──┃◄──[k steps]──
Update:        ✓                ✓              ✓
```

### Variant 2: k₁ > k₂

Forward through more steps than we backprop through. Hidden state has richer context but gradients still truncated.

```
Forward:  ────────[k₁ = 8 steps]────────┃────────[k₁ = 8]────────
Backward:              ◄──[k₂ = 4]──────┃            ◄──[k₂ = 4]──
```

### Variant 3: k₁ = 1, k₂ = τ (Real-Time TBPTT)

Update after every single time step, but backprop through the last τ steps. Used in online/streaming settings.

```
t=1: Forward x₁, backprop through 1 step
t=2: Forward x₂, backprop through 2 steps
t=3: Forward x₃, backprop through 3 steps
...
t=τ: Forward xₜ, backprop through τ steps
t=τ+1: Forward x_{τ+1}, backprop through τ steps (truncated)
```

---

## 🧮 Worked Example: TBPTT with k=3

### Setup

Sequence: `x₁, x₂, x₃, x₄, x₅, x₆` (length 6, truncation k=3)

```
Initial: h₀ = [0, 0]

Chunk 1: Steps 1-3
──────────────────
Forward:
  h₁ = tanh(W_xh·x₁ + W_hh·h₀ + b)     Loss: L₁
  h₂ = tanh(W_xh·x₂ + W_hh·h₁ + b)     Loss: L₂
  h₃ = tanh(W_xh·x₃ + W_hh·h₂ + b)     Loss: L₃

Backward (only within chunk):
  ∂L₃/∂W = ∂L₃/∂h₃ · (∂h₃/∂W)
  ∂L₂/∂W = ∂L₂/∂h₂ · (∂h₂/∂W)    ← only 1 step back
         + ∂L₃/∂h₃ · (∂h₃/∂h₂) · (∂h₂/∂W)
  ...similar for L₁

  Update weights using accumulated gradients
  Save h₃ (DETACH from computation graph)

Chunk 2: Steps 4-6
──────────────────
Forward:
  h₄ = tanh(W_xh·x₄ + W_hh·h₃ + b)     Loss: L₄   ← h₃ treated as constant
  h₅ = tanh(W_xh·x₅ + W_hh·h₄ + b)     Loss: L₅
  h₆ = tanh(W_xh·x₆ + W_hh·h₅ + b)     Loss: L₆

Backward (only within chunk):
  Gradients computed for steps 4-6 only
  Cannot flow back to steps 1-3 (h₃ detached)

  Update weights
```

---

## 💻 PyTorch Implementation

### Basic TBPTT Training Loop

```python
import torch
import torch.nn as nn

class SimpleRNN(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super().__init__()
        self.hidden_size = hidden_size
        self.rnn = nn.RNN(input_size, hidden_size, batch_first=True)
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x, hidden):
        out, hidden = self.rnn(x, hidden)
        out = self.fc(out)
        return out, hidden

def train_tbptt(model, data, targets, chunk_size, epochs, lr=0.001):
    """
    Train RNN using Truncated BPTT.
    
    Args:
        model: RNN model
        data: input tensor [batch, total_seq_len, input_size]
        targets: target tensor [batch, total_seq_len, output_size]
        chunk_size: truncation length (k₁ = k₂)
        epochs: number of training epochs
        lr: learning rate
    """
    criterion = nn.MSELoss()
    optimizer = torch.optim.Adam(model.parameters(), lr=lr)
    
    batch_size = data.size(0)
    seq_len = data.size(1)
    
    for epoch in range(epochs):
        # Initialize hidden state at start of each epoch
        hidden = torch.zeros(1, batch_size, model.hidden_size)
        epoch_loss = 0.0
        num_chunks = 0
        
        for start in range(0, seq_len, chunk_size):
            end = min(start + chunk_size, seq_len)
            
            # Extract chunk
            x_chunk = data[:, start:end, :]
            y_chunk = targets[:, start:end, :]
            
            # CRITICAL: Detach hidden state from previous computation graph
            hidden = hidden.detach()
            
            # Forward pass through chunk
            output, hidden = model(x_chunk, hidden)
            
            # Compute loss for this chunk
            loss = criterion(output, y_chunk)
            
            # Backward pass (only through this chunk)
            optimizer.zero_grad()
            loss.backward()
            
            # Optional: gradient clipping
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=5.0)
            
            optimizer.step()
            
            epoch_loss += loss.item()
            num_chunks += 1
        
        if (epoch + 1) % 10 == 0:
            print(f"Epoch {epoch+1}, Avg Loss: {epoch_loss/num_chunks:.4f}")

# Example usage
model = SimpleRNN(input_size=10, hidden_size=64, output_size=10)
data = torch.randn(32, 1000, 10)      # batch=32, seq_len=1000
targets = torch.randn(32, 1000, 10)

train_tbptt(model, data, targets, chunk_size=50, epochs=100)
```

### TBPTT with LSTM

```python
class LSTMModel(nn.Module):
    def __init__(self, input_size, hidden_size, output_size, num_layers=2):
        super().__init__()
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.lstm = nn.LSTM(input_size, hidden_size, num_layers, 
                           batch_first=True, dropout=0.2)
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x, hidden):
        out, hidden = self.lstm(x, hidden)
        out = self.fc(out)
        return out, hidden
    
    def init_hidden(self, batch_size):
        h0 = torch.zeros(self.num_layers, batch_size, self.hidden_size)
        c0 = torch.zeros(self.num_layers, batch_size, self.hidden_size)
        return (h0, c0)

def train_lstm_tbptt(model, data, targets, chunk_size, epochs, lr=0.001):
    criterion = nn.CrossEntropyLoss()
    optimizer = torch.optim.Adam(model.parameters(), lr=lr)
    
    batch_size = data.size(0)
    seq_len = data.size(1)
    
    for epoch in range(epochs):
        hidden = model.init_hidden(batch_size)
        
        for start in range(0, seq_len, chunk_size):
            end = min(start + chunk_size, seq_len)
            x_chunk = data[:, start:end, :]
            y_chunk = targets[:, start:end]
            
            # Detach BOTH h and c for LSTM
            hidden = (hidden[0].detach(), hidden[1].detach())
            
            output, hidden = model(x_chunk, hidden)
            
            # Reshape for cross-entropy: [batch*seq, vocab] vs [batch*seq]
            loss = criterion(
                output.reshape(-1, output.size(-1)),
                y_chunk.reshape(-1)
            )
            
            optimizer.zero_grad()
            loss.backward()
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=5.0)
            optimizer.step()
```

---

## 📊 Choosing the Truncation Length

### Trade-offs

```
┌──────────────────────────────────────────────────────────────┐
│           Truncation Length (τ) Trade-offs                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Small τ (5-20)          │  Large τ (100-500)                │
│  ─────────────────       │  ───────────────────              │
│  ✓ Fast training         │  ✓ Captures longer dependencies   │
│  ✓ Low memory            │  ✓ Better gradient signal         │
│  ✗ Short-range only      │  ✗ Slower per update              │
│  ✗ Poor long-term deps   │  ✗ Higher memory usage            │
│                          │  ✗ Vanishing gradient still       │
│                          │    affects end of window           │
│                                                              │
│  Rule of thumb: τ should cover the longest dependency        │
│  you care about in your task                                 │
└──────────────────────────────────────────────────────────────┘
```

### Practical Guidelines

| Task | Recommended τ |
|------|--------------|
| Character-level language model | 100–200 |
| Word-level language model | 35–70 |
| Sentiment classification | Full sequence (short) |
| Music generation | 128–256 |
| Speech recognition | 50–100 frames |
| Time series forecasting | Depends on seasonality |

---

## 🔬 Comparison: Full BPTT vs Truncated BPTT

| Aspect | Full BPTT | Truncated BPTT |
|--------|-----------|----------------|
| Memory | O(T) | O(τ) |
| Computation | O(T) per update | O(τ) per update |
| Long-range gradients | ✅ Full (but vanishing) | ❌ Truncated at τ |
| Hidden state | Full sequence | Carries across chunks |
| Practical for long seqs | ❌ | ✅ |
| Gradient accuracy | Exact | Approximate |
| Update frequency | Once per sequence | Every τ steps |

---

## ⚠️ Common Pitfalls

### 1. Forgetting to Detach Hidden State

```python
# WRONG — graph grows indefinitely, OOM error
for chunk in chunks:
    output, hidden = model(chunk, hidden)  # hidden still connected!
    loss.backward()

# CORRECT — detach at chunk boundaries
for chunk in chunks:
    hidden = hidden.detach()  # or tuple detach for LSTM
    output, hidden = model(chunk, hidden)
    loss.backward()
```

### 2. Resetting Hidden State Between Chunks

```python
# WRONG — loses all context between chunks
for chunk in chunks:
    hidden = torch.zeros(...)  # Context destroyed!
    output, hidden = model(chunk, hidden)

# CORRECT — carry hidden state forward
hidden = torch.zeros(...)  # Initialize once
for chunk in chunks:
    hidden = hidden.detach()  # Detach, don't reset
    output, hidden = model(chunk, hidden)
```

### 3. Inconsistent Chunk Sizes

The last chunk may be shorter. Handle it:

```python
for start in range(0, seq_len, chunk_size):
    end = min(start + chunk_size, seq_len)  # Handle last chunk
    if end - start == 0:
        break
    x_chunk = data[:, start:end, :]
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Full BPTT | Backprop through entire sequence — exact but expensive |
| Truncated BPTT | Backprop limited to τ steps — approximate but practical |
| Hidden state | Carried forward across chunks (preserves context) |
| Gradients | Truncated at chunk boundaries (limits computation) |
| `hidden.detach()` | Critical operation to break computation graph |
| Chunk size τ | Trade-off between dependency length and efficiency |
| k₁ vs k₂ | Forward steps vs backward steps (often k₁ = k₂) |

---

## ❓ Revision Questions

1. **Why can't we use full BPTT for very long sequences?** What are the memory and computational costs?

2. **Explain the critical difference** between carrying the hidden state forward and letting gradients flow back across chunk boundaries in truncated BPTT.

3. **What happens if you forget to call `hidden.detach()`** in a PyTorch truncated BPTT loop? What error would you eventually see?

4. **A language model needs to capture paragraph-level context (~200 words).** What truncation length would you choose, and why?

5. **Compare Variant 1 (k₁ = k₂) with Variant 2 (k₁ > k₂).** When would you prefer one over the other?

6. **How does truncated BPTT interact with the vanishing gradient problem?** Does truncation make vanishing gradients better or worse?

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬆️ Unit | [09 Training RNNs](../README.md) |
| ➡️ Next | [Gradient Clipping](02-gradient-clipping.md) |
| 🏠 Home | [RNN Home](../README.md) |

---

*Last updated: 2024*
