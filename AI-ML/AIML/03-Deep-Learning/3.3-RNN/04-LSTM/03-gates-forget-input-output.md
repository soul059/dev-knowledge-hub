# Gates: Forget, Input, Output

> **Unit 4, Chapter 3** — Detailed formulas and intuition for each LSTM gate: forget gate f_t, input gate i_t, and output gate o_t.

---

## 📋 Overview

The three gates are the control mechanisms of the LSTM. Each gate is a sigmoid layer that outputs values between 0 and 1, acting as a **soft switch** that controls how much information flows through. Understanding each gate's role is essential for understanding how LSTM maintains and updates its memory.

---

## 1️⃣ Forget Gate (f_t)

```
"What should I ERASE from my memory?"

┌─────────────────────────────────────────────────────────────┐
│                    FORGET GATE                               │
│                                                             │
│  f_t = σ(W_f · [h_{t-1}, x_t] + b_f)                      │
│                                                             │
│  Dimensions:                                                │
│    [h_{t-1}, x_t] ∈ ℝ^{n+d}   (concatenated input)        │
│    W_f ∈ ℝ^{n×(n+d)}          (forget gate weights)       │
│    b_f ∈ ℝ^n                   (forget gate bias)          │
│    f_t ∈ ℝ^n                   (one value per cell unit)   │
│                                                             │
│  Used in cell update:                                       │
│    C_t = f_t ⊙ C_{t-1} + ...                              │
│           ──────────────                                    │
│    Each element of f_t ∈ (0,1) multiplies corresponding    │
│    element of C_{t-1}:                                     │
│                                                             │
│    f_t[i] ≈ 1 → KEEP cell[i]    (remember)                │
│    f_t[i] ≈ 0 → ERASE cell[i]   (forget)                  │
└─────────────────────────────────────────────────────────────┘
```

### Intuition

```
Example: Language modeling
   "The cat, which was very fluffy and loved to play, sat on the ___"

   When processing "sat":
   • Forget gate should KEEP: subject ("cat"), context
   • Forget gate should ERASE: irrelevant details

   Cell state dimension for "subject" might have f_t ≈ 0.99
   Cell state dimension for "adjective detail" might have f_t ≈ 0.1
```

---

## 2️⃣ Input Gate (i_t) + Candidate (C̃_t)

```
"What NEW information should I WRITE to memory?"

┌─────────────────────────────────────────────────────────────┐
│                 INPUT GATE + CANDIDATE                       │
│                                                             │
│  Input gate:                                                │
│    i_t = σ(W_i · [h_{t-1}, x_t] + b_i)                    │
│                                                             │
│  Candidate values (what COULD be written):                  │
│    C̃_t = tanh(W_C · [h_{t-1}, x_t] + b_C)                │
│                                                             │
│  Used in cell update:                                       │
│    C_t = ... + i_t ⊙ C̃_t                                  │
│              ──────────────                                 │
│                                                             │
│  Two-part process:                                          │
│    1. C̃_t creates candidate new values (tanh → [-1, 1])   │
│    2. i_t decides WHICH candidates to actually write        │
│                                                             │
│    i_t[i] ≈ 1 → WRITE C̃_t[i] to cell state               │
│    i_t[i] ≈ 0 → DON'T WRITE (ignore this candidate)       │
└─────────────────────────────────────────────────────────────┘
```

### Why Two Separate Components?

```
C̃_t (tanh, range [-1,1]):  WHAT to write   → content
i_t  (sigmoid, range [0,1]): WHETHER to write → gating

Separation allows:
• C̃_t to propose both positive and negative updates
• i_t to independently control which updates happen
• Different information can be written at different strengths
```

---

## 3️⃣ Output Gate (o_t)

```
"What from my memory should I EXPOSE/OUTPUT?"

┌─────────────────────────────────────────────────────────────┐
│                    OUTPUT GATE                               │
│                                                             │
│  o_t = σ(W_o · [h_{t-1}, x_t] + b_o)                      │
│                                                             │
│  Used to compute hidden state:                              │
│    h_t = o_t ⊙ tanh(C_t)                                  │
│                                                             │
│  The cell state C_t contains ALL stored information.       │
│  The output gate selects WHICH parts to expose.            │
│                                                             │
│    o_t[i] ≈ 1 → EXPOSE cell info → h_t[i] ≈ tanh(C_t[i]) │
│    o_t[i] ≈ 0 → HIDE cell info   → h_t[i] ≈ 0            │
│                                                             │
│  This means C_t can store information that is NOT          │
│  visible in h_t — hidden memory for later use!             │
└─────────────────────────────────────────────────────────────┘
```

### Intuition

```
Cell state might store:
   C_t = [subject_info, tense_info, sentiment, topic, ...]

At a particular time step, only some are relevant for output:
   Processing a verb → expose tense_info (o_t[tense] ≈ 1)
   Processing a noun → expose subject_info (o_t[subject] ≈ 1)
   Both keep sentiment stored but hidden (o_t[sentiment] ≈ 0)
```

