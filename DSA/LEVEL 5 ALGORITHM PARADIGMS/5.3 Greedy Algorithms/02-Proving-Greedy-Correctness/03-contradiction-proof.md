# Chapter 3: Contradiction Proof

## Overview
**Proof by contradiction** is another powerful technique for proving greedy correctness. Instead of transforming the optimal solution, we **assume** the greedy solution is NOT optimal and derive a logical impossibility, thus proving greedy must be optimal.

---

## Core Idea

```
┌─────────────────────────────────────────────────────────┐
│              CONTRADICTION PROOF                        │
│                                                         │
│  1. ASSUME greedy solution G is NOT optimal             │
│  2. Then there exists a BETTER solution O               │
│  3. Analyze how O and G differ                          │
│  4. Derive a CONTRADICTION:                             │
│     - O can't actually be better, OR                    │
│     - O isn't actually feasible, OR                     │
│     - Greedy would have done what O does                │
│  5. CONCLUDE: G must be optimal                         │
│                                                         │
│  Logic: ¬(G is optimal) → CONTRADICTION                │
│         Therefore: G IS optimal                         │
└─────────────────────────────────────────────────────────┘
```

---

## Formal Template

```
CONTRADICTION PROOF:
═════════════════════

CLAIM: Greedy algorithm produces optimal solution G.

PROOF:
1. Suppose for contradiction that G is NOT optimal.
2. Then ∃ solution O with cost(O) < cost(G)  [or > for max].
3. Consider G and O. Since they differ...
4. [Problem-specific analysis showing O can't exist]
5. This contradicts our assumption.
6. Therefore G is optimal. □

     ┌──────────────────────────┐
     │  Assume G NOT optimal    │
     │          │                │
     │          ▼                │
     │  ∃ better solution O     │
     │          │                │
     │          ▼                │
     │  Analyze G vs O          │
     │          │                │
     │          ▼                │
     │   CONTRADICTION! ✗       │
     │          │                │
     │          ▼                │
     │  ∴ G IS optimal ✓       │
     └──────────────────────────┘
```

---

## Example 1: Activity Selection

**Claim:** Greedy (earliest finish time) gives maximum number of non-overlapping activities.

```
PROOF BY CONTRADICTION:

Suppose G = {g1, g2, ..., gk} is NOT optimal.
Then ∃ O = {o1, o2, ..., om} with m > k.

Let j be the first index where gj ≠ oj.

Observation: finish(gj) ≤ finish(oj)
  (Because greedy picks earliest finish among eligible)

Since finish(gj) ≤ finish(oj):
  - Activity oj+1 in O starts after finish(oj) ≥ finish(gj)
  - So oj+1 was available when greedy chose gj+1
  - Greedy could have picked oj+1 or something with 
    even earlier finish time

This means greedy could always match O step-for-step.
If m > k, then o(k+1) exists and starts after finish(gk).
But then greedy would have picked o(k+1) or similar!

CONTRADICTION: Greedy would not have stopped at k if 
there was a compatible activity remaining.

Therefore k ≥ m, and G is optimal. □
```

---

## Example 2: Huffman Coding Optimality

**Claim:** Huffman coding produces an optimal prefix code.

```
PROOF BY CONTRADICTION:

Suppose Huffman tree H is NOT optimal.
Then ∃ optimal tree T with cost(T) < cost(H).

Let x, y be the two lowest-frequency characters.
Huffman merges x and y as siblings at the deepest level.

Key observations:
1. In ANY optimal tree, the two deepest leaves must be siblings
   (otherwise we could improve the tree)
2. In optimal tree T, lowest-frequency chars should be deepest
   (otherwise swap with deeper char → cost decreases)

CONTRADICTION:
If T doesn't have x,y as deepest siblings:
  - Swap x with a deeper leaf → cost decreases or stays same
  - Swap y with other deepest leaf → cost decreases or stays same
  - But T was supposedly optimal → can't decrease!
  
So T must have x,y as siblings at deepest level — 
exactly what Huffman does.

By induction on number of characters, Huffman = optimal at 
every merge step.

Therefore H is optimal. □
```

---

## Example 3: Minimum Coin Problem (Standard Denominations)

**Claim:** For denominations {1, 5, 10, 25}, greedy (largest first) uses minimum coins.

