# What is NLP?

## Overview

Natural Language Processing (NLP) is a field of artificial intelligence that gives machines the ability to read, understand, and derive meaning from human languages. It bridges the gap between human communication (unstructured, ambiguous, context-dependent) and computer understanding (structured, precise, rule-based).

```
┌─────────────────────────────────────────────────────────┐
│                    NLP AT A GLANCE                       │
│                                                         │
│   Human Language          NLP              Machine      │
│  ┌──────────────┐    ┌──────────┐    ┌──────────────┐  │
│  │ "The movie   │───▶│ Process  │───▶│ Sentiment:   │  │
│  │  was great!" │    │ Analyze  │    │ POSITIVE 0.95│  │
│  └──────────────┘    │ Extract  │    └──────────────┘  │
│                      └──────────┘                       │
│                                                         │
│  Unstructured ─────────────────────────▶ Structured     │
│  Ambiguous    ─────────────────────────▶ Precise        │
│  Context-rich ─────────────────────────▶ Computable     │
└─────────────────────────────────────────────────────────┘
```

---

## What Makes NLP Unique?

Unlike other forms of data (images, numbers, sensor readings), natural language has unique characteristics that make it extraordinarily challenging:

### 1. Ambiguity at Every Level

```
Lexical Ambiguity:
  "bank" → financial institution OR river bank?
  "bat"  → animal OR sports equipment?

Syntactic Ambiguity:
  "I saw the man with the telescope"
  → I used a telescope to see the man?
  → I saw a man who had a telescope?

Semantic Ambiguity:
  "Every student passed the exam"
  → Each individual student passed?
  → The students collectively passed?

Pragmatic Ambiguity:
  "Can you pass the salt?"
  → Question about ability? OR request to act?
```

### 2. The Scale of Human Language

| Aspect | Approximate Scale |
|--------|-------------------|
| Words in English | ~170,000 active |
| Languages worldwide | ~7,000 |
| Possible sentences | Infinite (recursive grammar) |
| Wikipedia articles | 60M+ |
| Daily emails sent | 300B+ |
| Web pages indexed | 50B+ |

---

## Subfields of NLP

```
┌──────────────────────────────────────────────────┐
│                  NLP Subfields                    │
│                                                  │
│  ┌──────────────┐  ┌──────────────────────────┐ │
│  │ NLU          │  │ NLG                      │ │
│  │ (Understanding)  │ (Generation)             │ │
│  │              │  │                          │ │
│  │ • Parsing    │  │ • Text summarization     │ │
│  │ • NER       │  │ • Machine translation    │ │
│  │ • Sentiment │  │ • Dialogue systems       │ │
│  │ • QA        │  │ • Story generation       │ │
│  │ • Intent    │  │ • Data-to-text           │ │
│  └──────────────┘  └──────────────────────────┘ │
│                                                  │
│  ┌──────────────────────────────────────────────┐│
│  │ Computational Linguistics                    ││
│  │ • Morphology • Syntax • Semantics • Pragmatics│
│  └──────────────────────────────────────────────┘│
└──────────────────────────────────────────────────┘
```

- **NLU (Natural Language Understanding):** Machine reads and comprehends text → classification, entity extraction, relation extraction
- **NLG (Natural Language Generation):** Machine produces human-readable text → summarization, translation, chatbots
- **Computational Linguistics:** Formal study of language structure and meaning using computational methods

---

## Brief History of NLP

| Era | Period | Approach | Key Milestones |
|-----|--------|----------|----------------|
| Rule-based | 1950s–1980s | Hand-crafted rules, grammars | ELIZA (1966), SHRDLU (1970) |
| Statistical | 1990s–2000s | Probabilistic models, ML | HMMs, n-grams, SVM classifiers |
| Neural | 2013–2017 | Word embeddings, RNNs/LSTMs | Word2Vec, Seq2Seq, attention |
| Pre-trained | 2018–present | Transformers, transfer learning | BERT, GPT, T5, ChatGPT |

```
Evolution of NLP:

Rules ──▶ Statistics ──▶ Deep Learning ──▶ Foundation Models
                                                  │
                                                  ▼
                                        GPT-4, Claude, LLaMA
                                        (General-purpose NLP)
```

---

## Levels of Language Analysis

```
┌────────────────────────────────────────────────────────┐
│ Level            │ Studies            │ Example         │
├──────────────────┼────────────────────┼─────────────────┤
│ Phonology        │ Sound systems      │ /kæt/ → "cat"   │
│ Morphology       │ Word structure     │ un+break+able   │
│ Syntax           │ Sentence structure │ NP + VP + NP    │
│ Semantics        │ Meaning            │ "cat" = animal  │
│ Pragmatics       │ Context & intent   │ "Nice weather"  │
│                  │                    │  (= small talk)  │
│ Discourse        │ Multi-sentence     │ Story coherence  │
└────────────────────────────────────────────────────────┘
```

NLP systems may operate at one or more of these levels depending on the task.

---

## Python: Your First NLP Program

```python
import nltk
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
from collections import Counter

# Download required data
nltk.download('punkt_tab', quiet=True)
nltk.download('stopwords', quiet=True)

text = """Natural Language Processing enables computers to understand 
human language. NLP is used in search engines, virtual assistants, 
and machine translation systems."""

# Tokenize
tokens = word_tokenize(text.lower())
print(f"Tokens: {tokens[:10]}...")

# Remove stopwords
stop_words = set(stopwords.words('english'))
filtered = [w for w in tokens if w.isalpha() and w not in stop_words]
print(f"Filtered: {filtered}")

# Word frequency
freq = Counter(filtered)
print(f"Most common: {freq.most_common(5)}")
```

Output:
```
Tokens: ['natural', 'language', 'processing', 'enables', 'computers', ...]
Filtered: ['natural', 'language', 'processing', 'enables', 'computers', 
           'understand', 'human', 'language', 'nlp', 'used', 'search', 
           'engines', 'virtual', 'assistants', 'machine', 'translation', 'systems']
Most common: [('language', 2), ('natural', 1), ('processing', 1), ...]
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| NLP Definition | AI field for understanding and generating human language |
| NLU vs NLG | Understanding (reading) vs Generation (writing) |
| Key Challenge | Ambiguity at lexical, syntactic, semantic, pragmatic levels |
| Language Levels | Phonology → Morphology → Syntax → Semantics → Pragmatics |
| Modern Approach | Pre-trained transformers with fine-tuning |
| Core Libraries | NLTK, spaCy, HuggingFace Transformers |

---

## Revision Questions

1. **What is the difference between NLU and NLG? Give an example of each.**
2. **Explain lexical ambiguity and syntactic ambiguity with examples.**
3. **What are the main levels of language analysis in NLP?**
4. **How has NLP evolved from rule-based systems to modern approaches?**
5. **Why is natural language harder for computers to process than structured data?**

---

[Next: 02-nlp-applications.md](02-nlp-applications.md) | [Up: README](../README.md)
