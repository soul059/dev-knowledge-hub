# Face Verification vs Face Identification

## Overview

Face recognition systems operate in two fundamentally different modes: **verification** (1:1 matching — "Is this person who they claim to be?") and **identification** (1:N search — "Who is this person?"). Understanding the distinction is critical for system design, evaluation, and deployment.

---

## Verification (1:1)

```
"Is this the same person?"

  Claimed identity: "I am Alice"
  
  Probe face              Gallery face (Alice's enrolled photo)
  ┌──────────┐            ┌──────────┐
  │          │            │          │
  │   🧑    │     ?      │   🧑    │
  │          │    ═══     │          │
  └──────────┘            └──────────┘
       │                       │
  ┌────▼────┐            ┌────▼────┐
  │  CNN    │            │  CNN    │
  └────┬────┘            └────┬────┘
       │                       │
    f_probe                 f_gallery
       │                       │
       └───────┬───────────────┘
               │
        similarity(f_probe, f_gallery)
               │
          ┌────▼────┐
          │  > τ ?  │    τ = threshold
          └────┬────┘
          ┌────┴────┐
       Accept    Reject
    (Same person) (Different)

  Examples:
    Phone unlock (compare with stored face)
    Airport e-gates (compare with passport photo)
    Banking KYC (compare selfie with ID photo)

  Metrics:
    TAR (True Accept Rate) = TP / (TP + FN)
    FAR (False Accept Rate) = FP / (FP + TN)
    
    ROC curve: TAR vs FAR at different thresholds
    
    Key operating points:
      TAR @ FAR=1e-3  (1 in 1000 imposters accepted)
      TAR @ FAR=1e-4  (1 in 10000 — high security)
      TAR @ FAR=1e-6  (1 in million — ultra-high security)
```

---

## Identification (1:N)

```
"Who is this person?"

  Probe face              Gallery database (N enrolled identities)
  ┌──────────┐            ┌──────────┐ ┌──────────┐ ┌──────────┐
  │          │            │  Alice   │ │  Bob     │ │  Carol   │
  │   🧑    │     →      │  f_1     │ │  f_2     │ │  f_3     │
  │          │            └──────────┘ └──────────┘ └──────────┘
  └──────────┘                  ...    (N identities)
       │
  ┌────▼────┐
  │  CNN    │ → f_probe
  └────┬────┘
       │
  Compare f_probe with ALL gallery embeddings:
    sim_1 = cos(f_probe, f_1) = 0.85   ← highest!
    sim_2 = cos(f_probe, f_2) = 0.23
    sim_3 = cos(f_probe, f_3) = 0.31
    ...
    
  Closed-set: Must be one of N identities
    → Return argmax(sim_i) = Alice
    
  Open-set:  Might NOT be in database
    → If max(sim) < threshold → "Unknown"
    → Else → return identity with highest similarity

  Examples:
    Surveillance (find suspect in camera feed)
    Photo tagging (who is in this group photo?)
    Attendance systems (identify each person entering)
```

---

## Closed-Set vs Open-Set

```
Closed-set identification:
  Assumption: probe IS in the gallery
  Metric: Rank-1 accuracy (is top match correct?)
  
  Easier — always returns an answer

Open-set identification:
  Assumption: probe MAY NOT be in the gallery
  Must handle "unknown" / "not enrolled" case
  
  Harder — two decisions:
    1. Is this person enrolled? (detection)
    2. If yes, who are they? (identification)
  
  Metrics:
    DIR (Detection and Identification Rate)
      = Correctly identified / Total enrolled probes
    FPIR (False Positive Identification Rate)
      = Non-enrolled probes incorrectly identified / Total non-enrolled

  ┌────────────────────────────────────────┐
  │           Open-Set Decision            │
  │                                        │
  │  Probe → Compute max similarity        │
  │           │                            │
  │      ┌────▼────┐                       │
  │      │max > τ? │                       │
  │      └────┬────┘                       │
  │      Yes  │  No                        │
  │      ┌────┴────┐                       │
  │  ID = argmax  "Unknown"                │
  └────────────────────────────────────────┘
```

---

## Threshold Selection

