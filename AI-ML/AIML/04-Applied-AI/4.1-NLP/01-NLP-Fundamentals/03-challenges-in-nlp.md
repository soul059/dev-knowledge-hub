# Challenges in NLP

## Overview

Natural language is extraordinarily complex — evolved over millennia for human-to-human communication, not for machine parsing. This chapter catalogs the fundamental challenges that make NLP one of the hardest problems in AI.

---

## The Core Challenges

```
┌──────────────────────────────────────────────────────────┐
│                 NLP CHALLENGES HIERARCHY                  │
│                                                          │
│  Language-Inherent          Data & Engineering            │
│  ├─ Ambiguity               ├─ Data scarcity             │
│  ├─ Context dependence       ├─ Noisy text               │
│  ├─ Variability              ├─ Domain adaptation         │
│  ├─ Idioms & metaphors       ├─ Multilingual support      │
│  ├─ Sarcasm & irony          ├─ Evaluation difficulty     │
│  ├─ Negation                 ├─ Bias & fairness           │
│  └─ Coreference              └─ Computational cost        │
└──────────────────────────────────────────────────────────┘
```

---

### 1. Ambiguity

The same text can have multiple valid interpretations.

```
Lexical:    "He went to the bank"
             → river bank? financial bank?

Syntactic:  "Flying planes can be dangerous"
             → The act of flying is dangerous?
             → Planes that are flying are dangerous?

Semantic:   "Every child loves a dog"
             → Each child loves some (possibly different) dog?
             → There is one specific dog all children love?

Referential: "Alice told Bob she was leaving"
             → Who is "she"? Alice or someone else?
```

### 2. Context Dependence

```
Sentence: "It's cold in here."
  Context A (room): → Request to close a window / turn up heat
  Context B (conversation): → The person feels unwelcome
  Context C (food review): → The food temperature is too low

Same words, completely different meanings depending on context.
```

### 3. Variability — Same Meaning, Different Forms

```
"The cat sat on the mat"
"A feline was sitting upon the rug"
"On the mat, there sat a cat"
"The kitty was resting on the carpet"
"Mat. Cat. Sitting." (informal/telegram style)

All express roughly the same idea — but vastly different surface forms.
```

### 4. Sarcasm & Irony

```
Text: "Oh great, another Monday morning meeting. Just what I needed."
  Literal interpretation: POSITIVE ✗
  Actual meaning:         NEGATIVE ✓ (sarcasm)

Text: "The food was so good I couldn't stop not eating it."
  → Requires world knowledge to parse the double negative + sarcasm
```

### 5. Negation

```
"I don't think this is not a bad idea"
  don't + not + bad = ??? 
  → Triple negation: Actually means "it's a good idea"

"Nobody doesn't like ice cream"
  → Everybody likes ice cream (double negative = positive)
```

### 6. Long-Range Dependencies

```
"The trophy didn't fit in the suitcase because it was too [big/small]"

  If "big"   → "it" = trophy
  If "small" → "it" = suitcase

  Resolving "it" requires understanding relationships across the whole sentence.
```

### 7. World Knowledge

```
"He dropped the ball on the glass table and it broke"
  → "it" = glass table (because glass breaks, balls usually don't)
  
Requires physics knowledge that NLP models don't inherently have.
```

### 8. Low-Resource Languages

```
Available NLP Resources by Language:

English     ████████████████████████████ Abundant
Chinese     █████████████████████        Rich
Spanish     ████████████████             Good
Hindi       ████████                     Moderate
Swahili     ███                          Limited
Quechua     █                            Scarce

~7,000 languages, but most NLP research covers <100
```

### 9. Noisy Text

```
Real-world text is messy:
  "omg thats soooo cool 😍😍 #bestday"    → Social media
  "u shld c this asap lol"                 → SMS/chat
  "Recieved teh packge, thnx"              → Typos
  "Patient presents w/ SOB, hx of CHF"     → Medical abbreviations
```

### 10. Bias and Fairness

```python
# Embedding bias example (historical concern)
# Word2Vec trained on biased data:
# "doctor" - "man" + "woman" ≈ "nurse"   ← Gender bias!
# "programmer" - "man" + "woman" ≈ "homemaker" ← Harmful stereotype

# Modern approaches: debiasing techniques, balanced training data,
# evaluation for demographic parity
```

---

## Challenge Impact by Task

| Challenge | Sentiment Analysis | Translation | QA | Chatbots |
|-----------|:-:|:-:|:-:|:-:|
| Ambiguity | Medium | High | High | High |
| Sarcasm | Critical | Low | Low | High |
| Negation | High | Medium | Medium | Medium |
| Context | High | High | Critical | Critical |
| Noise | High | Medium | Low | High |
| Low-resource | Medium | Critical | Medium | Medium |
| Bias | High | Medium | Medium | Critical |

---

## How Modern NLP Addresses These Challenges

| Challenge | Classical Solution | Modern Solution |
|-----------|-------------------|-----------------|
| Ambiguity | Rule-based disambiguation | Contextual embeddings (BERT) |
| Context | Bag-of-words (limited) | Transformers (full context) |
| Variability | Stemming, lemmatization | Learned representations |
| Sarcasm | Feature engineering | Large pre-trained models |
| Low-resource | Translation dictionaries | Multilingual models (mBERT, XLM-R) |
| Bias | Manual debiasing | Fairness-aware training, RLHF |
| Noisy text | Regex cleaning | Subword tokenization (BPE) |

---

## Revision Questions

1. **Explain the four types of ambiguity in NLP with examples.**
2. **Why is sarcasm detection particularly difficult for NLP systems?**
3. **What is the coreference resolution problem? Give an example.**
4. **How do modern transformer models address the context dependence challenge?**
5. **Why is NLP for low-resource languages still a major challenge?**
6. **How can bias in training data affect NLP model outputs?**

---

[Previous: 02-nlp-applications.md](02-nlp-applications.md) | [Next: 04-nlp-pipeline-overview.md](04-nlp-pipeline-overview.md)
