# FastText

## Overview

FastText (Bojanowski et al., 2017, Facebook AI) extends Word2Vec by representing each word as a **bag of character n-grams**. Instead of learning one vector per word, FastText learns vectors for character n-grams and sums them to get the word vector. This enables FastText to generate embeddings for **unseen words** (OOV) — a critical advantage over Word2Vec and GloVe.

---

## Key Innovation: Subword Embeddings

```
Word: "learning"

Character n-grams (n=3 to 6, with boundary markers < >):
  <le, lea, lear, learn
  ear, earn, earni
  arn, arni, arnin
  rni, rnin, rning
  nin, ning, ning>
  ing, ing>
  ng>

Word vector = Σ (all n-gram vectors) + whole word vector

"learning" = v(<le) + v(lea) + v(lear) + ... + v(ng>) + v(learning)
```

---

## Why Subword N-grams Matter

```
Word2Vec / GloVe:
  "unfriendliness" → not in vocabulary → <UNK> ❌
  
FastText:
  "unfriendliness" → decompose into n-grams
  → shares n-grams with "unfriendly", "friendliness", "friendly"
  → can compute a meaningful vector! ✅

Typos:
  "learning" → known word
  "lerning"  → unknown to Word2Vec → <UNK>
  → FastText: shares n-grams with "learning" → reasonable vector!
```

---

## Training

FastText uses the same Skip-gram architecture as Word2Vec, but modifies the scoring function:

```
Word2Vec Skip-gram score:
  s(w, c) = v_w^T · v_c

FastText score:
  s(w, c) = Σ_{g ∈ G(w)} v_g^T · v_c

Where G(w) = set of all character n-grams of word w (plus whole word)
```

---

## Implementation

```python
from gensim.models import FastText

sentences = [
    ["machine", "learning", "is", "fascinating"],
    ["deep", "learning", "enables", "breakthroughs"],
    ["natural", "language", "processing", "uses", "embeddings"],
    ["neural", "networks", "learn", "representations"],
]

# Train FastText
model = FastText(
    sentences,
    vector_size=100,
    window=3,
    min_count=1,
    sg=1,           # Skip-gram
    min_n=3,        # min character n-gram length
    max_n=6,        # max character n-gram length
    epochs=50
)

# Regular word vector
vec = model.wv["learning"]

# OOV word — FastText can still produce a vector!
vec_oov = model.wv["learnings"]   # never seen in training
print(f"'learnings' vector shape: {vec_oov.shape}")  # (100,) — works!

# Similarity
print(model.wv.similarity("learning", "learn"))   # high
print(model.wv.similarity("learning", "machine")) # moderate
```

---

## Word2Vec vs GloVe vs FastText

| Feature | Word2Vec | GloVe | FastText |
|---------|----------|-------|----------|
| Unit | Whole word | Whole word | Character n-grams |
| OOV handling | ❌ `<UNK>` | ❌ `<UNK>` | ✅ Subword composition |
| Typo robustness | ❌ | ❌ | ✅ Shared n-grams |
| Morphology | ❌ Ignores | ❌ Ignores | ✅ Captures (un-, -ing, -tion) |
| Training data | Local context | Global co-occurrence | Local context + subwords |
| Speed | Fast | Fast | Slower (more parameters) |
| Model size | Small | Small | Larger (n-gram vectors) |
| Best for | Common words | Common words | Morphologically rich langs |

---

## When to Use FastText

```
✅ Use FastText when:
  • Many OOV / rare words expected
  • Morphologically rich languages (Turkish, Finnish, Arabic)
  • Noisy text with typos (social media, OCR)
  • Small training corpus (subwords help generalize)

❌ Stick with Word2Vec/GloVe when:
  • Clean text, common vocabulary
  • Memory is limited (FastText models are larger)
  • Using pre-trained models (GloVe pre-trained widely available)
```

---

## Pre-trained FastText Models

Available at [fasttext.cc](https://fasttext.cc/docs/en/crawl-vectors.html):
- **157 languages** with 300-dimensional vectors
- Trained on Common Crawl + Wikipedia
- Both `.bin` (with subword info) and `.vec` (text format)

---

## Revision Questions

1. **How does FastText represent a word differently from Word2Vec?**
2. **Why can FastText generate vectors for unseen words but Word2Vec cannot?**
3. **What are character n-grams and how are they extracted from a word?**
4. **For which types of languages is FastText especially beneficial?**
5. **What is the trade-off of using FastText over Word2Vec in terms of model size?**

---

[Previous: 02-glove.md](02-glove.md) | [Next: 04-embedding-properties.md](04-embedding-properties.md)
