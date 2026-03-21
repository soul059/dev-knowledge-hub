# Dropout in RNNs

> **Unit 9, Chapter 4** — Regularizing RNNs with dropout: why naive dropout fails, variational dropout, zoneout, and PyTorch best practices.

---

## 📋 Overview

Dropout is one of the most effective regularizers in deep learning — randomly zeroing activations during training to prevent co-adaptation of neurons. However, **naive application of dropout to RNNs destroys the hidden state signal** over time, making the network unable to learn long-term dependencies. This chapter covers why standard dropout fails in RNNs and presents specialized alternatives: **variational (locked) dropout**, **recurrent dropout**, and **zoneout**.

---

## ❌ Why Naive Dropout Fails in RNNs

### Standard Dropout Recap

In feedforward networks, dropout randomly zeros activations at each layer:

```
Feedforward with dropout (works fine):

  x → [Layer 1] → dropout → [Layer 2] → dropout → [Layer 3] → y
                    ↑                      ↑
              New random mask         New random mask
              each forward pass       each forward pass
```

### The Problem with RNNs

If we apply a **different** dropout mask at each time step to the recurrent connection:

```
Time step 1:  h₀ ──→ [RNN] ──→ h₁  (mask₁ drops neurons {2, 5, 7})
Time step 2:  h₁ ──→ [RNN] ──→ h₂  (mask₂ drops neurons {1, 3, 8})
Time step 3:  h₂ ──→ [RNN] ──→ h₃  (mask₃ drops neurons {4, 6, 2})
...
Time step T:  h_{T-1} → [RNN] → hₜ (mask_T drops neurons {3, 7, 1})

Over T steps: EVERY neuron has been dropped multiple times!
```

### Mathematical Analysis

Probability that neuron `i` **survives** all T time steps:

```
P(survive all T steps) = (1 - p)^T

With dropout rate p = 0.5 and T = 20:
  P(survive) = 0.5^20 = 0.00000095 ≈ 0%

The hidden state is essentially DESTROYED for long sequences!
```

```
Signal degradation with naive recurrent dropout:

  Signal
  strength
     │
  1.0│╲
     │  ╲
  0.8│    ╲
     │      ╲
  0.5│        ╲___
     │             ╲___
  0.1│                  ╲_____
  0.0│                        ╲___________
     └──────────────────────────────────── t
     0   2   4   6   8  10  12  14  16

Information in hidden state decays exponentially!
```

---

## ✅ Where Standard Dropout DOES Work in RNNs

Dropout can be safely applied to **non-recurrent** connections:

```
┌──────────────────────────────────────────────────────────────┐
│          Safe Dropout Locations in RNNs                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ✅ Input connections (x_t → hidden)                         │
│     Different mask per time step is OK                       │
│                                                              │
│  ✅ Output connections (hidden → y_t)                        │
│     Different mask per time step is OK                       │
│                                                              │
│  ❌ Recurrent connections (h_{t-1} → h_t)                   │
│     Different mask per time step DESTROYS signal             │
│     → Need variational/locked dropout                        │
│                                                              │
│  ✅ Between stacked RNN layers                               │
│     These are non-recurrent connections                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### PyTorch's Built-in LSTM Dropout

PyTorch's `dropout` parameter applies dropout **between layers** (not within recurrent connections):

```python
lstm = nn.LSTM(
    input_size=100,
    hidden_size=256,
    num_layers=3,
    dropout=0.3,        # Applied between layer 1→2 and 2→3
    batch_first=True     # NOT applied to recurrent connections
)
```

```
Layer 3: [LSTM Cell] ─────────────────────── [LSTM Cell] ─── → output
                ↑                                    ↑
            dropout (0.3)                       dropout (0.3)
                ↑                                    ↑
Layer 2: [LSTM Cell] ─────────────────────── [LSTM Cell]
                ↑                                    ↑
            dropout (0.3)                       dropout (0.3)
                ↑                                    ↑
Layer 1: [LSTM Cell] ─────────────────────── [LSTM Cell]
                ↑                                    ↑
              x_t=1                                x_t=2
