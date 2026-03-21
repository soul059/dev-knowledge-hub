# Deep (Stacked) RNN

> **Unit 6, Chapter 2** — Stacking multiple RNN layers, multi-layer hidden states, residual connections, and PyTorch `num_layers` parameter.

---

## 📋 Overview

Just as feedforward networks get deeper by stacking layers, RNNs can be **stacked** by feeding the output of one RNN layer as input to the next. Deeper RNNs learn hierarchical representations: lower layers capture local patterns while higher layers capture abstract, long-range features.

---

## 🏗️ Architecture

```
                    Deep RNN (3 layers)

Layer 3:  ┌───┐   ┌───┐   ┌───┐   ┌───┐
          │h³₁│──▶│h³₂│──▶│h³₃│──▶│h³₄│   ← Abstract features
          └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘
            │       │       │       │
            ▼       ▼       ▼       ▼
           y₁      y₂      y₃      y₄       ← Outputs

Layer 2:  ┌───┐   ┌───┐   ┌───┐   ┌───┐
          │h²₁│──▶│h²₂│──▶│h²₃│──▶│h²₄│   ← Intermediate
          └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘
            │       │       │       │
            ▲       ▲       ▲       ▲

Layer 1:  ┌───┐   ┌───┐   ┌───┐   ┌───┐
          │h¹₁│──▶│h¹₂│──▶│h¹₃│──▶│h¹₄│   ← Low-level features
          └───┘   └───┘   └───┘   └───┘
            ▲       ▲       ▲       ▲
            │       │       │       │
           x₁      x₂      x₃      x₄      ← Inputs

Each layer has its OWN set of weights.
Output of layer l becomes input to layer l+1.
```

---

## 🧮 Equations

```
For L layers:

Layer 1:
   h¹_t = f(W¹_xh · x_t + W¹_hh · h¹_{t-1} + b¹)

Layer l (for l = 2, ..., L):
   hˡ_t = f(Wˡ_xh · h^{l-1}_t + Wˡ_hh · hˡ_{t-1} + bˡ)
                       ────────
                       Input to layer l = output of layer l-1

Output:
   y_t = W_y · h^L_t + b_y    (from top layer)

Parameters per layer: n_l × (n_l + n_{l-1}) + n_l
Total: Σ_l parameters for each layer
```

---

## 📐 Hierarchical Feature Learning

```
┌────────────────────────────────────────────────────────┐
│              WHAT EACH LAYER LEARNS                    │
│                                                        │
│  Layer 1: Local patterns                              │
│    • Character n-grams, word morphology               │
│    • Basic audio features (phonemes)                  │
│    • Short-term temporal patterns                     │
│                                                        │
│  Layer 2: Syntactic patterns                          │
│    • Phrase structure, clause boundaries              │
│    • Intonation patterns                              │
│    • Medium-range temporal dependencies               │
│                                                        │
│  Layer 3: Semantic/Abstract patterns                  │
│    • Sentence meaning, sentiment                      │
│    • Topic, intent                                    │
│    • Long-range contextual understanding              │
│                                                        │
│  Analogy: Like CNN layers learning edges → shapes →   │
│  objects, RNN layers learn tokens → phrases → meaning │
└────────────────────────────────────────────────────────┘
```

---

## 🔗 Residual Connections

