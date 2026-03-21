# Word Embeddings Introduction

## Overview

Word embeddings are dense, low-dimensional vector representations of words where semantically similar words have similar vectors. Unlike one-hot encoding (sparse, no similarity), embeddings capture meaning: "king" and "queen" are close in embedding space, while "king" and "pizza" are far apart. This is the single most important breakthrough in modern NLP.

---

## The Key Idea

```
One-Hot (no similarity):
  king  = [1, 0, 0, 0, 0, ...]   (50,000 dims)
  queen = [0, 1, 0, 0, 0, ...]
  similarity = 0 

Word Embedding (captures meaning):
  king  = [0.52, -0.31, 0.89, 0.12, ...]   (300 dims)
  queen = [0.49, -0.28, 0.91, 0.15, ...]
  similarity = 0.95 ← high!

  pizza = [-0.22, 0.67, -0.13, 0.45, ...]
  similarity(king, pizza) = 0.08 ← low!
```

---

## Distributional Hypothesis

> "You shall know a word by the company it keeps" — J.R. Firth (1957)

Words that appear in similar contexts have similar meanings:

```
"The ___ sat on the throne"     → king, queen, emperor
"The ___ ran across the field"  → dog, cat, horse
"I ate ___ for breakfast"       → eggs, toast, cereal

Words filling the same blank → similar embeddings
```

---

## What Embedding Dimensions Capture

Each dimension loosely corresponds to a semantic feature (though not explicitly labeled):

```
Hypothetical dimensions (simplified):

          Royalty  Gender  Animate  Food
king    [  0.9     0.8      0.9    0.1  ]
queen   [  0.9     0.2      0.9    0.1  ]
man     [  0.1     0.8      0.9    0.1  ]
woman   [  0.1     0.2      0.9    0.1  ]
apple   [  0.0     0.0      0.0    0.9  ]

king - man + woman ≈ queen  (famous analogy!)
```

---

## Famous Word Analogies

```
vector("king") - vector("man") + vector("woman") ≈ vector("queen")
vector("Paris") - vector("France") + vector("Italy") ≈ vector("Rome")
vector("walking") - vector("walked") + vector("swam") ≈ vector("swimming")
```

This arithmetic shows embeddings capture **relationships**:
- Gender: king→queen, man→woman
- Capital: France→Paris, Italy→Rome
- Tense: walked→walking, swam→swimming

---

## Methods to Learn Embeddings

```
┌────────────────────────────────────────────────────┐
│         WORD EMBEDDING METHODS                      │
│                                                    │
│  Count-based:              Prediction-based:       │
│  ├─ Co-occurrence matrix    ├─ Word2Vec (2013)     │
│  ├─ SVD/PCA reduction      │   ├─ CBOW            │
│  └─ GloVe (hybrid)         │   └─ Skip-gram       │
│                             ├─ FastText (2016)     │
│                             └─ ELMo (2018)         │
│                                                    │
│  Contextual (modern):                              │
│  ├─ BERT embeddings                                │
│  ├─ GPT embeddings                                 │
│  └─ Per-token, context-dependent                   │
└────────────────────────────────────────────────────┘
```

---

## Quick Demo with Gensim

```python
import gensim.downloader as api

# Load pre-trained Word2Vec embeddings
model = api.load("word2vec-google-news-300")

# Find similar words
print(model.most_similar("king", topn=5))
# [('kings', 0.71), ('queen', 0.65), ('monarch', 0.64), ...]

# Word analogy: king - man + woman = ?
result = model.most_similar(positive=['king', 'woman'], 
                             negative=['man'], topn=1)
print(result)  # [('queen', 0.71)]

# Similarity between words
print(model.similarity('cat', 'dog'))     # 0.76 (both animals)
print(model.similarity('cat', 'car'))     # 0.20 (unrelated)
```

---

## Static vs Contextual Embeddings

| Feature | Static (Word2Vec/GloVe) | Contextual (BERT/GPT) |
|---------|------------------------|----------------------|
| One vector per word? | Yes | No — different per context |
| "bank" (river vs money) | Same vector | Different vectors! |
| Pre-trained? | Yes | Yes |
| Dimensions | 50–300 | 768–1024 |
| Handles polysemy? | No | Yes |

```
Static:  "bank" → always [0.3, -0.1, 0.5, ...]
                   regardless of context

BERT:    "river bank"  → "bank" = [0.2, 0.8, -0.3, ...]
         "bank account" → "bank" = [-0.5, 0.1, 0.6, ...]
         Different vectors for different meanings!
```

---

## Revision Questions

1. **What is the distributional hypothesis and how does it justify word embeddings?**
2. **How does the king–queen analogy work mathematically?**
3. **What are the key differences between one-hot and dense embeddings?**
4. **What is the difference between static and contextual word embeddings?**
5. **Why is it significant that word embeddings capture analogies?**
6. **Name three methods for learning word embeddings.**

---

[Previous: 04-one-hot-encoding.md](04-one-hot-encoding.md) | [Next: 06-sparse-vs-dense.md](06-sparse-vs-dense.md)