```

---

## 🔒 Variational (Locked) Dropout

### Core Idea (Gal & Ghahramani, 2016)

Use the **same dropout mask for all time steps** within a sequence. This ensures consistent information flow through time.

```
Standard dropout (WRONG for recurrence):
  t=1: mask = [1, 0, 1, 0, 1]    ← different mask
  t=2: mask = [0, 1, 0, 1, 1]    ← different mask
  t=3: mask = [1, 1, 0, 0, 1]    ← different mask

Variational dropout (CORRECT):
  t=1: mask = [1, 0, 1, 0, 1]    ← SAME mask
  t=2: mask = [1, 0, 1, 0, 1]    ← SAME mask
  t=3: mask = [1, 0, 1, 0, 1]    ← SAME mask

New mask sampled for each SEQUENCE (or mini-batch), not each time step.
```

### Implementation

```python
class VariationalDropout(nn.Module):
    """
    Applies the same dropout mask across all time steps.
    """
    def __init__(self, dropout_rate=0.5):
        super().__init__()
        self.dropout_rate = dropout_rate
    
    def forward(self, x):
        """
        Args:
            x: tensor of shape [batch, seq_len, features]
        Returns:
            Dropped-out tensor with same mask for all time steps
        """
        if not self.training or self.dropout_rate == 0:
            return x
        
        # Create mask for ONE time step: [batch, 1, features]
        mask = x.new_ones(x.size(0), 1, x.size(2))
        mask = torch.nn.functional.dropout(mask, p=self.dropout_rate, training=True)
        
        # Broadcast across all time steps
        return x * mask  # [batch, seq_len, features] * [batch, 1, features]


class LSTMWithVariationalDropout(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers=1, 
                 input_dropout=0.3, hidden_dropout=0.3, output_dropout=0.3):
        super().__init__()
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        # Build LSTM cells manually for per-connection dropout control
        self.cells = nn.ModuleList([
            nn.LSTMCell(
                input_size if i == 0 else hidden_size,
                hidden_size
            ) for i in range(num_layers)
        ])
        
        self.input_drop = VariationalDropout(input_dropout)
        self.hidden_drop = VariationalDropout(hidden_dropout)
        self.output_drop = VariationalDropout(output_dropout)
    
    def forward(self, x, hidden=None):
        batch_size, seq_len, _ = x.size()
        
        if hidden is None:
            hidden = self._init_hidden(batch_size, x.device)
        
        h_states, c_states = hidden
        
        # Apply variational dropout to input
        x = self.input_drop(x)
        
        outputs = []
        for t in range(seq_len):
            inp = x[:, t, :]
            new_h, new_c = [], []
            
            for layer, cell in enumerate(self.cells):
                h, c = cell(inp, (h_states[layer], c_states[layer]))
                new_h.append(h)
                new_c.append(c)
                
                # Apply locked dropout between layers
                if layer < self.num_layers - 1:
                    inp = h  # Will be dropped by next layer's input
                else:
                    inp = h
            
            h_states = new_h
            c_states = new_c
            outputs.append(h)
        
        output = torch.stack(outputs, dim=1)  # [batch, seq_len, hidden]
        output = self.output_drop(output)
        
        return output, (h_states, c_states)
    
    def _init_hidden(self, batch_size, device):
        h = [torch.zeros(batch_size, self.hidden_size, device=device)
             for _ in range(self.num_layers)]
        c = [torch.zeros(batch_size, self.hidden_size, device=device)
             for _ in range(self.num_layers)]
        return h, c
```

---

## 🔄 Recurrent Dropout

### Concept (Semeniuta et al., 2016)

Apply dropout specifically to the **candidate cell state** (or candidate hidden state) in LSTM/GRU, using the same mask across time steps.

```
Standard LSTM:
  C̃_t = tanh(W_c · [h_{t-1}, x_t] + b_c)
  C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t

With recurrent dropout:
  C̃_t = tanh(W_c · [h_{t-1}, x_t] + b_c)
  C_t = f_t ⊙ C_{t-1} + i_t ⊙ dropout(C̃_t)    ← dropout on candidate
                                    ↑
                          Same mask for all t
