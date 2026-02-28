# Chapter 4: Reservoir Sampling

[← Previous: Random Algorithms](03-random-algorithms.md) | [Back to README](../README.md) | [Next: Shuffle Algorithm →](05-shuffle-algorithm.md)

---

## Overview

**Reservoir sampling** selects $k$ items uniformly at random from a stream of unknown (or very large) length $n$, using only $O(k)$ memory. You never need to know $n$ in advance.

---

## 4.1 Problem Statement

```
  ┌──────────────────────────────────────────────────────────┐
  │  Given: A stream of elements arriving one at a time      │
  │  Goal:  Maintain a sample of k elements such that        │
  │         every element has equal probability k/n of       │
  │         being in the sample (where n = total seen)       │
  │  Constraint: O(k) memory, one pass, don't know n        │
  └──────────────────────────────────────────────────────────┘
```

---

## 4.2 Algorithm R (Vitter, 1985)

### Case k = 1 (Single Element)

```
FUNCTION reservoirSample1(stream):
    result = stream[0]
    FOR i = 1 TO end of stream:
        j = random integer in [0, i]
        IF j == 0:
            result = stream[i]
    RETURN result

  At step i (0-indexed), P(replace) = 1/(i+1)
  
  After n elements, P(element i is chosen):
  = P(picked at step i) × P(not replaced after)
  = 1/(i+1) × (i+1)/(i+2) × (i+2)/(i+3) × ... × (n-1)/n
  = 1/n  ✓ (telescoping!)
```

### General Case: k Elements

```
FUNCTION reservoirSampleK(stream, k):
    reservoir = first k elements of stream
    FOR i = k TO end of stream:
        j = random integer in [0, i]
        IF j < k:
            reservoir[j] = stream[i]
    RETURN reservoir

  Time:  O(n) — one pass
  Space: O(k)
```

---

## 4.3 Correctness Proof

```
  Claim: After processing n elements, each element has
  probability exactly k/n of being in the reservoir.
  
  Proof by induction:
  
  Base: After k elements, all are in reservoir. P = k/k = 1. ✓
  
  Step: Assume true after i elements (each has P = k/i).
  When element (i+1) arrives:
  
  P(new element enters) = k/(i+1)
  
  P(old element survives)
    = P(not replaced)
    = 1 - P(this slot is chosen for replacement)
    = 1 - (1/(i+1))          [new enters with P k/(i+1),
                               replaces one of k slots uniformly,
                               so P any specific slot replaced = 1/(i+1)]
    = i/(i+1)
  
  P(old element in reservoir after step i+1)
    = k/i × i/(i+1) = k/(i+1) ✓
  
  P(new element) = k/(i+1) ✓
  
  By induction, all elements have P = k/n after n elements. □
```

---

## 4.4 Step-by-Step Trace (k = 2)

```
  Stream: [A, B, C, D, E]
  
  Step 0-1: reservoir = [A, B]
  
  Step 2 (i=2): j = random(0..2)
    j=0 → reservoir[0] = C → [C, B]
    j=1 → reservoir[1] = C → [A, C]
    j=2 → no change         → [A, B]
  
  Step 3 (i=3): j = random(0..3)
    j=0 → replace slot 0 with D
    j=1 → replace slot 1 with D
    j=2,3 → no change
  
  Step 4 (i=4): j = random(0..4)
    j=0 → replace slot 0 with E
    j=1 → replace slot 1 with E
    j=2,3,4 → no change
  
  Final: each element has P = 2/5 = 0.4 of being in sample
```

---

## 4.5 Weighted Reservoir Sampling

```
  Each element has a weight wᵢ. Want sample proportional to weights.
  
  Algorithm A-Res (Efraimidis & Spirakis):
  
FUNCTION weightedReservoir(stream, weights, k):
    // For first k elements, compute key = random^(1/w)
    heap = min-heap of size k
    FOR each element (item, w) in stream:
        key = random(0,1) ^ (1/w)    // u^(1/w)
        IF heap.size < k:
            heap.insert(key, item)
        ELSE IF key > heap.minKey:
            heap.replaceMin(key, item)
    RETURN items in heap

  Time:  O(n log k)
  Space: O(k)
  
  Higher weight → higher key → more likely to stay in reservoir
```

---

## 4.6 Practical Applications

```
  ┌──────────────────────────────────────────────────────────┐
  │  Application              │  Why Reservoir Sampling?     │
  ├────────────────────────────┼─────────────────────────────┤
  │  Random line from huge file│  Can't load entire file     │
  │  Stream analytics          │  Unbounded data stream      │
  │  Database sampling         │  Table too large for memory │
  │  A/B testing               │  Random user selection      │
  │  Random graph edge sample  │  Too many edges             │
  └────────────────────────────┴─────────────────────────────┘
  
  ★ "Random line from file" is a classic interview problem:
     Use k=1 reservoir sampling!
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Algorithm | Replace with P = k/(i+1) at step i |
| Correctness | Each element has P = k/n |
| Time | O(n) single pass |
| Space | O(k) |
| Weighted variant | Key = u^(1/w), keep top-k keys |
| Key property | Works without knowing n |

---

## Quick Revision Questions

1. **Why does** the telescoping product show each element has P = 1/n?
2. **Implement** reservoir sampling for k = 3 and trace on [1,2,3,4,5,6].
3. **What happens** if you replace with P = 1/2 always? Why is that wrong?
4. **How does** weighted reservoir sampling differ from standard?
5. **Can reservoir sampling** be parallelized? If so, how?
6. **What is** the expected number of replacements in the reservoir?

---

[← Previous: Random Algorithms](03-random-algorithms.md) | [Back to README](../README.md) | [Next: Shuffle Algorithm →](05-shuffle-algorithm.md)
