# NLP Applications

## Overview

NLP powers many technologies we use daily — from search engines and virtual assistants to spam filters and autocomplete. This chapter surveys the major application areas, showing how NLP techniques map to real-world products and services.

---

## Application Landscape

```
┌────────────────────────────────────────────────────────────────┐
│                    NLP APPLICATION AREAS                        │
│                                                                │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────┐   │
│  │ Information   │  │ Text         │  │ Conversational    │   │
│  │ Extraction    │  │ Analytics    │  │ AI                │   │
│  │ • NER         │  │ • Sentiment  │  │ • Chatbots        │   │
│  │ • Relation    │  │ • Topic      │  │ • Virtual assist. │   │
│  │ • Event       │  │ • Classify   │  │ • Customer svc    │   │
│  └──────────────┘  └──────────────┘  └───────────────────┘   │
│                                                                │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────┐   │
│  │ Machine       │  │ Content      │  │ Search &          │   │
│  │ Translation   │  │ Generation   │  │ Retrieval         │   │
│  │ • Google      │  │ • Summaries  │  │ • Web search      │   │
│  │   Translate   │  │ • Reports    │  │ • Semantic search  │   │
│  │ • Real-time   │  │ • Creative   │  │ • Document QA     │   │
│  └──────────────┘  └──────────────┘  └───────────────────┘   │
└────────────────────────────────────────────────────────────────┘
```

---

## Major Applications in Detail

### 1. Search Engines

Search engines use NLP at every stage: query understanding, document indexing, ranking, and snippet generation.

```
User Query: "best pizza near me"
    │
    ├─ Intent Detection → Local restaurant search
    ├─ Entity Extraction → food_type: pizza
    ├─ Query Expansion → "pizza restaurant nearby"
    ├─ Semantic Matching → Match against restaurant pages
    └─ Snippet Generation → Summarize relevant content
```

### 2. Virtual Assistants (Siri, Alexa, Google Assistant)

```
"Set an alarm for 7 AM tomorrow"
    │
    ├─ Speech-to-Text (ASR) → raw text
    ├─ Intent Classification → set_alarm
    ├─ Slot Filling → time: 7:00 AM, date: tomorrow
    └─ Action Execution → Create alarm
```

### 3. Sentiment Analysis

```python
from transformers import pipeline

sentiment = pipeline("sentiment-analysis")

reviews = [
    "This product is absolutely amazing!",
    "Terrible quality, broke after one day.",
    "It's okay, nothing special."
]

for review in reviews:
    result = sentiment(review)[0]
    print(f"{result['label']:>8} ({result['score']:.3f}): {review}")
```
```
POSITIVE (0.999): This product is absolutely amazing!
NEGATIVE (0.999): Terrible quality, broke after one day.
NEGATIVE (0.997): It's okay, nothing special.
```

### 4. Machine Translation

| System | Approach | Example |
|--------|----------|---------|
| Google Translate | Transformer-based NMT | 130+ languages |
| DeepL | Custom transformer | Higher quality for European langs |
| Meta NLLB | Multilingual model | 200 languages including low-resource |

### 5. Text Summarization

- **Extractive:** Select important sentences from the original text
- **Abstractive:** Generate new sentences that capture the key ideas

### 6. Spam Detection & Content Moderation

```
Email: "Congratulations! You won $1,000,000!!!"
    │
    ├─ Text Features → exclamation marks, urgency words, monetary values
    ├─ Classification → SPAM (0.99)
    └─ Action → Move to spam folder
```

### 7. Healthcare NLP

- Clinical note extraction (symptoms, medications, diagnoses)
- Medical literature mining
- Patient record de-identification

### 8. Legal & Finance

- Contract analysis and clause extraction
- Financial news sentiment for trading
- Regulatory compliance checking

---

## Application Comparison Table

| Application | NLP Tasks Used | Key Challenge | Industry Impact |
|------------|---------------|---------------|-----------------|
| Search Engines | Intent detection, semantic matching | Ambiguity, scale | $200B+ market |
| Virtual Assistants | ASR, intent, slot filling | Context, noise | Consumer tech |
| Sentiment Analysis | Classification, aspect extraction | Sarcasm, negation | Marketing, finance |
| Machine Translation | Seq2seq, attention | Low-resource langs | Global communication |
| Summarization | Extraction/abstraction | Faithfulness | Media, research |
| Chatbots | Dialogue management, NLG | Coherence, safety | Customer service |
| Spam Detection | Text classification | Adversarial evasion | Email, social media |
| Healthcare NLP | NER, relation extraction | Domain expertise | Clinical decision support |

---

## Python: Quick Application Demo

```python
from transformers import pipeline

# Named Entity Recognition
ner = pipeline("ner", grouped_entities=True)
text = "Apple CEO Tim Cook announced new products in Cupertino, California."
entities = ner(text)
for ent in entities:
    print(f"  {ent['entity_group']:>5}: {ent['word']} ({ent['score']:.3f})")
# Output:
#   ORG: Apple (0.998)
#   PER: Tim Cook (0.999)
#   LOC: Cupertino, California (0.997)
```

---

## Revision Questions

1. **Name five major NLP applications and the core NLP task each relies on.**
2. **How does a virtual assistant use NLP to process a voice command?**
3. **What is the difference between extractive and abstractive summarization?**
4. **Why is sentiment analysis challenging for sarcastic text?**
5. **How is NLP used in healthcare applications?**

---

[Previous: 01-what-is-nlp.md](01-what-is-nlp.md) | [Next: 03-challenges-in-nlp.md](03-challenges-in-nlp.md)