```

This approach:
- Doesn't corrupt the cell state highway (f_t path is untouched)
- Regularizes the new information being written to memory
- Preserves long-term dependencies

---

## ⏸️ Zoneout

### Concept (Krueger et al., 2017)

Instead of zeroing activations, **randomly maintain the previous hidden state** at some neurons:

```
Standard dropout:  h_t[i] = 0          with probability p
Zoneout:          h_t[i] = h_{t-1}[i]  with probability p

┌────────────────────────────────────────────────────────┐
│  Zoneout: "Forget to update" some hidden units         │
│                                                        │
│  For each neuron i at time t:                          │
│    With prob (1-p): h_t[i] = new_h_t[i]   (update)    │
│    With prob p:     h_t[i] = h_{t-1}[i]   (keep old)  │
│                                                        │
│  Effect: Creates random skip connections through time  │
│          Similar to stochastic depth for RNNs          │
└────────────────────────────────────────────────────────┘
```

### Implementation

```python
class ZoneoutLSTMCell(nn.Module):
    def __init__(self, input_size, hidden_size, zoneout_h=0.05, zoneout_c=0.5):
        super().__init__()
        self.cell = nn.LSTMCell(input_size, hidden_size)
        self.zoneout_h = zoneout_h
        self.zoneout_c = zoneout_c
    
    def forward(self, x, state):
        h_prev, c_prev = state
        h_new, c_new = self.cell(x, state)
        
        if self.training:
            # Random mask: 1 = keep old, 0 = use new
            h_mask = torch.bernoulli(
                torch.full_like(h_new, self.zoneout_h)
            )
            c_mask = torch.bernoulli(
                torch.full_like(c_new, self.zoneout_c)
            )
            h = h_mask * h_prev + (1 - h_mask) * h_new
            c = c_mask * c_prev + (1 - c_mask) * c_new
        else:
            # At test time: use expected value
            h = self.zoneout_h * h_prev + (1 - self.zoneout_h) * h_new
            c = self.zoneout_c * c_prev + (1 - self.zoneout_c) * c_new
        
        return h, c
