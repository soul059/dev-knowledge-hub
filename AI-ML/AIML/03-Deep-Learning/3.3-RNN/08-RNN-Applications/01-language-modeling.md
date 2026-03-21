# Language Modeling

> **Unit 8, Chapter 1** — Next word prediction with RNNs: perplexity metric, training procedure, and PyTorch example.

---

## 📋 Overview

**Language modeling** is the task of predicting the next token given a sequence of preceding tokens. It's the foundational task for many NLP applications and is how RNNs learn the structure and patterns of language.

---

## 🧮 Problem Formulation

```
Given: sequence x₁, x₂, ..., x_{t-1}
Predict: P(x_t | x₁, x₂, ..., x_{t-1})

Example:
  "The cat sat on the" → P(next_word | context)
  P("mat") = 0.15, P("floor") = 0.12, P("chair") = 0.08, ...

Training objective:
  Maximize P(x₁, x₂, ..., x_T) = ∏ P(x_t | x₁, ..., x_{t-1})
  
  Equivalently, minimize negative log-likelihood:
  L = -Σ log P(x_t | x₁, ..., x_{t-1})
    = Σ CrossEntropy(prediction_t, x_t)
```

---

## 📊 Perplexity

```
Perplexity (PPL) = exp(average cross-entropy loss)
                 = exp(-(1/T) Σ log P(x_t | x_{<t}))

Interpretation:
  PPL = N means the model is as confused as if choosing
  uniformly among N equally likely words

  PPL = 1:     Perfect prediction (impossible in practice)
  PPL = 10:    Very good (like choosing among 10 words)
  PPL = 100:   Moderate
  PPL = 1000:  Poor (like choosing among 1000 words)
  PPL = |V|:   Random guess (uniform over vocabulary)

Lower perplexity = better language model
```

---

## 💻 PyTorch Language Model

```python
import torch
import torch.nn as nn

class RNNLanguageModel(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_layers=2, dropout=0.3):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.LSTM(embed_dim, hidden_dim, num_layers=num_layers,
                           batch_first=True, dropout=dropout)
        self.fc = nn.Linear(hidden_dim, vocab_size)
        self.drop = nn.Dropout(dropout)
        
        # Weight tying: share embedding and output weights
        if embed_dim == hidden_dim:
            self.fc.weight = self.embedding.weight
    
    def forward(self, x, hidden=None):
        """
        x: (batch, seq_len) — input token indices
        Returns: (batch, seq_len, vocab_size) — next-token logits
        """
        embedded = self.drop(self.embedding(x))
        output, hidden = self.rnn(embedded, hidden)
        output = self.drop(output)
        logits = self.fc(output)
        return logits, hidden

# Training
model = RNNLanguageModel(vocab_size=10000, embed_dim=256, hidden_dim=256)
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

# Simulated batch: predict next token at each position
x = torch.randint(0, 10000, (32, 50))  # Input tokens
target = torch.randint(0, 10000, (32, 50))  # Next tokens (shifted by 1)

logits, _ = model(x)
loss = criterion(logits.view(-1, 10000), target.view(-1))
perplexity = torch.exp(loss)

print(f"Loss: {loss.item():.4f}")
print(f"Perplexity: {perplexity.item():.2f}")

# Generation
@torch.no_grad()
def generate(model, seed_tokens, max_len=20, temperature=1.0):
    model.eval()
    tokens = seed_tokens.tolist() if isinstance(seed_tokens, torch.Tensor) else seed_tokens
    hidden = None
    
    for _ in range(max_len):
        x = torch.tensor([tokens[-1:]]) 
        logits, hidden = model(x, hidden)
        logits = logits[0, -1] / temperature
        probs = torch.softmax(logits, dim=0)
        next_token = torch.multinomial(probs, 1).item()
        tokens.append(next_token)
    
    return tokens
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Language Modeling | Predict P(next_token \| context) |
| Training | Minimize cross-entropy at each position |
| Perplexity | exp(avg CE loss) — lower is better |
| Weight Tying | Share embedding and output layer weights |
| Generation | Sample from predicted distribution autoregressively |
| Applications | Text generation, spell check, speech recognition |

---

## ❓ Revision Questions

1. **What is the training objective for a language model? Write the loss function.**
2. **If a language model has perplexity of 50, what does this mean intuitively?**
3. **Why is weight tying between the embedding and output layer beneficial?**
4. **How does temperature affect text generation? What happens at T→0 and T→∞?**
5. **Why is language modeling considered a self-supervised task?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Seq2Seq Limitations](../07-Sequence-to-Sequence/04-limitations.md) |
| ➡️ Next | [Text Generation](02-text-generation.md) |
| ⬆️ Unit Overview | [README](../README.md) |
