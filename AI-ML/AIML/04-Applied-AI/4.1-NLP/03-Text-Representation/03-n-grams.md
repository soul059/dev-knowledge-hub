# N-grams

## Overview

An n-gram is a contiguous sequence of **n** items (words, characters, or subwords) from a given text. N-grams capture local word order and context that Bag of Words misses entirely. They are fundamental to language models, text classification, and spelling correction.

---

## Types of N-grams

```
Sentence: "I love natural language processing"

Unigrams (n=1): ["I", "love", "natural", "language", "processing"]
Bigrams  (n=2): ["I love", "love natural", "natural language", "language processing"]
Trigrams (n=3): ["I love natural", "love natural language", "natural language processing"]
4-grams  (n=4): ["I love natural language", "love natural language processing"]
```

---

## Character N-grams

```
Word: "hello"

Char bigrams:  ["he", "el", "ll", "lo"]
Char trigrams: ["hel", "ell", "llo"]

Useful for:
  • Spelling correction ("teh" → char bigrams overlap with "the")
  • Language detection (char n-gram frequencies differ by language)
  • Handling typos and morphological variants
```

---

## Implementation

```python
from sklearn.feature_extraction.text import CountVectorizer

corpus = [
    "I love machine learning",
    "I love deep learning",
    "machine learning is powerful"
]

# Word bigrams
bigram_vec = CountVectorizer(ngram_range=(2, 2))
X = bigram_vec.fit_transform(corpus)
print("Bigrams:", bigram_vec.get_feature_names_out())
# ['deep learning', 'is powerful', 'love deep', 'love machine', 
#  'machine learning', 'learning is']

# Unigrams + Bigrams combined
combo_vec = CountVectorizer(ngram_range=(1, 2))
X = combo_vec.fit_transform(corpus)
print(f"Features: {len(combo_vec.get_feature_names_out())}")
# More features, but captures both individual words AND pairs
```

### Character N-grams

```python
char_vec = CountVectorizer(analyzer='char', ngram_range=(2, 3))
X = char_vec.fit_transform(["hello", "world"])
print(char_vec.get_feature_names_out()[:10])
# [' w', 'el', 'he', 'hel', 'ell', 'll', 'llo', 'lo', 'or', ...]
```

---

## N-grams in Language Modeling

```
Bigram probability:
  P("learning" | "machine") = count("machine learning") / count("machine")

If corpus has:
  "machine learning" appears 50 times
  "machine" appears 60 times

  P("learning" | "machine") = 50/60 = 0.833

Predict next word:
  "I love machine ___"
  → P("learning" | "machine") = 0.833  ← most likely
  → P("gun" | "machine") = 0.10
  → P("room" | "machine") = 0.05
```

---

## Trade-offs

| N-gram Size | Context Captured | Vocabulary Size | Sparsity |
|:-:|---|---|---|
| 1 (unigram) | None | Small | Low |
| 2 (bigram) | 1 word | Medium | Medium |
| 3 (trigram) | 2 words | Large | High |
| 4+ | 3+ words | Very large | Very high |

```
Vocabulary explosion:
  V words in vocabulary
  Possible bigrams:  V²    (e.g., 50K² = 2.5 billion)
  Possible trigrams: V³    (astronomical)
  
  Most n-grams never appear → extreme sparsity
```

---

## Applications

| Application | N-gram Type | Example |
|-------------|-------------|---------|
| Language modeling | Word n-grams | "machine learning" predicts "is" |
| Spelling correction | Char n-grams | "teh" → "the" (similar char trigrams) |
| Language detection | Char n-grams | French has "les", "des" char patterns |
| Text classification | Word bigrams | "not good" captured as negative |
| Search engines | Word n-grams | Phrase matching "New York" |

---

## Revision Questions

1. **What is the key advantage of bigrams over unigrams?**
2. **Why does increasing n-gram size lead to sparsity problems?**
3. **How are character n-grams used for spelling correction?**
4. **Calculate the bigram probability P("learning" | "deep") given counts: "deep learning" = 30, "deep" = 40.**
5. **Why might you use `ngram_range=(1,2)` instead of just `(2,2)` in a classifier?**

---

[Previous: 02-tf-idf.md](02-tf-idf.md) | [Next: 04-one-hot-encoding.md](04-one-hot-encoding.md)
