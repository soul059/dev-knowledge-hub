# Encoder-Decoder Architecture

> **Unit 6, Chapter 3** — Encoder compresses input → context vector → decoder generates output: the seq2seq architecture and the information bottleneck problem.

---

## 📋 Overview

The **encoder-decoder** (or sequence-to-sequence) architecture handles tasks where input and output sequences have different lengths. The encoder reads the entire input and compresses it into a fixed-size **context vector**, which the decoder then uses to generate the output sequence.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ENCODER-DECODER ARCHITECTURE                     │
│                                                                     │
│   ENCODER                                    DECODER                │
│   ───────                                    ───────                │
│                                                                     │
│   x₁    x₂    x₃   <eos>                   <sos>  y₁    y₂        │
│    │     │     │     │                       │     │     │         │
│    ▼     ▼     ▼     ▼                       ▼     ▼     ▼         │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐    context       ┌───┐ ┌───┐ ┌───┐      │
│  │ h₁│▶│ h₂│▶│ h₃│▶│ h₄│──────────────────▶│ s₁│▶│ s₂│▶│ s₃│      │
│  └───┘ └───┘ └───┘ └───┘    vector (c)     └─┬─┘ └─┬─┘ └─┬─┘      │
│                              h₄ = c           │     │     │         │
│                                               ▼     ▼     ▼         │
│                                              y₁    y₂   <eos>      │
│                                                                     │
│   Input sequence ──▶ Fixed vector ──▶ Output sequence               │
│   (variable length)  (fixed size)    (variable length)             │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🧮 Equations

```
ENCODER (reads input, produces context):
   h_t^enc = f_enc(x_t, h_{t-1}^enc)    for t = 1, ..., T_x
   
   Context vector:
   c = h_{T_x}^enc    (final encoder hidden state)

DECODER (generates output from context):
   s_0 = c    (initialize decoder with context)
   s_t^dec = f_dec(y_{t-1}, s_{t-1}^dec)    for t = 1, ..., T_y
   
   Output:
   P(y_t | y_{<t}, x) = softmax(W_y · s_t^dec + b_y)

Training (teacher forcing):
   Feed GROUND TRUTH y_{t-1} as decoder input

Inference (autoregressive):
   Feed MODEL'S OWN PREDICTION ŷ_{t-1} as decoder input
```

---

## 📐 The Information Bottleneck

```
                ALL information about input must pass through
                this single fixed-size vector!

   x₁ x₂ x₃ ... x₁₀₀                    y₁ y₂ y₃ ...
    └────┬────────┘                        └────┬────┘
         │                                      │
         ▼                                      ▼
    ┌─────────┐     ┌──────────┐     ┌─────────────────┐
    │ ENCODER │────▶│ c ∈ ℝⁿ  │────▶│    DECODER      │
    │         │     │(n=256?)  │     │                 │
    └─────────┘     └──────────┘     └─────────────────┘
                         ↑
                    BOTTLENECK!
    
    100 words of meaning compressed into 256 numbers?
    That's lossy! Information WILL be lost.

    Problem gets WORSE with longer inputs.
    This is why ATTENTION was invented (next chapter).
```

---

## 💻 PyTorch Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class Encoder(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_layers=1, dropout=0.2):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.LSTM(embed_dim, hidden_dim, num_layers=num_layers,
                           batch_first=True, dropout=dropout if num_layers > 1 else 0)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, src):
        """src: (batch, src_len)"""
        embedded = self.dropout(self.embedding(src))
        outputs, (hidden, cell) = self.rnn(embedded)
        return hidden, cell  # Context = final hidden + cell state

class Decoder(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_layers=1, dropout=0.2):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.LSTM(embed_dim, hidden_dim, num_layers=num_layers,
                           batch_first=True, dropout=dropout if num_layers > 1 else 0)
        self.fc_out = nn.Linear(hidden_dim, vocab_size)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, input_token, hidden, cell):
        """One decoding step"""
        embedded = self.dropout(self.embedding(input_token.unsqueeze(1)))
        output, (hidden, cell) = self.rnn(embedded, (hidden, cell))
        prediction = self.fc_out(output.squeeze(1))
        return prediction, hidden, cell

class Seq2Seq(nn.Module):
    def __init__(self, encoder, decoder, device='cpu'):
        super().__init__()
        self.encoder = encoder
        self.decoder = decoder
        self.device = device
    
    def forward(self, src, trg, teacher_forcing_ratio=0.5):
        """
        src: (batch, src_len)
        trg: (batch, trg_len) — target sequence
        """
        batch_size = src.shape[0]
        trg_len = trg.shape[1]
        trg_vocab_size = self.decoder.fc_out.out_features
        
        outputs = torch.zeros(batch_size, trg_len, trg_vocab_size)
        
        # Encode
        hidden, cell = self.encoder(src)
        
        # First decoder input: <sos> token
        input_token = trg[:, 0]
        
        # Decode step by step
        for t in range(1, trg_len):
            prediction, hidden, cell = self.decoder(input_token, hidden, cell)
            outputs[:, t] = prediction
            
            # Teacher forcing: use ground truth or prediction
            use_teacher = torch.rand(1).item() < teacher_forcing_ratio
            input_token = trg[:, t] if use_teacher else prediction.argmax(dim=1)
        
        return outputs

# Build model
enc = Encoder(vocab_size=8000, embed_dim=256, hidden_dim=512)
dec = Decoder(vocab_size=10000, embed_dim=256, hidden_dim=512)
model = Seq2Seq(enc, dec)

src = torch.randint(0, 8000, (32, 20))    # English
trg = torch.randint(0, 10000, (32, 25))   # French
output = model(src, trg)
print(f"Source: {src.shape}, Target: {trg.shape}")
print(f"Output: {output.shape}")  # (32, 25, 10000)
```

---

## 📋 Summary Table

| Component | Role | Key Detail |
|-----------|------|------------|
| Encoder | Reads input sequence | Produces context vector c |
| Context Vector | Fixed-size summary | c = final hidden state |
| Decoder | Generates output | Initialized with c, autoregressive |
| Bottleneck | ALL info in one vector | Limits performance on long inputs |
| Teacher Forcing | Training trick | Use ground truth as decoder input |
| Special Tokens | \<sos\>, \<eos\> | Start and end of sequence |

---

## ❓ Revision Questions

1. **Draw the encoder-decoder architecture for translating a 5-word sentence into a 3-word sentence.**

2. **What is the information bottleneck? Why does it get worse with longer input sequences?**

3. **Explain the difference between teacher forcing and autoregressive decoding.**

4. **Why does the decoder need a \<sos\> token? What happens without it?**

5. **How is the context vector used to initialize the decoder? What are alternative approaches?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Deep RNN](02-deep-rnn.md) |
| ➡️ Next | [Attention Mechanism Basics](04-attention-mechanism-basics.md) |
| ⬆️ Unit Overview | [README](../README.md) |
