# What is a Trie?

## Overview
A **Trie** (from the word "re**trie**val", pronounced "try") is a tree-shaped data structure designed for efficient storage, retrieval, and prefix searching of strings. Unlike binary search trees where each node stores a complete key, in a trie each node represents a **single character** of a key.

---

## Core Idea

Every path from the root to a marked node spells out a stored word. Nodes sharing a common prefix share the same path from the root.

```
                     Words: ["app", "apple", "apt", "bat", "bar"]

                              (root)
                             /      \
                            a        b
                           /          \
                          p            a
                        /   \         / \
                       p*    t*      t*   r*
                      /
                     l
                    /
                   e*

              * = marks end of a complete word
```

---

## Key Insight: Prefix Sharing

The trie above stores 5 words but shares common prefixes:
- `"app"` and `"apple"` share the prefix `"app"`
- `"app"` and `"apt"` share the prefix `"ap"`
- `"bat"` and `"bar"` share the prefix `"ba"`

This prefix sharing is what makes tries **space-efficient** for dictionaries with overlapping prefixes and **time-efficient** for prefix queries.

```
  Without Trie (separate storage):
  ┌───────┐ ┌───────┐ ┌─────┐ ┌─────┐ ┌─────┐
  │ a p p │ │a p p l│ │a p t│ │b a t│ │b a r│
  └───────┘ └───────┘ └─────┘ └─────┘ └─────┘
  Total characters stored: 3+5+3+3+3 = 17

  With Trie (shared prefixes):
  a ─ p ─ p* ─ l ─ e*        (app, apple share "app")
           └─ t*              (apt shares "ap")
  b ─ a ─ t*                  (bat, bar share "ba")
         └─ r*
  Total nodes: 10  (saved 7 characters via sharing!)
```

---

## Formal Definition

> A **Trie** is a rooted tree where:
> 1. Each node contains a set of children (one per possible character)
> 2. Each edge represents a character
> 3. The path from root to any node represents a prefix
> 4. Certain nodes are marked as "end of word"
> 5. The root node represents the empty string ""

---

## Building a Trie Step-by-Step

Let's build a trie by inserting words one at a time:

### Insert "to"
```
  (root)
    │
    t
    │
    o*
```

### Insert "tea"
```
  (root)
    │
    t
   / \
  o*   e
       │
       a*
```

### Insert "ten"
```
  (root)
    │
    t
   / \
  o*   e
      / \
     a*   n*
```

### Insert "inn"
```
       (root)
      /     \
     t       i
    / \      │
   o*  e     n
      / \    │
     a*  n*  n*
```

### Insert "in"
```
       (root)
      /     \
     t       i
    / \      │
   o*  e     n*    ← marked as end-of-word
      / \    │
     a*  n*  n*
```

> Note: "in" and "inn" share the prefix "in". The node for 'n' at level 2 is marked as end-of-word for "in", and has a child 'n' for "inn".

---

## Trie vs Other Data Structures

```
  ┌──────────────────────────────────────────────────────────────┐
  │  ARRAY / LIST                                                │
  │  ["apple", "app", "application"]                             │
  │  Search: O(N × L) — scan every word                          │
  │  Prefix: O(N × L) — scan every word                          │
  ├──────────────────────────────────────────────────────────────┤
  │  HASH SET                                                    │
  │  {"apple", "app", "application"}                              │
  │  Search: O(L) — hash + compare                               │
  │  Prefix: O(N × L) — must check EVERY key                    │
  ├──────────────────────────────────────────────────────────────┤
  │  BALANCED BST (e.g., TreeSet)                                │
  │  Sorted: ["app", "apple", "application"]                     │
  │  Search: O(L × log N) — compare at each level               │
  │  Prefix: O(L × log N + K) — find range                      │
  ├──────────────────────────────────────────────────────────────┤
  │  TRIE  ★                                                     │
  │  Tree structure with shared prefixes                         │
  │  Search: O(L) — follow path                                  │
  │  Prefix: O(P + K) — navigate then collect                    │
  └──────────────────────────────────────────────────────────────┘
  
  L = word length, N = total words, P = prefix length, K = matches
```

---

## Key Properties of a Trie

| Property | Description |
|----------|-------------|
| **Root** | Always empty — represents the empty string |
| **Edges** | Each edge is labeled with exactly one character |
| **Paths** | Each root-to-node path represents a unique prefix |
| **End markers** | Distinguish complete words from mere prefixes |
| **No duplicates** | Each word stored exactly once (same path) |
| **Depth** | Maximum depth = length of the longest word |
| **Alphabet** | Fixed alphabet determines branching factor |

---

## Terminology

| Term | Meaning |
|------|---------|
| **Trie** | The data structure itself (prefix tree) |
| **Node** | Single element storing children + end flag |
| **Root** | Starting node (empty, no character) |
| **Edge** | Connection labeled with a character |
| **Leaf** | Node with no children |
| **End-of-word** | Flag marking a complete word at this node |
| **Prefix** | String represented by path from root to a node |
| **Depth** | Number of edges from root to a node |
| **Branching factor** | Max number of children = alphabet size |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Full name | Trie / Prefix Tree / Digital Tree |
| Core idea | Each path from root = a stored string |
| Key property | Common prefixes share the same path |
| Node contains | Children map + isEndOfWord flag |
| Root | Empty node representing "" |
| Insert/Search | O(L) where L = word length |
| Best at | Prefix-based queries |

---

## Quick Revision Questions

1. **What does "Trie" stand for and why is it named that way?**
   > From "re**trie**val" — designed for efficient information retrieval from a dataset of strings.

2. **What does the root node of a trie represent?**
   > The empty string "". It has no character itself but holds children for the first characters of all stored words.

3. **How does a trie store the word "apple"? Describe the path.**
   > root → 'a' → 'p' → 'p' → 'l' → 'e' (marked as end-of-word). Each node represents one character.

4. **If "app" and "apple" are both in the trie, how many nodes hold shared data?**
   > 3 shared nodes: 'a', first 'p', and second 'p'. The word "app" marks the second 'p' as end-of-word; "apple" continues with 'l' and 'e'.

5. **What is the maximum depth of a trie?**
   > Equal to the length of the longest word stored in the trie.

---

## Navigation

| | |
|:---|---:|
| [📚 Back to README](../README.md) | [Next: Trie vs Hash Table ▶](02-trie-vs-hash-table.md) |
