# One-Hot Encoding

## Overview

One-hot encoding represents each word as a binary vector where exactly one element is 1 and all others are 0. Each word gets a unique position in a vector of size V (vocabulary size). It's the simplest word representation but has fundamental limitations that motivate the need for word embeddings.

---

## How It Works

```
Vocabulary: {cat, dog, fish, bird}  (V = 4)

cat  → [1, 0, 0, 0]
dog  → [0, 1, 0, 0]
fish → [0, 0, 1, 0]
bird → [0, 0, 0, 1]

Each word is orthogonal to every other word:
  dot(cat, dog) = 0    ← no similarity!
  dot(cat, cat) = 1    ← identical only
```

---

## Implementation

```python
from sklearn.preprocessing import LabelEncoder, OneHotEncoder
import numpy as np

words = ["cat", "dog", "fish", "cat", "bird", "dog"]

# Method 1: Manual
vocab = sorted(set(words))
word_to_idx = {w: i for i, w in enumerate(vocab)}

def one_hot(word, vocab_size):
    vec = np.zeros(vocab_size)
    vec[word_to_idx[word]] = 1
    return vec

print(f"cat  = {one_hot('cat', len(vocab))}")
print(f"dog  = {one_hot('dog', len(vocab))}")
# cat  = [0. 1. 0. 0.]   (alphabetically: bird=0, cat=1, dog=2, fish=3)
# dog  = [0. 0. 1. 0.]

# Method 2: Sklearn
le = LabelEncoder()
integer_encoded = le.fit_transform(words)
ohe = OneHotEncoder(sparse_output=False)
one_hot_matrix = ohe.fit_transform(integer_encoded.reshape(-1, 1))
print(one_hot_matrix)
```

### PyTorch One-Hot

```python
import torch
import torch.nn.functional as F

vocab_size = 5
word_indices = torch.tensor([0, 2, 4, 1])  # indices of words

one_hot_vectors = F.one_hot(word_indices, num_classes=vocab_size)
print(one_hot_vectors)
# tensor([[1, 0, 0, 0, 0],
#         [0, 0, 1, 0, 0],
#         [0, 0, 0, 0, 1],
#         [0, 1, 0, 0, 0]])
```

---

## Problems with One-Hot Encoding

```
Problem 1: NO SEMANTIC SIMILARITY
  
  "king"  = [1, 0, 0, 0, ...]
  "queen" = [0, 1, 0, 0, ...]
  
  cosine_similarity(king, queen) = 0
  → They're treated as completely unrelated!

Problem 2: HUGE DIMENSIONALITY
  
  Vocabulary of 50,000 words
  → Each word is a 50,000-dimensional vector
  → 99.998% of values are zero (extremely sparse)

Problem 3: NO GENERALIZATION
  
  Model learns about "cat" — knows NOTHING about "kitten"
  (orthogonal vectors, no shared information)
```

---

## One-Hot vs Dense Embeddings

| Property | One-Hot | Dense Embedding |
|----------|---------|-----------------|
| Dimensions | V (vocabulary size) | d (50–300 typically) |
| Values | Binary (0 or 1) | Continuous floats |
| Similarity | Always 0 between words | Captures semantics |
| Memory | V × V identity matrix | V × d matrix |
| Learned? | No (fixed) | Yes (from data) |
| "king" ≈ "queen" | No | Yes |

```
One-Hot (V=50000):  [0, 0, ..., 1, ..., 0, 0]   50,000 dims
Dense Embedding:     [0.23, -0.45, 0.89, ...]     300 dims

Dense embeddings solve all three problems:
  ✓ Semantic similarity captured
  ✓ Low dimensional (50–300)
  ✓ Related words have similar vectors
```

---

## When One-Hot Is Still Used

- **Input to embedding layers**: Neural networks convert one-hot (or word index) to dense embeddings via `nn.Embedding`
- **Categorical features**: Non-NLP tabular data (e.g., country, color)
- **Small vocabularies**: When V is small, one-hot is fine
- **Output layer**: Softmax over vocabulary produces probability distribution matching one-hot targets

---

## Revision Questions

1. **Why is the dot product of any two different one-hot vectors always 0?**
2. **What are the three main problems with one-hot encoding for NLP?**
3. **How does an embedding layer in PyTorch relate to one-hot encoding?**
4. **Calculate the memory needed for one-hot encoding of a 100K word vocabulary.**
5. **When is one-hot encoding still appropriate in modern NLP?**

---

[Previous: 03-n-grams.md](03-n-grams.md) | [Next: 05-word-embeddings-intro.md](05-word-embeddings-intro.md)
