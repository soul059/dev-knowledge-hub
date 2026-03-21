# Text vs Language

## Overview

"Text" and "language" are often used interchangeably, but they are fundamentally different concepts. Understanding this distinction is crucial for NLP practitioners — text is the written artifact, while language encompasses the full system of communication including grammar, meaning, context, and intent.

---

## Key Distinctions

```
┌────────────────────────────────────┬────────────────────────────────────┐
│            TEXT                     │            LANGUAGE                │
│                                    │                                    │
│  • Written/typed symbols           │  • Complete communication system   │
│  • Static artifact                 │  • Dynamic, evolving rules         │
│  • Observable surface form         │  • Underlying structure & meaning  │
│  • Can be processed directly       │  • Requires understanding          │
│  • Finite string of characters     │  • Infinite generative capacity    │
│                                    │                                    │
│  Example:                          │  Example:                          │
│  "I saw her duck"                  │  Grammar, semantics, pragmatics    │
│  (just characters)                 │  that explain WHY this is          │
│                                    │  ambiguous (verb vs noun)          │
└────────────────────────────────────┴────────────────────────────────────┘
```

---

## Modalities of Language

Language is expressed through multiple modalities — NLP traditionally focuses on text, but modern systems increasingly handle others:

```
┌────────────────────────────────────────────────────────┐
│              LANGUAGE MODALITIES                        │
│                                                        │
│  Spoken     ──▶ Speech Recognition (ASR) ──▶ Text     │
│  Written    ──▶ OCR / Direct Input ──────▶ Text      │
│  Signed     ──▶ Computer Vision ─────────▶ Text      │
│  Gestural   ──▶ Multimodal Processing ───▶ Text      │
│                                                        │
│  All modalities → eventually processed as text in NLP  │
└────────────────────────────────────────────────────────┘
```

---

## What Text Captures (and What It Doesn't)

| Information Type | Captured in Text? | Example |
|-----------------|:-:|---------|
| Words & sentences | ✅ | "Hello, how are you?" |
| Punctuation | ✅ | "Stop!" vs "Stop." |
| Tone of voice | ❌ | Angry "fine" vs happy "fine" |
| Facial expression | ❌ | Smiling while saying something |
| Gestures | ❌ | Pointing, shrugging |
| Emphasis/stress | Partially | *italic*, **bold**, CAPS |
| Sarcasm | ❌ (usually) | "Oh, fantastic" (sincere or sarcastic?) |
| Shared context | ❌ | "That thing we discussed" |

---

## Formal vs Informal Text

```
Formal Text (structured):
┌─────────────────────────────────────────────┐
│ "The quarterly revenue increased by 15%     │
│  compared to the previous fiscal year."     │
│                                             │
│ → Well-formed grammar                       │
│ → Standard vocabulary                       │
│ → Complete sentences                        │
└─────────────────────────────────────────────┘

Informal Text (noisy):
┌─────────────────────────────────────────────┐
│ "omg revenue up 15%!! 🚀🚀 lets gooo"      │
│                                             │
│ → Abbreviations, slang                      │
│ → Emojis as meaning carriers                │
│ → No formal grammar                         │
│ → Same information, different form          │
└─────────────────────────────────────────────┘

NLP must handle BOTH.
```

---

## Language Properties Relevant to NLP

### Compositionality
Meaning of a whole is derived from meaning of parts:
```
"red" + "car" → red car (compositional ✓)
"kick" + "bucket" → die (non-compositional ✗ — idiom)
```

### Recursion
Language can embed structures within structures infinitely:
```
"The cat sat on the mat"
"The cat [that the dog chased] sat on the mat"
"The cat [that the dog [that the boy owned] chased] sat on the mat"
→ Grammatically valid but increasingly hard to parse
```

### Zipf's Law
Word frequency follows a power law — a few words are extremely common, most are rare:
```python
from collections import Counter
import matplotlib.pyplot as plt

text = "the cat sat on the mat the cat is on the mat again"
freq = Counter(text.split())
# 'the': 4, 'cat': 2, 'mat': 2, 'on': 2, 'sat': 1, 'is': 1, 'again': 1

# Rank 1 word appears ~2x as often as rank 2, ~3x as rank 3, etc.
# This affects vocabulary size, embedding coverage, and model training
```

---

## Implications for NLP System Design

| Language Property | NLP Design Implication |
|-------------------|----------------------|
| Text is just surface form | Need deeper analysis beyond character matching |
| Multiple modalities | Pipeline must handle ASR, OCR before NLP |
| Formal vs informal | Preprocessing must adapt to text type |
| Compositionality breaks | Idiom/phrase detection needed |
| Zipf's law | Most words are rare — need subword/embedding approaches |
| Recursion | Need models that handle nested structures (trees, transformers) |
| Missing context | Conversational models need dialogue history |

---

## Revision Questions

1. **What is the fundamental difference between "text" and "language"?**
2. **What information present in spoken language is lost in text?**
3. **Why does Zipf's law create challenges for NLP vocabulary handling?**
4. **Explain compositionality and give an example where it breaks down.**
5. **How should NLP systems differ when processing formal vs informal text?**

---

[Previous: 04-nlp-pipeline-overview.md](04-nlp-pipeline-overview.md) | [Next: Unit 2 - 01-tokenization.md](../02-Text-Preprocessing/01-tokenization.md)
