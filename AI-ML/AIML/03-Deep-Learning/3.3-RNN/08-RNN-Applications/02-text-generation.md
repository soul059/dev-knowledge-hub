# Text Generation

> **Unit 8, Chapter 2** — Character-level and word-level generation, sampling strategies (greedy, temperature, top-k, nucleus), and creative text generation.

---

## 📋 Overview

Text generation uses a trained language model to produce new text autoregressively. The **sampling strategy** dramatically affects output quality — from deterministic and repetitive (greedy) to creative and diverse (nucleus sampling).

---

## 📊 Sampling Strategies

```
Given logits z = [2.0, 1.5, 0.8, 0.3, -0.5] for 5 tokens:

1. GREEDY: Always pick argmax
   → Token 0 (always same output — boring!)

2. TEMPERATURE SAMPLING: z' = z / T, then softmax + sample
   T=0.1 (sharp):  P ≈ [0.95, 0.04, 0.01, 0.00, 0.00]  (almost greedy)
   T=1.0 (normal):  P = [0.42, 0.26, 0.13, 0.08, 0.03]  (balanced)
   T=2.0 (flat):    P = [0.30, 0.25, 0.20, 0.15, 0.10]  (creative/random)

3. TOP-K: Sample only from top K most likely tokens
   K=3: Only consider tokens 0, 1, 2 → renormalize → sample

4. NUCLEUS (TOP-P): Sample from smallest set with cumulative P ≥ p
   p=0.9: Sort by prob, keep tokens until sum ≥ 0.9
   [0.42, 0.26, 0.13, 0.08] → sum=0.89 < 0.9 → add next
   [0.42, 0.26, 0.13, 0.08, 0.03] → sum=0.92 ≥ 0.9 → sample from these
```

---

## 🧮 Character-Level vs Word-Level

```
CHARACTER-LEVEL:
  Vocabulary: {a, b, c, ..., z, A-Z, 0-9, punctuation} ≈ 100
  Input:  "Hello worl"
  Output: "d"
  
  Pros: Small vocab, handles any word, no OOV
  Cons: Long sequences, harder to learn, slower

WORD-LEVEL:
  Vocabulary: {the, cat, sat, ...} ≈ 30,000-50,000
  Input:  "The cat"
  Output: "sat"
  
  Pros: Shorter sequences, easier to learn
  Cons: Large vocab, OOV problems, memory heavy

SUBWORD-LEVEL (BPE — best of both):
  Vocabulary: {Hel, lo, wor, ld, the, cat, ...} ≈ 30,000
  Input:  ["Hel", "lo", "wor"]
  Output: "ld"
```

---

## 💻 Complete Text Generator

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class CharRNN(nn.Module):
    """Character-level RNN for text generation"""
    def __init__(self, vocab_size, embed_dim=64, hidden_dim=256, num_layers=2):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(embed_dim, hidden_dim, num_layers,
                            batch_first=True, dropout=0.2)
        self.fc = nn.Linear(hidden_dim, vocab_size)
        self.hidden_dim = hidden_dim
        self.num_layers = num_layers
    
    def forward(self, x, hidden=None):
        emb = self.embedding(x)
        out, hidden = self.lstm(emb, hidden)
        logits = self.fc(out)
        return logits, hidden

def top_k_sampling(logits, k=10, temperature=1.0):
    """Sample from top-k tokens"""
    logits = logits / temperature
    top_k_logits, top_k_indices = logits.topk(k)
    probs = F.softmax(top_k_logits, dim=-1)
    idx = torch.multinomial(probs, 1)
    return top_k_indices[idx]

def nucleus_sampling(logits, p=0.9, temperature=1.0):
    """Top-p (nucleus) sampling"""
    logits = logits / temperature
    sorted_logits, sorted_indices = torch.sort(logits, descending=True)
    cumulative_probs = torch.cumsum(F.softmax(sorted_logits, dim=-1), dim=-1)
    
    # Remove tokens with cumulative prob above p
    sorted_indices_to_remove = cumulative_probs > p
    sorted_indices_to_remove[1:] = sorted_indices_to_remove[:-1].clone()
    sorted_indices_to_remove[0] = False
    
    logits[sorted_indices[sorted_indices_to_remove]] = float('-inf')
    probs = F.softmax(logits, dim=-1)
    return torch.multinomial(probs, 1)

@torch.no_grad()
def generate_text(model, seed_text, char_to_idx, idx_to_char,
                  length=200, strategy='nucleus', **kwargs):
    """Generate text with various strategies"""
    model.eval()
    tokens = [char_to_idx.get(c, 0) for c in seed_text]
    hidden = None
    
    # Process seed
    x = torch.tensor([tokens])
    _, hidden = model(x, hidden)
    
    generated = list(seed_text)
    
    for _ in range(length):
        x = torch.tensor([[tokens[-1]]])
        logits, hidden = model(x, hidden)
        logits = logits[0, -1]
        
        if strategy == 'greedy':
            next_token = logits.argmax().item()
        elif strategy == 'temperature':
            T = kwargs.get('temperature', 0.8)
            probs = F.softmax(logits / T, dim=-1)
            next_token = torch.multinomial(probs, 1).item()
        elif strategy == 'top_k':
            k = kwargs.get('k', 10)
            next_token = top_k_sampling(logits, k=k).item()
        elif strategy == 'nucleus':
            p = kwargs.get('p', 0.9)
            next_token = nucleus_sampling(logits, p=p).item()
        
        tokens.append(next_token)
        generated.append(idx_to_char.get(next_token, '?'))
    
    return ''.join(generated)

print("Sampling strategies affect generation quality dramatically!")
print("Greedy → repetitive, nucleus/top-k → creative and coherent")
```

---

## 📋 Summary Table

| Strategy | Description | Quality | Diversity |
|----------|-------------|---------|-----------|
| Greedy | Always argmax | Repetitive | None |
| Temperature (low) | Sharper distribution | Good | Low |
| Temperature (high) | Flatter distribution | Variable | High |
| Top-k | Sample from top k | Good | Moderate |
| Nucleus (top-p) | Sample from top cumulative p | Best | Adaptive |

---

## ❓ Revision Questions

1. **What is the difference between greedy decoding and temperature sampling?**
2. **Why does top-k sampling sometimes include unlikely words while nucleus sampling doesn't?**
3. **Compare character-level and word-level language models. What are the trade-offs?**
4. **At temperature T=0.01, how does sampling behave? At T=100?**
5. **Why is nucleus sampling generally preferred over top-k for open-ended generation?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Language Modeling](01-language-modeling.md) |
| ➡️ Next | [Sentiment Analysis](03-sentiment-analysis.md) |
| ⬆️ Unit Overview | [README](../README.md) |
