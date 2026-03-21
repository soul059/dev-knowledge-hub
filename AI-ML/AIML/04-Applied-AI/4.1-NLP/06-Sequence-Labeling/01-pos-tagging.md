# POS Tagging (Part-of-Speech Tagging)

## Overview

POS tagging assigns a grammatical label (noun, verb, adjective, etc.) to each word in a sentence. It's one of the oldest and most fundamental NLP tasks — a prerequisite for parsing, NER, and many other tasks.

---

## Common POS Tags (Penn Treebank)

```
Tag   Description          Example
────────────────────────────────────
NN    Noun, singular       cat, table
NNS   Noun, plural         cats, tables
NNP   Proper noun          London, Alice
VB    Verb, base form      run, eat
VBD   Verb, past tense     ran, ate
VBG   Verb, gerund         running, eating
JJ    Adjective            happy, big
RB    Adverb               quickly, very
DT    Determiner           the, a, an
IN    Preposition          in, on, at
PRP   Personal pronoun     I, you, he
CC    Coordinating conj.   and, but, or
```

---

## Implementation

```python
import nltk
from nltk import pos_tag, word_tokenize
nltk.download('averaged_perceptron_tagger_eng', quiet=True)

text = "The quick brown fox jumps over the lazy dog"
tokens = word_tokenize(text)
tagged = pos_tag(tokens)

for word, tag in tagged:
    print(f"  {word:10s} → {tag}")
# The        → DT
# quick      → JJ
# brown      → NN
# fox        → NN
# jumps      → VBZ
# over       → IN
# the        → DT
# lazy       → JJ
# dog        → NN
```

### spaCy POS Tagging

```python
import spacy
nlp = spacy.load("en_core_web_sm")

doc = nlp("She sells seashells by the seashore")
for token in doc:
    print(f"  {token.text:12s} POS: {token.pos_:6s} Tag: {token.tag_}")
# She          POS: PRON   Tag: PRP
# sells        POS: VERB   Tag: VBZ
# seashells    POS: NOUN   Tag: NNS
# by           POS: ADP    Tag: IN
# the          POS: DET    Tag: DT
# seashore     POS: NOUN   Tag: NN
```

---

## POS Tagging Approaches

| Approach | Method | Accuracy |
|----------|--------|:--------:|
| Rule-based | Hand-crafted rules | ~77% |
| HMM | Hidden Markov Model | ~95% |
| Perceptron | Averaged perceptron | ~97% |
| BiLSTM | Neural sequence labeling | ~97.5% |
| BERT | Fine-tuned transformer | ~98%+ |

---

## Why POS Tagging Matters

```
Disambiguation:
  "I can fish"    → fish is VERB (I am able to catch fish)
  "I eat fish"    → fish is NOUN (fish is the food)

  POS helps disambiguate word meaning!

Downstream tasks:
  • Parsing (sentence structure)
  • NER (proper nouns → entities)
  • Lemmatization (need POS for correct lemma)
  • Information extraction (verb phrases → events)
```

---

## Revision Questions

1. **What is POS tagging and why is it useful for downstream NLP tasks?**
2. **What is the difference between NN, NNS, and NNP tags?**
3. **How does POS tagging help with word sense disambiguation?**
4. **Compare rule-based vs neural approaches for POS tagging.**
5. **How does spaCy differentiate between `pos_` and `tag_`?**

---

[Previous: Unit 5 - 07-evaluation-metrics.md](../05-Text-Classification/07-evaluation-metrics.md) | [Next: 02-named-entity-recognition.md](02-named-entity-recognition.md)
