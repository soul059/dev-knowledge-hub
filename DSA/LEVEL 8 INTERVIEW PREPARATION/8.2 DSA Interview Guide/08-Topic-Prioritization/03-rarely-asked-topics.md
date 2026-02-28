# Chapter 8.3: Rarely Asked Topics

```
╔══════════════════════════════════════════════════════════╗
║          RARELY ASKED TOPICS                             ║
║   Know they exist, but don't over-invest time here       ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Some DSA topics appear in fewer than 15% of interviews. While they're interesting and valuable for competitive programming, they have **low ROI for interview preparation**. Know what they are so you recognize them, but don't prioritize them over must-know topics.

---

## Rarely Asked Topics List

```
┌─────────────────────────────────────────────────────────┐
│ TOPIC                     │ INTERVIEW FREQUENCY         │
├───────────────────────────┼─────────────────────────────┤
│ Advanced DP (bitmask DP,  │ 5-10%                      │
│ digit DP, profile DP)     │ (mostly competitive prog)  │
│                           │                             │
│ Geometry / Computational  │ < 5%                        │
│ Geometry                  │ (convex hull, line sweep)   │
│                           │                             │
│ Heavy-Light Decomposition │ < 2%                        │
│                           │ (tree paths queries)        │
│                           │                             │
│ Suffix Array / Suffix Tree│ < 5%                        │
│                           │ (advanced string matching)  │
│                           │                             │
│ Network Flow              │ < 3%                        │
│ (Max flow, min cut)       │                             │
│                           │                             │
│ Game Theory               │ < 5%                        │
│ (Sprague-Grundy, Nim)     │                             │
│                           │                             │
│ Number Theory             │ < 5%                        │
│ (Primes, GCD, modular     │ (sometimes at quant firms) │
│  arithmetic)              │                             │
│                           │                             │
│ String Algorithms         │ < 10%                       │
│ (KMP, Rabin-Karp, Z-algo) │                            │
│                           │                             │
│ Centroid Decomposition    │ < 2%                        │
│                           │                             │
│ Persistent Data Structures│ < 2%                        │
│                           │                             │
│ Sqrt Decomposition        │ < 3%                        │
│                           │                             │
│ Fenwick Tree / BIT        │ ~10%                        │
│ (borderline good-to-know) │                             │
└───────────────────────────┴─────────────────────────────┘
```

---

## When These Might Appear

```
SPECIAL CONTEXTS:
┌────────────────────────────┬─────────────────────────────┐
│ Context                     │ Rare Topics That May Appear│
├────────────────────────────┼─────────────────────────────┤
│ Quant / Trading Firms       │ Number Theory, Game Theory,│
│ (Jane Street, Citadel)      │ Math-heavy problems        │
│                             │                             │
│ Competitive Programming     │ Advanced DP, Suffix Arrays,│
│ focused teams               │ Segment Trees, Network Flow│
│                             │                             │
│ Infrastructure / Low-level  │ Bit manipulation (advanced),│
│ Systems roles               │ Memory-efficient structures│
│                             │                             │
│ Search / NLP teams          │ Suffix trees, KMP, Tries   │
│                             │                             │
│ Senior / Staff level        │ Any topic (broader range)   │
│ (Staff+ at FAANG)           │                             │
└────────────────────────────┴─────────────────────────────┘
```

---

## What to Do If You Encounter a Rare Topic

```
STRATEGY:
  1. DON'T PANIC — You're not expected to have memorized
     every algorithm
  
  2. Apply fundamentals:
     - Break down the problem
     - Try brute force
     - Look for patterns
  
  3. Communicate what you know:
     "I recognize this relates to [concept]. I haven't
      implemented it recently, but I know the general
      approach involves [key idea]."
  
  4. Ask for guidance:
     "I'm not as familiar with this specific algorithm.
      Could you point me in the right direction?"

  5. Show problem-solving skills:
     Even without knowing the exact algorithm, you can
     often derive a solution from first principles.
```

---

## Minimum Awareness Level

```
YOU SHOULD AT LEAST KNOW:

KMP / Rabin-Karp:
  → Purpose: Pattern matching in strings
  → When: "Find pattern in text efficiently"
  → Complexity: O(n + m) vs brute force O(n × m)

Network Flow:
  → Purpose: Maximum flow through a network
  → When: Matching problems, bipartite graphs
  → Don't need to code it; know when it applies

Number Theory:
  → GCD: Euclidean algorithm (easy to code)
  → Prime check: O(√n)
  → Modular arithmetic: (a + b) % m = ((a%m) + (b%m)) % m

Game Theory:
  → Minimax concept
  → Nim: XOR of all piles = 0 → losing position
  → Can usually solve with DP/recursion
```

---

## Time Investment Guide

```
PREPARATION TIME BUDGET (12 weeks total):

Must-Know Topics:     8 weeks (67% of time)
Good-to-Know Topics:  3 weeks (25% of time)
Rarely Asked Topics:  1 week  (8% of time)
                      ──────
                      12 weeks

FOR THE 1 WEEK ON RARE TOPICS:
  Day 1: KMP string matching (understand, don't memorize)
  Day 2: Number theory basics (GCD, primes)
  Day 3: Advanced DP patterns (bitmask DP overview)
  Day 4: Geometry basics (distance, area formulas)
  Day 5: Review and read about other rare topics
  Day 6-7: Mixed practice from all tiers

IF YOU HAVE LESS THAN 12 WEEKS:
  Skip rare topics entirely.
  Focus 100% on must-know + good-to-know.
```

---

## Summary Table

| Topic | Frequency | Time Investment | ROI |
|-------|-----------|----------------|-----|
| Advanced DP | 5-10% | 1 day awareness | Low |
| Geometry | < 5% | 1 day awareness | Very low |
| Suffix Trees | < 5% | Read about | Very low |
| Network Flow | < 3% | Read about | Very low |
| Number Theory | < 5% | 1 day basics | Low-Medium |
| KMP/Rabin-Karp | < 10% | 1 day understanding | Low |
| Game Theory | < 5% | Read about | Low |

---

## Quick Revision Questions

1. **Should you spend significant time on rarely asked topics?**
   - No — allocate at most 1 week (8% of total prep time). If short on time, skip them entirely

2. **What should you do if a rare topic appears in your interview?**
   - Stay calm, apply fundamentals, communicate what you know about the concept, and ask for guidance if needed

3. **Which rare topics have the highest chance of appearing?**
   - KMP/string algorithms (~10%) and advanced DP/bitmask DP (5-10%) are the most common among rare topics

4. **At which companies might rare topics appear?**
   - Quant/trading firms (number theory, game theory), competitive-programming-focused teams, and Staff+ level interviews at FAANG

5. **What's the minimum you should know about rare topics?**
   - Know their purpose and when they apply — you don't need to perfectly code them, but you should recognize the problem type

6. **How does time ROI compare across topic tiers?**
   - Must-know topics: highest ROI (80%+ interview coverage). Good-to-know: medium ROI (20-50%). Rare: low ROI (< 15%)

---

[← Previous: Good-to-Know Topics](02-good-to-know-topics.md) | [Next: Company-Specific Focus →](04-company-specific-focus.md)

---
[← Back to README](../README.md)
