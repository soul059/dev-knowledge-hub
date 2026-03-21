# BIO Tagging Scheme

## Overview

BIO (Beginning-Inside-Outside) tagging converts sequence labeling tasks (NER, chunking) into a per-token classification problem. Each token gets a tag indicating whether it **B**egins an entity, is **I**nside an entity, or is **O**utside any entity. This scheme enables neural models to handle multi-word entities cleanly.

---

## The BIO Format

```
Sentence: "Barack Obama visited New York City yesterday"

Token        BIO Tag
──────────────────────
Barack       B-PER       ← Begins a Person entity
Obama        I-PER       ← Inside the Person entity
visited      O           ← Outside any entity
New          B-LOC       ← Begins a Location entity
York         I-LOC       ← Inside Location
City         I-LOC       ← Inside Location
yesterday    B-DATE      ← Begins a Date entity
```

---

## BIO Variants

### IOB2 (most common — same as BIO)
```
B-X: Beginning of entity X
I-X: Inside entity X (continuation)
O:   Outside
```

### BIOES / BILOU (more expressive)
```
B-X: Beginning of multi-token entity
I-X: Inside multi-token entity
E-X: End of multi-token entity        ← new
S-X: Single-token entity              ← new
O:   Outside

Example:
"Barack Obama met Tim in NYC"
Barack  B-PER
Obama   E-PER     ← End of person
met     O
Tim     S-PER     ← Single-token person entity
in      O
NYC     S-LOC     ← Single-token location
```

---

## Why BIO Matters

```
Without BIO:
  "Barack Obama met Michelle Obama"
  PER PER O PER PER

  → Is this one PER entity "Barack Obama met Michelle Obama"?
  → Or two: "Barack Obama" and "Michelle Obama"?
  → Can't tell!

With BIO:
  Barack    B-PER  ← new entity starts
  Obama     I-PER
  met       O
  Michelle  B-PER  ← another entity starts
  Obama     I-PER

  → Clearly two separate entities ✓
```

---

## Working with BIO Tags

```python
def bio_to_entities(tokens, tags):
    """Convert BIO-tagged tokens to entity spans."""
    entities = []
    current_entity = None
    
    for i, (token, tag) in enumerate(zip(tokens, tags)):
        if tag.startswith('B-'):
            if current_entity:
                entities.append(current_entity)
            current_entity = {
                'type': tag[2:],
                'tokens': [token],
                'start': i
            }
        elif tag.startswith('I-') and current_entity:
            current_entity['tokens'].append(token)
        else:
            if current_entity:
                entities.append(current_entity)
                current_entity = None
    
    if current_entity:
        entities.append(current_entity)
    
    return entities

tokens = ["Barack", "Obama", "visited", "New", "York"]
tags = ["B-PER", "I-PER", "O", "B-LOC", "I-LOC"]
print(bio_to_entities(tokens, tags))
# [{'type': 'PER', 'tokens': ['Barack', 'Obama'], 'start': 0},
#  {'type': 'LOC', 'tokens': ['New', 'York'], 'start': 3}]
```

---

## BIO Scheme Comparison

| Scheme | Tags | Distinguishes Entity Boundaries | Single-Token Entities |
|--------|:---:|:---:|:---:|
| IO | I-X, O | ❌ | ❌ |
| BIO / IOB2 | B-X, I-X, O | ✅ | Via B-X only |
| BIOES / BILOU | B, I, E, S, O | ✅ | ✅ Explicit |

---

## Revision Questions

1. **Why is BIO tagging needed instead of simply labeling each token with an entity type?**
2. **What is the difference between BIO and BIOES schemes?**
3. **Given BIO tags, how would you extract multi-word entities?**
4. **Why is B-tag important when two entities of the same type appear consecutively?**
5. **Which BIO variant is most commonly used in modern NER systems?**

---

[Previous: 03-chunking.md](03-chunking.md) | [Next: 05-crf.md](05-crf.md)