```
Deep RNNs with many layers suffer from vanishing gradients
across LAYERS (in addition to across time steps).

Solution: Residual (skip) connections between layers.

Without residual:              With residual:
   hˡ_t = f(Wˡ · h^{l-1}_t)     hˡ_t = f(Wˡ · h^{l-1}_t) + h^{l-1}_t
                                                              ──────────
                                                              skip connection!

   Layer 3:  ┌───┐                Layer 3:  ┌───┐
             │ f │                          │ f │──┐
   ─────────▶│   │──▶            ──┬──────▶│   │  │(+)──▶
             └───┘                 │       └───┘  │
                                   └──────────────┘

Requires matching dimensions (or a projection layer).
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn

# ─── Built-in multi-layer LSTM ───
deep_lstm = nn.LSTM(
    input_size=10,
    hidden_size=20,
    num_layers=3,        # ← Stack 3 layers
    batch_first=True,
    dropout=0.3          # Dropout between layers (not within!)
)

x = torch.randn(4, 15, 10)
output, (h_n, c_n) = deep_lstm(x)

print(f"Input:  {x.shape}")       # (4, 15, 10)
print(f"Output: {output.shape}")   # (4, 15, 20) ← from TOP layer
print(f"h_n:    {h_n.shape}")     # (3, 4, 20) ← one per layer
print(f"c_n:    {c_n.shape}")     # (3, 4, 20)

# h_n[0] = layer 1 final hidden, h_n[1] = layer 2, h_n[2] = layer 3

# ─── Deep LSTM with residual connections ───
class DeepLSTMResidual(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, dropout=0.3):
        super().__init__()
        self.layers = nn.ModuleList()
        self.dropouts = nn.ModuleList()
        
        for i in range(num_layers):
            in_size = input_size if i == 0 else hidden_size
            self.layers.append(nn.LSTM(in_size, hidden_size, batch_first=True))
            self.dropouts.append(nn.Dropout(dropout))
        
        self.input_proj = nn.Linear(input_size, hidden_size) if input_size != hidden_size else None
    
    def forward(self, x):
        output = x
        for i, (lstm, dropout) in enumerate(zip(self.layers, self.dropouts)):
            residual = output
            output, _ = lstm(output)
            output = dropout(output)
            
            # Residual connection (skip first layer if dims don't match)
            if i > 0:
                output = output + residual
            elif self.input_proj is not None:
                pass  # Skip residual for first layer (dim mismatch)
        
        return output

model = DeepLSTMResidual(10, 20, num_layers=4)
out = model(x)
print(f"\nResidual Deep LSTM output: {out.shape}")  # (4, 15, 20)

# Parameter comparison
for n_layers in [1, 2, 3, 4]:
    lstm = nn.LSTM(10, 20, num_layers=n_layers)
    params = sum(p.numel() for p in lstm.parameters())
    print(f"Layers={n_layers}: {params:>6,} parameters")
```

---

## 📊 Practical Recommendations

```
┌─────────────────────────────────────────────────────────┐
│ Num Layers │ Use Case                                   │
├────────────┼────────────────────────────────────────────┤
│ 1          │ Simple tasks, small data, baselines        │
│ 2          │ Most common, good default for NLP          │
│ 3          │ Complex tasks (translation, speech)        │
│ 4+         │ Rarely beneficial, risk of overfitting    │
│            │ Use residual connections if > 3 layers     │
└────────────┴────────────────────────────────────────────┘

Google's Neural Machine Translation (2016): 8 LSTM layers
with residual connections → state of the art at the time
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Stacking | Output of layer l is input to layer l+1 |
| Hierarchical | Lower layers = local, higher layers = abstract |
| num_layers | PyTorch parameter to control depth |
| Dropout | Applied between layers, not within |
| Residual | Skip connections between layers for gradient flow |
| Typical depth | 2-3 layers for most tasks |
| Output | From top layer at each time step |

---

## ❓ Revision Questions

1. **Draw a 2-layer RNN for a 3-step sequence. Label the hidden states h¹_t and h²_t.**

2. **What is the input to layer 2 at time t? How does this differ from the input to layer 1?**

3. **Why do deeper RNNs benefit from residual connections? What problem do they solve?**

4. **What is the shape of h_n for a 3-layer bidirectional LSTM with hidden_size=100, batch_size=32?**

5. **Why is dropout applied between layers but not within the recurrent computation at each layer?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Bidirectional RNN](01-bidirectional-rnn.md) |
| ➡️ Next | [Encoder-Decoder](03-encoder-decoder.md) |
| ⬆️ Unit Overview | [README](../README.md) |