```
PROOF BY CONTRADICTION:

Suppose greedy solution G uses more coins than optimal O.

Key structural lemma: In any optimal solution:
  - At most four 1¢ coins (else replace five 1¢ with one 5¢)
  - At most one 5¢ coin (else replace two 5¢ with one 10¢)
  - At most two 10¢ coins (else replace: 10+10+5 = 25)
  - Combined: 10¢ and 5¢ together at most 20¢

Greedy always picks the largest possible coin.
Any optimal solution that uses more small coins can be 
improved by combining them into larger coins.

If O uses different coins than G:
  O must have more small coins somewhere that combine 
  to a larger denomination. But then O isn't optimal!

CONTRADICTION: O can't be better than G because any 
non-greedy choice leads to wasted coin slots. □
```

---

## When to Use Contradiction

```
┌────────────────────────────────────────────────────────┐
│        CONTRADICTION IS BEST WHEN:                    │
│                                                        │
│  ✓ Direct construction is difficult                   │
│  ✓ The "optimal can't be better" is intuitive         │
│  ✓ There are structural constraints that help         │
│  ✓ You can derive impossible situations easily        │
│                                                        │
│  TYPICAL CONTRADICTION SOURCES:                       │
│  • "Greedy would have picked that too"                │
│  • "O can be improved, contradicting optimality"      │
│  • "O violates a structural constraint"               │
│  • "Greedy covers all cases that O does"              │
└────────────────────────────────────────────────────────┘
```

---

## Comparison of All Three Proof Techniques

```
┌──────────────┬──────────────┬──────────────┐
│  EXCHANGE    │ STAYS AHEAD  │ CONTRADICTION│
├──────────────┼──────────────┼──────────────┤
│ Transform    │ Compare      │ Assume ¬G    │
│ OPT → G     │ step-by-step │ → impossible │
│              │              │              │
│ Constructive │ Inductive    │ Indirect     │
│              │              │              │
│ Show: each   │ Show: G ≥ O  │ Show: ¬G    │
│ swap is safe │ at each step │ → ⊥         │
│              │              │              │
│ Best for:    │ Best for:    │ Best for:    │
│ orderings    │ incremental  │ when O has   │
│ permutations │ selection    │ structural   │
│              │              │ constraints  │
└──────────────┴──────────────┴──────────────┘
```

---

## Common Contradiction Patterns

### Pattern 1: "Greedy Would Have Done It Too"

```
Assume a better solution O exists.
Show that at some step, O makes a choice C.
Show that greedy also had access to C (or better).
So greedy would be at least as good. Contradiction. □
```

### Pattern 2: "O Can Be Improved"

```
Assume O is optimal and different from G.
Find a part of O that can be improved.
But O is already optimal — can't improve!
Contradiction. □
```

### Pattern 3: "Structural Impossibility"

```
Assume greedy misses some element X.
Show that including X requires excluding Y, Z.
Show Y + Z are worth more than X.
So O with X isn't actually better. Contradiction. □
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **What** | Assume greedy isn't optimal, derive impossibility |
| **Structure** | Assume ¬P, derive contradiction, conclude P |
| **Strength** | Works when direct proof is difficult |
| **Common Use** | Structural constraints, counting arguments |
| **Key Step** | Finding the right contradiction source |
| **Compared To** | Less constructive than exchange, less inductive than stays-ahead |

---

## Quick Revision Questions

1. **What is the logical structure of a proof by contradiction?**
   > Assume the greedy solution is NOT optimal, analyze the hypothetical better solution, derive a logical impossibility, conclude that greedy must be optimal.

2. **What are the three common sources of contradiction in greedy proofs?**
   > (1) Greedy would have made that choice too, (2) The "better" solution can be improved (contradicting its optimality), (3) Structural constraints make the scenario impossible.

3. **How does contradiction differ from the exchange argument?**
   > Exchange constructively transforms OPT into GREEDY; contradiction assumes GREEDY isn't optimal and shows this is impossible.

4. **When is contradiction preferred over other proof techniques?**
   > When the problem has structural constraints (like coin denominations) that make direct construction difficult but impossibility arguments natural.

5. **In the Huffman coding proof, what is the key contradiction?**
   > Any optimal tree must have the two least frequent symbols as siblings at the deepest level — which is exactly what Huffman does.

6. **Can a contradiction proof be converted to a constructive proof?**
   > Often yes — many contradiction proofs implicitly contain an exchange argument. The contradiction "O can be improved" is essentially "transform O toward G."

---

[← Previous: Stays Ahead Argument](02-stays-ahead-argument.md) | [Next: Why Proofs Matter →](04-why-proofs-matter.md)

[← Back to README](../README.md)
