# Chapter 3: Optimal Substructure

## 📋 Overview
**Optimal Substructure** means that the optimal solution to a problem can be constructed from optimal solutions of its subproblems. This is the second key property required for Dynamic Programming.

---

## 🧠 Core Concept

```
┌─────────────────────────────────────────────────────────┐
│              OPTIMAL SUBSTRUCTURE                       │
│                                                         │
│  If the BEST solution to a problem CONTAINS the         │
│  BEST solutions to its subproblems, the problem         │
│  has optimal substructure.                              │
│                                                         │
│  ┌──────────────────────┐                               │
│  │ Optimal Solution     │                               │
│  │   to Problem P       │                               │
│  │  ┌────────┐ ┌──────┐ │                               │
│  │  │Optimal │ │Optimal│ │                               │
│  │  │Sol to  │ │Sol to │ │                               │
│  │  │Sub P1  │ │Sub P2 │ │                               │
│  │  └────────┘ └──────┘ │                               │
│  └──────────────────────┘                               │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 Understanding Optimal Substructure

### Example: Shortest Path

```
Find shortest path from A to D:

    A ──3──► B ──2──► D
    │                 ▲
    └──4──► C ──1────┘

Shortest path: A → B → D (cost = 5)

Does it have optimal substructure?

  Shortest A→D via B = Shortest(A→B) + Shortest(B→D)
                      =     3        +     2
                      =     5  ✓

  The optimal solution A→D CONTAINS optimal sub-paths:
    • Shortest A→B = 3 (optimal)
    • Shortest B→D = 2 (optimal)
    
  YES! Optimal substructure holds.
```

### Counter-Example: Longest Simple Path

```
Find longest simple path from A to D:

    A ──3──► B ──2──► D
    │        │        ▲
    └──4──► C ──1────┘

Longest simple path: A→C→B→...? Wait, can't go B→D if came from C.
Actually: A→B→C→... stuck (no path C→D without revisiting)
Or: A→C→B→D? If edges allow: cost = 4+?+2

The problem: Longest path A→D via B is NOT necessarily
  Longest(A→B) + Longest(B→D)
  
Because subpaths may share vertices, creating conflicts.

NO optimal substructure for longest simple path!
(This is why longest path is NP-Hard)
```

---

## 📐 Formal Definition

```
A problem has OPTIMAL SUBSTRUCTURE if:

  OPT(problem) = combine( OPT(subproblem₁), OPT(subproblem₂), ... )

