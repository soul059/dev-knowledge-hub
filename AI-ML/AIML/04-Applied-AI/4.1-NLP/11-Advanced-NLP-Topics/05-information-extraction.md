# Information Extraction

## Overview

Information Extraction (IE) automatically identifies structured information from unstructured text. It pulls out entities, relationships, events, and facts, transforming free text into structured data that can populate databases, knowledge graphs, and analytics systems.

---

## IE Pipeline

```
Raw Text → [NER] → [Relation Extraction] → [Event Extraction] → Structured Data

Example:
  Text: "Apple CEO Tim Cook announced the iPhone 15 in Cupertino on 
         September 12, 2023."

  NER:
    Apple          → ORG
    Tim Cook       → PERSON
    iPhone 15      → PRODUCT
    Cupertino      → LOC
    September 12   → DATE

  Relation Extraction:
    (Tim Cook, CEO_of, Apple)
    (iPhone 15, manufactured_by, Apple)
    (announcement, location, Cupertino)

  Event Extraction:
    Event: Product_Launch
    Agent: Tim Cook
    Product: iPhone 15
    Location: Cupertino
    Date: September 12, 2023

  Structured output:
    ┌──────────────────────────────────────┐
    │ event: product_launch                │
    │ company: Apple                       │
    │ person: Tim Cook (role: CEO)         │
    │ product: iPhone 15                   │
    │ location: Cupertino                  │
    │ date: 2023-09-12                     │
    └──────────────────────────────────────┘
```

---

## Named Entity Recognition (Deep Dive)

```
Beyond basic NER — advanced entity types:

Fine-grained NER (18+ types):
  Standard: PER, ORG, LOC, DATE
  Fine-grained: POLITICIAN, COMPANY, CITY, COUNTRY, DISEASE, 
                DRUG, GENE, PROTEIN, CHEMICAL, CURRENCY, ...

Nested NER:
  "[Bank of [America]]"
  America → LOC (nested inside)
  Bank of America → ORG (outer)

Cross-document NER:
  "Apple" in tech article → ORG (Apple Inc.)
  "Apple" in recipe → FOOD (fruit)
  → Context-dependent entity typing
```

---

## Relation Extraction

```
Given entities, determine the relationship between them:

Input: "Marie Curie was born in Warsaw, Poland."
Entities: Marie Curie (PERSON), Warsaw (CITY), Poland (COUNTRY)

Relations:
  (Marie Curie, born_in, Warsaw)     → birth_place
  (Warsaw, located_in, Poland)       → location

Methods:
  1. Pattern-based:
     "X was born in Y" → born_in(X, Y)
     Simple but limited coverage

  2. Supervised classification:
     Given (entity1, entity2, context) → predict relation type
     Requires labeled data

  3. Distant supervision:
     Use existing KB facts to auto-label training data
     (Marie Curie, born_in, Warsaw) in Wikidata
     → Any sentence with both "Marie Curie" and "Warsaw" 
        is a potential training example for born_in
```

---

## Implementation

```python
import spacy
from transformers import pipeline

# NER with spaCy
nlp = spacy.load("en_core_web_trf")
text = "Google CEO Sundar Pichai announced Gemini AI at their Mountain View headquarters in December 2023."
doc = nlp(text)

print("Entities:")
for ent in doc.ents:
    print(f"  {ent.text:30s} → {ent.label_}")
# Google                         → ORG
# Sundar Pichai                  → PERSON
# Gemini AI                     → PRODUCT
# Mountain View                 → GPE
# December 2023                 → DATE

# Relation Extraction with transformers
re_pipeline = pipeline("text2text-generation", model="Babelscape/rebel-large")

result = re_pipeline(text)
print("\nRelations:")
print(result[0]["generated_text"])
# <triplet> Sundar Pichai <subj> Google <obj> chief executive officer
# <triplet> Gemini AI <subj> Google <obj> developer
```

---

## Open Information Extraction (OpenIE)

```
Extract relations without predefined schema:

Input: "Einstein developed the theory of relativity in 1905."

OpenIE output:
  (Einstein, developed, the theory of relativity)
  (Einstein, developed ... in, 1905)

No predefined relation types — extracts any (subject, predicate, object)

Tools:
  - Stanford OpenIE
  - AllenNLP OpenIE
  - OpenIE 5.0

Advantages: No schema needed, covers all relations
Disadvantages: Noisy, hard to normalize
```

---

## Knowledge Graph Construction

```
IE feeds into Knowledge Graph construction:

  Documents → [NER] → [Coref Resolution] → [Relation Extraction]
                                                    ↓
                                          Knowledge Graph
                                          
  ┌─────────────────────────────────────────────┐
  │              Knowledge Graph                 │
  │                                             │
  │  Sundar_Pichai ──CEO_of──→ Google           │
  │       │                      │              │
  │       └──announced──→ Gemini_AI             │
  │                          │                  │
  │  Google ──HQ_in──→ Mountain_View            │
  │       │                                     │
  │       └──type──→ Technology_Company         │
  └─────────────────────────────────────────────┘
```

---

## IE Tasks Comparison

| Task | Input | Output | Example |
|------|-------|--------|---------|
| NER | Text | Entity spans + types | "Google" → ORG |
| Relation Extraction | Text + entities | (entity, relation, entity) | (Pichai, CEO_of, Google) |
| Event Extraction | Text | Event type + arguments | Launch(agent=Pichai, product=Gemini) |
| Temporal IE | Text | Time expressions, timelines | "Dec 2023" → 2023-12 |
| OpenIE | Text | (subject, predicate, object) | (Einstein, developed, relativity) |
| Slot Filling | Text + query | Missing KB attributes | birthdate(Pichai) = ? |

---

## Revision Questions

1. **What are the main components of an IE pipeline?**
2. **How does relation extraction differ from NER?**
3. **What is distant supervision and how is it used for relation extraction?**
4. **What is the difference between closed IE and open IE?**
5. **How does IE contribute to knowledge graph construction?**

---

[Previous: 04-topic-modeling.md](04-topic-modeling.md) | [Next: 06-low-resource-nlp.md](06-low-resource-nlp.md)
