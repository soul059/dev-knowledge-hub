# Sparse vs Dense Representations

## Overview

Text can be represented as either **sparse** vectors (one-hot, BoW, TF-IDF) or **dense** vectors (word embeddings, BERT outputs). This distinction is fundamental in NLP — it affects model performance, memory usage, computation speed, and semantic capability.

---

## Visual Comparison

```
Word: "neural"    Vocabulary size: 50,000

Sparse (one-hot):
[0, 0, 0, ..., 0, 1, 0, ..., 0, 0]    ← 50,000 dims, one non-zero
                   ↑
              position 27,341

Dense (embedding):
[0.23, -0.45, 0.12, 0.89, -0.31, ...]  ← 300 dims, all non-zero


Document: "machine learning is great"

Sparse (TF-IDF):
[0, 0, ..., 0.42, ..., 0, 0.31, ..., 0.55, ..., 0]  ← 50,000 dims, ~4 non-zero

Dense (avg embedding):
[0.15, -0.28, 0.67, 0.33, -0.19, ...]  ← 300 dims, all non-zero
```

---

## Detailed Comparison

| Property | Sparse | Dense |
|----------|--------|-------|
| **Dimensions** | V (10K–100K+) | d (50–1024) |
| **Values** | Mostly 0, few non-zero | All dimensions active |
| **Similarity** | Exact word overlap only | Semantic similarity |
| **Memory** | Efficient with sparse matrices | Fixed per vector |
| **Interpretable** | Each dim = specific word | Dims not directly interpretable |
| **OOV handling** | Cannot represent unseen words | Subword fallback possible |
| **Training** | No training needed (counts) | Requires training data |
| **"happy" ≈ "joyful"** | Similarity = 0 | Similarity ≈ 0.8 |

---

## Memory and Compute

```
10,000 documents, 50,000 vocabulary:

Sparse TF-IDF matrix:
  Full: 10,000 × 50,000 × 4 bytes = 2 GB
  Sparse storage: ~50 MB (only non-zeros stored)
  → scipy.sparse.csr_matrix handles this efficiently

Dense embedding matrix:
  10,000 × 300 × 4 bytes = 12 MB
  → Always this size, no sparsity optimization needed
```

```python
from scipy.sparse import csr_matrix
import numpy as np

# Sparse: efficient for BoW/TF-IDF
sparse_vec = csr_matrix(([0.5, 0.3], ([0, 0], [100, 5000])), shape=(1, 50000))
print(f"Non-zeros: {sparse_vec.nnz}, Memory: ~{sparse_vec.data.nbytes} bytes")

# Dense: efficient for embeddings
dense_vec = np.random.randn(300).astype(np.float32)
print(f"Memory: {dense_vec.nbytes} bytes")  # 1200 bytes
```

---

## When to Use Which

```
Use SPARSE when:
  ✅ Large vocabulary, few words per doc
  ✅ Need exact keyword matching
  ✅ Interpretability matters (which words?)
  ✅ Simple baseline (TF-IDF + SVM)
  ✅ Limited compute / no GPU

Use DENSE when:
  ✅ Need semantic similarity
  ✅ Related words should match ("car" ≈ "automobile")
  ✅ Using neural network models
  ✅ Transfer learning / pre-trained models
  ✅ Any modern NLP task

Use HYBRID (both):
  ✅ Combine TF-IDF features with embeddings
  ✅ Information retrieval (sparse for recall, dense for ranking)
  ✅ RAG systems (BM25 + vector search)
```

---

## From Sparse to Dense: The Evolution

```
1990s: BoW/TF-IDF (sparse)
  → Works for search, basic classification
  → No semantics

2000s: LSA/SVD (dense, from sparse)
  → Apply SVD to TF-IDF matrix
  → Reduce to 100-300 dims
  → Some semantics captured

2013: Word2Vec (dense, learned)
  → Train on context prediction
  → Rich semantic features

2018: BERT (dense, contextual)
  → Context-dependent embeddings
  → State-of-the-art for most tasks
```

---

## Revision Questions

1. **What makes a representation "sparse" vs "dense"?**
2. **Why can sparse representations not capture semantic similarity?**
3. **How does scipy handle sparse matrices efficiently?**
4. **When might sparse TF-IDF features outperform dense embeddings?**
5. **Explain how hybrid sparse+dense approaches work in search systems.**

---

[Previous: 05-word-embeddings-intro.md](05-word-embeddings-intro.md) | [Next: Unit 4 - 01-word2vec.md](../04-Word-Embeddings/01-word2vec.md)
