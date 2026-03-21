# LSTM Cell Architecture

> **Unit 4, Chapter 2** — The cell state highway, three gates, and detailed ASCII diagram of the LSTM cell with all connections.

---

## 📋 Overview

The LSTM cell is more complex than a vanilla RNN cell but elegantly designed. It has two state vectors (cell state C_t and hidden state h_t) and three gates that regulate information flow. The cell state acts as a **conveyor belt** running through the entire sequence, with gates adding or removing information.

---

## 🏗️ Complete LSTM Cell Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         LSTM CELL at time t                            │
│                                                                         │
│    C_{t-1} ──────────────[×]──────────────[+]──────────────── C_t      │
│              (cell state)  ↑    forget     ↑ ↑   add new               │
│                           │              │ │   info                    │
│                           │              │ │                           │
│                          f_t            i_t C̃_t                       │
│                        (forget         (input  (candidate              │
│                         gate)           gate)   values)               │
│                           │              │   │                         │
│                        ┌──┴──┐       ┌──┴──┐ ┌──┴──┐                  │
│                        │  σ  │       │  σ  │ │tanh │                  │
│                        └──┬──┘       └──┬──┘ └──┬──┘                  │
│                           │              │      │                      │
│                     ┌─────┴──────────────┴──────┴─────┐               │
│                     │      [h_{t-1}, x_t]             │               │
│                     └─────────────┬───────────────────┘               │
│                                   │                                    │
│    h_{t-1} ──────────────────────┤                                    │
│                                   │                                    │
│    x_t ──────────────────────────┘                                    │
│                                                                         │
│                                   │                                    │
│                                   │                                    │
│                              ┌────┴────┐                               │
│    C_t ─────────────────────▶│  tanh   │                               │
│                              └────┬────┘                               │
│                                   │                                    │
│                                  [×]◀──── o_t (output gate)           │
│                                   │        │                           │
│                                   │     ┌──┴──┐                       │
│                                   │     │  σ  │                       │
│                                   │     └──┬──┘                       │
│                                   │        │                           │
│                                   ▼     [h_{t-1}, x_t]               │
│                                  h_t                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Simplified Flow Diagram

```
                  C_{t-1}
                    │
        ┌───────────┼───────────────────────┐
        │           │                       │
        │     ┌─────▼─────┐          ┌─────▼─────┐
        │     │  × (f_t)  │          │  + (add)  │
        │     └─────┬─────┘          └─────┬─────┘
        │           │                      ↑│
        │           │               ┌──────┘│
        │           │               │       │
        │           └───────────────┘       │
        │                                   │── C_t
        │         ┌──────────┐              │
        │         │ × (i_t ⊙ │              │
        │         │   C̃_t)   │──────────────┘
        │         └────┬─────┘
        │              │
        │    ┌────┐  ┌─┴──┐
        │    │ i_t│  │ C̃_t│
        │    │  σ │  │tanh│
        │    └─┬──┘  └─┬──┘
        │      │       │
        │      └───┬───┘
h_{t-1}─┤         │                    ┌────┐
        │    [h_{t-1},x_t]            │ o_t│
        │         │                    │  σ │
        │    ┌────┘                    └─┬──┘
        │    │                           │
x_t ────┤    │         ┌─────┐          │
        │    └────────▶│ f_t │          │
        │              │  σ  │          │
        │              └─────┘          │
        │                               │
        │      C_t ──▶ tanh ──▶ × ◀────┘
        │                        │
        └────────────────────────▼
                                h_t
```

---

## 🔑 The Two State Vectors

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  CELL STATE (C_t) — "Long-term memory"                     │
│  ─────────────────                                         │
│  • Runs along the top of the cell                          │
│  • Linear interactions (addition, element-wise multiply)   │
│  • Information can flow unchanged for many steps           │
│  • Modified ONLY by gates (controlled updates)             │
│  • Gradient highway — avoids vanishing gradients           │
│                                                             │
│  HIDDEN STATE (h_t) — "Short-term memory / output"         │
│  ──────────────────                                        │
│  • Filtered version of cell state                          │
│  • Passed to output layer and next time step               │
│  • Undergoes tanh + gating (more processed)               │
│  • Used for predictions at current time step               │
│                                                             │
│  Analogy:                                                   │
│  • C_t = your complete knowledge (notebook)                │
│  • h_t = what you're currently thinking about              │
│         (relevant subset of knowledge)                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚪 The Three Gates

