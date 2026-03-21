# GloVe (Global Vectors for Word Representation)

## Overview

GloVe (Pennington et al., 2014) learns word embeddings by factorizing the global **word-word co-occurrence matrix**. While Word2Vec uses local context windows, GloVe combines global statistics with local context — capturing both co-occurrence patterns and meaning. The result is embeddings that often match or exceed Word2Vec quality.

---

## Core Idea

```
Step 1: Build co-occurrence matrix X
  X[i][j] = how often word i appears near word j in the corpus

  Example (window=1):
  Corpus: "the cat sat on the mat"

          the  cat  sat  on  mat
  the   [  0    1    0    1    1  ]
  cat   [  1    0    1    0    0  ]
  sat   [  0    1    0    1    0  ]
  on    [  1    0    1    0    1  ]
  mat   [  1    0    0    1    0  ]

Step 2: Learn embeddings such that:
  w_i^T · w_j ≈ log(X[i][j])
  
  Dot product of two word vectors should approximate
  the log of how often they co-occur.
```

---

## GloVe Objective Function

```
J = Σ_{i,j=1}^{V} f(X_{ij}) (w_i^T · w̃_j + b_i + b̃_j - log X_{ij})²

Where:
  w_i, w̃_j = word and context embeddings
  b_i, b̃_j = bias terms
  X_{ij}    = co-occurrence count
  f(x)      = weighting function

Weighting function f(x):
  f(x) = { (x/x_max)^α   if x < x_max
          { 1              otherwise

  (α = 0.75, x_max = 100 typically)
  
  → Prevents very common pairs (e.g., "the the") from dominating
  → Rare pairs contribute less but aren't ignored
```

---

## Word2Vec vs GloVe

| Feature | Word2Vec | GloVe |
|---------|----------|-------|
| Training | Predict context (local) | Factorize co-occurrence (global) |
| Statistics used | Local context windows | Global co-occurrence matrix |
| Objective | Cross-entropy / neg. sampling | Weighted least squares |
| Scales with | Corpus size (streaming) | Vocabulary × Vocabulary matrix |
| Memory | Low (streaming) | High (stores full matrix) |
| Quality | Excellent | Excellent (often comparable) |
| Training speed | Fast with neg. sampling | Fast matrix factorization |

---

## Using Pre-trained GloVe

```python
import numpy as np

def load_glove(path, dim=100):
    """Load pre-trained GloVe embeddings."""
    embeddings = {}
    with open(path, 'r', encoding='utf-8') as f:
        for line in f:
            values = line.split()
            word = values[0]
            vector = np.array(values[1:], dtype='float32')
            embeddings[word] = vector
    return embeddings

# Download from: https://nlp.stanford.edu/projects/glove/
# Available: 6B (50/100/200/300d), 42B (300d), 840B (300d)
glove = load_glove('glove.6B.100d.txt', dim=100)

# Word similarity
def cosine_sim(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

print(cosine_sim(glove['king'], glove['queen']))    # ~0.75
print(cosine_sim(glove['cat'], glove['dog']))        # ~0.80
print(cosine_sim(glove['cat'], glove['computer']))   # ~0.15

# Analogy: king - man + woman ≈ queen
result = glove['king'] - glove['man'] + glove['woman']
# Find nearest word to result vector
```

---

## GloVe in PyTorch (Embedding Layer)

```python
import torch
import torch.nn as nn

def create_embedding_matrix(word_to_idx, glove, embed_dim):
    """Create embedding matrix from GloVe for PyTorch."""
    vocab_size = len(word_to_idx)
    matrix = np.zeros((vocab_size, embed_dim))
    
    found, missing = 0, 0
    for word, idx in word_to_idx.items():
        if word in glove:
            matrix[idx] = glove[word]
            found += 1
        else:
            matrix[idx] = np.random.normal(scale=0.6, size=embed_dim)
            missing += 1
    
    print(f"Found: {found}, Missing: {missing}")
    return torch.FloatTensor(matrix)

# Use in model
embedding_matrix = create_embedding_matrix(word_to_idx, glove, 100)
embedding_layer = nn.Embedding.from_pretrained(embedding_matrix, freeze=False)
# freeze=False → fine-tune during training
# freeze=True  → keep pre-trained weights fixed
```

---

## Available Pre-trained GloVe Models

| Model | Tokens | Vocab | Dims | Size |
|-------|--------|-------|------|------|
| glove.6B | 6B (Wikipedia+Gigaword) | 400K | 50/100/200/300 | 822MB |
| glove.42B | 42B (Common Crawl) | 1.9M | 300 | 5.0GB |
| glove.840B | 840B (Common Crawl) | 2.2M | 300 | 8.6GB |
| glove.twitter.27B | 27B (Twitter) | 1.2M | 25/50/100/200 | 6.3GB |

---

## Revision Questions

1. **How does GloVe's approach differ from Word2Vec's?**
2. **What is the co-occurrence matrix and how is it constructed?**
3. **What role does the weighting function f(x) play in GloVe's objective?**
4. **Why might you choose GloVe over Word2Vec (or vice versa)?**
5. **How do you integrate pre-trained GloVe embeddings into a PyTorch model?**

---

[Previous: 01-word2vec.md](01-word2vec.md) | [Next: 03-fasttext.md](03-fasttext.md)
