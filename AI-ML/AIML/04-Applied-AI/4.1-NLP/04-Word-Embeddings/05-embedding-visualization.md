# Embedding Visualization

## Overview

Word embeddings live in high-dimensional space (100–300 dimensions) which we can't directly visualize. Dimensionality reduction techniques like **t-SNE** and **PCA** project these vectors down to 2D or 3D, revealing clusters, relationships, and patterns that help us understand and debug our embeddings.

---

## Visualization Techniques

```
300-dimensional embeddings
         │
         ├──▶ PCA (2D/3D)       → Fast, preserves global structure
         ├──▶ t-SNE (2D/3D)     → Slow, preserves local neighborhoods  
         └──▶ UMAP (2D/3D)      → Fast, preserves both local + global
```

---

## PCA Visualization

```python
import numpy as np
from sklearn.decomposition import PCA
import matplotlib.pyplot as plt

# Sample word vectors (using pre-trained)
import gensim.downloader as api
model = api.load("glove-wiki-gigaword-100")

words = ['king', 'queen', 'man', 'woman', 'prince', 'princess',
         'cat', 'dog', 'fish', 'bird',
         'happy', 'sad', 'angry', 'joyful',
         'paris', 'london', 'berlin', 'tokyo']

vectors = np.array([model[w] for w in words])

# PCA to 2D
pca = PCA(n_components=2)
reduced = pca.fit_transform(vectors)

plt.figure(figsize=(12, 8))
for i, word in enumerate(words):
    plt.scatter(reduced[i, 0], reduced[i, 1])
    plt.annotate(word, (reduced[i, 0]+0.02, reduced[i, 1]+0.02))
plt.title("Word Embeddings - PCA Projection")
plt.xlabel(f"PC1 ({pca.explained_variance_ratio_[0]:.1%} variance)")
plt.ylabel(f"PC2 ({pca.explained_variance_ratio_[1]:.1%} variance)")
plt.grid(True, alpha=0.3)
plt.show()
```

---

## t-SNE Visualization

```python
from sklearn.manifold import TSNE

# t-SNE to 2D (better for cluster visualization)
tsne = TSNE(n_components=2, perplexity=5, random_state=42, n_iter=1000)
reduced_tsne = tsne.fit_transform(vectors)

# Color by category
categories = {
    'royalty': ['king', 'queen', 'prince', 'princess', 'man', 'woman'],
    'animals': ['cat', 'dog', 'fish', 'bird'],
    'emotions': ['happy', 'sad', 'angry', 'joyful'],
    'cities': ['paris', 'london', 'berlin', 'tokyo']
}

colors = {'royalty': 'red', 'animals': 'blue', 
          'emotions': 'green', 'cities': 'orange'}

plt.figure(figsize=(12, 8))
for cat, cat_words in categories.items():
    indices = [words.index(w) for w in cat_words]
    plt.scatter(reduced_tsne[indices, 0], reduced_tsne[indices, 1],
                c=colors[cat], label=cat, s=100)
    for i in indices:
        plt.annotate(words[i], (reduced_tsne[i, 0]+0.5, reduced_tsne[i, 1]+0.5))

plt.legend()
plt.title("Word Embeddings - t-SNE Projection")
plt.show()
```

---

## PCA vs t-SNE vs UMAP

| Method | Speed | Global Structure | Local Structure | Deterministic |
|--------|:-----:|:---:|:---:|:---:|
| PCA | ⚡ Fast | ✅ Preserved | ❌ Distorted | ✅ Yes |
| t-SNE | 🐢 Slow | ❌ Lost | ✅ Excellent | ❌ No |
| UMAP | ⚡ Fast | ✅ Good | ✅ Good | ❌ No |

```
Use PCA when:  → Quick overview, interpretable axes
Use t-SNE when: → Finding clusters, exploring neighborhoods
Use UMAP when:  → Best of both worlds, large datasets
```

---

## Interactive Visualization

```python
# TensorBoard Embedding Projector
from torch.utils.tensorboard import SummaryWriter
import torch

writer = SummaryWriter('runs/embeddings')

# Stack word vectors into a tensor
embeddings_tensor = torch.FloatTensor(vectors)

# Add to TensorBoard
writer.add_embedding(
    embeddings_tensor,
    metadata=words,
    tag='word_embeddings'
)
writer.close()

# Run: tensorboard --logdir=runs
# Open: http://localhost:6006/#projector
# → Interactive 3D visualization with PCA/t-SNE/UMAP
```

---

## What to Look For in Visualizations

```
✅ Good embeddings show:
  • Semantic clusters (animals together, cities together)
  • Analogical parallels (man→woman parallel to king→queen)
  • Smooth gradients (happy → content → neutral → sad)

❌ Warning signs:
  • Random scatter (no clustering)
  • Known synonyms far apart
  • Unrelated words clustered together
```

---

## Revision Questions

1. **Why can't we directly visualize 300-dimensional word embeddings?**
2. **What is the key difference between PCA and t-SNE for visualization?**
3. **What patterns should you expect to see in well-trained embeddings?**
4. **How does the perplexity parameter affect t-SNE output?**
5. **How can you use TensorBoard to interactively explore embeddings?**

---

[Previous: 04-embedding-properties.md](04-embedding-properties.md) | [Next: 06-pretrained-embeddings.md](06-pretrained-embeddings.md)