---

## 📐 Gate Interactions

```
Step 1: FORGET — Erase old info
    C_{t-1} ──[× f_t]──▶ (partial C_{t-1})

Step 2: INPUT — Add new info
    C̃_t ──[× i_t]──▶ (filtered new info)

Step 3: UPDATE — Combine
    C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t

Step 4: OUTPUT — Filter for h_t
    h_t = o_t ⊙ tanh(C_t)

Full pipeline:
    [h_{t-1}, x_t] ──┬──▶ f_t ──┐
                      ├──▶ i_t ──┤
                      ├──▶ C̃_t ──┤──▶ C_t ──▶ h_t
                      └──▶ o_t ──┘
```

---

## 💻 Implementation from Scratch

```python
import torch
import torch.nn as nn

class LSTMCellManual(nn.Module):
    """LSTM cell with explicit gate computation"""
    def __init__(self, input_size, hidden_size):
        super().__init__()
        self.hidden_size = hidden_size
        
        # Four weight matrices (one per gate + candidate)
        combined_size = input_size + hidden_size
        self.W_f = nn.Linear(combined_size, hidden_size)  # Forget gate
        self.W_i = nn.Linear(combined_size, hidden_size)  # Input gate
        self.W_c = nn.Linear(combined_size, hidden_size)  # Candidate
        self.W_o = nn.Linear(combined_size, hidden_size)  # Output gate
        
        # Initialize forget gate bias to 1 (remember by default)
        nn.init.ones_(self.W_f.bias)
    
    def forward(self, x_t, h_prev, c_prev):
        """
        x_t:    (batch, input_size)
        h_prev: (batch, hidden_size)
        c_prev: (batch, hidden_size)
        """
        # Concatenate input and previous hidden state
        combined = torch.cat([h_prev, x_t], dim=1)
        
        # Gate computations
        f_t = torch.sigmoid(self.W_f(combined))    # Forget gate
        i_t = torch.sigmoid(self.W_i(combined))    # Input gate
        c_tilde = torch.tanh(self.W_c(combined))   # Candidate
        o_t = torch.sigmoid(self.W_o(combined))    # Output gate
        
        # Cell state update
        c_t = f_t * c_prev + i_t * c_tilde
        
        # Hidden state
        h_t = o_t * torch.tanh(c_t)
        
        return h_t, c_t, {
            'forget_gate': f_t,
            'input_gate': i_t,
            'candidate': c_tilde,
            'output_gate': o_t
        }

# Demo: Watch gate values
cell = LSTMCellManual(input_size=5, hidden_size=3)
x = torch.randn(1, 5)
h = torch.zeros(1, 3)
c = torch.zeros(1, 3)

h_new, c_new, gates = cell(x, h, c)

print("=== Gate Values ===")
print(f"Forget gate: {gates['forget_gate'].detach().numpy().round(4)}")
print(f"Input gate:  {gates['input_gate'].detach().numpy().round(4)}")
print(f"Output gate: {gates['output_gate'].detach().numpy().round(4)}")
print(f"Candidate:   {gates['candidate'].detach().numpy().round(4)}")
print(f"\nCell state:  {c_new.detach().numpy().round(4)}")
print(f"Hidden state:{h_new.detach().numpy().round(4)}")
```

---

## 📋 Summary Table

| Gate | Formula | Range | Purpose |
|------|---------|-------|---------|
| Forget (f_t) | σ(W_f·[h_{t-1},x_t]+b_f) | (0,1) | What to erase from C_{t-1} |
| Input (i_t) | σ(W_i·[h_{t-1},x_t]+b_i) | (0,1) | What new info to write |
| Candidate (C̃_t) | tanh(W_C·[h_{t-1},x_t]+b_C) | (-1,1) | New values to potentially add |
| Output (o_t) | σ(W_o·[h_{t-1},x_t]+b_o) | (0,1) | What to expose from C_t |
| Cell Update | f_t⊙C_{t-1} + i_t⊙C̃_t | (-∞,∞) | Updated long-term memory |
| Hidden State | o_t⊙tanh(C_t) | (-1,1) | Current output/short-term memory |

---

## ❓ Revision Questions

1. **Write all three gate equations. What activation function does each use and why?**

2. **If f_t = [0.9, 0.1, 0.8] and C_{t-1} = [5.0, -3.0, 2.0], what is f_t ⊙ C_{t-1}? Interpret each element.**

3. **Why does the input gate need TWO components (i_t and C̃_t) instead of just one?**

4. **How does the output gate allow the LSTM to store "hidden" information that doesn't affect the current output?**

5. **Why is the forget gate bias typically initialized to 1 rather than 0?**

6. **All three gates receive the same input [h_{t-1}, x_t]. How can they produce different outputs?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [LSTM Cell Architecture](02-lstm-cell-architecture.md) |
| ➡️ Next | [Cell State](04-cell-state.md) |
| ⬆️ Unit Overview | [README](../README.md) |
