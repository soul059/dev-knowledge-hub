# Pre-trained Embeddings

## Overview

Pre-trained embeddings are word vectors trained on massive corpora (billions of words) that you can download and use directly in your models. They provide high-quality representations without requiring your own large dataset, acting as a form of **transfer learning** for NLP.

---

## Available Pre-trained Embeddings

| Model | Training Data | Vocab | Dims | Download |
|-------|--------------|-------|------|----------|
| Word2Vec Google News | 100B words (news) | 3M | 300 | ~1.5 GB |
| GloVe 6B | Wikipedia + Gigaword | 400K | 50/100/200/300 | 822 MB |
| GloVe 840B | Common Crawl | 2.2M | 300 | 8.6 GB |
| GloVe Twitter | 27B tweets | 1.2M | 25/50/100/200 | 6.3 GB |
| FastText Wiki | Wikipedia | 1M | 300 | per language |
| FastText Crawl | Common Crawl | 2M | 300 | per language |

---

## Loading Pre-trained Embeddings

### Using Gensim

```python
import gensim.downloader as api

# Download and load (cached after first download)
word2vec = api.load("word2vec-google-news-300")   # 1.6 GB
glove = api.load("glove-wiki-gigaword-100")       # 128 MB
glove300 = api.load("glove-wiki-gigaword-300")    # 376 MB

# Use immediately
print(glove.most_similar("python", topn=5))
print(glove.similarity("king", "queen"))
```

### Loading GloVe from File

```python
import numpy as np

def load_glove_embeddings(file_path):
    embeddings = {}
    with open(file_path, 'r', encoding='utf-8') as f:
        for line in f:
            parts = line.strip().split()
            word = parts[0]
            vector = np.array(parts[1:], dtype=np.float32)
            embeddings[word] = vector
    print(f"Loaded {len(embeddings)} word vectors")
    return embeddings

glove = load_glove_embeddings("glove.6B.300d.txt")
# Loaded 400000 word vectors
```

---

## Using Pre-trained Embeddings in PyTorch

```python
import torch
import torch.nn as nn
import numpy as np

class TextClassifier(nn.Module):
    def __init__(self, pretrained_embeddings, num_classes, freeze=True):
        super().__init__()
        vocab_size, embed_dim = pretrained_embeddings.shape
        
        # Initialize embedding layer with pre-trained weights
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.embedding.weight = nn.Parameter(
            torch.FloatTensor(pretrained_embeddings),
            requires_grad=not freeze  # freeze=True → don't update embeddings
        )
        
        self.fc = nn.Sequential(
            nn.Linear(embed_dim, 128),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(128, num_classes)
        )
    
    def forward(self, x):
        embedded = self.embedding(x)       # (batch, seq_len, embed_dim)
        pooled = embedded.mean(dim=1)       # (batch, embed_dim) — average pooling
        return self.fc(pooled)

# Build embedding matrix for your vocabulary
def build_embedding_matrix(word_to_idx, glove, embed_dim=300):
    matrix = np.zeros((len(word_to_idx), embed_dim))
    found = 0
    for word, idx in word_to_idx.items():
        if word in glove:
            matrix[idx] = glove[word]
            found += 1
        else:
            matrix[idx] = np.random.normal(0, 0.1, embed_dim)
    print(f"Coverage: {found}/{len(word_to_idx)} ({found/len(word_to_idx):.1%})")
    return matrix

# Usage
embedding_matrix = build_embedding_matrix(word_to_idx, glove, 300)
model = TextClassifier(embedding_matrix, num_classes=3, freeze=False)
```

---

## Freeze vs Fine-tune

| Strategy | When to Use | Pros | Cons |
|----------|-------------|------|------|
| **Freeze** (`freeze=True`) | Small dataset, risk of overfitting | Prevents overfitting, faster training | Can't adapt to domain |
| **Fine-tune** (`freeze=False`) | Larger dataset, domain-specific | Adapts to your data | May overfit if dataset small |
| **Gradual unfreeze** | Medium dataset | Best of both worlds | More complex to implement |

```python
# Gradual unfreezing strategy:
# 1. Train with frozen embeddings for 5 epochs
# 2. Then unfreeze and train with lower LR

# Freeze
model.embedding.weight.requires_grad = False
train(model, epochs=5, lr=0.001)

# Unfreeze with lower LR
model.embedding.weight.requires_grad = True
train(model, epochs=10, lr=0.0001)
```

---

## Choosing the Right Pre-trained Embeddings

```
Domain of your text?
├─ General (news, Wikipedia)  → GloVe 6B or Word2Vec Google News
├─ Social media / informal    → GloVe Twitter
├─ Domain-specific            → Train your own or fine-tune
└─ Multilingual               → FastText (157 languages)

Dataset size?
├─ Small (<10K samples)       → Freeze pre-trained embeddings
├─ Medium (10K-100K)          → Fine-tune pre-trained
└─ Large (>100K)              → Can train from scratch (or fine-tune)
```

---

## Revision Questions

1. **What are the advantages of using pre-trained embeddings over training from scratch?**
2. **When should you freeze vs fine-tune pre-trained embeddings?**
3. **How do you handle words in your vocabulary that aren't in the pre-trained embeddings?**
4. **Which pre-trained embedding would you choose for analyzing tweets?**
5. **What does "coverage" mean when loading pre-trained embeddings for your vocabulary?**

---

[Previous: 05-embedding-visualization.md](05-embedding-visualization.md) | [Next: 07-oov-handling.md](07-oov-handling.md)
