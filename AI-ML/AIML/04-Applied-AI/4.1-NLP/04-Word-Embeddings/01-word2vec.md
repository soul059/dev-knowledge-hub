# Word2Vec (CBOW & Skip-gram)

## Overview

Word2Vec (Mikolov et al., 2013) learns dense word embeddings by training a shallow neural network on a simple prediction task. It comes in two flavors: **CBOW** (predict center word from context) and **Skip-gram** (predict context words from center word). Word2Vec demonstrated that simple models trained on massive text can capture rich semantic relationships.

---

## Architecture

```
CBOW (Continuous Bag of Words):
  Predict CENTER word from CONTEXT words

  Context: ["the", "cat", "on", "the"]  →  Predict: "sat"

  Input          Projection       Output
  the ──┐
  cat ──┤──▶  Average  ──▶  Softmax ──▶ "sat"
  on  ──┤    (d dims)        (V words)
  the ──┘


Skip-gram:
  Predict CONTEXT words from CENTER word

  Center: "sat"  →  Predict: ["the", "cat", "on", "the"]

  Input          Projection       Output
                                  ──▶ "the"
  "sat" ──▶    Embedding  ──▶   ──▶ "cat"
               (d dims)          ──▶ "on"
                                  ──▶ "the"
```

---

## Mathematical Formulation

### Skip-gram Objective

Maximize the probability of context words given center word:

```
J = (1/T) Σ_{t=1}^{T} Σ_{-c≤j≤c, j≠0} log P(w_{t+j} | w_t)

Where:
  T = total words in corpus
  c = context window size
  w_t = center word at position t

P(w_O | w_I) = exp(v'_{w_O}^T · v_{w_I}) / Σ_{w=1}^{V} exp(v'_w^T · v_{w_I})

  v_{w_I} = input embedding of center word
  v'_{w_O} = output embedding of context word
```

### Training Efficiency: Negative Sampling

Computing softmax over entire vocabulary (V ~ 100K) is expensive. Negative sampling approximates it:

```
Instead of: P(context | center) using full softmax over V words
Use:        P(context | center) vs P(random_words | center)

For each positive pair (center, context):
  Sample k negative words randomly
  Train binary classifier: is this a real context word or not?

log σ(v'_{w_O}^T · v_{w_I}) + Σ_{i=1}^{k} E[log σ(-v'_{w_i}^T · v_{w_I})]

Typical k = 5-20 (much cheaper than V = 100,000)
```

---

## Training Example

```
Corpus: "I love machine learning and deep learning"
Window size c = 2

For center word "machine" (position 3):
  Context words: ["love", "learning"]  (within window of 2)

Skip-gram training pairs:
  (machine, love)      → positive pair
  (machine, learning)  → positive pair
  (machine, pizza)     → negative sample
  (machine, orange)    → negative sample
```

---

## Implementation with Gensim

```python
from gensim.models import Word2Vec

# Training corpus (list of tokenized sentences)
sentences = [
    ["i", "love", "machine", "learning"],
    ["deep", "learning", "is", "powerful"],
    ["natural", "language", "processing", "uses", "machine", "learning"],
    ["neural", "networks", "enable", "deep", "learning"],
]

# Train Word2Vec (Skip-gram)
model = Word2Vec(
    sentences,
    vector_size=100,   # embedding dimensions
    window=3,          # context window size
    min_count=1,       # min word frequency
    sg=1,              # 1=Skip-gram, 0=CBOW
    negative=5,        # negative samples
    epochs=100
)

# Get word vector
vec = model.wv["learning"]
print(f"Shape: {vec.shape}")  # (100,)

# Find similar words
print(model.wv.most_similar("learning", topn=3))

# Save and load
model.save("word2vec.model")
loaded = Word2Vec.load("word2vec.model")
```

---

## CBOW vs Skip-gram

| Feature | CBOW | Skip-gram |
|---------|------|-----------|
| Predicts | Center from context | Context from center |
| Speed | Faster (one prediction) | Slower (multiple predictions) |
| Rare words | Worse (averaged context) | Better (each word gets trained) |
| Common words | Better | Good |
| Training data | Less data needed | More data needed |
| Default choice | Small datasets | Large datasets (recommended) |

---

## Hyperparameters

| Parameter | Typical Values | Effect |
|-----------|:---:|------|
| `vector_size` | 100–300 | Higher = more expressive, more data needed |
| `window` | 2–10 | Larger = more syntactic; smaller = more semantic |
| `min_count` | 5–10 | Filter rare words |
| `sg` | 0 or 1 | CBOW vs Skip-gram |
| `negative` | 5–20 | More negatives = better quality, slower training |
| `epochs` | 5–20 | More epochs for small corpora |

---

## Revision Questions

1. **What is the key difference between CBOW and Skip-gram architectures?**
2. **Why is negative sampling necessary, and how does it work?**
3. **Write the Skip-gram objective function and explain each term.**
4. **How does window size affect the type of relationships captured?**
5. **When would you choose CBOW over Skip-gram?**
6. **How does Word2Vec relate to the distributional hypothesis?**

---

[Previous: Unit 3 - 06-sparse-vs-dense.md](../03-Text-Representation/06-sparse-vs-dense.md) | [Next: 02-glove.md](02-glove.md)
