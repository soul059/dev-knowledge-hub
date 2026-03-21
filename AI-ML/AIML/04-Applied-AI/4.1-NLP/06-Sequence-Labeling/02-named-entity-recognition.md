# Named Entity Recognition (NER)

## Overview

Named Entity Recognition identifies and classifies named entities in text into predefined categories: person names, organizations, locations, dates, monetary values, etc. NER is essential for information extraction, question answering, and knowledge graph construction.

---

## Standard Entity Types

```
Sentence: "Apple CEO Tim Cook announced a $2B investment in Austin on March 5th"

Entities:
  Apple       → ORG    (Organization)
  Tim Cook    → PER    (Person)
  $2B         → MONEY  (Monetary value)
  Austin      → LOC    (Location)
  March 5th   → DATE   (Date/Time)
```

| Entity Type | Tag | Examples |
|------------|-----|---------|
| Person | PER | Tim Cook, Marie Curie |
| Organization | ORG | Google, United Nations |
| Location | LOC | Paris, Mount Everest |
| Date/Time | DATE | March 2024, yesterday |
| Money | MONEY | $50, €1 million |
| Percent | PERCENT | 15%, fifty percent |
| Product | PRODUCT | iPhone, Tesla Model 3 |
| Event | EVENT | World Cup, COVID-19 |

---

## Implementation

### spaCy NER

```python
import spacy
nlp = spacy.load("en_core_web_sm")

text = "Barack Obama was born in Honolulu, Hawaii on August 4, 1961"
doc = nlp(text)

for ent in doc.ents:
    print(f"  {ent.text:20s}  {ent.label_:10s}  ({ent.start_char}-{ent.end_char})")
# Barack Obama          PERSON      (0-12)
# Honolulu              GPE         (25-33)
# Hawaii                GPE         (35-41)
# August 4, 1961        DATE        (45-60)
```

### HuggingFace Transformers NER

```python
from transformers import pipeline

ner = pipeline("ner", grouped_entities=True)
text = "Microsoft CEO Satya Nadella visited Berlin last week"
entities = ner(text)

for ent in entities:
    print(f"  {ent['word']:20s}  {ent['entity_group']:5s}  ({ent['score']:.3f})")
# Microsoft             ORG    (0.999)
# Satya Nadella         PER    (0.998)
# Berlin                LOC    (0.999)
```

---

## NER Challenges

```
Ambiguity:
  "Apple" → ORG (Apple Inc.) or fruit?
  "Washington" → PER (George Washington) or LOC (Washington D.C.)?
  "Jordan" → PER (Michael Jordan) or LOC (country)?
  
  Context is crucial for disambiguation.

Nested entities:
  "New York University" → LOC (New York) inside ORG (NYU)?
  
Emerging entities:
  New companies, people, products not in training data.
```

---

## Evaluation

```python
from seqeval.metrics import classification_report

y_true = [['O', 'O', 'B-PER', 'I-PER', 'O', 'B-LOC']]
y_pred = [['O', 'O', 'B-PER', 'I-PER', 'O', 'O']]

print(classification_report(y_true, y_pred))
#           precision  recall  f1-score  support
# LOC          0.00    0.00      0.00        1
# PER          1.00    1.00      1.00        1
```

NER uses **entity-level** evaluation: the entire span must be correct (both boundaries and type).

---

## Revision Questions

1. **What are the standard entity types in NER?**
2. **Why is "Apple" ambiguous for NER systems?**
3. **What makes entity-level evaluation stricter than token-level?**
4. **How do transformer-based NER models outperform traditional approaches?**
5. **What is the challenge of nested entities?**

---

[Previous: 01-pos-tagging.md](01-pos-tagging.md) | [Next: 03-chunking.md](03-chunking.md)