Where:
  • OPT(x) = optimal solution to problem x
  • combine = some operation (add, max, min, etc.)
  • Subproblems are INDEPENDENT (don't share resources)
```

---

## 🧪 Testing for Optimal Substructure

### The Cut-and-Paste Argument

```
┌─────────────────────────────────────────────────────────┐
│  PROOF TECHNIQUE: Cut-and-Paste                         │
│                                                         │
│  1. Assume you have an optimal solution to the problem  │
│  2. Extract the solution to a subproblem from it        │
│  3. Ask: "Is this extracted part optimal for the sub?"  │
│  4. Assume NOT → you can "cut" the suboptimal part      │
│     and "paste" a better solution                       │
│  5. This would improve the overall solution             │
│  6. CONTRADICTION! (We assumed it was optimal)          │
│  7. Therefore, the subproblem solution MUST be optimal  │
└─────────────────────────────────────────────────────────┘
```

### Example: Rod Cutting Problem

```
Rod of length 4, prices: p[1]=1, p[2]=5, p[3]=6, p[4]=9

Optimal: cut into 2+2, revenue = 5+5 = 10

Cut-and-paste argument:
  If the left piece (length 2) wasn't optimally used,
  we could replace it with a better use → higher total.
  Contradiction! So each piece must be optimal.
  
  OPT(4) = max over all cuts k:
            price[k] + OPT(4-k)
          = max(1+OPT(3), 5+OPT(2), 6+OPT(1), 9+OPT(0))
          = max(1+8, 5+5, 6+1, 9+0)
          = max(9, 10, 7, 9)
          = 10 ✓
```

---

## 🏗️ Common Patterns of Optimal Substructure

### Pattern 1: Choose and Recurse
```
OPT(i) = best choice among:
  • Include item i → value[i] + OPT(remaining after i)
  • Exclude item i → OPT(remaining without i)

Example: 0/1 Knapsack
  OPT(i, w) = max(
      val[i] + OPT(i-1, w-wt[i]),   // take item i
      OPT(i-1, w)                    // skip item i
  )
```

### Pattern 2: Split and Combine
```
OPT(i, j) = best split point k where:
  OPT(i, j) = min/max over k:
      OPT(i, k) + OPT(k+1, j) + cost(i, j)

Example: Matrix Chain Multiplication
  OPT(i, j) = min over k from i to j-1:
      OPT(i, k) + OPT(k+1, j) + dims[i]*dims[k+1]*dims[j+1]
```

### Pattern 3: Previous States
```
OPT(i) = f(OPT(i-1), OPT(i-2), ...)

Example: Fibonacci / Climbing Stairs
  OPT(i) = OPT(i-1) + OPT(i-2)
```

### Pattern 4: Two Sequences
```
OPT(i, j) = f(OPT(i-1, j), OPT(i, j-1), OPT(i-1, j-1))

Example: Longest Common Subsequence
  If s1[i] == s2[j]:
      OPT(i,j) = 1 + OPT(i-1, j-1)
  Else:
      OPT(i,j) = max(OPT(i-1,j), OPT(i,j-1))
```

---

## ⚠️ Problems WITHOUT Optimal Substructure

```
┌──────────────────────────────────────────────┐
│  These do NOT have optimal substructure:      │
│                                               │
│  1. Longest Simple Path                       │
│     - Subpaths may share vertices             │
│     - Subproblems not independent             │
│                                               │
│  2. Cheapest Flight with ≤ K stops            │
│     - Adding constraint K breaks structure    │
│     - BUT can be fixed by adding K to state!  │
│                                               │
│  3. Traveling Salesman (naive formulation)    │
│     - "Visited" constraint couples subprobs   │
│     - Fixed by adding visited set to state    │
└──────────────────────────────────────────────┘

KEY INSIGHT: Sometimes reformulating the state
can RESTORE optimal substructure!
```

---

## 🔄 Optimal Substructure + Overlapping Subproblems

```
┌────────────────────┐    ┌────────────────────┐
│    Overlapping     │    │     Optimal        │
│    Subproblems     │    │    Substructure    │
│                    │    │                    │
│ "Same work done    │    │ "Best solution     │
│  multiple times"   │    │  built from best   │
│                    │    │  sub-solutions"    │
└────────┬───────────┘    └────────┬───────────┘
         │                         │
         └───────────┬─────────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │  DYNAMIC PROGRAMMING  │
         │                       │
         │  Both properties must │
         │  hold for DP to work  │
         └───────────────────────┘

What if only ONE holds?

  Only Overlapping → Memoization helps speed up, 
                     but can't guarantee optimality
                     
  Only Optimal Substructure → Greedy or D&C may work,
                              but no reuse benefit
```

---

## 💡 Step-by-Step: Proving Optimal Substructure

### Example: Coin Change (Minimum Coins)

```
Problem: Find minimum coins to make amount = 11
Coins: [1, 5, 6]

Step 1: Define what "optimal" means
  OPT(11) = minimum number of coins to make 11

Step 2: Express in terms of subproblems
  If we use coin c as the LAST coin:
    OPT(11) = 1 + OPT(11 - c)

Step 3: Prove subproblem solution is optimal
  Suppose OPT(11) = 3 coins (e.g., 5+5+1)
  The remaining after using coin 5: OPT(6) = 2 (must be optimal)
  
  If OPT(6) could be done in 1 coin (it can! coin=6):
    Then OPT(11) = 1 + 1 = 2 (using 5+6)
    
  Contradiction? No — we need to check ALL coins:
    OPT(11) = min(1 + OPT(10), 1 + OPT(6), 1 + OPT(5))
            = min(1 + 2,        1 + 1,       1 + 1)
            = min(3, 2, 2)
            = 2 ✓

Step 4: Verify with cut-and-paste
  If in OPT(11) the sub-solution wasn't optimal,
  we could swap it for a better one → fewer total coins.
  Contradiction. ∴ Optimal substructure holds. ✓
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Optimal Substructure** | Optimal solution contains optimal solutions to subproblems |
| **Proof Technique** | Cut-and-paste argument |
| **Key Requirement** | Subproblems must be independent |
| **Common Patterns** | Choose/Recurse, Split/Combine, Previous States, Two Sequences |
| **With DP** | Guarantees we build globally optimal solution from locally optimal parts |
| **Without It** | DP may give wrong answer; need other approaches |

---

## ❓ Quick Revision Questions

1. **Define optimal substructure in your own words.**
2. **Why does the Longest Simple Path problem NOT have optimal substructure?**
3. **Explain the cut-and-paste proof technique.**
4. **Does the Shortest Path problem always have optimal substructure? What about with negative cycles?**
5. **How can reformulating the state sometimes restore optimal substructure?**
6. **Which DP pattern does the Knapsack problem follow — Choose/Recurse or Split/Combine?**

---

[← Previous: Overlapping Subproblems](02-overlapping-subproblems.md) | [Next: Memoization →](04-memoization.md)

[← Back to README](../README.md)
