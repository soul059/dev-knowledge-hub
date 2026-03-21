# Out-of-Vocabulary (OOV) Handling

## Overview

Out-of-vocabulary (OOV) words are tokens that appear at inference time but were not in the training vocabulary. This is a critical problem for static embeddings (Word2Vec, GloVe) — if a word isn't in the vocabulary, there's no vector for it. This chapter covers strategies to handle OOV words gracefully.

---

## The OOV Problem

```
Training vocabulary: {"cat", "dog", "happy", "learning", "neural", ...}

At inference time:
  "My labradoodle is adorbs"
       ↑              ↑
   Not in vocab    Not in vocab

Word2Vec / GloVe response: <UNK> token → loses all meaning
FastText response: Compose from subwords → reasonable vector ✓
```

---

## OOV Handling Strategies

### 1. Unknown Token (`<UNK>`)

```python
# Simplest approach: map all OOV words to a single <UNK> vector
UNK_IDX = 0
def encode_word(word, word_to_idx):
    return word_to_idx.get(word, UNK_IDX)

# Problem: ALL unknown words get the same representation
# "labradoodle" == "cryptocurrency" == "asdfghjkl"
```

### 2. Random Initialization

```python
import numpy as np

def get_embedding(word, embeddings, embed_dim=300):
    if word in embeddings:
        return embeddings[word]
    else:
        # Random vector with similar statistics to trained embeddings
        return np.random.normal(0, 0.1, embed_dim)
```

### 3. Subword Approaches (Best for Static Embeddings)

```python
# FastText: automatically handles OOV via character n-grams
from gensim.models import FastText

model = FastText(sentences, vector_size=100, min_n=3, max_n=6)

# These words were never in training, but get vectors:
vec1 = model.wv["unseen_word"]      # composed from n-grams
vec2 = model.wv["misspeling"]       # similar to "misspelling"
```

### 4. Character-Level Fallback

```python
import torch
import torch.nn as nn

class CharEmbedding(nn.Module):
    """Generate word embedding from characters (handles any word)."""
    def __init__(self, char_vocab_size, char_embed_dim, word_embed_dim):
        super().__init__()
        self.char_embed = nn.Embedding(char_vocab_size, char_embed_dim)
        self.cnn = nn.Conv1d(char_embed_dim, word_embed_dim, kernel_size=3, padding=1)
        self.pool = nn.AdaptiveMaxPool1d(1)
    
    def forward(self, char_indices):
        # char_indices: (batch, max_word_len)
        x = self.char_embed(char_indices)    # (batch, max_word_len, char_embed)
        x = x.permute(0, 2, 1)              # (batch, char_embed, max_word_len)
        x = self.cnn(x)                      # (batch, word_embed, max_word_len)
        x = self.pool(x).squeeze(-1)         # (batch, word_embed)
        return x

# Any word can be embedded, even completely novel ones
```

### 5. Subword Tokenization (Modern Solution)

```
BPE/WordPiece eliminates OOV entirely:

"labradoodle" → ["lab", "##ra", "##doo", "##dle"]
"cryptocurrency" → ["crypto", "##currency"]
"asdfghjkl" → ["as", "##df", "##gh", "##j", "##kl"]

Every possible string can be decomposed into known subwords.
No word is ever truly "unknown."
```

---

## Strategy Comparison

| Strategy | OOV Quality | Implementation | When to Use |
|----------|:-:|:-:|---|
| `<UNK>` token | ❌ Poor | Trivial | Quick baseline |
| Random init | ⚠️ Okay | Easy | With fine-tuning |
| FastText subwords | ✅ Good | Use FastText | Static embeddings |
| Character CNN/LSTM | ✅ Good | Moderate | Custom models |
| BPE/WordPiece | ✅ Excellent | Use tokenizer | Transformers (standard) |
| Spelling correction | ✅ Good | NLP library | Noisy text |

---

## Practical Recommendations

```
Using static embeddings (Word2Vec/GloVe)?
  → Use FastText instead (built-in OOV handling)
  → Or add character-level CNN as fallback

Using transformers (BERT/GPT)?
  → OOV handled automatically by subword tokenization
  → No special handling needed

High OOV rate (>5% of tokens)?
  → Check if vocabulary covers your domain
  → Consider domain-specific embeddings
  → Use subword-based model
```

---

## Revision Questions

1. **What causes the OOV problem and why is it critical for NLP?**
2. **Why is using a single `<UNK>` token a poor solution?**
3. **How does FastText handle OOV words that Word2Vec cannot?**
4. **How does subword tokenization (BPE) eliminate OOV entirely?**
5. **When would you use a character-level CNN to generate word embeddings?**

---

[Previous: 06-pretrained-embeddings.md](06-pretrained-embeddings.md) | [Next: Unit 5 - 01-problem-formulation.md](../05-Text-Classification/01-problem-formulation.md)