```
┌──────────┬────────────┬────────────────────────────────────┐
│ Gate     │ Activation │ Purpose                            │
├──────────┼────────────┼────────────────────────────────────┤
│ Forget   │ σ (sigmoid)│ What to ERASE from cell state     │
│ (f_t)    │ → (0,1)    │ f_t ≈ 0: forget everything       │
│          │            │ f_t ≈ 1: remember everything      │
├──────────┼────────────┼────────────────────────────────────┤
│ Input    │ σ (sigmoid)│ What new info to WRITE             │
│ (i_t)    │ → (0,1)    │ i_t ≈ 0: don't write anything    │
│          │            │ i_t ≈ 1: write candidate fully   │
├──────────┼────────────┼────────────────────────────────────┤
│ Output   │ σ (sigmoid)│ What to READ from cell state      │
│ (o_t)    │ → (0,1)    │ o_t ≈ 0: hide cell state         │
│          │            │ o_t ≈ 1: expose cell state        │
└──────────┴────────────┴────────────────────────────────────┘

All gates use sigmoid because:
• Output range (0,1) acts as a "percentage" / dimmer switch
• 0 = completely off, 1 = completely on
• Smooth gradient for training
```

---

## 🧮 Complete Equations (Preview)

```
1. Forget gate:    f_t = σ(W_f · [h_{t-1}, x_t] + b_f)
2. Input gate:     i_t = σ(W_i · [h_{t-1}, x_t] + b_i)
3. Candidate:      C̃_t = tanh(W_C · [h_{t-1}, x_t] + b_C)
4. Cell update:    C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t
5. Output gate:    o_t = σ(W_o · [h_{t-1}, x_t] + b_o)
6. Hidden state:   h_t = o_t ⊙ tanh(C_t)

where ⊙ = element-wise (Hadamard) product
      [h_{t-1}, x_t] = concatenation of h_{t-1} and x_t
```

---

## 💻 PyTorch LSTM

```python
import torch
import torch.nn as nn

# PyTorch's built-in LSTM
lstm = nn.LSTM(input_size=10, hidden_size=20, batch_first=True)

x = torch.randn(4, 15, 10)  # batch=4, seq_len=15, features=10

# LSTM returns (output, (h_n, c_n)) — note the TWO states!
output, (h_n, c_n) = lstm(x)

print(f"Input:         {x.shape}")         # (4, 15, 10)
print(f"Output:        {output.shape}")     # (4, 15, 20) — h_t at each step
print(f"Final hidden:  {h_n.shape}")       # (1, 4, 20) — last h_t
print(f"Final cell:    {c_n.shape}")       # (1, 4, 20) — last C_t

# Key difference from nn.RNN: LSTM returns BOTH h_n AND c_n!
print(f"\nRNN returns:  (output, h_n)")
print(f"LSTM returns: (output, (h_n, c_n))")

# Parameter count comparison
rnn_params = sum(p.numel() for p in nn.RNN(10, 20).parameters())
lstm_params = sum(p.numel() for p in nn.LSTM(10, 20).parameters())
print(f"\nRNN parameters:  {rnn_params}")
print(f"LSTM parameters: {lstm_params}")
print(f"LSTM has ~4x more parameters (4 weight matrices vs 1)")
```

---

## 📋 Summary Table

| Component | Symbol | Description |
|-----------|--------|-------------|
| Cell State | C_t | Long-term memory, gradient highway |
| Hidden State | h_t | Short-term memory, used for output |
| Forget Gate | f_t | Controls what to erase from C_{t-1} |
| Input Gate | i_t | Controls what new info to write |
| Output Gate | o_t | Controls what to expose from C_t |
| Candidate | C̃_t | New information to potentially add |
| Gate Activation | sigmoid (σ) | Outputs (0,1) for soft gating |
| Candidate Activation | tanh | Outputs (-1,1) for new values |

---

## ❓ Revision Questions

1. **Draw the LSTM cell diagram from memory, showing both state vectors and all three gates.**

2. **What is the difference between the cell state C_t and the hidden state h_t? Why does LSTM need both?**

3. **Why do gates use sigmoid activation while the candidate uses tanh?**

4. **In the cell state update C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t, what happens when f_t = 1 and i_t = 0?**

5. **How many weight matrices does an LSTM cell have? Why does it have ~4x more parameters than a vanilla RNN?**

6. **Explain the notebook analogy: cell state as notebook, gates as eraser/pen/glasses.**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [LSTM Motivation](01-lstm-motivation.md) |
| ➡️ Next | [Gates: Forget, Input, Output](03-gates-forget-input-output.md) |
| ⬆️ Unit Overview | [README](../README.md) |
