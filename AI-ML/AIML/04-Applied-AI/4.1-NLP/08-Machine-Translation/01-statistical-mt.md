# Statistical Machine Translation (SMT) Overview

## Overview

Statistical Machine Translation uses probabilistic models to translate text between languages. Dominant from the 1990s to ~2015, SMT decomposes translation into a **language model** (fluency) and a **translation model** (adequacy), combined via the noisy channel framework.

---

## Noisy Channel Model

```
Goal: Find best English translation ê given French input f

ê = argmax_e P(e|f)
  = argmax_e P(f|e) × P(e)       ← Bayes' theorem
                ↑         ↑
         Translation    Language
           Model         Model
         (adequacy)    (fluency)
```

```
French:  "La maison est grande"
                │
    ┌───────────┴──────────┐
    │  Translation Model   │  P(f|e): word alignment probabilities
    │  "maison" → "house"  │
    │  "grande" → "big"    │
    └───────────┬──────────┘
                │
    ┌───────────┴──────────┐
    │   Language Model     │  P(e): "The house is big" >> "big house is the"
    └───────────┬──────────┘
                │
English: "The house is big"
```

---

## Phrase-Based SMT

```
Instead of word-by-word, translate phrases:

French:  "Il ne va pas à la maison"
Phrases: [Il ne va pas] [à] [la maison]
             ↓           ↓      ↓
         [He does not go] [to] [the house]

Phrase table (learned from parallel corpus):
  "la maison"    → "the house"     P=0.7
  "la maison"    → "home"          P=0.2
  "ne ... pas"   → "not"           P=0.8
```

---

## Components

| Component | Role | How It's Trained |
|-----------|------|-----------------|
| Language Model | Ensure fluent output | Monolingual target corpus (n-gram) |
| Translation Model | Map source → target phrases | Parallel corpus + word alignment |
| Reordering Model | Handle word order differences | Learned from aligned data |
| Decoder | Search for best translation | Beam search over hypotheses |

---

## Word Alignment

```
Alignment: mapping words between source and target

English:  The    house   is    big
           │  ╲    │      │     │
French:   La  maison  est  grande

Learned with EM algorithm (IBM Models 1-5, GIZA++)
```

---

## Why SMT Was Replaced

| Limitation | Description |
|-----------|-------------|
| Complex pipeline | Many separate components to train |
| Limited context | Phrase tables have fixed window |
| Poor long-range reordering | Struggles with distant word movements |
| Language-pair specific | Need separate model per language pair |
| Requires parallel corpora | Large aligned datasets needed |

Neural MT addressed all of these with end-to-end learning.

---

## Revision Questions

1. **Explain the noisy channel model for machine translation.**
2. **What is the difference between word-based and phrase-based SMT?**
3. **What role does the language model play in SMT?**
4. **What is word alignment and why is it important?**
5. **Why did neural MT largely replace statistical MT?**

---

[Previous: Unit 7 - 05-transfer-learning.md](../07-Language-Models/05-transfer-learning.md) | [Next: 02-neural-mt.md](02-neural-mt.md)
