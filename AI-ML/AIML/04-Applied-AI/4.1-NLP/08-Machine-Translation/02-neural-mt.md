# Neural Machine Translation (Seq2Seq)

## Overview

Neural Machine Translation (NMT) uses an encoder-decoder neural network to translate text end-to-end. The encoder compresses the source sentence into a context vector, and the decoder generates the target translation word by word. This replaced the complex multi-component SMT pipeline with a single differentiable model.

---

## Seq2Seq Architecture

```
Source: "I love cats"     Target: "J'aime les chats"

    I     love    cats    <EOS>     <SOS>  J'    aime   les   chats  <EOS>
    │      │       │       │          │     │      │      │     │
    ▼      ▼       ▼       ▼          ▼     ▼      ▼      ▼     ▼
  ┌────┐ ┌────┐ ┌────┐ ┌────┐     ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐
  │LSTM│→│LSTM│→│LSTM│→│LSTM│─c──▶│LSTM│→│LSTM│→│LSTM│→│LSTM│→│LSTM│
  └────┘ └────┘ └────┘ └────┘     └────┘ └────┘ └────┘ └────┘ └────┘
  ──────── Encoder ────────         ──────── Decoder ────────
                                      │      │      │     │      │
                                      ▼      ▼      ▼     ▼      ▼
                                    J'    aime    les   chats   <EOS>

  c = context vector (last hidden state of encoder)
```

---

## PyTorch Implementation

```python
import torch
import torch.nn as nn

class Encoder(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, n_layers=2):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, n_layers,
                           batch_first=True, dropout=0.3)
    
    def forward(self, src):
        embedded = self.embedding(src)
        outputs, (hidden, cell) = self.lstm(embedded)
        return hidden, cell  # context = final hidden state

class Decoder(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, n_layers=2):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, n_layers,
                           batch_first=True, dropout=0.3)
        self.fc = nn.Linear(hidden_dim, vocab_size)
    
    def forward(self, trg_token, hidden, cell):
        embedded = self.embedding(trg_token.unsqueeze(1))
        output, (hidden, cell) = self.lstm(embedded, (hidden, cell))
        prediction = self.fc(output.squeeze(1))
        return prediction, hidden, cell

class Seq2Seq(nn.Module):
    def __init__(self, encoder, decoder, device):
        super().__init__()
        self.encoder = encoder
        self.decoder = decoder
        self.device = device
    
    def forward(self, src, trg, teacher_forcing_ratio=0.5):
        batch_size = src.shape[0]
        trg_len = trg.shape[1]
        trg_vocab_size = self.decoder.fc.out_features
        
        outputs = torch.zeros(batch_size, trg_len, trg_vocab_size).to(self.device)
        hidden, cell = self.encoder(src)
        
        input_token = trg[:, 0]  # <SOS> token
        for t in range(1, trg_len):
            prediction, hidden, cell = self.decoder(input_token, hidden, cell)
            outputs[:, t] = prediction
            
            # Teacher forcing: use ground truth or model prediction
            if torch.rand(1).item() < teacher_forcing_ratio:
                input_token = trg[:, t]           # ground truth
            else:
                input_token = prediction.argmax(1) # model's prediction
        
        return outputs
```

---

## Teacher Forcing

```
With teacher forcing (training):
  Decoder input:  <SOS>  J'    aime    les    chats
  Decoder output:  J'   aime   les    chats   <EOS>
  → Always feeds correct previous word (fast convergence)

Without teacher forcing (inference):
  Decoder input:  <SOS>  J'    aime    les    chats
  → Uses its own previous prediction (what happens at test time)

Problem: Exposure bias — model never sees its own mistakes during training
Solution: Scheduled sampling — gradually reduce teacher forcing ratio
```

---

## NMT vs SMT

| Feature | SMT | NMT |
|---------|-----|-----|
| Architecture | Multi-component pipeline | End-to-end neural |
| Context | Local (phrase window) | Full sentence (encoder) |
| Fluency | Moderate | Very high |
| Training | Separate components | Single model |
| Data needed | Parallel + monolingual | Parallel corpus |
| Reordering | Explicit model needed | Learned implicitly |
| Rare words | Phrase table | Can struggle (OOV) |

---

## Revision Questions

1. **How does the encoder-decoder architecture work for translation?**
2. **What is the context vector and what is its limitation?**
3. **Explain teacher forcing and the exposure bias problem.**
4. **Why does NMT produce more fluent translations than SMT?**
5. **What happens when the source sentence is very long?**

---

[Previous: 01-statistical-mt.md](01-statistical-mt.md) | [Next: 03-attention-for-translation.md](03-attention-for-translation.md)
