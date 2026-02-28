# Chapter 2: Disjoint Sets Concept

## Overview

The mathematical foundation of Union-Find lies in the concept of **disjoint sets** — sets that have no elements in common. Understanding this concept is crucial before diving into the data structure itself.

---

## What Are Disjoint Sets?

Two sets are **disjoint** if their intersection is empty — they share no common elements.

```
SET A = {1, 2, 3}        SET B = {4, 5, 6}

A ∩ B = {} (empty)   →   A and B are DISJOINT ✓

         ┌─────────┐         ┌─────────┐
         │  1  2   │         │  4  5   │
         │    3    │         │    6    │
         └─────────┘         └─────────┘
           Set A               Set B
        (no overlap between A and B)
```

**Non-disjoint** sets share at least one element:
```
SET C = {1, 2, 3}        SET D = {3, 4, 5}

C ∩ D = {3}           →   C and D are NOT disjoint ✗

         ┌─────────┐─────────┐
         │  1  2   │ 3 │ 4  5│
         └─────────┘─────────┘
           Set C    overlap  Set D
```

---

## Partition of a Universe

A **partition** of a set S is a collection of non-empty, **mutually disjoint** subsets whose union equals S.

```
Universe U = {0, 1, 2, 3, 4, 5, 6, 7}

Valid Partition:
  S1 = {0, 1, 2}
  S2 = {3, 4}
  S3 = {5, 6, 7}

Check:
  ✓ S1 ∩ S2 = {}        (all pairs disjoint)
  ✓ S1 ∩ S3 = {}
  ✓ S2 ∩ S3 = {}
  ✓ S1 ∪ S2 ∪ S3 = U    (covers entire universe)

  ┌─────────┬───────┬─────────┐
  │ 0  1  2 │ 3   4 │ 5  6  7 │
  └─────────┴───────┴─────────┘
      S1        S2       S3
```

**Rules for a valid partition:**
1. Every element belongs to **exactly one** subset
2. No subset is empty
3. Subsets don't overlap
4. Union of all subsets = the entire universe

---

## The DSU Invariant

Union-Find **always maintains a valid partition**. At any point:

```
INVARIANT: Every element belongs to exactly ONE set

Step 0 (Initial):     {0} {1} {2} {3} {4}    ← 5 singleton sets
Step 1 (Union 0,1):   {0,1} {2} {3} {4}      ← 4 sets
Step 2 (Union 2,3):   {0,1} {2,3} {4}        ← 3 sets
Step 3 (Union 0,3):   {0,1,2,3} {4}          ← 2 sets
Step 4 (Union 0,4):   {0,1,2,3,4}            ← 1 set

Number of sets: starts at n, decreases by 1 with each Union
```

---

## Representative Element

Each set has a **representative** (also called the **leader** or **root**). This is the element that identifies the entire set.

```
Set = {3, 7, 1, 9}
Representative = 3  (could be any element, but must be consistent)

When we call Find(7), Find(1), Find(9), or Find(3),
they ALL return 3 (the representative)

  Find(7) = 3  ┐
  Find(1) = 3  ├── Same representative → same set!
  Find(9) = 3  │
  Find(3) = 3  ┘
```

**Key property**: Two elements x and y are in the same set if and only if `Find(x) == Find(y)`.

---

## Operations on Disjoint Sets

### Make-Set(x)
Creates a new set containing only element x.

```
Before:  {0, 1}  {2, 3}
MakeSet(4)
After:   {0, 1}  {2, 3}  {4}

Element 4 becomes its own representative:
  Find(4) = 4
```

### Find(x)
Returns the representative of the set containing x.

```
Sets:  {0, 1, 5}  {2, 3}  {4}
       rep = 0     rep = 2  rep = 4

Find(5) = 0
Find(3) = 2
Find(4) = 4
```

### Union(x, y)
Merges the sets containing x and y.

```
Before:  {0, 1}  {2, 3}     reps: 0, 2
Union(1, 3)
After:   {0, 1, 2, 3}       rep: 0 (or 2)

Now Find(1) == Find(3)   ← both return the same root
```

---

## Equivalence Relations

Disjoint sets naturally model **equivalence relations**. An equivalence relation satisfies:

```
┌─────────────────────────────────────────────────────┐
│  Property        │  Meaning           │  DSU         │
├──────────────────┼────────────────────┼──────────────┤
│  Reflexive       │  a ~ a             │  Find(a)=    │
│                  │                    │  Find(a) ✓   │
├──────────────────┼────────────────────┼──────────────┤
│  Symmetric       │  a ~ b → b ~ a    │  Union is    │
│                  │                    │  bidirectional│
├──────────────────┼────────────────────┼──────────────┤
│  Transitive      │  a~b ∧ b~c → a~c  │  If a,b same │
│                  │                    │  set & b,c   │
│                  │                    │  same → a,c  │
│                  │                    │  same set    │
└──────────────────┴────────────────────┴──────────────┘
```

**Example**: "Is in the same connected component" is an equivalence relation.
- Node A is connected to itself (reflexive)
- If A connects to B, then B connects to A (symmetric)
- If A connects to B and B connects to C, then A connects to C (transitive)

---

## Visualization: Set Evolution

```
Time 0: MakeSet for each element
┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐
│0│ │1│ │2│ │3│ │4│ │5│
└─┘ └─┘ └─┘ └─┘ └─┘ └─┘

Time 1: Union(0, 1)
┌───┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐
│0 1│ │2│ │3│ │4│ │5│
└───┘ └─┘ └─┘ └─┘ └─┘

Time 2: Union(2, 3)
┌───┐ ┌───┐ ┌─┐ ┌─┐
│0 1│ │2 3│ │4│ │5│
└───┘ └───┘ └─┘ └─┘

Time 3: Union(4, 5)
┌───┐ ┌───┐ ┌───┐
│0 1│ │2 3│ │4 5│
└───┘ └───┘ └───┘

Time 4: Union(0, 2)
┌───────┐ ┌───┐
│0 1 2 3│ │4 5│
└───────┘ └───┘

Time 5: Union(3, 4)
┌───────────┐
│0 1 2 3 4 5│
└───────────┘
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Disjoint Sets | Sets with no common elements |
| Partition | Division of universe into disjoint sets |
| Representative | A designated element identifying each set |
| Make-Set | Create a singleton set |
| Find | Get the representative of an element's set |
| Union | Merge two sets into one |
| Equivalence Relation | Reflexive, symmetric, transitive relation modeled by DSU |

---

## Quick Revision Questions

1. **What makes two sets "disjoint"?**
2. **What is a partition of a universe set?**
3. **What is the role of the representative element?**
4. **How do you check if two elements are in the same set?**
5. **By how much does the number of sets decrease after each Union?**
6. **What three properties make DSU model an equivalence relation?**

---

[← Previous: What is Union-Find?](01-what-is-union-find.md) | [Next: Forest Representation →](03-forest-representation.md)