```

---

## 📊 Comparison of Dropout Methods for RNNs

```
┌─────────────────┬──────────────┬──────────────┬──────────────┐
│    Method       │   Applied To │  Same Mask   │   Effect     │
│                 │              │  Across Time │              │
├─────────────────┼──────────────┼──────────────┼──────────────┤
│ Naive dropout   │ All connect. │     No       │ Destroys     │
│ (DON'T USE)     │              │              │ hidden state │
├─────────────────┼──────────────┼──────────────┼──────────────┤
│ Between-layer   │ Layer→Layer  │     No       │ Safe,        │
│ (PyTorch LSTM)  │ connections  │ (not needed) │ standard     │
├─────────────────┼──────────────┼──────────────┼──────────────┤
│ Variational     │ Input, hidden│    Yes       │ Best         │
│ (locked)        │ output       │              │ general use  │
├─────────────────┼──────────────┼──────────────┼──────────────┤
│ Recurrent       │ Candidate    │    Yes       │ LSTM/GRU     │
│ dropout         │ state only   │              │ specific     │
├─────────────────┼──────────────┼──────────────┼──────────────┤
│ Zoneout         │ Hidden/cell  │     No       │ Skip         │
│                 │ state update │  (random)    │ connections  │
└─────────────────┴──────────────┴──────────────┴──────────────┘
```

---

## 🧮 Worked Example: AWD-LSTM Dropout Strategy

The AWD-LSTM (Merity et al., 2018) uses **5 different dropout types**:

```
┌──────────────────────────────────────────────────────────────┐
│  AWD-LSTM Dropout Strategy (State-of-the-art LM recipe)     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Embedding dropout (p=0.1)                                │
│     → Randomly zero entire word embeddings                   │
│                                                              │
│  2. Input dropout (p=0.3)                                    │
│     → Variational dropout on input to first LSTM layer       │
│                                                              │
│  3. Hidden dropout (p=0.3)                                   │
│     → Variational dropout between LSTM layers                │
│                                                              │
│  4. Output dropout (p=0.4)                                   │
│     → Variational dropout on final LSTM output               │
│                                                              │
│  5. Weight dropout (p=0.5)                                   │
│     → DropConnect on recurrent weight matrices W_hh          │
│     → Prevents co-adaptation of recurrent weights            │
│                                                              │
│  Total effect: Strong regularization, SOTA perplexity        │
└──────────────────────────────────────────────────────────────┘
```

```python
class WeightDrop(nn.Module):
    """Drop entire columns of the recurrent weight matrix."""
    
    def __init__(self, module, weights, dropout=0.5):
        super().__init__()
        self.module = module
        self.weights = weights
        self.dropout = dropout
        
        for name_w in self.weights:
            w = getattr(self.module, name_w)
            # Save original weight
            self.module.register_parameter(name_w + '_raw', nn.Parameter(w.data))
            delattr(self.module, name_w)
    
    def forward(self, *args):
        for name_w in self.weights:
            raw_w = getattr(self.module, name_w + '_raw')
            if self.training:
                mask = torch.bernoulli(
                    torch.full_like(raw_w, 1 - self.dropout)
                )
                w = raw_w * mask / (1 - self.dropout)
            else:
                w = raw_w
            setattr(self.module, name_w, w)
        
        return self.module(*args)
```

---

## 💡 Practical Recommendations

### Dropout Rate Guidelines

| Model Component | Dropout Rate | Method |
|----------------|-------------|--------|
| Embedding | 0.0–0.1 | Standard |
| Input to RNN | 0.2–0.4 | Variational |
| Between layers | 0.2–0.5 | Standard or Variational |
| Recurrent (h→h) | 0.1–0.3 | Variational or Weight Drop |
| Output | 0.3–0.5 | Variational |

### Quick Setup with PyTorch

```python
# Simple approach: just use between-layer dropout
model = nn.LSTM(input_size, hidden_size, num_layers=3, 
                dropout=0.3, batch_first=True)

# Better approach: add input/output dropout manually
class BetterLSTM(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, dropout=0.3):
        super().__init__()
        self.input_drop = nn.Dropout(dropout)
        self.lstm = nn.LSTM(input_size, hidden_size, num_layers,
                           dropout=dropout, batch_first=True)
        self.output_drop = nn.Dropout(dropout)
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x, hidden=None):
        x = self.input_drop(x)
        out, hidden = self.lstm(x, hidden)
        out = self.output_drop(out)
        out = self.fc(out)
        return out, hidden
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Naive recurrent dropout | Destroys hidden state over time — don't use |
| Between-layer dropout | Safe and standard — PyTorch's built-in LSTM dropout |
| Variational dropout | Same mask across time — best for recurrent connections |
| Recurrent dropout | Drops candidate state — preserves cell highway |
| Zoneout | Keeps old state randomly — creates temporal skip connections |
| Weight dropout | Drops weight matrix columns — AWD-LSTM technique |
| Typical rates | Input: 0.2–0.4, Hidden: 0.1–0.3, Output: 0.3–0.5 |

---

## ❓ Revision Questions

1. **Explain mathematically why naive dropout on recurrent connections destroys the hidden state** for long sequences. What is the survival probability after T steps?

2. **What makes variational (locked) dropout different from standard dropout?** Why does using the same mask solve the signal destruction problem?

3. **Compare zoneout with variational dropout.** How does each affect gradient flow through time?

4. **PyTorch's `nn.LSTM(dropout=0.3)` — where exactly is dropout applied?** Is this sufficient for regularization? What's missing?

5. **The AWD-LSTM uses 5 types of dropout simultaneously.** Why not just use one type with a higher rate?

6. **Design a dropout strategy for a 3-layer BiLSTM for sentiment analysis.** Specify dropout type and rate for each connection.

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Learning Rate Scheduling](03-learning-rate-scheduling.md) |
| ⬆️ Unit | [09 Training RNNs](../README.md) |
| ➡️ Next | [Layer Normalization](05-layer-normalization.md) |
| 🏠 Home | [RNN Home](../README.md) |

---

*Last updated: 2024*
