# Chapter 6: 2-SAT Concept with Union-Find

## Overview

While 2-SAT is traditionally solved with strongly connected components (Tarjan's/Kosaraju's), **some special cases** of 2-SAT-like problems can be solved with Union-Find, particularly equality/inequality constraint problems.

---

## What is 2-SAT?

```
2-SAT (2-Satisfiability):
  Given boolean variables x₁, x₂, ..., xₙ
  And clauses of the form (a ∨ b) where a,b are 
  literals (xᵢ or ¬xᵢ)
  
  Find an assignment that satisfies ALL clauses,
  or determine it's impossible.

  Full 2-SAT: O(V + E) with SCC (Tarjan's)
  
  Special case that DSU handles:
  EQUALITY constraints: "x and y must be equal/different"
```

---

## Equality Constraints with DSU

```
Problem: Given constraints like:
  x₁ == x₂  (x₁ equals x₂)
  x₃ != x₄  (x₃ not equal to x₄)
  
  Is it satisfiable?

This is solvable with Union-Find!

Key idea:
  "x == y" → Union(x, y)  (same group)
  "x != y" → they must be in DIFFERENT groups

Algorithm:
  1. Process all EQUALITY constraints → Union
  2. Check all INEQUALITY constraints → Find
     If Find(x) == Find(y) but x != y → CONTRADICTION!
```

---

## Problem: Satisfiability of Equality Equations (LeetCode 990)

```
Given equations like "a==b" and "a!=b", determine if 
they can all be satisfied simultaneously.

Input: ["a==b", "b!=a"]
Output: false (a==b contradicts b!=a)

Input: ["a==b", "b==c", "a==c"]
Output: true (all consistent)

Input: ["a==b", "b!=c", "c==a"]
Output: false (a==b, c==a → b==c, but b!=c!)
```

```python
def equationsPossible(equations):
    uf = UnionFind(26)  # 26 letters
    
    # Step 1: Process all equalities
    for eq in equations:
        if eq[1:3] == '==':
            x = ord(eq[0]) - ord('a')
            y = ord(eq[3]) - ord('a')
            uf.union(x, y)
    
    # Step 2: Check all inequalities
    for eq in equations:
        if eq[1:3] == '!=':
            x = ord(eq[0]) - ord('a')
            y = ord(eq[3]) - ord('a')
            if uf.connected(x, y):
                return False  # contradiction!
    
    return True
```

---

## Why Process Equalities First?

```
Order matters!

WRONG (process in input order):
  equations = ["a!=b", "a==b"]
  
  Process "a!=b": a and b are in different sets → OK
  Process "a==b": union(a, b) → merged... but wait, 
                  we already said they're different!
  
  Problem: If we process inline, we miss the contradiction.

CORRECT (two-pass):
  Pass 1: Process ALL "==" → build equivalence classes
  Pass 2: Check ALL "!=" → verify no contradiction
  
  This works because:
  - "==" constraints are TRANSITIVE (a==b, b==c → a==c)
  - Union-Find captures transitivity automatically
  - "!=" just needs to check against final grouping
```

---

## Generalization: Multiple Values

```
What if variables can take values from {1, 2, ..., k}?

"x == y": Union(x, y)
"x != y": Must be in different groups

For k = 2 (boolean): This is exactly 2-coloring / bipartite
  Use virtual nodes: same[x] and diff[x]

For k ≥ 3: Standard DSU can't handle it directly
  Need constraint propagation or full CSP solver
```

---

## Extended Example: Friend/Enemy Relations

```
N people. Statements:
  "A and B are friends" → same group
  "A and B are enemies" → different groups
  
  With "enemy of enemy is friend":
  If A-enemy-B and B-enemy-C, then A-friend-C.

Using virtual nodes (3 groups per person):
  group[x][0] = friends of x
  group[x][1] = enemies of x
  group[x][2] = enemies of enemies of x

Friends(A, B):
  union(A.0, B.0)  ← friend groups merge
  union(A.1, B.1)  ← their enemies merge
  union(A.2, B.2)  ← enemies of enemies merge

Enemies(A, B):
  union(A.0, B.1)  ← A's friends = B's enemies
  union(A.1, B.2)
  union(A.2, B.0)

Contradiction: connected(A.0, A.1) → A is own enemy!
```

---

## Relationship to Full 2-SAT

```
Full 2-SAT:
  - Variables: x₁, ..., xₙ (boolean)
  - Clauses: (a ∨ b) where a,b ∈ {xᵢ, ¬xᵢ}
  - Example: (x₁ ∨ ¬x₂) ∧ (¬x₁ ∨ x₃)
  - Solution: Implication graph + SCC
  
What DSU handles (subset):
  - Only EQUALITY and INEQUALITY of variables
  - x == y ↔ (x ∨ ¬y) ∧ (¬x ∨ y) in 2-SAT
  - x != y ↔ (x ∨ y) ∧ (¬x ∨ ¬y) in 2-SAT
  
What DSU CANNOT handle:
  - General implications: "if x then y"
  - Complex clauses: (x₁ ∨ x₂)
  
For full 2-SAT: Use Tarjan's algorithm on implication graph.
```

---

## Template

```python
def solve_equality_constraints(n, equalities, inequalities):
    """
    n: number of variables (0 to n-1)
    equalities: list of (x, y) meaning x == y
    inequalities: list of (x, y) meaning x != y
    Returns: True if satisfiable
    """
    uf = UnionFind(n)
    
    # Phase 1: Build equivalence classes
    for x, y in equalities:
        uf.union(x, y)
    
    # Phase 2: Check inequalities
    for x, y in inequalities:
        if uf.connected(x, y):
            return False  # x == y forced, but x != y required
    
    return True
```

---

## Summary Table

| Problem Type | DSU Approach | Full 2-SAT Needed? |
|-------------|-------------|-------------------|
| x == y / x != y | Two-pass Union-Find | No |
| Bipartite check | Virtual nodes | No |
| Friend/Enemy | 2-3 virtual nodes per person | No |
| General (a ∨ b) clauses | - | Yes (SCC) |
| Implications (x → y) | - | Yes (SCC) |

---

## Quick Revision Questions

1. **What subset of 2-SAT can Union-Find solve?**
2. **Why must equalities be processed before inequalities?**
3. **How does the "Satisfiability of Equality Equations" problem work?**
4. **How do you handle "enemy of my enemy is my friend" with DSU?**
5. **When do you need full 2-SAT (SCC) instead of Union-Find?**

---

[← Previous: Virtual Nodes](05-virtual-nodes.md) | [Next: Unit 9 - Accounts Merge →](../09-Problem-Patterns/01-accounts-merge.md)

[← Back to Main README](../README.md)