```
The threshold τ controls the trade-off:

  Low threshold (τ = 0.3):
    → Accepts more matches → Higher TAR
    → Also accepts more impostors → Higher FAR
    → Use for: convenience-focused (phone unlock)

  High threshold (τ = 0.7):
    → Rejects more matches → Lower TAR
    → Also rejects more impostors → Lower FAR
    → Use for: security-focused (border control)

  TAR
  100% ┤─────────────●
       │          ●
   90% ┤       ●
       │     ●
   80% ┤   ●
       │  ●
   70% ┤●
       │
   60% ┤
       ┼───┬───┬───┬───┬───
       1e-6 1e-5 1e-4 1e-3 1e-2   FAR

  EER (Equal Error Rate):
    Threshold where FAR = FRR (False Reject Rate)
    Single number summarizing system performance
    Lower EER = better system
```

---

## Scalability for Identification

```
Challenge: Comparing probe with millions of gallery faces

  Brute force: O(N) comparisons per probe → too slow for large N

  Solutions:
  
  1. Approximate Nearest Neighbor (ANN):
     FAISS (Facebook): GPU-accelerated similarity search
     - IVF (Inverted File Index): partition space into clusters
     - PQ (Product Quantization): compress vectors
     - 1 billion faces searchable in milliseconds!
  
  2. Hierarchical search:
     First narrow by coarse features (age, gender)
     Then fine search within subset
  
  3. Hashing:
     Locality-Sensitive Hashing (LSH)
     Map similar vectors to same hash bucket

  FAISS example:
    1M gallery faces, 512-D embeddings
    Index build: ~30 seconds
    Query time: ~1 ms per probe (vs 100ms brute force)
```

---

## Python: Verification and Identification

```python
import numpy as np

class FaceSystem:
    def __init__(self, threshold=0.5):
        self.threshold = threshold
        self.gallery = {}  # name → embedding
    
    def enroll(self, name, embedding):
        self.gallery[name] = embedding / np.linalg.norm(embedding)
    
    def verify(self, probe_embedding, claimed_identity):
        """1:1 verification"""
        probe = probe_embedding / np.linalg.norm(probe_embedding)
        gallery = self.gallery[claimed_identity]
        similarity = np.dot(probe, gallery)
        return similarity > self.threshold, similarity
    
    def identify(self, probe_embedding, open_set=True):
        """1:N identification"""
        probe = probe_embedding / np.linalg.norm(probe_embedding)
        
        best_match = None
        best_sim = -1
        
        for name, gallery_emb in self.gallery.items():
            sim = np.dot(probe, gallery_emb)
            if sim > best_sim:
                best_sim = sim
                best_match = name
        
        if open_set and best_sim < self.threshold:
            return "Unknown", best_sim
        
        return best_match, best_sim

# Usage
system = FaceSystem(threshold=0.5)
system.enroll("Alice", alice_embedding)
system.enroll("Bob", bob_embedding)

# Verification
match, score = system.verify(probe, "Alice")
print(f"Verification: {match}, Score: {score:.3f}")

# Identification
identity, score = system.identify(probe)
print(f"Identified as: {identity}, Score: {score:.3f}")
```

---

## Summary

| Aspect | Verification (1:1) | Identification (1:N) |
|--------|:---:|:---:|
| Question | "Is this person X?" | "Who is this person?" |
| Comparisons | 1 | N |
| Speed | O(1) | O(N) or O(log N) with ANN |
| Threshold | Required | Required (open-set) |
| Primary metric | TAR@FAR | Rank-1, DIR |
| Difficulty | Easier | Harder (grows with N) |
| Use case | Access control, KYC | Surveillance, tagging |

---

## Revision Questions

1. **What is the fundamental difference between verification and identification?**
2. **What do TAR and FAR measure and how does the threshold affect them?**
3. **How does open-set identification differ from closed-set?**
4. **Why is scalability a challenge for identification but not verification?**
5. **How does FAISS enable fast search over millions of faces?**

---

[Previous: 03-face-recognition.md](03-face-recognition.md) | [Next: 05-emotion-recognition.md](05-emotion-recognition.md)
